---
description: Создание резервных копий сайта на Grav CMS.
---

# Резервные копии

Система резервного копирования в Grav была полностью переписана для Grav 1.6, чтобы обеспечить больше функций и возможностей. Улучшения включают:

* Интеграция с новым Grav [Scheduler](/advanced/scheduler), чтобы автономное резервное копирование могло запускаться, когда вы хотите
* Возможность создания нескольких резервных ** профилей **, каждый со своим собственным набором файлов, исключая правила пути и файлов, а также настройку расписания
* Новые возможности **автоматической очистки** на основе `number`, `space` или `time`.
* Новая специальная страница резервного копирования в разделе **Инструменты** подключаемого модуля администратора.

## Конфигурация

Для обратной совместимости конфигурация по умолчанию имитирует систему до Grav 1.6, однако теперь она имеет ограничение в 5 ГБ по умолчанию для резервного пространства. Вы должны скопировать файл конфигурации по умолчанию (`system/config/backups.yaml`) в ваш `user/config/`

!!! tip ""

	Если вы используете **плагин админки** и сохраните конфигурацию, файл `user/config/backups.yaml` будет создан автоматически.

Конфигурация по умолчанию следующая:

```yaml
purge:
    trigger: space
    max_backups_count: 25
    max_backups_space: 5
    max_backups_time: 365

profiles:
  -
    name: 'Default Site Backup'
    root: '/'
    schedule: false
    schedule_at: '0 3 * * *'
    exclude_paths: "/backup\r\n/cache\r\n/images\r\n/logs\r\n/tmp"
    exclude_files: ".DS_Store\r\n.git\r\n.svn\r\n.hg\r\n.idea\r\n.vscode\r\nnode_modules"
```

#### Очистка

* `space` - удалит старые резервные копии, когда вы достигнете предельного места. Контролируется `max_backups_space`, измеряется в ГБ
* `time` - удалит старые резервные копии, превышающие количество дней. Контролируется `max_backups_time`, измеряемым в `днях`.
* `number` - удалит старые резервные копии сверх определенного количества. Контролируется `max_backups_count`.

#### Профили

Можно настроить массив профилей. Профиль «Резервное копирование сайта по умолчанию» настроен аналогично профилю резервного копирования Grav по умолчанию в предыдущих версиях. По умолчанию резервное копирование не обрабатывается планировщиком автоматически, но вы можете установить `schedule: true` и настроить параметр `schedule_at:` с предпочтительным [выражением cron](https://crontab.guru/).

Примером более сложного набора профилей может быть:

```yaml
profiles:
  -
    name: 'Default Site Backup'
    root: /
    exclude_paths: "/backup\r\n/cache\r\n/images\r\n/logs\r\n/tmp"
    exclude_files: ".DS_Store\r\n.git\r\n.svn\r\n.hg\r\n.idea\r\n.vscode\r\nnode_modules"
    schedule: true
    schedule_at: '0 4 * * *'
  -
    name: 'Pages Backup'
    root: 'page://'
    exclude_files: .git
    schedule: true
    schedule_at: '* 3 * * *'
```

## Команда CLI

Более подробно это описано в разделе [Cli Console -> Grav Command](/cli-console/grav-cli), но вот пример запуска резервного копирования вручную:

```bash
cd ~/workspace/portfolio
bin/grav backup

Grav Backup
===========

Choose a backup?
  [0] Default Site Backup
  [1] Pages Backup

Archiving 36 files [===================================================] 100% < 1 sec Done...

 [OK] Backup Successfully Created: /users/joe/workspace/portfolio/backup/pages_backup--20190227120510.zip
```
