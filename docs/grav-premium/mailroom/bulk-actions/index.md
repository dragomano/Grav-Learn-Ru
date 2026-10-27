---
title: "Массовые действия"
description: "Выберите подписчиков вручную, по странице или по фильтру, затем добавьте или уберите список либо тег, отпишите или удалите их и прочитайте, что было сделано и кого не тронули."
tags:
  - "Mailroom"
  - "Почта"
---

# Массовые действия {: #bulk-actions }

Таблица Subscribers и страница каждого списка умеют действовать сразу на многих людей. Массовым действиям нужен `mailroom.manage`.

## Выбор людей {: #select-people }

- Отметьте флажок в строке, чтобы выбрать этого человека.
- Отметьте флажок в заголовке, чтобы выбрать всех на странице («Select everybody on this page»).
- Затем выберите **Select all N matching these filters**, чтобы действовать на всех, кто совпадает с текущими фильтрами, включая людей на других страницах.

Панель показывает «N selected» и ссылку **Clear selection**.

## Шесть действий {: #the-six-actions }

| Действие | Что делает |
|---|---|
| **Add to list** | Добавляет их в список, спрашивая как: приглашение подтвердить или как уже согласившихся с заметкой (см. [Подписчиков](../subscribers/index.md#add-people)). |
| **Remove from list** | Убирает их из одного списка с записью, что это сделали вы. Остальные их списки не трогаются, а все, кого нет в этом списке, остаются как есть. |
| **Add tag** | Ставит на них тег: существующий или новый, названный в диалоге. |
| **Remove tag** | Снимает с них тег. |
| **Unsubscribe** | Убирает их из всех списков с записью, что это сделал сайт. Сначала спрашивает: «Take N people off every list?» |
| **Delete** | Удаляет их так же, как одиночное **Delete**, не трогая список исключений. Сначала спрашивает со счётчиком: «Delete N subscribers?» |

Большой выбор обрабатывается по 500 человек за раз, с надписью «Working: X of Y done.», пока идёт работа. Люди, которые начинают совпадать с фильтрами посередине, не включаются, поэтому никто, подписавшийся во время прохода, не будет захвачен.

## Что сообщается после {: #what-it-says-afterwards }

Когда всё заканчивается, панель говорит, что было сделано («12 added», «3 invited to confirm», «40 tagged»), и по причинам — кого не тронули:

| Причина | Почему |
|---|---|
| suppressed, bounced or complained | На адрес никогда не пишут, поэтому его никогда не добавляют. |
| left every list themselves / left this list themselves | Никого не возвращают в список, из которого он ушёл по собственной воле. |
| already on the list / already waiting to confirm | Добавлять нечего. |
| not on the list / without the tag / already tagged / already off every list | Менять нечего. |
| not an email address | Вставленная строка не является адресом. |
| no longer here | Удалён кем-то другим тем временем. |

«Nothing changed.» означает, что каждого выбранного человека не тронули.

## Из API {: #from-the-api }

Те же действия — `POST /mailroom/subscribers/bulk` и инструмент MCP `mailroom_bulk_subscribers`. Назовите кого ровно одним из:

- `ids`: до 500 id подписчиков;
- `filter`: фильтры таблицы Subscribers (`q`, `status`, `source`, `list`, `list_status`, `tag`, `campaign_id`, `campaign_received`, `test`), проходимые по 500 за раз. Передавайте `next` из ответа обратно как `after`, а `max_id` — как `max_id`, пока ответ не скажет, что закончено;
- `addresses`: вставленные адреса по одному в строке или через запятую, каждый при желании с именем, только для `add_to_list` (не более 1000). Новые адреса становятся подписчиками.

```bash
curl -k -X POST https://example.test/api/v1/mailroom/subscribers/bulk \
  -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"action": "add_tag", "tag": "webinar-2026", "filter": {"list": 3, "status": "subscribed"}}'
```

| Поле | Для чего |
|---|---|
| `action` | `add_to_list`, `remove_from_list`, `add_tag`, `remove_tag`, `unsubscribe` или `delete` |
| `list_id` | `add_to_list` и `remove_from_list` |
| `mode`, `note` | `add_to_list`: `invite` или `subscribed`, а с `subscribed` — где они согласились (обязательно) |
| `tag` | `add_tag`: тег, создаётся, если новый |
| `tag_id` | `remove_tag` |
| `all` | `unsubscribe` и `delete` по фильтру без условий: `true` означает всех |

Ответ считает сделанное и по причинам — кого пропустили. См. [REST API](../rest-api/index.md#subscribers).

## Связанные разделы {: #related }

- [Подписчики](../subscribers/index.md)
- [Теги и сегменты](../tags-and-segments/index.md)
- [REST API](../rest-api/index.md)
