# Точки входа (entry poynts)

Точки входа — это тип метаданных, которые могут предоставляться пакетами при установке. Это очень полезная функция экосистемы Python, которая особенно удобна в двух сценариях:

1. Пакет хотел бы предоставить команды для запуска на терминале. Эта функциональность известна как **консольные** **сценарии**. Команда также может открыть графический интерфейс, и в этом случае она называется **сценарием графического интерфейса**. Примером консольного сценария является сценарий, предоставляемый пакетом **pip**, который позволяет вам запускать в терминале такие команды, как `pip install`.
2. Пакет хотел бы включить настройку своих функций с помощью **плагинов**. Например, тестовая среда **pytest** позволяет выполнять настройку через точку входа **pytest11**, а инструмент подсветки синтаксиса **pygments** позволяет указывать дополнительные стили с помощью точки входа `pygments.styles`.

## Консольные скрипты

Начнем с консольных скриптов. Сначала рассмотрим пример без точек входа. Представьте себе пакет, определенный следующим образом:

```
project_root_directory
├── pyproject.toml        # и/или setup.cfg, setup.py
└── src
    └── timmins
        ├── __init__.py
        └── ...
```

с `__init.py__` как:

```python
def hello_world():
    print("Hello world")
```

Теперь предположим, что мы хотели бы обеспечить какой-то способ выполнения функции `hello_world()` из командной строки. Один из способов сделать это — создать файл `src/timmins/__main__.py`, предоставляющий хук следующим образом:

```python
from . import hello_world

if __name__ == '__main__':
    hello_world()
```

