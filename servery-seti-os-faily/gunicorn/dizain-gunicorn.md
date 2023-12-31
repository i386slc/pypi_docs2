# Дизайн gunicorn

Краткое описание архитектуры **Gunicorn**.

## Модель сервера

**Gunicorn** основан на модели **pre-fork worker**. Это означает, что существует центральный главный процесс, который управляет набором рабочих процессов. Мастер никогда ничего не знает об отдельных клиентах. Все запросы и ответы полностью обрабатываются рабочими процессами.

### Master

Главный процесс представляет собой простой цикл, который прослушивает различные сигналы процесса и реагирует соответствующим образом. Он управляет списком запущенных воркеров, прослушивая такие сигналы, как **TTIN**, **TTOU** и **CHLD**. **TTIN** и **TTOU** указывают мастеру увеличить или уменьшить количество работающих рабочих. **CHLD** указывает, что дочерний процесс завершен, в этом случае главный процесс автоматически перезапускает отказавший рабочий процесс.

### Синхронные воркеры

Самый простой и используемый по умолчанию тип воркера — это синхронный класс воркера, который обрабатывает один запрос за раз. Эта модель является самой простой для понимания, поскольку любые ошибки влияют не более чем на один запрос. Хотя, как мы опишем ниже, обработка только одного запроса за раз требует некоторых предположений о том, как программируются приложения.

**sync worker** не поддерживает постоянные соединения — каждое соединение закрывается после отправки ответа (даже если вы вручную добавите заголовок **Keep-Alive** или **Connection: keep-alive** в своем приложении).

### Асинхронные воркеры

Доступные асинхронные процессы воркеров основаны на [Greenlets](https://github.com/python-greenlet/greenlet) (через [Eventlet](http://eventlet.net/) и [Gevent](http://www.gevent.org/)). **Greenlets** — это реализация совместной многопоточности для Python. В общем, приложение должно иметь возможность использовать эти классы воркеров без каких-либо изменений.

Для полной поддержки приложений **Greenlet** может потребоваться адаптация. При использовании, например, [Gevent](http://www.gevent.org/) и [Psycopg](http://initd.org/psycopg/) имеет смысл убедиться, что [psycogreen](https://bitbucket.org/dvarrazzo/psycogreen) установлен и [настроен](http://www.gevent.org/api/gevent.monkey.html#plugins).

Другие приложения могут быть вообще несовместимы, поскольку они, например, полагаются на исходное неисправленное поведение.

### Воркеры Tornado

Также есть класс воркера **Tornado**. Его можно использовать для написания приложений с использованием фреймворка **Tornado**. Хотя рабочие процессы **Tornado** могут обслуживать приложения **WSGI**, это не рекомендуемая конфигурация.

### Воркеры AsyncIO

Эти процессы воркеров совместимы с Python 3.

Воркер **gthread** — это многопоточный рабочий поток. Он принимает соединения в основном цикле, принятые соединения добавляются в пул потоков как задание соединения. При сохранении активности соединения возвращаются в цикл ожидания события. Если по истечении тайм-аута поддержания активности не происходит никаких событий, соединение закрывается.

Вы также можете портировать свое приложение, чтобы использовать API **web.Application** [aiohttp](https://docs.aiohttp.org/en/stable/deployment.html#nginx-gunicorn) и использовать воркер **aiohttp.worker.GunicornWebWorker**.

## Выбор типа воркера

Синхронные процессы воркеров по умолчанию предполагают, что ваше приложение ограничено ресурсами с точки зрения пропускной способности CPU и сети. Обычно это означает, что ваше приложение не должно делать ничего, что занимает неопределенное количество времени. Примером того, что занимает неопределенное время, является запрос в Интернет. В какой-то момент внешняя сеть даст сбой, и клиенты скопятся на ваших серверах. Таким образом, в этом смысле любое веб-приложение, которое делает исходящие запросы к API, выиграет от асинхронного рабочего процесса.

Это предположение о привязке к ресурсам является причиной того, что нам требуется прокси-сервер буферизации перед конфигурацией **Gunicorn** по умолчанию. Если бы вы предоставили синхронным рабочим процессам доступ к Интернету, DOS-атака была бы тривиальной, поскольку создавала нагрузку, которая передавала бы данные на серверы. Для любопытных, [Hey](https://github.com/rakyll/hey) является примером такого типа нагрузки.

Некоторые примеры поведения, требующего асинхронных рабочих процессов:

* Приложения, выполняющие длительные блокирующие вызовы (например, внешние веб-сервисы)
* Обслуживание запросов непосредственно в Интернете
* Потоковые запросы и ответы
* Долгий опрос
* Веб-сокеты
* Comet

## Сколько воркеров?

НЕ масштабируйте количество воркеров до количества клиентов, которых вы ожидаете. **Gunicorn** требуется всего 4-12 рабочих процессов для обработки сотен или тысяч запросов в секунду.

**Gunicorn** полагается на операционную систему для обеспечения всей балансировки нагрузки при обработке запросов. Обычно мы рекомендуем `(2 x $num_cores) + 1` в качестве количества воркеров для начала. Хотя формула не слишком научна, она основана на предположении, что для данного ядра один рабочий процесс будет читать или писать из сокета, в то время как другой рабочий процесс обрабатывает запрос.

Очевидно, что ваше конкретное оборудование и приложение повлияют на оптимальное количество воркеров. Мы рекомендуем начать с приведенного выше предположения и настроить с помощью сигналов **TTIN** и **TTOU**, пока приложение находится под нагрузкой.

Всегда помните, что бывает слишком много воркеров. Через некоторое время ваши рабочие процессы начнут перегружать системные ресурсы, уменьшая пропускную способность всей системы.

## Сколько потоков?

Начиная с **Gunicorn 19**, параметр **threads** можно использовать для обработки запросов в несколько потоков. Использование потоков предполагает использование рабочего процесса **gthread**. Одним из преимуществ потоков является то, что запросы могут занимать больше времени, чем время ожидания рабочего процесса, при этом главный процесс уведомляется о том, что он не заморожен и не должен быть уничтожен. В зависимости от системы использование нескольких потоков, нескольких рабочих процессов или некоторой их комбинации может дать наилучшие результаты. Например, **CPython** может работать не так хорошо, как **Jython**, при использовании потоков, поскольку потоки реализованы по-разному в каждом из них. Использование потоков вместо процессов — хороший способ уменьшить объем памяти **Gunicorn**, в то же время позволяя обновлять приложения с помощью сигнала перезагрузки, поскольку код приложения будет совместно использоваться рабочими процессами, но загружаться только в рабочие процессы (в отличие от использования параметра **preload**, который загружает код в мастер-процесс).

{% hint style="info" %}
В Python 2.x вам необходимо установить пакет `'futures'`, чтобы использовать эту функцию.
{% endhint %}
