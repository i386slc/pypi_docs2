# celery-heimda

**Celery Heimdall** — это набор общих утилит, полезных для среды фоновых рабочих процессов Celery, построенной на основе **Redis**. Он не пытается обработать каждый вариант использования, а является простым, современным и удобным в обслуживании решением для 90% проектов.

## Особенности

* Глобально уникальные задачи, позволяющие выполнять только одну копию задачи одновременно или в течение определенного периода времени (например: «Не разрешать стоять в очереди, пока не пройдет час»)
* Глобальное ограничение скорости. В Celery есть встроенное ограничение скорости, но это ограничение скорости на один воркер, что делает его непригодным для таких целей, как ограничение запросов к API.

## Установка

```bash
pip install celery-heimdall
```

## Использование

### Уникальные задачи

Представьте, что у вас есть задача, которая запускается, когда пользователь нажимает кнопку. Эта задача требует много времени и ресурсов для создания отчета. Вы же не хотите, чтобы пользователь нажимал кнопку 10 раз и запускал 10 задач. В этом случае вам нужно то, что **Heimdall** называет уникальной задачей:

```python
from celery import shared_task
from celery_heimdall import HeimdallTask

@shared_task(base=HeimdallTask, heimdall={'unique': True})
def generate_report(customer_id):
    pass
```

Все, что мы здесь сделали, это изменили базовый класс **Task**, который Celery будет использовать для запуска задачи, и передали некоторые параметры для использования **Heimdall**. Теперь эта задача уникальна — для заданных аргументов одновременно будет выполняться только 1.

### Срок действия

