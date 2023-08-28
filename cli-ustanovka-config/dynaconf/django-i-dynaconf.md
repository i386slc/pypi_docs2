# Django и dynaconf

> Новое в версии 2.0.0

Расширение Dynaconf для Django работает путем исправления файла `settings.py` с помощью загрузочных хуков **dynaconf**. Изменения выполняются в одном файле, а затем во всем проекте. Каждый раз, когда вы вызываете `django.conf.settings`, вы получаете доступ к атрибутам и методам **dynaconf**.

Убедитесь, что dynaconf установлен в вашей среде `pip install dynaconf[yaml]`

## Инициализация расширения

Вы можете вручную добавить в конец файла `settings.py` вашего проекта django следующий код:

```python
# HERE STARTS DYNACONF EXTENSION LOAD (Keep at the very bottom of settings.py)
# Read more at https://www.dynaconf.com/django/
import dynaconf  # noqa
settings = dynaconf.DjangoDynaconf(__name__)  # noqa
# HERE ENDS DYNACONF EXTENSION LOAD (No more code below this line)
```

Или, опционально, вы можете в том же каталоге, где находится ваш файл `manage.py`, запустить:

```bash
export DJANGO_SETTINGS_MODULE=yourapp.settings
$ dynaconf init

# или передать расположение файла настроек

$ dynaconf init --django yourapp/settings.py
```

Dynaconf добавит код загрузки расширения в конец вашего файла `yourapp/settings.py` и создаст `settings.toml` и `.secrets.toml` в текущей папке (там же, где находится `manage.py`).

{% hint style="success" %}
**Совет**

