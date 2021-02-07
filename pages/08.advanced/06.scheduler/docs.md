---
title: Планировщик
metadata:
    description: 'Знакомство с планировщиком Grav CMS. Настройка задач. Запуск вручную.'
taxonomy:
    category: docs
---

Планировщик Grav — это новая функция, которая была добавлена ​​в Grav 1.6, которая позволяет запускать задания на периодической основе. Базовая обработка зависит от серверного планировщика **cron**, но после добавления одной записи в службу cron все задания и определенные расписания могут быть настроены через Grav.

Одним из основных преимуществ использования планировщика для обработки задач является то, что они могут выполняться без какого-либо взаимодействия с пользователем и независимо от внешнего интерфейса. Такие задачи, как периодическая очистка кэша, резервное копирование, синхронизация, поисковое индексирование и т. д., являются основными кандидатами на выполнение запланированных заданий.

## Установка

Первым шагом на пути к настройке планировщика и его готовности к выполнению задач является добавление команды `bin/grav scheduler` в службу cron. Самый простой подход - использовать саму команду CLI для вывода соответствующей команды для запуска для установки:

[prism classes="language-bash command-line" highlight="10" cl-output="2-10"]
$ bin/grav scheduler -i

Install Scheduler
=================

 [ERROR] You still need to set up Grav's Scheduler in your crontab

 ! [NOTE] To install, run the following command from your terminal:

 (crontab -l; echo "* * * * * cd /Users/andym/grav;/usr/local/bin/php bin/grav scheduler 1>> /dev/null 2>&1") | crontab -
[/prism]

В системе Mac отображается полная требуемая команда, поэтому всё, что вам нужно сделать, это скопировать и вставить команду в терминал и нажать return.

!! Вам необходимо войти в оболочку под тем же пользователем, что и ваш веб-сервер. Это необходимо для того, чтобы пользователь, выполняющий команды schdeduler, соответствовал пользователю веб-сервера, которому необходимо взаимодействовать с этими файлами. Если вы устанавливаете запись crontab с другим пользователем (например, `root`), любые созданные файлы будут создаваться от имени этого пользователя `root`, а не пользователя `webserver`, что может привести к проблемам.

[prism classes="language-bash command-line"]
(crontab -l; echo "* * * * * cd /Users/andym/grav;/usr/local/bin/php bin/grav scheduler 1>> /dev/null 2>&1") | crontab -
[/prism]

Вы не получите ответа, но и ошибок получить не должны. После этого можно убедиться, что всё в порядке, повторно запустив команду `bin/grav scheduler -i`:

[prism classes="language-bash command-line" cl-output="2-10"]
bin/grav scheduler -i

Install Scheduler
=================

 [OK] All Ready! You have already set up Grav's Scheduler in your crontab
[/prism]

Вы также можете получить необходимую команду из плагина админки, просто перейдя в **Инструменты** → **Планировщик**.

## Основы планирования

Для планирования работы частота контролируется гибким форматом.

[prism classes="language-text"]
* * * * * *
| | | | | |
| | | | | +-- Year              (диапазон: 1900-3000)
| | | | +---- Day of the Week   (диапазон: 1-7, 1 — понедельник)
| | | +------ Month of the Year (диапазон: 1-12)
| | +-------- Day of the Month  (диапазон: 1-31)
| +---------- Hour              (диапазон: 0-23)
+------------ Minute            (диапазон: 0-59)
[/prism]

Некоторые примеры:

`0 * * * *`	запускать один раз в час (каждый час на нулевой минуте)
`0 0 * * *`	запускать один раз в день (каждый день в полночь и в нулевую минуту)
`0 0 1 * *`	запускать один раз в месяц (первого числа каждого месяца в полночь и в нулевую минуту)
`0 0 1 1 *`	запускать один раз в год (в первый день первого месяца каждого года в полночь и в нулевую минуту)

Расширенные опции:

`*/5 * * * *` запускать каждые 5 минут



## Файл конфигурации

Вы можете увидеть, какие задания в настоящее время доступны для Планировщика, выполнив команду `bin/grav scheduler -j`:

[prism classes="language-bash command-line" cl-output="2-16"]
bin/grav scheduler -j

Scheduler Jobs Listing
======================

┌─────────────────────┬────────────────────────────────────┬───────────┬─────────┬──────────────────┬─────────┐
│ Job ID              │ Command                            │ Run At    │ Status  │ Last Run         │ State   │
├─────────────────────┼────────────────────────────────────┼───────────┼─────────┼──────────────────┼─────────┤
│ cache-purge         │ Grav\Common\Cache::purgeJob        │ * * * * * │ Success │ 2019-02-21 11:23 │ Enabled │
│ cache-clear         │ Grav\Common\Cache::clearJob        │ * * * * * │ Success │ 2019-02-21 11:23 │ Enabled │
│ default-site-backup │ Grav\Common\Backup\Backups::backup │ 0 3 * * * │ Ready   │ Never            │ Enabled │
│ pages-backup        │ Grav\Common\Backup\Backups::backup │ * 3 * * * │ Success │ 2018-09-20 09:55 │ Enabled │
│ ls-job              │ ls                                 │ * * * * * │ Success │ 2019-02-21 11:23 │ Enabled │
└─────────────────────┴────────────────────────────────────┴───────────┴─────────┴──────────────────┴─────────┘

 ! [NOTE] For error details run "bin/grav scheduler -d"
