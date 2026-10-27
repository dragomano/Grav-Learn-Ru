---
title: "Начало работы"
description: "Плагин Grav API добавляет к вашему сайту RESTful API, обеспечивая полноценный headless-доступ к страницам, медиафайлам, конфигурации, пользователям и управлению системой."
tags:
  - "REST API"
---

# Начало работы с Grav API

Плагин [Grav API](https://github.com/getgrav/grav-plugin-api) добавляет к вашему сайту RESTful API, обеспечивая полноценный headless-доступ к страницам, медиафайлам, конфигурации, пользователям и управлению системой.

## Требования

- Grav CMS 2.1.5+
- PHP 8.3+
- Плагин Login 3.9.2+

## Установка

Установите через GPM:

```bash
bin/grav install api
```

Или вручную: скачайте плагин в `user/plugins/api` и выполните `composer install`.

## Быстрая настройка

### 1. Включите плагин

```yaml
# user/config/plugins/api.yaml
enabled: true
```

### 2. Сгенерируйте ключ API

**Через CLI** (рекомендуется для первоначальной настройки):

```bash
bin/plugin api keys:generate --user=admin --name="My First Key"
```

**Через админ-панель**: откройте профиль пользователя и воспользуйтесь разделом **API Keys**.

Ключ показывается только один раз — сохраните его сразу.

### 3. Выполните первый запрос

```bash
curl https://my-site.example/api/v1/pages \
  -H "X-API-Key: grav_abc123..."
```

## Конфигурация

API настраивается через `user/config/plugins/api.yaml`:

```yaml
enabled: true
route: /api                    # Базовый маршрут для всех эндпоинтов API
version_prefix: v1             # Префикс версии

auth:
  api_keys_enabled: true       # Включить аутентификацию по ключу API
  jwt_enabled: true            # Включить аутентификацию по JWT-токену
  session_enabled: true        # Включить проброс сессии

cors:
  enabled: true                # Включить заголовки CORS
  origins: []                  # Дополнительные источники, которым разрешены кросс-доменные запросы (по умолчанию только тот же источник)
  credentials: false           # Разрешить передачу учётных данных

rate_limit:
  enabled: true                # Включить ограничение частоты запросов
  requests: 120                # Число запросов за окно на пользователя (или на IP для анонимных)
  window: 60                   # Длина окна в секундах
  # excluded_paths: ['/sync/'] # Префиксы маршрутов, для которых ограничение не действует (по умолчанию только опрос совместной работы)

pagination:
  default_per_page: 20         # Число элементов на странице по умолчанию
  max_per_page: 1000           # Максимальное число элементов на странице
```

## Окружения

Grav поддерживает несколько окружений. API учитывает это через заголовок `X-Grav-Environment`:

```bash
curl -H "X-Grav-Environment: my-site.example" \
     -H "X-API-Key: ..." \
     https://my-site.example/api/v1/pages
```

## Формат ответа

Все ответы API имеют единообразную структуру:

```json
{
  "data": { ... }
}
```

Ответы с постраничной разбивкой содержат метаданные:

```json
{
  "data": [ ... ],
  "meta": {
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 42,
      "total_pages": 3
    }
  },
  "links": {
    "self": "/api/v1/pages?page=1&per_page=20",
    "next": "/api/v1/pages?page=2&per_page=20",
    "last": "/api/v1/pages?page=3&per_page=20"
  }
}
```

Ссылки в `links` сохраняют все остальные параметры запроса, которые вы отправили (фильтры, поиск, сортировку), поэтому переход по `next` в отфильтрованном списке не выходит за рамки этого фильтра.

Ошибки оформляются в формате RFC 7807 с типом содержимого `application/problem+json`. Ошибка валидации `422` также может содержать список `errors` с записями `{field, message}`:

```json
{
  "status": 404,
  "title": "Not Found",
  "detail": "Page '/missing' not found."
}
```

## Управление конкурентным доступом {: #concurrency-control }

API использует ETag для оптимистичного контроля конкурентного доступа. При обновлении ресурсов передавайте заголовок `If-Match` с ETag из ответа на GET-запрос:

```bash
# Получение с ETag
curl -H "X-API-Key: ..." https://my-site.example/api/v1/pages/blog
# Ответ содержит: ETag: "abc123"

# Обновление с If-Match
curl -X PATCH \
  -H "X-API-Key: ..." \
  -H "If-Match: \"abc123\"" \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}' \
  https://my-site.example/api/v1/pages/blog
```

Если ресурс был изменён после вашего последнего запроса, вы получите ответ `409 Conflict`.

## Использование API с ИИ

API — это основа интеграции ИИ в Grav 2.0. Официальный [Grav MCP Server](../../advanced/mcp-server/index.md) подключает агентные ИИ-клиенты (Claude Code, Claude Desktop, Cursor и другие клиенты [Model Context Protocol](https://modelcontextprotocol.io)) к вашему сайту, предоставляя страницы, медиафайлы, конфигурацию, пользователей и управление системой в виде инструментов, вызываемых ИИ. Он аутентифицируется теми же ключами API, что описаны здесь, и соблюдает ту же модель прав доступа. Чтобы начать, см. документацию [MCP Server](../../advanced/mcp-server/index.md).
