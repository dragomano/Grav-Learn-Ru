---
description: Руководство по установке VPS на Ubuntu 24.04 LTS.
---

# Установка VPS на Ubuntu 24.04 LTS

### Первоначальная настройка сервера

Сначала добавьте запись в локальный файл `/etc/hosts`, чтобы назначить IP-адресу вашего сервера понятное имя, например `<< page.header.localname|default('myserver.local') >>`. Это упростит подключение к серверу по SSH:

```bash
ssh root@<< page.header.localname|default('myserver.local') >><% if ssh_port %> -p<< ssh_port >><% endif %>
```

### Обновление системных пакетов

После подключения под пользователем **root** обновите все установленные пакеты:

```bash
apt update && apt upgrade -y
```

### Установка необходимых пакетов

Установите Nginx, PHP 8.3 и необходимые для работы Grav расширения:

```bash
apt install -y vim zip unzip nginx git \
    php8.3-fpm php8.3-cli php8.3-gd php8.3-curl \
    php8.3-mbstring php8.3-xml php8.3-zip php8.3-intl php8.3-apcu
```

Будут установлены:
- **Nginx** — высокопроизводительный веб-сервер.
- **PHP 8.3-FPM** — менеджер процессов FastCGI для PHP.
- **Расширения PHP** — необходимы Grav для обработки изображений, кэширования и других функций.

### Настройка PHP-FPM

Измените конфигурацию PHP для повышения безопасности:

```bash
vim /etc/php/8.3/fpm/php.ini
```

Найдите параметр `cgi.fix_pathinfo` (для поиска в Vim используйте `/cgi.fix_pathinfo`), раскомментируйте его и установите значение `0`:

```ini
cgi.fix_pathinfo=0
```

!!! warning
    Этот параметр запрещает PHP выполнять ближайший подходящий файл, если запрошенный файл не найден. Если оставить его включённым, это создаёт серьёзную угрозу безопасности.

### Создание отдельного пользователя

Создайте пользователя `grav`, от имени которого будет работать сайт (не запускайте веб-приложения от имени `root`):

```bash
adduser grav
```

При появлении запроса задайте надёжный пароль.

### Настройка пула PHP-FPM

Создайте отдельный пул PHP-FPM для пользователя `grav`:

```bash
cd /etc/php/8.3/fpm/pool.d
mv www.conf www.conf.bak
vim grav.conf
```

Добавьте следующую конфигурацию:

```ini
[grav]
user = grav
group = grav

listen = /run/php/php8.3-fpm-grav.sock

listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 10
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 5

chdir = /
```

### Создание веб-каталога

Переключитесь на пользователя `grav` и создайте веб-каталог:

```bash
su - grav
mkdir -p ~/www/html
```

Создайте тестовый файл, чтобы проверить правильность настройки:

```bash
echo '<?php phpinfo();' > ~/www/html/info.php
exit
```

### Настройка Nginx

Создайте блок `server` для Nginx:

```bash
vim /etc/nginx/sites-available/grav
```

Добавьте следующую конфигурацию:

```nginx
server {
    listen 80;
    index index.html index.php;

    ## Begin - Server Info
    root /home/grav/www/html;
    server_name _;
    ## End - Server Info

    ## Begin - Index
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    ## End - Index

    ## Begin - Security
    # deny all direct access for these folders
    location ~* /(\.git|cache|bin|logs|backup|tests)/.*$ { return 403; }
    # deny all direct access to these sensitive user folders, whatever the file type
    location ~* /user/(accounts|config|env)/.*$ { return 403; }
    # allow public media uploads under user/data (e.g. Flex Object images) to be
    # served directly; this must come before the user/data deny so it wins the
    # first-match. SVG is intentionally excluded as a stored-XSS vector.
    location ~* /user/data/.*\.(jpe?g|png|gif|webp|avif|bmp|ico|mp4|webm|ogg|ogv|mov|mp3|wav|m4a|flac|pdf)$ { try_files $uri =404; }
    # deny everything else under user/data, whatever the file type
    location ~* /user/data/.*$ { return 403; }
    # deny running scripts inside core system folders
    location ~* /(system|vendor)/.*\.(txt|xml|md|html|htm|shtml|shtm|json|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny running scripts inside user folder
    location ~* /user/.*\.(txt|md|json|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny access to specific files in the root folder
    location ~ /(LICENSE\.txt|composer\.lock|composer\.json|nginx\.conf|web\.config|htaccess\.txt|\.htaccess) { return 403; }
    # deny access to .env environment files
    location ~ /\.env(\.|$) { return 403; }
    ## End - Security

    ## Begin - PHP
    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.3-fpm-grav.sock;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }
    ## End - PHP
}
```

Включите сайт и отключите конфигурацию по умолчанию:

```bash
ln -s /etc/nginx/sites-available/grav /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
```

Протестируйте конфигурацию:

```bash
nginx -t
```

Вы должны увидеть:

```txt
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Запуск служб

Перезапустите Nginx и PHP-FPM:

```bash
systemctl restart nginx
systemctl restart php8.3-fpm
```

Убедитесь, что PHP работает, открыв в браузере страницу `http://YOUR_SERVER_IP/info.php`. Должна отобразиться информационная страница PHP, на которой указаны PHP 8.3 и APCu.

!!! warning
    После проверки удалите файл `info.php`: `rm /home/grav/www/html/info.php`

### Установка Grav

Переключитесь на пользователя `grav` и загрузите Grav:

```bash
su - grav
cd ~/www
wget -O grav.zip https://getgrav.org/download/core/grav/latest
unzip grav.zip
rm -rf html
mv grav html
```

### Проверка установки

Откройте в браузере страницу `http://YOUR_SERVER_IP`. Если установка выполнена успешно, вы увидите страницу **Grav is Running!**.

### Проверка инструментов командной строки

Поскольку вы работаете от имени пользователя `grav`, инструменты командной строки будут работать сразу, без дополнительной настройки:

```bash
cd ~/www/html
bin/grav clear
```

Вывод:

```txt
Clearing cache

Cleared:  cache/twig/*
Cleared:  cache/compiled/*

Touched: /home/grav/www/html/user/config/system.yaml
```

Команды GPM также будут работать:

```bash
bin/gpm index
```

### Необязательно: установка плагина админки

Чтобы установить панель администрирования Grav:

```bash
bin/gpm install admin
```

Затем откройте в браузере страницу `http://YOUR_SERVER_IP/admin`, чтобы создать учётную запись администратора.

### Необязательно: включение HTTPS с помощью Let's Encrypt

Для рабочих сайтов рекомендуется включить HTTPS с помощью Certbot:

```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d yourdomain.com
```

Certbot автоматически настроит Nginx для работы с SSL и включит автоматическое продление сертификатов.

### Следующие шаги

- [Настройка вашего сайта](/basics/grav-configuration)
- [Установка тем](/themes)
- [Добавление плагинов](/plugins)
- [Создание контента](/content)
