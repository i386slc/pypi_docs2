# Начало работы с kombu

* Версия: 5.3.2
* Веб: [https://kombu.readthedocs.io/](https://kombu.readthedocs.io/)
* Скачать: [https://pypi.org/project/kombu/](https://pypi.org/project/kombu/)
* Исходник: [https://github.com/celery/kombu/](https://github.com/celery/kombu/)
* Ключевые слова: messaging, amqp, rabbitmq, redis, mongodb, python, queue

## О Kombu

**Kombu** — это библиотека сообщений для Python.

Цель **Kombu** — максимально упростить обмен сообщениями на Python, предоставив идиоматический высокоуровневый интерфейс для протокола **AMQ**, а также предоставить проверенные и протестированные решения распространенных проблем обмена сообщениями.

[AMQP](https://amqp.org/) — это расширенный протокол очереди сообщений, открытый стандартный протокол для ориентации сообщений, организации очередей, маршрутизации, надежности и безопасности, для которого сервер обмена сообщениями [RabbitMQ](https://www.rabbitmq.com/) является наиболее популярной реализацией.

## Функции

* Позволяет авторам приложений поддерживать несколько решений серверов сообщений с помощью подключаемых транспортных средств.
  * Транспорт AMQP с использованием библиотек [py-amqp](https://pypi.org/project/amqp/), [redis](https://redis.io/) или ['SQS'](https://docs.celeryq.dev/projects/kombu/en/stable/introduction.html#id10).
  * Виртуальные транспорты позволяют очень легко добавить поддержку транспортов, отличных от AMQP. Уже имеется встроенная поддержка [Redis](https://redis.io/), [Amazon SQS](https://aws.amazon.com/sqs/), [Azure Storage Queues](https://azure.microsoft.com/en-us/services/storage/queues/), [Azure Service Bus](https://azure.microsoft.com/en-us/services/service-bus/), [ZooKeeper](https://zookeeper.apache.org/), [SoftLayer MQ](http://www.softlayer.com/services/additional/message-queue), [MongoDB](https://www.mongodb.com/) и [Pyro](https://pyro4.readthedocs.io/).
  * Перенос в памяти для модульного тестирования.
* Поддерживает автоматическое кодирование, сериализацию и сжатие полезных данных сообщений.
* Согласованная обработка исключений на всех транспортах.
* Возможность гарантировать выполнение операции путем корректной обработки ошибок соединения и канала.
* Исправлено несколько недостатков [amqplib](http://barryp.org/software/py-amqplib/), таких как поддержка таймаутов и возможность ожидания событий на нескольких каналах.
* Проекты, уже использующие [carrot](https://pypi.org/project/carrot/), можно легко портировать, используя уровень совместимости.

Для знакомства с AMQP вам следует прочитать статью [Rabbits and Warrens](http://web.archive.org/web/20160323134044/http://blogs.digitar.com/jjww/2009/01/rabbits-and-warrens/) и статью в [Википедии об AMQP](http://en.wikipedia.org/wiki/AMQP).

## Сравнение транспорта

<table data-full-width="false"><thead><tr><th width="133">Клиент</th><th width="144">Тип</th><th width="78">Direct</th><th width="80">Topic</th><th width="115">Fanout</th><th width="116">Приоритет</th><th width="100">TTL</th></tr></thead><tbody><tr><td>amqp</td><td>Родной</td><td>Да</td><td>Да</td><td>Да</td><td>Да [3]</td><td>Да [4]</td></tr><tr><td>qpid</td><td>Родной</td><td>Да</td><td>Да</td><td>Да</td><td>Нет</td><td>Нет</td></tr><tr><td>redis</td><td>Виртуальный</td><td>Да</td><td>Да</td><td>Да (PUB/SUB)</td><td>Да</td><td>Нет</td></tr><tr><td>mongodb</td><td>Виртуальный</td><td>Да</td><td>Да</td><td>Да</td><td>Да</td><td>Да</td></tr><tr><td>SQS</td><td>Виртуальный</td><td>Да</td><td>Да [1]</td><td>Да [2]</td><td>Нет</td><td>Нет</td></tr><tr><td>zookeeper</td><td>Виртуальный</td><td>Да</td><td>Да [1]</td><td>Нет</td><td>Да</td><td>Нет</td></tr><tr><td>in-memory</td><td>Виртуальный</td><td>Да</td><td>Да [1]</td><td>Нет</td><td>Нет</td><td>Нет</td></tr><tr><td>SLMQ</td><td>Виртуальный</td><td>Да</td><td>Да [1]</td><td>Нет</td><td>Нет</td><td>Нет</td></tr><tr><td>Pyro</td><td>Виртуальный</td><td>Да</td><td>Да [1]</td><td>Нет</td><td>Нет</td><td>Нет</td></tr></tbody></table>

* \[1] (1, 2, 3, 4, 5) - Объявления хранятся только в памяти, поэтому обмены/очереди должны объявляться всеми клиентами, которым они нужны.
* \[2] - Fanout поддерживается за счет хранения таблиц маршрутизации в **SimpleDB**. По умолчанию отключено, но его можно включить с помощью опции транспорта **support\_fanout**.
* \[3] - Поддержка приоритета сообщений **AMQP** зависит от реализации брокера.
* \[4] - Поддержка TTL сообщения/очереди **AMQP** зависит от реализации брокера.

## Документация

Kombu использует **Sphinx**, а последнюю версию документации можно найти здесь:

[https://kombu.readthedocs.io/](https://kombu.readthedocs.io/)

### Краткая информация

```python
from kombu import Connection, Exchange, Queue

media_exchange = Exchange('media', 'direct', durable=True)
video_queue = Queue('video', exchange=media_exchange, routing_key='video')

def process_media(body, message):
    print(body)
    message.ack()

# соединения
with Connection('amqp://guest:guest@localhost//') as conn:

    # создание
    producer = conn.Producer(serializer='json')
    producer.publish({'name': '/tmp/lolcat1.avi', 'size': 1301013},
                      exchange=media_exchange, routing_key='video',
                      declare=[video_queue])

    # объявление выше гарантирует, что видеоочередь объявлена так,
    # что сообщения могут быть доставлены.
    # В Kombu рекомендуется, чтобы и производители, и потребители объявляли очередь.
    # Вы также можете объявить очередь вручную, используя:
    #     video_queue(conn).declare()

    # потребление
    with conn.Consumer(video_queue, callbacks=[process_media]) as consumer:
        # Обработать сообщения и обработать события по всем каналам
        while True:
            conn.drain_events()

# Потребление из нескольких очередей на одном канале:
video_queue = Queue('video', exchange=media_exchange, key='video')
image_queue = Queue('image', exchange=media_exchange, key='image')

with connection.Consumer([video_queue, image_queue],
                         callbacks=[process_media]) as consumer:
    while True:
        connection.drain_events()
```

Или обрабатывайте каналы вручную:

```python
with connection.channel() as channel:
    producer = Producer(channel, ...)
    consumer = Consumer(channel)
```

Все объекты можно использовать и вне контекстного менеджера **with**, просто не забудьте закрыть объекты после использования:

```python
from kombu import Connection, Consumer

connection = Connection()
    # ...
connection.release()

consumer = Consumer(channel_or_connection, ...)
consumer.register_callback(my_callback)
consumer.consume()
    # ....
consumer.cancel()
```

**Exchange** и **Queue** — это просто объявления, которые можно выбрать и использовать в файлах конфигурации и т. д.

Они также поддерживают операции, но для этого их необходимо привязать к каналу.

Привязка обменов и очередей к соединению заставит его использовать канал этого соединения по умолчанию.

```python
>>> exchange = Exchange('tasks', 'direct')

>>> connection = Connection()
>>> bound_exchange = exchange(connection)
>>> bound_exchange.delete()

# исходный обмен не затрагивается и остается несвязанным.
>>> exchange.delete()
raise NotBoundError: Can't call delete on Exchange not bound to
    a channel.
```

## Терминология

Прежде чем начать, вам следует ознакомиться с некоторыми понятиями:

#### Производители (producers)

Производители отправляют сообщения в exchange.

#### Обмены (exchanges)

Сообщения отправляются в обмены. Обменам присваиваются имена, и их можно настроить на использование одного из нескольких алгоритмов маршрутизации. Обмен маршрутизирует сообщения потребителям, сопоставляя ключ маршрутизации (_**routing key**_) в сообщении с ключом маршрутизации, который потребитель предоставляет при привязке к обмену.

#### Потребители (consumers)

Потребитель объявляет очередь, привязывает ее к обмену и получает от нее сообщения.

#### Очереди (queues)

Очереди принимают сообщения, отправленные на обмен. Очереди объявляются потребителями.

#### Ключи маршрутизации (_**routing keys**_)

Каждое сообщение имеет ключ маршрутизации. Интерпретация ключа маршрутизации зависит от типа обмена. Существует четыре типа обмена по умолчанию, определенные стандартом **AMQP**, и поставщики могут определять собственные типы (подробную информацию см. в руководстве поставщика).

Это типы обмена по умолчанию, определенные `AMQP/0.8`:

* Прямой обмен (direct exchange) - Совпадает, если свойство ключа маршрутизации сообщения и атрибут **router\_key** потребителя идентичны.
* Обмен fan-out (fan-out exchange) - Всегда совпадает, даже если привязка не имеет ключа маршрутизации.
* Обмен темами (topic exchange) - Сопоставляет свойство ключа маршрутизации сообщения с помощью примитивной схемы сопоставления шаблонов. В этом случае ключ маршрутизации сообщений состоит из слов, разделенных точками (`"."`, как и имена доменов), и доступны два специальных символа; звездочка (`"*"`) и решетка (`«#»`). Звездочка соответствует любому слову, а хэш соответствует нулю или более словам. Например, `"*.stock.#"` соответствует ключам маршрутизации `"usd.stock"` и `"eur.stock.db"`, но не `"stock.nasdaq"`.

## Установка

Вы можете установить **Kombu** либо через индекс пакетов Python (PyPI), либо из исходного кода.

Для установки с помощью **pip**:

```bash
pip install kombu
```

Для установки с помощью **easy\_install**:

```bash
easy_install kombu
```

Если вы загрузили исходный архив, вы можете установить его, выполнив следующие действия:

```python
python setup.py build

python setup.py install # as root
```

## Получить помощь

### Список рассылки

Присоединяйтесь к списку рассылки [пользователей carrot](http://groups.google.com/group/carrot-users/).

## Баг трекер

Если у вас есть какие-либо предложения, сообщения об ошибках или замечания, сообщите о них в нашу систему отслеживания проблем по адресу: [http://github.com/celery/kombu/issues/](http://github.com/celery/kombu/issues/)

## Содействие

Разработка **Kombu** происходит на Github: [http://github.com/celery/kombu](http://github.com/celery/kombu)

Приглашаем вас принять участие в разработке. Если вам не нравится Github (по какой-то причине), вы можете регулярно присылать патчи.

## Лицензия

Это программное обеспечение распространяется по лицензии **New BSD**. Полный текст лицензии см. в файле **LICENSE** в верхнем каталоге распространения.
