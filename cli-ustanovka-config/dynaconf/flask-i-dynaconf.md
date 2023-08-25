# Flask и dynaconf

Dynaconf обеспечивает замену `app.config`.

Поскольку **Flask** поощряет композицию, переопределяя атрибут **config\_class**, это расширение следует [шаблонам Flask](http://flask.pocoo.org/docs/0.12/patterns/subclassing/) и превращает `app.config` вашего Flask в экземпляр dynaconf.

## Инициализация расширения

Инициализируйте расширение **FlaskDynaconf** в своем приложении **app**.

```python
from flask import Flask
from dynaconf import FlaskDynaconf

app = Flask(__name__)
FlaskDynaconf(app)
```

Вы также можете использовать **init\_app**.

## Используйте переменные среды FLASK\_

Тогда `app.config` будет работать как экземпляр `dynaconf.settings`, а **FLASK\_** будет глобальным префиксом для экспорта переменных среды.

Пример:

```bash
export FLASK_DEBUG=true              # app.config.DEBUG
export FLASK_INTVALUE=1              # app.config['INTVALUE']
export FLASK_MAIL_SERVER='host.com'  # app.config.get('MAIL_SERVER')
```

Вы также можете использовать пользовательские переменные среды, как в классе Dynaconf по умолчанию, например:

Пример:

```python
from flask import Flask
from dynaconf import FlaskDynaconf

app = Flask(__name__)
FlaskDynaconf(app, envvar_prefix="PEANUT")
```

Теперь вы можете объявить свои переменные с помощью собственного префикса, и он обычно будет доступен в встроенной конфигурации Flask `app.config`.

```python
export PEANUT_DEBUG=true              # app.config.DEBUG
export PEANUT_INTVALUE=1              # app.config['INTVALUE']
export PEANUT_MAIL_SERVER='host.com'  # app.config.get('MAIL_SERVER')
```

{% hint style="info" %}
Версия 3.1.7 наоборот учитывала регистр при определении **ENVVAR\_PREFIX** и принимала только **kwargs** в верхнем регистре (в отличие от `Dynaconf(envvar_prefix)`). Начиная с версии XX.X.X, **kwargs** должны быть нечувствительны к регистру, чтобы улучшить согласованность между расширениями **Dynaconf** и **Flask/Django**, сохраняя при этом обратную совместимость.
{% endhint %}

## Файлы настроек

Вы также можете иметь файлы настроек для вашего приложения **Flask**: в корневом каталоге (тот же, где вы выполняете запуск `flask run`) поместите файлы `settings.toml` и `.secrets.toml`, а затем определите свои среды `[default]`, `[development]` и `[production]`.

Для переключения рабочей среды можно использовать переменную **FLASK\_ENV**, поэтому `FLASK_ENV=development` для работы в режиме разработки **development** или `FLASK_ENV=production` для переключения в производственный режим **production**.

{% hint style="info" %}
**ВАЖНО**: Чтобы использовать `$ dynaconf` CLI, необходимо определить **FLASK\_APP**.
{% endhint %}

Если вы не хотите вручную создавать файлы конфигурации, взгляните на [CLI](cli-dynaconf.md).

## Динамическая загрузка расширений Flask

Вы можете указать Dynaconf динамически загружать расширения **Flask**, если расширения соответствуют шаблонам расширений Flask.

Единственное требование состоит в том, что расширение должно быть вызываемым объектом **callable**, который принимает приложение **app** в качестве первого аргумента. например: `flask_admin:Admin` или `custom_extension.module:instance.init_app` и, конечно же, для импорта расширение должно находиться в пространстве имен Python.

Для инициализированных расширений просто используйте ссылку на объект [точки входа](https://packaging.python.org/specifications/entry-points/), например: `"flask_admin:Admin"` или `"extension.module:instance.init_app"`.

наличие `settings.toml`

```toml
[default]
EXTENSIONS = [
  "flask_admin:Admin",
  "flask_bootstrap:Bootstrap",
  "custom_extension.module:init_app"
]
```

Учитывая `app.py`, например:

```python
from flask import Flask
from dynaconf import FlaskDynaconf

app = Flask(__name__)
flask_dynaconf = FlaskDynaconf(app, extensions_list="EXTENSIONS")
```

Вышеупомянутое немедленно загрузит все расширения Flask, перечисленные в ключе **EXTENSIONS** в настройках.

Вы также можете загружать его лениво.

```python
# в любой момент запуска вашего приложения
app.config.load_extensions()
```

При необходимости вы можете передать `load_extensions(key="OTHER_NAME")`, указывающий на ваш список расширений.

Также можно использовать переменные среды для установки загружаемых расширений.

```bash
# .env
export FLASK_EXTENSIONS="['flask_admin:Admin']"
```

Расширения будут загружены по порядку.

## Расширения разработки

Также возможно иметь расширения, которые загружаются только для среды разработки.
