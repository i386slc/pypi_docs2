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
