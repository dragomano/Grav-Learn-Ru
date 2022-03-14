---
description: Использование Grav CMS с помощью встроенного сервера
---

# Встроенный веб-сервер Grav

Вы можете запустить Grav, используя простую команду из терминала/командной строки, используя встроенный сервер PHP, доступный для любой системы с установленным PHP.

Всё, что вам нужно сделать, это перейти к корню вашей установки Grav с помощью терминала или командной строки и ввести `bin/grav server`.

!!! tip ""

    Хотя технически всё, что вам нужно, это установить PHP, если вы установите приложение [Symfony CLI](https://symfony.com/download), сервер предоставит SSL-сертификат, чтобы вы могли использовать `https://` и PHP-FPM для повышения производительности.

Ввод этой команды приведет к выводу, аналогичному следующему:

```bash
➜ bin/grav server

Grav Web Server
===============

Tailing Web Server log file (/Users/joeblow/.symfony/log/96e710135f52930318e745e901e4010d0907cec3.log)
Tailing PHP-FPM log file (/Users/joeblow/.symfony/log/96e710135f52930318e745e901e4010d0907cec3/53fb8ec204547646acb3461995e4da5a54cc7575.log)
Tailing PHP-FPM log file (/Users/joeblow/.symfony/log/96e710135f52930318e745e901e4010d0907cec3/53fb8ec204547646acb3461995e4da5a54cc7575.log)

[OK] Web server listening
The Web server is using PHP FPM 8.0.8
https://127.0.0.1:8000


[Web Server ] Jul 30 14:54:53 |DEBUG  | PHP    Reloading PHP versions
[Web Server ] Jul 30 14:54:54 |DEBUG  | PHP    Using PHP version 8.0.8 (from default version in $PATH)
[PHP-FPM    ] Jul  6 14:40:17 |NOTICE | FPM    fpm is running, pid 64992
[PHP-FPM    ] Jul  6 14:40:17 |NOTICE | FPM    ready to handle connections
[PHP-FPM    ] Jul  6 14:40:17 |NOTICE | FPM    fpm is running, pid 64992
[PHP-FPM    ] Jul  6 14:40:17 |NOTICE | FPM    ready to handle connections
[Web Server ] Jul 30 14:54:54 |INFO   | PHP    listening path="/usr/local/Cellar/php/8.0.8_2/sbin/php-fpm" php="8.0.8" port=65140
[PHP-FPM    ] Jul 30 14:54:54 |NOTICE | FPM    fpm is running, pid 73709
[PHP-FPM    ] Jul 30 14:54:54 |NOTICE | FPM    ready to handle connections
[PHP-FPM    ] Jul 30 14:54:54 |NOTICE | FPM    fpm is running, pid 73709
[PHP-FPM    ] Jul 30 14:54:54 |NOTICE | FPM    ready to handle connections
```

Ваш терминал также будет предоставлять вам в режиме реального времени обновления любой активности на этом специальном сервере. Вы можете скопировать URL-адрес, указанный в строке `[OK] Web server listening`, и вставить его в используемый браузер, чтобы получить доступ к своему сайту, включая админку.

```
https://127.0.0.1:8000
```

!!! danger ""

    Это полезный инструмент для быстрой разработки, который **не** следует использовать вместо выделенного веб-сервера, такого как Apache или Nginx.
