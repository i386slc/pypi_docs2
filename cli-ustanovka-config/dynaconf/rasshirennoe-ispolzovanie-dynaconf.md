# Расширенное использование dynaconf

## Хуки

> НОВОЕ в версии 3.1.6

Хуки полезны, когда вам нужно условно загрузить данные, которые зависят от ранее загруженных настроек.

Прежде чем углубляться в хуки, давайте разберемся, почему они полезны.

Представьте, что ваше приложение использует 2 модуля настроек: `plugin_settings.py` и `settings.py`.

`plugin_settings.py`

```python
DEBUG = True
```

`settings.py`

```python
if DEBUG:
    # сделать что-то
```

`app.py`

```python
from dynaconf import Dynaconf
settings = Dynaconf(
    preload=['plugin_settings.py'],
    settings_file="settings.py"
)
```

Приведенный выше код завершится ошибкой `NameError: name 'DEBUG' is not defined on settings.py`, что происходит потому, что `plugin_settings.py` загружается до `settings.py`, но также до полной загрузки `app.py`.

### Хуки для решения

Хук — это, по сути, функция, которая принимает необязательный позиционный аргумент **settings**, доступный только для чтения, и возвращает данные для объединения с объектом **Settings**.

Есть два способа добавления хуков: из специальных модулей или непосредственно в экземпляре Dynaconf.

### Модульный подход

Используя модульный подход, вы можете создать файл `dynaconf_hooks.py` по тому же пути, что и любой файл настроек. Затем хуки из этого модуля будут загружены после обычного процесса загрузки.

```python
# dynaconf_hooks.py

def post(settings):
    data = {"dynaconf_merge": True}
    if settings.DEBUG:
        data["DATABASE_URL"] = "sqlite://"
    return data
```

Dynaconf выполнит функцию **post** и объединит возвращенные данные с существующими настройками.

### Подход с экземплярами объекта

При использовании экземплярного подхода просто добавьте функцию-перехватчик в аргумент инициализации Dynaconf **post\_hook**. Он принимает один вызываемый объект `Callable` или список вызываемых объектов `Callable`.

```python
def hook_function(settings):
    data = {"dynaconf_merge": True}
    if settings.DEBUG:
        data["DATABASE_URL"] = "sqlite://"
    return data

settings = Dynaconf(post_hooks=hook_function)
```

Вы также можете настроить слияние индивидуально для каждой переменной настроек, как показано в [документации по слиянию](sliyanie-merging.md).

## Анализ истории

> НОВОЕ в версии 3.2.0

## `inspect_settings` <a href="#inspect_settings" id="inspect_settings"></a>

## Создание новых загрузчиков
