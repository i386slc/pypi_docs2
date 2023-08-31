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

## Программная загрузка файла настроек

Вы можете загружать файлы из скрипта Python.

При использовании относительных путей в качестве базового пути будет использоваться **root\_path**. Узнайте больше о том, как работает резервный вариант **root\_path**, [здесь](konfiguraciya-dynaconf.md#settings\_file-or-settings\_files).

```python
from dynaconf import Dynaconf

settings = Dynaconf()

# единственный файл
settings.load_file(path="/path/to/file.toml")

# список
settings.load_file(path=["/path/to/file.toml", "/path/to/another-file.toml"])

# разделены по ; или ,
settings.load_file(path="/path/to/file.toml;/path/to/another-file.toml")
```

Обратите внимание, что данные, загруженные этим методом, не сохраняются.

Как только **env** будет изменен с помощью вызова `setenv|using_env`, `reload` или `configure`, загруженные данные будут очищены. Чтобы сохранить это, рассмотрите возможность использования переменной **INCLUDES\_FOR\_DYNACONF** или убедитесь, что она будет загружена программно снова.

## Фильтрация префиксов

```toml
[production]
PREFIX_VAR = TEST
OTHER = FOO
```

```python
from dynaconf import Dynaconf
from dynaconf.strategies.filtering import PrefixFilter

settings = Dynaconf(
    settings_file="settings.toml",
    environments=False,
    filter_strategy=PrefixFilter("prefix")
)
```

Загружает только переменные с префиксом **prefix\_**

## Создание новых загрузчиков

В вашем проекте, т.е. под названием **myprogram**, создайте свой собственный загрузчик.

#### `myprogram/my_custom_loader.py`

```python
def load(obj, env=None, silent=True, key=None, filename=None):
    """
    Считывает и загружает в "obj" один ключ или все ключи из источника.
    :param obj: экземпляр настроек
    :param env: настройки текущего окружения (заглавные буквы) default='DEVELOPMENT'
    :param silent: если ошибки нужно их поднять
    :param key: если определена загрузка одного ключа, иначе загружайте все из `env`
    :param filename: Пользовательское имя файла для загрузки (полезно для тестов)
    :return: None
    """
    # Загрузите данные из вашего пользовательского источника данных
    # (файл, база данных, память и т. д.)
    # используйте `obj.set(key, value)` или `obj.update(dict)` чтобы загрузить данные
    # используйте `obj.find_file('filename.ext')` чтобы найти файл в дереве поиска
    # Ничего не возвращает
```

В файле `.env` или при экспорте envvar определите:

```bash
LOADERS_FOR_DYNACONF=['myprogram.my_custom_loader', 'dynaconf.loaders.env_loader']
```

Dynaconf импортирует ваш файл `myprogram.my_custom_loader.load` и вызовет его.

{% hint style="info" %}
**ВАЖНО**: `"dynaconf.loaders.env_loader"` должен быть последним в списке загрузчиков, если вы хотите сохранить поведение envvars для переопределения параметров.
{% endhint %}

В случае, если вам необходимо отключить все внешние загрузчики и полагаться только на `settings.*` загрузчики файлов определяют:

```python
LOADERS_FOR_DYNACONF=false
```

Если вам нужно отключить все основные загрузчики и полагаться только на внешние загрузчики:

```toml
CORE_LOADERS_FOR_DYNACONF='[]'  # пустой список TOML
```

Например, если вы хотите добавить загрузчик [SOPS](https://github.com/mozilla/sops)

```python
def load(
    obj: LazySettings,
    env: str = "DEVELOPMENT",
    silent: bool = True,
    key: str = None,
    filename: str = None,
) -> None:
    sops_filename = f"secrets.{env}.yaml"
    sops_file = obj.find_file(sops_filename)
    if not sops_file:
        logger.error(f"{sops_filename} not found! Secrets not loaded!")
        return

    _output = run(["sops", "-d", sops_file], capture_output=True)
    if _output.stderr:
        logger.warning(f"SOPS error: {_output.stderr}")
    decrypted_config = yaml.load(_output.stdout, Loader=yaml.CLoader)

    # поддержка при анализе данных inspect
    source_metadata = SourceMetadata('sops', sops_file, env)

    if key:
        value = decrypted_config.get(key.lower())
        obj.set(key, value, loader_identifier=source_metadata)
    else:
        obj.update(decrypted_config, loader_identifier=source_metadata)

    obj._loaded_files.append(sops_file)
```

Посмотреть больше [test\_functional/custom\_loader](https://github.com/dynaconf/dynaconf/tree/master/tests\_functional/custom\_loader)

## Подражание модулю

В некоторых случаях вам может потребоваться выдать себя за устаревший модуль настроек **settings**, например, у вас уже есть программа, которая это делает.

```python
from myprogram import settings
```

и теперь вы хотите использовать dynaconf без необходимости менять всю кодовую базу.

Перейдите в `myprogram/settings.py` и примените подражание модулю.

```python
import sys
from dynaconf import LazySettings

sys.modules[__name__] = LazySettings()
```

последняя строка приведенного выше кода заставит модуль заменить себя экземпляром dynaconf при первом импорте.

## Переключение рабочей среды

Вы можете переключаться между существующими средами, используя:

* **from\_env**: (_**рекомендуется**_) Создаст новый экземпляр настроек, указывающий на определенный **env**.
* **setenv**: установит для существующего экземпляра определенный **env**.
* **using\_env**: Менеджер контекста, который будет определять **env** только внутри своей области действия.

### from\_env

> Новое в версии 2.1.0

Вернуть новый изолированный объект настроек, указывающий на уточненный **env**.

Пример `settings.toml`:

```toml
[development]
message = 'This is in dev'
foo = 1
[other]
message = 'this is in other env'
bar = 2
```

Программа:

```python
>>> from dynaconf import settings
>>> print(settings.MESSAGE)
'This is in dev'
>>> print(settings.FOO)
1
>>> print(settings.BAR)
AttributeError: settings object has no attribute 'BAR'
```

Затем вы можете использовать **from\_env**:

```python
>>> print(settings.from_env('other').MESSAGE)
'This is in other env'
>>> print(settings.from_env('other').BAR)
2
>>> print(settings.from_env('other').FOO)
AttributeError: settings object has no attribute 'FOO'
```

Существующий объект настроек **settings** остается прежним.

```python
>>> print(settings.MESSAGE)
'This is in dev'
```

Вы можете назначать новые объекты настроек различным окружениям, например:

```python
development_settings = settings.from_env('development')
other_settings = settings.from_env('other')
```

И вы можете выбрать, будут ли переменные из разных окружений объединяться и переопределяться последовательно:

```python
all_settings = settings.from_env('development', keep=True).from_env('other', keep=True)

>>> print(all_settings.MESSAGE)
'This is in other env'
>>> print(all_settings.FOO)
1
>>> print(all_settings.BAR)
2
```

Переменные из `[development]` загружаются с сохранением предварительно загруженных значений, затем загружаются переменные из `[other]` с сохранением предварительно загруженных из `[development]` и переопределением их.

Также можно передать дополнительные переменные [конфигурации](konfiguraciya-dynaconf.md) в метод **from\_env**.

```python
new_settings = settings.from_env(
    'production', keep=True, SETTINGS_FILE_FOR_DYNACONF='another_file_path.yaml'
)
```

Затем **new\_settings** унаследует все переменные из существующей среды **env**, а также загрузит производственную среду `another_file_path.yaml`.

### setenv

Изменяет **in\_place** окружение существующего объекта.

```python
from dynaconf import settings

settings.setenv('other')
# теперь значения берутся из раздела [other] конфигурации
assert settings.MESSAGE == 'This is in other env'

settings.setenv()
# теперь рабочая среда вернулась к предыдущей
```

### using\_env

Использование контекстного менеджера

```python
from dynaconf import settings

with settings.using_env('other'):
    # теперь значения берутся из раздела [other] конфигурации
    assert settings.MESSAGE == 'This is in other env'

# существующие настройки возвращаются в нормальное состояние
# после области действия контекстного менеджера
assert settings.MESSAGE == 'This is in dev'
```

## Наполнение объектов

> Новое в версии 2.0.0

Вы можете использовать значения dynaconf для заполнения объектов (экземпляров) Python.

пример:

```python
class Obj:
   ...
```

затем вы можете сделать:

```python
# предположим, что это имеет DEBUG=True и VALUE=42.1
from dynaconf import settings
obj = Obj()

settings.populate_obj(obj)

assert obj.DEBUG is True
assert obj.VALUE == 42.1
```

Также вы можете указать только некоторые ключи:

<pre class="language-python"><code class="lang-python"># предположим, что это имеет DEBUG=True и VALUE=42.1
<strong>from dynaconf import settings
</strong>obj = Obj()

settings.populate_obj(obj, keys=['DEBUG'])

assert obj.DEBUG is True  # ok

assert obj.VALUE == 42.1  # AttributeError
</code></pre>

## Экспорт

Вы можете создать файл с текущими конфигурациями, вызвав `dynaconf list -o /path/to/file.ext`, подробнее см. в [CLI](cli-dynaconf.md).

Вы также можете сделать это программно с помощью:

```python
from dynaconf import loaders
from dynaconf import settings
from dynaconf.utils.boxing import DynaBox

# генерирует словарь со всеми ключами для среды разработки `development`
data = settings.as_dict(env='development')

# записывает в файл, формат которого определяется расширением
# он может быть .yaml, .toml, .ini, .json, .py
loaders.write(
    '/path/to/file.yaml', DynaBox(data).to_dict(), merge=False,
    env='development'
)
```

## Предварительная загрузка файлов

> Новое в версии 2.2.0

Полезно для приложений на основе плагинов.

```python
from dynaconf import Dynaconf

settings = Dynaconf(
  # <-- Загружается первым
  preload=["/path/*", "other/settings.toml"],
  # <-- Загружается вторым (основной файл)
  settings_file="/etc/foo/settings.py",
  # <-- Загружается в конце
  includes=["other.module.settings", "other/settings.yaml"]
)
```

## Тестирование

Для тестирования рекомендуется просто переключиться на среду тестирования и прочитать те же файлы конфигурации.

#### `settings.toml`

```toml
[default]
value = "On Default"

[testing]
value = "On Testing"
```

#### `program.py`

```python
from dynaconf import settings

print(settings.VALUE)
```

```bash
ENV_FOR_DYNACONF=testing python program.py
```

Затем ваш `program.py` напечатает красный цвет `"On Testing"` из среды `[testing]`.

### Pytest

Для **pytest** обычно создаются фикстуры для предоставления предварительно настроенного объекта настроек или для настройки параметров до того, как будут собраны все тесты.

Примеры доступны на [https://github.com/dynaconf/dynaconf/tree/master/tests\_functional/pytest\_example](https://github.com/dynaconf/dynaconf/tree/master/tests\_functional/pytest\_example).

С фикстурами **pytest** рекомендуется использовать **FORCE\_ENV\_FOR\_DYNACONF** вместо просто **ENV\_FOR\_DYNACONF**, поскольку он имеет приоритет.

#### Настройте Dynaconf с помощью Pytest

Определите свой **root\_path**

```python
import os

from dynaconf import Dynaconf

current_directory = os.path.dirname(os.path.realpath(__file__))

settings = Dynaconf(
    root_path=current_directory, # определение root_path
    envvar_prefix="DYNACONF",
    settings_files=["settings.toml", ".secrets.toml"],
)
```

#### Программа на Python

`settings.toml` со средой `[testing]`.

```toml
[default]
VALUE = "On Default"

[testing]
VALUE = "On Testing"
```

`app.py`, который считывает это значение из текущей среды.

```python
from dynaconf import settings


def return_a_value():
    return settings.VALUE
```

`tests/conftest.py` с фикстурами для принудительного запуска настроек, указывающих на среду `[testing]`.

```python
import pytest
from dynaconf import settings


@pytest.fixture(scope="session", autouse=True)
def set_test_settings():
    settings.configure(FORCE_ENV_FOR_DYNACONF="testing")
```

`tests/test_dynaconf.py`, чтобы убедиться, что загружена правильная среда.

```python
from app import return_a_value


def test_dynaconf_is_in_testing_env():
    assert return_a_value() == "On Testing"
```

#### Программа Flask

`settings.toml` со средой `[testing]`.

```toml
[default]
VALUE = "On Default"

[testing]
VALUE = "On Testing"
```

`src.py`, в котором есть фабрика приложений **Flask**.

```python
from flask import Flask
from dynaconf.contrib import FlaskDynaconf


def create_app(**config):
    app = Flask(__name__)
    FlaskDynaconf(app, **config)
    return app
```

`tests/conftest.py` с фикстурами для внедрения зависимостей приложения во все тесты и заставить это приложение указывать на среду конфигурации `[testing]`.

```python
import pytest
from src import create_app


@pytest.fixture(scope="session")
def app():
    app = create_app(FORCE_ENV_FOR_DYNACONF="testing")
    return app
```

te`sts/test_flask_dynaconf.py`, чтобы убедиться, что загружена правильная среда.

```python
def test_dynaconf_is_on_testing_env(app):
    assert app.config["VALUE"] == "On Testing"
    assert app.config.current_env == "testing"
```

## Имитация (mocking)

Но в модульных тестах часто используется имитация **mock** некоторых объектов, и в редких случаях вам может понадобиться имитировать настройки `dynaconf.settings` при запуске тестов.

```python
from dynaconf.utils import DynaconfDict
mocked_settings = DynaconfDict({'FOO': 'BAR'})
```

**DynaconfDict** — это объект, похожий на словарь, который можно заполнить из файла:

```python
from dynaconf.loaders import toml_loader
toml_loader.load(mocked_settings, filename='my_file.toml', env='testing')
```
