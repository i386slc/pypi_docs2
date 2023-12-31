# Режим разработки (a.k.a. "Editable Installs")

При создании проекта Python разработчики обычно хотят внедрять и тестировать изменения итеративно, прежде чем вырезать релиз и подготовить архив дистрибутива.

В обычных обстоятельствах это может быть довольно громоздко и требует, чтобы разработчики манипулировали переменной среды **PYTHONPATH** или постоянно пересобирали и переустанавливали проект.

Чтобы облегчить итеративное исследование и экспериментирование, **setuptools** позволяет пользователям указать интерпретатору Python и его механизму импорта загружать разрабатываемый код непосредственно из папки проекта без необходимости копировать файлы в другое место на диске. Это означает, что изменения в исходном коде Python могут быть внесены немедленно, без необходимости новой установки.

Вы можете войти в этот режим разработки _"development mode"_, выполнив [редактируемую установку](https://pip.pypa.io/en/latest/topics/local-project-installs/) (_editable installation_) внутри виртуальной среды, используя флаг `pip -e/--editable`, как показано ниже:

```bash
$ cd your-python-project
$ python -m venv .venv
# Activate your environemt with:
#      `source .venv/bin/activate` on Unix/macOS
# or   `.venv\Scripts\activate` on Windows

$ pip install --editable .

# Теперь у вас есть доступ к вашему пакету, как если бы он был установлен в .venv
$ python -c "import your_python_project"
```

"_editable installation_" работает очень похоже на обычную установку с помощью `pip install .`, за исключением того, что она устанавливает только ваши зависимости пакета, метаданные и оболочки для <mark style="color:purple;">консольных скриптов и графического интерфейса</mark>. Под капотом **setuptools** попытается создать специальный файл `.pth` в целевом каталоге (обычно `site-packages`), который расширяет **PYTHONPATH**, или установит пользовательский [import hook](https://docs.python.org/3/reference/import.html).

Когда вы закончите с данной задачей разработки, вы можете просто удалить свой пакет (как вы обычно делаете с `pip uninstall <package_name>` ).

Обратите внимание, что по умолчанию редактируемая установка предоставляет как минимум все файлы, которые были бы доступны при обычной установке. Однако, в зависимости от организации файлов и каталогов в вашем проекте, он также может предоставлять в качестве побочного эффекта файлы, которые обычно недоступны. Это разрешено, поэтому вы можете итеративно создавать новые модули Python. Пожалуйста, ознакомьтесь со следующим разделом, если вы ищете другое поведение.

{% hint style="info" %}
**Виртуальная среда**

Вы можете думать о виртуальных средах как об «изолированных средах выполнения Python», которые позволяют пользователям устанавливать различные наборы библиотек и инструментов, не вмешиваясь в глобальное поведение системы.

Они представляют собой безопасный способ тестирования новых проектов и могут быть легко созданы с помощью модуля [venv](https://docs.python.org/3/library/venv.html#module-venv) из стандартной библиотеки.

Однако обратите внимание, что в зависимости от вашей операционной системы или дистрибутива **venv** может не устанавливаться по умолчанию вместе с Python. В этих случаях вам может потребоваться использовать диспетчер пакетов ОС для его установки. Например, в системах на основе Debian/Ubuntu вы можете получить его через:

```bash
sudo apt install python3-venv
```

Кроме того, вы также можете попробовать установить [virtualenv](https://pypi.org/project/virtualenv). Дополнительная информация доступна в Руководстве пользователя по упаковке Python по [установке пакетов с использованием pip и виртуальных сред](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/).
{% endhint %}

{% hint style="info" %}
_Изменено в версии v64.0.0_: Редактируемые установочные хуки реализованы в соответствии с [PEP 660](https://peps.python.org/pep-0660/). Поддержка [пакетов пространства имен](https://peps.python.org/pep-0420/) все еще **ЭКСПЕРИМЕНТАЛЬНА**.
{% endhint %}

## «Строгие» ("Strict") редактируемые установки

Размышляя о редактируемых установках, у пользователей могут быть следующие ожидания:

1. Это должно позволять разработчикам добавлять новые файлы (или разделять/переименовывать существующие) и автоматически отображать их.
2. Он должен вести себя как можно ближе к обычной установке и помогать пользователям обнаруживать проблемы (например, новые файлы, не включенные в дистрибутив).

К сожалению, эти ожидания противоречат друг другу. Чтобы решить эту проблему, **setuptools** позволяет разработчикам выбирать более строгий (_"strict"_) режим для редактируемой установки. Это можно сделать, передав специальный _параметр конфигурации_ через **pip**, как показано ниже:

```bash
pip install -e . --config-settings editable_mode=strict
```

В этом режиме новые файлы **не будут** отображаться, а редактируемые установки будут пытаться максимально имитировать поведение обычной установки. Под капотом **setuptools** создаст дерево ссылок на файлы во вспомогательном каталоге (`$your_project_dir/build`) и добавит его в **PYTHONPATH** через файл `.pth`. (Пожалуйста, будьте осторожны, чтобы не удалить этот репозиторий по ошибке, иначе ваши файлы могут стать недоступными).

{% hint style="warning" %}
Строго редактируемые установки требуют, чтобы вспомогательные файлы были помещены в каталог `build/__editable__.*` (относительно корня вашего проекта).

Пожалуйста, будьте осторожны, чтобы не удалить этот каталог во время тестирования вашего проекта, иначе ваша редактируемая установка может быть скомпрометирована.

Вы можете удалить каталог `build/__editable__.*` после деинсталляции.
{% endhint %}

{% hint style="info" %}
_Новое в версии v64.0.0_: Добавлен новый строгий режим для редактируемых установок. Точные детали реализации этого режима могут различаться.
{% endhint %}

## Ограничения

* Термин _**editable**_ используется для обозначения только модулей Python внутри каталогов пакетов. Файлы, отличные от Python, внешние файлы (данные), исполняемые файлы скриптов, двоичные расширения, заголовки и метаданные могут быть представлены в виде снимка версии _snapshot_, которой они были на момент установки.
* Добавление новых зависимостей, точек входа или изменение метаданных вашего проекта требует новой «редактируемой» переустановки.
* Консольные сценарии и сценарии графического интерфейса **ДОЛЖНЫ** быть указаны через <mark style="color:purple;">точки входа</mark> для правильной работы.
* _Строгие_ редактируемые установки требуют, чтобы файловая система поддерживала либо [символические](https://wikipedia.org/wiki/symbolic%20link), либо [жесткие ссылки](https://wikipedia.org/wiki/hard%20link). Этот режим установки может также создавать вспомогательные файлы в каталоге проекта.
* Нет _никакой гарантии_, что редактируемая установка будет выполнена с использованием определенной техники. В зависимости от каждого проекта **setuptools** может выбрать другой подход, чтобы обеспечить возможность импорта пакета во время выполнения.
* Нет _никакой гарантии_, что файлы за пределами каталога пакета верхнего уровня будут доступны после редактируемой установки.
* Нет _никакой гарантии_, что такие атрибуты, как `__path__` или `__file__`, будут соответствовать точному расположению исходных файлов (например, **setuptools** может использовать ссылки на файлы для выполнения редактируемой установки). Пользователям рекомендуется использовать такие инструменты, как `importlib.resources` или `importlib.metadata`, при попытке прямого доступа к файлам пакета.
* Редактируемые установки могут не работать с [пространствами имен, созданными с помощью pkgutil или pkg\_resources](https://packaging.python.org/en/latest/guides/packaging-namespace-packages/). Пожалуйста, используйте неявные пространства имен в стиле [PEP 420](https://peps.python.org/pep-0420/) [\[1\]](rezhim-razrabotki-a.k.a.-editable-installs.md#1).
* Поддержка пакетов неявного пространства имен в стиле [PEP 420](https://peps.python.org/pep-0420/) для проектов, структурированных с использованием макета [flat-layout](obnaruzhenie-paketov-i-pakety-prostranstva-imen.md#flat-layout), все еще является **экспериментальной**. Если у вас возникли проблемы, вы можете попробовать преобразовать структуру вашего пакета в [src-layout](obnaruzhenie-paketov-i-pakety-prostranstva-imen.md#src-layout).
* Записи файловой системы в текущем рабочем каталоге, имена которых совпадают с установленными пакетами, могут иметь приоритет в [системе импорта Python](https://docs.python.org/3/reference/import.html). Пользователям рекомендуется избегать таких сценариев [\[2\]](rezhim-razrabotki-a.k.a.-editable-installs.md#2).
* **Setuptools** попытается отдать приоритет модулям в редактируемой установке. Однако это не всегда простая задача. Если у вас есть определенный порядок в `sys.path` или какой-то особый приоритет импорта, который необходимо соблюдать, редактируемая установка, поддерживаемая **Setuptools**, может не соответствовать этому требованию, и, следовательно, это может быть неподходящий инструмент для вашего варианта использования.

{% hint style="danger" %}
Редактируемые установки **не являются идеальной заменой обычным установкам** в тестовой среде. Если вы сомневаетесь, пожалуйста, проверьте свои проекты, установленные через обычное колесо. В экосистеме Python есть инструменты, такие как [tox](https://pypi.org/project/tox) или [nox](https://pypi.org/project/nox), которые могут помочь вам в этом (при использовании с соответствующей конфигурацией).
{% endhint %}

## Устаревшее поведение

Если ваш проект не совместим с новыми «редактируемыми установками» или вы хотите воспроизвести устаревшее поведение, на данный момент вы также можете выполнить установку в режиме **compat**:

```bash
pip install -e . --config-settings editable_mode=compat
```

Этот режим установки попытается подражать тому, как работает `python setup.py develop` (все еще в контексте [PEP 660](https://peps.python.org/pep-0660/)).

{% hint style="warning" %}
Режим **compat** является _**переходным**_ и будет удален в будущих версиях **setuptools**, он существует только для помощи в период миграции. Также обратите внимание, что поддержка этого режима ограничена: можно с уверенностью предположить, что режим совместимости **compat** предлагается «как есть», и вряд ли будут реализованы улучшения. Пользователям предлагается опробовать новые редактируемые методы установки и внести необходимые изменения.
{% endhint %}

{% hint style="info" %}
Более новые версии **pip** больше не запускают резервную команду `python setup.py develop` при наличии файла `pyproject.toml`. Это означает, что установка переменной среды `SEUPTOOLS_ENABLE_FEATURES="legacy-editable"` не будет иметь никакого эффекта при установке пакета с **pip**.
{% endhint %}

## Как работают редактируемые установки

_Расширенная тема_

Существует множество методов, которые можно использовать для представления разрабатываемых пакетов таким образом, чтобы они были доступны, как если бы они были установлены. В зависимости от файловой структуры проекта и выбранного режима **setuptools** выберет один из этих подходов для редактируемой установки [\[3\]](rezhim-razrabotki-a.k.a.-editable-installs.md#3).

Ниже представлен неполный список механизмов реализации. Более подробная информация доступна в тексте [PEP 660](https://peps.python.org/pep-0660/#what-to-put-in-the-wheel).

* Статический файл `.pth` [\[4\]](rezhim-razrabotki-a.k.a.-editable-installs.md#4) можно добавить в один из каталогов, перечисленных в `site.getsitepackages()` или `site.getusersitepackages()`, чтобы расширить `sys.path`.
* Можно использовать каталог, содержащий _набор ссылок на файлы_, которые имитируют структуру проекта и указывают на исходные файлы. Затем этот каталог можно добавить в `sys.path` с помощью статического файла `.pth`.
* Динамический файл `.pth` [\[5\]](rezhim-razrabotki-a.k.a.-editable-installs.md#5) также можно использовать для установки «impor [finder](https://docs.python.org/3/glossary.html#term-finder)» ([MetaPathFinder](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder) или [PathEntryFinder](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder)), который подключается к механизму [import system](https://docs.python.org/3/reference/import.html) Python.

{% hint style="danger" %}
Setuptools не дает **никаких гарантий** того, какой метод будет использоваться для выполнения редактируемой установки. Это будет варьироваться от проекта к проекту и может меняться в зависимости от конкретной используемой версии **setuptools**.
{% endhint %}

## Примечания

#### \[ 1 ]

Вы _**можете**_ использовать строгие редактируемые установки с пакетами пространств имен, созданными с помощью **pkgutil** или **pkg\_namespaces**, однако официально это не поддерживается.

#### \[ 2 ]

Для предотвращения подобных ситуаций можно использовать такие методы, как [src-layout](obnaruzhenie-paketov-i-pakety-prostranstva-imen.md#src-layout), или параметры инструментов, такие как [changeir tox](https://tox.wiki/en/stable/config.html#conf-changedir) (ознакомьтесь с этой [записью в блоге](https://blog.ganssle.io/articles/2019/08/test-as-installed.html), чтобы узнать больше).

#### \[ 3 ]

**setuptools** стремится найти баланс между тем, чтобы позволить пользователю видеть результаты редактирования файлов проекта, и в то же время пытаться сохранить редактируемую установку как можно более похожей на обычную установку.

#### \[ 4 ]

т. е. файл `.pth`, где каждая строка соответствует пути, который следует добавить в [sys.path](https://docs.python.org/3/library/sys.html#sys.path). См. [Хук конфигурации для конкретного сайта](https://docs.python.org/3/library/site.html#module-site).

#### \[ 5 ]

т. е. файл `.pth`, который начинается там, где каждая строка начинается с оператора **import**, и выполняет произвольный код Python. См. [Хук конфигурации для конкретного сайта](https://docs.python.org/3/library/site.html#module-site).
