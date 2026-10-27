---
title: "Боковая панель"
description: "Собирает элементы навигации боковой панели, зарегистрированные плагинами. Вызывает событие onApiSidebarItems, отбрасывает элементы, на которые у пользователя нет разрешения `authorize`, удаляет `authorize`, переводит метки, являющиеся ключами перевода, на язык админки пользователя и сортирует по `priority`. Каждый элемент может иметь `badge`, `badgeEndpoint` или `children`…"
tags:
  - "REST API"
---

# Боковая панель

## Получение элементов боковой панели {: #sidebar-items }

<span class="api-method api-method-get">GET</span> `/sidebar/items`

Собирает элементы навигации боковой панели, зарегистрированные плагинами. Вызывает событие onApiSidebarItems, отбрасывает элементы, на которые у пользователя нет разрешения `authorize`, удаляет `authorize`, переводит метки, являющиеся ключами перевода, на язык админки пользователя и сортирует по `priority` (сначала большее значение). Каждый элемент задаёт id, метку, иконку, маршрут и необязательные `badge`, `badgeEndpoint` или `children` (ярлыки, которые каждый пользователь может показывать под элементом; дочерние элементы, на которые у пользователя нет разрешения `authorize`, а также с неверным или повторяющимся id, отбрасываются, а `default` сохраняется только при значении `true`). Требуется `api.access`.

**Ответ**

```json
{"data": [{"id": "license-manager", "plugin": "license-manager", "label": "Licenses", "icon": "fa-key", "route": "/plugin/license-manager", "priority": 10}, {"id": "my-plugin", "plugin": "my-plugin", "label": "My Plugin", "icon": "fa-store", "route": "/plugin/my-plugin", "priority": 5, "children": [{"id": "orders", "label": "Orders", "route": "/plugin/my-plugin/orders", "default": true}, {"id": "coupons", "label": "Coupons", "route": "/plugin/my-plugin/coupons"}]}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.access` |
