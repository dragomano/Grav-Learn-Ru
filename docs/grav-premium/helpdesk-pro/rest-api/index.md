---
title: "REST API"
description: "Маршруты Helpdesk Pro под /api/v1/helpdesk-pro, кто может их вызывать, JSON заявки и хронологии и защита от конфликтов."
tags:
  - "Helpdesk Pro"
  - "Поддержка"
  - "REST API"
---

# REST API {: #rest-api }

Helpdesk Pro добавляет свои маршруты в плагин API Grav под `/api/v1/helpdesk-pro/`. Рабочая область сотрудников полностью построена на этих маршрутах, поэтому всё, что она делает, может делать и скрипт, и ИИ-клиент (см. [MCP](../mcp/index.md)). Пути ниже опускают префикс `/api/v1`.

## Аутентификация {: #authenticate }

Используйте любые учётные данные, которые принимает плагин API:

- **Ключ API**, отправляемый как `X-API-Key`. Создайте его в Admin Next (ваша учётная запись, затем **API Keys**) или на сервере через `bin/plugin api keys:generate --user=<username>`. Ключ, созданный со списком областей доступа, ограничен ими: выдайте ему `helpdesk-pro.desk` и любые другие нужные действия `helpdesk-pro.*`. Ключ без области рабочей области не дойдёт ни до одного маршрута Helpdesk Pro, даже если его владелец администратор.
- **JWT** из `POST /auth/token`, отправляемый как `X-API-Token` (так делает Admin Next).

Заявки, созданные и ответы, написанные с ключом API, записывают канал `api`; то же самое из Admin Next записывает `staff`.

```bash
curl -k "https://example.test/api/v1/helpdesk-pro/tickets?assignee=me" \
  -H "X-API-Key: $KEY"
```

## Кто что может вызывать {: #who-may-call-what }