Затем, после установки пакета **timmins**, мы можем вызвать функцию `hello_world()` следующим образом через модуль [runpy](https://docs.python.org/3/library/runpy.html):

```bash
$ python -m timmins
Hello world
```

Вместо этого подхода с использованием `__main__.py` вы также можете создать удобный исполняемый файл CLI, который можно вызывать напрямую без `python -m`. В приведенном выше примере, чтобы создать команду **hello-world**, которая вызывает `timmins.hello_world`, добавьте в свою конфигурацию точку входа сценария консоли:

#### pyproject.toml

```toml
[project.scripts]
hello-world = "timmins:hello_world"
```

#### setup.cfg

```ini
[options.entry_points]
console_scripts =
    hello-world = timmins:hello_world
```

#### setup.py

```python
from setuptools import setup

setup(
    # ...,
    entry_points={
        'console_scripts': [
            'hello-world = timmins:hello_world',
        ]
    }
)
```

После установки пакета пользователь может вызвать эту функцию, просто вызвав **hello-world** в командной строке:

```bash
$ hello-world
Hello world
```

Обратите внимание, что любая функция, используемая в качестве консольного скрипта, например `hello_world()` в этом примере, не должна принимать никаких аргументов. Если ваша функция требует какого-либо ввода от пользователя, вы можете использовать обычные утилиты анализа аргументов командной строки, такие как [argparse](https://docs.python.org/3/library/argparse.html#module-argparse), внутри тела функции для анализа ввода пользователя, переданного через [sys.argv](https://docs.python.org/3/library/sys.html#sys.argv).

Вы могли заметить, что мы использовали специальный синтаксис для указания функции, которая должна быть вызвана консольным сценарием, т. е. мы написали `timmins:hello_world` с двоеточием `:`, разделяя имя пакета и имя функции. Полная спецификация этого синтаксиса обсуждается в [последнем разделе](tochki-vkhoda-entry-poynts.md#sintaksis-tochek-vkhoda) этого документа, и его можно использовать для указания функции, расположенной в любом месте вашего пакета, а не только в `__init__.py`.

## Скрипты графического интерфейса

В дополнение к **console\_scripts**, Setuptools поддерживает **gui\_scripts**, которые запускают приложение с графическим интерфейсом без запуска в окне терминала.

Например, если у нас есть проект с той же структурой каталогов, что и раньше, с файлом `__init__.py`, содержащим следующее:

```python
import PySimpleGUI as sg

def hello_world():
    sg.Window(title="Hello world", layout=[[]], margins=(100, 50)).read()
```

Затем мы можем добавить точку входа GUI-скрипта:

#### pyproject.toml

```toml
[project.gui-scripts]
hello-world = "timmins:hello_world"
```

#### setup.cfg

```ini
[options.entry_points]
gui_scripts =
    hello-world = timmins:hello_world
```

#### setup.py

```python
from setuptools import setup

setup(
    # ...,
    entry_points={
        'gui_scripts': [
            'hello-world = timmins:hello_world',
        ]
    }
)
```

{% hint style="info" %}
Чтобы иметь возможность импортировать **PySimpleGUI**, вам нужно добавить **pysimplegui** в зависимости вашего пакета. Дополнительные сведения см. в разделе [Управление зависимостями в Setuptools](upravlenie-zavisimostyami-v-setuptools.md).
{% endhint %}

Теперь запускаю:

```bash
$ hello-world
```

откроется небольшое окно приложения с заголовком «Hello world».

Обратите внимание, что, как и в случае консольных сценариев, любая функция, используемая в качестве сценария графического интерфейса, не должна принимать никаких аргументов, и любой ввод пользователя может быть проанализирован в теле функции. Сценарии с графическим интерфейсом также используют тот же синтаксис (который будет рассмотрен в [последнем разделе](tochki-vkhoda-entry-poynts.md#sintaksis-tochek-vkhoda)) для указания вызываемой функции.

{% hint style="info" %}
Разница между **console\_scripts** и **gui\_scripts** влияет только на системы Windows. [\[1\]](tochki-vkhoda-entry-poynts.md#1) **console\_scripts** завернуты в исполняемый файл консоли, поэтому они присоединены к консоли и могут использовать `sys.stdin`, `sys.stdout` и `sys.stderr` для ввода и вывода. **gui\_scripts** завернуты в исполняемый файл с графическим интерфейсом, поэтому их можно запускать без консоли, но они не могут использовать стандартные потоки, если код приложения не перенаправляет их. Другие платформы не имеют такого различия.
{% endhint %}

{% hint style="info" %}
Сценарии консоли и графического интерфейса работают, потому что за кулисами установщики, такие как [pip](https://pypi.org/project/pip), создают сценарии-оболочки вокруг вызываемых функций. Например, точка входа **hello-world** в двух приведенных выше примерах создаст команду **hello-world**, запускающую такой скрипт: [\[1\]](tochki-vkhoda-entry-poynts.md#1)

```python
import sys
from timmins import hello_world
sys.exit(hello_world())
```
{% endhint %}

## Рекламное поведение

Сценарии консоли/графического пользовательского интерфейса — это одно из применений более общей концепции точек входа. Точки входа в более общем смысле позволяют упаковщику объявлять поведение для обнаружения другими библиотеками и приложениями. Эта функция обеспечивает функциональность, подобную "plug-in", когда одна библиотека запрашивает точки входа, а любое количество других библиотек предоставляет эти точки входа.

Хороший пример такого поведения подключаемого модуля можно увидеть в [подключаемых модулях pytest](https://docs.pytest.org/en/latest/writing\_plugins.html), где **pytest** — это тестовая среда, которая позволяет другим библиотекам расширять или изменять свою функциональность через точку входа **pytest11**.

Сценарии консоли/графического интерфейса пользователя работают аналогично, когда библиотеки рекламируют свои команды, а инструменты, такие как **pip**, создают сценарии-оболочки, которые вызывают эти команды.

## Точки входа для плагинов

Давайте рассмотрим простой пример, чтобы понять, как мы можем реализовать точки входа, соответствующие плагинам. Скажем, у нас есть пакет **timmins** со следующей структурой каталогов:

```
timmins
├── pyproject.toml        # и/или setup.cfg, setup.py
└── src
    └── timmins
        └── __init__.py
```

а в `src/timmins/__init__.py` у нас есть следующий код:

```python
def hello_world():
    print('Hello world')
```

По сути, мы определили функцию `hello_world()`, которая будет печатать текст «Hello world». Теперь предположим, что мы хотим напечатать текст «Hello world» разными способами. Текущая функция просто печатает текст как есть — скажем, нам нужен другой стиль, в котором текст заключен в восклицательные знаки:

```bash
!!! Hello world !!!
```

Давайте посмотрим, как это можно сделать с помощью плагинов. Во-первых, давайте отделим стиль печати текста от самого текста. Другими словами, мы можем изменить код в `src/timmins/__init__.py` примерно так:

```python
def display(text):
    print(text)

def hello_world():
    display('Hello world')
```

Здесь функция `display()` управляет стилем печати текста, а функция `hello_world()` вызывает функцию `display()` для печати текста «Hello world».

Прямо сейчас функция `display()` просто печатает текст как есть. Чтобы иметь возможность настроить его, мы можем сделать следующее. Давайте введем новую группу (_group_) точек входа с именем `timmins.display` и ожидаем, что пакеты плагинов, реализующие эту точку входа, будут предоставлять функцию, подобную `display()`. Затем, чтобы иметь возможность автоматически обнаруживать пакеты плагинов, которые реализуют эту точку входа, мы можем использовать модуль `importlib.metadata` следующим образом:

```python
from importlib.metadata import entry_points
display_eps = entry_points(group='timmins.display')
```

{% hint style="info" %}
Каждый объект `importlib.metadata.EntryPoint` — это объект, содержащий **name**, **group** и **value**. Например, после настройки пакета плагина, как описано ниже, **display\_eps** в приведенном выше коде будет выглядеть так: [\[2\]](tochki-vkhoda-entry-poynts.md#2)

```python
(
    EntryPoint(
        name='excl',
        value='timmins_plugin_fancy:excl_display',
        group='timmins.display'
    ),
    ...,
)
```
{% endhint %}

**display\_eps** теперь будет списком объектов **EntryPoint**, каждый из которых ссылается на функции, подобные `display()`, определенные одним или несколькими установленными пакетами плагинов. Затем, чтобы импортировать конкретную функцию, подобную `display()` — давайте выберем ту, которая соответствует первой обнаруженной точке входа — мы можем использовать метод `load()` следующим образом:

```python
display = display_eps[0].load()
```

Наконец, разумным поведением было бы то, что если мы не можем найти какие-либо пакеты плагинов, настраивающие функцию `display()`, мы должны вернуться к нашей реализации по умолчанию, которая печатает текст как есть. С учетом этого поведения код в `src/timmins/__init__.py`, наконец, становится таким:

```python
from importlib.metadata import entry_points

display_eps = entry_points(group='timmins.display')

try:
    display = display_eps[0].load()
except IndexError:
    def display(text):
        print(text)

def hello_world():
    display('Hello world')
```

На этом установка на стороне **timmins** заканчивается. Далее нам нужно реализовать плагин, реализующий точку входа `timmins.display`. Давайте назовем этот плагин **timmins-plugin-fancy** и настроим его со следующей структурой каталогов:

```bash
timmins-plugin-fancy
├── pyproject.toml        # и/или setup.cfg, setup.py
└── src
    └── timmins_plugin_fancy
        └── __init__.py
```

А затем внутри `src/timmins_plugin_fancy/__init__.py` мы можем поместить функцию с именем `excl_display()`, которая печатает заданный текст, окруженный восклицательными знаками:

```python
def excl_display(text):
    print('!!!', text, '!!!')
```

Это функция, подобная `display()`, которую мы собираемся добавить в пакет **timmins**. Мы можем сделать это, добавив следующее в конфигурацию **timmins-plugin-fancy**:

#### pyproject.toml

```toml
# Обратите внимание на кавычки вокруг timmins.display, чтобы экранировать точки .
[project.entry-points."timmins.display"]
excl = "timmins_plugin_fancy:excl_display"
```

#### setup.cfg

```ini
[options.entry_points]
timmins.display =
    excl = timmins_plugin_fancy:excl_display
```

#### setup.py

```python
from setuptools import setup

setup(
    # ...,
    entry_points = {
        'timmins.display': [
            'excl = timmins_plugin_fancy:excl_display'
        ]
    }
)
```

По сути, в этой конфигурации указано, что мы являемся точкой входа в группу `timmins.display`. Точка входа называется **excl** и относится к функции **excl\_display**, определенной пакетом **timmins-plugin-fancy**.

Теперь, если мы установим и **timmins**, и **timmins-plugin-fancy**, мы должны получить следующее:

```python
>>> from timmins import hello_world

>>> hello_world()
!!! Hello world !!!
```

тогда как если мы установим только **timmins**, а не **timmins-plugin-fancy**, мы должны получить следующее:

```python
>>> from timmins import hello_world

>>> hello_world()
Hello world
```

Поэтому наш плагин работает.

Наш плагин также мог бы определить несколько точек входа в группе `timmins.display`. Например, в `src/timmins_plugin_fancy/__init__.py` у нас могут быть две функции, подобные `display()`, следующим образом:

```python
def excl_display(text):
    print('!!!', text, '!!!')

def lined_display(text):
    print(''.join(['-' for _ in text]))
    print(text)
    print(''.join(['-' for _ in text]))
```

Затем конфигурация **timmins-plugin-fancy** изменится на:

#### pyproject.toml

```toml
[project.entry-points."timmins.display"]
excl = "timmins_plugin_fancy:excl_display"
lined = "timmins_plugin_fancy:lined_display"
```

#### setup.cfg

```ini
[options.entry_points]
timmins.display =
    excl = timmins_plugin_fancy:excl_display
    lined = timmins_plugin_fancy:lined_display
```

#### setup.py

```python
from setuptools import setup

setup(
    # ...,
    entry_points = {
        'timmins.display': [
            'excl = timmins_plugin_fancy:excl_display',
            'lined = timmins_plugin_fancy:lined_display',
        ]
    }
)
```

Что касается **timmins**, мы также можем использовать другую стратегию загрузки точек входа. Например, мы можем искать определенный стиль отображения:

```python
display_eps = entry_points(group='timmins.display')
try:
    display = display_eps['lined'].load()
except KeyError:
    # если 'lined' дисплей недоступен, используйте что-то другое
    ...
```

Или мы также можем загрузить все плагины в данной группе. Хотя это может быть не очень полезно в нашем текущем примере, есть несколько сценариев, в которых это полезно:

```python
display_eps = entry_points(group='timmins.display')
for ep in display_eps:
    display = ep.load()
    # сделать что-нибудь с дисплеем
    ...
```

Еще один момент заключается в том, что в этом конкретном примере мы использовали плагины для настройки поведения функции (`display()`). В общем, мы можем использовать точки входа, чтобы плагины могли настраивать поведение не только функций, но и целых классов и модулей. Это отличается от сценария консоли/графического интерфейса, где точки входа могут ссылаться только на функции. Синтаксис, используемый для указания точек входа, остается таким же, как и для консольных/графических сценариев, и обсуждается в [последнем разделе](tochki-vkhoda-entry-poynts.md#sintaksis-tochek-vkhoda).

{% hint style="success" %}
**СОВЕТ**

Рекомендуемым подходом для загрузки и импорта точек входа является модуль [importlib.metadata](https://docs.python.org/3/library/importlib.metadata.html#module-importlib.metadata), который является частью стандартной библиотеки, начиная с Python 3.8. Для более старых версий Python следует использовать его бэкпорт [importlib\_metadata](https://pypi.org/project/importlib\_metadata). При использовании бэкпорта единственное изменение, которое необходимо сделать, это заменить `importlib.metadata` на **importlib\_metadata**, т.е.

```python
from importlib_metadata import entry_points
...
```
{% endhint %}

Таким образом, точки входа позволяют пакету открывать свои функции для настройки с помощью плагинов. Пакет, запрашивающий точки входа, не должен иметь какой-либо зависимости или предварительных знаний о подключаемых модулях, реализующих точки входа, а последующие пользователи могут создавать функциональные возможности, объединяя подключаемые модули, реализующие точки входа.

## Синтаксис точек входа

Синтаксис для точек входа определяется следующим образом:

```bash
<name> = <package_or_module>[:<object>[.<attr>[.<nested-attr>]*]]
```

Здесь квадратные скобки `[]` обозначают необязательность, а звездочка `*` обозначает повторение. **name** — это имя скрипта/точки входа, которую вы хотите создать, левая часть `:` — это пакет или модуль, содержащий объект, который вы хотите вызвать (представьте, что вы написали бы это в операторе импорта), и правая сторона - это объект, который вы хотите вызвать (например, функция).

Чтобы сделать этот синтаксис более понятным, рассмотрим следующие примеры:

#### Пакет или модуль

* Если вы предоставляете:

```bash
<name> = <package_or_module>
```

в качестве точки входа, где `<package_or_module>` может содержать `.` в случае подмодулей или подпакетов инструменты в экосистеме Python будут приблизительно интерпретировать это значение как:

```python
import <package_or_module>
parsed_value = <package_or_module>
```

#### Объект уровня модуля

* Если вы предоставляете:

```bash
<name> = <package_or_module>:<object>
```

где `<object>` не содержит `.`, это будет приблизительно интерпретироваться как:

```python
from <package_or_module> import <object>
parsed_value = <object>
```

#### Вложенный объект

* Если вы предоставляете:

```bash
<name> = <package_or_module>:<object>.<attr>.<nested_attr>
```

это будет примерно интерпретироваться как:

```python
from <package_or_module> import <object>
parsed_value = <object>.<attr>.<nested_attr>
```

В случае консольных/графических сценариев этот синтаксис можно использовать для указания функции, в то время как в общем случае точек входа, используемых для подключаемых модулей, его можно использовать для указания функции, класса или модуля.

#### \[ 1 ]

Ссылка: [https://packaging.python.org/en/latest/specifications/entry-points/#use-for-scripts](https://packaging.python.org/en/latest/specifications/entry-points/#use-for-scripts)

#### \[ 2 ]

Ссылка:[ https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/#using-package-metadata](https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/#using-package-metadata)
