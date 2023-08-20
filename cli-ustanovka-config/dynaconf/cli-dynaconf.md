# CLI dynaconf

## Экземпляр настроек settings

Каждая команда (кроме **init**) требует, чтобы экземпляр можно было установить с помощью параметра `-i` или экспортировать **INSTANCE\_FOR\_DYNACONF**.

## Интерфейс командной строки dynaconf

Командная строка `$ dynaconf -i config.settings` предоставляет несколько полезных команд.

{% hint style="info" %}
**ВАЖНО!** Если вы используете [Flask Extension](flask-i-dynaconf.md), env var **FLASK\_APP** должен быть определен для использования CLI, а если вы используете [расширение Django](django-i-dynaconf.md), должен быть определен **DJANGO\_SETTINGS\_MODULE**.
{% endhint %}

### dynaconf --help

```
Usage: dynaconf [OPTIONS] COMMAND [ARGS]...

  Dynaconf - Command Line Interface

  Документация: https://dynaconf.com/

Options:
  --version            Показать версию dynaconf
  --docs               Открыть документацию в браузере
  --banner             Показать крутой баннер
  -i, --instance TEXT  Пользовательский экземпляр LazySettings
  --help               Показать это сообщение и выйти.

Команды:
  get       Возвращает необработанное значение для ключа настроек.
  init      Инициализирует проект dynaconf.
  inspect   Проверяет историю загрузки данного экземпляра настроек.
  list      Список пользовательских настроек или всех (включая внутренние конфигурации).
  validate  Проверяет настройки Dynaconf на основе предоставленных правил.
  write     Записывает данные в определенный источник.
```

### dynaconf init <a href="#dynaconf-init" id="dynaconf-init"></a>

Используйте **init**, чтобы легко настроить конфигурацию вашего приложения. После установки dynaconf перейдите в корневой каталог вашего приложения и запустите:

```bash
$ dynaconf init -v key=value -v foo=bar -s token=1234
```

Приведенная выше команда создаст в текущем каталоге

`settings.toml`

```toml
KEY = "value"
FOO = "bar"
```

также `.secrets.toml`

```toml
TOKEN = "1234"
```

а также файл `.gitignore`, игнорирующий сгенерированный `.secrets.toml`

```gitignore
# Игнорировать секретные файлы dynaconf
.secrets.*
```

Для конфиденциальных данных в производстве рекомендуется использовать [Vault Server](sekretnaya-informaciya-secrets.md#ispolzovanie-servera-vault).

```
Использование: dynaconf init [OPTIONS]

  Инициализирует проект dynaconf.

  По умолчанию он создает файл settings.toml и .secrets.toml
  для [default|development|staging|testing|production|global]
  envs.

  Формат файлов можно изменить, добавив --format=yaml|json|ini|py.

  Эта команда должна выполняться в корневой папке проекта, или вы должны передать
  --path=/myproject/root/folder.

  --env/-e устарела (сохранено для совместимости, но не используется)

Параметры:
  -f, --format [ini|toml|yaml|json|py|env]
  -p, --path TEXT                 по умолчанию текущий каталог
  -e, --env TEXT                  Устанавливает рабочую среду в файле `.env`
  -v, --vars TEXT                 дополнительные значения для записи в файл настроек,
                                  например: `dynaconf init -v NAME=foo -v X=2

  -s, --secrets TEXT              значения секретного ключа, которые будут записаны
                                  в .secrets, н-р: `dynaconf init -s TOKEN=kdslmflds

  --wg / --no-wg
  -y
  --django TEXT
  --help                          Показать это сообщение и выйти.
```

Обратите внимание, что `-i/--instance` нельзя использовать с **init**, так как `-i` должен указывать на существующий экземпляр настроек.

### Dynaconf inspect (технический предварительный просмотр) <a href="#dynaconf-inspect-tech-preview" id="dynaconf-inspect-tech-preview"></a>

> НОВОЕ в версии 3.2.0

{% hint style="warning" %}
Эта функция находится в технической предварительной версии, интерфейс использования и формат вывода могут быть изменены.
{% endhint %}

Просмотрите и создайте дамп истории загрузки данных о конкретном ключе или среде.

Эта команда также доступна как служебная функция в `dynaconf.inspect_settings` (подробнее).

```
Использование: dynaconf inspect [OPTIONS]

  Проверяет историю загрузки данного экземпляра настроек.

  Фильтрует по ключу и окружению, иначе показывает все.

Параметры:
  -k, --key TEXT                  Фильтрует результат по ключу.
  -e, --env TEXT                  Фильтрует результат по среде.
  -f, --format [yaml|json|json-compact]
                                  Выходной формат.
  -d, --descending                Установить порядок загрузки истории 'last-first'
  --help                          Показать это сообщение и выйти.
```

Пример использования:

```yaml
# dynaconf -i app.settings inspect -k foo -f yaml
header:
  filters:
    env: None
    key: foo
    history_ordering: ascending
  active_value: from_environ
history:
- loader: yaml
  identifier: file_a.yaml
  env: default
  merged: false
  value:
    FOO: from_yaml
- loader: env_global
  identifier: unique
  env: global
  merged: false
  value:
    FOO: from_environ
    BAR: environ_only
```

Для сохранения в файл используйте обычные методы перенаправления потока:

```bash
$ dynaconf -i app.settings inspect -k foo -f yaml > dump.yaml
```

### dynaconf get

Получить необработанное значение для одного ключа

```
Использование: dynaconf get [OPTIONS] KEY

  Возвращает необработанное значение для ключа настроек.

  Если результатом является словарь, список или кортеж, он печатается
  как допустимая строка json.

Параметры:
  -d, --default TEXT  Значение по умолчанию, если настройки не существуют
  -e, --env TEXT      Фильтрует env, чтобы получить значения
  -u, --unparse       Парсит данные, добавив маркеры, такие как @none, @int и т.д...
  --help              Показать это сообщение и выйти.
```

Пример:

```bash
export FOO=$(dynaconf get DATABASE_NAME -d 'default')
```

Если ключ не существует и значение по умолчанию не указано, он завершится с кодом 1.

### dynaconf list

Перечислите все определенные параметры и при необходимости экспортируйте в файл json.

```
Использование: dynaconf list [OPTIONS]

  Список пользовательских настроек или всех (включая внутренние конфигурации).

  По умолчанию показывает только определенные пользователем.
  Если передан `--all`, он также показывает внутренние переменные dynaconf.

Параметры:
  -e, --env TEXT     Фильтрует env, чтобы получить значения
  -k, --key TEXT     Фильтрует отдельный ключ
  -m, --more         Разбиение на страницы стиля больше|меньше
  -l, --loader TEXT  идентификатор загрузчика для фильтрации, например: toml|yaml
  -a, --all          показать внутренние настройки dynaconf?
  -o, --output FILE  Путь к файлу для записи перечисленных значений в виде json
  --output-flat      Выходной файл плоский (не включает имя [env])
  --help             Показать это сообщение и выйти.
```

#### Экспорт текущей среды в виде файла

```bash
dynaconf list -o path/to/file.yaml
```

Приведенная выше команда экспортирует все элементы, показанные `dynaconf list`, в нужный формат, который определяется расширением файла `-o`, поддерживаемые форматы `yaml, toml, ini, json, py`.

При использовании `py` вам может понадобиться плоский вывод (без вложения внутри ключа env)

```bash
dynaconf list -o path/to/file.py --output-flat
```

### dynaconf write
