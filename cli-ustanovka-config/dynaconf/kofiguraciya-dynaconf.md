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