Что произойдет, если наша задача умрет или что-то пойдет не так? Мы можем оказаться в ситуации, когда наша блокировка никогда не очищается, что называется [тупиком](https://en.wikipedia.org/wiki/Deadlock). Чтобы обойти эту проблему, мы добавляем максимальное время, прежде чем задачу снова можно будет поставить в очередь:

```python
from celery import shared_task
from celery_heimdall import HeimdallTask

@shared_task(
  base=HeimdallTask,
  heimdall={
    'unique': True,
    'unique_timeout': 60 * 60
  }
)
def generate_report(customer_id):
  pass
```

Теперь **generate\_report** можно будет запустить снова через час, даже если задача зависла, у воркера закончилась память, машина загорелась и т. д.

### Пользовательские ключи

По умолчанию в качестве ключа блокировки используется хеш имени задачи и ее аргументов. Но часто это может быть не то, что вы хотите. Что делать, если вам нужен только один отчет за раз, даже для разных клиентов? Например:

```python
from celery import shared_task
from celery_heimdall import HeimdallTask

@shared_task(
  base=HeimdallTask,
  heimdall={
    'unique': True,
    'key': lambda args, kwargs: 'generate_report'
  }
)
def generate_report(customer_id):
  pass
```

Указав собственную ключевую функцию, мы можем полностью настроить способ определения уникальности задачи.

### Существующая задача

По умолчанию, если вы попытаетесь поставить в очередь уникальную задачу, которая уже выполняется, **Heimdall** вернет **AsyncResult** существующей задачи. Это позволяет вам писать простой код, которому не нужно заботиться об уникальности задачи или нет. Представьте себе простую конечную точку API, которая запускает отчет при его получении, но мы хотим, чтобы он запускался только по одному. Ниже это все, что вам нужно:

```python
import time
from celery import shared_task
from celery_heimdall import HeimdallTask

@shared_task(base=HeimdallTask, heimdall={'unique': True})
def generate_report(customer_id):
  time.sleep(10)

def my_api_call(customer_id: int):
  return {
    'status': 'RUNNING',
    'task_id': generate_report.delay(customer_id).id
  }
```

Каждый раз, когда **my\_api\_call** вызывается с одним и тем же идентификатором клиента **customer\_id**, тот же идентификатор задачи **task\_id** будет возвращен функцией `generate_report.delay()` до тех пор, пока исходная задача не будет завершена.

Иногда вам может понадобиться обнаружить, что задача уже выполнялась, когда вы попытались снова поставить ее в очередь. В этом случае мы можем сказать **Heimdall**, чтобы он вызвал исключение:

```python
import time
from celery import shared_task
from celery_heimdall import HeimdallTask, AlreadyQueuedError


@shared_task(
  base=HeimdallTask,
  heimdall={
    'unique': True,
    'unique_raises': True
  }
)
def generate_report(customer_id):
  time.sleep(10)


def my_api_call(customer_id: int):
  try:
    task = generate_report.delay(customer_id)
    return {'status': 'STARTED', 'task_id': task.id}
  except AlreadyQueuedError as exc:
    return {'status': 'ALREADY_RUNNING', 'task_id': exc.likely_culprit}
```

Если при определении нашей задачи для **unique\_raises** установлено значение `True`, при попытке дважды поставить в очередь уникальную задачу будет возникать ошибка **AlwaysQueuedError**. У **AlwaysQueuedError** есть два свойства:

* **likely\_culprit**, который содержит идентификатор уже запущенной задачи
* **expires\_in** — время, оставшееся (в секундах) до того, как уже запущенная задача будет считаться истекшей.

### Уникальная интервальная задача

Что, если мы хотим, чтобы задача запускалась только один раз в час, даже если она завершена? В таких случаях мы хотим, чтобы она запускалась, но не снимала блокировку после завершения:

```python
from celery import shared_task
from celery_heimdall import HeimdallTask

@shared_task(
  base=HeimdallTask,
  heimdall={
    'unique': True,
    'unique_timeout': 60 * 60,
    'unique_wait_for_expiry': True
  }
)
def generate_report(customer_id):
  pass
```

Если для **unique\_wait\_for\_expiry** установлено значение `True`, задача завершится и не позволит поставить в очередь другой метод `generate_report()`, пока не пройдет **unique\_timeout**.

### Ограничение скорости

Celery предлагает ограничение скорости «из коробки». Однако это ограничение ставки применяется в расчете на каждый воркер. Не существует надежного способа ограничить оценку задачи для всех ваших воркеров. **Heimdall** делает это легко:

```python
from celery import shared_task
from celery_heimdall import HeimdallTask, RateLimit

@shared_task(
  base=HeimdallTask,
  heimdall={
    'rate_limit': RateLimit((2, 60))
  }
)
def download_report_from_amazon(customer_id):
  pass
```

Здесь говорится: «Каждые 60 секунд разрешайте запуск этой задачи только 2 раза». Если задачу невозможно запустить, поскольку она нарушает ограничение скорости, она будет перенесена.

Важно отметить, что это не гарантирует, что ваша задача будет выполняться ровно два раза в секунду, просто она не будет выполняться чаще двух раз в секунду. Задачи перепланируются со случайным колебанием, чтобы предотвратить проблему [громоподобного стада](https://en.wikipedia.org/wiki/Thundering\_herd\_problem).

### Динамическое ограничение скорости

Точно так же, как вы можете динамически предоставлять ключ для задачи, вы также можете динамически предоставлять ограничение скорости на основе этого ключа.

```python
from celery import shared_task
from celery_heimdall import HeimdallTask, RateLimit


@shared_task(
  base=HeimdallTask,
  heimdall={
    # Предоставьте более низкий лимит скорости для клиента с идентификатором 10,
    # для всех остальных — более высокий лимит скорости.
    'rate_limit': RateLimit(lambda args: (1, 30) if args[0] == 10 else (2, 30)),
    'key': lambda args, kwargs: f'customer_{args[0]}'
  }
)
def download_report_from_amazon(customer_id):
  pass
```

Идеи

Это более зрелые проекты, которые вдохновили эту библиотеку и могут поддерживать более старые версии Celery и Python, чем этот проект.

* [celery\_once](https://github.com/cameronmaske/celery-once), который, к сожалению, заброшен и является причиной существования этого проекта.
* [celery\_singleton](https://github.com/steinitzu/celery-singleton)
* Этот [сниппет](https://gist.github.com/Vigrond/2bbea9be6413415e5479998e79a1b11a) от Vigrond и последующие улучшения, внесенные различными участниками.
