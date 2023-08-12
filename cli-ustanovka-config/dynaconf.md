# dynaconf

<figure><img src="../.gitbook/assets/1111.png" alt=""><figcaption></figcaption></figure>

## Характеристики

* Вдохновлено [руководством по применению 12-факторов](https://12factor.net/config)
* **Управление настройками** (значения по умолчанию, проверка, парсинг, шаблоны)
* Защита **конфиденциальной информации** (пароли/токены)
* Несколько **форматов файлов** (`toml|yaml|json|ini|py`), а также настраиваемые загрузчики.
* Полная поддержка **переменных среды** для переопределения существующих настроек (включая поддержку **dotenv**).
* Дополнительная многоуровневая система для **нескольких сред** `[default, development, testing, production]` (также называемая мульти-профилями)
* Встроенная поддержка **Hashicorp Vault** и **Redis** в качестве хранилища настроек и секретов.
* Встроенные расширения для веб-фреймворков **Django** и **Flask**.
* **CLI** для общих операций, таких как **init**, **list**, **write**, **validate**, **export**.

## Установка

### Установка из [PyPI](https://pypi.org/project/dynaconf)

```python
pip install dynaconf
```

### Инициализация Dynaconf в вашем проекте

### Использование только Python

В корневом каталоге вашего проекта запустите команду `dynaconf init`.

```bash
cd path/to/your/project/
dynaconf init -f toml
```

Вывод команды должен быть:

```
⚙️  Configuring your Dynaconf environment
------------------------------------------
🐍 The file `config.py` was generated.

🎛️  settings.toml created to hold your settings.

🔑 .secrets.toml created to hold your secrets.

🙈 the .secrets.* is also included in `.gitignore`
beware to not push your secrets to a public repo.

🎉 Dynaconf is configured! read more on https://dynaconf.com
```

{% hint style="info" %}
Вы можете выбрать `toml|yaml|json|ini|py` в `dynaconf init -f <filename>`, **toml** используется по умолчанию, а также является наиболее рекомендуемым форматом для конфигурации.
{% endhint %}

Команда `dynaconf init` создает следующие файлы

```
.
├── config.py       # Где вы импортируете свой объект настроек (обязательно)
├── .secrets.toml   # Конфиденциальные данные - пароли и токены (необязательно)
└── settings.toml   # Настройки приложения (необязательно)
```

#### your\_program.py

В своем собственном коде вы импортируете и используете объект **settings**, импортированный из вашего файла **config.py**.

```python
from config import settings

assert settings.key == "value"
assert settings.number == 789
assert settings.a_dict.nested.other_level == "nested value"
assert settings['a_boolean'] is False
assert settings.get("DONTEXIST", default=1) == 1
```

#### config.py

В этом файле инициализируется и настраивается новый экземпляр объекта настроек **Dynaconf**.

```python
from dynaconf import Dynaconf

settings = Dynaconf(
    settings_files=['settings.toml', '.secrets.toml'],
)
```

Дополнительные параметры описаны в <mark style="color:purple;">Dynaconf Configuration</mark>.

#### settings.toml

**Опционально** можно сохранить настройки в файле (или в нескольких файлах)

```toml
key = "value"
a_boolean = false
number = 1234
a_float = 56.8
a_list = [1, 2, 3, 4]
a_dict = {hello="world"}

[a_dict.nested]
other_level = "nested value"
```

Подробнее в файлах настроек (<mark style="color:purple;">Settings files</mark>)

#### .secrets.toml

**Опционально** можно сохранить конфиденциальные данные в локальном файле `.secrets.toml`.

```toml
password = "s3cr3t"
token = "dfgrfg5d4g56ds4gsdf5g74984we5345-"
message = "This file doesn't go to your pub repo"
```

{% hint style="warning" %}
Команда `dynaconf init` помещает `.secrets.*` в ваш `.gitignore`, чтобы избежать его раскрытия в общедоступных репозиториях, но вы несете ответственность за его безопасность в вашей локальной среде, также рекомендуется для производственных сред использовать встроенную поддержку для сервис **Hashicorp Vault** для паролей и токенов.
{% endhint %}

```gitignore
# Секреты не попадают в публичные репозитории
.secrets.*
```

подробнее о <mark style="color:purple;">секретах</mark>

#### env vars

**При необходимости** переопределите переменные среды с префиксом. (файлы `.env` также поддерживаются)

```bash
export DYNACONF_NUMBER=789
export DYNACONF_FOO=false
export DYNACONF_DATA__CAN__BE__NESTED=value
export DYNACONF_FORMATTED_KEY="@format {this.FOO}/BAR"
export DYNACONF_TEMPLATED_KEY="@jinja {{ env['HOME'] | abspath }}"
```

{% hint style="info" %}
Вы можете создавать файлы самостоятельно вместо использования команды **dynaconf init**, и она дает любое имя, которое вы хотите, вместо **config.py** по умолчанию (файл должен быть в вашем импортируемом пути python)
{% endhint %}

### Использование Flask

#### app.py

В вашем проекте **Flask** импортируйте расширение **FlaskDynaconf** и инициализируйте его как расширение **Flask**.

```python
from flask import Flask
from dynaconf import FlaskDynaconf

app = Flask(__name__)

FlaskDynaconf(app, settings_files=["settings.toml"])
```

В вашем приложении **Flask** вы можете получить доступ ко всем переменным настроек непосредственно из `app.config`, который теперь заменен экземпляром настроек **dynaconf**.

```python
@app.route("/a_view)
def a_view():
    app.config.NAME == "BRUNO"
    app.config['DEBUG'] is True
    app.config.SQLALCHEMY_DB_URI == "sqlite://data.db"
```

#### settings.toml

**При необходимости** сохраните настройки в файле, используя многоуровневую среду.

```toml
[default]
key = "value"
a_boolean = false
number = 1234

[development]
key = "development value"
SQLALCHEMY_DB_URI = "sqlite://data.db"

[production]
key = "production value"
SQLALCHEMY_DB_URI = "postgresql://..."
```

{% hint style="info" %}
В **Flask** файлы настроек по умолчанию располагаются слоями в нескольких средах, вы можете отключить их, передав `environments=False` в расширение **FlaskDynaconf**.
{% endhint %}

Подробнее в файлах настроек (<mark style="color:purple;">Settings files</mark>)

#### .secrets.toml

**Опционально** можно сохранить конфиденциальные данные в локальном файле `.secrets.toml`.

```toml
[development]
password = "s3cr3t"
token = "dfgrfg5d4g56ds4gsdf5g74984we5345-"
message = "This file doesn't go to your pub repo"
```

{% hint style="warning" %}
поместите `.secrets.*` в свой `.gitignore`, чтобы избежать его раскрытия в общедоступных репозиториях, но вы несете ответственность за его безопасность в вашей локальной среде, также рекомендуется использовать встроенную поддержку службы **Hashicorp Vault** для пароль и токены.
{% endhint %}

```gitignore
# Секреты не попадают в публичные репозитории
.secrets.*
```

подробнее о <mark style="color:purple;">секретах</mark>

#### env vars

**При необходимости** переопределите любой параметр, используя переменные среды с префиксом **FLASK\_**. (файлы `.env` также поддерживаются)

```bash
export FLASK_ENV=production
export FLASK_NUMBER=789
export FLASK_FOO=false
```

Dynaconf также может **загружать ваши расширения Flask**, более подробную информацию см. в <mark style="color:purple;">расширении Flask</mark>.

### Использование Django

Убедитесь, что вы экспортировали **DJANGO\_SETTINGS\_MODULE**:

```bash
export DJANGO_SETTINGS_MODULE=yourproject.settings
```

В той же папке, где находится ваш `manage.py`, запустите команду `dynaconf init`.

```bash
dynaconf init -f yaml
```

Затем следуйте инструкциям на терминале.

```
Django app detected
⚙️  Configuring your Dynaconf environment
------------------------------------------
🎛️  settings.yaml created to hold your settings.

🔑 .secrets.yaml created to hold your secrets.

🙈 the .secrets.yaml is also included in `.gitignore`
beware of not pushing your secrets to a public repo
or use dynaconf builtin support for Vault Servers.

⁉  path/to/yourproject/settings.py is found do you want to add dynaconf? [y/N]:
```

Ответ **y**

```
🎠  Now your Django settings are managed by Dynaconf
🎉  Dynaconf is configured! read more at https://dynaconf.com
```

{% hint style="info" %}
В Django рекомендуемым форматом файла является **yaml**, потому что он может легче хранить сложные структуры данных, однако вы можете использовать **toml**, **json**, **ini** или даже сохранить свои настройки в формате `.py`.
{% endhint %}

#### appname/views.py

В ваших представлениях, моделях и других местах Django теперь вы можете использовать `django.conf.settings` в обычном режиме, потому что он заменен объектом настроек **Dynaconf**.

```python
from django.conf import settings

def index(request):
    assert settings.DEBUG is True
    assert settings.NAME == "Bruno"
    assert settings.DATABASES.default.name == "db"
    assert settings.get("NONEXISTENT", 2) == 2
```

#### settings.yaml

**Опционально** сохраните настройки в файле, используя многоуровневую среду. Этот файл должен находиться в папке, где находится ваш `manage.py`.

```yaml
default:
    ALLOWED_HOSTS:
        - '*'
    INSTALLED_APPS:
        - django.contrib.admin
        - django.contrib.auth
        - django.contrib.contenttypes
        - django.contrib.sessions
        - django.contrib.messages
        - django.contrib.staticfiles
production:
    ALLOWED_HOSTS:
        - 'server.prod.com'
```

{% hint style="info" %}
Файлы настроек Django по умолчанию располагаются слоями в нескольких средах, вы можете отключить их, передав `environments=False` в расширение **DjangoDynaconf**.
{% endhint %}

Подробнее в файлах настроек (<mark style="color:purple;">Settings files</mark>)

#### .secrets.toml

**Опционально** можно сохранить конфиденциальные данные в локальном файле `.secrets.toml`.

```yaml
development:
    SECRET_KEY: 43grng9398534nfkjer
production:
    SECRET_KEY: vfkjndkjg098gdf90gudfsg
```

{% hint style="warning" %}
поместите `.secrets.*` в свой `.gitignore`, чтобы избежать его раскрытия в общедоступных репозиториях, но вы несете ответственность за его безопасность в вашей локальной среде, также рекомендуется использовать встроенную поддержку службы **Hashicorp Vault** для пароль и токены.
{% endhint %}

```gitignore
# Секреты не попадают в публичные репозитории
.secrets.*
```

подробнее о <mark style="color:purple;">секретах</mark>

#### env vars

**Опционально** переопределите любой параметр, используя переменные окружения с префиксом **DJANGO\_**. (файлы `.env` также поддерживаются)

```bash
export DJANGO_ENV=production
export DJANGO_NUMBER=789
export DJANGO_FOO=false
export DJANGO_ALLOWED_HOSTS="['*']"
export DJANGO_DEBUG=false
export DJANGO_DATABASES__default__NAME=othername
```

#### projectname/settings.py

После инициализации **dynaconf** включает следующее в нижней части вашего существующего `settings.py`, и вам нужно сохранить эти строки там.

```python
# HERE STARTS DYNACONF EXTENSION LOAD
import dynaconf  # noqa
settings = dynaconf.DjangoDynaconf(__name__)  # noqa
# HERE ENDS DYNACONF EXTENSION LOAD (No more code below this line)
```

Подробнее в <mark style="color:purple;">расширении Django</mark>

{% hint style="success" %}
**Совет**

В интерфейсе командной строки **dynaconf** есть более полезные команды, такие как **list** | **export**, **init**, **write** и **validate**. Подробнее о <mark style="color:purple;">CLI</mark>
{% endhint %}

## Определение ваших переменных настроек
