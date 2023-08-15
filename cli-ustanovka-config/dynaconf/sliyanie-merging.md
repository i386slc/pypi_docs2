# Слияние (merging)

## Обзор

Dynaconf предоставляет глобальные и локальные инструменты для управления тем, будут ли значения конфликтующих настроек (имеющих один и тот же ключ) объединяться или переопределять друг друга.

По умолчанию ничего не объединяется. Кроме того, объединять можно только типы контейнеров (**list** и **dict**). Неконтейнерные типы, такие как **str** и **int**, всегда будут переопределять предыдущее значение (если вы хотите объединить их, оберните их списком или словарем).

Вы можете глобально включить стратегию слияния, установив для параметра [merge\_enable](konfiguraciya-dynaconf.md#merge\_enabled) значение `True` (через настройки экземпляра или envvars).

## Объединение существующих структур данных

Если в ваших настройках есть существующие переменные типов **list** или **dict**, и вы хотите выполнить слияние **merge** вместо переопределения **override**, тогда **dynaconf\_merge** и **dynaconf\_merge\_unique** могут пометить эту переменную как кандидата на слияние.

Для значения **словаря**:

Ваш основной файл настроек (например, `settings.toml`) имеет существующую настройку **DATABASE** в словаре - `[default]` env.

Теперь вы хотите внести свой вклад в тот же ключ **DATABASE**, добавив новые ключи, поэтому вы можете использовать **dynaconf\_merge** в конце вашего словаря:

В конкретных `[envs]`

```ini
[default]
database = {host="server.com", user="default"}

[development]
database = {user="dev_user", dynaconf_merge=true}

[production]
database = {user="prod_user", dynaconf_merge=true}
```

также разрешен альтернативный короткий формат

```ini
[default]
database = {host="server.com", user="default"}

[development.database]
dynaconf_merge = {user="dev_user"}

[production.database]
dynaconf_merge = {user="prod_user"}
```

В переменной окружения:

Использование отметки **@merge**

```bash
# Envvar в формате Toml
export DYNACONF_DATABASE='@merge {password=1234}'
```

или краткий формат @merge маркирования

```bash
# Envvar в формате Toml
export DYNACONF_DATABASE='@merge password=1234'
```

Также можно использовать вложенный обход **dunder**, например:

```bash
export DYNACONF_DATABASE__password=1234
export DYNACONF_DATABASE__user=admin
export DYNACONF_DATABASE__ARGS__timeout=30
export DYNACONF_DATABASE__ARGS__retries=5
```

Каждый `__` анализируется как уровень, проходящий через ключи словаря. Читайте больше в [переменных окружения](peremennye-okruzheniya-env-vars.md)

Таким образом, результатом приведенного выше будет:

```python
DATABASE = {
    'password': 1234, 'user': 'admin',
    'ARGS': {'timeout': 30, 'retries': 5}
}
```

{% hint style="info" %}
**ВАЖНО**: клавиши нижнего регистра соблюдаются только в системах \*nix. К сожалению, переменные среды Windows нечувствительны к регистру, и Python читает их как все буквы верхнего регистра, а это означает, что если вы работаете в Windows, словарь может содержать только ключи верхнего регистра.
{% endhint %}

Вы также можете экспортировать словарь **toml**.

```bash
# Envvar в формате Toml
export DYNACONF_DATABASE='{password=1234, dynaconf_merge=true}'
```

Или в дополнительном файле (например, `settings.yaml`, `.secrets.yaml` и т. д.), используя токен **dynaconf\_merge**:

```yaml
default:
  database:
    password: 1234
    dynaconf_merge: true
```

или

```yaml
default:
  database:
    dynaconf_merge:
      password: 1234
```

Токен **dynaconf\_merge** пометит этот объект как объединенный с существующими значениями (разумеется, ключ **dynaconf\_merge** не будет добавлен в окончательные настройки, это просто пометка)

Конечный результат будет в среде `[development]`:

```python
settings.DATABASE == {'host': 'server.com', 'user': 'dev_user', 'password': 1234}
```

То же самое можно применить к спискам:

`settings.toml`

```toml
[default]
plugins = ["core"]

[development]
plugins = ["debug_toolbar", "dynaconf_merge"]
```

или

```toml
[default]
plugins = ["core"]

[development.plugins]
dynaconf_merge = ["debug_toolbar"]
```

И в переменной окружения

используя токен **@merge**

```bash
export DYNACONF_PLUGINS='@merge ["ci_plugin"]'
```

или короткая версия

```bash
export DYNACONF_PLUGINS='@merge ci_plugin'
```

также поддерживаются значения, разделенные запятыми:

```bash
export DYNACONF_PLUGINS='@merge ci_plugin,other_plugin'
```

или явно

```bash
export DYNACONF_PLUGINS='["ci_plugin", "dynaconf_merge"]'
```

Тогда конечный результат `[developmant]`:

```python
settings.PLUGINS == ["ci_plugin", "debug_toolbar", "core"]
```

Если ваше значение является словарем:

```bash
export DYNACONF_DATA="@merge {foo='bar'}"

# или коротко

export DYNACONF_DATA="@merge foo=bar"
```

### Избегайте дублирования в списках

**dynaconf\_merge\_unique** — это токен, когда вы хотите избежать дублирования в списке.

Пример:

```toml
[default]
scripts = ['install.sh', 'deploy.sh']

[development]
scripts = ['dev.sh', 'test.sh', 'deploy.sh', 'dynaconf_merge_unique']
```

```bash
export DYNACONF_SCRIPTS='["deploy.sh", "run.sh", "dynaconf_merge_unique"]'
```

Конечным результатом для `[development]` будет:

```python
settings.SCRIPTS == ['install.sh', 'dev.sh', 'test.sh', 'deploy.sh', 'run.sh']
```

{% hint style="info" %}
Обратите внимание, что `deploy.sh` устанавливается 3 раза, но не повторяется в окончательных настройках. **Также обратите внимание**, что он позволяет избежать дублирования, но переопределяет порядок элементов.
{% endhint %}

## Локальные файлы конфигурации и слияние с существующими данными

> Новое в версии `2.2.0`

Эта функция полезна для поддержки общего набора файлов конфигурации для команды, но при этом позволяет выполнять локальную настройку.

Любой файл, соответствующий поиску glob `*.local.*`, будет прочитан в конце порядка загрузки файлов. Таким образом, локальные файлы настроек могут быть, например, не привязаны к репозиторию с контролем версий. (например, добавьте `**/*.local*` в свой `.gitignore`)

Итак, если у вас есть `settings.toml`, Dynaconf загрузит его и, в конце концов, также попытается загрузить файл с именем `settings.local.toml`, если он существует. И то же самое относится ко всем другим поддерживаемым расширениям `settings.local.{py,json,yaml,toml,ini,cfg}`

Пример:

```toml
# settings.toml        # <-- 1-й загруженный
[default]
colors = ["green", "blue"]
parameters = {enabled=true, number=42}

# .secrets.toml        # <-- 2-й загруженный
# (переопределяет предыдущие существующие переменные)
[default]
password = 1234

# settings.local.toml  # <-- 3-й загруженный
# (переопределяет предыдущие существующие переменные)
[default]
colors = ["pink"]
parameters = {enabled=false}
password = 9999
```

Таким образом, с приведенным выше, например, значения будут:

```python
settings.COLORS == ["pink"]
settings.PARAMETERS == {"enabled": False}
settings.PASSWORD == 9999
```

Для каждого загружаемого файла dynaconf переопределяет **override** предыдущие существующие ключи, поэтому, если вы хотите добавить новые значения к существующим переменным, вы можете использовать 3 стратегии.

### Отметить локальный файл как полностью объединенный

> Новое в версии `2.2.0`

```toml
# settings.local.toml
dynaconf_merge = true
[default]
colors = ["pink"]
parameters = {enabled=false}
```

Если добавить **dynaconf\_merge** в верхний корень файла, весь файл будет помечен для слияния.

Затем значения будут обновлены в существующих структурах данных.

```python
settings.COLORS == ["pink", "green", "blue"]
settings.PARAMETERS == {"enabled": False, "number": 42}
settings.PASSWORD == 9999
```

Вы также можете пометить одну среду **env**, например `[development]`, для слияния.

```toml
# settings.local.toml
[development]
dynaconf_merge = true
colors = ["pink"]
parameters = {enabled=false}
```

### Токен слияния dynaconf

```toml
# settings.local.toml
[default]
colors = ["pink", "dynaconf_merge"]
parameters = {enabled=false, dynaconf_merge=true}
```

При добавлении **dynaconf\_merge** в список list или словарь dict они будут помечены как кандидаты на слияние.

Затем значения будут обновлены в существующих структурах данных.

```python
settings.COLORS == ["pink", "green", "blue"]
settings.PARAMETERS == {"enabled": False, "number": 42}
settings.PASSWORD == 9999
```

> Новое в версии `2.2.0`

И это также работает с **dynaconf\_merge** в качестве ключей dict, содержащих значение для слияния.

```toml
# settings.local.toml
[default.colors]
# <-- значение ["pink"] будет объединено с существующими цветами
dynaconf_merge = ["pink"]

[default.parameters]
dynaconf_merge = {enabled=false}
```

### Dunder слияние для вложенных структур

Для вложенных структур рекомендуется использовать слияние **dunder**, потому что его легче читать, а также нет ограничений с точки зрения уровней вложенности.

```toml
# settings.local.yaml
[default]
parameters__enabled = false
```

Использование `__` для обозначения вложенного уровня гарантирует, что ключ будет объединен с существующими значениями, подробнее см. в разделе об [объединении существующих значений](sliyanie-merging.md#obedinenie-sushestvuyushikh-struktur-dannykh).

## Вложенные ключи в словари через переменные окружения.
