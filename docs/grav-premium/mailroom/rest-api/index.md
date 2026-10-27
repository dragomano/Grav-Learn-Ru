---
title: "REST API"
description: "Маршруты админки Mailroom под /api/v1/mailroom, как аутентифицироваться, какое право нужно каждому и маршруты по экранам."
tags:
  - "Mailroom"
  - "Почта"
  - "REST API"
---

# REST API {: #rest-api }

Mailroom добавляет свои маршруты админки в плагин API Grav под `/api/v1/mailroom/`. Каждый экран админки построен на этих маршрутах, поэтому всё, что делает она, может сделать и скрипт, и ИИ-клиент (см. [MCP](../mcp/index.md)). Пути ниже опускают префикс `/api/v1`.

Это маршруты админки. Публичный эндпоинт подписки — другое дело, без ключа: см. [API подписки](../subscribe-api/index.md).

## Аутентификация {: #authenticate }

Используйте любые учётные данные, которые принимает плагин API:

- **Ключ API**, отправленный как `X-API-Key`. Создайте его в Admin Next (свой аккаунт, затем **API Keys**) или на сервере командой `bin/plugin api keys:generate --user=<username>`. Ключ, созданный с областями, ограничен ими: дайте ему `mailroom.view`, а также `mailroom.manage` и `mailroom.send`, если он должен писать и отправлять.
- **JWT** из `POST /auth/token`, отправленный как `X-API-Token` (так делает Admin Next).

```bash
curl -k "https://example.test/api/v1/mailroom/overview" -H "X-API-Key: $KEY"
```

## Кому что можно {: #who-may-call-what }

Каждому маршруту нужен `api.access` и одно из прав Mailroom. Чтение требует `mailroom.view`, запись — `mailroom.manage`, а всё, что кладёт почту в ящики, требует `mailroom.send` вместе с `mailroom.manage`. `api.super` считается всеми тремя. Таблицы ниже говорят **view**, **manage** или **send**.

## Соглашения {: #conventions }

- Ответы заворачивают полезную нагрузку в `data`: `{"data": {...}}`.
- Ошибки — сведения о проблеме (`application/problem+json`) с `status`, `title` и `detail`. Отказ, с которым можно что-то сделать (код списка, который нельзя изменить, исключённый адрес, недопустимый тег слияния), говорит об этом в `detail`.
- Время — секунды эпохи UTC в обе стороны.
- Адрес электронной почты никогда не возвращается вызывающему, у которого его не было.
- Списки принимают `page` и `per_page` (не более 200 там, где маршрут листает).

## Маршруты {: #routes }

### Обзор и настройки {: #overview-and-settings }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/overview` | view | Вкладка Overview: счётчики по состояниям, рост, последние кампании, предупреждения |
| `GET /mailroom/settings` | view | Что читают карточки Settings: транспорт, обработчик, адрес From, провайдеры |
| `GET /mailroom/labels` | view | Собственные строки админки. Инструмента MCP нет. |

### Кампании {: #campaigns }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/campaigns` | view | Каждая кампания, начиная с новых. `status`, `page`, `per_page`. Счётчики состояний — по всей таблице. |
| `POST /mailroom/campaigns` | manage | Создать черновик. См. ниже. |
| `GET /mailroom/campaigns/audience` | view | Посчитать, кого достигнет кампания до её создания: `lists=1,3`, `segment=5`. |
| `GET /mailroom/campaigns/{id}` | view | Одна кампания со её счётчиками, процентами, тестом темы и возможными следующими шагами |
| `PATCH /mailroom/campaigns/{id}` | manage | Редактировать черновик или запланированную кампанию |
| `DELETE /mailroom/campaigns/{id}` | manage | Удалить черновик или отменённую кампанию |
| `GET /mailroom/campaigns/{id}/sends` | view | Её получатели и то, что произошло с каждым. `status`, `page`, `per_page`. |
| `GET /mailroom/campaigns/{id}/links` | view | Её ссылки и переходы по ним |
| `GET /mailroom/campaigns/{id}/timeline` | view | Её первые 48 часов по часам |
| `GET /mailroom/campaigns/{id}/preview` | view | Нарисовать сохранённую кампанию |
| `POST /mailroom/campaigns/{id}/preview` | manage | Нарисовать несохранённые поля (`subject`, `preheader`, `body_md`, `layout`) |
| `POST /mailroom/campaigns/{id}/test` | send | `{email}`: отправить одну тестовую копию |
| `POST /mailroom/campaigns/{id}/start` | send | Отправить сейчас или `{scheduled_at}` (секунды эпохи UTC), чтобы запланировать |
| `POST /mailroom/campaigns/{id}/pause` | manage | Приостановить |
| `POST /mailroom/campaigns/{id}/resume` | send | Продолжить приостановленную или остановленную кампанию |
| `POST /mailroom/campaigns/{id}/cancel` | manage | Отменить навсегда |
| `GET /mailroom/campaigns/{id}/catch-up` | view | Посчитать людей, присоединившихся к её спискам после её отправки |
| `POST /mailroom/campaigns/{id}/catch-up` | send | Отправить её им |
| `POST /mailroom/campaigns/{id}/retry-failed` | send | Повторить каждую неудачную отправку или `{send_id}` для одной |

