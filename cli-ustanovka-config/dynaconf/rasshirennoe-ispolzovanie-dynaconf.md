# Расширенное использование dynaconf

## Хуки

> НОВОЕ в версии 3.1.6

Хуки полезны, когда вам нужно условно загрузить данные, которые зависят от ранее загруженных настроек.

Прежде чем углубляться в хуки, давайте разберемся, почему они полезны.

Представьте, что ваше приложение использует 2 модуля настроек: `plugin_settings.py` и `settings.py`.

`plugin_settings.py`

```python
DEBUG = True
```

`settings.py`

```python
if DEBUG:
    # сделать что-то
```

`app.py`

```python
from dynaconf import Dynaconf
settings = Dynaconf(
    preload=['plugin_settings.py'],
    settings_file="settings.py"
)
```

Приведенный выше код завершится ошибкой `NameError: name 'DEBUG' is not defined on settings.py`, что происходит потому, что `plugin_settings.py` загружается до `settings.py`, но также до полной загрузки `app.py`.

### Хуки для решения

Хук — это, по сути, функция, которая принимает необязательный позиционный аргумент **settings**, доступный только для чтения, и возвращает данные для объединения с объектом **Settings**.

Есть два способа добавления хуков: из специальных модулей или непосредственно в экземпляре Dynaconf.

### Модульный подход

Используя модульный подход, вы можете создать файл `dynaconf_hooks.py` по тому же пути, что и любой файл настроек. Затем хуки из этого модуля будут загружены после обычного процесса загрузки.

```python
# dynaconf_hooks.py

def post(settings):
    data = {"dynaconf_merge": True}
    if settings.DEBUG:
        data["DATABASE_URL"] = "sqlite://"
    return data
```

Dynaconf выполнит функцию **post** и объединит возвращенные данные с существующими настройками.

### Подход с экземплярами объекта

При использовании экземплярного подхода просто добавьте функцию-перехватчик в аргумент инициализации Dynaconf **post\_hook**. Он принимает один вызываемый объект `Callable` или список вызываемых объектов `Callable`.

```python
def hook_function(settings):
    data = {"dynaconf_merge": True}
    if settings.DEBUG:
        data["DATABASE_URL"] = "sqlite://"
    return data

settings = Dynaconf(post_hooks=hook_function)
```

Вы также можете настроить слияние индивидуально для каждой переменной настроек, как показано в [документации по слиянию](sliyanie-merging.md).

## Анализ истории

> НОВОЕ в версии 3.2.0

{% hint style="warning" %}
Эта функция находится на стадии **технической предварительной версии**. Интерфейс использования и формат вывода могут быть изменены.
{% endhint %}

