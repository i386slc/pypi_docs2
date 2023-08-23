# Валидация в dynaconf

Dynaconf позволяет проверять параметры настроек, в некоторых случаях вы можете захотеть проверить настройки перед запуском программы.

Допустим, у вас есть `settings.toml`

```toml
[default]
version = "1.0.0"
age = 35
name = "Bruno"
DEV_SERVERS = ['127.0.0.1', 'localhost', 'development.com']
PORT = 8001
JAVA_BIN = "/usr/bin/java"

[production]
PROJECT = "This is not hello_world"
```

## Валидация в Python программно

### При создании экземпляра

Когда вы создадите свои настройки, Dynaconf запустит все валидаторы, которые вы определили, в отношении ваших исходных данных.

```python
from pathlib import Path
from dynaconf import Dynaconf, Validator


settings = Dynaconf(
    validators=[
        # Убедитесь, что некоторые параметры существуют (обязательны)
        Validator('VERSION', 'AGE', 'NAME', must_exist=True),

        # Убедитесь, что какой-либо пароль не может существовать
        Validator('PASSWORD', must_exist=False),

        # Убедитесь, что какой-то параметр соответствует условию
        # условия: (eq, ne, lt, gt, lte, gte, identity, is_type_of, is_in, is_not_in)
        Validator('AGE', lte=30, gte=10),

        # проверить значение eq в конкретной среде env
        Validator('PROJECT', eq='hello_world', env='production'),

        # Убедитесь, что какой-то параметр (строка) соответствует условию
        # условия: (len_eq, len_ne, len_min, len_max, cont)
        # Определяет минимальную и максимальную длину значения
        Validator("NAME", len_min=3, len_max=125),

        # Означает наличие значения в наборе, тексте или слове
        Validator("DEV_SERVERS", cont='localhost'),

        # Проверяет, совпадает ли длина с определенной в настройках
        Validator("PORT", len_eq=4),

        # Убедитесь, что java_bin возвращается как экземпляр Path
        Validator("JAVA_BIN", must_exist=True, cast=Path),

        # Убедитесь, что значение соответствует условию, указанному вызовом функции 
        Validator("VERSION", must_exist=True, condition=lambda v: v.startswith("1.")),
    ]
)
```

Вышеупомянутое поднимет:

`dynaconf.validator.ValidationError("AGE must be lte=30 but it is 35 in env DEVELOPMENT")`

и

`dynaconf.validator.ValidationError("PROJECT must be eq='hello_world' but it is 'This is not hello_world' in env PRODUCTION")`

### Ленивая валидация

Вместо передачи аргумента `validators=` классу Dynaconf вы можете зарегистрировать валидаторы после создания экземпляра и запустить его вручную.

#### Регистрация

Во-первых, зарегистрируйте несколько валидаторов. Это еще не вызовет проверку.

```python
settings = Dynaconf()

settings.validators.register(
    Validator("MYSQL_HOST", eq="development.com", env="DEVELOPMENT"),
    Validator("MYSQL_HOST", ne="development.com", env="PRODUCTION"),
)
```

#### Запуск вручную

Вы можете выбрать две стратегии проверки:

* **validate**: вызывает **ValidationError** при первой обнаруженной ошибке
* **validate\_all**: вызывает **ValidationError** в конце. Накопленные данные об ошибках хранятся в **details**

```python
# поднимается при обнаружении первой ошибки
settings.validators.validate()

# возникает после оценки всех возможных ошибок
try:
    settings.validators.validate_all()
except dynaconf.ValidationError as e:
    accumulative_errors = e.details
    print(accumulative_errors)
```

#### Запуск при обновлении данных

По умолчанию, если данные экземпляра обновляются с помощью методов **update**, **set** или **load\_file**, проверка не запускается.

