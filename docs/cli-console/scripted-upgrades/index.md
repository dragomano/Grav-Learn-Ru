---
title: "Скриптовые обновления"
description: "Или: обновление нескольких установок Grav одновременно."
tags:
  - "CLI"
  - "GPM"
  - "Обновление"
---

# Скриптовые обновления

Или: обновление нескольких установок Grav одновременно.

Это руководство о том, как упростить обновление нескольких установок Grav с помощью [Deployer](https://deployer.org/). Под «несколькими» имеются в виду отдельные установки, а не [мультисайт-установка](../../advanced/multisite-setup/index.md). Мы будем использовать путь к каждой установке для запуска команд [CLI Grav](../grav-cli/index.md), но без необходимости вводить их вручную.

Преимущество такого средства запуска задач, как Deployer, в том, что по мере выполнения задач оно точно сообщает, что делает. Оно также показывает любой вывод сервера от выполняемых команд. Например, вот вывод Deployer:

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

И вот так просто Deployer дал Grav команду обновить все пакеты, что обновило плагин Email до новейшей версии.

## Предварительные требования

Как и для Grav, требуется PHP **v7.3.6** или выше. Это относится и к командной строке (CLI), поэтому при наличии нескольких установленных версий используйте ту, что соответствует нужной версии. Проверить версию по умолчанию можно командой `php -v`; в моём случае это **PHP 7.2.34**.

В средах с общим хостингом уточните у хостера, какую команду использовать для CLI. В моём случае это `php74`, которая с `-v` возвращает **PHP 7.4.12**. Это также означает, что нужно добавлять её перед каждым путём: `php74 vendor/bin/dep list`.

Некоторые хостеры также позволяют выбрать версию PHP по умолчанию для CLI — уточните у хостера, как это сделать.

## Настройка

В публичном корне вашего сервера (например, **public_html** или **www**) создайте папку `deployer` и перейдите в неё. Она станет основой проекта. Эту папку стоит защитить паролем (см. [руководство DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-apache-on-ubuntu-14-04) для ручного варианта или используйте [CPanel](https://www.siteground.com/tutorials/cpanel/pass_protected_directories.htm), если он доступен).

Вам понадобится рабочая установка Grav, а также [Composer](https://getcomposer.org/). У некоторых хостеров Composer уже установлен — это можно проверить командой `composer -v`. Если он не установлен, его можно установить через SSH — из корневой папки — следующим образом:

```bash
export COMPOSERDIR=~/bin;mkdir bin
curl -sS https://getcomposer.org/installer | php -- --install-dir=$COMPOSERDIR --filename=composer
```

Или, если предпочитаете [локальную установку](https://getcomposer.org/download/), выполните в папке `public_html/deployer/`:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

При локальной установке composer запускается через `composer.phar`, а не просто `composer`. Теперь, находясь в папке `public_html/deployer/`, выполните следующее, чтобы установить [Deployer](https://deployer.org/docs/installation):

```bash
composer require deployer/deployer
```

Теперь, всё в той же папке, создайте файл `deploy.php`. Он будет использоваться для запуска каждой задачи через Deployer. Скопируйте в файл следующее:

```php linenums="1"
<?php
namespace Deployer;
require 'vendor/autoload.php';

// Конфигурация
set('default_stage', 'production');
set(php, 'php56');

// Серверы
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

Поскольку Deployer требует явно указанной среды развёртывания (stage), мы задаём `production`. Кроме того, чтобы использовать конкретную версию PHP, мы задаём исполняемый файл по умолчанию. Это может быть имя исполняемого файла или путь к конкретной версии PHP. Теперь наша конфигурация выглядит так:

```php linenums="1"
// Конфигурация
set('default_stage', 'production');
set(php, 'php56');
```

Если ваша версия PHP CLI по умолчанию — **5.6\*** или выше, измените это на `set(php, 'php');`.

### Серверы

Можно настроить сколько угодно серверов/сайтов — скрипт будет выполнен для каждого из них по порядку. Это могут быть локальные установки или внешние серверы, но поскольку здесь настройка локальная, мы используем `localServer` (другие варианты конфигурации см. в [Deployer/servers](https://deployer.org/docs/servers)). Вот пример с несколькими сайтами:

```php linenums="1"
// Серверы
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

Обратите внимание, что мы используем абсолютные пути к установкам, но они зависят от того, как путь интерпретируется через SSH. Это значит, что на сервере нужен полный путь, поскольку Deployer интерпретирует его правильно, но можно также использовать тильду, если задан HOMEDIR, например: `~/public_html/deployer/grav1`.

### Задачи

Сейчас настроены три задачи: `backup`, `core` и `packages`. Запуск `php vendor/bin/dep backup` создаёт резервную копию каждой установки, доступную в **deploy_path/backup/BACKUP.zip**, где `deploy_path` — пути, настроенные для серверов.

Запуск `php vendor/bin/dep core` обновляет сам Grav и делает это с параметром `--all-yes`, чтобы пропустить все вопросы «да/нет». То же относится к `php vendor/bin/dep packages`, которая обновляет все плагины и темы.

## Заключение

Теперь можно легко обновить все заданные сайты, выполнив задачи по порядку. Сначала переходим в папку `public_html/deployer/`, затем выполняем нужные команды:

```bash
php vendor/bin/dep backup
php vendor/bin/dep core
php vendor/bin/dep packages
```

Теперь у нас есть резервная копия каждого сайта, обновлённый Grav, а также обновлённые плагины и темы.
