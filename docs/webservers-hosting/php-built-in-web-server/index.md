---
description: Использование встроенного веб-сервера.
---

# Тестовый хостинг со встроенным веб-сервером PHP

Командная строка PHP (CLI SAPI) имеет встроенный веб-сервер, который полезен для быстрого тестирования или демонстрации сайта Grav. Это *не* полнофункциональный веб-сервер и потому он не должен использоваться в публичной сети.

## Использование веб-сервера CLI

1. В командной строке перейдите в папку [GRAV_ROOT].
2. Выполните команду `php -S localhost:8080 system/router.php`. Вы должны увидеть ответ, подобный приведенному ниже.

```
php -S localhost:8080 system/router.php
PHP 7.3.27 Development Server started at Thu Jun 17 09:24:46 2021
Listening on http://localhost:8080
Document root is /Users/somerandom/Desktop/quick-grav-test
Press Ctrl-C to quit.
```

3. Перейдите к указанному URL-адресу, например: `http://localhost:8080/`.
4. Чтобы остановить веб-сервер, нажмите Ctrl-C.

### Ошибка "Адрес уже используется"

Если вы получите сообщение "Адрес уже используется" при выполнении команды `php -S`, значит на вашей машине уже запущен веб-сервер на указанном номере порта (например, `:8080`). Вы можете решить эту проблему, изменив номер порта в вашей команде (например, `:8181`) и повторив попытку.

## Отображение журнала в режиме реального времени

Веб-сервер CLI отображает свой журнал в режиме реального времени, пока вы просматриваете сайт, что может быть полезно для быстрого тестирования.

```
PHP 7.3.27 Development Server started at Thu Jun 17 09:24:46 2021
Listening on http://localhost:8080
Document root is /Users/somerandom/Desktop/quick-grav-test
Press Ctrl-C to quit.
[Thu Jun 17 09:26:15 2021] 127.0.0.1:63965 [200]: /
[Thu Jun 17 09:26:15 2021] 127.0.0.1:64007 [200]: /assets/fd2c5827e1f18bb54d20265f4fc56b59.css?g-74e4c5a3
[Thu Jun 17 09:26:15 2021] 127.0.0.1:64008 [200]: /assets/d87a2d24fae663a8c55e144c963a1915.js?g-74e4c5a3
[Thu Jun 17 09:26:15 2021] 127.0.0.1:64014 [200]: /assets/1d8c5ea92966046d4649472f1630a253.js?g-74e4c5a3
[Thu Jun 17 09:26:16 2021] 127.0.0.1:64024 [200]: /user/images/navigation/logo_small.png
[Thu Jun 17 09:26:16 2021] 127.0.0.1:64028 [200]: /user/images/navigation/bgdark.svg
[Thu Jun 17 09:26:16 2021] 127.0.0.1:64030 [200]: /user/images/navigation/bglight_50.png
[Thu Jun 17 09:26:16 2021] 127.0.0.1:64032 [200]: /user/images/navigation/brand.svg
```

## Дополнительная информация

Посетите веб-сайт PHP, чтобы узнать больше о [встроенном веб-сервере](https://www.php.net/manual/ru/features.commandline.webserver.php?target=_blank).
