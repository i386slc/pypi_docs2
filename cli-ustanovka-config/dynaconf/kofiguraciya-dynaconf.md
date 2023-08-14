# Кофигурация dynaconf

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
**default\_env** не является средой, которая будет активна при запуске. Для этого см. [env](kofiguraciya-dynaconf.md#env).
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

Предпочтительный способ определения файлов настроек — через файл [settings\_file](kofiguraciya-dynaconf.md#settings\_file-or-settings\_files). Подумайте, действительно ли вам нужно это использовать.
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

### **settings\_file** (или **settings\_files**) <a href="#settings_file-or-settings_files" id="settings_file-or-settings_files"></a>