Цель этой функции — позволить отслеживать любую историю загрузки данных конфигурации, то есть шаги загрузки, которые приводят к заданному конечному значению. Он может возвращать отчет о данных dict, печатать на стандартный вывод **stdout** или записывать в файл, а также доступен как [команда CLI](cli-dynaconf.md#dynaconf-inspect-tech-preview).

Он работает, устанавливая объект **SourceMetadata** для всех принятых данных, чтобы их можно было восстановить и отфильтровать для создания содержательных отчетов. Свойства этого объекта:

* **loader**: тип загрузчика (например, yaml, envvar, validation\_default)
* **identifier**: конкретный идентификатор (например, имя файла для файлов)
* **env**: к какой среде принадлежат эти данные (global, main, development и т. д.)
* **merged**: если эти данные были объединены (True или False)

Dynaconf предлагает две очень похожие служебные функции для получения данных: **inspect\_settings** и **get\_history**. Разница в том, что **get\_history** возвращает простой список dict-записей с исходными метаданными, в то время как **inspect\_settings** фокусируется на создании отчета для печати (и, таким образом, он предлагает для этого некоторые удобные опции).

Они доступны по адресу `dynaconf.inspect_settings` и `dynaconf.get_history`.

### `inspect_settings` <a href="#inspect_settings" id="inspect_settings"></a>

По умолчанию он возвращает только dict-отчет. При желании вы можете распечатать отчет в определенном формате, например "yaml" или "json", или записать его в файл:

```python
# по умолчанию: возвращает объект Python
>>> inspect_settings(settings_obj)
{
    "header": ... # используемые параметры фильтрации
    "current": ... # текущие активные данные/значение
    "history": ... # список записей истории
}

# печать отчета
>>> inspect_settings(setting_obj, print_report=True, dumper="yaml")
header:
  - ...
current:
  - ...
history:
  - ...

# запись в файл
>>> inspect_settings(settings_obj, to_file="report.yaml", dumper="json")
```

История выглядит так:

```python
>>> inspect_settings(
>>>     settings_obj,
>>>     key="foo",
>>>     print_report=True,
>>>     dumper="yaml"
>>> )
>>>

header:
- ...
current:
- ...
history:
- loader: env_global
  identifier: unique
  env: global
  merged: false
  value: FOO
- loader: toml
  identifier: path/to/file.yaml
  env: development
  merged: false
  value: FOO
```

### Обзор опций

Существует несколько предопределенных способов фильтрации и настройки отчета. Вот краткий обзор доступных вариантов аргументов:

Позиционные:

* **settings** (обязательно): экземпляр Dynaconf.
* **key**: фильтровать по этому ключу. Например, `"path.to.key"`
* **env**: Фильтровать по этому окружению. Например, `"production"`.

Только kwargs:

* **new\_first**: Если `True`, используется порядок загрузки от самого нового к самому старому.
* **history\_limit**: ограничивает количество отображаемых записей. По умолчанию показывает все.
* **include\_internal**: если `True`, включить внутренние загрузчики (например, defaults). Это имеет эффект только в том случае, если ключ не предоставлен.
* **to\_file**: если указано, записать дамп отчета в файл с этим именем.
* **print\_report**: если `True`, выводит дамп отчета на стандартный вывод.
* **dumper**: принимает заданные строки (например, "yaml", "json") или вызываемый пользовательский дампер `(dict, TextIO) -> None`. Если не указано, ничего не делает.
* **report\_builder**: если он указан, он используется для создания словаря отчета.

### `get_history` <a href="#get_history" id="get_history"></a>

Возвращает список записей истории (тот же, что и в ключе **history** записей **inspect\_settings**). Фактически, **inspect\_settings** использует **get\_history**, поэтому это доступно только в том случае, если ваша главная цель — использовать данные напрямую. Он предлагает некоторые базовые возможности фильтрации, но предполагается, что если вы решите использовать это, вы, вероятно, захотите обрабатывать и фильтровать данные самостоятельно.

## Обновление настроек dynaconf, используя аргументы командной строки (cli)

Dynaconf предназначен для загрузки файла настроек и, если применимо, определения приоритета переопределений из envvar.

Следующие примеры демонстрируют процесс включения аргументов командной строки для обновления настроек Dynaconf, гарантируя, что входные данные CLI имеют приоритет как над файлом `settings.toml`, так и над переменными среды.

### Аргументы командной строки с использованием argparse

На этом примере показано использование модуля Python **argparse** для иллюстрации создания интерфейса командной строки (CLI), способного переопределять файл `settings.toml` и переменные среды.

#### app.py

```python
from __future__ import annotations

from dynaconf import settings

import argparse


def main(argv=None):
    parser = argparse.ArgumentParser(
        description="Simple argparse example for overwrite dynaconf settings"
    )

    parser.add_argument("--env", default=settings.current_env)
    parser.add_argument("--host", default=settings.HOST)
    parser.add_argument("--port", default=settings.PORT)

    options, args = parser.parse_known_args(argv)

    # изменяет среду, чтобы обновить правильные настройки
    settings.setenv(options.env)

    # обновляет настройки dynaconfig
    settings.update(vars(options))

    #

if __name__ == "__main__":
    main(argv=None)
```

### Аргументы командной строки с использованием click

На этом примере показано использование модуля Python **click** для иллюстрации создания интерфейса командной строки (CLI), способного переопределять файл `settings.toml` и переменные среды.

#### app.py

```python
from __future__ import annotations

from dynaconf import settings

import click


@click.command()
@click.option("--host", default=settings.HOST, help="Host")
@click.option("--port", default=settings.PORT, help="Port")
@click.option("--env", default=settings.current_env, help="Env")
def app(host, port, env):
    """Простой пример click для перезаписи настроек dynaconf"""

    # изменяет среду, чтобы обновить правильные настройки
    settings.setenv(env)

    # обновляет настройки dynaconfig
    settings.update({"host": host, "port": port})

if __name__ == "__main__":
    app()
```

В качестве альтернативы вы можете использовать более общий подход, используя `**options` и передавая параметры в dynaconf.

#### app.py

```python
from __future__ import annotations

from dynaconf import settings

import click


@click.command()
@click.option("--host", default=settings.HOST, help="Host")
@click.option("--port", default=settings.PORT, help="Port")
@click.option("--env", default=settings.current_env, help="Env")
def app(**options):
    """Простой пример click для перезаписи настроек dynaconf"""

    # изменяет среду, чтобы обновить правильные настройки
    settings.setenv(options['env'])

    # обновляет настройки dynaconfig
    settings.update(options)

if __name__ == "__main__":
    app()
```

## Создание новых загрузчиков
