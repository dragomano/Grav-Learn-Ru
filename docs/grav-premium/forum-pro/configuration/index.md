---
title: "Конфигурация"
description: "Полный справочник настроек Forum Pro, блок за блоком, с указанием вкладки админки для каждого параметра."
tags:
  - "Forum Pro"
  - "Конфигурация"
---

# Конфигурация {: #configuration }

Все настройки находятся в `user/config/plugins/forum-pro.yaml`, и почти все их можно изменить в **Admin → Plugins → Forum Pro**, где параметры распределены по семи вкладкам: **General**, **Posting**, **Members**, **Moderation**, **Spam Protection**, **Storage** и **System**.

Несколько ключей намеренно доступны только в YAML — либо потому, что это учётные данные, либо потому, что они представляют собой списки, которые плохо выглядят в поле формы. Они отмечены ниже.

## Карта блоков {: #block-map }

| Блок | Вкладка админки | Что управляет |
|---|---|---|
| `maintenance` | General | Режим только для чтения и его баннер |
| `frontend` | General | Маршрут, количество элементов на странице, хронология, предложенные темы, навигация внутри страницы, устаревшие редиректы |
| `search` | General | Выбор движка |
| `comments` | General | Комментарии к статьям |
| `posting` | Posting | Окно редактирования, длина заголовка и текста |
| `tags` / `polls` / `reactions` | Posting | Метки тем, опросы, типы реакций и веса репутации |
| `attachments` | Posting | Ограничения размера, количества и суточного лимита, разрешённые расширения, очистка |
| `auth` / `security` | Members | Регистрация, подтверждение, правила паролей, captcha, доверенный хост |
| `account` | Members | Самостоятельное удаление участником и льготный период |
| `messages` | Members | Личные сообщения |
| `email` | Members | Ограничение частоты уведомлений по электронной почте |
| `moderation` | Moderation | Автоматическое скрытие по жалобам, порог предмодерации, дайджест модераторов |
| `retention` | Moderation | Сроки очистки |
| `spam` | Spam Protection | Пороги эвристик и ML-классификатор |
| `ai_spam` | Spam Protection | Ежечасная проверка ИИ |
| `provisional` | Spam Protection | Ограничения для новых учётных записей |
| `storage` | Storage | Локальное или S3-совместимое хранилище blob-объектов и CDN |
| `database` / `auto_migrate` | System | Подключение и политика миграций |
| `jobs` | System | Пакетная обработка фоновым процессом |
| `realtime` | System | Обновления в реальном времени и присутствие |
| `admin2` | System | Виджет панели управления админки |

## General {: #general }

### `maintenance`

Режим только для чтения. Участники могут просматривать форум, но не могут писать, ставить реакции или регистрироваться — на форуме или через [REST API](../rest-api/index.md).

```yaml
maintenance:
  enabled: false
  include_staff: false   # true также запрещает сотрудникам писать
  message: ""            # необязательное текстовое уведомление на баннере
```

### `frontend`

```yaml
frontend:
  enabled: false              # показывать публичные страницы форума
  route: /forum               # базовый маршрут
  topics_per_page: 25
  posts_per_page: 20
  members_per_page: 48
  online_window_minutes: 15   # окно активности для счётчика «кто онлайн»
  stats_cache_seconds: 60     # 0 — пересчитывать итоги форума при каждом просмотре
  topic_timeline: true        # шкала хронологии на страницах тем (компьютер)
  suggested_topics: 5         # 0 отключает
  htmx: true                  # навигация между страницами форума без полной перезагрузки
  legacy_redirects:
    enabled: false
    discourse:
      hosts: []               # старые хосты Discourse; пустой список отключает
      target_base: ''         # абсолютная база для целей; пусто — тот же хост
```

`legacy_redirects` подробно описаны в разделе [Миграция из Discourse](../discourse-import/index.md).

### `search`

```yaml
search:
  engine: builtin    # builtin | yetisearch
```

См. [Поиск](../search/index.md).

### `comments`

```yaml
comments:
  enabled: false
  category: null         # id категории форума, где хранятся ветки статей
  templates: []          # шаблоны страниц, принимающие комментарии, например [item]
  author: ""             # имя участника, от которого создаются темы ('' = System)
  max_inline: 20         # комментариев до ссылки «показать все»
  excerpt_length: 300    # символов фрагмента статьи в стартовом сообщении
```

См. [Комментарии к статьям](../article-comments/index.md).

## Posting {: #posting }

```yaml
posting:
  edit_window_minutes: 60   # 0 — авторы могут редактировать бессрочно
  min_title_length: 3
  max_title_length: 255
  min_body_length: 2
  max_body_length: 65535
```

### `tags`

```yaml
tags:
  enabled: true
  allow_create: staff   # staff | members | none
```

### `polls`

```yaml
polls:
  enabled: true    # опросы в новых темах
```

### `reactions`

Каждый тип реакции имеет **вес репутации**, который применяется к автору сообщения. Вес `0` означает, что реакция выразительная, но не влияет на репутацию.

```yaml
reactions:
  enabled: true
  types:
    like: 1
    heart: 1
    laugh: 0
    confused: 0
  # emoji:            # только YAML: необязательный эмодзи для типа; можно добавлять свои типы
  #   like: "👍"
  #   rocket: "🚀"
```