Каждому маршруту нужен `api.access` плюс право из его строки ([Проекты и права](../projects-and-permissions/index.md#permissions)). `api.super` считается обладателем всех прав Helpdesk Pro.

Маршруты заявок, помеченные **client**, также отвечают учётной записи, у которой есть `api.access`, но нет прав Helpdesk Pro. Такая учётная запись — клиент и получает ровно то, что показал бы портал: заявки, которые он отправил или в копии которых был (и общие заявки проектов, в которых состоит), только публичные ответы и публичную активность, без заметок, меток и адресов электронной почты. Он может отвечать, отключать уведомления, отмечать своё обращение решённым и открывать его снова. Заявка, которую он не может читать, отвечает `404`, как несуществующая.

Сотрудники видят заметки и внутреннюю активность заявки только в проектах, где работают. Заявка вне этих проектов для них тоже `404`.

## Соглашения {: #conventions }

- Ответы оборачивают полезную нагрузку в `data`. Списки листаются через `page` и `per_page` (не более 100) и несут `meta.pagination` и `links`.
- Ошибки — problem details (`application/problem+json`) с `status`, `title` и `detail`. Ошибки проверки (`422`) добавляют `errors: [{field, message}]`.
- Время — секунды epoch.
- Люди появляются как `{id, name, kind}`, плюс `email`, когда вызывающий — сотрудник по этой заявке.
- Записи отправляют заголовок `X-Invalidates` (`helpdesk-pro:tickets`, `helpdesk-pro:ticket:<id>`, `helpdesk-pro:projects`, …), чтобы Admin Next перезапрашивал изменённое.
- `GET /helpdesk-pro/tickets/{id}` отправляет `ETag` редактируемых полей заявки. Верните его как `If-Match` в `PATCH`, чтобы получить `409`, когда кто-то тем временем изменил одно из этих полей. Новые ответы и заметки его не меняют. Люди и организации работают так же.

## Заявка {: #the-ticket }

Строки списка и деталь используют одну форму:

```json
{
  "id": 1042, "subject": "Can't reset my password", "kind": "support",
  "project": {"id": 1, "slug": "support", "name": "Support"},
  "status": {"id": 2, "name": "Open", "category": "open", "color": "#22c55e"},
  "priority": 1, "visibility_scope": "requester", "channel": "email", "triage_state": "accepted",
  "requester": {"id": 55, "name": "Ada Lovelace", "kind": "client", "email": "ada@example.com"},
  "assignee": {"id": 3, "name": "Anna", "kind": "staff"},
  "labels": [{"id": 4, "name": "billing", "color": "#f59e0b"}],
  "first_message_id": 7001, "last_public_message_id": 7010,
  "counts": {"public": 3, "attachments": 2, "notes": 1},
  "times": {"created_at": 0, "last_public_at": 0, "resolved_at": null, "closed_at": null,
            "updated_at": 0, "last_activity_at": 0, "last_requester_at": 0, "last_staff_reply_at": 0,
            "first_response_at": 0, "status_changed_at": 0, "waiting_since": null},
  "me": {"watching": true, "muted": false, "unread": true},
  "duplicate_of_id": null, "merged_into_id": null, "reopen_count": 0, "deleted": false,
  "extra": {}
}
```

`triage_state`, `labels`, `counts.notes`, времена сотрудников, `duplicate_of_id`, `merged_into_id`, `reopen_count` и `deleted` есть только у сотрудников, работающих с проектом заявки. Деталь добавляет `participants` и `etag`, а `timeline` — когда его запросили.

`extra` несёт то, что прикрепляют функции и дополнения:

| Ключ | Что содержит |
|---|---|
| `extra.fields` | Значения [пользовательских полей](../custom-fields/index.md) заявки, которые вызывающий может видеть, ключ → значение |
| `extra.organization` | [Организация](../organizations/index.md) заявки как `{id, name}`, только для сотрудников |
| `extra.sla` | Состояние [SLA](../sla/index.md) заявки, только для сотрудников |
| `extra.csat` | [Оценка](../customer-ratings/index.md) заявки полностью для сотрудников; собственный ответ заявителя для заявителя |

## Хронология {: #the-timeline }

`GET /helpdesk-pro/tickets/{id}/timeline` возвращает сообщения и активность вперемешку, начиная со старого:

```json
[
  {"type": "message", "id": 7001, "ticket_id": 1042, "visibility": "public",
   "author": {"id": 55, "name": "Ada Lovelace", "kind": "client"}, "source": "email",
   "body_html": "<p>…</p>", "body_md": "…", "stripped": false, "has_original": true,
   "edited_at": null, "created_at": 0, "attachments": []},
  {"type": "activity", "id": 301, "action": "status", "actor": {"id": 3, "name": "Anna", "kind": "staff"},
   "before": {"status_id": 1, "category": "new", "name": "New"},
   "after": {"status_id": 2, "category": "open", "name": "Open"},
   "visibility": "public", "created_at": 0}
]
```

`visibility` — `public` для ответов и `internal` для заметок. `body_md` и `has_original` только для сотрудников. `attachments` перечисляет файлы сообщения; клиент получает только публичные файлы публичных сообщений, без `api_path`.

`action` активности — одно из `created`, `status`, `priority`, `kind`, `subject`, `assignee`, `labels`, `project`, `visibility_scope`, `triage`, `merged`, `merged_from`, `participant`, `message_edited`, `message_deleted`, `deleted`, `restored`, `erased` или `fields`. Клиенты видят только публичную активность: решение, закрытие, повторное открытие, отклонение и объединение.

## Защита от конфликтов {: #the-collision-guard }

Когда рабочая область отправляет ответ, она передаёт `based_on_message_id` — новейшее показанное публичное сообщение. Если существует более новое публичное сообщение (клиент написал тем временем), `POST /helpdesk-pro/tickets/{id}/messages` отвечает `409` и ничего не публикует:

```json
{"status": 409, "title": "Conflict", "detail": "The conversation has new messages since you started writing",
 "code": "thread_changed", "messages": [{"type": "message", "id": 7011, "visibility": "public", "…": "…"}]}
```

`messages` содержит новые публичные сообщения в форме хронологии. Отправьте снова с `force: true`, чтобы всё равно опубликовать.

## Маршруты {: #routes }

### Рабочая область {: #desk }

| Маршрут | Право | Ответ |
|---|---|---|
| `GET /helpdesk-pro/config` | desk | Настройки интерфейса: `routes`, `assetBase`, `default_view`, `board`, `batch_minutes`, `semantic`, `realtime` и ваши `prefs`. Никогда секрет. |
| `GET /helpdesk-pro/bootstrap` | desk | `{config, me, projects, statuses, labels, staff, saved_views, counts}` одним вызовом. |
| `GET /helpdesk-pro/counts` | desk | `triage`, `mine_needing_reply`, `unassigned`, `by_category`, `notifications_unread`, `inbound_attention` и `channels_attention` (оба 0 без `helpdesk-pro.settings`). |
| `GET /helpdesk-pro/badge` | desk | `{count, triage, mine_needing_reply}` для значка боковой панели Admin Next. Без инструмента MCP. |
| `GET /helpdesk-pro/status` | settings | Отчёт о состоянии (см. [Устранение неполадок](../troubleshooting/index.md#the-status-report)). |
| `POST /helpdesk-pro/mail/test` | settings | Отправляет тестовое письмо на `{to}` (см. [Исходящую почту](../outbound-email/index.md)). |

Счётчики: `triage` — заявки, ждущие в сортировке (ноль, если у вас нет `helpdesk-pro.triage`); `mine_needing_reply` — принятые заявки, назначенные вам, в `new`, `open` или `waiting_us`, чьё последнее публичное сообщение пришло от клиента; `unassigned` — принятые заявки в этих категориях без исполнителя; `by_category` — живые заявки по категориям статусов.

### Заявки {: #tickets }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `GET /helpdesk-pro/tickets` | desk, client | Фильтры ниже, `sort`, `order`, `page`, `per_page` | Строки заявок. Сотрудники также получают `meta.facets`. |
| `POST /helpdesk-pro/tickets` | desk | `{project_id, subject, body, requester: {id} or {email, name}, kind, priority, assignee_id, labels, visibility_scope, notify_requester, attachments_token, fields}` | `201` с заявкой и `Location`. Заявки, созданные сотрудниками, пропускают сортировку; без `requester` заявитель — вы. Обязательные пользовательские поля здесь не проверяются. |
| `GET /helpdesk-pro/tickets/{id}` | desk, client | `include=timeline` | Заявка с `participants` и `etag`. Запрос хронологии также отмечает заявку и ваши уведомления о ней прочитанными. |
| `PATCH /helpdesk-pro/tickets/{id}` | desk, client | `If-Match`; любое из `subject, status_id, priority, kind, project_id, assignee_id, labels, add_labels, remove_labels, visibility_scope` | Заявка. Клиент может только отметить решённой или открыть снова. |
| `DELETE /helpdesk-pro/tickets/{id}` | tickets.delete | `?permanent=1` также требует settings | `204`. Мягкое удаление, если не постоянное. |
| `POST /helpdesk-pro/tickets/{id}/restore` | tickets.delete | | Заявка. |
| `GET /helpdesk-pro/tickets/{id}/timeline` | desk, client | `after_id`, `after_activity_id`, `mark_read=1` | Элементы хронологии. |
| `POST /helpdesk-pro/tickets/{id}/mute` | desk, client | `{muted: bool}` | `{muted}` для вас. |
| `POST /helpdesk-pro/tickets/bulk` | desk | См. [Сохранённые ответы и представления](../saved-replies-and-views/index.md#api-and-mcp) | `{updated, failed}` |

Фильтры списка, каждое значение одно или несколько через запятую:

| Фильтр | Значения |
|---|---|
| `q` | Текст темы или номер заявки (`1042` или `#1042`). Для сотрудников также совпадает с именем или адресом заявителя и с перепиской и заметками через поисковый индекс, и каждая строка несёт `matched_in_note`. Для клиентов это только совпадение по теме. |
| `project`, `status`, `label`, `requester` | Id |
| `category` | `new`, `open`, `waiting_client`, `waiting_us`, `resolved`, `closed` |
| `assignee` | `me`, `none` или id людей |
| `priority` | `0`–`3` или `low`, `normal`, `high`, `urgent` |
| `kind`, `channel` | Типы и каналы |
| `triage` | `pending`, `accepted`, `declined`, `duplicate` |
| `organization` | Id организаций или `none` |
| `sla` | `breached`, `due_soon` (только сотрудники) |
| `unread=1`, `deleted=1` | Непрочитанные заявки; удалённые заявки (нужно `helpdesk-pro.tickets.delete`) |
| `cf_<key>` | Пользовательское поле (только сотрудники): значения вариантов для select или multi-select, `1` или `0` для флажка, `from..to` для даты, точное значение в остальных случаях. Неизвестный ключ игнорируется. |

Сортировка: `updated` (по умолчанию), `created`, `priority`, `waiting`, `id` или (сотрудники, при SLA) `due`; `order` — `desc` (по умолчанию) или `asc`. `meta.facets` считает весь отфильтрованный набор по `category`, `assignee`, `project`, `priority`, `kind`, `label` и `sla`; каждый фасет игнорирует собственный фильтр.

```bash
curl -k "https://example.test/api/v1/helpdesk-pro/tickets?category=new,open&assignee=me&per_page=50" -H "X-API-Key: $KEY"
```

### Сообщения {: #messages }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `POST /helpdesk-pro/tickets/{id}/messages` | desk, client | `{body, visibility: public or internal, status_after_id, based_on_message_id, force, attachments_token, saved_reply_id}` | `201` с сообщением и его `attachments`; `409` `thread_changed` |
| `PATCH /helpdesk-pro/messages/{id}` | desk | `{body}` | Сообщение. Свои сообщения или любые с `helpdesk-pro.tickets.delete`. |
| `DELETE /helpdesk-pro/messages/{id}` | tickets.delete | | `204` |
| `GET /helpdesk-pro/messages/{id}/original` | desk | | Исходное письмо за сообщением |

`visibility: public` (по умолчанию) — ответ: заявитель и клиенты в копии читают его и получают по почте. `visibility: internal` — заметка: только сотрудники. Клиенты могут только отвечать.

```bash
curl -k -X POST https://example.test/api/v1/helpdesk-pro/tickets/1042/messages \
  -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"body": "Fixed in 2.1.3, please update.", "status_after_id": 5}'

curl -k -X POST https://example.test/api/v1/helpdesk-pro/tickets/1042/messages \
  -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"body": "Customer is on the legacy plan.", "visibility": "internal"}'
```

### Участники {: #participants }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `GET /helpdesk-pro/tickets/{id}/participants` | desk | | `[{person, role, reason, muted, created_at}]`: заявитель, копии и наблюдатели |
| `POST /helpdesk-pro/tickets/{id}/participants` | desk | `{person_id or email, role: cc or watcher}` | Все участники заявки. Копия должна быть клиентом (неизвестный адрес становится новым человеком); наблюдатель — сотрудник, работающий с проектом. |
| `DELETE /helpdesk-pro/tickets/{id}/participants/{personId}` | desk | | `204`. Заявителя удалить нельзя (`409`). |

### Люди {: #people }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `GET /helpdesk-pro/people` | desk | `q, kind, state, page, per_page` или `q, for=requester` | Люди: `{id, name, display_name, email, email_verified, kind, state, account, locale, timezone, last_seen_at, created_at, updated_at}`. С `for=requester` — одна запись на адрес с добавленными `send_as`, `unverified` и `email_proven`. |
| `POST /helpdesk-pro/people` | people.manage | `{email, name}` | `201` с человеком; `409`, когда адрес известен |
| `GET /helpdesk-pro/people/{id}` | desk | | Человек с `tickets` (десять последних, которые вы можете читать), `ticket_total` и `ETag` |
| `PATCH /helpdesk-pro/people/{id}` | people.manage | `If-Match`; `{name, email, locale, timezone, state: active or blocked}` | Человек |

Объединение, удаление, юридические удержания и связи учётных записей — в разделе [Люди и конфиденциальность](../people-and-privacy/index.md#api-and-mcp).

### Проекты {: #projects }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `GET /helpdesk-pro/projects` | desk | `archived=1` | Проекты, в которых вы работаете, по порядку |
| `POST /helpdesk-pro/projects` | projects.manage | `{name, slug, description, visibility, staff_access, intake, triage_required, default_kind, default_assignee_id, board_enabled, color, icon, email_alias, kb_category}` | `201` с проектом |
| `POST /helpdesk-pro/projects/reorder` | projects.manage | `{ids}`, каждый проект один раз | `204` |
| `GET /helpdesk-pro/projects/{id}` | desk | | Проект с `members: {people: [{person, role}], groups: [{group, role}]}` |
| `PATCH /helpdesk-pro/projects/{id}` | projects.manage | Любое поле проекта, `archived` | Проект |
| `DELETE /helpdesk-pro/projects/{id}` | projects.manage | | `204`, когда он был пуст и удалён; `200` с `result: archived`, когда в нём были заявки |
| `PUT /helpdesk-pro/projects/{id}/members` | projects.manage | `{people: [{id, role}], groups: [{group, role}]}`, роли `agent` или `client` | Новые участники. Заменяет всех. |
| `GET /helpdesk-pro/projects/{id}/board` | desk | Фильтры списка плюс `closed` и `limit` | `{project, columns: [{status, tickets, total}], closed: {shown, total}, facets}`. Проект без доски — `404`. |

### Пользовательские поля {: #custom-fields }

| Маршрут | Право | Запрос | Ответ |
|---|---|---|---|
| `GET /helpdesk-pro/fields` | desk, client | `project`, `archived=1` (сотрудники) | Определения в порядке списка: `{id, key, label, help, type, options, required, visibility, projects, position, list_column, filterable, archived, value_count}`. Клиент видит публичные и readonly поля с меньшим числом ключей. |
| `POST /helpdesk-pro/fields` | projects.manage | `{key, label, type, help, options, required, visibility, projects, list_column, position}` | `201` с полем |
| `PUT /helpdesk-pro/fields/order` | projects.manage | `{ids: [3, 1, 2]}` | Каждое поле в новом порядке |
| `GET /helpdesk-pro/fields/{id}` | desk, client | | Поле |
| `PATCH /helpdesk-pro/fields/{id}` | projects.manage | Любое поле создания, кроме `key` | Поле. Тип меняется, только пока ни одна заявка не хранит значение. |
| `DELETE /helpdesk-pro/fields/{id}` | projects.manage | `permanent=1` | `204`, когда у поля не было значений; `200` с `result: archived`, когда они были. `permanent=1` удаляет архивное поле и его значения. |
| `POST /helpdesk-pro/fields/{id}/restore` | projects.manage | | Поле, больше не архивное |
| `GET /helpdesk-pro/tickets/{id}/fields` | desk, client | | `[{key, label, type, visibility, value, display}]` |
| `PATCH /helpdesk-pro/tickets/{id}/fields` | desk (редактирование) | `{fields: {key: value}}`; `null` или `''` очищает | Тот же список после. Одно неверное значение не сохраняет ничего. |

### Маршруты на других страницах {: #routes-on-other-pages }

| Область | Где |
|---|---|
| Статусы и метки | [Статусы и метки](../statuses-and-labels/index.md#api-and-mcp) |
| Сортировка и объединение | [Сортировка и объединение](../triage-and-merging/index.md#api-and-mcp) |
| Сохранённые ответы, представления и массовое редактирование | [Сохранённые ответы и представления](../saved-replies-and-views/index.md#api-and-mcp) |
| База знаний | [База знаний](../knowledge-base/index.md#api) |
| Поиск | [Поиск](../search/index.md#api-and-mcp) |
| Вложения | [Вложения](../attachments/index.md#the-upload-flow) |
| Уведомления и предпочтения | [Уведомления](../notifications/index.md#api-and-mcp) |
| Каналы уведомлений | [Каналы уведомлений](../notification-channels/index.md#api-and-mcp) |
| Входящая почта | [Входящая почта](../inbound-email/index.md#routes-and-mcp-tools) |
| Правила | [Правила и автоматизация](../rules-and-automations/index.md#api-and-mcp) |
| SLA | [SLA](../sla/index.md#api-and-mcp) |
| Оценки | [Оценки клиентов](../customer-ratings/index.md#api-and-mcp) |
| Организации | [Организации](../organizations/index.md#api-and-mcp) |
| Отчёты | [Отчёты](../reports/index.md#api-and-mcp) |
| Люди: объединение, удаление, юридические удержания | [Люди и конфиденциальность](../people-and-privacy/index.md#api-and-mcp) |
| Обновления в реальном времени и присутствие | [Обновления в реальном времени](../live-updates/index.md#routes) |

## Быстрая проверка через curl {: #a-quick-check-with-curl }

```bash
KEY=...   # API-ключ учётной записи сотрудника
BASE=https://example.test/api/v1/helpdesk-pro

curl -k "$BASE/bootstrap" -H "X-API-Key: $KEY"

curl -k -X POST "$BASE/tickets" -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"project_id": 1, "subject": "Printer on fire", "body": "It is hot.", "requester": {"email": "ada@example.com"}}'

curl -k -X PATCH "$BASE/tickets/1" -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"status_id": 2, "priority": 3}'
```

## Связанные разделы {: #related }

- [MCP](../mcp/index.md)
- [Расширение](../extending/index.md)
- [Проекты и права](../projects-and-permissions/index.md)
