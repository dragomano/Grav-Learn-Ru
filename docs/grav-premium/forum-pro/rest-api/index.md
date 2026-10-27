---
title: "REST API"
description: "Маршруты Forum Pro под /api/v1/forum-pro, как ключ пишет от имени участника, какие маршруты требуют права Grav и все маршруты, сгруппированные по разделам."
tags:
  - "Forum Pro"
  - "REST API"
---

# REST API {: #rest-api }

Forum Pro добавляет свои маршруты в плагин API Grav под `/api/v1/forum-pro/`. Область «Форум» в Admin Next построена на этих маршрутах, и участники могут через них читать, создавать темы, отвечать, редактировать, ставить реакции и жаловаться — то есть скрипт или бот может делать то же, что участник на форуме. Пути ниже опускают префикс `/api/v1`.

## Аутентификация {: #authenticate }

Используйте любые учётные данные, которые принимает плагин API:

- **Ключ API**, отправляемый как `X-API-Key`. Создайте его в Admin Next (ваша учётная запись, затем **API Keys**) или на сервере через `bin/plugin api keys:generate --user=<username>`.
- **JWT** из `POST /auth/token`, отправляемый как `X-API-Token` (так делает Admin Next).

```bash
curl -k "https://example.test/api/v1/forum-pro/structure" -H "X-API-Key: $KEY"
```

## Кто что может вызывать {: #who-may-call-what }

Есть два вида маршрутов, и авторизуются они по-разному.