Типы — это просто названия: добавьте `wow`, `sad` и `angry` с эмодзи, и в панели выбора будет шесть реакций. Настройте их до [импорта из XenForo](../xenforo-import/index.md#reactions), и реакции XenForo перенесутся один к одному.

### `attachments`

```yaml
attachments:
  enabled: true             # загрузка файлов и изображений в сообщениях (только участники)
  max_size_kb: 4096         # на один файл
  max_per_post: 8
  member_daily_limit: 40    # 0 отключает
  draft_ttl_hours: 24       # неиспользованные загрузки истекают через это время
  purge_grace_days: 7       # осиротевшие файлы удаляются через столько дней после пометки
  # allowed_extensions:     # только YAML
  #   [png, jpg, gif, webp, svg, pdf, zip, txt, log, md, yaml, json, csv, mp4, mov]
```

Загрузки адресуются по содержимому, поэтому один и тот же файл, отправленный дважды, хранится один раз. О том, где физически лежат данные, см. [Файловое хранилище](../file-storage/index.md).

## Members {: #members }

### `auth`

```yaml
auth:
  enabled: true
  grav_members: true               # считать авторизованных пользователей Grav участниками форума
  allow_registration: true
  require_email_verification: true
  min_password_length: 8
  captcha: false                   # прозрачная проверка proof-of-work (нужен плагин Form)
```

### `security`

```yaml
security:
  site_url: ''    # абсолютный базовый URL для ссылок сброса и подтверждения
```

Задайте это значение в рабочем окружении. См. [Установку](../installation/index.md#set-your-trusted-site-url).

### `account`

```yaml
account:
  self_deletion: true       # разрешить участникам удалять свою учётную запись (удаление по GDPR)
  deletion_grace_days: 30   # 0 — удалять сразу после подтверждения
```

### `messages`

```yaml
messages:
  enabled: true
  max_participants: 5           # человек в переписке, включая инициатора
  max_threads_per_hour: 10      # 0 отключает
  max_messages_per_minute: 10   # 0 отключает
```

### `email`

```yaml
email:
  throttle_minutes: 15   # не более одного письма-уведомления участнику по теме за окно
```

## Moderation {: #moderation }

```yaml
moderation:
  new_member_post_threshold: 1     # предмодерация 'new_members' действует ниже этого числа сообщений
  auto_hide_flag_threshold: 3      # открытые жалобы, скрывающие сообщение до проверки (0 отключает)
  mod_digest: true                 # ежедневное письмо модераторам
  mod_digest_time: '00:00'         # местное время, ЧЧ:ММ
  mod_digest_timezone: UTC         # например, America/Denver
```

### `retention`

Все сроки соблюдает фоновый процесс. `0` означает «хранить бессрочно».

```yaml
retention:
  purge_deleted_after_days: 30            # окончательно удалять мягко удалённый контент
  purge_spam_after_days: 30               # окончательно удалять контент из очереди спама
  anonymize_ip_after_days: 90             # очищать сохранённые IP-адреса
  purge_flag_snapshots_after_days: 180    # удалять снимки контента из закрытых жалоб
  purge_read_notifications_after_days: 90
  purge_sent_emails_after_days: 7         # удалять отправленные письма
```

См. [Конфиденциальность и хранение](../privacy-and-retention/index.md).

## Защита от спама {: #spam-protection }

Полностью описана в разделе [Защита от спама](../spam-protection/index.md). Кратко:

```yaml
spam:
  enabled: true
  pending_threshold: 30
  spam_threshold: 60
  min_submit_seconds: 3
  max_links: 2
  new_member_post_count: 3
  duplicate_window_minutes: 60
  max_posts_per_minute: 6      # 0 отключает
  max_topics_per_hour: 12      # 0 отключает
  max_flags_per_hour: 10       # 0 отключает
  classifier:
    enabled: false
    url: ""
    timeout_ms: 800
    min_probability: 0.6
    weight: 60
    trust_post_count: 20

ai_spam:
  enabled: false
  provider: ""
  model: ""
  lookback_hours: 24
  max_posts: 50
  trust_post_count: 20
  min_confidence: 0.85

provisional:
  enabled: true
  post_threshold: 5
  min_age_hours: 24
  strip_links: true
  hide_profile: true
  noindex_profile: true
  newbie_badge: true
  block_attachments: true
  block_messages: true
```

## Storage {: #storage }

```yaml
storage:
  driver: local     # local | s3
  s3:
    endpoint: ""    # например, https://<account>.r2.cloudflarestorage.com
    bucket: ""
    region: auto
    access_key: ""
    secret_key: ""
    prefix: forum
    public_url: ""  # публичный домен или домен CDN
```

См. [Файловое хранилище](../file-storage/index.md).

## System {: #system }

### `database` и `auto_migrate` {: #database-and-auto-migrate }

Описаны в разделе [Установка](../installation/index.md#database-setup).

### `jobs`

```yaml
jobs:
  enabled: true
  max_jobs_per_run: 50
  max_seconds_per_run: 50
  stale_lock_seconds: 300
```

Значение `max_seconds_per_run` должно быть заметно ниже лимита времени PHP CLI. `stale_lock_seconds` задаёт, сколько времени блокировка фонового процесса считается действительной, прежде чем другой запуск сможет её забрать; именно это восстанавливает очередь после сбоя.

### `realtime`

```yaml
realtime:
  enabled: true                # не действует без плагина sync
  new_replies_threshold: 5     # при большем числе непросмотренных ответов показывать кнопку загрузки
  presence:
    enabled: true              # счётчики «кто смотрит» и индикаторы набора текста
    heartbeat_seconds: 15
```

См. [Обновления в реальном времени](../realtime/index.md).

### `admin2`

```yaml
admin2:
  dashboard: true    # показывать виджет Forum Activity на общей панели управления админки
```
