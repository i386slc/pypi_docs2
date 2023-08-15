# Конфигурация dynaconf

При создании экземпляра объекта настроек Dynaconf **settings** есть несколько настраиваемых параметров.

## Параметры экземпляра

Конфигурации могут быть явно переданы вашему экземпляру.

```python
from dynaconf import Dynaconf

settings = Dynaconf(
    envvar_prefix="MYPROGRAM",
    settings_files=["settings.toml", ".secrets.toml"],
    environments=True,
    load_dotenv=True,
    env_switcher="MYPROGRAM_ENV",
    **more_options
)
```

## Параметры окружения

И также могут быть экспортированы как переменные среды, следуя правилу: все они должны быть **ЗАГЛАВНЫМИ** и иметь суффикс **\_FOR\_DYNACONF**.

```bash
export ENVVAR_PREFIX_FOR_DYNACONF=MYPROGRAM
export SETTINGS_FILES_FOR_DYNACONF="['settings.toml',  ...]"
export ENVIRONMENTS_FOR_DYNACONF=true
export LOAD_DOTENV_FOR_DYNACONF=true
export ENV_SWITCHER_FOR_DYNACONF=MYPROGRAM_ENV
```

## Доступные настройки

{% hint style="info" %}
При экспорте в виде переменных окружения убедитесь, что вы используете **ЗАГЛАВНЫЕ** буквы и добавляете суффикс **\_FOR\_DYNACONF**. Пример: `export AUTO_CAST_FOR_DYNACONF=true`
{% endhint %}

{% hint style="success" %}
**Совет**

Переменные окружения преобразуются с помощью синтаксического анализатора **TOML**, поэтому `=FOO` — это **str**, `=42` — **int**, `=42.1` — **float**, `=[1, 2]` — **list**, `={foo="bar"}` — **dict** и, наконец, `=true` является логическим значением **boolean**. (если принудительно заключить строковое значение в кавычки: `="'42'"` это будет **str**, `'42'`)
{% endhint %}

### **apply\_default\_on\_none**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `DOTTED_LOOKUP_FOR_DYNACONF`

YAML считывает пустые значения как `None` в этом случае, если вы хотите, чтобы значения по умолчанию **Validator** применялись к значениям `None`, вы должны установить для **apply\_default\_on\_none** значение `True` в классе **Dynaconf** или специально для экземпляра **Validator**.

### **auto\_cast**

* Тип - `bool`
* По умолчанию - `True`
* env-var - `AUTO_CAST_FOR_DYNACONF`

Автоприведение — это возможность использования специальных токенов для приведения типов переменных настроек. Например: `@format {env[HOME]}`, `@int 32`, `@json {...}`

### **commentjson\_enabled**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `COMMENTJSON_ENABLED_FOR_DYNACONF`

Включает комментарии в файлах настроек JSON, но **commentjson** должен быть установлен, то есть: `pip install commentjson`

### **core\_loaders**

* Тип - `list`
* По умолчанию - `[‘YAML’, ‘TOML’, ‘INI’, ‘JSON’, ‘PY’]`
* env-var - `CORE_LOADERS_FOR_DYNACONF`

Список включенных загрузчиков, которые dynaconf будет использовать для загрузки файлов настроек. Если ваше приложение использует только YAML, вы можете, например, изменить его на `['YAML']`, чтобы dynaconf прекратил попытки загрузки toml и других форматов.

### **default\_env**

* Тип - `str`
* По умолчанию - `"default"`

Когда `environments=True`, это определяет имя среды, которое будет содержать значения по умолчанию/резервные значения.

Например:

```toml
# settings.toml

[my_default]
test_value = "test value from default"
other_value = "other value from default"

[development]
# нет test_value здесь
other_value = "other value from dev"
```

```python
>>> settings = Dynaconf(
...     settings_file="settings.toml",
...     environments=True,
...     default_env="my_default",
...     env="development", # это активная среда по умолчанию
... )

>>> settings.other_value
"other value from dev"

>>> settings.test_value
"test value from default" # запасной вариант `my_default`
```

