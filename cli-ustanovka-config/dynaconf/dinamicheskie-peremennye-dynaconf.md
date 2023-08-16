# Динамические переменные dynaconf

## Подстановки шаблонов

Dynaconf имеет 2 токена для включения замены строк, интерполяция работает с **@format** и **@jinja**.

### Токен @format

Dynaconf позволяет заменять шаблоны строковыми значениями, используя префикс токена **@format** и включая заполнители, принятые методом Python `str.format`. Dynaconf будет лениво вызывать его во время доступа.

Вызов будет таким:

```python
"<YOURVALUE>".format(env=os.environ, this=dynaconf.settings)
```

Таким образом, в вашей строке вы можете ссылаться на переменные среды через объект **env**, а также на переменные, определенные в самом объекте настроек с помощью ссылки **this**. Он лениво оценивается при доступе, он будет использовать окончательное значение для настроек независимо от порядка загрузки.

Пример:

```bash
export PROGRAM_NAME=calculator
```

`settings.toml`

```toml
[default]
DB_NAME = "mydb.db"

[development]
DB_PATH = "@format {env[HOME]}/{this.current_env}/{env[PROGRAM_NAME]}/{this.DB_NAME}"
```

* `{env[HOME]}` совпадает с `os.environ["HOME"]` или `$HOME` в оболочке.
* `{this.current_env}` совпадает с `settings.current_env`.
* `{env[PROGRAM_NAME]}` совпадает с `os.environ["PROGRAM_NAME"]` или `$PROGRAM_NAME` в оболочке.
* `{this.DB_NAME}` совпадает с `settings.DB_NAME` или `settings["DB_NAME"]`

так в вашей программе `program`

```python
from dynaconf import settings

settings.DB_PATH == '~/DEVELOPMENT/calculator/mydb.db'
```

Токен **@format** можно использовать вместе с такими токенами, как **@str**, **@int**, **@float**, **@bool** и **@json**, для приведения результатов, проанализированных **@format**.

Пример:

Приведение к целому числу из шаблонных значений **@format**

```yaml
NUM_GPUS: 4
# Это возвращает целое число после парсинга
FOO_INT: "@int @format {this.NUM_GPUS}"
# Это возвращает строку после парсинга
FOO_STR: "@format {this.NUM_GPUS}"
```

### @jinja token <a href="#jinja-token" id="jinja-token"></a>

Если установлен пакет **jinja2**, то dynaconf также позволит использовать jinja для отображения строковых значений.

Пример:

```bash
export PROGRAM_NAME=calculator
```

`settings.toml`

```toml
[default]
DB_NAME = "mydb.db"

[development]
DB_PATH = "@jinja {{env.HOME}}/{{this.current_env | lower}}/{{env['PROGRAM_NAME']}}/{{this.DB_NAME}}"
```

так в вашей программе **program**

```python
from dynaconf import settings

settings.DB_PATH == '~/development/calculator/mydb.db'
```

Основное отличие состоит в том, что Jinja позволяет вычислять некоторые выражения Python, такие как `{% for, if, while %}`, а также поддерживает методы вызова и имеет множество фильтров, таких как `| lower`.

Jinja поддерживает свои встроенные фильтры, перечисленные на [странице встроенных фильтров](http://jinja.palletsprojects.com/en/master/templates/#builtin-filters), а Dynaconf включает дополнительные фильтры для модуля `os.path`: **abspath**, **realpath**, **relpath**, **basename** и **dirname** и использование похоже на:

`VALUE = "@jinja {{this.FOO | abspath}}"`

Токен **@jinja** можно использовать вместе с такими токенами, как **@str**, **@int**, **@float**, **@bool** и **@json**, для приведения результатов, проанализированных **@jinja**.

Пример:

Приведение к целому числу из шаблонных значений **@jinja**

```yaml
NUM_GPUS: 4
# Это возвращает целое число после парсинга
FOO_INT: "@int @jinja {{ this.NUM_GPUS }}"
# Это возвращает строку после парсинга
FOO_STR: "@jinja {{ this.NUM_GPUS }}"
```

Пример:

Приведение к json словаря из шаблонных значений **@jinja**

```yaml
MODEL_1:
  MODEL_PATH: "src.main.models.CNNModel"
  GPU_COUNT: 4
  OPTIMIZER_KWARGS:
    learning_rate: 0.002

MODEL_2:
  MODEL_PATH: "src.main.models.VGGModel"
  GPU_COUNT: 2
  OPTIMIZER_KWARGS:
    learning_rate: 0.001

# Флаг, чтобы выбрать, какую модель использовать
MODEL_TYPE: "MODEL_2"

# Это возвращает dict, загруженный из полученного json
MODEL_SETTINGS_DICT: "@json @jinja {{ this|attr(this.MODEL_TYPE) }}"

# Это возвращает необработанную строку json
MODEL_SETTINGS_STR: "@jinja {{ this|attr(this.MODEL_TYPE) }}"
```
