# Файлы настроек dynaconf

**Опционально** вы можете хранить настройки в файлах, dynaconf поддерживает несколько форматов файлов, вам рекомендуется выбрать один формат, но вы также можете использовать смешанные форматы настроек в своем приложении.

{% hint style="success" %}
**Совет**

Вы не обязаны использовать файлы настроек, если они не указаны, dynaconf может загрузить ваши данные только из [переменных окружения](peremennye-okruzheniya-env-vars.md).
{% endhint %}

## Поддерживаемые форматы

* `.toml` — формат файла по умолчанию и **рекомендуемый**.
* `.yaml|.yml` — рекомендуется для приложений Django. (см. [предостережения yaml](faily-nastroek-dynaconf.md#predosterezheniya-yaml))
* `.json` — полезен для повторного использования существующих или экспортированных настроек.
* `.ini` — полезно для повторного использования устаревших настроек.
* `.py` — **не рекомендуется**, но поддерживается для обратной совместимости.
* `.env` — полезно для автоматизации загрузки переменных окружения.

{% hint style="info" %}
Создайте свои настройки в нужном формате и укажите их в аргументе **settings\_files** в вашем экземпляре **dynaconf** или передайте их в `-f <format>` при использовании команды **dynaconf init**.
{% endhint %}

{% hint style="success" %}
**Совет**

Не можете найти нужный формат файла для ваших настроек? Вы можете создать свой собственный загрузчик и читать любой источник данных. Подробнее в [расширении dynaconf](rasshirennoe-ispolzovanie-dynaconf.md)
{% endhint %}

{% hint style="warning" %}
Чтобы использовать формат файла `.ini` или `.properties`, вам необходимо установить дополнительную зависимость `pip install configobj` или же `pip install dynaconf[ini]`
{% endhint %}

## Чтение настроек из файлов

В файлах по умолчанию **dynaconf** загружает все существующие ключи и разделы как настройки первого уровня.

#### settings.toml

```toml
name = "Bruno"
```

#### settings.yaml

```yaml
name: Bruno
```

#### settings.json

```json
{"name": "Bruno"}
```

#### settings.ini

```ini
name = 'Bruno'
```

#### settings.py

```python
NAME = "Bruno"
```

{% hint style="warning" %}
в файлах `.py` **dynaconf** читает только переменные, написанные **ЗАГЛАВНЫМИ БУКВАМИ**.
{% endhint %}

Затем в коде вашего приложения:

```python
settings.name == "Bruno"
```

{% hint style="info" %}
Кодировка по умолчанию при загрузке файлов настроек — `utf-8`, и ее можно настроить с помощью параметра кодировки **encoding**.
{% endhint %}

## Загрузка файлов настроек

Dynaconf начнет искать каждый файл, определенный в **settings\_files**, в папке, где находится файл python вашей точки входа (например, `app.py`). Затем он просматривает каждого родителя вплоть до корня системы. Для каждой посещенной папки он также попытается просмотреть папку `/config`.

* Если вы определите [root\_path](konfiguraciya-dynaconf.md#settings\_file-or-settings\_files), вместо этого он будет искать оттуда. Имейте в виду, что **root\_path** относится к **cwd**, откуда был вызван интерпретатор Python.
* Абсолютные пути распознаются, и dynaconf попытается загрузить их напрямую.
* Для каждого файла, указанного в **settings\_files**, dynaconf также попытается загрузить необязательное расширение `name.local.extension`. Например, `settings_file="settings.toml"` также будет искать `settings.local.toml`.
* Glob принимаются.

Определите его в своем экземпляре настроек или экспортируйте соответствующие envvars.

```python
# по умолчанию
settings = Dynaconf(settings_files=["settings.toml", "*.yaml"])

# используя root_path
settings = Dynaconf(
    root_path="my/project/root"
    settings_files=["settings.toml", "*.yaml"],
)
```

```bash
export ROOT_PATH_FOR_DYNACONF='my/project/root'
export SETTINGS_FILES_FOR_DYNACONF='["settings.toml", "*.yaml"]'
```

{% hint style="info" %}
Чтобы использовать `python -m module`, где модуль использует dynaconf, вам нужно будет указать путь к вашему `settings.toml`, например, так: `settings_file="module/config/settings.toml"`.
{% endhint %}

## includes и preload

Если вам нужно, вы можете указать файлы для загрузки до или после **settings\_files**, используя опции [preload](konfiguraciya-dynaconf.md#preload) и [includes](konfiguraciya-dynaconf.md#includes). Их стратегия загрузки более строгая и будет использовать **root\_path** в качестве базового пути для предоставленных относительных путей. Если **root\_path** не определен, **includes** также попытается использовать последний найденный каталог настроек в качестве базового пути.

Их можно определить в экземпляре Dynaconf или в файле:

```python
# в экземпляре Dynaconf
settings = Dynaconf(
    includes=["path/to/file.toml", "or/a/glob/*.yaml"],
    preload=["path/to/file.toml", "or/a/glob/*.yaml"])
```

или

```toml
# в файле toml
dynaconf_include = ["path/to/file.toml"]
key = value
anotherkey = value
```

## Многоуровневые среды для файлов

Также можно заставить dynaconf читать файлы, разделенные многоуровневыми средами, чтобы каждая секция или ключ первого уровня загружались как отдельная среда.

#### config.py

```python
settings = Dynaconf(environments=True)
```

{% hint style="info" %}
Вы можете определить пользовательскую среду, используя желаемое имя `[defualt]` и `[global]` — единственные среды, которые являются особыми. Вы можете, например, назвать это `[testing]` или `[anything]`.
{% endhint %}

#### settings.toml

```toml
[default]
name = ""
[development]
name = "developer"
[production]
name = "admin"
```

#### settings.yaml

```yaml
default:
    name: ''
development:
    name: developer
production:
    name: admin
```

#### settings.json

```json
{
    "default": {
        "name": ""
    },
    "development": {
        "name": "developer"
    },
    "production": {
        "name": "admin"
    }
}
```

#### settings.ini

```ini
[default]
name = ""
[development]
name = "developer"
[production]
name = "admin"
```

#### program.py

Затем в вашей программе вы можете использовать переменные среды для переключения сред.

```bash
export ENV_FOR_DYNACONF=development
```

```python
settings.name == "developer"
```

```bash
export ENV_FOR_DYNACONF=production
```

```python
settings.name == "admin"
```

{% hint style="warning" %}
В расширениях **Flask** и **Django** поведение по умолчанию уже является многоуровневой средой. Также для переключения среды вы используете `export FLASK_ENV=production` или `export DJANGO_ENV=production` соответственно.
{% endhint %}

{% hint style="success" %}
**Совет**

Также можно переключать среды, программно передавая `env="development"` классу **Dynaconf** при создании экземпляра.
{% endhint %}

### Предостережения YAML

#### Тип None

Парсер Yaml, используемый dynaconf (`ruamel.yaml`), читает неопределенные значения как `None`, поэтому

```yaml
key:
key: ~
key: null
```

Все эти 3 ключа будут проанализированы как объект `None` Python.

При использовании валидатора для установки значения по умолчанию для этих значений вы можете использовать одно из:

```python
Validator("key", default="thing", apply_default_on_none=True)
```

Таким образом, dynaconf будет учитывать значение по умолчанию, даже если для yaml установлено значение `None`.

или в yaml вы можете установить значение **@empty**

```yaml
key: "@empty"
```

> **НОВОЕ** в версии 3.1.9