Взгляните на [test\_functional/django\_example](https://github.com/dynaconf/dynaconf/tree/master/tests\_functional/django\_example).
{% endhint %}

## Использование переменных среды DJANGO\_

Затем `django.conf.settings` будет работать как экземпляр `dynaconf.settings`, а **DJANGO\_** будет глобальным префиксом для экспорта переменных среды.

Пример:

```bash
export DJANGO_DEBUG=true     # django.conf.settings.DEBUG
export DJANGO_INTVALUE=1     # django.conf.settings['INTVALUE']
export DJANGO_HELLO="Hello"  # django.conf.settings.get('HELLO')
```

{% hint style="success" %}
**Совет**

Если вы не хотите использовать **DJANGO\_** в качестве префикса для envvars, вы можете настроить его, передав новое имя, например: `dynaconf.DjangoDynaconf(__name__, ENVVAR_PREFIX_FOR_DYNACONF="FOO")`, и затем `export FOO_DEBUG=true`
{% endhint %}

Вы также можете установить вложенные значения словаря. Например, предположим, что у вас есть такая конфигурация:

```python
# settings.py

...
DATABASES = {
    'default': {
        'NAME': 'db',
        'ENGINE': 'module.foo.engine',
        'ARGS': {'timeout': 30}
    }
}
...
```

И теперь вы хотите изменить значения **ENGINE** на `other.module`, через переменные среды вы можете использовать формат `${ENVVAR_PREFIX}_${VARIABLE}__${NESTED_ITEM}__${NESTED_ITEM}`

Каждый `__` (dunder, он же двойное подчеркивание) обозначает доступ к вложенным элементам словаря.

Так:

```bash
export DYNACONF_DATABASES__default__ENGINE=other.module
```

приведет к

```python
DATABASES = {
    'default': {
        'NAME': 'db',
        'ENGINE': 'other.module',
        'ARGS': {'timeout': 30}
    }
}
```

{% hint style="warning" %}
Обратите внимание, что регистр важен для настроек Django, поэтому `DYNACONF_DATABASES__default__ENGINE` — это не то же самое, что `DYNACONF_DATABASES__DEFAULT__ENGINE`, вы должны использовать первый, который соответствует правильным настройкам Django.
{% endhint %}

Подробнее о [переменных среды](sliyanie-merging.md#vlozhennye-klyuchi-v-slovari-cherez-peremennye-okruzheniya.)

## Файлы настроек

Вы также можете иметь файлы настроек для вашего приложения Django.

В корневой каталог (тот же, где находится `manage.py`) поместите файлы `settings.{yaml, toml, ini, json, py}` и `.secrets.{yaml, toml, ini, json, py}`, а затем определите свою среду `[default]`, `[deveopment]` и `[production]`.

Для переключения рабочей среды можно использовать переменную **DJANGO\_ENV**, поэтому `DJANGO_ENV=development` для работы в режиме разработки или `DJANGO_ENV=production` для переключения в производственный режим.

Если вы не хотите вручную создавать файлы конфигурации, взгляните на [CLI](cli-dynaconf.md).

{% hint style="success" %}
**Совет**

`.yaml` — рекомендуемый формат для приложений Django, поскольку он позволяет легко писать сложные структуры данных. Тем не менее, смело выбирайте любой привычный вам формат.
{% endhint %}

{% hint style="info" %}
Чтобы использовать `$ dynaconf` CLI, необходимо определить переменную среды **DJANGO\_SETTINGS\_MODULE**.
{% endhint %}

## Индивидуальные настройки

### Загрузка настроек

Вы можете настроить способ загрузки настроек вашего проекта django.

Пример: вы хотите, чтобы ваши пользователи настроили файл настроек, определенный в `export PROJECTNAME_SETTINGS=/path/to/settings.toml`, и вы хотите, чтобы переменные среды загружались из **PROJECTNAME\_VARNAME**.

Для этого отредактируйте файл django `settings.py` и измените часть расширения dynaconf:

от:

```python
# HERE STARTS DYNACONF EXTENSION LOAD
...
settings = dynaconf.DjangoDynaconf(__name__)
# HERE ENDS DYNACONF EXTENSION LOAD
```

к такому виду:

```python
# HERE STARTS DYNACONF EXTENSION LOAD
...
settings = dynaconf.DjangoDynaconf(
    __name__,
    ENVVAR_PREFIX_FOR_DYNACONF='PROJECTNAME',
    ENV_SWITCHER_FOR_DYNACONF='PROJECTNAME_ENV',
    SETTINGS_FILE_FOR_DYNACONF='/etc/projectname/settings.toml',
    ENVVAR_FOR_DYNACONF='PROJECTNAME_SETTINGS',
    INCLUDES_FOR_DYNACONF=['/etc/projectname/plugins/*'],
)
# HERE ENDS DYNACONF EXTENSION LOAD
```

Переменные в среде можно установить/переопределить с помощью префикса **PROJECTNAME\_**, например: `export PROJECTNAME_DEBUG=true`.

Рабочую среду теперь можно переключать с помощью команды `export PROJECTNAME_ENV=production`, которая по умолчанию используется для разработки **development**.

Ваши настройки теперь считываются из `/etc/projectname/settings.toml` (dynaconf не будет выполнять поиск всех форматов настроек). Местоположение этих настроек можно изменить с помощью envvar, используя экспорт `PROJECTNAME_SETTINGS=/other/path/to/settings.py{yaml,toml,json,ini}`

Дополнительные настройки можно прочитать из `/etc/projectname/plugins/*`. Любой поддерживаемый файл из этой папки будет загружен.

Вы можете установить дополнительные параметры, посмотрите [конфигурацию](konfiguraciya-dynaconf.md).

### Используйте функции Django внутри пользовательских настроек

Если вам нужно использовать функции django в настройках, вы можете зарегистрировать собственные конвертеры с помощью утилиты **add\_converters**.

При определении их в `settings.py` есть некоторые функции django, которые нельзя импортировать непосредственно в область действия модуля. По этой причине вы можете добавить их в [хук](rasshirennoe-ispolzovanie-dynaconf.md#khuki), который выполняется после загрузки.

Например, если вам нужно использовать **reverse\_lazy**, вы можете сделать это:

```python
# myprj/settings.py

import dynaconf

def converters_setup():
    from django.urls import reverse_lazy  # noqa

    dynaconf.add_converter("reverse_lazy", reverse_lazy)

settings = dynaconf.DjangoDynaconf(__name__, post_hooks=converters_setup)

# HERE ENDS DYNACONF EXTENSION LOAD (No more code below this line)
```

И тогда будет работать следующий код:

```yaml
# settings.yaml

default:
    ADMIN_NAMESPACE: admin
    LOGIN_URL: "@reverse_lazy @format {this.ADMIN_NAMESPACE}:login"
```

{% hint style="info" %}
Некоторые распространенные конвертеры могут быть добавлены в Dynaconf в будущих выпусках. См. [#865](https://github.com/dynaconf/dynaconf/issues/865).

Информацию о **gettext** см. в [#648](https://github.com/dynaconf/dynaconf/issues/648).
{% endhint %}

## Чтение настроек автономных скриптов

Рекомендуемый способ создания автономных сценариев — создание команд управления `management commands` внутри приложений или плагинов Django.

В приведенных ниже примерах предполагается, что у вас установлена переменная среды **DJANGO\_SETTINGS\_MODULE**, либо экспортировав ее в свою среду, либо явно добавив ее в словарь `os.environ`.

{% hint style="success" %}
Если вам нужно, чтобы скрипт был доступен вне области вашего приложения Django, предпочитайте использовать `settings.DYNACONF.configure()` вместо обычного `settings.configure()`, предоставляемого Django. Последнее приведет к отключению dynaconf.

В конце концов, вам, вероятно, не нужно его вызывать, поскольку у вас экспортирован **DJANGO\_SETTINGS\_MODULE**.
{% endhint %}

### Общий случай

```python
# /etc/my_script.py

from django.conf import settings
print(settings.DATABASES)
```

### Явное добавление модуля настройки

```python
# /etc/my_script.py

import os
os.environ['DJANGO_SETTINGS_MODULE'] = 'foo.settings'

from django.conf import settings
print(settings.DATABASES)
```

### Когда вам нужна настройка

Вызов `DYNACONF.configure()` необходим, когда вы хотите получить доступ к специальным методам dynaconf, таким как **using\_env**, **get**, **get\_fresh** и т. д.

```python
# /etc/my_script.py

from django.conf import settings
settings.DYNACONF.configure()
print(settings.get('DATABASES'))
```

### Импорт настроек напрямую

Это рекомендуется для вышеуказанного случая.

```python
# /etc/my_script.py

from foo.settings import settings
print(settings.get('DATABASES'))
```

### Импорт настроек через importlib

```python
# /etc/my_script.py

import os
import importlib
settings = importlib.import_module(os.environ['DJANGO_SETTINGS_MODULE'])
print(settings.get('DATABASES'))
```

## Тестирование на Django

Тестирование Django должно работать «из коробки»!

## Фиктивные envvars с помощью Django

Но в некоторых случаях, когда вы имитируете что-то и вам нужно добавить переменные среды в `os.environ` по требованию для тестовых случаев, может потребоваться перезагрузка **reload** dynaconf.

Для этого напишите часть настройки вашего тестового примера:

```python
import os
import importlib
from myapp import settings # ПРИМЕЧАНИЕ. Здесь используется модуль вашего приложения,
                           # а не django.conf.

class TestCase(...):
    def setUp(self):
        os.environ['DJANGO_FOO'] = 'BAR'  # dynaconf должен прочитать его
                                          # и установить `settings.FOO`
        importlib.reload(settings)

    def test_foo(self):
        self.assertEqual(settings.FOO, 'BAR')
```

## Использование pytest и django

Установите `pip install pytest-django`

Добавьте в свой `conftest.py`

`project/tests/conftest.py`

```python
import pytest

@pytest.fixture(scope="session", autouse=True)
def set_test_settings():
    # https://github.com/dynaconf/dynaconf/issues/491#issuecomment-745391955
    from django.conf import settings
    settings.setenv('testing')  # заставить окружающую среду быть такой, какой вы хотите
```

## Явный режим

Некоторые пользователи предпочитают явно загружать каждую переменную настройки в файл `settings.py`, а затем позволять django управлять ею обычным способом. Это возможно, но имейте в виду, что это предотвратит использование методов dynaconf, таких как **using\_env**, **get**.

Dynaconf будет доступен только в области `settings.py`. В остальной части вашего приложения настройками Django управляет обычным образом.

```python
# settings.py

import sys
from dynaconf import LazySettings

settings = LazySettings(**YOUR_OPTIONS_HERE)

DEBUG = settings.get('DEBUG', False)
DATABASES = settings.get('DATABASES', {
    'default': {
        'ENGINE': '...',
        'NAME': '...
    }
})
...

# В конце вашего settings.py
settings.populate_obj(sys.modules[__name__], ignore=locals())
```

Вы по-прежнему можете изменить окружение с помощью команды `export DJANGO_ENV=production,` а также экспортировать переменные, например, `export DJANGO_DEBUG=true`.

{% hint style="info" %}
Начиная с версии `2.1.1`, аргумент **ignore** указывает Dynaconf не переопределять переменные, которые уже существуют в текущем файле настроек; удалите его, если вы хотите, чтобы все существующие локальные переменные были перезаписаны dynaconf.
{% endhint %}

## Известные предостережения

* Если `settings.configure()` вызвать напрямую, Dynaconf отключается; используйте `settings.DYNACONF.configure()`.

## Примечание об устаревании

В старых версиях dynaconf решением было добавить `dynaconf.contrib.django_dynaconf` в **INSTALLED\_APPS** в качестве первого элемента. Это все еще работает, но имеет некоторые ограничения, поэтому больше не рекомендуется.