[/prism]

Планировщик Grav управляется основным файлом конфигурации. Он находится в `user/config/scheduler.yaml`, и для его выполнения требуется, чтобы любое задание было `enabled`.

Ниже конфигурации показаны доступные задания и разрешено ли им выполнение. Просто установите для записи значение `disabled`, чтобы остановить ее выполнение.

[prism classes="language-yaml line-numbers"]
status:
  ls-job: enabled
  cache-purge: enabled
  cache-clear: enabled
  default-site-backup: enabled
  pages-backup: enabled
[/prism]

Чтобы увидеть более подробную информацию о любых потенциальных **ошибках** или увидеть, когда задание будет запущено в следующий раз, вы можете использовать команду `/bin/grav scheduler -d`:

[prism classes="language-bash command-line" cl-output="2-14"]
bin/grav scheduler -d

Job Details
===========

┌─────────────────────┬──────────────────┬──────────────────┬────────┐
│ Job ID              │ Last Run         │ Next Run         │ Errors │
├─────────────────────┼──────────────────┼──────────────────┼────────┤
│ cache-purge         │ 2019-02-21 11:29 │ 2019-02-21 11:31 │ None   │
│ cache-clear         │ 2019-02-21 11:29 │ 2019-02-21 11:31 │ None   │
│ default-site-backup │ Never            │ 2019-02-22 03:00 │ None   │
│ pages-backup        │ 2018-09-20 09:55 │ 2019-02-22 03:00 │ None   │
│ ls-job              │ 2019-02-21 11:29 │ 2019-02-21 11:31 │ None   │
└─────────────────────┴──────────────────┴──────────────────┴────────┘
[/prism]

## Запуск заданий вручную

Команда CLI предоставляет простой способ вручную запускать любые задания. Фактически, это то, что планировщик делает, когда он запускается периодически.

[prism classes="language-bash command-line"]
bin/grav scheduler
[/prism]

Это автоматически запустит задания, но вы также можете увидеть подробности того, что запускается, используя:

[prism classes="language-bash command-line"]
bin/grav scheduler -v
[/prism]

## Системные задания Grav

Ядро Grav обеспечивает несколько рабочих мест прямо из коробки. К ним относятся некоторые полезные задачи технического обслуживания:

* `cache-purge` — Эта задача полезна, если вы используете кэширование файлов Grav, поскольку оно удаляет старые файлы, срок действия которых истек. Это отличная задача для планирования, поскольку в противном случае пользователю потребовалось бы вручную очистить старые кэши. Если вы не уследите за этим, а ваше файловое пространство ограничено, у вас может закончиться место и сервер выйдет из строя.

* `cache-clear` — Очистка кэша — это задание, которое работает так же, как команда `bin/grav clear`, которую вы выполняете вручную. Вы можете настроить, хотите ли вы использовать «стандартную» очистку кэша или вариант `all`, который удаляет все файлы и папки в папке `cache/` для более тщательной очистки кэша.

* `default-site-backup` — Задание резервного копирования по умолчанию, доступное в новой конфигурации Grav Backup. Вы можете создавать собственные конфигурации резервного копирования, и они также будут доступны для запуска в качестве запланированного задания.

## Пользовательские задания

В Grav Scheduler можно вручную настроить любое количество пользовательских заданий. Их можно настроить в том же файле конфигурации `scheduler.yaml`, упомянутом выше. Например, указанная выше команда `ls-job` может быть настроена:

[prism classes="language-yaml line-numbers"]
custom_jobs:
  ls-job:
    command: ls
    args: '-lah'
    at: '* * * * *'
    output: logs/cron-ls.out
    output_mode: overwrite
    email: user@email.com
[/prism]

Командой должен быть любой локальный сценарий, который можно запустить из командной строки/терминала. Требуются только атрибуты `command` и `at`.

## Задания, предоставляемые плагинами

Одна из самых мощных функций Grav Scheduler — это возможность сторонних плагинов создавать собственные задания. Прекрасный пример этого — плагин TNTSearch. TNTSearch — это полнофункциональная система текстового поиска, которая требует, чтобы контент был проиндексирован, прежде чем его можно будет искать. Это задание индексации может выполняться различными способами, но Grav Scheduler позволяет вам периодически переиндексировать ваш контент, вместо того, чтобы делать это вручную.

Первым делом ваш плагин должен подписаться на событие `onSchedulerInitialized()`. А затем создайте метод в своем файле плагина, который может добавлять настраиваемое задание при вызове:

[prism classes="language-bash line-numbers"]
public function onSchedulerInitialized(Event $e): void
{
    $config = $this->config();

    if (!empty($config['scheduled_index']['enabled']))) {
        $scheduler = $e['scheduler'];
        $at = $config['scheduled_index']['at'] ?? '* * * * *';
        $logs = $config['scheduled_index']['logs'] ?? '';
        $job = $scheduler->addFunction('Grav\Plugin\TNTSearchPlugin::indexJob', [], 'tntsearch-index');
        $job->at($at);
        $job->output($logs);
        $job->backlink('/plugins/tntsearch');
    }
}
[/prism]

Здесь вы можете увидеть, как некоторая соответствующая конфигурация планировщика получается из настроек конфигурации плагина TNTSearch, а затем создается новое `Job` с **статической** функцией, называемой `indexJob()`.