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

## Документация по загрузке отчетов из LOGS API Яндекс Метрики

[Официальная документация Яндекс Метрика LOGS API](https://yandex.ru/dev/metrika/doc/api2/api\_v1/data.html)

```python
from tapi_yandex_metrika import YandexMetrikaLogsapi

ACCESS_TOKEN = ""
COUNTER_ID = ""

client = YandexMetrikaLogsapi(
    access_token=ACCESS_TOKEN,
    default_url_params={'counterId': COUNTER_ID}
)

params = {
    "fields": "ym:s:date,ym:s:clientID",
    "source": "visits",
    "date1": "2021-01-01",
    "date2": "2021-01-01"
}

# Проверяет возможность создания отчета. Через метод HTTP GET.
result = client.evaluate().get(params=params)
print(result)


# Заказывает отчет. Через метод HTTP POST.
result = client.create().post(params=params)
request_id = result["log_request"]["request_id"]
print(result)


# Отменить создание отчета. Через метод HTTP POST.
result = client.cancel(requestId=request_id).post()
print(result)


# Удалить отчет. Через метод HTTP POST.
result = client.clean(requestId=request_id).post()
print(result)


# Получите информацию обо всех отчетах, хранящихся на сервере. Через метод HTTP GET.
result = client.allinfo().get()
print(result)


# Получите информацию о конкретном отчете. Через метод HTTP GET.
result = client.info(requestId=request_id).get()
print(result)


# Скачать отчет. Через метод HTTP POST.
result = client.create().post(params=params)
request_id = result["log_request"]["request_id"]

# Отчет можно скачать после его создания на сервере. Через метод HTTP GET.
info = client.info(requestId=request_id).get()

if info["log_request"]["status"] == "processed":

    # Отчет может состоять из нескольких частей.
    parts = info["log_request"]["parts"]
    print("Number of parts in the report", parts)

    # Параметр partNumber указывает номер части отчета, которую вы хотите загрузить.
    # По умолчанию partNumber=0
    part = client.download(requestId=request_id, partNumber=0).get()

    print("Raw data")
    data = part.data[:1000]

    print("Column names")
    print(part.columns)

    # Преобразование в значения
    print(part().to_values()[:3])

    # Преобразование в строки
    print(part().to_lines()[:3])

    # Преобразование в словари
    print(part().to_dicts()[:3])
else:
    print("Report not ready yet")
```

### Автоматически загружать отчет после его подготовки

добавить параметр **wait\_report**

```python
from tapi_yandex_metrika import YandexMetrikaLogsapi

ACCESS_TOKEN = ""
COUNTER_ID = ""

client = YandexMetrikaLogsapi(
    access_token=ACCESS_TOKEN,
    default_url_params={'counterId': COUNTER_ID},
    # Загрузить отчет, когда он будет создан
    wait_report=True,
)
params={
    "fields": "ym:s:date,ym:s:clientID,ym:s:dateTime,ym:s:startURL,ym:s:endURL",
    "source": "visits",
    "date1": "2019-01-01",
    "date2": "2019-01-01"
}
info = client.create().post(params=params)
request_id = info["log_request"]["request_id"]

report = client.download(requestId=request_id).get()

print("Raw data")
data = report.data

print("Column names")
print(report.columns)

# Преобразование в значения
print(report().to_values())

# Преобразование в строки
print(report().to_lines())

# Преобразование в словарь
print(report().to_dicts())
```

### Экспорт всех частей отчета

```python
from tapi_yandex_metrika import YandexMetrikaLogsapi

client = YandexMetrikaLogsapi(...)
info = client.create().post(params=...)
request_id = info["log_request"]["request_id"]
report = client.download(requestId=request_id).get()

print(report.columns)

# Части итерации
for part in report().parts():
    print(part.data)  # "сырые" данные
    print(part().to_values())
    print(part().to_lines())
    print(part().to_columns())  # столбцы с ориентацией на данные
    print(part().to_dicts())

for part in report().parts():
    # Строки итерации
    for row_as_text in part().lines():
        print(row_as_text)

    # Значения итерации
    for row_as_values in part().values():
        print(row_as_values)

    # Словари итерации
    for row_as_dict in part().dicts():
        print(row_as_dict)
```

### Перебрать все строки всех частей отчета

Будет перебирать все строки всех частей

```python
from tapi_yandex_metrika import YandexMetrikaLogsapi

client = YandexMetrikaLogsapi(...)
info = client.create().post(params=...)
request_id = info["log_request"]["request_id"]
report = client.download(requestId=request_id).get()

print(report.columns)

for row_as_line in report().iter_lines():
    print(row_as_line)

for row_as_values in report().iter_values():
    print(row_as_values)

for row_as_dict in report().iter_dicts():
    print(row_as_dict)
```

### Ограничение итерации

```python
.parts(max_parts: int = None)
.lines(max_rows: int = None)
.values(max_rows: int = None)
.iter_lines(max_parts: int = None, max_rows: int = None)
.iter_values(max_parts: int = None, max_rows: int = None)
```

### Ответ

```python
from tapi_yandex_metrika import YandexMetrikaLogsapi

client = YandexMetrikaLogsapi(...)
info = client.create().post(params=...)

print(info.data)
print(info.response)
print(info.response.headers)
print(info.status_code)

report = client.download(requestId=info["log_request"]["request_id"]).get()

for part in report().parts():
    print(part.data)
    print(part.response)
    print(part.response.headers)
    print(part.response.status_code)
```

### Предупреждение

Обратите внимание на то, каким HTTP-методом вы отправляете запрос. Некоторые ресурсы работают только с POST или только с GET запросами. Например, создание ресурса только с помощью метода **POST**.

```python
client.create().post(params=params)
```

И оценивание результата только с помощью метода **GET**.

```python
client.evaluate().get(params=params)
```

### Изменения

#### Релиз 2022.4.8

* Прекращает ожидание отчета, если статус отчета недействителен, вызвав ошибку.

#### Релиз 2021.5.28

* Добавить файл-заглушку (подсветка синтаксиса)

#### Релиз 2021.5.15

* добавить метод итерации "dicts"
* добавить метод итерации "iter\_dicts"
* добавить метод "to\_dicts"
* переименование параметра **max\_items** в **max\_rows** в **iter\_lines**, **iter\_values**, **lines**, **values**

#### Релиз 2021.2.21

Обратная несовместимость изменений

* Метод удаления "transform"
* Удаление параметра "receive\_all\_data"

Новая особенность

* переведен на английский
* добавлен метод итерации "parts"
* добавлен метод итерации "lines"
* добавлен метод итерации "values"
* добавлен метод итерации "iter\_lines"
* добавлен метод итерации "iter\_values"
* добавлен атрибут "columns"
* добавлен атрибут "data"
* добавлен атрибут "response"
* добавлен метод "to\_lines"
* добавлен метод "to\_values"
* добавлен метод "to\_columns"

### Авторы

Павел Максимов - [Telegram](https://t.me/pavel\_maksimow), [Facebook](https://www.facebook.com/pavel.maksimow)

Удачи друг! Ставьте звездочку ;)

Copyright (c) Павел Максимов.