{% hint style="info" %}
**default\_env** не является средой, которая будет активна при запуске. Для этого см. [env](konfiguraciya-dynaconf.md#env).
{% endhint %}

### **dotenv\_override**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `DOTENV_OVERRIDE_FOR_DYNACONF`

Когда **load\_dotenv** включен, это определяет, будут ли переменные в `.env` переопределять значения, экспортированные как **env vars**.

### **dotenv\_path**

* Тип - `str`
* По умолчанию - `"."` (или то же, что и **project\_root**)
* env var - `DOTENV_PATH_FOR_DYNACONF`

Устанавливает расположение файла `.env` либо на полный путь, либо на содержащий его каталог.

### **dotenv\_verbose**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `DOTENV_VERBOSE_FOR_DYNACONF`

Контролирует многословие загрузчика **dotenv**.

### **dotted\_lookup**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `DOTTED_LOOKUP_FOR_DYNACONF`

По умолчанию dynaconf читает `.` в качестве разделителя при установке и чтении значений. Эту функцию можно отключить, установив `dotted_lookup=False`

{% hint style="success" %}
**Совет**

Это также можно установить для каждого файла с помощью `dynaconf_dotted_lookup: false` на верхнем уровне файла и работает для установки значений. Для чтения вы можете передать `settings.get("key.other", dotted_lookup=False)`
{% endhint %}

### **encoding**

* Тип - `str`
* По умолчанию - `"utf-8"`
* env-var - `ENCODING_FOR_DYNACONF`

Какую кодировку использовать при загрузке файлов настроек.

### **environments**

* Тип - `bool`
* По умолчанию - `False`
* env-var - `ENVIRONMENTS_FOR_DYNACONF`

Определяет, будет ли dynaconf работать в многоуровневой среде, допускающей разделение среды `[default]`, `[development]`, `[production]` и других.

### envvar

* Тип - `str`
* По умолчанию - `SETTINGS_FILE_FOR_DYNACONF`
* env-var - `ENVVAR_FOR_DYNACONF`

Задает имя переменной среды, которое будет использоваться для загрузки файлов настроек.

Например:

```bash
# устанавливает MY_SETTINGS_PATH в качестве новой переменной окружения,
# определяющей пути к файлам настроек.
export ENVVAR_FOR_DYNACONF="MY_SETTINGS_PATH"

# теперь его значение будет переопределять значения *settings_file*
# или SETTINGS_FILE_FOR_DYNACONF.
export MY_SETTINGS_PATH="path.to.settings"
```

{% hint style="warning" %}
**Устаревший вариант**

Предпочтительный способ определения файлов настроек — через файл [settings\_file](konfiguraciya-dynaconf.md#settings\_file-or-settings\_files). Подумайте, действительно ли вам нужно это использовать.
{% endhint %}

### **envvar\_prefix**

* Тип - `str`
* По умолчанию - `"DYNACONF"`
* env-var - `ENVVAR_PREFIX_FOR_DYNACONF`

Префикс, используемый dynaconf для загрузки значений из переменных среды. Возможно, вы захотите, чтобы ваши пользователи экспортировали значения, используя имя вашего приложения, например: `export MYPROGRAM_DEBUG=true`

### env

* Тип - `str`
* По умолчанию - `"development"`
* env-var - `ENV_FOR_DYNACONF`

Когда **environments** установлена, она управляет текущей средой, используемой во время выполнения. Обычно это устанавливается с помощью переменной среды.

```bash
export ENV_FOR_DYNACONF=production
```

Или при выполнении: `ENV_FOR_DYNACONF=production program.py`

### **env\_switcher**

* Тип - `str`
* По умолчанию - `"ENV_FOR_DYNACONF"`
* env-var - `ENVVAR_SWITCHER_FOR_DYNACONF`

По умолчанию **ENV\_FOR\_DYNACONF** — это переменная, используемая для переключения **envs**, но вы можете установить для нее другое имя переменной. пример: `MYPROGRAM_ENV=production`

### **filtering\_strategy**

* Тип - `class`
* По умолчанию - `None`
* env-var - `FILTERING_STRATEGY_FOR_DYNACONF`

Вызываемый объект приема данных для фильтрации, встроенные функции в настоящее время включают **PrefixFilter**

### **force\_env**

* Тип - `str`
* По умолчанию - `None`
* env-var - `FORCE_ENV_FOR_DYNACONF`

Когда **environments** включена, может быть полезно принудительно использовать конкретную среду для целей тестирования.

### **fresh\_vars**

* Тип - `list`
* По умолчанию - `[]`
* env-var - `FRESH_VARS_FOR_DYNACONF`

В некоторых случаях вам может потребоваться, чтобы некоторые переменные всегда перезагружались из исходного кода без необходимости перезагрузки приложения. Переменные, переданные в этот список, при доступе всегда вызывают перезагрузку из источника, файлы настроек будут считаны, **envvars** проанализированы и внешние загрузчики подключены. Пример:

`fresh_vars=["password"]` поэтому при доступе к `settings.password` он будет прочитан из источника, а не из кэша.

### includes

* Тип - `list | str`
* По умолчанию - `[]`
* env-var - `INCLUDES_FOR_DYNACONF`

После загрузки файлов, указанных в **settings\_files**, dynaconf загрузит все файлы, добавленные во **includes**, в процессе пост-загрузки. Обратите внимание, что:

* glob разрешен
* Когда даны относительные пути, он будет использовать [root\_path](konfiguraciya-dynaconf.md#settings\_file-or-settings\_files) в качестве их базового каталога.
* Если **root\_path** не указан явно, произойдет возврат к каталогу последней загруженной настройки или к **cwd**. Например:

```python
# будет искать src/extra.yaml
Dynaconf(root_path="src/", includes="extra.yaml")

# будет искать conf/extra.yaml
Dynaconf(settings_files="conf/settings.yaml", includes="extra.yaml")

# будет искать $PWD/extra.yaml
Dynaconf(includes="extra.yaml")
```

{% hint style="success" %}
**Совет**

**includes** также могут быть добавлены внутри самих файлов, например:

```python
dynaconf_include: ['otherfile.toml'] # или
dynaconf_include: 'otherfile.toml'
key: "value"
```

Или используя env vars:

```bash
export INCLUDES_FOR_DYNACONF="['otherfile.toml', 'path/*.yaml']"
```
{% endhint %}

### loaders

* Тип - `list`
* По умолчанию - `['dynaconf.loaders.env_loader']`
* env-var - `LOADERS_FOR_DYNACONF`

Список загрузчиков, которые dynaconf будет запускать после основных загрузчиков, этот список полезен для включения [пользовательских загрузчиков](rasshirennoe-ispolzovanie-dynaconf.md#sozdanie-novykh-zagruzchikov), а также для управления загрузкой переменных **env** в качестве последнего шага.

{% hint style="warning" %}
По умолчанию **env\_loader** будет последним в этом списке, поэтому он гарантирует, что переменные среды имеют приоритет в соответствии с философией Unix.
{% endhint %}

Узнать больше о [загрузчиках](rasshirennoe-ispolzovanie-dynaconf.md)

### load\_dotenv

* Тип - `bool`
* По умолчанию - `False`
* env-var - `LOAD_DOTENV_FOR_DYNACONF`

При включении dynaconf попытается загрузить переменные из файла `.env`.

### lowercase\_read

* Тип - `bool`
* По умолчанию - `True`
* env-var - `LOWERCASE_READ_FOR_DYNACONF`

По умолчанию dynaconf разрешает доступ к переменным первого уровня в нижнем регистре, поэтому `settings.foo` и `settings.FOO` эквивалентны (без учета регистра). При необходимости его можно отключить.

### merge\_enabled

* Тип - `bool`
* По умолчанию - `False`
* env-var - `MERGE_ENABLED_FOR_DYNACONF`

Если указанный Dynaconf входит в режим **GLOBAL MERGE** и при загрузке новых файлов или источников не переопределяет, а объединяет структуры данных по умолчанию.

Например:

```toml
# settings.toml
[server]
data = {port=8888}
```

```toml
# other.toml
[server]
data = {address='server.com'}
```

С **merge\_enabled** конечный файл `settings.server` будет

```python
{"port": 8888, "address": "server.com"}
```

В противном случае будет только то, что указано в последнем загруженном файле. подробнее о [слиянии стратегий](sliyanie-merging.md)

### nested\_separator

* Тип - `str`
* По умолчанию - `"__"`  (двойное подчеркивание)
* env-var - `NESTED_SEPARATOR_FOR_DYNACONF`

Одной из [стратегий слияния](sliyanie-merging.md) является использование `__` для доступа к структурам данных вложенного уровня. По умолчанию разделителем является `__` (двойное подчеркивание), эта переменная позволяет вам изменить его.

Пример:

```bash
export DYNACONF_DATABASES__default__ENGINE__Address="0.0.0.0"
```

генерирует:

```python
DATABASES = {
    "default": {
        "ENGINE": {
            "Address": "0.0.0.0"
        }
    }
}
```

{% hint style="warning" %}
Выберите то, что подходит для **env vars**, обычно вам не нужно менять эту переменную.
{% endhint %}

{% hint style="warning" %}
В Windows env vars все преобразуются в верхний регистр.
{% endhint %}

### preload

* Тип - `list | str`
* По умолчанию - `[]`
* env-var - `PRELOAD_FOR_DYNACONF`

Иногда вам может понадобиться загрузить какой-то файл, прежде чем dynaconf начнет искать свои файлы настроек **settings\_files** или **includes**, например, вы можете иметь `dynaconf.toml` для хранения конфигурации dynaconf.

### redis\_enabled

* Тип - `bool`
* По умолчанию - `False`
* env-var - `REDIS_ENABLED_FOR_DYNACONF`

Если установлено значение `True`, dynaconf будет включать `dynaconf.loaders.redis_loaders` в список загрузчиков **loaders**.

### redis

* Тип - `dict`
* По умолчанию - `{ слишком длинный, см. ниже }`
* env-var - `REDIS_FOR_DYNACONF`

Когда **redis\_enabled** имеет значение `True`, словарь, содержащий настройки redis.

```python
default = {
    "host": "REDIS_HOST_FOR_DYNACONF" or "localhost",
    "port": int("REDIS_PORT_FOR_DYNACONF" or 6379),
    "db": int("REDIS_DB_FOR_DYNACONF" or 0),
    "decode_responses": "REDIS_DECODE_FOR_DYNACONF" or True,
}
```

### **root\_path** <a href="#settings_file-or-settings_files" id="settings_file-or-settings_files"></a>

* Тип - `str`
* По умолчанию - `None`
* env-var - `ROOT_PATH_FOR_DYNACONF`

Рабочий путь для dynaconf, используемый в качестве отправной точки при поиске файлов.

Обратите внимание, что:

* Для относительного пути в качестве базового пути используется **cwd**.
* Если явно не задано, внутреннее значение для **root\_path** будет установлено следующим образом:
  * Место последнего загруженного файла, если какие-либо файлы уже были загружены
  * CWD

Подробнее читайте в [settings\_files](faily-nastroek-dynaconf.md#zagruzka-failov-nastroek).

{% hint style="info" %}
**cwd** — это место, откуда был вызван интерпретатор Python.
{% endhint %}

### secrets

* Тип - `str`
* По умолчанию - `None`
* env-var - `SECRETS_FOR_DYNACONF`

Эта переменная полезна для CI и Jenkins, обычно устанавливается как переменная среды, указывающая на файл, содержащий загружаемые секреты. Пример:

```bash
export SECRETS_FOR_DYNACONF=path/to/secrets.yaml
```

### **settings\_file** (или **settings\_files**) <a href="#settings_file-or-settings_files" id="settings_file-or-settings_files"></a>

* Тип - `str | list`
* По умолчанию - `[]`
* env-var - `SETTINGS_FILE_FOR_DYNACONF` или `SETTINGS_FILES_FOR_DYNACONF`

{% hint style="info" %}
Эта переменная требуется начиная с версии 3.0.0 (см. [#361](https://github.com/dynaconf/dynaconf/pull/361)).
{% endhint %}

Путь к файлам, из которых вы хотите, чтобы dynaconf загружал настройки. Это может быть `settings_file` (единственное число) или `settings_files` (множественное).

Это может быть список **list** или строка **str**, разделенные точкой с запятой/запятой:

```python
settings_files=["file1.toml", "file2.toml"] # список list
settings_files="file1.toml;file2.toml" # точка с запятой
settings_files="file1.toml,file2.toml" # запятая
```

Также это может быть один файл.

```python
settings_file="settings.toml"
```

Или указывается как env var.

```bash
# одиночный
export SETTINGS_FILE_FOR_DYNACONF="/etc/program/settings.toml"

# множественный
export SETTINGS_FILES_FOR_DYNACONF="/etc/program/settings.toml;/tmp/foo.yaml"
```

Подробнее о [settings\_files](faily-nastroek-dynaconf.md)

### skip\_files

* Тип - `list`
* По умолчанию - `None`
* env-var - `SKIP_FILES_FOR_DYNACONF`

При использовании glob для **includes** вы можете захотеть, чтобы dynaconf игнорировал некоторые совпадающие файлы.

```python
skip_files=["path/to/ignored.toml"]
```

### sysenv\_fallback

* Тип - `bool | list[str]`
* По умолчанию - `False`
* env-var - `SYSENV_FALLBACK_FOR_DYNACONF`

Определяет, будет ли dynaconf пытаться загрузить системные переменные среды (без префикса) в качестве запасного варианта при использовании `settings.get()`.

Допустимые варианты:

* `False` (по умолчанию): не будет пытаться загрузить системную среду **envvar**
* `True`: попытается загрузить **sys envvar** для любого имени, запрошенного в `get()`
* `list[str]`: попытается загрузить **sys envvar** для имен, указанных в списке.

Это поведение также можно определить для каждого вызова, указав параметр **sysenv\_fallback** в `settings.get()`.

Пример:

```python
from dynaconf import Dynaconf

# будет искать PATH (верхний регистр) в переменных системной среды
settings = Dynaconf(sysenv_fallback=True)
settings.get("path")

# может быть установлено для каждого вызова
settings = Dynaconf()
settings.get("path", sysenv_fallback=True)

# если используется список, он фильтрует разрешенные имена.
settings = Dynaconf(sysenv_fallback=["path"])
settings.get("path")
settings.get("user") # не буду пытаться загружать без префикса USER
```

### validate\_on\_update

* Тип - `False | True | "all"`
* По умолчанию - `False`
* env-var - `VALIDATE_ON_UPDATE_FOR_DYNACONF`

Определяет, будет ли запускаться проверка при обновлении экземпляра Dynaconf с помощью методов `update()`, `set()` или `load_file()`. Также определяет, какая стратегия повышения будет использоваться (см. подробнее о проверке [Validation](validaciya-v-dynaconf.md#lenivaya-validaciya)).

Допустимые варианты:

* `False` (по умолчанию): автозапуск отключен
* `True`: срабатывает как `settings.validate()`, который вызывается при первых ошибках
* `"all"`: триггер как `settings.validate_all()`, который накапливает ошибки

Это поведение также может быть определено для каждого вызова. Просто укажите параметр validate для любого из методов обновления данных.

Пример:

```python
settings = Dynaconf(validate_on_update="all")
settings.validators.register(Validator("value_a", must_exist=True))

settings.update({"value_b": "foo"}, validate=False) # будет обходить проверку
settings.update({"value_b": "foo"}, validate=True) # будет вызывать .validate()
settings.update({"value_b": "foo"}) # будет вызывать .validate_all()
```
