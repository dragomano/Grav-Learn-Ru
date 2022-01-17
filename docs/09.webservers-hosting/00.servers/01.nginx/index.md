# Nginx

> Настройка сервера Nginx. Конфигурация сайта Grav CMS.

*Nginx* - это программное обеспечение HTTP-сервера, ориентированное на основные функции веб-сервера и прокси. Это очень распространено из-за своей ресурсной эффективности и отзывчивости под нагрузкой. Nginx порождает рабочие процессы, каждый из которых может обрабатывать тысячи соединений. Каждое из соединений, обрабатываемых работником, помещается в цикл событий, где они существуют с другими соединениями. Внутри цикла события обрабатываются асинхронно, что позволяет обрабатывать работу неблокирующим образом. Когда соединение закрывается, оно удаляется из цикла. Такой стиль обработки соединения позволяет Nginx невероятно масштабироваться при ограниченных ресурсах.

<!-- source: https://www.digitalocean.com/community/tutorials/apache-vs-nginx-practical-considerations -->

## Требования

На этой странице объясняется, как запустить Grav с *Nginx* в качестве HTTP-сервера и *PHP-FPM* (FastCGI Process Manager) для обработки скриптов PHP, поэтому эти пакеты необходимо установить на ваш сервер:

* `nginx`
* `php-fpm`

## Конфигурация

