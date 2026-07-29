---
description: Настройка сервера Caddy. Конфигурация сайта Grav CMS.
---

# Caddy

*Caddy* — это современный веб-сервер с открытым исходным кодом, написанный на Go. Его главное преимущество — автоматическая поддержка HTTPS: Caddy самостоятельно получает и продлевает TLS-сертификаты без какой-либо дополнительной настройки. Как и Nginx, Caddy **не** читает файлы Apache `.htaccess`, поэтому в каждом релизе Grav поставляется эквивалентный файл `Caddyfile`, расположенный в каталоге `webserver-configs/`.

## Требования

* `caddy` (v2 or newer)
* `php-fpm` (FastCGI Process Manager) to process PHP scripts

## Быстрый старт

Grav поставляется с готовым к использованию файлом `Caddyfile`, расположенным в каталоге `webserver-configs/`. Чтобы запустить его локально, выполните следующую команду из корневого каталога вашего сайта на Grav:

```bash
caddy run --config webserver-configs/Caddyfile
```

По умолчанию сайт будет доступен по адресу `https://localhost`, а обращения к `http://localhost` будут автоматически перенаправляться на него (в Caddy v2 поддержка HTTPS включается автоматически). Убедитесь, что процесс PHP-FPM прослушивает адрес `127.0.0.1:9000`, либо измените адрес в директиве `php_fastcgi` в соответствии с вашей конфигурацией.

## Поставляемый файл Caddyfile

Поставляемый файл является основным эталоном конфигурации. При использовании в рабочей среде замените `localhost` на имя вашего домена, чтобы Caddy мог получить настоящий TLS-сертификат, и укажите в `root` путь к каталогу вашего сайта на Grav:

```nginx
example.com
encode gzip
root * /var/www/grav
file_server

php_fastcgi 127.0.0.1:9000

# Begin - Security
# deny all direct access for these folders
rewrite /(\.git|cache|bin|logs|backups|tests)/.* /403

# deny all direct access to these sensitive user folders, whatever the file type
rewrite /user/(accounts|config|env)/.* /403

# block user/data too, but allow public media uploads (e.g. Flex Object images)
# to be served directly. SVG is intentionally excluded as a stored-XSS vector.
# (Go's RE2 has no lookbehind, so this uses a negated matcher.)
@user_data_nonmedia {
	path_regexp /user/data/.*
	not path_regexp (?i)\.(jpe?g|png|gif|webp|avif|bmp|ico|mp4|webm|ogg|ogv|mov|mp3|wav|m4a|flac|pdf)$
}
rewrite @user_data_nonmedia /403

# deny running scripts inside core system folders
rewrite /(system|vendor)/.*\.(txt|xml|md|html|htm|shtml|shtm|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ /403

# deny running scripts inside user folder
rewrite /user/.*\.(txt|md|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ /403

# deny access to specific files in the root folder
rewrite /(LICENSE\.txt|composer\.lock|composer\.json|nginx\.conf|web\.config|htaccess\.txt|\.htaccess) /403

# deny access to .env environment files
@dotenv path_regexp \.env(\..+)?$
respond @dotenv 403

respond /403 403
## End - Security

# global rewrite should come last.
try_files {path} {path}/ /index.php?_url={uri}&{query}
```

!!! note
    Глобальное правило `try_files` должно располагаться **последним**, чтобы сначала применялись расположенные выше правила безопасности. Поскольку движок RE2, используемый Go, не поддерживает просмотр назад (lookbehind), исключение для медиафайлов в каталоге `user/data` реализовано с помощью отрицательного сопоставления, а не одного шаблона.

## Запуск в качестве службы

Для постоянной установки поместите изменённый файл в `/etc/caddy/Caddyfile` и управляйте Caddy через systemd:

```bash
sudo systemctl reload caddy
```

## Проверка работы

Запрос к закрытому файлу, например `https://example.com/user/config/system.yaml`, должен возвращать ответ **403 Forbidden**, а медиафайл, загруженный вами в каталог `user/data` (например, `.jpg`), должен по-прежнему успешно открываться. Подробнее о предупреждении, отображаемом в панели администратора, когда эти правила не применяются, см. на странице [Раскрытие папки `user`](/security/user-folder-exposure).
