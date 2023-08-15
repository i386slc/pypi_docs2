# Секретная информация (secrets)

## Использование файлов .secrets

Для безопасного хранения конфиденциальных данных Dynaconf также ищет файл `.secrets.{toml|py|json|ini|yaml}` для поиска таких данных, как токены и пароли.

Пример `.secrets.toml`:

```toml
password = "sek@987342$"
```

{% hint style="success" %}
**Совет**

Если параметр `environments=True` включен, файл секретов поддерживает все определения среды **environment**, поддерживаемые в файле настроек **settings**.
{% endhint %}

{% hint style="info" %}
Причиной использования файла `.secrets.*` является возможность опустить этот файл при фиксации в репозитории, поэтому рекомендуемый `.gitignore` должен включать строку `.secrets.*`.
{% endhint %}

## Использование сервера Vault

[vaultproject.io/](https://www.vaultproject.io/) — это хранилище секретов типа «ключ:значение», и Dynaconf может загружать переменные из секрета Vault.

* Запустите сервер хранилища

Запустите установленный сервер **Vault** или через докер:

```bash
$ docker run -d -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' -p 8200:8200 vault
```

* Установить поддержку хранилища в dynaconf

```bash
$ pip install dynaconf[vault]
```

* В вашем файле `.env` или в экспортированных переменных среды определите:

```bash
VAULT_ENABLED_FOR_DYNACONF=true
VAULT_URL_FOR_DYNACONF="http://localhost:8200"
# Укажите механизм секретов для kv, по умолчанию 1
VAULT_KV_VERSION_FOR_DYNACONF=1
# Авторизоваться с помощью токена https://www.vaultproject.io/docs/auth/token
VAULT_TOKEN_FOR_DYNACONF="myroot"
# Авторизоваться с AppRole https://www.vaultproject.io/docs/auth/approle
VAULT_ROLE_ID_FOR_DYNACONF="role-id"
VAULT_SECRET_ID_FOR_DYNACONF="secret-id"
# Авторизоваться с AWS IAM https://www.vaultproject.io/docs/auth/aws
# Учетные данные IAM можно получить у стандартных поставщиков:
# https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html
VAULT_AUTH_WITH_IAM_FOR_DYNACONF=True
VAULT_AUTH_ROLE_FOR_DYNACONF="vault-role"
# Аутентификация с корневым токеном
VAULT_ROOT_TOKEN_FOR_DYNACONF="root-token"
```

Или передайте его экземпляру Dynaconf.

```python
settings = Dynaconf(
    environment=True,
    vault_enabled=True,
    # рекомендуется сохранить как env var.
    vault={'url': 'http://localhost:8200', 'token': 'myroot'}
)
```

Теперь у вас могут быть такие ключи, как **PASSWORD** и **TOKEN**, определенные в хранилище, и dynaconf прочитает их.

Чтобы написать новый секрет, вы можете использовать веб-администратор `http://localhost:8200` и написать ключи в секретной базе данных `/secret/dynaconf/<env>`.

Вы также можете использовать модуль записи Dynaconf через консоль:

```
# запишет {'password': 123456} в secret/dynaconf/default
$ dynaconf -i config.settings write vault -s password=123456

# запишет {'password': 123456, 'username': 'admin'} в secret/dynaconf/default
$ dynaconf -i config.settings write vault -s password=123456 -s username=admin

# запишет {'password': 555555} в secret/dynaconf/development
$ dynaconf -i config.settings write vault -s password=555555  -e development

# запишет {'password': 777777, 'username': 'admin'} в secret/dynaconf/production
$ dynaconf -i config.settings write vault -s password=777777 -s username=produser -e production
```

Затем вы можете нормально получать доступ к значениям в вашей программе

```python
from dynaconf import settings

settings.PASSWORD == 555555  # если ENV_FOR_DYNACONF по умолчанию `development`
settings.USERNAME == 'admin'  # если ENV_FOR_DYNACONF по умолчанию `development`

settings.PASSWORD == 777777  # если ENV_FOR_DYNACONF - `production`
settings.USERNAME == 'produser'  # если ENV_FOR_DYNACONF - `production`
```

Вы также можете запросить загрузку настроек из определенной среды с помощью метода **from\_env**:

```python
settings.from_env('production').PASSWORD == 777777
settings.from_env('production').USERNAME == 'produser'
```

## Дополнительный файл секретов (для CI, jenkins и т. п.)

Обычно файл дополнительных секретов **secrets** доступен только при работе в определенной среде **CI**, такой как **Jenkins**, обычно там будет переменная среды, указывающая на файл.

В **Jenkins** это делается в настройках задания путем экспорта секретной информации **secrets**.

Dynaconf может справиться с этим через переменную среды **SECRETS\_FOR\_DYNACONF**.

Например:

```python
settings = Dynaconf(secrets="path/to/secrets.toml")
```

или

```bash
export SECRETS_FOR_DYNACONF=/path/to/secrets.toml{json|py|ini|yaml}
```

Если эта переменная существует в вашей среде, Dynaconf также загрузит ее.

## Внешнее хранилище

Внешнее хранилище необходимо в некоторых программах для таких сценариев, как:

1. Наличие единого хранилища для настроек и их распределение по нескольким экземплярам
2. Необходимость изменять настройки на лету без повторного развертывания или перезапуска приложения
3. Хранение конфиденциальных данных в безопасном закрытом хранилище

### Использование Redis

* Запустите установленный сервер Redis или через докер:

```bash
$ docker run -d -p 6379:6379 redis
```

* Установите поддержку Redis в dynaconf

```bash
$ pip install dynaconf[redis]
```

* В вашем файле `.env` или в экспортированных переменных среды определите:

```bash
REDIS_ENABLED_FOR_DYNACONF=true
REDIS_HOST_FOR_DYNACONF=localhost
REDIS_PORT_FOR_DYNACONF=6379
REDIS_USERNAME_FOR_DYNACONF=<ACL username>(optional)
REDIS_PASSWORD_FOR_DYNACONF=<password>(optional)
```

Теперь вы можете записывать переменные непосредственно в хэш redis с именем `DYNACONF_<env>`, например:

* **DYNACONF\_DEFAULT**: значения по умолчанию
* **DYNACONF\_DEVELOPMENT**: загружается только тогда, когда `ENV_FOR_DYNACONF=development` (по умолчанию)
* **DYNACONF\_PRODUCTION**: загружается только тогда, когда `ENV_FOR_DYNACONF=production`
* **DYNACONF\_CUSTOM**: загружается, только если `ENV_FOR_DYNACONF=custom`

Вы также можете использовать Redis Writer

```bash
$ dynaconf write redis -v name=Bruno -v database=localhost -v port=1234
```

Приведенные выше данные будут записаны в Redis в виде хэша:

```python
DYNACONF_DEFAULT {
    NAME='Bruno'
    DATABASE='localhost'
    PORT='@int 1234'
}
```

Если вы хотите писать в конкретную **env**, используйте опцию `-e`.

```bash
$ dynaconf write redis -v name=Bruno -v database=localhost -v port=1234 -e production
```

Приведенные выше данные будут записаны в Redis в виде хэша:

```python
DYNACONF_PRODUCTION {
    NAME='Bruno'
    DATABASE='localhost'
    PORT='@int 1234'
}
```

Затем для доступа к этим значениям вы можете установить `export ENV_FOR_DYNACONF=production` или напрямую через `settings.from_env('production').NAME`

> если вы хотите пропустить приведение типов, напишите в виде строки, вместо `PORT=1234` используйте `PORT="'1234'"`.

Данные считываются из redis и других загрузчиков только один раз при первом доступе к `dynaconf.settings` или при вызове `from_env`, `.setenv()` или `using_env()`.

Вы можете получить доступ к свежему значению, используя **settings.get\_fresh(key)**

Также есть **свежий** контекстный менеджер

```python
from dynaconf import settings

print(settings.FOO)  # эти данные были загружены один раз при импорте

with settings.fresh():
    print(settings.FOO)  # эти данные заново загружаются из источника

# эти данные заново загружаются из источника
print(settings.get('FOO', fresh=True))
```

И вы также можете заставить некоторые переменные быть **свежими** в вашем файле настроек.

```python
FRESH_VARS_FOR_DYNACONF = ['MYSQL_HOST']
```

или используя env vars

```bash
export FRESH_VARS_FOR_DYNACONF='["MYSQL_HOST", "OTHERVAR"]'
```

Затем

```python
from dynaconf import settings

print(settings.FOO)         # Эти данные были загружены один раз при импорте

print(settings.MYSQL_HOST)  # Эти данные считываются из Redis немедленно!
```

## Пользовательские хранилища

Вы хотите хранить настройки в других базах данных, таких как NoSQL, реляционные базы данных или другие службы?

Пожалуйста, посмотрите, как [расширить dynaconf](rasshirennoe-ispolzovanie-dynaconf.md), чтобы добавить свои собственные загрузчики.
