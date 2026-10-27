---
title: "Команда Grav"
description: "Grav поставляется со встроенным интерфейсом командной строки (CLI), который находится в bin/grav. CLI крайне полезен для выполнения повторяющихся задач, таких как очистка кэша, создание резервных копий и многого другого."
tags:
  - "CLI"
---

# Команда Grav

Grav поставляется со встроенным интерфейсом командной строки (CLI), который находится в `bin/grav`. CLI крайне полезен для выполнения повторяющихся задач, таких как **очистка кэша**, создание **резервных копий** и многое другое.

Доступ к CLI — простой процесс, но вам понадобится **терминал**. В Mac он называется `Terminal`, в Windows — `cmd`, а в Linux это просто оболочка. Команды в стиле UNIX изначально недоступны в командной строке Windows. Установка пакета [msysgit](http://msysgit.github.io/) в Windows добавляет [Git](https://git-scm.com/) и Git BASH — альтернативную командную строку, в которой доступны команды UNIX. Если вы работаете с сервером удалённо, скорее всего, вы будете использовать **SSH** для входа на него. Ознакомьтесь с этим [отличным руководством по SSH](http://code.tutsplus.com/tutorials/ssh-what-and-how--net-25138).

Хотя некоторые операции можно выполнять вручную, _полагаясь_ на CLI, эти задачи можно автоматизировать с помощью _заданий cron_, выполняемых ежедневно.

Чтобы получить список всех команд, доступных в Grav, выполните команду:

```bash
bin/grav list
```

Вывод должен быть примерно таким:

```text
Available commands:
  backup       Creates a backup of the Grav instance
  cache        [clearcache|cache-clear] Clears Grav cache
  clean        Handles cleaning chores for Grav distribution
  composer     Updates the composer vendor dependencies needed by Grav.
  help         Displays help for a command
  install      Installs the dependencies needed by Grav. Optionally can create symbolic links
  list         Lists commands
  logviewer    Display the last few entries of Grav log
  new-project  [newproject] Creates a new Grav project with all the dependencies installed
  sandbox      Setup of a base Grav system in your webroot, good for development, playing around or starting fresh
  scheduler    Run the Grav Scheduler.  Best when integrated with system cron
  security     Capable of running various Security checks
```

Чтобы получить справку по конкретной команде, добавьте `help` перед командой:

```bash
bin/grav help install
```

## Backup

Система резервного копирования Grav была полностью переработана в Grav 1.6 и теперь поддерживает несколько профилей резервного копирования. Эти профили настраиваются в `user/config/backups.yaml`. Если у вас нет собственного файла конфигурации, Grav использует файл по умолчанию из `system/config/backups.yaml`.

Если Grav обнаруживает несколько профилей резервного копирования, команда CLI предложит выбрать тот, с которым нужно создать резервную копию.

```bash
cd ~/workspace/portfolio
bin/grav backup

Grav Backup
===========

Choose a backup?
  [0] Default Site Backup
  [1] Pages Backup
```

Либо можно сразу передать индекс профиля:

```bash
$ cd ~/workspace/portfolio
bin/grav backup 1

Archiving 36 files [===================================================] 100% < 1 sec Done...

 [OK] Backup Successfully Created: /users/joe/workspace/portfolio/backup/pages_backup--20190227120510.zip
```

Подробнее о функциях резервного копирования — в разделе [Резервные копии](../../advanced/backups/index.md).

## Clean

Эта команда CLI в основном используется в процессе сборки пакетов, так как удаляет лишние файлы и папки из Grav. Настоятельно рекомендуется **не использовать её** самостоятельно, если только вы не собираете собственные пакеты Grav.

```bash
bin/grav clean
```

## Очистка кэша {: #clear-cache }

Очистить кэш можно, удалив все файлы и папки в `cache/`.

Эквивалентная команда CLI:

```bash
$ cd ~/webroot/my-grav-project
bin/grav cache
```

Для совместимости есть несколько псевдонимов (`cache`, `cache-clear`, `clearcache`, `clear`).

По умолчанию выполняется стандартная очистка кэша, но её можно точнее настроить с помощью следующих параметров:

```text
--purge           If set purge old caches
--all             If set will remove all including compiled, twig, doctrine caches
--assets-only     If set will remove only assets/*
--images-only     If set will remove only images/*
--cache-only      If set will remove only cache/*
--tmp-only        If set will remove only tmp/*
```

## Composer

Если вы установили Grav через GitHub и вручную установили vendor-пакеты на основе composer, их можно легко обновить командой:

```bash
bin/grav composer
```

Также можно передавать параметры composer, например `install`:

```bash
bin/grav composer --install
```

или

```bash
bin/grav composer --update
```

> [!CAUTION] Внимание
> Все они используют параметр composer `--no-dev`, поэтому для проведения тестирования следует использовать composer напрямую: `bin/composer.phar`

## Install

Чтобы установить зависимости, на которые опирается Grav (плагин **error**, плагин **problems**, тема **antimatter**), запустите **терминал** или **консоль**, перейдите в папку grav, где нужно установить зависимости, и выполните команду CLI.

```bash
$ cd ~/webroot/my-grav-project
bin/grav install
```

Теперь зависимости должны быть установлены в:
* `~/webroot/my-grav-project/user/plugins/error`
* `~/webroot/my-grav-project/user/plugins/problems`
* `~/webroot/my-grav-project/user/themes/antimatter`

## Просмотр журналов

В Grav 1.6 появилась новая команда CLI `logviewer` для быстрого просмотра журналов Grav.

Проще всего использовать эту команду так:

```bash
cd ~/webroot/my-grav-project
bin/grav logviewer
```

Она выведет последние 20 записей из файла `logs/grav.log`. Есть несколько параметров:

```text linenums="1"
-f, --file[=FILE]     custom log file location (default = grav.log)
-l, --lines[=LINES]   number of lines (default = 10)
-v, --verbose         verbose output including a stack trace if available
```

Например:

```bash
bin/grav logviewer --lines=4                                                                           [12:27:20]

Log Viewer
==========

viewing last 4 entries in grav.log

2019-02-27 12:00:30 [WARNING] Plugin 'foo-plugin' enabled but not found! Try clearing cache with I17
2019-02-27 12:04:57 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/default_site_backup--20190227120450.zip
2019-02-27 12:05:10 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/pages_backup--20190227120510.zip
2019-02-27 12:26:00 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/pages_backup--20190227122600.zip
```

А вот подробный вывод с трассировками стека:

```bash
bin/grav logviewer -v                                                                                                       [16:12:12]

Log Viewer
==========

viewing last 20 entries in grav.log

2019-03-14 05:52:44 [WARNING] Plugin 'simplesearch.bak' enabled but not found! Try clearing cache with I18
2019-03-14 05:52:44 [CRITICAL] A function must be an instance of \Twig_FunctionInterface or \Twig_SimpleFunction.
0 /Users/joe/my-grav-project/plugins/acme-twig-filters/acme-twig-filters.php(52): Twig\Environment->addFunction(Object(Twig\TwigFilter))
1 /Users/joe/my-grav-project/vendor/symfony/event-dispatcher/EventDispatcher.php(212): Grav\Plugin\ACMETwigFiltersPlugin->onTwigInitialized(Object(RocketTheme\Toolbox\Event\Event), 'onTwigInitializ...', Object(RocketTheme\Toolbox\Event\EventDispatcher))
2 /Users/joe/my-grav-project/vendor/symfony/event-dispatcher/EventDispatcher.php(44): Symfony\Component\EventDispatcher\EventDispatcher->doDispatch(Array, 'onTwigInitializ...', Object(RocketTheme\Toolbox\Event\Event))
3 /Users/joe/my-grav-project/vendor/rockettheme/toolbox/Event/src/EventDispatcher.php(23): Symfony\Component\EventDispatcher\EventDispatcher->dispatch('onTwigInitializ...', Object(RocketTheme\Toolbox\Event\Event))
4 /Users/joe/my-grav-project/system/src/Grav/Common/Grav.php(365): RocketTheme\Toolbox\Event\EventDispatcher->dispatch('onTwigInitializ...', Object(RocketTheme\Toolbox\Event\Event))
5 /Users/joe/my-grav-project/system/src/Grav/Common/Twig/Twig.php(175): Grav\Common\Grav->fireEvent('onTwigInitializ...')
6 /Users/joe/my-grav-project/system/src/Grav/Common/Processors/TwigProcessor.php(24): Grav\Common\Twig\Twig->init()
7 /Users/joe/my-grav-project/system/src/Grav/Framework/RequestHandler/Traits/RequestHandlerTrait.php(45): Grav\Common\Processors\TwigProcessor->process(Object(Nyholm\Psr7\ServerRequest), Object(Grav\Framework\RequestHandler\RequestHandler))
8 /Users/joe/my-grav-project/system/src/Grav/Framework/RequestHandler/Traits/RequestHandlerTrait.php(57): Grav\Framework\RequestHandler\RequestHandler->handle(Object(Nyholm\Psr7\ServerRequest))
9 /Users/joe/my-grav-project/system/src/Grav/Common/Processors/AssetsProcessor.php(28): Grav\Framework\RequestHandler\RequestHandler->handle(Object(Nyholm\Psr7\ServerRequest))

2019-03-14 05:52:46 [WARNING] Plugin 'simplesearch.bak' enabled but not found! Try clearing cache with I19
...
```

## Новый проект

Каждый раз, начиная новый проект на Grav, нужно исходить из чистого экземпляра Grav. Через CLI этот процесс очень прост и занимает всего несколько секунд.

1. Запустите **терминал** или **консоль** и перейдите в папку _grav_ (в этом документе будем считать, что она находится в `~/Projects/grav`)

```bash
cd ~/Projects/grav
```

2. Выполните CLI Grav, чтобы создать новый проект; в качестве назначения укажите место, где будет размещён проект (обычно [корневая папка веб-сервера](http://en.wikipedia.org/wiki/Webroot)). Предположим, мы создаём **portfolio** и хотим разместить его в `~/Webroot/portfolio`.

```bash
bin/grav new-project ~/webroot/portfolio
```

Будет создан новый экземпляр Grav и загружены все необходимые зависимости.

## Sandbox

В Grav есть удобная утилита `sandbox`, которая быстро создаёт [символьно связанную](../command-line-intro/index.md#symbolic-links) копию установки Grav. Проще говоря, команда `bin/grav sandbox -s DESTINATION` — где «DESTINATION» — путь к папке, в которой нужна копия установки — воссоздаёт установку Grav в другой папке.

Например, команда:

```bash
bin/grav sandbox -s ../copy
```

выполненная из текущей папки Grav, создаёт рядом папку `copy`, в которой следующие папки являются виртуальными копиями: `/bin, /system, /vendor, /webserver-configs`, а также стандартные файлы, обычно находящиеся в корневой папке Grav. Всё содержимое в /user будет точными копиями, а не виртуальными, так что можно сразу приступить к настройке новой установки, не создавая накладных расходов из-за файлов ядра.

## Планировщик

Как описано в разделе [Планировщик](../../advanced/scheduler/index.md), планировщик можно контролировать с помощью команды CLI.

Базовая команда вручную запускает задачи планировщика, срок которых наступил:

```bash
bin/grav scheduler
```

Чтобы получить больше подробностей, запустите с необязательным параметром `-v`:

```bash
bin/grav scheduler -v

Running Scheduled Jobs
======================

[2019-02-27T12:34:07-07:00] Success: Grav\Common\Cache::purgeJob
[2019-02-27T12:34:07-07:00] Success: Grav\Common\Cache::clearJob
[2019-02-27T12:34:07-07:00] Success: ls -lah
```

Другие параметры:

```text linenums="1"
-i, --install         Show Install Command
-j, --jobs            Show Jobs Summary
-d, --details         Show Job Details
```

Более подробную информацию об этих параметрах см. в разделе [Планировщик](../../advanced/scheduler/index.md).

## Безопасность

В Grav 1.5 появилась новая команда CLI для сканирования безопасности. Её можно запустить, чтобы быстро проверить содержимое на соответствие [настроенным параметрам безопасности](../../basics/grav-configuration/index.md#security).

```bash
bin/grav security                                                                                       [12:34:12]

Grav Security Check
===================

Scanning 11 pages [===================================================] 100% < 1 sec

[OK] Security Scan complete: No issues found...
```

#### Информация о PHP CGI-FCGI

Чтобы определить, работает ли на вашем сервере `cgi-fcgi` в командной строке, введите:

```bash
$ php -v
PHP 5.5.17 (cgi-fcgi) (built: Sep 19 2014 09:49:55)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with the ionCube PHP Loader v4.6.1, Copyright (c) 2002-2014, by ionCube Ltd.
```

Если вы видите упоминание `(cgi-fcgi)`, нужно добавлять `php-cli` перед всеми командами `bin/grav`. Либо можно настроить псевдоним в оболочке, например `alias php="php-cli"`, чтобы из командной строки запускалась именно **CLI**-версия PHP.