Поля кампании для создания и редактирования: `name` (обязательно при создании), `subject`, `preheader`, `body_md`, `layout` (`store` для брендированного макета или `plain`), `list_id` или `list_ids` (все списки по порядку), `segment_id`, `template_id` (только при создании: один раз копирует поля шаблона), `from_name`, `from_email`, `reply_to`, `track_opens`, `track_clicks`, `subject_b`, `ab_sample_percent` (от 10 до 50), `ab_window_minutes` и `utm` (`{source, medium, campaign, content}`). Оставьте `list_ids` пустым и дайте `segment_id`, чтобы отправить всем в любом списке, кто совпадает с ним.

```bash
curl -k -X POST https://example.test/api/v1/mailroom/campaigns \
  -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"name": "October news", "subject": "What changed in October", "body_md": "Hello {{ subscriber.first_name }},", "list_ids": [1, 3]}'
```

### Подписчики {: #subscribers }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/subscribers` | view | Таблица. `q`, `status`, `source`, `list`, `list_status` (вместе с `list`), `tag`, `campaign_id` с `campaign_received` (`yes` или `no`), `page`, `per_page`. Счётчики состояний по всей таблице и `list_counts` со списком. |
| `POST /mailroom/subscribers` | manage | Добавить одного человека: `{email, name, list_id, tags, language, mode, note}`. `mode` — `invite` или `subscribed` (тогда `note` обязательна); без него решает список. `outcome` ответа говорит `added`, `invited`, `already_on` или `already_invited`. |
| `POST /mailroom/subscribers/bulk` | manage | Одно действие над многими людьми. См. [Массовые действия](../bulk-actions/index.md#from-the-api). |
| `GET /mailroom/subscribers/{id}` | view | Один человек: согласие, списки, теги, импорт, с которым он пришёл, исключён ли адрес, что ему отправляли и `consent_log` |
| `PATCH /mailroom/subscribers/{id}` | manage | `{name, language, lists, tags}`. Адрес изменить нельзя. Меняются только списки, в которых он уже состоит, а список, из которого он ушёл сам, остаётся покинутым. |
| `DELETE /mailroom/subscribers/{id}` | manage | Удалить его. Исключение остаётся. |
| `POST /mailroom/subscribers/{id}/unsubscribe` | manage | Убрать его из всех списков с записью, что это сделал сайт |
| `POST /mailroom/subscribers/{id}/resubscribe` | manage | `{note, list_id}`: вернуть его с тем, как он просил. Без `list_id` — во все списки, где он был. |
| `POST /mailroom/subscribers/{id}/confirm` | manage | Подтвердить ждущего вручную |
| `GET /mailroom/subscribers/{id}/export` | manage | Всё, что о нём хранится, одним документом JSON. См. [Конфиденциальность](../privacy/index.md#data-export-for-one-person). |
| `GET /mailroom/subscribers/export` | manage | Таблица как CSV с теми же фильтрами. Инструмента MCP нет. |

### Импорт {: #imports }

| Маршрут | Право | Что делает |
|---|---|---|
| `POST /mailroom/imports/preview` | manage | Multipart: `file` и необязательно `mapping` (JSON) и `list_id`. Отвечает столбцами, найденным пресетом, первыми строками так, как их прочитают, и `dry_run` со счётчиками. Инструмента MCP нет. |
| `POST /mailroom/imports` | manage | Multipart: `file`, `mapping` (JSON со столбцом `email`), `list_id`, `as_confirmed`, `tag`, `basis`. Ставит импорт в очередь. Инструмента MCP нет. |
| `GET /mailroom/imports` | view | Последние запуски со счётчиками |
| `GET /mailroom/imports/{id}` | view | Один запуск с его ходом |

### Списки и теги {: #lists-and-tags }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/lists` | view | Каждый список с числом подписанных |
| `POST /mailroom/lists` | manage | `{code, name, description, double_opt_in, public, is_default, position, from_name, from_email}` |
| `PATCH /mailroom/lists/{id}` | manage | То же, кроме `code`, который нельзя изменить. Списки нельзя удалить. |
| `GET /mailroom/tags` | view | Каждый тег с числом людей, которые его несут |
| `POST /mailroom/tags` | manage | `{code, label}`. Код нормализуется; существующий тег возвращается, а не отклоняется. |
| `DELETE /mailroom/tags/{id}` | manage | Удалить тег и снять его со всех |

### Шаблоны {: #templates }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/templates` | view | Каждый шаблон |
| `POST /mailroom/templates` | manage | `{name, subject, preheader, body_md, layout, is_active}` |
| `GET /mailroom/templates/{id}` | view | Один шаблон |
| `PATCH /mailroom/templates/{id}` | manage | Редактировать его |
| `DELETE /mailroom/templates/{id}` | manage | Удалить его. Кампании, написанные по нему, сохраняют свою копию. |
| `GET /mailroom/templates/{id}/preview` | view | Нарисовать его |
| `POST /mailroom/templates/{id}/preview` | manage | Нарисовать несохранённые поля |

### Исключения {: #suppressions }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/suppressions` | view | Список. `reason` (`hard_bounce`, `complaint`, `manual`, `invalid`), `page`, `per_page`. |
| `POST /mailroom/suppressions` | manage | `{email, reason, note}`. `reason` — `manual` (по умолчанию) или `invalid`. |
| `DELETE /mailroom/suppressions/{hash}` | manage | Удалить одно по SHA-256 адреса в нижнем регистре |

### Сегменты {: #segments }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/segments/vocabulary` | view | Каждое поле, которое может назвать условие, с его типом, операторами и параметрами |
| `POST /mailroom/segments/preview` | view | `{definition, list_id}`: посчитать и показать выборку по документу условий, не сохраняя его |
| `GET /mailroom/segments` | view | Каждый сегмент с его кэшированным счётчиком и временем подсчёта |
| `POST /mailroom/segments` | manage | `{name, description, list_id, definition}` |
| `GET /mailroom/segments/{id}` | view | Один сегмент с его документом |
| `PATCH /mailroom/segments/{id}` | manage | Редактировать его. Изменение условий или списка очищает счётчик. |
| `DELETE /mailroom/segments/{id}` | manage | Удалить его, если на него не указывает черновик или запланированная кампания |
| `POST /mailroom/segments/{id}/count` | manage | Пересчитать его сейчас |

Формат документа — в [Тегах и сегментах](../tags-and-segments/index.md#in-the-api).

### Доставляемость и провайдеры {: #deliverability-and-providers }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/deliverability` | view | Проверки экрана Health, кэшированные на десять минут |
| `POST /mailroom/deliverability/recheck` | manage | Заново посмотреть всё |
| `POST /mailroom/providers/{name}/secret` | manage | Создать новый секрет webhook для провайдера, записать его в настройки и ответить карточке с URL |
| `POST /mailroom/providers/{name}/setup` | manage | Создать webhook через API провайдера, где он это позволяет |

### Автоматизации {: #automations }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/flows/recipes` | view | Готовые автоматизации |
| `POST /mailroom/flows/recipes/{key}` | manage | Написать по ней черновик |
| `GET /mailroom/flows` | view | Каждая автоматизация с тем, где её люди |
| `POST /mailroom/flows` | manage | `{name, trigger, trigger_json, list_id, segment_id, allow_reentry, steps}`. Создаётся как черновик. |
| `GET /mailroom/flows/{id}` | view | Одна автоматизация с её шагами и страницей её людей. `status` (`active`, `waiting`, `done`, `exited`), `page`. |
| `PATCH /mailroom/flows/{id}` | manage | Редактировать её. Шаги сохраняются целиком; шаг, возвращённый со своим `id`, сохраняет своих людей. |
| `DELETE /mailroom/flows/{id}` | manage | Удалить её. С людьми в ней `force=1` сначала выводит их. |
| `POST /mailroom/flows/{id}/activate` | manage | Включить её |
| `POST /mailroom/flows/{id}/pause` | manage | Приостановить её |
| `POST /mailroom/flows/{id}/resume` | manage | Продолжить её |
| `POST /mailroom/flows/{id}/enrol` | manage | `{subscriber_id}`: добавить одного человека |
| `POST /mailroom/flows/{id}/enrolments/{eid}/exit` | manage | Вывести одного человека |

В Mailroom `trigger` — `subscribed`, `confirmed` или `tag_added` (с `trigger_json: {"tag": "code"}`). Формат шага — в [Автоматизациях](../automations/index.md#from-the-api).

### Отчёты {: #reports }

| Маршрут | Право | Что делает |
|---|---|---|
| `GET /mailroom/reports/{type}` | view | Один из семи отчётов как JSON |
| `GET /mailroom/reports/{type}.csv` | view | То же как CSV. Инструмента MCP нет. |

Типы и параметры — в [Отчётах и отслеживании](../reports-and-tracking/index.md#from-the-api).

### Удаление данных {: #erasure }

| Маршрут | Право | Что делает |
|---|---|---|
| `POST /mailroom/erase` | manage | `{email}`: забыть один адрес. Отвечает `{erased, counts}` и никогда не повторяет адрес. См. [Конфиденциальность](../privacy/index.md#erase-an-address). |

## Быстрая проверка через curl {: #a-quick-check-with-curl }

```bash
KEY=...   # API-ключ учётной записи с mailroom.view и mailroom.manage
BASE=https://example.test/api/v1/mailroom

curl -k "$BASE/lists" -H "X-API-Key: $KEY"

curl -k -X POST "$BASE/subscribers" -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"email": "ada@example.com", "name": "Ada", "list_id": 1, "mode": "invite"}'

curl -k "$BASE/subscribers?status=pending&per_page=50" -H "X-API-Key: $KEY"
```

## Связанные разделы {: #related }

- [MCP](../mcp/index.md)
- [API подписки](../subscribe-api/index.md)
- [Установка](../installation/index.md#give-your-team-access)
