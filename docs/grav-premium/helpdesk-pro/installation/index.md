---
title: "Установка"
description: "Установите Helpdesk Pro, защитите его базу данных, настройте cron и выдайте сотрудникам необходимые права."
tags:
  - "Helpdesk Pro"
  - "Установка"
---

# Установка {: #installation }

Эта страница помогает установить и запустить Helpdesk Pro: сам плагин, его базу данных, фоновый процесс и права сотрудников.

## Перед началом {: #before-you-begin }

- Grav 2.0 с Admin Next и плагинами API, Login, Email и Form.
- PHP 8.3 или новее с `pdo_sqlite`, `sqlite3`, `mbstring` и `dom`.
- Лицензия Helpdesk Pro, добавленная в [License Manager](https://getgrav.org/premium/license-manager).

## Установка плагина {: #install-the-plugin }

Установите плагин через GPM, как любой другой:

```bash
bin/gpm install helpdesk-pro
```

Его также можно установить из раздела **Plugins** в Admin Next. После включения Helpdesk Pro добавляет сотрудникам пункт **Helpdesk** в боковую панель Admin Next и собственные команды в `bin/plugin helpdesk-pro`.

Чтобы включить плагин вручную, установите `enabled: true` в `user/config/plugins/helpdesk-pro.yaml`.

## База данных {: #the-database }

Helpdesk Pro хранит всё в одном файле SQLite. На сервере базы данных настраивать нечего.

При первой необходимости Helpdesk Pro создаёт базу по адресу `user/data/helpdesk-pro/db/helpdesk.sqlite` и применяет миграции. Это происходит, когда кто-то открывает рабочую область, страницу портала, запускает фоновый процесс или выполняет `bin/plugin helpdesk-pro migrate`. Обычные просмотры страниц базу не затрагивают.

В папке создаются `.htaccess` с запретом доступа и пустой `index.html`. Поиск, вложения и база данных находятся в `user/data/helpdesk-pro/`.

> [!WARNING] Предупреждение
> Файл `.htaccess` защищает базу данных только на Apache. Стандартные правила сервера Grav не мешают скачать файл `.sqlite` из `user/`, а nginx и Caddy игнорируют `.htaccess`. На таких серверах запретите доступ к папке самостоятельно. Для nginx:
>
> ```nginx
> location ^~ /user/data/ { deny all; }
> ```

Чтобы хранить файл в другом месте, укажите `database.sqlite.path` как потоковый путь (`user-data://…`) или абсолютный путь.

### Политика миграций {: #migration-policy }

`database.auto_migrate` определяет, когда применяются изменения схемы после обновления:

| Значение | Что происходит |
|---|---|
| `sqlite` (по умолчанию) или `auto` | Ожидающие миграции применяются при первом запросе после обновления. Оба значения работают одинаково, поскольку Helpdesk Pro поддерживает только SQLite. |
| `manual` | Ничего не применяется, пока вы не выполните `bin/plugin helpdesk-pro migrate`. |

```bash
bin/plugin helpdesk-pro migrate --status   # показать, что ожидает применения
bin/plugin helpdesk-pro migrate            # применить
```

Фоновый процесс отказывается работать, пока есть ожидающие миграции.

## Настройка cron {: #set-up-cron }

Фоновая работа (почта, доставка уведомлений, дайджесты, входящая почта, индексация поиска, проверки SLA, правила по времени) выполняется как задания в очереди. Большинство заданий запускается сразу после запроса, который их поставил, но только планировщик Grav гарантирует, что очередь будет обработана. Добавьте его в crontab со строкой `PATH`:

```
PATH=/usr/local/bin:/usr/bin:/bin
* * * * * cd /path/to/site && php bin/grav scheduler >> /dev/null 2>&1
```

Укажите в строке `PATH` папку, где находится ваш бинарник PHP.

> [!IMPORTANT] Важно
> Строка `PATH` не является необязательной. `bin/plugin` начинается с `#!/usr/bin/env php`, а планировщик запускает его как дочерний процесс с минимальным `PATH` из cron. Абсолютный путь к PHP в строке crontab исправляет родительский процесс, но не дочерний, поэтому планировщик может выглядеть здоровым, а каждое задание Helpdesk Pro будет завершаться с `env: php: No such file or directory`. Запустите `bin/grav scheduler -d`, чтобы увидеть ошибки каждого задания.

[Задания и cron](../jobs-and-cron/index.md) объясняют очередь, встроенную обработку и догоняющую обработку для сайта без cron.

## Доступ для сотрудников {: #give-staff-access }

Сотрудникам нужен `api.access` для доступа к Admin Next, а также собственные права Helpdesk Pro. Клиентам ничего из этого не нужно: они пользуются порталом через обычный вход на сайте.

Helpdesk Pro никогда не создаёт группы. Создайте две в Admin Next в разделе **Accounts**:

| Группа | Права |
|---|---|
| `helpdesk-agents` | `api.access`, `helpdesk-pro.desk`, `helpdesk-pro.triage` |
| `helpdesk-admins` | `api.access` и все права `helpdesk-pro.*` |

Затем добавьте сотрудников в нужную группу. [Проекты и права](../projects-and-permissions/index.md) перечисляют все права и объясняют, в каких проектах работает каждый агент.

## Первые проверки {: #first-checks }

1. Откройте Admin Next как сотрудник. Пункт **Helpdesk** появляется в боковой панели у всех, у кого есть `helpdesk-pro.desk`.
2. Выполните `bin/plugin helpdesk-pro status`. Команда покажет путь к базе данных, схему и очередь заданий.
3. Выполните `bin/plugin helpdesk-pro work`. Она запустит один проход фонового процесса и сообщит, что сделала.

Экран **Operations → System** в рабочей области показывает тот же отчёт о состоянии.

## Связанные разделы {: #related }

- [Начало работы](../getting-started/index.md)
- [Задания и cron](../jobs-and-cron/index.md)
- [Конфигурация](../configuration/index.md)
- [Устранение неполадок](../troubleshooting/index.md)
