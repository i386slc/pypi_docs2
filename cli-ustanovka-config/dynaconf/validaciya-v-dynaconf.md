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
