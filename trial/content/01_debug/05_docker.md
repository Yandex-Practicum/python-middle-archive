### Отладка кода в Docker

![10_docker](pictures/4_10_docker.png)

Docker — это открытая платформа для разработки, доставки и эксплуатации приложений. Если разработчик использует Docker, ему не нужно устанавливать дополнения для запуска приложения. Например, для запуска Python-приложения без этой платформы разработчику необходимо установить интерпретатор и библиотеки. Поэтому для более простого запуска приложения часто упаковывают в Docker-контейнер, чтобы они не зависели от окружающей среды сервера.

Контейнер — это маленькая виртуальная машина, в которой установлено всё необходимое для работы программы. Если вы используете PyCharm Professional, то через него вы сможете запустить программу и отладчик внутри такого контейнера.

Во время отладки разработчик запускает полностью идентичный контейнер на своем компьютере и на сервере. Без контейнера это сделать довольно тяжело: если разработчик работает на Windows или MacOS, а на сервере установлен Linux, то собрать идентичное окружение невозможно. 

Пример из жизни: была система, которая интегрировалась с SIP-телефонией с помощью C-библиотеки libexosip2. Проблема заключалась в том, что всё работало только на ОС Ubuntu 12.04. На более новых версиях ОС запустить систему телефонии не получалось, потому что их библиотеки были не совместимы. Более того, нельзя было собрать библиотеку под OSX или Windows. Для того, чтобы не привязывать разработчиков к использовании Ubuntu 12.04 и иметь возможность обновлять ОС сервера, приложение запускалось в Docker.

Контейнеры исключают ситуацию, что у программиста всё работает, а на сервере — нет.

[Инструкция по настройке отладки в контейнере для самостоятельного изучения на английском](https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html)

Отладка кода в Docker поможет при работе над сервисом Admin panel. Вам предстоит запустить базу данных и Python-приложение в Docker, связать их и убедиться, что всё работает. Вы изучите практики IT-компаний, чтобы ответить на вопрос: запускать базы данных в Docker или нет. А ещё вы поймёте, может ли Docker заменить Virtual Environment. 

Если что-то пойдет не так, с этим видом отладки искать проблемы будет гораздо проще.

## Что делать, если нет возможности воспроизвести prod-среду, а баги есть?

Иногда у пользователя всё работает, а у владельца сервера — нет. У разработчика не всегда есть возможность воспроизвести prod-среду на своём компьютере или ошибку на dev-сервере. Если у вас есть доступ на сервер, на котором возник злостный баг, можно поступить следующим образом: 

- Добавить брейкпоинт в код программы с помощью ipdb. Скорее всего, у вас в распоряжении будет только он.
- Поднять приложение на порту, который отличается от рабочего приложения, добавив 1 к его номеру: 8000-й порт превратится в 8001-й. Это нужно для того, чтобы разделить запросы пользователей и запросы разработчика. Если этого не сделать, вы рискуете повесить весь сервер, когда приложение остановится на брейкпоинте.
- Сделать запрос к дебаг-версии приложения с помощью [curl](https://curl.haxx.se/docs/manual.html) или любого другого HTTP-клиента — [Postman](https://www.postman.com), [Insomnia](https://insomnia.rest), [HTTPie](https://httpie.org).
- После этого приложение перейдет в консоль ipdb на то место, где указан брейкпоинт.
- Теперь можно приступить к удалённой отладке кода. Весь процесс будет похож на тот, если бы вы дебажили код локально на своём компьютере.

Если у разработчика нет доступа к prod-серверу, он не сможет интуитивно понять, в чём ошибка. В этом случае нужны дополнительные инструменты, которые помогут диагностировать проблемы.

При разработке сервиса Acync API вы будете использовать агрегатор ошибок [Sentry](https://sentry.io/) и сборщик метрик [Prometheus](https://prometheus.io). А при работе над сервисом авторизации, вы соберёте все логи в одно место и настроите [ELK](https://www.elastic.co/what-is/elk-stack).