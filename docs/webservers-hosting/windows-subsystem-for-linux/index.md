---
description: Установка Grav в подсистеме Windows для Linus.
---

# Подсистема Windows для Linux

Подсистема Windows для Linux позволяет разработчикам запускать среду GNU/Linux - включая большинство инструментов командной строки, служебных программ и приложений - непосредственно в Windows без изменений, без дополнительных затрат на виртуальную машину.

Вы можете:

* Выбирать свои любимые дистрибутивы GNU/Linux из Магазина Windows.
* Запускать распространенное бесплатное программное обеспечение из командной строки, такое как grep, sed, awk или другие двоичные файлы ELF-64.
* Запускать сценарии оболочки Bash и приложения командной строки GNU/Linux, включая:
	* Инструменты: vim, emacs, tmux
	* Языки: Javascript/Node.js, Ruby, Python, C/C++, C# и F#, Rust, Go и т. д.
	* Сервисы: sshd, MySQL, Apache, lighttpd
* Установить дополнительное программное обеспечение с помощью собственного менеджера пакетов дистрибутива GNU/Linux.
* Вызывать приложения Windows с помощью Unix-подобной оболочки командной строки.
* Вызывать приложения GNU/Linux в Windows.

Для получения дополнительной информации смотрите [Документацию по подсистеме Windows для Linux](https://docs.microsoft.com/en-us/windows/wsl/about).

## Установка подсистемы Windows для Linux
Установка *подсистемы Windows для Linux* хорошо описана в статье Microsoft [Установить подсистему Windows для Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
Вместо стандартного дистрибутива Ubuntu, упомянутого в руководстве по установке, найдите и выберите последнюю версию Ubuntu 18.04 LTS.

Чтобы инициализировать и обновить установку Ubuntu, выполните [Инициализацию вновь установленного дистрибутива](https://docs.microsoft.com/en-us/windows/wsl/initialize-distro).
Этот шаг можно пропустить, если вы уже инициализировали дистрибутив Ubuntu на предыдущем шаге.

!!! note ""

	Важным аспектом WSL является то, что **инструменты Windows** **не** могут получить доступ к файлам, хранящимся внутри Ubuntu. Однако Ubuntu может (почти) свободно читать/записывать файловую систему Windows. Следовательно, файлы, к которым требуется доступ для инструментов Windows (например, ваша IDE, резервное копирование), должны храниться в файловой системе Windows.

!!! note ""

	При доступе к файловой системе Windows из оболочки bash вам необходимо добавить путь к `/mnt/c/`. Хотя это и не обязательно, при создании символических ссылок лучше всего использовать тот же самый регистр пути к файлу.

## Установка Apache
Используйте следующую команду в оболочке bash для установки Apache:

```bash
sudo apt install apache2
```

!!! tip ""

	Терминал, используемый WSL, не поддерживает вставку текста, как вы привыкли. Используйте **правую кнопку мыши** для вставки.

Создайте папку проекта для своих веб-сайтов. По причинам, указанным выше, эта папка должна находиться за пределами файловой системы WSL. Вы можете использовать, например: `C:/Users/<Username>/Documents/Development/Web/webroot`, или просто `C:/webroot`.

В Ubuntu создайте символическую ссылку на папку `webroot`.

```bash
sudo ln -s /mnt/c/your/path/to/webroot /var/www/webroot
```

Откройте файл конфигурации виртуального хоста Apache по умолчанию:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

!!! tip ""

	Удалите существующий контент, удерживая нажатой клавишу ++shift++, и прокрутите вниз с помощью клавиши ++down++. Затем нажмите ++ctrl+k++, чтобы вырезать выделение.

Вставьте следующую конфигурацию VirtualHost:

```apacheconf
<VirtualHost *:80>

    ServerName localhost

    ServerAdmin webmaster@localhost
    DocumentRoot  /var/www/webroot

    <Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

!!! info ""

	Сохраните файл, нажав ++ctrl+o++, и нажмите Enter для подтверждения. Выйдите с помощью ++ctrl+x++.

	(В командной строке `^` означает ++ctrl++, а `M` означает ++alt++)

Откройте ваш любимый редактор/IDE в Windows и создайте файл `index.html` в корневой папке со следующим содержимым:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>It works!</title>
</head>
<body>
  <h1>It works!</h1>
</body>
</html>
```

Запустите службу Apache:

```bash
sudo service apache2 start
```

> Вероятно, вы получите следующее известное сообщение об ошибке, [которое можно игнорировать](https://github.com/Microsoft/WSL/issues/1953):

> *(92)Protocol not available: AH00076: Failed to enable APR_TCP_DEFER_ACCEPT*

Откройте [http://localhost](http://localhost) в вашем браузере, и вы должны увидеть текст 'It works!'.

Для правильной работы ваших будущих Grav-сайтов необходимо включить модуль Apache `rewrite`.

```bash
sudo a2enmod rewrite
```

## Установка PHP
Используйте следующую команду для установки последней версии PHP:

```bash
sudo apt install php
```

Чтобы убедиться, что PHP установлен, и проверить его версию, выполните следующую команду:

```bash
php -v
```
Вы должны получить примерно такой ответ:

```bash
PHP 7.2.7-0ubuntu0.18.04.2 (cli) (built: Jul  4 2018 16:55:24) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```

Чтобы соответствовать требованиям Grav к PHP, необходимо установить несколько дополнительных расширений PHP:

```bash
sudo apt install php-mbstring php-gd php-curl php-xml php-zip
```

Перезапустите Apache, чтобы изменения вступили в силу:

```bash
sudo service apache2 restart
```

## Установка Grav
Вы можете установить Grav как из Windows, так и из Ubuntu.

### Вариант 1: Windows

Установите Grav, загрузив ZIP-архив и распаковав его:
1. Загрузите самую последнюю и лучшую версию [**Grav**](https://getgrav.org/download/core/grav/latest) или пакет [**Grav + Admin**](https://getgrav.org/download/core/grav-admin/latest).
1. Распакуйте ZIP-файл в созданный вами ранее веб-корневой каталог.
1. Переименуйте извлеченную папку в `mysite`.
1. Откройте ссылку [http://localhost/mysite](http://localhost/mysite) в браузере, и у вас должна быть работающая установка Grav.

### Вариант 2: Ubuntu

Выполните следующие команды, чтобы установить Grav в корневой веб-каталог Apache по умолчанию:

```bash
wget -O grav.zip https://getgrav.org/download/core/grav/latest
sudo apt install unzip  # unzip is not installed by default on WSL/Ubuntu
unzip grav.zip -d /var/www/webroot
mv /var/www/webroot/grav /var/www/webroot/mysite
```

Откройте ссылку [http://localhost/mysite](http://localhost/mysite) в браузере, и у вас должна быть работающая установка Grav.

Чтобы узнать о других вариантах установки, посетите [документацию Grav](/basics/installation).

## Установка XDebug (необязательно)

Если вы разработчик и хотите разрабатывать свои собственные плагины и темы, вам вероятно в какой-то момент неизбежно потребуется отладить свой код...

Установите XDebug с помощью следующей команды:

```bash
sudo apt install php-xdebug
```

XDebug должен быть включен в `php.ini`.<br>
Откройте редактор:

```bash
sudo nano /etc/php/7.2/apache2/php.ini
```

И в конец файла добавьте следующие строки:

```text
[XDebug]
xdebug.remote_enable = 1
```

!!! tip ""

	В Nano вы можете использовать ++alt+slash++ для перехода к нижней части файла.

Снова перезапустите Apache:

```bash
sudo service apache2 restart
```

### Активация отладчика
Для того, чтобы начать отладку, вам сначала нужно активировать отладчик на сервере. Для этого нужно установить специальный параметр GET/POST или COOKIE. Вы можете сделать это [вручную](https://xdebug.org/docs/remote#starting), но гораздо удобнее использовать расширение браузера. Это позволяет вам включить отладчик одним нажатием кнопки. Когда расширение активно, оно отправляет cookie XDEBUG_SESSION напрямую, вместо того, чтобы проходить через XDEBUG_SESSION_START. Ниже вы можете найти таблицу со ссылкой на соответствующее расширение для вашего браузера.

| Браузер       | Расширение  |
| ------------- | ----------- |
| Chrome        | [Xdebug Helper](https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc) |
| Firefox       | [Xdebug Helper](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/) или [The easiest Xdebug](https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/) |
| Opera         | [Xdebug launcher](https://addons.opera.com/addons/extensions/details/xdebug-launcher/) |

Если вы хотите включить/выключить отладку для веб-сайта, просто включите «Отладка» в расширении браузера.

### Запуск отладчика в Visual Studio Code (необязательно)
При использовании Vistual Studio Code средства запуска отладки PHP по умолчанию не будут работать, когда Apache/PHP работает в WSL, из-за сопоставлений файлов.

Вставьте следующую конфигурацию в уже созданную [конфигурацию запуска PHP](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations) в `.vscode/launch.json`:

```json
{
    "name": "LSW Listen for XDebug",
    "type": "php",
    "request": "launch",
    "port": 9000,
    "pathMappings": {
        "/mnt/c": "c:/",
    }
}
```

## Добавление дополнительных виртуальных хостов (необязательно)
На разных этапах жизненного цикла нашего сайта (разработка, тестирование, производство) могут потребоваться разные конфигурации Grav. Возьмем, к примеру, кэширование или конвейеры активов. Возможно, вы захотите отключить их во время разработки и включить их при тестировании производительности. Для получения дополнительной информации см. документацию по [автоматической настройке среды](/advanced/environment-config/#avtomaticheskaia-nastroika-okruzheniia):

* Запустите редактор от имени администратора и откройте файл `C:/Windows/System32/drivers/etc/hosts`.
	Например, вы можете добавить следующие хосты:

	```bash
	127.0.0.1 mysite-dev
	127.0.0.1 mysite-prod
	```

	Хосты, определённые в файле hosts Windows, будут автоматически доступны в `/etc/hosts` в WSL/Ubuntu.
* Создайте новые файлы конфигурации VirtualHost в папке `/etc/apache2/sites-available`.
    ```bash
    sudo nano /etc/apache2/sites-available/mysite-dev.conf
    ```
    Вставьте следующий код в редактор:
    ```apacheconf
    <VirtualHost *:80>

        ServerName mysite-dev

        ServerAdmin webmaster@localhost
        DocumentRoot  /var/www/webroot/mysite

        <Directory /var/www/>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

    </VirtualHost>
    ```
Повторите приведенные выше команды для `mysite-prod.conf` и используйте `ServerName mysite-prod` в качестве сервера.

Включите новый VirtualHosts в конфигурации Apache:
```bash
sudo a2ensite mysite-*
sudo service apache2 reload
sudo service apache2 restart
```
Теперь вы можете указать в браузере [http://mysite-dev](http://mysite-dev) и он откроет установку Grav по адресу `C:/your/path/to/webroot/mysite`, используя файлы конфигурации в папке `/user/mysite-dev/config/`.

## Автоматический запуск Apache (необязательно)
Для запуска и остановки Apache требуются повышенные привилегии. А чтобы получить повышенные привилегии, требуется пароль. Чтобы Ubuntu не запрашивал пароль, вы можете предоставить себе постоянные повышенные привилегии для определённых служб.

Запустите редактор [visudo](http://manpages.ubuntu.com/manpages/trusty/man8/visudo.8.html), чтобы отредактировать файл sudoer:

```bash
sudo visudo -f /etc/sudoers.d/services
```

Скопируйте в редактор следующие строки:
```bash
%sudo ALL=(root) NOPASSWD: /usr/sbin/service *
%wheel ALL=(root) NOPASSWD: /usr/sbin/service *
```

Теперь Apache можно запустить с повышенными привилегиями без ввода пароля.

Чтобы запускать Apache всякий раз, когда запускается оболочка Ubuntu, необходимо добавить команду `sudo service apache2 start` в сценарий запуска `.bashrc`. Этот сценарий запускается всякий раз, когда вы запускаете терминал WSL.

```bash
nano .bashrc
```

Добавьте в конец файла следующий скрипт:

```apacheconf
## Start apache2 if not running
status=`service apache2 status`
if [[ $status == *"apache2 is not running" ]]
then
  sudo service apache2 start
fi
```

И добавьте следующее в `.bash_logout`, чтобы остановить Apache при закрытии оболочки bash:

```apacheconf
## Stop apache2 if running
status=`service apache2 status`
if [[ $status == *"apache2 is running" ]]
then
  sudo service apache2 stop
fi
```

## Советы и рекомендации

### Графический эмулятор терминала Linux
Если вы не являетесь поклонником терминала по умолчанию и хотели бы установить «собственный» терминал с графическим интерфейсом пользователя Linux, вы можете прочитать статью [Настройка красивого и удобного эмулятора терминала для WSL](https://blog.ropnop.com/configuring-a-pretty-and-usable-terminal-emulator-for-wsl/).

### Несколько веб-сайтов, одна кодовая база Grav
Если вы похожи на меня и у вас несколько веб-сайтов Grav, развернутых для отдельных проектов, вы можете прочитать документацию по [символическим ссылкам](/cli-console/command-line-intro/#simvolicheskie-ssylki) и о [копировании проекта](/cli-console/grav-cli/#sandbox-pesochnitsa) для создания символьной копии одного ядра Grav.