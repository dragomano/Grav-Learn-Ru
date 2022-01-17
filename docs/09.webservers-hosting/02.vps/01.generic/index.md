# Универсальная установка VPS

> Установка Grav на хостинге VPS.

### Пакеты обновлений и улучшений

На этом этапе вы можете захотеть либо настроить локальную запись `/etc/hosts`, чтобы дать IP-адресу удобное понятное имя, такое как `{{page.header.localname}}`. Так вам будет проще подключиться к серверу по SSH с помощью `ssh root@{{ page.header.localname }}`.

После успешного подключения по SSH к вашему серверу как **root** первое, что вам нужно сделать, это обновить и обновить все установленные пакеты. Это гарантирует, что вы используете самые последние и самые лучшие:

```
$ apt-get update
$ apt-get upgrade
```

Просто ответьте `Y`, если будет предложено.

Далее вам нужно будет установить несколько важных пакетов:

```
$ apt-get install vim zip unzip nginx git php5-fpm php5-cli php5-gd php5-curl php5-apcu
```

При этом будет установлен полный редактор VIM (а не мини-версия, которая поставляется с Ubuntu), веб-сервер Nginx, команды GIT и **PHP 5.5**.

> Если вы предпочитаете использовать **PHP 5.6**, вам, вероятно, следует начать заново и настроить свой сервер **Ubuntu 15.10**, который является наиболее современной версией Ubuntu.

### Настройка пула подключений Nginx

Nginx уже установлен, но вы должны настроить его так, чтобы он использовал пул соединений PHP для конкретного пользователя. Это обеспечит вашу безопасность и позволит избежать любых потенциальных разрешений на файлы при работе с файлами в качестве вашей учетной записи пользователя и через веб-сервер.

Перейдите в каталог пула и создайте новую конфигурацию `grav`:

```
$ cd /etc/php5/fpm/pool.d
$ vi grav.conf
```

В VIM вы можете вставить следующую конфигурацию пула:

```
[grav]

user = grav
group = grav

listen = /var/run/php5-fpm.grav.sock

listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

chdir = /
```

Ключевыми вещами здесь являются `user` и `group`, устанавливаемые на пользователя с именем `grav`, а также прослушивающий сокет, имеющий уникальное имя из стандартного сокета. Сохраните и выйдите из этого файла.

Теперь нам нужно создать специального пользователя `grav`:

```
$ adduser grav
```

Задайте надежный пароль и оставьте другие значения по умолчанию. Затем нам нужно создать подходящее место для Nginx для обслуживания файлов, поэтому давайте сменим пользователя и создадим эту папку, а также создадим пару тестовых файлов:

```
$ su - grav
$ mkdir www;cd www;mkdir html;cd html
```

Создайте простой `index.html` с содержимым `<h1> Working! </h1>` и файл с именем ʻinfo.php` с содержимым `<?php phpinfo();`

Теперь мы можем выйти из этого пользователя и вернуться к root, чтобы настроить конфигурацию сервера Nginx:

```
$ exit
$ cd /etc/nginx/sites-available/
$ vi grav
```

Затем просто вставьте эту конфигурацию:

```
server {
    #listen 80;
    index index.html index.php;

    ## Begin - Server Info
    root /home/USER/www/html;
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

Это стандартный файл `nginx.conf`, который поставляется с Grav с двумя изменениями: 1) `root` был адаптирован к нашему пользователю/папке, которую мы только что создали, а опция `fastcgi_pass` была установлена ​​на сокет, который мы определили в нашем пуле `grav`. Теперь нам просто нужно правильно связать этот файл, чтобы он **был включен**:

```
$ cd ../sites-enabled
$ ln -s ../sites-available/grav
$ rm default
```

Теперь все, что нам нужно сделать, это перезапустить Nginx и процесс php5-fpm и протестировать, чтобы убедиться, что мы правильно настроили Nginx и пул соединений PHP:

```
$ service nginx restart
$ service php5-fpm restart
```

Теперь укажите в браузере свой сервер: `http://{{page.header.localname}}`, и вы должны увидеть текст: **Working!**

Вы также можете проверить, что PHP установлен и правильно работает, указав в своем браузере: `http://{{page.header.localname}}/info.php`. Вы должны увидеть стандартную информационную страницу PHP с перечисленными APCu, Opcache и т. д.

### Установка Grav

Это легкая часть! Сначала нам нужно вернуться к пользователю Grav, поэтому либо SSH как `grav@{{page.header.localname}}`, либо `su - grav` от имени пользователя root. Затем выполните следующие действия:

```
$ cd ~/www
$ wget https://getgrav.org/download/core/grav/latest
$ unzip grav-v{{ grav_version }}.zip
$ rm -Rf html
$ mv grav html
```

Теперь, когда все готово, вы можете подтвердить установку Grav, указав в браузере страницу `http://{{page.header.localname}}`, и вы должны увидеть страницу **Grav is Running!**.

Поскольку вы тщательно следовали этим инструкциям, вы также сможете использовать [Grav CLI](/07.cli-console/02.grav-cli/index) и [Grav GPM](/07.cli-console/04.grav-cli-gpm/index) такие команды, как:

```
$ cd ~/www/html
$ bin/grav clear

Clearing cache

Cleared:  cache/twig/*
Cleared:  cache/compiled/*

Touched: /home/grav/www/html/user/config/system.yaml
```

и команды GPM:

```
$ bin/gpm index
```
