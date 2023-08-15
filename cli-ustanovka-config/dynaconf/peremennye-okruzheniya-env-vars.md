# Переменные окружения (env vars)

Dynaconf _**отдает приоритет переменным среды над файлами**_, что является лучшей рекомендацией для сохранения ваших настроек.

Согласно [12factorapp](https://12factor.net/), рекомендуется сохранять свои конфигурации в зависимости от среды.

В дополнение к этому Dynaconf предлагает несколько подходов, которым вы, **возможно**, захотите следовать:

* Добавьте список валидаторов в `validators=[Validator()]` в вашем экземпляре настроек dynaconf и укажите [значения по умолчанию и ограничения](validaciya-v-dynaconf.md).
* Включите **load\_dotenv** и предоставьте файл `.env.example` или `.env` с переменными по умолчанию.
* Запишите свои переменные в [файл настроек](faily-nastroek-dynaconf.md) в формате по вашему выбору.
* Загрузите свои настройки из [хранилища vault или Redis](sekretnaya-informaciya-secrets.md)

## Переменные окружения

Вы можете переопределить любой ключ настройки, экспортировав переменную среды с префиксом **DYNACONF\_** (или с помощью [пользовательского префикса](konfiguraciya-dynaconf.md#envvar\_prefix)).

{% hint style="warning" %}
Dynaconf будет искать только **envvar** с префиксом **ВЕРХНЕГО РЕГИСТРА**. Это означает, что **envvar**, экспортированный как **dynaconf\_value** или **myprefix\_value**, не будет загружен, в то время как **DYNACONF\_VALUE** и **MYPREFIX\_VALUE** (при правильной установке) будут загружены.
{% endhint %}

### Пример

Обратите внимание, что при загрузке переменных среды dynaconf будет анализировать значение, используя язык **toml**, поэтому попытается автоматически преобразовать его в правильный тип данных.

```bash
export DYNACONF_NAME=Bruno                                   # str: "Bruno"
export DYNACONF_NUM=42                                       # int: 42
export DYNACONF_AMOUNT=65.6                                  # float: 65.6
export DYNACONF_THING_ENABLED=false                          # bool: False
export DYNACONF_COLORS="['red', 'gren', 'blue']"             # list: ['red', 'gren', 'blue']
export DYNACONF_PERSON="{name='Bruno', email='foo@bar.com'}" # dict: {"name": "Bruno", ...}
export DYNACONF_STRING_NUM="'76'"                            # str: "76"
export DYNACONF_PERSON__IS_ADMIN=true                        # bool: True (nested)
```

С помощью вышеизложенного теперь можно прочитать настройки в вашем `program.py`, используя:

```python
from dynaconf import Dynaconf

settings = Dynaconf()

assert settings.NAME == "Bruno"
assert settings.num == 42
assert settings.amount == 65.6
assert settings['thing_enabled'] is False
assert 'red' in settings.get('colors')
assert settings.person.email == "foo@bar.com"
assert settings.person.is_admin is True
assert settings.STRING_NUM == "76"
```

{% hint style="success" %}
**Совет**

Dynaconf имеет несколько допустимых способов доступа к ключам настроек, поэтому он совместим с вашим существующим решением для настройки. Вы можете получить доступ, используя `.notation`, индексирование `[item]`, метод `.get`, а также позволяет `.notation.nested` для структур данных, таких как словари. **Также переменный доступ нечувствителен к регистру для ключа первого уровня**.
{% endhint %}

{% hint style="warning" %}
При экспорте структур данных, таких как **dict** и **list**, вы должны использовать один из:

```bash
export DYNACONF_TOML_DICT={key="value"}
export DYNACONF_TOML_LIST=["item"]
export DYNACONF_JSON_DICT='@json {"key": "value"}'
export DYNACONF_JSON_LIST='@json ["item"]'
```

Эти два способа являются единственными способами загрузки dynaconf **dict** и **list** из **envvars**.
{% endhint %}

## Пользовательский префикс

Вам разрешено настраивать префикс для ваших env vars. Допустим, ваше приложение называется `"PEANUT"`

```bash
export PEANUT_USER=admin
export PEANUT_PASSWD=1234
export PEANUT_DB={name='foo', port=2000}
export PEANUT_DB__SCHEME=main
```

Теперь для вашего экземпляра настроек вы можете настроить префикс:

```python
from dynaconf import Dynaconf

settings = Dynaconf(envvar_prefix="PEANUT")


assert settings.USER == "admin"
assert settings.PASSWD == 1234
assert settings.db.name == "foo"
assert settings.db.port == 2000
assert settings.db.scheme == 'main'
```

Если вы не хотите использовать какой-либо префикс (загружать переменные без префикса), правильный способ установить его следующим образом:

```python
from dynaconf import Dynaconf

settings = Dynaconf(envvar_prefix=False)
```

## Don env

Когда вам нужно установить несколько переменных среды, полезно автоматизировать определение, а Dynaconf поставляется с поддержкой **Python-Dotenv**.

Вы можете поместить в корень вашего приложения файл с именем `.env`

```bash
PREFIX_USER=admin
```

Затем передайте `load_dotenv=True` в свои настройки.

```python
from dynaconf import Dynaconf

settings = Dynaconf(load_dotenv=True)

assert settings.USER == "admin"
```

## Приведение типов и ленивые значения

В дополнение к синтаксическому анализатору **toml** вы также можете принудительно преобразовать ваши переменные в желаемый тип данных.

```bash
export PREFIX_NONE='@none None'                        # None
export PREFIX_NONE='@int 16'                           # int: 16
export PREFIX_NONE='@bool off'                         # bool: False
export PREFIX_ARRAY='@json [42, "Oi", {"foo": "bar"}]' # Неоднородный список list
```

### Ленивые значения

Dynaconf поддерживает 2 типа ленивых значений - **format** и **jinja**, который позволяет заменять шаблоны.

```bash
export PREFIX_PATH='@format {env[HOME]}/.config/{this.DB_NAME}'
export PREFIX_PATH='@jinja {{env.HOME}}/.config/{{this.DB_NAME}} | abspath'
```

### Добавление пользовательского токена трансляции

Если вы хотите добавить собственный токен приведения типов, вы можете сделать это, добавив конвертер. Например, если мы хотим привести строки к объекту `pathlib.Path`, мы можем добавить в наш код Python:

```python
# app.py
from pathlib import Path
from dynaconf import add_converter

add_converter("path", Path)
```

В файле настроек теперь мы можем использовать токен приведения **@path**. Как и в случае с другими токенами приведения типов, вы также можете комбинировать их:

```toml
# settings.toml
my_path = "@path /home/foo/example.txt"
parent = "@path @format {env[HOME]}/parent"
child = "@path @format {this.parent}/child"
```

## Фильтрация переменных среды

Все переменные среды (естественно, с учетом префиксных правил) будут использованы и подтянуты в пространство настроек.

Вы можете изменить это поведение, включив фильтрацию неизвестных переменных среды:

```python
from dynaconf import Dynaconf

settings = Dynaconf(ignore_unknown_envvars=True)
```

В этом случае из среды будут загружены только ранее определенные переменные (например, указанные в `settings_file`/`settings_files`, `preload` или `includes`). Таким образом, ваше пространство настроек может избежать загрязнения или потенциальной загрузки конфиденциальной информации.
