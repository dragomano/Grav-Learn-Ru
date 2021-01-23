---
title: Команды Grav
metadata:
    description: 'Подробнее о командах Grav, доступных во встроенной командной строке.'
page-toc:
  active: true
taxonomy:
    category: docs
---

Grav поставляется со встроенным интерфейсом командной строки (CLI), который можно найти в разделе `bin/grav`. CLI чрезвычайно полезен для выполнения повторяющихся задач, таких как очистка кэша, создание резервных копий и многое другое.

Доступ к CLI — это простой процесс, но вам нужно использовать **терминал**. На mac это называется «терминал», на windows это называется «cmd», а на Linux это просто оболочка. Команды в стиле UNIX изначально недоступны в Windows cmd. Установка пакета [Git SCM](https://gitforwindows.org) в Windows добавляет [Git](https://git-scm.com/) и Git BASH, который является альтернативой командной строке, делая доступными команды UNIX. Если вы получаете удаленный доступ к своему серверу, то, скорее всего, будете использовать **SSH** для удаленного входа на свой сервер. Посмотрите этот [отличный учебник для получения дополнительной информации о SSH](https://code.tutsplus.com/tutorials/ssh-what-and-how--net-25138).

Хотя некоторые операции могут быть выполнены вручную, с помощью _relying_ на CLI, эти задачи могут быть автоматизированы с помощью _cronjobs_, которые выполняются ежедневно.

Чтобы получить список всех команд, доступных в Grav, вы можете запустить команду:

[prism classes="language-bash command-line"]
bin/grav list
[/prism]

Это должно отображать что-то вроде:

[prism classes="language-text"]
Доступные команды:
  backup       Создает резервную копию экземпляра Grav
  cache        [clearcache|cache-clear] Очищает кэш
  clean        Запустить очистку
  composer     Обновляет зависимости поставщика composer, необходимые Grav.
  help         Отображает справку по команде
  install      Устанавливает зависимости Grav. При желании можно создать символические ссылки
  list         Перечисляет команды
  logviewer    Отображение последних записей журнала Grav
  new-project  [newproject] Создает новый проект Grav со всеми установленными зависимостями
  sandbox      Настройка базовой системы Grav в вашем webroot, хорошо подходит для разработки, игры или начала новой работы
  scheduler    Запустить планировщик Grav. Лучше всего при интеграции с системой cron
  security     Способен выполнять различные проверки безопасности
[/prism]

Чтобы получить справку для конкретной команды, вы можете добавить перед ней `help`:

[prism classes="language-bash command-line"]
bin/grav help install
[/prism]

## Резервные копии

Система резервного копирования Grav была полностью переработана в Grav 1.6 для поддержки нескольких профилей резервного копирования. Эти профили настраиваются в файле `user/config/backups.aml`. Если у вас нет специального файла конфигурации, Grav будет использовать файл по умолчанию, указанный в `system/config/backups.yaml`.

Если Grav обнаруживает несколько профилей резервного копирования, команда CLI предложит вам выбрать тот, который вы хотите сохранить, с помощью команды CLI.

[prism classes="language-bash command-line" cl-output="2-10"]
cd ~/workspace/portfolio
bin/grav backup

Grav Backup
===========

Choose a backup?
  [0] Default Site Backup
  [1] Pages Backup
[/prism]

В качестве альтернативы вы можете напрямую передать индекс профиля:

[prism classes="language-bash command-line" cl-output="2-10"]
$ cd ~/workspace/portfolio
bin/grav backup 1

Archiving 36 files [===================================================] 100% < 1 sec Done...

 [OK] Backup Successfully Created: /users/joe/workspace/portfolio/backup/pages_backup--20190227120510.zip
[/prism]

Более подробную информацию о функциях резервного копирования можно найти в разделе [Дополнительно -> Резервные копии](/advanced/backups).

## Очистка

Эта команда CLI в основном используется в процессе сборки пакета, поскольку она удаляет из Grav посторонние файлы и папки. Настоятельно рекомендуется **не использовать это** самостоятельно, если вы не используете его для создания собственных пакетов Grav.

[prism classes="language-bash command-line"]
bin/grav clean
[/prism]

## Очистка кэша

Вы можете очистить кэш, удалив все файлы и папки в директории `cache/`.

Эквивалентная команда CLI:

[prism classes="language-bash command-line"]
$ cd ~/webroot/my-grav-project
bin/grav cache
[/prism]

Существует несколько псевдонимов для совместимости ('cache`, `cache-clear`, `clearcache`,`clear').

По умолчанию используется стандартный процесс очистки кэша, однако вы можете дополнительно управлять им с помощью следующих параметров:

[prism classes="language-text"]
--purge           Если задано, удалит старый кэш
--all             Если задано, удалит всё, включая скомпилированные файлы, twig, кэши doctrine
--assets-only     Если задано, удалит только assets/*
--images-only     Если задано, удалит только images/*
--cache-only      Если задано, удалит только cache/*
--tmp-only        Если задано, удалит только tmp/*
[/prism]

## Composer

Если вы установили Grav через GitHub и вручную установили пакеты поставщиков на основе composer, вы можете легко обновить их с помощью:

[prism classes="language-bash command-line"]
bin/grav composer
[/prism]

Вы также можете передать параметры композитора, такие как `install`:

[prism classes="language-bash command-line"]
bin/grav composer --install
[/prism]

или

[prism classes="language-bash command-line"]
bin/grav composer --update
[/prism]

!! Все они используют параметр Composer `--no-dev`, поэтому, чтобы иметь возможность выполнять тестирование, вы должны использовать композитор напрямую: `bin/composer.phar`

## Install

Чтобы установить зависимости, на которые полагается Grav (плагин **errors**, плагин **errors**, тема **Antimatter**), запустите **терминал** или **консоль** и перейдите в папку grav, где вы хотите установить зависимости и запустить команду CLI.

[prism classes="language-bash command-line"]
$ cd ~/webroot/my-grav-project
bin/grav install
[/prism]

Теперь у вас должны быть установлены зависимости:
* `~/webroot/my-grav-project/user/plugins/error`
* `~/webroot/my-grav-project/user/plugins/problems`
* `~/webroot/my-grav-project/user/themes/antimatter`

## Просмотр журнала

В рамках Grav 1.6 была создана новая команда CLI logviewer, позволяющая быстро просматривать журналы Grav.

Самый простой способ использовать эту команду - просто набрать:

[prism classes="language-bash command-line"]
cd ~/webroot/my-grav-project
bin/grav logviewer
[/prism]

Это выведет последние 20 записей журнала из файла `logs/grav.log`. Есть несколько вариантов:

[prism classes="language-text line-numbers"]
-f, --file[=FILE]     расположение настраиваемого файла журнала (по умолчанию = grav.log)
-l, --lines[=LINES]   количество строк (по умолчанию = 10)
-v, --verbose         подробный вывод, включая трассировку стека, если она доступна
[/prism]

например

[prism classes="language-bash command-line" cl-output="2-11"]
bin/grav logviewer --lines=4                                                                           [12:27:20]

Log Viewer
==========

viewing last 4 entries in grav.log

2019-02-27 12:00:30 [WARNING] Plugin 'foo-plugin' enabled but not found! Try clearing cache with `bin/grav cache`
2019-02-27 12:04:57 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/default_site_backup--20190227120450.zip
2019-02-27 12:05:10 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/pages_backup--20190227120510.zip
2019-02-27 12:26:00 [NOTICE] Backup Created: /Users/joe/my-grav-project/backup/pages_backup--20190227122600.zip
[/prism]

И подробный вывод с трассировкой стека:

[prism classes="language-bash command-line" cl-output="2-22"]
bin/grav logviewer -v                                                                                                       [16:12:12]

Log Viewer
==========

viewing last 20 entries in grav.log

2019-03-14 05:52:44 [WARNING] Plugin 'simplesearch.bak' enabled but not found! Try clearing cache with `bin/grav clear-cache`
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

2019-03-14 05:52:46 [WARNING] Plugin 'simplesearch.bak' enabled but not found! Try clearing cache with `bin/grav clear-cache`
...
[/prism]

## Новый проект

Каждый раз, когда вы хотите начать новый проект с Grav, вам нужно начинать с чистого экземпляра Grav. Через интерфейс командной строки этот процесс очень прост и занимает всего несколько секунд.

1. Запустите **терминал** или **консоль** и перейдите в папку _grav_ (мы предполагаем, что она находится в папке `~/Projects/grav`).

[prism classes="language-bash command-line"]
cd ~/Projects/grav
[/prism]

2. Запустите GRAV CLI для создания нового проекта, причем конечным пунктом будет местоположение, в котором будет находиться ваш проект (обычно это [webroot](http://en.wikipedia.org/wiki/Webroot) вашего веб-сервера). Предположим, что мы создаем **портфолио** и хотим получить его по адресу `~/Webroot/portfolio`.

[prism classes="language-bash command-line"]
bin/grav new-project ~/webroot/portfolio
[/prism]

Это создаст новый экземпляр Grav и загрузит все необходимые зависимости.

## Песочница

Grav имеет отличную утилиту под названием `sandbox`, которая может быстро создать копию Grav-установки [symlinked](/cli-console/command-line-intro#symbolic-links). Проще говоря, запуск `bin/grav sandbox-S DESTINATION` - где «DESTINATION» - это путь к папке, в которую вы хотите скопировать установку,-воссоздает установку Grav в другой папке.

Например, выполните:

[prism classes="language-bash command-line"]
bin/grav sandbox -s ../copy
[/prism]

Из текущей Grav-папки создается родственная папка с именем `copy`, где следующие папки являются виртуальными копиями: '/bin, / system, /vendor, /webserver-configs`, а также стандартные файлы, которые обычно находятся в корневой папке Grav. Все содержимое в /user будет копией под копирку, а не виртуальным, поэтому вы можете легко начать настройку новой установки, не создавая накладных расходов из основных файлов.

## Планировщик

Как описано в разделе [Advanced - > Scheduler](/advanced/scheduler), планировщик можно контролировать с помощью команды CLI.

Базовая команда будет вручную запускать задачи планировщика, которые должны быть выполнены:

[prism classes="language-bash command-line"]
bin/grav scheduler
[/prism]

Чтобы получить более подробную информацию, вы можете запустить с помощью дополнительной опции `-v` :

[prism classes="language-bash command-line" cl-output="2-10"]
bin/grav scheduler -v

Running Scheduled Jobs
======================

[2019-02-27T12:34:07-07:00] Success: Grav\Common\Cache::purgeJob
[2019-02-27T12:34:07-07:00] Success: Grav\Common\Cache::clearJob
[2019-02-27T12:34:07-07:00] Success: ls -lah
[/prism]

Другие варианты включают в себя:

[prism classes="language-text line-numbers"]
-i, --install         Показать команду установки
-j, --jobs            Показать сводку заданий
-d, --details         Показать детали заданий
[/prism]

Пожалуйста, обратитесь к разделу [Advanced -> Scheduler](/advanced/scheduler) для получения более подробной информации об этих параметрах.

## Безопасность

В Grav 1.5 добавлена ​​новая команда интерфейса командной строки сканера безопасности. Вы можете запустить это, чтобы быстро проверить свое содержимое на соответствие [настроенным параметрам безопасности](/basics/grav-configuration#security).

[prism classes="language-bash command-line" cl-output="2-10"]
bin/grav security                                                                                       [12:34:12]

Grav Security Check
===================

Scanning 11 pages [===================================================] 100% < 1 sec

[OK] Security Scan complete: No issues found...
[/prism]

#### Информация PHP CGI-FCGI

Чтобы определить, работает ли на вашем сервере `cgi-fcgi` в командной строке, введите следующее:

[prism classes="language-bash command-line" cl-output="2-5"]
$ php -v
PHP 5.5.17 (cgi-fcgi) (built: Sep 19 2014 09:49:55)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with the ionCube PHP Loader v4.6.1, Copyright (c) 2002-2014, by ionCube Ltd.
[/prism]

Если вы видите ссылку на `(cgi-fcgi)`, вам нужно будет поставить перед всеми командами `bin/grav` префикс `php-cli`. В качестве альтернативы вы можете настроить псевдоним в своей оболочке с помощью чего-то вроде: `alias php="php-cli"`, который обеспечит запуск **CLI** версии PHP из командной строки.






