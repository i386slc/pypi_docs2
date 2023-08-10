# Файл .pypirc

Файл `.pypirc` позволяет вам определить конфигурацию [индексов пакетов](https://packaging.python.org/en/latest/glossary/#term-Package-Index) (называемых здесь "репозиториями"), поэтому вам не нужно вводить URL-адрес, имя пользователя или пароль всякий раз, когда вы загружаете пакет с помощью repositories или [flit](https://packaging.python.org/en/latest/key\_projects/#flit).

Формат (первоначально определенный пакетом [distutils](https://packaging.python.org/en/latest/key\_projects/#distutils)):

```ini
[distutils]
index-servers =
    first-repository
    second-repository

[first-repository]
repository = <first-repository URL>
username = <first-repository username>
password = <first-repository password>

[second-repository]
repository = <second-repository URL>
username = <second-repository username>
password = <second-repository password>
```

Раздел **distutils** определяет поле **index-servers**, в котором перечислены имена всех разделов, описывающих репозиторий.

Каждый раздел, описывающий репозиторий, определяет три поля:

* `repository`: URL-адрес репозитория.
* `username`: зарегистрированное имя пользователя в репозитории.
* `password`: пароль, который будет использоваться для аутентификации имени пользователя.

{% hint style="warning" %}
**Предупреждение**

Имейте в виду, что это сохраняет ваш пароль в виде обычного текста. Для большей безопасности рассмотрите альтернативу, например набор ключей [keyring](https://pypi.org/project/keyring/), настройку переменных среды или ввод пароля в командной строке.

В противном случае установите разрешения для `.pypirc`, чтобы только вы могли просматривать или изменять его. Например, в Linux или macOS запустите:

```bash
chmod 600 ~/.pypirc
```
{% endhint %}

## Общие конфигурации

{% hint style="info" %}
Эти примеры относятся к [twine](https://packaging.python.org/en/latest/key\_projects/#twine). Другие проекты (например, [flit](https://packaging.python.org/en/latest/key\_projects/#flit)) также используют `.pypirc`, но с другими значениями по умолчанию. Пожалуйста, обратитесь к документации каждого проекта для получения более подробной информации и инструкций по использованию.
{% endhint %}

Конфигурация **Twine** по умолчанию имитирует `.pypirc` с разделами репозитория для PyPI и TestPyPI:

```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
repository = https://upload.pypi.org/legacy/

[testpypi]
repository = https://test.pypi.org/legacy/
```

**Twine** добавит дополнительную конфигурацию из `$HOME/.pypirc`, командной строки и переменных среды к этой конфигурации по умолчанию.

## Использование токена PyPI

Чтобы настроить [токен API](https://pypi.org/help/#apitoken) для PyPI, вы можете создать файл `$HOME/.pypirc`, аналогичный следующему:

```ini
[pypi]
username = __token__
password = <PyPI token>
```

Для [TestPyPI](https://packaging.python.org/en/latest/guides/using-testpypi/#using-test-pypi) добавьте раздел `[testpypi]`, используя токен API из вашей учетной записи TestPyPI.

## Использование другого индекса пакета

Чтобы настроить дополнительный репозиторий, вам нужно переопределить поле **index-servers**, включив в него имя репозитория. Вот полный пример `$HOME/.pypirc` для PyPI, TestPyPI и частного репозитория:

```ini
[distutils]
index-servers =
    pypi
    testpypi
    private-repository

[pypi]
username = __token__
password = <PyPI token>

[testpypi]
username = __token__
password = <TestPyPI token>

[private-repository]
repository = <private-repository URL>
username = <private-repository username>
password = <private-repository password>
```

{% hint style="warning" %}
Вместо использования поля **password** рассмотрите возможность безопасного сохранения ваших токенов и паролей API с помощью набора ключей [keyring](https://pypi.org/project/keyring/) (который устанавливается **Twine**).
{% endhint %}

```bash
keyring set https://upload.pypi.org/legacy/ __token__
keyring set https://test.pypi.org/legacy/ __token__
keyring set <private-repository URL> <private-repository username>
```
