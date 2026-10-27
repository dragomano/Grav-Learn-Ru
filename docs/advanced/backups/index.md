---
title: "Резервные копии"
description: "Система резервного копирования в Grav была полностью переписана для Grav 1.6, чтобы дать больше фич и функциональности. Улучшения включают…"
tags:
  - "Резервные копии"
---

# Резервные копии

Система резервного копирования в Grav была полностью переписана для Grav 1.6, чтобы дать больше фич и функциональности. Улучшения включают:

* Интеграцию с новым [планировщиком](../scheduler/index.md) Grav, позволяющую запускать резервное копирование в удобное время
* Возможность создавать несколько **профилей** резервного копирования, каждый со своим набором файлов, правилами исключения путей и файлов и настройками расписания
* Новые параметры **автоочистки** по `number`, `space` или `time`.
* Новую отдельную страницу резервных копий в разделе **Tools** плагина админки.

## Конфигурация

Для обратной совместимости конфигурация по умолчанию повторяет систему до Grav 1.6, однако теперь по умолчанию есть ограничение в 5 ГБ для места под резервные копии. Скопируйте файл конфигурации по умолчанию (`system/config/backups.yaml`) в ваш `user/config/`.

> [!CAUTION] Внимание
> Если вы используете **плагин админки** и сохраняете конфигурацию, файл `user/config/backups.yaml` будет создан автоматически.

Конфигурация по умолчанию выглядит так:

```yaml linenums="1"
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

* `space` — удаляет старые резервные копии при достижении предела места. Управляется параметром `max_backups_space`, измеряемым в `GB`
* `time` — удаляет резервные копии старше определённого числа дней. Управляется параметром `max_backups_time`, измеряемым в `days`
* `number` — удаляет резервные копии сверх определённого количества. Управляется параметром `max_backups_count`.

#### Профили

Можно настроить массив профилей. Профиль `Default Site Backup` настраивается так же, как резервное копирование Grav по умолчанию в предыдущих версиях. По умолчанию резервная копия не обрабатывается планировщиком автоматически, но можно задать `schedule: true` и настроить параметр `schedule_at:` с нужным [cron-выражением](https://crontab.guru/).

Пример более сложного набора профилей:

```yaml linenums="1"
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

Подробнее об этом рассказано в разделе [Консоль CLI → Команды Grav](../../cli-console/grav-cli/index.md), а здесь приведён пример ручного запуска резервного копирования:

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
