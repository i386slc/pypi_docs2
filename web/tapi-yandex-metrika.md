# tapi-yandex-metrika

## Документация по API управления счетчиками Яндекс Метрики

[Официальная документация API Яндекс Метрика](https://yandex.ru/dev/metrika/doc/api2/management/intro.html)

```python
from tapi_yandex_metrika import YandexMetrikaManagement

ACCESS_TOKEN = ""
COUNTER_ID = ""

client = YandexMetrikaManagement(
    access_token=ACCESS_TOKEN,
    default_url_params={'counterId': COUNTER_ID}
)
```

### Ресурсы

```python
print(dir(client))
['accounts', 'chart_annotation', 'chart_annotations', 'clients', 'counter',
 'counter_undelete', 'counters', 'delegate', 'delegates', 'filter', 'filters', 'goal',
 'goals', 'grant', 'grants', 'label', 'labels',
 'offline_conversions_calls_extended_threshold', 'offline_conversions_calls_uploading',
 'offline_conversions_calls_uploadings', 'offline_conversions_extended_threshold',
 'offline_conversions_upload', 'offline_conversions_upload_calls',
 'offline_conversions_uploading', 'offline_conversions_uploadings', 'operation',
 'operations', 'public_grant', 'segment', 'segments', 'set_counter_label',
 'user_params_upload', 'user_params_uploading', 'user_params_uploading_confirm',
 'user_params_uploadings', 'yclid_conversions_upload', 'yclid_conversions_uploading',
 'yclid_conversions_uploadings']

# Справочная информация о методе
client.counters().help()
# Documentation: https://yandex.ru/dev/metrika/doc/api2/management/direct_clients/getclients-docpage/
# Resource path: management/v1/clients
# Available HTTP methods:
# ['GET']
# Available query parameters:
# 'counters=<list>'

# Открыть документацию ресурса в браузере
client.counters().open_docs()
```

Как отправлять различные типы HTTP-запросов

* `:param params:` аргументы строки запроса в URL-адресе.
* `:param data:` отправить данные в теле запроса

```python
# Отправить HTTP-запрос 'GET'
client.counters().get(data: dict = None, params: dict = None)
# Отправить HTTP-запрос 'POST'
client.counters().post(data: dict = None, params: dict = None)
# Отправить HTTP-запрос 'DELETE'
client.counters().delete(data: dict = None, params: dict = None)
# Отправить HTTP-запрос 'PUT'
client.counters().put(data: dict = None, params: dict = None)
# Отправить HTTP-запрос 'PATCH'
client.counters().patch(data: dict = None, params: dict = None)
# Отправить HTTP-запрос 'OPTIONS'
client.counters().options(data: dict = None, params: dict = None)
```

```python
from tapi_yandex_metrika import YandexMetrikaManagement

client = YandexMetrikaManagement(...)

# Получить счетчики. Через метод HTTP GET.
counters = client.counters().get()
print(counters.data)

# Получить счетчики, отсортированные по посещениям. Через метод HTTP GET.
counters = client.counters().get(params={"sort": "Visits"})
print(counters.data)

# Создайте цель. Через метод HTTP POST.
body = {
        "goal": {
            "name": "2 страницы",
            "type": "number",
            "is_retargeting": 0,
            "depth": 2
        }
    }
client.goals().post(data=body)

# Создать цель для события JavaScript. Через метод HTTP POST.
body2 = {
    "goal": {
        "name": "Название вашей цели в метрике",
        "type": "action",
        "is_retargeting": 0,
        "conditions": [
                {
                    "type": "exact",
                    "url": <your_value>
                }
            ]
        }
    }
client.goals().post(data=body2)

# Для некоторых ресурсов необходимо подставлять идентификатор объекта в URL.
# Это делается путем добавления идентификатора к самому методу.
# Получить информацию о цели. Через метод HTTP GET.
client.goal(goalId=10000).get()

# Изменить цель. Через метод HTTP PUT.
body = {
    "goal" : {
        "id" : <int>,
        "name" :  <string> ,
        "type" :  <goal_type>,
        "is_retargeting" :  <boolean>,
        ...
    }
}
client.goal(goalId=10000).put(data=body)

# Удалить цель. С помощью метода HTTP DELETE.
client.goal(goalId=10000).delete()
```

Вы можете получить информацию по запросу.

```python
counters = client.counters().get()
print(counters.response)
print(counters.response.headers)
print(counters.status_code)
```

### Изменения

#### Релиз 2022.4.8

* Никаких изменений для этого API

#### Релиз 2021.5.28

* Добавить файл-заглушку (подсветка синтаксиса)

#### Релиз 2021.2.21

Новая особенность

* добавлен атрибут "data"
* добавлен атрибут "response"