**Маршруты контента** (чтение и запись тем и сообщений) действуют от имени участника форума, связанного с вызывающей учётной записью. Они не требуют никаких прав Grav, даже `api.access`. Форум решает сам, в точности по тем правилам, которые применяет на сайте: [права категории](../administration/index.md#category-permissions) для чтения, создания тем и ответов, закрытые темы и категории, окно редактирования, блокировки, ограничения частоты, оценка спама, премодерация и ограничения [предварительной учётной записи](../spam-protection/index.md). Сообщение, созданное через API, — такое же, как созданное на форуме: оно отправляет уведомления, учитывается в счётчиках и появляется в реальном времени точно так же.

Участник — это связанная с учётной записью учётная запись форума, создаваемая при первом обращении так же, как при первом посещении ([учётные записи Grav](../members-and-accounts/index.md#grav-accounts)). Запись в контент отклоняется, когда:

- `auth.grav_members` равен `false`, то есть учётные записи Grav не являются участниками форума (`403 You must be logged in to …`).
- Учётная запись форума приостановлена или удалена.
- Учётная запись ещё не выбрала имя пользователя на форуме.
- Категория не разрешает это группам участника.
- Форум в [режиме только для чтения](../configuration/index.md#maintenance) (`503`). Сотрудники продолжают писать, если не включён `maintenance.include_staff`, — как и на форуме.

Учётная запись, у которой есть `admin.forum-pro.moderate`, `admin.forum-pro.admin` или `api.super`, либо [роль модератора](../moderation/index.md#moderator-roles), считается на этих маршрутах сотрудником и в пределах своей области модерации может редактировать и удалять чужие сообщения и менять темы.

**Маршруты управления** (структура, очередь, блокировки, модераторы, группы, теги, участники, приглашения, статус и метрики) требуют `api.access` плюс право Forum Pro ([Права](../administration/index.md#permissions)). В таблицах **admin** — это `admin.forum-pro.admin`, а **moderate** — `admin.forum-pro.moderate`; `admin` и `api.super` считаются обоими. Режим только для чтения не закрывает их, как не закрывает и область «Форум» в Admin Next.

### Ключи с областями действия {: #keys-with-scopes }

Ключ, созданный без областей действия, а также JWT и сессия имеют полный доступ учётной записи: ключ участника пишет от имени этого участника, а ключ модератора — модерирует. Ключ, созданный с областями действия, ограничен ими:

| Область действия | Что разрешает ключ с областью |
|---|---|
| (ни одной из них) | Читать форум от имени своего участника |
| `api.forum-pro.post` | Писать на маршрутах контента: создавать темы, отвечать, редактировать, удалять, ставить реакции и жаловаться |
| `admin.forum-pro.moderate` | Сохранять полномочия сотрудника на маршрутах контента и вызывать маршруты **moderate** |
| `admin.forum-pro.admin` | То же самое и вызывать маршруты **admin** |

Область действия никогда не добавляет полномочий, которых у учётной записи нет. Ключу модератора, которому нужно менять темы, нужны и `api.forum-pro.post`, и `admin.forum-pro.moderate`; с одним только `api.forum-pro.post` он пишет как обычный участник. Области задаются при создании ключа — через `POST /users/{username}/api-keys` и `{"name": "Forum bot", "scopes": ["api.forum-pro.post"]}`.

Для бота или интеграции создайте отдельную учётную запись Grav, один раз войдите на форум под ней, чтобы выбрать имя пользователя, и создайте ключ на этой учётной записи с областью `api.forum-pro.post`. Тогда её сообщения будут носить собственное имя, и больше она ничего не сможет.

## Публичное чтение {: #public-reads }

Четыре маршрута чтения — `GET /forum-pro/structure`, `GET /forum-pro/categories/{id}/topics`, `GET /forum-pro/topics/{id}` и `GET /forum-pro/topics/{id}/posts` — отвечают без учётных данных. Гость получает то, что гость может читать на форуме; если отправить ключ, ответ включает и то, что может читать этот участник. Тема, которой вызывающий не видит, отвечает `404` — так же, как несуществующая. При включённом `gate.private` у форума нет публичного API, и этим маршрутам тоже нужны учётные данные.

## Соглашения {: #conventions }

- Ответы оборачивают полезную нагрузку в `data`. Списки принимают `page` и `per_page` и несут `meta.pagination` (`page`, `per_page`, `total`, `total_pages`) и `links`.
- Ошибки — problem details (`application/problem+json`) с `status`, `title` и `detail`. Причина отказа форума находится в `detail`: `422 Title must be between 3 and 255 characters`, `403 You are not allowed to reply in this category`, `409 This topic is closed`.
- Время — секунды epoch в UTC.
- Тела сообщений передаются в Markdown в `body_md` и рендерятся на сервере, вместе с упоминаниями и цитатами. Чтение возвращает уже отрендеренный `body_html`.
- Записи отправляют заголовок `X-Invalidates` (`forum:topic:<id>`, `forum:category:<id>`, `forum:structure`, …), чтобы Admin Next перезапрашивал изменившееся. `GET /forum-pro/topics/{id}` отправляет `ETag`.

## Маршруты {: #routes }

### Чтение {: #reading }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /forum-pro/structure` | публичный | Разделы со своими категориями (и подкатегориями в `children`), счётчики и последнее сообщение каждой категории |
| `GET /forum-pro/categories/{id}/topics` | публичный | Темы категории, сначала закреплённые. `sort` — `bumped` (по умолчанию), `created` или `replies`; `page`, `per_page` (не более 100). |
| `GET /forum-pro/topics/{id}` | публичный | Одна тема вместе со своей `category` |
| `GET /forum-pro/topics/{id}/posts` | публичный | Её сообщения, сначала самые старые, с автором, рангом, значками, `reactions`, `my_reactions` и `replies`. `page`, `per_page`. |

### Темы и сообщения {: #topics-and-posts }

| Маршрут | Право | Что делает |
|---|---|---|
| `POST /forum-pro/topics` | участник | `{category_id, title, body_md}`: создать тему. Отвечает `201` с `{topic, post}`. |
| `POST /forum-pro/topics/{id}/posts` | участник | `{body_md, parent_post_id, quoted_post_ids}`: ответить. `parent_post_id` вкладывает ответ под сообщение; оба должны принадлежать теме. Отвечает `201` с сообщением. |
| `PATCH /forum-pro/posts/{id}` | участник | `{body_md}`: отредактировать сообщение. Его автор — в пределах `posting.edit_window_minutes`, сотрудник — в любое время. Прежнее содержимое сохраняется как ревизия. |
| `DELETE /forum-pro/posts/{id}` | участник | `{reason}` необязателен: удалить сообщение. Его автор или сотрудник. Первое сообщение темы удалить нельзя — удаляйте тему. |
| `POST /forum-pro/posts/{id}/react` | участник | `{type}`: переключить реакцию, одну из ключей `reactions.types` (`like`, `heart`, `laugh`, `confused` по умолчанию). Не на своём сообщении. Отвечает `{reacted, type, counts}`. |
| `POST /forum-pro/posts/{id}/report` | участник | `{reason, message}`: пожаловаться на сообщение модераторам. `reason` — `spam`, `abuse`, `off_topic` или `other`; `message` — не более 1000 символов. Один раз на сообщение. |
| `PATCH /forum-pro/topics/{id}` | сотрудник | Любое из `title`, `slug`, `is_pinned`, `state` (`open`, `closed`, `archived`, `hidden`) и `category_id` (переместить) |
| `DELETE /forum-pro/topics/{id}` | сотрудник | `{reason}` необязателен: удалить тему |
| `POST /forum-pro/topics/{id}/merge` | сотрудник | `{target_topic_id}`: перенести все сообщения в другую тему. Отвечает целевой темой. |
| `POST /forum-pro/topics/{id}/split` | сотрудник | `{post_ids, title, category_id}`: перенести сообщения в новую тему, в эту категорию, если `category_id` не сказано иначе. Отвечает `201` с новой темой. |

Тема или ответ может появиться в состоянии `pending`, а не опубликованным — когда категория проводит премодерацию участника или оценка спама удерживает его на проверку. Тогда он появляется после одобрения модератора.

### Модерация {: #moderation }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /forum-pro/moderation/counts` | moderate | Сколько ожидает в каждой вкладке очереди |
| `GET /forum-pro/moderation/queue` | moderate | Одна вкладка очереди: `tab` — `pending` (по умолчанию), `spam`, `deleted` или `flagged`; `page`, `per_page` |
| `POST /forum-pro/moderation/posts/{id}` | moderate | `{action, reason}`: `approve`, `hide`, `spam` или `delete` для одного сообщения. Открытые жалобы на него закрываются вместе с ним. |
| `POST /forum-pro/moderation/bulk` | moderate | `{action, post_ids, reason}`: то же самое от 1 до 100 сообщений. Отвечает `{done, errors}`. |
| `POST /forum-pro/moderation/flags/{id}/dismiss` | moderate | Закрыть одну жалобу |
| `GET /forum-pro/bans` | moderate | Активные блокировки; `all=1` включает истёкшие и отозванные. `page`, `per_page`. |
| `POST /forum-pro/bans` | moderate | `{user_id, scope, scope_id, reason, expires_at, content_disposition}`. `scope` — `global` (по умолчанию), `section` или `category`; `content_disposition` — `keep` (по умолчанию), `hide_all`, `anonymize` или `delete_all` ([судьба контента](../moderation/index.md#content-disposition)). |
| `DELETE /forum-pro/bans/{id}` | moderate | `{reason}` необязателен: снять блокировку |
| `GET /forum-pro/appeals` | moderate | Открытые апелляции на блокировки |
| `POST /forum-pro/appeals/{id}` | moderate | `{accept, note}`: принять или отклонить апелляцию |

### Структура {: #structure }

| Маршрут | Право | Что делает |
|---|---|---|
| `POST /forum-pro/sections` | admin | `{title, description, position, icon}` |
| `PATCH /forum-pro/sections/{id}` | admin | Любое из того же |
| `DELETE /forum-pro/sections/{id}` | admin | Удалить пустой раздел. У раздела с категориями ответ `409`. |
| `POST /forum-pro/categories` | admin | `{section_id, title, parent_id, description, icon, perms, state, position}`. `parent_id` делает категорию подкатегорией в том же разделе. |
| `PATCH /forum-pro/categories/{id}` | admin | Любое из того же, плюс `slug`. Новый слаг оставляет перенаправление со старого. |
| `DELETE /forum-pro/categories/{id}` | admin | Удалить категорию без тем и подкатегорий, иначе `409` |
| `POST /forum-pro/structure/reorder` | admin | `{sections: [{id, position}], categories: [{id, position}]}` |
| `GET /forum-pro/icons` | admin | Все имена иконок, доступные разделу или категории |

### Модераторы, группы и теги {: #moderators-groups-and-tags }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /forum-pro/moderators` | admin | Все роли модераторов |
| `POST /forum-pro/moderators` | admin | `{user_id or username, scope, scope_id}`: выдать роль. `scope` — `global`, `section` или `category`. |
| `DELETE /forum-pro/moderators/{id}` | admin | Отозвать одну |
| `GET /forum-pro/groups` | admin | Все группы |
| `POST /forum-pro/groups` | admin | `{title, description, color, icon, position, grav_group}`. `grav_group` связывает группу с группой Grav ([Группы](../members-and-accounts/index.md#groups)). |
| `PATCH /forum-pro/groups/{id}` | admin | Любое из того же |
| `DELETE /forum-pro/groups/{id}` | admin | Удалить группу |
| `GET /forum-pro/groups/{id}/members` | admin | Её участники. `page`, `per_page`. |
| `POST /forum-pro/groups/{id}/members` | admin | `{username}`: добавить участника |
| `DELETE /forum-pro/groups/{id}/members/{userId}` | admin | Исключить одного |
| `GET /forum-pro/tags` | admin | Все теги со счётчиком использований |
| `PATCH /forum-pro/tags/{id}` | admin | `{title}`: переименовать |
| `POST /forum-pro/tags/{id}/merge` | admin | `{target_id}`: слить с другим тегом |
| `DELETE /forum-pro/tags/{id}` | admin | Удалить и снять со всех тем |

### Участники и приглашения {: #members-and-invitations }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /forum-pro/members` | admin | Таблица участников. `search`, `group`, `badge`, `moderators=1`, `sort` (`joined`, `active`, `state`, `rank`, `posts`, `topics`, `rep`, `badges`), `dir` (`asc` или `desc`), `page`, `per_page`. |
| `GET /forum-pro/members/meta` | admin | Группы, ранги и значки, которые предлагают экраны участников |
| `POST /forum-pro/members` | admin | `{username, email, group_ids}`: создать участника силами форума и отправить ему ссылку для задания пароля. Отвечает `201` с `{member, email_sent, email_error}`. |
| `GET /forum-pro/members/{id}` | admin | Один участник целиком |
| `PATCH /forum-pro/members/{id}` | admin | Любое из `display_name`, `email`, `email_verified`, `title`, `rank_id`, `custom_rank`, `custom_rank_icon` и `state` (`active` или `suspended`). Участник, входящий с учётной записью сайта, сохраняет её адрес электронной почты. |
| `DELETE /forum-pro/members/{id}` | admin | Удалить участника |
| `POST /forum-pro/members/{id}/password` | admin | `{password}`: задать пароль участнику форума |
| `POST /forum-pro/members/{id}/set-password-link` | admin | Отправить участнику без пароля свежую ссылку для задания пароля |
| `POST /forum-pro/members/{id}/badges` | admin | `{badge_id}`: выдать значок |
| `DELETE /forum-pro/members/{id}/badges/{badgeId}` | admin | Забрать значок |
| `GET /forum-pro/invites` | admin | Приглашения в закрытый форум. `status` — `open` (по умолчанию) или `all`. |
| `POST /forum-pro/invites` | admin | `{email, group_ids}`: пригласить кого-то |
| `POST /forum-pro/invites/{id}/resend` | admin | Отправить ещё раз |
| `DELETE /forum-pro/invites/{id}` | admin | Отозвать приглашение |

### Статус и метрики {: #status-and-metrics }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /forum-pro/status` | admin | Вкладка «Статус»: база данных, схема, задания и состояние планировщика |
| `GET /forum-pro/dashboard/metrics` | admin | Вкладка «Панель управления». `range` в днях, по умолчанию 30. |

## Быстрая проверка через curl {: #a-quick-check-with-curl }

```bash
KEY=...   # ключ API учётной записи, от имени которой пишет бот
BASE=https://example.test/api/v1/forum-pro

# Найти идентификатор категории
curl -k "$BASE/structure" -H "X-API-Key: $KEY"

# Создать в ней тему
curl -k -X POST "$BASE/topics" -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"category_id": 3, "title": "Nightly build 2.0.4 is out", "body_md": "Changes since yesterday: ..."}'

# Ответить в тему 9434
curl -k -X POST "$BASE/topics/9434/posts" -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"body_md": "Thanks, that fixed it for me."}'
```

## См. также {: #related }

- [Администрирование](../administration/index.md#permissions)
- [Участники и учётные записи](../members-and-accounts/index.md#grav-accounts)
- [Модерация](../moderation/index.md)
- [Grav API: аутентификация](../../../api/authentication/index.md)