Если вы новичок в Nginx и ещё не имеете базовых знаний о директивах/контексте блока, рекомендуется прочитать следующие материалы по Nginx: [Руководство для начинающих](http://nginx.org/en/docs/beginners_guide.html), особенно разделы [Структура файла конфигурации](http://nginx.org/en/docs/beginners_guide.html#conf_structure) и [Обслуживание статического содержимого](http://nginx.org/en/docs/beginners_guide.html#static).

Предполагается, что ваша конфигурация Nginx находится в `/etc/nginx/`, а ваша установка Grav хранится в `/var/www/grav/`. Структура конфигурации представляет собой блок http, который содержит общие директивы, относящиеся ко всем страницам, обслуживаемым Nginx, а также один или несколько блоков server для каждой страницы, содержащие директивы для конкретного сайта. Главный файл конфигурации сервера — это `nginx.conf` и хранит блок `http`, в то время как специфичные для сайта конфигурации (блоки `server`) хранятся в `sites-available` и имеют символические ссылки на `sites-enabled`.

### Права доступа к файлам

Каталог `/var/www` и все содержащиеся в нем файлы и папки должны принадлежать `$USER: www-data` (или как там вы называете пользователя/группу Nginx). В разделе [Устранение неполадок / Разрешения](/11.troubleshooting/05.permissions/index) объясняется, как настроить права доступа к файлам и каталогам для Grav, в данном случае с использованием общей группы. По сути, вам нужно `775` для каталогов и `664` для файлов в каталоге Grav, поэтому Grav может изменять контент и обновлять себя. Вы должны добавить своего пользователя в группу `www-data`, чтобы вы могли получить доступ к файлам, созданным Grav/Nginx.


### Пример nginx.conf

Следующая конфигурация представляет собой улучшенную версию файла по умолчанию `/etc/nginx/nginx.conf`, в основном с улучшениями из [github.com/h5bp/server-configs-nginx](https://github.com/h5bp/server-configs-nginx). См. Их репозиторий для объяснения этих настроек или Nginx [основной модуль](http://nginx.org/en/docs/ngx_core_module.html) и [http-модуль](http://nginx.org/en/docs/http/ngx_http_core_module.html) документацию для поиска конкретных директив.

> Рекомендуется использовать обновленный файл определения типов MIME (`mime.types`) с сайта [github.com/h5bp/server-configs-nginx](https://github.com/h5bp/server-configs-nginx). Это гарантирует, что типы правильно установлены для сжатия gzip.

**nginx.conf**:

```nginx
user www-data;
worker_processes auto;
worker_rlimit_nofile 8192; # should be bigger than worker_connections
pid /run/nginx.pid;

events {
    use epoll;
    worker_connections 8000;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 30; # longer values are better for each ssl client, but take up a worker connection longer
    types_hash_max_size 2048;
    server_tokens off;

    # maximum file upload size
    # update 'upload_max_filesize' & 'post_max_size' in /etc/php/fpm/php.ini accordingly
    client_max_body_size 32m;
    # client_body_timeout 60s; # increase for very long file uploads

    # set default index file (can be overwritten for each site individually)
    index index.html;

    # load MIME types
    include mime.types; # get this file from https://github.com/h5bp/server-configs-nginx
    default_type application/octet-stream; # set default MIME type

    # logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # turn on gzip compression
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 5;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_min_length 256;
    gzip_types
        application/atom+xml
        application/javascript
        application/json
        application/ld+json
        application/manifest+json
        application/rss+xml
        application/vnd.geo+json
        application/vnd.ms-fontobject
        application/x-font-ttf
        application/x-web-app-manifest+json
        application/xhtml+xml
        application/xml
        font/opentype
        image/bmp
        image/svg+xml
        image/x-icon
        text/cache-manifest
        text/css
        text/plain
        text/vcard
        text/vnd.rim.location.xloc
        text/vtt
        text/x-component
        text/x-cross-domain-policy;

    # disable content type sniffing for more security
    add_header "X-Content-Type-Options" "nosniff";

    # force the latest IE version
    add_header "X-UA-Compatible" "IE=Edge";

    # enable anti-cross-site scripting filter built into IE 8+
    add_header "X-XSS-Protection" "1; mode=block";

    # include virtual host configs
    include sites-enabled/*;
}
```

### Конфигурация сайта Grav

Grav поставляется с файлом конфигурации для вашего сайта в каталоге `webserver-configs` вашей установки Grav. Вы можете скопировать этот файл в каталог конфигурации nginx:

```bash
cp /var/www/grav/webserver-configs/nginx.conf /etc/nginx/sites-available/grav-site
```

Откройте этот файл в редакторе и замените `example.com` на свой домен/IP (или `localhost`, если вы хотите просто запустить его локально), замените строку `root` на `root/var/www/grav/;`, а затем создайте символическую ссылку на конфигурацию вашего сайта в `sites-enabled`:

```bash
ln -s /etc/nginx/sites-available/grav-site /etc/nginx/sites-enabled/grav-site
```

<!--
> Рекомендуется использовать файл `expires.conf` от [github.com/h5bp/server-configs-nginx](https://github.com/h5bp/server-configs-nginx) (в каталоге `h5bp/location/`). Он установит заголовки "Expires" для различных типов файлов, чтобы браузер мог кэшировать их. Сохраните файл где-нибудь в вашем каталоге `/etc/nginx/` и включите его в конфигурацию вашего сайта, например, перед первой директивой location в `/etc/nginx/sites-available/grav-site`.
-->

Наконец, позвольте Nginx перезагрузить свою конфигурацию:

```bash
nginx -s reload
```

### Исправление уязвимостей httpoxy

> httpoxy это набор уязвимостей, влияющих на код приложения, работающего в CGI или CGI-подобных средах.
> (Источник: [httpoxy.org](https://httpoxy.org))

Чтобы защитить свой сайт от этой уязвимости, вы должны заблокировать заголовок `Proxy`. Это можно сделать, добавив параметр FastCGI в ваш config. Просто откройте файл `/etc/nginx/fastcgi.conf` и добавьте эту строку в конец:

```nginx
fastcgi_param  HTTP_PROXY         "";
```

### Использование SSL (с существующим сертификатом)

Если вы хотите использовать существующий сертификат SSL для шифрования трафика вашего веб-сайта, в этом разделе приведены необходимые шаги для изменения конфигурации Nginx для этого.

Сначала создайте файл `/etc/nginx/ssl.conf` со следующим содержимым и настройте пути к вашему файлу сертификата и ключа. Последний раздел касается сшивания OSCP и требует, чтобы вы предоставили цепочку + корневой сертификат. Если вы этого не хотите, вы можете прокомментировать или удалить все, что находится под комментарием `OCSP Stapling`. Если ваш веб-сайт поддерживает только SSL (включая субдомены), вы можете отправить свой домен для предварительной загрузки в браузеры по адресу <https://hstspreload.appspot.com>. Если это не так, вы можете удалить `preload;` из строки, которая добавляет заголовок `Strict-Transport-Security`. Обязательно проверьте, работают ли все эти параметры для вашей установки.

**ssl.conf**:

```nginx
# set the paths to your cert and key files here
ssl_certificate /etc/ssl/certs/example.com.crt;
ssl_certificate_key /etc/ssl/private/example.com.key;

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:ECDHE-RSA-DES-CBC3-SHA:ECDHE-ECDSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;
ssl_prefer_server_ciphers on;

ssl_session_cache shared:SSL:10m; # a 1mb cache can hold about 4000 sessions, so we can hold 40000 sessions
ssl_session_timeout 24h;

# Use a higher keepalive timeout to reduce the need for repeated handshakes
keepalive_timeout 300s; # up from 75 secs default

# submit domain for preloading in browsers at: https://hstspreload.appspot.com
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload;";

# OCSP stapling
# nginx will poll the CA for signed OCSP responses, and send them to clients so clients don't make their own OCSP calls.
# see https://sslmate.com/blog/post/ocsp_stapling_in_apache_and_nginx on how to create the chain+root
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/ssl/certs/example.com.chain+root.crt;
resolver 198.51.100.1 198.51.100.2 203.0.113.66 203.0.113.67 valid=60s;
resolver_timeout 2s;
```

Теперь измените содержимое вашей специфической для Grav конфигурации `/etc/nginx/sites-available/grav-site`, чтобы перенаправить незашифрованные HTTP-запросы на HTTPS, что означает, что серверный блок прослушивает порт 443 и включает ваш `ssl.conf` (замените `example.com` на свой домен/IP). Вы также можете изменить это для перенаправления с версии без www на версию вашего домена с www.

**grav-site**:

```nginx
# redirect http to non-www https
server {
    listen [::]:80;
    listen 80;
    server_name example.com www.example.com;

    return 302 https://example.com$request_uri;
}

# redirect www https to non-www https
server {
    listen [::]:443 ssl;
    listen 443 ssl;
    server_name www.example.com;

    # add ssl cert & options
    include ssl.conf;

    return 302 https://example.com$request_uri;
}

# serve website
server {
    listen [::]:443 ssl;
    listen 443 ssl;
    server_name example.com;

    # add ssl cert & options
    include ssl.conf;

    root /var/www/example.com;

    index index.html index.php;

    # ...
    # the rest of this server block (location directives) is identical to the one from the shipped config
}
```

Наконец, перезагрузите конфигурацию Nginx:

```bash
nginx -s reload
```

<!-- TODO: ### Using a Let's Encrypt SSL certificate -->

# Заголовки Nginx Cache для активов

Рекомендуется к включению на продакшене. 'expires' определяет время истечения срока действия кэша, в данном случае 30 дней. Пожалуйста, ознакомьтесь с [полной документацией о http-заголовках для Nginx](http://nginx.org/en/docs/http/ngx_http_headers_module.html).


```nginx
        location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
                expires 30d;
                add_header Vary Accept-Encoding;
                log_not_found off;
        }

        location ~* ^.+\.(?:css|cur|js|jpe?g|gif|htc|ico|png|html|xml|otf|ttf|eot|woff|woff2|svg)$ {
                access_log off;
                expires 30d;
                add_header Cache-Control public;

## No need to bleed constant updates. Send the all shebang in one
## fell swoop.
                tcp_nodelay off;

## Set the OS file cache.
                open_file_cache max=3000 inactive=120s;
                open_file_cache_valid 45s;
                open_file_cache_min_uses 2;
                open_file_cache_errors off;
        }
```