Вы можете переопределить это глобально с помощью опции [validate\_on\_update](konfiguraciya-dynaconf.md#validate\_on\_update) или установить это для каждого вызова.

```python
# validate_on_update=False (по умолчанию)
# триггеры validators.validate()
settings.update({"NEW_VALUE": 123}, validate=True)
# триггеры validators.validate_all()
settings.update({"NEW_VALUE": 123}, validate="all")

# validate_on_update=True или "all"
# сработает с глобальной стратегией
settings.update({"NEW_VALUE": 123})
```

## Параметры валидатора

Валидаторы могут быть созданы путем передачи следующих аргументов:

```python
# названия: list[str]
# может быть одной или несколькими позиционными строками 
Validator('VERSION', 'AGE', 'NAME'),
# также можно использовать точечную запись
Validator('DATABASE.HOST', 'DATABASE.PORT'),
Validator('DATABASE.HOST', 'DATABASE.PORT'),


# must_exist: bool (alias: required)
# Проверьте, должна ли переменная существовать или нет 
Validator('VERSION', must_exist=True), 
Validator('PASSWORD', must_exist=False),
# существует псевдоним для must_exist под названием `required`
Validator('VERSION', required=True), 

# condition: callable 
# Функция или любой другой вызываемый объект, который принимает значение `value`
# и должен возвращать логическое значение. 
Validator('VERSION', condition=lambda v: v.startswith("1.")),

# when: Validator
# Условно запускает валидатор только при прохождении переданного валидатора
Validator(
    'VERSION',
    condition=lambda v: v.endswith("-dev"),
    when=Validator('ENV_FOR_DYNACONF', eq='development')
),

# env: str 
# Запускает валидатор для указанного окружения только для настроек,
# загружаемых из файлов с environments=True 
Validator('VERSION', must_exist=True, env='production'),

# messages: dict[str, str]
# Словарь с настраиваемыми сообщениями для каждого типа проверки. 
Validator(
    "VERSION", 
    must_exist=True,
    condition=lambda v: v.startswith("1."),
    messages={
        "must_exist_true": "You forgot to set {name} in your settings.",
        "condition": "The allowed version must start with 1., you passed {value}"
    }
),

# cast: callable/class 
# Тип или вызываемый объект для преобразования типа переданного объекта
# также можно использовать для применения преобразований/нормализации.
Validator("VERSION", cast=str),
Validator("VERSION", cast=lambda v: v.replace("1.", "2.")),
Validator("STATIC_FOLDER", cast=Path)
# Приведение будет вызываться для значений по умолчанию, а также для значений,
# определенных в настройках через файлы или envvars.

# default: any value or a callable 
# Если значение не найдено, ему будет установлено значение по умолчанию. 
# Если по умолчанию используется вызываемый объект, он будет вызываться
# с настройками и экземпляром валидатора в качестве аргументов.
def default_connection_args(settings, validator):
    if settings.DATABASE.uri.startswith("sqlite://"):
        return {"echo": True}
    else:
        return {}

Validator("DATABASE.CONNECTION_ARGS", default=default_connection_args), 
# default будет вызываться только в том случае, если значение не установлено явно
# в настройках через файлы или переменные окружения.

# description: str
# Начиная с версии 3.1.12, dynaconf ни для чего не использует это,
# но существуют плагины и внешние инструменты, которые его используют.
# Это значение для создания документации 
Validator("VERSION", description="The version of the app"),

# apply_default_on_none: bool 
# Синтаксический анализатор YAML анализирует пустые значения как `None`,
# поэтому в этом случае вы можете принудительно применить значение по умолчанию,
# когда в настройках установлено значение `None`.
Validator("VERSION", default="1.0.0", apply_default_on_none=True),


# Operations: comparison operations 
# - eq: value == other
# - ne: value != other
# - gt: value > other
# - lt: value < other
# - gte: value >= other
# - lte: value <= other
# - is_type_of: isinstance(value, type)
# - is_in:  value in sequence
# - is_not_in: value not in sequence
# - identity: value is other
# - cont: contain value in
# - len_eq: len(value) == other
# - len_ne: len(value) != other
# - len_min: len(value) > other
# - len_max: len(value) < other
# - startswith: value.startswith(other) 
# - endswith: value.endswith(other)
# Примеры:
Validator("VERSION", eq="1.0.0"),
Validator("VERSION", ne="1.0.0"),
Validator("AGE", gt=18),
Validator("AGE", lt=18),
Validator("AGE", gte=18),
Validator("AGE", lte=18),
Validator("AGE", is_type_of=int),
Validator("AGE", is_in=[18, 19, 20]),
Validator("AGE", is_not_in=[18, 19, 20]),
Validator("THING", identity=thing),  # settings.THING is thing
Validator("THING", cont="hello"),  # "hello" in settings.THING
Validator("THING", len_eq=3),  # len(settings.THING) == 3
Validator("THING", len_ne=3),  # len(settings.THING) != 3
Validator("THING", len_min=3),  # len(settings.THING) > 3
Validator("THING", len_max=3),  # len(settings.THING) < 3
Validator("THING", startswith="hello"),  # settings.THING.startswith("hello")
Validator("THING", endswith="world"),  # settings.THING.endswith("world")
```

## Комплексные валидаторы

Один валидатор может иметь несколько условий.

```python
Validator(
  "NAME",
  ne="john",
  len_min=4,
  must_exist=True, # излишне, но разрешено 
  startswith="user_",
  cast=str,
  condition=lambda v: v not in FORBIDEN_USERS,
  ...
)
```

Но также может быть выражено в отдельных валидаторах, обратите внимание, что порядок имеет значение, поскольку валидаторы оцениваются в заданном порядке.

```python
validators = [
  Validator("NAME", ne="john"),
  Validator("NAME", len_min=4),
  Validator("NAME", must_exist=True),
  Validator("NAME", startswith="user_"),
]
```

## Пользовательские сообщения проверки

Сообщения можно настроить, передав аргумент **messages** конструктору **Validator**.

Аргументу **messages** должен быть передан словарь с одним из допустимых ключей:

Сообщения по умолчанию:

```python
{
    "must_exist_true": "{name} is required in env {env}",
    "must_exist_false": "{name} cannot exists in env {env}",
    "condition": "{name} invalid for {function}({value}) in env {env}",
    "operations": (
        "{name} must {operation} {op_value} "
        "but it is {value} in env {env}"
    ),
    "combined": "combined validators failed {errors}",
}
```

Пример:

```python
Validator(
    "VERSION",
    must_exist=True,
    messages={"must_exist_true": "You forgot to set {name} in your settings."}
)
```

## Предоставление значений по умолчанию или вычисленных значений

Валидаторы можно использовать для предоставления значений по умолчанию или вычисленных значений.

### Значения по умолчанию

```python
Validator("FOO", default="A default value for foo")
```

Затем, если невозможно загрузить значения из файлов или среды, для этого ключа будет установлено значение по умолчанию.

{% hint style="warning" %}
**YAML** считывает пустые ключи как **None**, и в этом случае значения по умолчанию не применяются. Если вы хотите изменить их, установите `apply_default_on_none=True` либо глобально для класса **Dynaconf**, либо индивидуально для валидатора **Validator**.
{% endhint %}

### Вычисленные значения

Иногда вам нужно вычислить некоторые значения путем вызова функций, просто передайте вызываемый объект в качестве аргумента по умолчанию.

```python
Validator("FOO", default=my_function)
```

затем

```python
def my_function(settings, validator):
    return "this is computed during validation time"
```

Если вы хотите, чтобы оценивалось лениво, **my\_function** необходимо переопределить как

```python
def my_lazy_function(value, **context):
    """
    value: Значение по умолчанию, передаваемое валидатору, по умолчанию равно `empty`
    context: Словарь, содержащий
            env: Все переменные среды
            this: Экземпляр настроек
    """
    return "When the first value is accessed, then the my_lazy_function will be called"
```

Впоследствии

```python
from dynaconf.utils.functional import empty
from dynaconf.utils.parse_conf import Lazy

Validator("FOO", default=Lazy(empty, formatter=my_lazy_function))
```

Вы также можете использовать пути, разделенные точками, для регистрации валидаторов во вложенных структурах:

```python
# Регистрация валидаторов
settings.validators.register(

    # Убедитесь, что поле database.host существует.
    Validator('DATABASE.HOST', must_exist=True),

    # Сделайте поле database.password необязательным. Это поведение по умолчанию.
    Validator('DATABASE.PASSWORD', must_exist=None),
)

# Запустите валидатор
settings.validators.validate()
```

### Приведение/Трансформация

Валидаторы можно использовать для приведения значений к определенному типу, аргумент **cast** ожидает класс/тип или вызываемый объект.

учитывая эти `settings.toml`

```toml
name = 'Bruno'
colors = ['red', 'green', 'blue']
```

Валидаторам можно передать атрибут **cast**

```python
settings = Dynaconf(
    validators=[
        # Здесь важен порядок
        Validator("name", len_eq=5),
        Validator("name", len_min=1),
        Validator("name", len_max=5),
        # Это приведет к отображению строки в списке
        Validator("name", cast=list),
        # С этого момента `name` конвейера проверки будет представлять
        # собой список символов, и это повлияет на settings.NAME

        Validator("colors", len_eq=3),
        Validator("colors", len_eq=3),
        # это приведет к преобразованию списка в строку
        Validator("colors", len_eq=24, cast=str),
        # С этого момента `colors` конвейера проверки будут представлять собой
        # строку из 24 символов, и это повлияет на settings.COLORS.

    ],
)
```

```python
assert settings.name == ['B', 'r', 'u', 'n', 'o']
assert type(settings.name ) == list
assert settings.colors == '["red", "green", "blue"]'
assert type(settings.colors) == str
```

### Вызываемые условия

Аргумент **condition** ожидает вызываемый объект, который получает значение и возвращает логическое значение. Если условие не выполнено, будет выдано сообщение **ValidationError**.

Чтобы пройти проверку, функция условия должна вернуть `True` (или истинный тип), если возвращаемое значение имеет значение `False` (или ложный тип), тогда условие не выполняется.

Вызываемое условие получает в качестве параметра только одно значение.

Пример:

```python
Validator("VERSION", condition=lambda v: v.startswith("1."))


def user_must_be_chuck_norris(value):
    return value == "Chuck Norris"

Validator("USER", condition=user_must_be_chuck_norris)
```

### Условные валидаторы

В некоторых случаях вам может потребоваться выполнить проверку только после прохождения другого валидатора, например:

> Убедитесь, что DATABASE.HOST установлен только тогда, когда установлен DATABASE.USER.

Чтобы проверить этот случай, можно передать параметр **when**:

```python
Validator(
    "DATABASE.HOST", 
    must_exist=True, 
    when=Validator("DATABASE.USER", must_exist=True)
)
```

Другой пример:

> Проверяет, что DATABASE.CONNECTION\_ARGS установлен только в том случае, если DATABASE.URI начинается с "sqlite://"

```python
Validator(
    "DATABASE.CONNECTION_ARGS", 
    must_exist=True, 
    when=Validator("DATABASE.URI", condition=lambda v: v.startswith("sqlite://")),
    messages={"must_exist_true": "{name} is required when DATABASE is SQLite"}
)
```

### Объединение валидаторов

Валидаторы можно комбинировать, используя:

#### Оператор ИЛИ `|`

```python
Validator('DATABASE.USER', must_exist=True) | Validator('DATABASE.KEY', must_exist=True)
```

#### Оператор И `&`

```python
Validator('DATABASE.HOST', must_exist=True) & Validator('DATABASE.CONN', must_exist=True)
```

## CLI и dynaconf\_validators.toml

Валидаторы можно определить в файле **TOML** с именем `dynaconf_validators.toml`, который находится в той же папке, что и ваши файлы настроек.

`dynaconf_validators.toml` эквивалентен приведенной ниже программе:

```toml
[default]

version = {must_exist=true}
name = {must_exist=true}
password = {must_exist=false}

# также поддерживается точечная запись
'a_big_dict.nested_1.nested_2.nested_3.nested_4' = {must_exist=true, eq=1}

  [default.age]
  must_exist = true
  lte = 30
  gte = 10

[production]
project = {eq="hello_world"}
```

Затем для запуска проверки используйте:

```bash
$ dynaconf validate
```

Это возвращает код 0 (success), если проверка прошла успешно.

{% hint style="info" %}
Все значения в dynaconf анализируются с использованием формата toml, TOML пытается быть умным и определить тип переменных настроек, некоторые переменные будут автоматически преобразованы в целые числа:

```toml
FOO = "0x..."  # hexadecimal
FOO = "0o..."  # Octal
FOO = "0b..."  # Binary
```

Все случаи соответствуют спецификациям **toml** [https://github.com/toml-lang/toml/blob/master/toml.abnf](https://github.com/toml-lang/toml/blob/master/toml.abnf).

Если вам нужно принудительно выполнить приведение определенного типа, есть 2 варианта.

* Используйте двойные кавычки для строк, например: `FOO = "'0x...'"` будет строкой.
* Укажите тип, используя **@** пример: `FOO = "@str 0x..."` (доступные конвертеры: **@int**, **@float**, **@bool**, **@json**)
{% endhint %}

## Выборочная проверка

> Новое в версии 3.1.6

Вы также можете выбрать, какие разделы настроек вы хотите или не хотите проверять. Это полезно, когда:

* Вы хотите добавить дополнительные валидаторы после создания объекта настроек.
* Вы хотите, чтобы настройки проверялись только при загрузке определенных разделов вашего проекта.
* Вы хотите предлагать дополнительные уровни конфигурации, проверяя только то, что необходимо.

Выборочная проверка может выполняться как при создании объекта настроек, так и при вызове валидаторов объекта настроек. Новые аргументы принимают либо строку, представляющую путь к настройкам, либо список строк, представляющих пути к настройкам.

Путь к настройкам начинается с элемента верхнего уровня и может быть указан до самого нижнего компонента. Например: `my_settings.server.user.password` может иметь следующие пути настроек, передаваемые в `server`, `server.user`, `server.user.password`.

{% hint style="info" %}
Выборочная проверка сопоставляет переданные значения с путями настроек, которые начинаются с этого значения. Это означает, что передача `ignore="FOO"` исключит не только пути, начинающиеся с FOO, но и FOOBAR.
{% endhint %}

Примеры:

#### **-- config.py --**

```python
...
# создает объект настроек, проверяя только настройки в settings.server
settings = Dynaconf(
    validators=[
        Validator(
            "server.hostname",
            "server.port",
            "server.auth",
            must_exist=True
        ),
        Validator(
            "module1.value1",
            "module1.value2",
            "module1.value3",
            must_exist=True
        ),
        Validator(
            "module2.value1",
            "module2.value2",
            "module2.bad",
            must_exist=True
        )
    ],
    validate_only="server"
)
```

#### **-- module1.py --**

```python
...
# вызывает проверку в настройках module1
settings.validators.validate(only=["module1"])
```

#### **-- module2.py --**

```python
...
# вызывает проверку в настройках module2
# игнорировать проверку подраздела настроек module2
settings.validators.validate(
    only=["module2"],
    exclude=["module2.bad"]
)
```

### Проверить только текущую среду

Вы можете указать, хотите ли вы проверять все среды, определенные для валидатора (поведение по умолчанию), или только текущую среду. В первом случае валидаторы будут работать со всеми возможными настройками, определенными в их списке сред, а во втором валидаторы со средами, отличными от текущей среды, будут пропущены.

Это полезно, когда ваша конфигурация для разных сред (скажем, **production** и **development**) исходит из разных файлов, к которым у вас не обязательно есть доступ во время разработки. Вы хотели бы написать разные валидаторы для своей среды **development** и **production** и запускать только тот валидатор, который подходит для текущей среды.

Вот пример использования опции:

#### settings.toml

```toml
[development]
version = "dev"
age = 35
name = "Bruno"
servers = ['127.0.0.1', 'localhost', 'development.com']
PORT = 80

[production]
version = "1.0.0"
age = 35
name = "Bruno"
servers = ['production.com']
PORT = 443
```

#### .secrets.toml

```toml
[production]
api_key = 'secret_api_key'
```

Затем вы могли бы иметь эти валидаторы:

```python
from dynaconf import Dynaconf, Validator

settings = Dynaconf(
    settings_files=['setting.toml', '.secrets.toml'],
    environments=True,
    validators=[
        # Убедитесь, что некоторые параметры существуют для обоих окружений.
        Validator('VERSION', 'NAME', 'SERVERS', envs=['development', 'production'], must_exist=True),

        # Убедитесь, что параметр проверяет определенное условие в среде разработки.
        Validator("SERVERS", env='development', cont='localhost'),

        # Убедитесь, что какой-то параметр существует в производственной среде
        Validator('API_KEY', env='production', must_exist=True),
    ]
)
```

И предположим, что во время разработки, когда `settings.current_env == 'development'`, у вас нет файла `.secrets.toml`.

Запуск `settings.validators.validate()` завершится неудачей, даже если `settings.current_env == 'development'`, потому что по умолчанию все валидаторы будут работать во всех своих средах, независимо от того, является ли это текущим окружением или нет. Однако вы можете создать свои настройки с параметром `validate_only_current_env=True`, и ничего не будет поднято, если `settings.current_env == 'development'`. Тем не менее, если `settings.current_env == 'production'`, произойдет сбой, и вам придется иметь файл `.secrets.toml` в каталоге в `production`, но не обязательно во время `development`.
