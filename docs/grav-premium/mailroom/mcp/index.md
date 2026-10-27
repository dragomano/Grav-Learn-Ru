---
title: "MCP"
description: "Управляйте рассылкой из ИИ-клиента через grav-mcp, со всеми инструментами Mailroom, нужным им правом и маршрутами, у которых его нет."
tags:
  - "Mailroom"
  - "Почта"
  - "AI"
  - "MCP"
---

# MCP {: #mcp }

Mailroom описывает свой API админки как инструменты MCP, поэтому ИИ-клиент (Claude Desktop, Claude Code, Cursor и другие) может читать вашу аудиторию, писать кампании и разбираться с доставляемостью через [grav-mcp](https://github.com/getgrav/grav-mcp). Описания лежат в `mcp.yaml` плагина. Плагин API отдаёт их из `GET /mcp/tools`, а grav-mcp превращает каждый в инструмент с именем `mailroom_<name>`, например `mailroom_list_subscribers`.

## Как это работает {: #how-it-works }

Каждый инструмент — один маршрут [REST API](../rest-api/index.md) с описанием, схемой ввода и правом. grav-mcp вызывает маршрут с вашими учётными данными, поэтому инструмент никогда не сделает больше, чем могли бы вы с тем же ключом API, а инструмент, права на который у вас нет, вам вообще не предлагается.

Инструменты чтения помечены как только для чтения, а удаления и массовые изменения — как разрушительные, чтобы клиент мог спросить перед их запуском.

## Подключение ИИ-клиента {: #connect-an-ai-client }

1. Создайте ключ API для аккаунта с правами Mailroom (см. [REST API](../rest-api/index.md#authenticate)). Не давайте ключу `mailroom.send`, если не хотите, чтобы клиент мог класть почту в ящики.
2. Настройте grav-mcp с адресом вашего сайта и этим ключом.
3. Начните с `mailroom_get_overview` и `mailroom_list_lists`: они возвращают счётчики и id списков, которые принимают все остальные инструменты.

> [!TIP] Совет
> Отправка — отдельное право, потому что кампанию нельзя вернуть. Ключ с `mailroom.view` и `mailroom.manage` позволяет ИИ-клиенту писать черновики кампаний, строить сегменты и отвечать на вопросы, а кнопку отправки оставляет вам.

Несколько привычек, которые держат ответы честными:

- Запускайте `mailroom_count_campaign_audience` или `mailroom_preview_segment` до сохранения, чтобы увидеть, кого достигнет кампания или сегмент.
- Счётчики сегментов кэшируются; читайте `counted_at` рядом со счётчиком и используйте `mailroom_count_segment`, когда число должно быть верным сейчас.
- `mailroom_add_subscriber` и `mailroom_bulk_subscribers` спрашивают, как добавлять людей: `mode: invite` или `mode: subscribed` с `note` о том, где они согласились.

## Инструменты {: #tools }

| Область | Инструменты | Право |
|---|---|---|
| Overview | `get_overview`, `get_settings` | view |
| Campaigns | `list_campaigns`, `get_campaign`, `count_campaign_audience`, `list_campaign_recipients`, `list_campaign_links`, `get_campaign_timeline`, `preview_campaign_catch_up` | view |
| Изменение кампаний | `create_campaign`, `update_campaign`, `delete_campaign`, `preview_campaign`, `pause_campaign`, `cancel_campaign` | manage |
| Отправка | `send_campaign_test`, `start_campaign`, `resume_campaign`, `catch_up_campaign`, `retry_failed_campaign_sends` | send |
| Subscribers | `list_subscribers`, `get_subscriber` | view |
| Изменение подписчиков | `add_subscriber`, `update_subscriber`, `unsubscribe_subscriber`, `resubscribe_subscriber`, `confirm_subscriber`, `delete_subscriber`, `bulk_subscribers`, `export_subscriber_data` | manage |
| Imports | `list_imports`, `get_import` | view |
| Списки и теги | `list_lists`, `list_tags` (view); `create_list`, `update_list`, `create_tag`, `delete_tag` | manage |
| Templates | `list_templates`, `get_template` (view); `create_template`, `update_template`, `delete_template`, `preview_template` | manage |
| Suppressions | `list_suppressions` (view); `add_suppression`, `remove_suppression` | manage |
| Segments | `list_segments`, `get_segment`, `get_segment_vocabulary`, `preview_segment` (view); `create_segment`, `update_segment`, `delete_segment`, `count_segment` | manage |
| Deliverability | `check_deliverability` (view); `recheck_deliverability` | manage |
| Providers | `generate_provider_secret`, `set_up_provider_webhook` | manage |
| Automations | `list_automations`, `get_automation`, `list_automation_recipes` (view); `create_automation`, `update_automation`, `delete_automation`, `activate_automation`, `pause_automation`, `resume_automation`, `enrol_in_automation`, `exit_automation_enrolment`, `use_automation_recipe` | manage |
| Reports | `get_report` | view |
| Удаление данных | `erase_address` | manage |

Каждое право — `mailroom.view`, `mailroom.manage` или `mailroom.send`; инструментам отправки также нужен `mailroom.manage`. `export_subscriber_data` — единственное чтение, которому нужен `mailroom.manage`, как и экспорту CSV.

## Маршруты без инструмента {: #routes-without-a-tool }

Несколько маршрутов нужны админке или возвращают файл, а не то, что может прочитать модель, и инструмента у них нет:

- `GET /mailroom/labels`, собственные строки админки, и `GET /mailroom/section-script`, который загружает страницу админки;
- `POST /mailroom/imports/preview` и `POST /mailroom/imports`, которые принимают загрузку файла;
- `GET /mailroom/subscribers/export` и `GET /mailroom/reports/{type}.csv`, которые отвечают файлами CSV;
- формы `GET` предпросмотров кампании и шаблона (инструменты используют форму `POST`).

Чтобы импортировать через ИИ-клиент, попросите его подготовить файл и запустить `bin/plugin mailroom import:csv` на сервере или используйте админку.

## Связанные разделы {: #related }

- [REST API](../rest-api/index.md)
- [Установка](../installation/index.md#give-your-team-access)
