---
title: Подсистема Windows для Linux
metadata:
    description: 'Установка Grav в подсистеме Windows для Linus.'
---
Подсистема Windows для Linux позволяет разработчикам запускать среду GNU/Linux - включая большинство инструментов командной строки, служебных программ и приложений - непосредственно в Windows без изменений, без дополнительных затрат на виртуальную машину.

Вы можете:
- Выбирать свои любимые дистрибутивы GNU/Linux из Магазина Windows.
- Запускать распространенное бесплатное программное обеспечение из командной строки, такое как grep, sed, awk или другие двоичные файлы ELF-64.
- Запускать сценарии оболочки Bash и приложения командной строки GNU/Linux, включая:
  - Инструменты: vim, emacs, tmux
  - Языки: Javascript/Node.js, Ruby, Python, C/C++, C# и F#, Rust, Go и т. д.
  - Сервисы: sshd, MySQL, Apache, lighttpd
- Установить дополнительное программное обеспечение с помощью собственного менеджера пакетов дистрибутива GNU/Linux.
- Вызывать приложения Windows с помощью Unix-подобной оболочки командной строки.
- Вызывать приложения GNU/Linux в Windows.

Для получения дополнительной информации смотрите: [Документация по подсистеме Windows для Linux](https://docs.microsoft.com/en-us/windows/wsl/about)

## Установка подсистемы Windows для Linux
Установка *подсистемы Windows для Linux* хорошо описана в статье Microsoft [Установить подсистему Windows для Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10?target=_blank ).
Вместо стандартного дистрибутива Ubuntu, упомянутого в руководстве по установке, найдите и выберите последнюю версию Ubuntu 18.04 LTS.

Чтобы инициализировать и обновить установку Ubuntu, выполните [Инициализацию вновь установленного дистрибутива](https://docs.microsoft.com/en-us/windows/wsl/initialize-distro?target=_blank).
Этот шаг можно пропустить, если вы уже инициализировали дистрибутив Ubuntu на предыдущем шаге.

! Важным аспектом WSL является то, что **инструменты Windows** **не** могут получить доступ к файлам, хранящимся внутри Ubuntu. Однако Ubuntu может (почти) свободно читать/записывать файловую систему Windows. Следовательно, файлы, к которым требуется доступ для инструментов Windows (например, ваша IDE, резервное копирование), должны храниться в файловой системе Windows.

! При доступе к файловой системе Windows из оболочки bash вам необходимо добавить путь к `/mnt/c/`. Хотя это и не обязательно, при создании символических ссылок лучше всего использовать тот же самый регистр пути к файлу.

## Установка Apache
Используйте следующую команду в оболочке bash для установки Apache:

[prism classes="language-bash command-line"]
sudo apt install apache2
[/prism]

!!! Терминал, используемый WSL, не поддерживает вставку текста, как вы привыкли. Используйте **правую кнопку мыши** для вставки.

Создайте папку проекта для своих веб-сайтов. По причинам, указанным выше, эта папка должна находиться за пределами файловой системы WSL. Вы можете использовать, например: `C:/Users/<Username>/Documents/Development/Web/webroot`, или просто `C:/webroot`.

В Ubuntu создайте символическую ссылку на папку `webroot`.

[prism classes="language-bash command-line"]
sudo ln -s /mnt/c/your/path/to/webroot /var/www/webroot
[/prism]

Откройте файл конфигурации виртуального хоста Apache по умолчанию:

[prism classes="language-bash command-line"]
sudo nano /etc/apache2/sites-available/000-default.conf
[/prism]

!!! Удалите существующий контент, удерживая нажатой клавишу `Shift`, и прокрутите вниз с помощью клавиши `↓`. Затем нажмите `Ctrl`<small>+</small>`K`, чтобы вырезать выделение.

Вставьте следующую конфигурацию VirtualHost:

[prism classes="language-apacheconf line-numbers"]
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
[/prism]

!!! Сохраните файл, нажав `Ctrl`<small>+</small>`O`, и нажмите Enter для подтверждения. Выйдите с помощью `Ctrl`<small>+</small>`X`.

!!! (В командной строке: `^` означает `Ctrl`, а `M` означает `Alt`)

Откройте ваш любимый редактор/IDE в Windows и создайте файл `index.html` в корневой папке со следующим содержимым:

[prism classes="language-html line-numbers"]
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
[/prism]

Запустите службу Apache:

[prism classes="language-bash command-line"]
sudo service apache2 start
[/prism]

!! Вероятно, вы получите следующее известное сообщение об ошибке, [которое можно игнорировать](https://github.com/Microsoft/WSL/issues/1953?target=_blank):

!! *(92)Protocol not available: AH00076: Failed to enable APR_TCP_DEFER_ACCEPT*

Откройте [http://localhost](http://localhost?target=_blank) в вашем браузере, и вы должны увидеть текст 'It works!'.

Для правильной работы ваших будущих Grav-сайтов необходимо включить модуль Apache `rewrite`.

[prism classes="language-bash command-line"]
sudo a2enmod rewrite
[/prism]

## Установка PHP
Используйте следующую команду для установки последней версии PHP:

[prism classes="language-bash command-line"]
sudo apt install php
[/prism]

Чтобы убедиться, что PHP установлен, и проверить его версию, выполните следующую команду:

[prism classes="language-bash command-line"]
php -v
[/prism]
Вы должны получить примерно такой ответ:

[prism classes="language-bash"]
PHP 7.2.7-0ubuntu0.18.04.2 (cli) (built: Jul  4 2018 16:55:24) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
[/prism]

Чтобы соответствовать требованиям Grav к PHP, необходимо установить несколько дополнительных расширений PHP:

[prism classes="language-bash command-line"]
sudo apt install php-mbstring php-gd php-curl php-xml php-zip
[/prism]

Перезапустите Apache, чтобы изменения вступили в силу:

[prism classes="language-bash command-line"]
sudo service apache2 restart
[/prism]

## Установка Grav
Вы можете установить Grav как из Windows, так и из Ubuntu.

#### Вариант 1: Windows
Установите Grav, загрузив ZIP-архив и распаковав его:
1. Загрузите самую последнюю и лучшую версию [**Grav**](https://getgrav.org/download/core/grav/latest?target=_blank) или пакет [**Grav + Admin**](https://getgrav.org/download/core/grav-admin/latest?target=_blank).
1. Распакуйте ZIP-файл в созданный вами ранее веб-корневой каталог.
1. Переименуйте извлеченную папку в `mysite`.
1. Откройте ссылку [http://localhost/mysite](http://localhost/mysite?target=_blank) в браузере, и у вас должна быть работающая установка Grav.

#### Вариант 2: Ubuntu
Выполните следующие команды, чтобы установить Grav в корневой веб-каталог Apache по умолчанию:

[prism classes="language-bash command-line"]
wget -O grav.zip https://getgrav.org/download/core/grav/latest
sudo apt install unzip  # unzip is not installed by default on WSL/Ubuntu
unzip grav.zip -d /var/www/webroot
mv /var/www/webroot/grav /var/www/webroot/mysite
[/prism]

Откройте ссылку [http://localhost/mysite](http://localhost/mysite?target=_blank) в браузере, и у вас должна быть работающая установка Grav.

Чтобы узнать о других вариантах установки, посетите [документацию Grav](/basics/installation?target=_blank).

## Установка XDebug (необязательно)
Если вы разработчик и хотите разрабатывать свои собственные плагины и темы, вам ~~вероятно~~ в какой-то момент неизбежно потребуется отладить свой код...

Установите XDebug с помощью следующей команды:

[prism classes="language-bash command-line"]
sudo apt install php-xdebug
[/prism]

XDebug должен быть включен в `php.ini`.<br>
Откройте редактор:

[prism classes="language-bash command-line"]
sudo nano /etc/php/7.2/apache2/php.ini
[/prism]

И в конец файла добавьте следующие строки:

[prism classes="language-text"]
[XDebug]
xdebug.remote_enable = 1
[/prism]

!!! В Nano вы можете использовать `Alt`<small>+</small>`/` для перехода к нижней части файла.

Снова перезапустите Apache:

[prism classes="language-bash command-line"]
sudo service apache2 restart
[/prism]

#### Активация отладчика
Для того, чтобы начать отладку, вам сначала нужно активировать отладчик на сервере. Для этого нужно установить специальный параметр GET/POST или COOKIE. Вы можете сделать это [вручную](https://xdebug.org/docs/remote?target=_blank#starting), но гораздо удобнее использовать расширение браузера. Это позволяет вам включить отладчик одним нажатием кнопки. Когда расширение активно, оно отправляет cookie XDEBUG_SESSION напрямую, вместо того, чтобы проходить через XDEBUG_SESSION_START. Ниже вы можете найти таблицу со ссылкой на соответствующее расширение для вашего браузера.


| Браузер       | Расширение  |
| ------------- |-------------|-----|
| Chrome        | [Xdebug Helper](https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc?target=_blank) |
| Firefox       | [Xdebug Helper](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/?target=_blank) или [The easiest Xdebug](https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/?target=_blank) |
| Opera         | [Xdebug launcher](https://addons.opera.com/addons/extensions/details/xdebug-launcher/?target=_blank) |

Если вы хотите включить/выключить отладку для веб-сайта, просто включите «Отладка» в расширении браузера.

#### Запуск отладчика в Visual Studio Code (необязательно)
При использовании Vistual Studio Code средства запуска отладки PHP по умолчанию не будут работать, когда Apache/PHP работает в WSL, из-за сопоставлений файлов.

Вставьте следующую конфигурацию в уже созданную [конфигурацию запуска PHP](https://code.visualstudio.com/docs/editor/debugging?target=_blank#_launch-configurations) в `.vscode/launch.json`:

[prism classes="language-json line-numbers"]
{
    "name": "LSW Listen for XDebug",
    "type": "php",
    "request": "launch",
    "port": 9000,
    "pathMappings": {
        "/mnt/c": "c:/",
    }
}
[/prism]

## Добавление дополнительных виртуальных хостов (необязательно)
На разных этапах жизненного цикла нашего сайта (разработка, тестирование, производство) могут потребоваться разные конфигурации Grav. Возьмем, к примеру, кэширование или конвейеры активов. Возможно, вы захотите отключить их во время разработки и включить их при тестировании производительности. Для получения дополнительной информации см. документацию по [автоматической настройке среды](/advanced/environment-config?target=_blank#automatic-environment-configuration).
- Запустите редактор от имени администратора и откройте файл `C:/Windows/System32/drivers/etc/hosts`.
Например, вы можете добавить следующие хосты:

    [prism classes="language-bash"]
    127.0.0.1 mysite-dev
    127.0.0.1 mysite-prod
    [/prism]

    Хосты, определенные в файле hosts Windows, будут автоматически доступны в `/etc/hosts` в WSL/Ubuntu.
- Создайте новые файлы конфигурации VirtualHost в папке `/etc/apache2/sites-available`.
    [prism classes="language-bash command-line"]
    sudo nano /etc/apache2/sites-available/mysite-dev.conf
    [/prism]
    Past the following into the editor:
    [prism classes="language-apacheconf line-numbers"]
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
    [/prism]
Повторите приведенные выше команды для `mysite-prod.conf` и используйте `ServerName mysite-prod` в качестве сервера.

Включите новый VirtualHosts в конфигурации Apache:
[prism classes="language-bash command-line"]
sudo a2ensite mysite-*
sudo service apache2 reload
sudo service apache2 restart
[/prism]
Теперь вы можете указать в браузере [http://mysite-dev](http://mysite-dev?target=_blank) и он откроет установку Grav по адресу `C:/your/path/to/webroot/mysite`, используя файлы конфигурации в папке `/user/mysite-dev/config/`.

## Автоматический запуск Apache (необязательно)
Для запуска и остановки Apache требуются повышенные привилегии. А чтобы получить повышенные привилегии, требуется пароль. Чтобы Ubuntu не запрашивал пароль, вы можете предоставить себе постоянные повышенные привилегии для определенных служб.

Запустите редактор [visudo](http://manpages.ubuntu.com/manpages/trusty/man8/visudo.8.html?target=_blank), чтобы отредактировать файл sudoer:
[prism classes="language-bash command-line"]
sudo visudo -f /etc/sudoers.d/services
[/prism]
Скопируйте в редактор следующие строки:
[prism classes="language-bash"]
%sudo ALL=(root) NOPASSWD: /usr/sbin/service *
%wheel ALL=(root) NOPASSWD: /usr/sbin/service *
[/prism]

Теперь Apache можно запустить с повышенными привилегиями без ввода пароля.

Чтобы запускать Apache всякий раз, когда запускается оболочка Ubuntu, необходимо добавить команду `sudo service apache2 start` в сценарий запуска `.bashrc`. Этот сценарий запускается всякий раз, когда вы запускаете терминал WSL.
[prism classes="language-bash command-line"]
nano .bashrc
[/prism]
Добавьте в конец файла следующий скрипт:
[prism classes="language-apacheconf line-numbers"]
## Start apache2 if not running
status=`service apache2 status`
if [[ $status == *"apache2 is not running" ]]
then
  sudo service apache2 start
fi
[/prism]
И добавьте следующее в `.bash_logout`, чтобы остановить Apache при закрытии оболочки bash.
[prism classes="language-apacheconf line-numbers"]
## Stop apache2 if running
status=`service apache2 status`
if [[ $status == *"apache2 is running" ]]
then
  sudo service apache2 stop
fi
[/prism]

## Советы и рекомендации

### Графический эмулятор терминала Linux
Если вы не являетесь поклонником терминала по умолчанию и хотели бы установить «собственный» терминал с графическим интерфейсом пользователя Linux, вы можете прочитать статью [Настройка красивого и удобного эмулятора терминала для WSL](https://blog.ropnop.com/configuring-a-pretty-and-usable-terminal-emulator-for-wsl/?target=_blank).

### Несколько веб-сайтов, одна кодовая база Grav
Если вы похожи на меня и у вас несколько веб-сайтов Grav, развернутых для отдельных проектов, вы можете прочитать документацию по [символическим ссылкам](/cli-console/command-line-intro#символические-ссылки) и о [копировании проекта](/cli-console/grav-cli#copying-a-project) для создания символьной копии одного ядра Grav.