---
title: "MCP"
description: "Работайте с заявками из ИИ-клиента через grav-mcp: все инструменты Helpdesk Pro, их права и разница между ответами и заметками."
tags:
  - "Helpdesk Pro"
  - "ИИ"
  - "MCP"
---

# MCP {: #mcp }

Helpdesk Pro описывает свой API как инструменты MCP, поэтому ИИ-клиент (Claude Desktop, Claude Code, Cursor и другие) может читать заявки и работать с ними через [grav-mcp](https://github.com/getgrav/grav-mcp). Описания находятся в `mcp.yaml` плагина. Плагин API отдаёт их по `GET /mcp/tools`, а grav-mcp превращает каждый в инструмент с именем `helpdesk_<name>`, например `helpdesk_list_tickets`.

## Как это работает {: #how-it-works }

Каждый инструмент — это один маршрут [REST API](../rest-api/index.md) с описанием, схемой входных данных и правом. grav-mcp вызывает маршрут с вашими учётными данными, поэтому инструмент никогда не может сделать больше, чем вы могли бы с тем же ключом API: маршрут проверяет право, и те же правила доступа определяют, какие заявки и сообщения вернутся. Инструмент, права на который у вас нет, вам вообще не предлагается.

Инструменты чтения помечены как только для чтения, а удаление — как разрушительное, чтобы клиент мог спросить перед запуском.

## Подключение ИИ-клиента {: #connect-an-ai-client }

1. Создайте ключ API для учётной записи сотрудника (см. [REST API](../rest-api/index.md#authenticate)). Ключу с областями доступа нужны `helpdesk-pro.desk` и всё остальное, что требуется нужным инструментам.
2. Настройте grav-mcp с адресом вашего сайта и этим ключом.
3. Начните с `get_bootstrap`: он возвращает проекты, статусы, метки и сотрудников с id, которые принимают остальные инструменты.

## Ответы и заметки {: #replies-and-notes }

`add_message` пишет сообщение в заявку. `visibility: public` — это ответ: заявитель и клиенты в копии читают его и получают по почте. `visibility: internal` — заметка сотрудников, которую клиент никогда не увидит.

> [!TIP] Совет
> Когда ИИ готовит текст от вашего имени, сначала попросите заметку, если хотите проверить текст до того, как клиент что-либо увидит.

Передайте `based_on_message_id` (самое новое публичное сообщение, которое прочитала модель), и инструмент откажется с `thread_changed`, если клиент написал за это время, вернув новые сообщения. Ответ никогда не пишется в устаревшую переписку.

Перед объединением ИИ-клиент должен вызвать `preview_merge`, показать причины, а затем вызвать `merge_ticket` с `confirm`.

## Инструменты {: #tools }

| Область | Инструменты | Право |
|---|---|---|
| Рабочая область | `get_config`, `get_bootstrap`, `get_counts` | desk |
| Статус | `get_status` | settings |
| Заявки | `list_tickets`, `create_ticket`, `get_ticket`, `update_ticket`, `get_ticket_timeline`, `mute_ticket`, `get_board`, `bulk_update_tickets` | desk |
| Удаление | `delete_ticket`, `restore_ticket`, `delete_message` | tickets.delete |
| Сообщения | `add_message`, `edit_message`, `get_message_original` | desk |
| Участники | `list_participants`, `add_participant`, `remove_participant` | desk |
| Сортировка и объединение | `triage_ticket`, `preview_merge`, `merge_ticket` | triage |
| Люди | `list_people`, `get_person` | desk |
| Изменения людей | `create_person`, `update_person`, `get_person_privacy`, `send_sign_in_link`, `merge_person`, `erase_person`, `link_person_account`, `unlink_person_account`, `place_legal_hold`, `release_legal_hold` | people.manage |
| Проекты | `list_projects`, `get_project` | desk |
| Изменения проектов | `create_project`, `reorder_projects`, `update_project`, `delete_project`, `set_project_members` | projects.manage |
| Статусы | `list_statuses` (desk); `create_status`, `reorder_statuses`, `update_status`, `delete_status` | projects.manage |
| Метки | `list_labels` (desk); `create_label`, `update_label`, `delete_label` | projects.manage |
| Пользовательские поля | `list_fields`, `get_field`, `get_ticket_fields`, `set_ticket_fields` (desk); `create_field`, `update_field`, `delete_field`, `restore_field`, `reorder_fields` | projects.manage |
| Сохранённые ответы | `list_saved_replies`, `get_saved_reply`, `create_saved_reply`, `update_saved_reply`, `delete_saved_reply`, `render_saved_reply`, `apply_saved_reply` | desk (для общих нужно library.manage) |
| Сохранённые представления | `list_saved_views`, `create_saved_view`, `update_saved_view`, `delete_saved_view` | desk (для общих нужно library.manage) |
| Уведомления | `list_notifications`, `mark_notifications_read`, `mark_all_notifications_read`, `get_preferences`, `update_preferences` | desk |
| Поиск и база знаний | `search`, `suggest_articles`, `find_similar_tickets`, `get_ticket_context` (desk); `rebuild_search` | settings |
| Почта | `send_test_email` | settings |
| Входящая почта | `list_inbound_log`, `get_inbound_entry`, `get_inbound_original`, `retry_inbound`, `release_inbound`, `get_inbound_status`, `test_inbound` | settings |
| Правила | `list_rules`, `get_rule_catalog`, `get_rule`, `create_rule`, `update_rule`, `delete_rule`, `reorder_rules`, `test_rule` | settings |
| SLA | `list_sla_policies`, `get_sla_policy`, `list_business_hours`, `get_business_hours`, `get_ticket_sla` (desk); `create_sla_policy`, `update_sla_policy`, `delete_sla_policy`, `reorder_sla_policies`, `create_business_hours`, `update_business_hours`, `delete_business_hours` | settings |
| Оценки | `list_ratings`, `get_ticket_rating` (desk); `get_ratings_summary` | reports |
| Отчёты | `get_report` | reports |
| Организации | `list_organizations`, `get_organization`, `list_organization_members`, `get_person_organization`, `set_ticket_organization` (desk); `create_organization`, `update_organization`, `delete_organization`, `add_organization_member`, `add_organization_members_by_domain`, `remove_organization_member`, `set_person_organization` | people.manage |
| Каналы уведомлений | `list_channels`, `get_channel`, `create_channel`, `update_channel`, `delete_channel`, `test_channel` | settings |

Каждое право — действие `helpdesk-pro.*` из раздела [Проекты и права](../projects-and-permissions/index.md#permissions).

- **Правила** действуют с полномочиями системы на каждой заявке, поэтому инструментам правил нужно `helpdesk-pro.settings`. Давайте их ИИ-клиенту только с ключом API, которому вы доверили бы настройку рабочей области. `list_rules` возвращает каждое правило одним предложением (`summary`) со всем, на что оно ссылается и чего больше нет (`problems`), чтобы клиент мог прочитать правило перед изменением.
- **Инструменты каналов** никогда не возвращают URL webhook или секрет подписи (оба приходят замаскированными, и отправка маски обратно сохраняет их). `test_channel` отправляет настоящее тестовое сообщение и возвращает ответ самого эндпоинта.
- **Значения пользовательских полей** уже есть в `get_ticket` и `list_tickets` под `extra.fields`. Чтобы задать их в новой заявке, вызовите `set_ticket_fields` после `create_ticket`.

## Маршруты без инструмента {: #routes-without-a-tool }

Несколько маршрутов предназначены только для Admin Next и браузера и не имеют инструмента: значок в боковой панели (`GET /helpdesk-pro/badge`), конфигурация обновлений в реальном времени и сигнал присутствия, а также маршруты загрузки, скачивания и удаления черновиков вложений. ИИ-клиенты видят файлы как список `attachments` в каждом сообщении хронологии.

## Связанные разделы {: #related }

- [REST API](../rest-api/index.md)
- [Проекты и права](../projects-and-permissions/index.md)
