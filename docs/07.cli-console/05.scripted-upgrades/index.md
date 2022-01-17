# Обновления по сценарию

> Руководство по пакетному обновлению нескольких сайтов на базе Grav CMS.

Или: Одновременное обновление нескольких Grav-установок.

Это руководство по упрощению обновления нескольких Grav-установок с помощью [Deployer](https://deployer.org/). Под несколькими я подразумеваю отдельные установки, а не [многосайтовую установку](/08.advanced/05.multisite-setup/index). Мы будем использовать путь к каждой установке для запуска команд [CLI Grav](/07.cli-console/02.grav-cli/index), но без необходимости вводить каждую из них вручную.

Преимущество такого средства выполнения задач, как Deployer, заключается в том, что, выполняя свои задачи, он позволяет вам точно знать, что он делает в процессе. Он также покажет вам любой вывод сервера из выполняемых команд. Например, это результат работы Deployer:

```text
Executing task packages

GPM Releases Configuration: Stable

Found 8 packages installed of which 1 need updating

01. Email           [v2.5.2 -> v2.5.3]

GPM Releases Configuration: Stable

Preparing to install Email [v2.5.3]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok
  '- Success!

Clearing cache

Cleared:  /home/username/public_html/deployer/grav/cache/twig/*
Cleared:  /home/username/public_html/deployer/grav/cache/doctrine/*
Cleared:  /home/username/public_html/deployer/grav/cache/compiled/*

Touched: /home/username/public_html/deployer/grav/user/config/system.yaml
```

И так просто: Deployer сказал Grav обновить все пакеты, что обновило плагин Email до последней версии.

## Предварительные требования

Как и в случае с Grav, вам потребуется PHP **v7.3.6** или выше. Это также относится к командной строке (CLI), поэтому, если у вас установлено несколько версий, используйте ту, которая относится к нужной версии. Используйте команду `php -v`, чтобы проверить версию по умолчанию, моя - **PHP 7.2.34**.

В общих средах уточните у своего хоста, какую команду использовать для CLI. В моем случае это `php74`, которая с `-v` возвращает **PHP 7.4.12**. Это также означает добавление каждого пути следующим образом: `php74 vendor/bin/dep list`.

Некоторые хосты также позволяют вам выбрать версию PHP по умолчанию для использования в CLI. Уточните у своего хоста, как это сделать.

## Установка

В общедоступном корне вашего сервера (например, **public_html** или **www**) создайте папку с именем `deployer` и введите ее. Мы будем использовать это как основу для проекта. Вы захотите защитить этот каталог паролем (см. [Руководство по DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-apache-on-ubuntu-14-04) для ручного подхода или используйте [CPanel](https://www.siteground.com/tutorials/cpanel/pass_protected_directories.htm), если доступно).

У вас должна быть рабочая установка Grav, а также [Composer](https://getcomposer.org/). На некоторых хостах уже установлен Composer, что вы можете проверить, запустив `composer -v`. Если он не установлен, вы можете установить его через SSH - из корневого каталога - со следующим:

```bash
export COMPOSERDIR=~/bin;mkdir bin
curl -sS https://getcomposer.org/installer | php -- --install-dir=$COMPOSERDIR --filename=composer
```

Или, если вы предпочитаете [локальную установку](https://getcomposer.org/download/), выполните следующую команду в папке `public_html/deployer/`:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

При локальной установке composer запускается через `composer.phar`, а не просто `composer`. Теперь, всё ещё находясь в папке `public_html/deployer/`, запустите следующее, чтобы установить [Deployer](https://deployer.org/docs/installation):

```bash
composer require deployer/deployer
```

Теперь, всё ещё в той же папке, создайте файл с именем `deploy.php`. Мы будем использовать это для запуска каждой задачи с помощью Deployer. Скопируйте и вставьте в файл следующее:

```php
<?php
namespace Deployer;
require 'vendor/autoload.php';

// Configuration
set('default_stage', 'production');
set(php, 'php56');

// Servers
localServer('site1')
	->stage('production')
	->set('deploy_path', '/home/username/public_html/deployer/grav');

desc('Backup Grav installations');
task('backup', function () {
	$backup = run('{{php}} bin/grav backup');
	writeln($backup);
});
desc('Upgrade Grav Core');
task('core', function () {
	$self_upgrade = run('{{php}} bin/gpm self-upgrade -y');
	writeln($self_upgrade);
});
desc('Upgrade Grav Packages');
task('packages', function () {
	$upgrade = run('{{php}} bin/gpm update -y');
	writeln($upgrade);
});
?>
```

## Конфигурация

Поскольку для Deployer требуется явная промежуточная среда, мы устанавливаем её на `production`. Кроме того, чтобы разрешить конкретную версию php, мы устанавливаем исполняемый файл по умолчанию. Это может быть именованный исполняемый файл или путь к определенной версии PHP. Наша конфигурация теперь выглядит так:

```php
// Configuration
set('default_stage', 'production');
set(php, 'php56');
```

Если по умолчанию используется версия PHP CLI **5.6*** или выше, вы измените её на `set(php, 'php');`.

### Серверы

Мы можем настроить столько серверов/сайтов, сколько нужно, скрипт будет запускаться для каждого из них по порядку. Они могут быть локальными или на внешних серверах, но поскольку это локальная установка, мы используем `localServer` (дополнительные настройки см. в главе [Deployer / servers](https://deployer.org/docs/servers)). Вот пример с несколькими сайтами:

```php
// Servers
localServer('site1')
	->stage('production')
	->set('deploy_path', '/home/username/public_html/deployer/grav1');
localServer('site2')
	->stage('production')
	->set('deploy_path', '/home/username/public_html/deployer/grav2');
localServer('site3')
	->stage('production')
	->set('deploy_path', 'C:\caddy\grav1');
localServer('site4')
	->stage('production')
	->set('deploy_path', 'C:\caddy\grav2');
```

Обратите внимание, что мы используем абсолютные пути к установкам, но они зависят от того, как путь интерпретируется SSH. Это означает, что на сервере нам нужен полный путь, потому что Deployer интерпретирует это правильно, но мы также можем использовать тильду, если установлен HOMEDIR, например: `~/public_html/deployer/grav1`.

### Задачи

В настоящее время настроены три задачи: «резервное копирование», «ядро» и «пакеты». Запуск `php vendor/bin/dep backup` создает резервную копию каждой установки, доступную в **deploy_path/backup/BACKUP.zip**, где `deploy_path` - это пути, которые вы настроили для серверов.

Запуск `php vendor/bin/dep core` обновляет сам Grav и делает это с параметром `--all-yes`, чтобы пропустить все задаваемые вопросы типа «Да/Нет». То же самое применяется при запуске пакетов php vendor/bin/dep, которые обновляют все плагины и темы.

## Вывод

Теперь мы можем легко обновить все ваши определенные сайты, выполнив задачи по порядку. Сначала мы вводим папку `public_html/deployer/`, а затем выполняем каждую команду по мере необходимости:

```bash
php vendor/bin/dep backup
php vendor/bin/dep core
php vendor/bin/dep packages
```

Теперь мы сделаем резервную копию каждого сайта, обновим саму CMS, а также все плагины и темы.
