# Введение в supervisor

## Обзор

**Supervisor** — это клиент-серверная система, позволяющая пользователям управлять рядом процессов в UNIX-подобных операционных системах. Это было вдохновлено следующим:

### Удобство

Часто неудобно писать сценарии `rc.d` для каждого отдельного экземпляра процесса. Скрипты `rc.d` — это отличная форма инициализации/автозапуска/управления процессами с наименьшим общим знаменателем, но их написание и поддержка могут быть болезненными. Кроме того, сценарии `rc.d` не могут автоматически перезапускать аварийный процесс, и многие программы не перезапускаются должным образом при сбое. **Supervisord** запускает процессы как свои подпроцессы, и его можно настроить на автоматический перезапуск при сбое. Его также можно автоматически настроить для запуска процессов при его собственном вызове.

### Точность

Часто бывает сложно получить точное состояние запуска/отключения процессов в UNIX. Пидфайлы часто врут. **Supervisord** запускает процессы как подпроцессы, поэтому он всегда знает истинное состояние включения/выключения своих дочерних процессов и может удобно запрашивать эти данные.

### Делегирование

Пользователям, которым необходимо контролировать состояние процесса, часто достаточно делать только это. Они не хотят и не нуждаются в полноценном доступе оболочки к машине, на которой запущены процессы. Процессы, которые прослушивают «low» TCP-порты, часто необходимо запускать и перезапускать от имени пользователя **root** (ошибка UNIX). Обычно совершенно нормально разрешать «обычным» людям останавливать или перезапускать такой процесс, но предоставление им доступа к оболочке часто нецелесообразно, а предоставление им доступа **root** или доступа **sudo** часто невозможно. Также (правильно) трудно объяснить им, почему существует эта проблема. Если супервизор запущен как **root**, можно позволить «обычным» пользователям управлять такими процессами без необходимости объяснять им тонкости проблемы. **Supervisorctl** предоставляет очень ограниченную форму доступа к машине, по существу позволяя пользователям видеть состояние процесса и управлять подпроцессами, контролируемыми супервизором, выдавая команды `«stop»`, `«start»` и `«restart»` из простой оболочки или веб-интерфейса.

### Группы процессов

Процессы часто нужно запускать и останавливать группами, иногда даже в «приоритетном порядке». Часто бывает сложно объяснить людям, как это сделать. **Supervisor** позволяет вам назначать приоритеты процессам и позволяет пользователю подавать команды через клиент **supervisorctl**, такие как `«start all»` и `«restart all»`, которые запускают их в заранее заданном порядке приоритета. Кроме того, процессы можно группировать в «группы процессов», а набор логически связанных процессов можно останавливать и запускать как единое целое.

## Особенности

### Простота

**Supervisor** настраивается с помощью простого конфигурационного файла в стиле **INI**, который легко освоить. Он предоставляет множество параметров для каждого процесса, которые облегчают вашу жизнь, например, перезапуск сбойных процессов и автоматическая ротация журналов.

### Централизованность

**Supervisor** предоставляет вам единое место для запуска, остановки и мониторинга ваших процессов. Процессами можно управлять индивидуально или в группах. Вы можете настроить **Supervisor** для предоставления локальной или удаленной командной строки и веб-интерфейса.

### Эффективность

**Supervisor** запускает свои подпроцессы через `fork/exec`, а подпроцессы не демонизируются. Операционная система немедленно сигнализирует **Supervisor**, когда процесс завершается, в отличие от некоторых решений, которые полагаются на проблемные файлы **PID** и периодический опрос для перезапуска отказавших процессов.

### Расширяемость

**Supervisor** имеет простой протокол уведомления о событиях, который программы, написанные на любом языке, могут использовать для его мониторинга, и интерфейс **XML-RPC** для управления. Он также построен с точками расширения, которые могут использоваться разработчиками Python.

### Совместимость

**Supervisor** работает практически со всем, кроме **Windows**. Он протестирован и поддерживается в **Linux**, **Mac OS X**, **Solaris** и **FreeBSD**. Он полностью написан на Python, поэтому для установки не требуется компилятор C.

### Доказанность

Хотя сегодня **Supervisor** очень активно развивается, это не новое программное обеспечение. **Supervisor** существует уже много лет и уже используется на многих серверах.

## Компоненты супервизора

### supervisord

Серверная часть супервизора называется **supervisord**. Он отвечает за запуск дочерних программ по своему собственному вызову, реагирование на команды клиентов, перезапуск аварийных или завершившихся подпроцессов, регистрацию вывода **stdout** и **stderr** своего подпроцесса, а также создание и обработку «событий», соответствующих точкам времени жизни подпроцесса.

Серверный процесс использует файл конфигурации. Обычно он находится в `/etc/supervisord.conf`. Этот файл конфигурации представляет собой файл конфигурации в стиле «Windows-INI». Важно обеспечить безопасность этого файла с помощью соответствующих разрешений файловой системы, поскольку он может содержать незашифрованные имена пользователей и пароли.

### supervisorctl

Клиентская часть супервизора для командной строки называется **supervisorctl**. Он предоставляет интерфейс, похожий на оболочку, для функций, предоставляемых **supervisord**. Из **supervisorctl** пользователь может подключаться к различным процессам **supervisord** (по одному), получать статус контролируемых подпроцессов, останавливать и запускать подпроцессы и получать списки запущенных процессов **supervisord**.

Клиент командной строки взаимодействует с сервером через сокет домена UNIX или через сокет Интернета (TCP). Сервер может утверждать, что пользователь клиента должен представить учетные данные для аутентификации, прежде чем он позволит ему выполнять команды. Клиентский процесс обычно использует тот же файл конфигурации, что и сервер, но подойдет любой файл конфигурации с разделом `[supervisorctl]`.

### Веб-сервер

Доступ к (разреженному) веб-интерфейсу пользователя с функциональностью, сравнимой с **supervisorctl**, можно получить через браузер, если вы запустите **supervisord** через интернет-сокет. Посетите URL-адрес сервера (например, `http://localhost:9001/`), чтобы просмотреть и контролировать состояние процесса через веб-интерфейс после активации раздела `[inet_http_server]` файла конфигурации.

### Интерфейс XML-RPC

Тот же HTTP-сервер, который обслуживает веб-интерфейс, обслуживает интерфейс XML-RPC, который можно использовать для опроса и управления супервизором и программами, которые он запускает. См. <mark style="color:purple;">Документацию API XML-RPC</mark>.

## &#x20;Требования к платформе

**Supervisor** был протестирован и, как известно, работает в **Linux** (`Ubuntu 18.04`), **Mac OS X** (`10.4/10.5/10.6`), **Solaris** (`10 for Intel`) и **FreeBSD 6.1**. Скорее всего, он будет нормально работать в большинстве систем **UNIX**.

**Supervisor** вообще _**не будет работать**_ под любой версией **Windows**.

**Supervisor** предназначен для работы с Python 3 версии 3.4 или новее и с Python 2 версии 2.7.
