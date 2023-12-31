# Быстрый старт setuptools

## Установка

Вы можете установить последнюю версию **setuptools** с помощью [pip](https://pypi.org/project/pip):

```bash
pip install --upgrade setuptools
```

Однако в большинстве случаев вам не нужно…

Вместо этого при создании новых пакетов Python рекомендуется использовать инструмент командной строки под названием [build](https://pypi.org/project/build). Этот инструмент автоматически загрузит **setuptools** и любые другие зависимости времени сборки, которые могут быть в вашем проекте. Вам просто нужно указать их в файле `pyproject.toml` в корне вашего пакета, как указано в [следующем разделе](bystryi-start-setuptools.md#osnovnoe-ispolzovanie).

Вы также можете [установить build](https://pypa-build.readthedocs.io/en/latest/installation.html) с помощью [pip](https://pypi.org/project/pip):

```bash
pip install --upgrade build
```

Это позволит вам запустить команду: `python -m build`.

{% hint style="success" %}
Обратите внимание, что некоторые операционные системы могут быть оснащены командами **python3** и **pip3** вместо **python** и **pip** (но они должны быть эквивалентны). Если в вашей системе нет **pip** или **pip3**, ознакомьтесь с [документацией по установке pip](https://pip.pypa.io/en/latest/installation/).
{% endhint %}

Каждый пакет Python должен предоставить `pyproject.toml` и указать серверную часть (систему сборки), которую он хочет использовать. Затем дистрибутив можно сгенерировать с помощью любого инструмента, который обеспечивает функциональность, подобную команде `build sdist`.

## Основное использование

При создании пакета Python необходимо предоставить файл `pyproject.toml`, содержащий раздел **build-system**, аналогичный приведенному ниже примеру:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

В этом разделе объявляются зависимости вашей системы сборки и какая библиотека будет использоваться для фактического создания упаковки.

{% hint style="info" %}
Исторически сложилось так, что в этой документации в списке **requires** указано ненужное **wheel**, и многие проекты все еще делают это. Это не рекомендуется. Серверная часть автоматически добавляет зависимость от **wheel**, когда это требуется, и ее явное указание делает ее ненужной для сборок исходного дистрибутива. Вы должны включать **wheel** в **requires** только в том случае, если вам нужен явный доступ к нему во время сборки (например, если вашему проекту нужен скрипт **setup.py**, который импортирует **wheel**).
{% endhint %}

В дополнение к указанию системы сборки вам также потребуется добавить некоторую информацию о пакете, такую как метаданные, содержимое, зависимости и т. д. Это можно сделать в том же файле `pyproject.toml` [\[2\]](bystryi-start-setuptools.md#2) или в отдельном: `setup. cfg` или `setup.py` [\[1\]](bystryi-start-setuptools.md#1).

В следующем примере демонстрируется минимальная конфигурация (которая предполагает, что проект зависит от [requests](https://pypi.org/project/requests) и [importlib-metadata](https://pypi.org/project/importlib-metadata) для возможности запуска):

#### pyproject.toml

```toml
[project]
name = "mypackage"
version = "0.0.1"
dependencies = [
    "requests",
    'importlib-metadata; python_version<"3.8"',
]
```

#### setup.cfg

```ini
[metadata]
name = mypackage
version = 0.0.1

[options]
install_requires =
    requests
    importlib-metadata; python_version < "3.8"
```

#### setup.py

```python
from setuptools import setup

setup(
    name='mypackage',
    version='0.0.1',
    install_requires=[
        'requests',
        'importlib-metadata; python_version == "3.8"',
    ],
)
```

Дополнительные сведения см. в разделе <mark style="color:purple;">Ключевые слова (keywords)</mark>.

Наконец, вам нужно будет организовать свой код Python, чтобы подготовить его к распространению, в нечто, похожее на следующее (необязательные файлы, отмеченные **#**):

```
mypackage
├── pyproject.toml  # и/или setup.cfg/setup.py (в зависимости от способа настройки)
|   # README.rst или README.md (хорошее описание вашего пакета)
|   # LICENCE (правильно подобранная лицензионная информация, например, MIT, BSD-3, GPL-3, MPL-2, др...)
└── mypackage
    ├── __init__.py
    └── ... (другие файлы Python)
```

Установив **build** в вашей системе, вы можете запустить:

```bash
python -m build
```

Теперь у вас есть готовый дистрибутив (например, файл `tar.gz` и файл `.whl` в каталоге **dist**), который вы можете [загрузить](https://twine.readthedocs.io/en/stable/index.html) в PyPI!

Конечно, прежде чем выпустить свой проект в [PyPI](https://pypi.org/), вы захотите добавить немного больше информации, чтобы помочь людям найти или узнать о вашем проекте. И, возможно, к тому времени ваш проект расширится и будет включать в себя несколько зависимостей и, возможно, несколько файлов данных и скриптов. В следующих нескольких разделах мы рассмотрим дополнительную, но важную информацию, которую необходимо указать для правильной упаковки вашего проекта.

{% hint style="info" %}
**Информация: Использование setup.py**

**Setuptools** предлагает первоклассную поддержку файлов **setup.py** в качестве механизма настройки.

Однако важно помнить, что запускать этот файл как скрипт (например, `python setup.py sdist`) настоятельно **не рекомендуется**, и что большинство интерфейсов командной строки **устарели** (или будут) объявлены устаревшими (например, `python setup.py install` , `python setup.py bdist_wininst`, …).

Мы также рекомендуем пользователям раскрывать как можно больше конфигурации более _декларативным_ способом через <mark style="color:purple;">pyproject.toml</mark> или <mark style="color:purple;">setup.cfg</mark> и сохранять минимальное количество **setup.py** только с динамическими частями (или даже полностью опустить его, если это применимо).

Дополнительные сведения см. в разделе <mark style="color:purple;">Почему не следует вызывать setup.py напрямую</mark>.
{% endhint %}

## Обзор

### Обнаружение пакетов

Для проектов с простой структурой каталогов **setuptools** должен иметь возможность автоматически обнаруживать все [пакеты](https://docs.python.org/3/glossary.html#term-package) и [пространства имен](https://docs.python.org/3/glossary.html#term-namespace). Однако сложные проекты могут включать в себя дополнительные папки и вспомогательные файлы, которые не обязательно следует распространять (или которые могут сбить с толку алгоритм автоматического обнаружения **setuptools**).

Таким образом, **setuptools** предоставляет удобный способ настроить, какие пакеты следует распространять и в каком каталоге их следует найти, как показано в примере ниже:

#### pyproject.toml (BETA)

```toml
# ...
[tool.setuptools.packages]
find = {}  # Сканировать каталог проекта с параметрами по умолчанию

# ИЛИ
[tool.setuptools.packages.find]
# Все следующие настройки являются необязательными:
where = ["src"]  # по умолчанию ["."]
include = ["mypackage*"]  # по умолчанию ["*"]
exclude = ["mypackage.tests*"]  # по умолчанию пусто
namespaces = false  # по умолчанию true
```

#### setup.cfg

```ini
[options]
packages = find: # ИЛИ `find_namespace:` если хотите использовать пространства имен

[options.packages.find]  # (всегда `find`, даже если `find_namespace:` был раньше)
# Этот раздел является необязательным, как и каждый из следующих параметров.:
where=src  # по умолчанию .
include=mypackage*  # по умолчанию *
exclude=mypackage.tests*  # по умолчанию пусто
```

#### setup.py

```python
from setuptools import find_packages  # или find_namespace_packages

setup(
    # ...
    packages=find_packages(
        # Все приведенные ниже аргументы ключевых слов являются необязательными.:
        where='src',  # по умолчанию '.'
        include=['mypackage*'],  # по умолчанию ['*']
        exclude=['mypackage.tests'],  # по умолчанию пусто
    ),
    # ...
)
```

Когда вы передаете указанную выше информацию, наряду с другой необходимой информацией, **setuptools** проходит через каталог, указанный в **where** (по умолчанию `.`), и фильтрует пакеты, которые он может найти, следуя шаблонам **include** (по умолчанию `*`), а затем удаляет те, которые соответствуют **exclude** (по умолчанию пусто) и возвращает список пакетов Python.

Дополнительные сведения и расширенные возможности использования см. в разделе <mark style="color:purple;">Обнаружение пакетов и Пакеты пространства имен</mark>.

{% hint style="success" %}
**Совет**

Начиная с версии 61.0.0, возможности автоматического обнаружения **setuptools** были улучшены для обнаружения популярных макетов проектов (таких как <mark style="color:purple;">flat-layout</mark> и <mark style="color:purple;">src-layout</mark>) без какой-либо специальной настройки. Ознакомьтесь с нашей справочной документацией для получения дополнительной информации, но имейте в виду, что эта функция по-прежнему считается бета-версией и может измениться в будущих выпусках.
{% endhint %}

### Точки входа (entry points) и автоматическое создание скрипта

**Setuptools** поддерживает автоматическое создание сценариев при установке, которые запускают код внутри вашего пакета, если вы укажете их в качестве <mark style="color:purple;">точек входа</mark>. Пример того, как эту функцию можно использовать в **pip**: она позволяет запускать такие команды, как `pip install`, вместо того, чтобы вводить `python -m pip install`.

В следующих примерах конфигурации показано, как это сделать:

#### pyproject.toml

```toml
[project.scripts]
cli-name = "mypkg.mymodule:some_func"
```

#### setup.cfg

```ini
[options.entry_points]
console_scripts =
    cli-name = mypkg.mymodule:some_func
```

#### setup.py

```python
setup(
    # ...
    entry_points={
        'console_scripts': [
            'cli-name = mypkg.mymodule:some_func',
        ]
    }
)
```

Когда этот проект будет установлен, будет создан исполняемый файл `cli-name`. `cli-name` будет вызывать функцию **some\_func** в файле `mypkg/mymodule.py` при вызове пользователем. Обратите внимание, что вы также можете использовать механизм **entry-points** для объявления компонентов между установленными пакетами и внедрения систем плагинов. Для получения подробной информации об использовании перейдите в раздел «<mark style="color:purple;">Точки входа</mark>».

### Управление зависимостями

Пакеты, созданные с помощью **setuptools**, могут указывать зависимости, которые будут автоматически устанавливаться при установке самого пакета. В приведенном ниже примере показано, как настроить такие зависимости:

#### pyproject.toml

```toml
[project]
# ...
dependencies = [
    "docutils",
    "requests <= 0.4",
]
# ...
```

#### setup.cfg

```ini
[options]
install_requires =
    docutils
    requests <= 0.4
```

#### setup.py

```python
setup(
    # ...
    install_requires=["docutils", "requests <= 0.4"],
    # ...
)
```

Каждая зависимость представлена строкой, которая может дополнительно содержать требования к версии (например, один из операторов `<`, `>`, `<=`, `>=`, `==` или `!=`, за которым следует идентификатор версии) и/или условные маркеры среды, например. `sys_platform == "win32"` (дополнительную информацию см. в разделе <mark style="color:purple;">Спецификаторы версии</mark>).

Когда ваш проект будет установлен, все еще не установленные зависимости будут найдены (через PyPI), загружены, построены (при необходимости) и установлены. Это, конечно, упрощенный сценарий. Вы также можете указать группы дополнительных зависимостей, которые не являются строго обязательными для работы вашего пакета, но которые обеспечат дополнительные функциональные возможности. Дополнительные сведения об использовании см. в разделе <mark style="color:purple;">Управление зависимостями в Setuptools</mark>.

### Включение файлов данных

**Setuptools** предлагает три способа указать файлы данных для включения в ваши пакеты. Для самого простого использования вы можете просто использовать ключевое слово **include\_package\_data**:

#### pyproject.toml (BETA)

```toml
[tool.setuptools]
include-package-data = true
# Это уже поведение по умолчанию,
# если вы используете pyproject.toml для настройки сборки.
# Вы можете отключить это с помощью `include-package-data = false`
```

#### setup.cfg

```ini
[options]
include_package_data = True
```

#### setup.py

```python
setup(
    # ...
    include_package_data=True,
    # ...
)
```

Это говорит **setuptools** установить любые файлы данных, которые он найдет в ваших пакетах. Файлы данных должны быть указаны через файл `MANIFEST.in` или автоматически добавлены подключаемым <mark style="color:purple;">модулем системы контроля версий</mark>. Дополнительные сведения см. в разделе <mark style="color:purple;">Поддержка файлов данных</mark>.

### Режим разработки

**setuptools** позволяет вам установить пакет без копирования каких-либо файлов в каталог вашего интерпретатора (например, каталог **site-packages**). Это позволяет вам модифицировать исходный код и заставить изменения вступить в силу без пересборки и переустановки. Вот как это сделать:

```bash
pip install --editable .
```

Дополнительную информацию см. в разделе <mark style="color:purple;">«Режим разработки» (он же «Редактируемые установки»)</mark>.

{% hint style="success" %}
**Совет**

До [pip v21.1](https://pip.pypa.io/en/latest/news/#v21-1) для совместимости с режимом разработки требовался сценарий **setup.py**. В поздних версиях **pip** в этом режиме могут быть установлены проекты без **setup.py**.

Если у вас есть версия **pip** старше v21.1 или вы используете другой инструмент, связанный с упаковкой, который не поддерживает [PEP 660](https://peps.python.org/pep-0660/), вам может потребоваться сохранить файл **setup.py** в файле в вашем репозитории, если вы хотите использовать редактируемые установки.

Достаточно простого скрипта, например:

```python
from setuptools import setup

setup()
```

Вы по-прежнему можете хранить всю конфигурацию в <mark style="color:purple;">pyproject.toml</mark> и/или <mark style="color:purple;">setup.cfg</mark>.
{% endhint %}

### Загрузка вашего пакета в PyPI

После создания файлов дистрибутива следующим шагом будет загрузка вашего дистрибутива, чтобы другие могли его использовать. Эта функциональность предоставляется [twine](https://pypi.org/project/twine) и задокументирована в [учебнике по упаковке Python](https://packaging.python.org/en/latest/tutorials/packaging-projects/).

### Переход с setup.py на setup.cfg

Чтобы избежать выполнения произвольных скриптов и стандартного кода, мы переходим на полноценный **setup.cfg**, чтобы объявлять информацию о вашем пакете вместо запуска **setup()**. Это неизбежно приводит к проблемам из-за другого синтаксиса. <mark style="color:purple;">Здесь</mark> мы даем краткое руководство, чтобы понять, как **setuptools** анализирует файл **setup.cfg**, чтобы упростить переход.

## Ресурсы по упаковке Python

Упаковка в Python может быть сложной и постоянно развивается. В [Руководстве пользователя по упаковке Python](https://packaging.python.org/) есть учебные пособия и актуальные справочные материалы, которые могут помочь вам, когда придет время распространять вашу работу.

## Примечания

#### \[ 1 ]

В новых проектах рекомендуется избегать конфигураций **setup.py** (помимо минимальной заглушки), когда пользовательские сценарии во время сборки не требуются. В этом документе приведены примеры, чтобы помочь людям, заинтересованным в поддержке или дополнении существующих пакетов, использующих **setup.py**. Обратите внимание, что вы по-прежнему можете оставить большую часть конфигурации декларативной в <mark style="color:purple;">setup.cfg</mark> или <mark style="color:purple;">pyproject.toml</mark> и использовать **setup.py** только для тех частей, которые не поддерживаются в этих файлах (например, расширения C). Смотрите [примечание](bystryi-start-setuptools.md#osnovnoe-ispolzovanie).

#### \[ 2 ]

Поддержка добавления параметров конфигурации сборки через таблицу `[tool.setuptools]` в файле `pyproject.toml` все еще находится на стадии **бета**-тестирования. См. раздел <mark style="color:purple;">Настройка инструментов настройки с помощью файлов pyproject.toml</mark>.
