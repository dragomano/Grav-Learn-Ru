# Установка VPS на Ubuntu 18.04

> Руководство по установке VPS на Ubuntu.

### Пакеты обновлений

На этом этапе вы можете либо настроить локальную запись `/etc/hosts`, чтобы дать IP-адресу удобное понятное имя, такое как `{{ page.header.localname }}`. Так вам будет проще подключиться к серверу по SSH с помощью `ssh root@{{ page.header.localname }}{% if ssh_port %} -p{{ ssh_port }}{% endif %}`.

После успешного подключения по SSH к вашему серверу как **root** первое, что вам нужно сделать, это обновить и обновить все установленные пакеты. Это гарантирует, что вы используете _самую свежую версию_:

```bash
# apt update
# apt upgrade
```

Просто ответьте `Y`, если будет предложено.

Прежде чем идти дальше, удалим **Apache2**, который мы заменим **Nginx**:

```bash
# apt remove apache2*
# apt autoremove
```

> ПРИМЕЧАНИЕ. Возможно, это не установлено. Но лучше перестраховаться!

Далее вам нужно будет установить несколько важных пакетов:

```bash
# apt install vim zip unzip nginx git php-fpm php-cli php-gd php-curl php-mbstring php-xml php-zip php-apcu
```

Это установит полный редактор VIM (а не мини-версию, которая поставляется с Ubuntu), веб-сервер Nginx, команды GIT и **PHP 7.2**.

### Конфигурация PHP7.2 FPM
После установки php-fpm необходимо внести небольшое изменение в конфигурацию для более безопасной установки.


```bash
# vim /etc/php/7.2/fpm/php.ini
```

Найдите `cgi.fix_pathinfo`. По умолчанию это будет закомментировано и установлено значение «1».

Это крайне небезопасный параметр, поскольку он сообщает PHP, что нужно попытаться выполнить ближайший файл, который он может найти, если запрошенный файл PHP не может быть найден. Это в основном позволит пользователям создавать PHP-запросы таким образом, чтобы они могли выполнять скрипты, которые им не разрешалось выполнять.

Раскомментируйте эту строку и измените «1» на «0», чтобы она выглядела так


```bash
cgi.fix_pathinfo=0
```

Сохраните и закройте файл, а затем перезапустите службу.


```bash
# systemctl restart php7.2-fpm
```

### Настройка пула подключений Nginx

Nginx уже установлен, но вы должны настроить его так, чтобы он использовал пул соединений PHP для конкретного пользователя. Это обеспечит вашу безопасность и позволит избежать любых потенциальных разрешений на файлы при работе с файлами в качестве учетной записи пользователя и через веб-сервер.

Перейдите в каталог пула и создайте новую конфигурацию `grav`:


```bash
# cd /etc/php/7.2/fpm/pool.d
# mv www.conf www.conf.bak
# vim grav.conf
```

В Vim вы можете вставить следующую конфигурацию пула:

```apacheconf
[grav]

user = grav
group = grav

listen = /var/run/php/php7.2-fpm.sock

listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

chdir = /
```

Ключевыми вещами здесь являются `user` и `group`, устанавливаемые для пользователя с именем `grav`, и сокет прослушивания, имеющий уникальное имя из стандартного сокета. Сохраните и выйдите из этого файла.

Теперь нам нужно создать специального пользователя `grav`:

```bash
# adduser grav
```

Задайте надежный пароль и оставьте другие значения по умолчанию. Затем нам нужно создать подходящее место для Nginx для обслуживания файлов, поэтому давайте сменим пользователя и создадим эту папку, а также создадим пару тестовых файлов:

```bash
# su - grav
$ mkdir -p www/html
$ cd www/html
```

Создайте простой `index.html` с содержимым:

```
 <h1>Working!</h1>
```

..и файл с именем `info.php` с содержимым:

```php
<?php phpinfo();
```

Теперь мы можем выйти из этого пользователя и вернуться к root, чтобы настроить конфигурацию сервера Nginx:

```bash
$ exit
# cd /etc/nginx/sites-available/
# vim grav
```

Затем просто вставьте эту конфигурацию:

```nginx
server {
    #listen 80;
    index index.html index.php;

    ## Begin - Server Info
    root /home/grav/www/html;
    server_name localhost;
    ## End - Server Info

    ## Begin - Index
    # for subfolders, simply adjust:
    # `location /subfolder {`
    # and the rewrite to use `/subfolder/index.php`
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    ## End - Index

    ## Begin - Security
    # deny all direct access for these folders
    location ~* /(\.git|cache|bin|logs|backup|tests)/.*$ { return 403; }
    # deny running scripts inside core system folders
    location ~* /(system|vendor)/.*\.(txt|xml|md|html|yaml|yml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny running scripts inside user folder
    location ~* /user/.*\.(txt|md|yaml|yml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny access to specific files in the root folder
    location ~ /(LICENSE\.txt|composer\.lock|composer\.json|nginx\.conf|web\.config|htaccess\.txt|\.htaccess) { return 403; }
    ## End - Security

    ## Begin - PHP
    location ~ \.php$ {
        # Choose either a socket or TCP/IP address
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        # fastcgi_pass unix:/var/run/php5-fpm.sock; #legacy
        # fastcgi_pass 127.0.0.1:9000;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }
    ## End - PHP
}
```

Это стандартный файл `nginx.conf`, который поставляется с Grav с двумя изменениями. 1) `root` был адаптирован к нашему пользователю/папке, которую мы только что создали, а опция fastcgi_pass была установлена ​​на сокет, который мы определили в нашем пуле `grav`. Теперь нам просто нужно правильно связать этот файл, чтобы он **был включен**:

```bash
# cd ../sites-enabled
# ln -s ../sites-available/grav
# rm default
```

Вы можете проверить конфигурацию с помощью команды `nginx -t`. Он должен вернуть следующее.

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Теперь все, что нам нужно сделать, это перезапустить Nginx и процесс php7-fpm и протестировать, чтобы убедиться, что мы правильно настроили Nginx и пул соединений PHP:

```bash
# systemctl restart nginx
# systemctl restart php7.2-fpm
```

Теперь укажите в браузере свой сервер: `http://{{page.header.localname}}`, и вы должны увидеть текст: **Working!**

Вы также можете проверить, что PHP установлен и правильно работает, указав в браузере: `http://{{page.header.localname}}/info.php`. Вы должны увидеть стандартную информационную страницу PHP с перечисленными APCu, Opcache и т. д.

### Установка Grav

Это легкая часть! Сначала нам нужно вернуться к пользователю Grav, поэтому либо SSH как `grav@{{page.header.localname}}`, либо `su - grav` от имени пользователя root, затем выполните следующие действия:

```bash
$ cd ~/www
$ wget -O grav.zip https://getgrav.org/download/core/grav/latest
$ unzip grav.zip
$ rm -Rf html
$ mv grav html
```

Теперь, когда все готово, вы можете подтвердить установку Grav, указав в браузере ссылку `http://{{page.header.localname}}`, и вы должны увидеть страницу **Grav is Running!**.

Поскольку вы тщательно следовали этим инструкциям, вы также сможете использовать такие команды, как:

```bash
$ cd ~/www/html
$ bin/grav clear

Clearing cache

Cleared:  cache/twig/*
Cleared:  cache/compiled/*

Touched: /home/grav/www/html/user/config/system.yaml
```

и команды GPM:

```bash
$ bin/gpm index
```
