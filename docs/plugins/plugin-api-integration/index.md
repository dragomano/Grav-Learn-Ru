---
title: "Интеграция с API"
description: "Плагин Grav API предоставляет RESTful API для Grav CMS. Любой плагин может расширить этот API, зарегистрировав собственные эндпоинты через событие onApiRegisterRoutes. Это руководство шаг за шагом описывает соответствующий шаблон."
tags:
  - "Плагины"
  - "REST API"
  - "PHP"
---

# Интеграция плагина с API

[Плагин Grav API](https://github.com/getgrav/grav-plugin-api) предоставляет RESTful API для Grav CMS. Любой плагин может расширить этот API, зарегистрировав собственные эндпоинты через событие `onApiRegisterRoutes`. Это руководство шаг за шагом описывает соответствующий шаблон.

## Обзор

Когда плагин API установлен, он вызывает событие `onApiRegisterRoutes` во время инициализации маршрутизатора. Ваш плагин слушает это событие и регистрирует собственные маршруты, указывая их на класс контроллера, расширяющий `AbstractApiController`. Это бесплатно даёт вам аутентификацию, разрешения, разбор запросов, пагинацию и обработку ошибок.

## Предварительные требования

- Плагин Grav API установлен и включён
- У вашего плагина должен быть `composer.json` с PSR-4-автозагрузкой для папки `classes/`
- У вашего плагина должен быть метод `autoload()`, загружающий `vendor/autoload.php`

## Пошаговое руководство

### 1. Добавление слушателя событий

В основном PHP-файле плагина подпишитесь на событие `onApiRegisterRoutes`:

```php
public static function getSubscribedEvents()
{
    return [
        // ... ваши существующие события ...
        'onApiRegisterRoutes' => ['onApiRegisterRoutes', 0],
    ];
}
```

### 2. Регистрация маршрутов

В обработчике события используйте `ApiRouteCollector` для регистрации эндпоинтов:

```php
public function onApiRegisterRoutes(Event $event): void
{
    $routes = $event['routes'];
    $controller = \Grav\Plugin\MyPlugin\MyApiController::class;

    $routes->get('/my-resource', [$controller, 'index']);
    $routes->get('/my-resource/{id}', [$controller, 'show']);
    $routes->post('/my-resource', [$controller, 'create']);
    $routes->patch('/my-resource/{id}', [$controller, 'update']);
    $routes->delete('/my-resource/{id}', [$controller, 'delete']);
}
```

Коллектор маршрутов поддерживает `get()`, `post()`, `patch()`, `put()`, `delete()` и `group()` для групп с префиксом:

```php
$routes->group('/my-plugin', function ($group) {
    $group->get('/items', [$controller, 'listItems']);
    $group->post('/items', [$controller, 'createItem']);
    $group->get('/stats', [$controller, 'stats']);
});
```

### 3. Создание API-контроллера

Создайте в папке `classes/` плагина класс контроллера, расширяющий `AbstractApiController`:

```php
<?php
// classes/MyApiController.php

declare(strict_types=1);

namespace Grav\Plugin\MyPlugin;

use Grav\Plugin\Api\Controllers\AbstractApiController;
use Grav\Plugin\Api\Response\ApiResponse;
use Grav\Plugin\Api\Exceptions\NotFoundException;
use Grav\Plugin\Api\Exceptions\ValidationException;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class MyApiController extends AbstractApiController
{
    public function index(ServerRequestInterface $request): ResponseInterface
    {
        $this->requirePermission($request, 'api.system.read');

        $pagination = $this->getPagination($request);

        // Здесь ваша логика получения данных
        $items = $this->getItems();
        $total = count($items);
        $paged = array_slice($items, $pagination['offset'], $pagination['limit']);

        return ApiResponse::paginated(
            $paged, $total,
            $pagination['page'], $pagination['per_page'],
            $this->getApiBaseUrl() . '/my-resource'
        );
    }

    public function show(ServerRequestInterface $request): ResponseInterface
    {
        $this->requirePermission($request, 'api.system.read');

        $id = $this->getRouteParam($request, 'id');
        $item = $this->findItemOrFail($id);

        return ApiResponse::create($item);
    }

    public function create(ServerRequestInterface $request): ResponseInterface
    {
        $this->requirePermission($request, 'api.system.write');

        $body = $this->getRequestBody($request);
        $this->requireFields($body, ['name']);

        // Здесь ваша логика создания
        $item = $this->createItem($body);

        return ApiResponse::created(
            $item,
            $this->getApiBaseUrl() . '/my-resource/' . $item['id']
        );
    }

    public function delete(ServerRequestInterface $request): ResponseInterface
    {
        $this->requirePermission($request, 'api.system.write');

        $id = $this->getRouteParam($request, 'id');
        $this->deleteItem($id);

        return ApiResponse::noContent();
    }
}
```

### 4. Доступные помощники

`AbstractApiController` из коробки предоставляет следующие методы:

| Метод | Назначение |
|--------|---------|
| `requirePermission($request, $perm)` | Проверить, что у пользователя есть разрешение; выбросить 403, если нет |
| `getUser($request)` | Получить аутентифицированного пользователя |
| `getRequestBody($request)` | Разобрать тело JSON-запроса |
| `getRouteParam($request, $name)` | Получить параметр маршрута FastRoute |
| `getPagination($request)` | Разобрать query-параметры `page`/`per_page` |
| `getSorting($request, $fields)` | Разобрать query-параметры `sort`/`order` |
| `getFilters($request, $fields)` | Разобрать query-параметры фильтра |
| `requireFields($body, $fields)` | Проверить обязательные поля; выбросить 422, если их нет |
| `validateEtag($request, $hash)` | Проверка оптимистической конкурентности |
| `respondWithEtag($data)` | Ответ с заголовком ETag |
| `fireEvent($name, $data)` | Вызвать событие Grav |
| `getApiBaseUrl()` | Получить базовый URL API для генерации ссылок |

### 5. Помощники ответов

Используйте статические методы `ApiResponse` и `ErrorResponse`:

```php
use Grav\Plugin\Api\Response\ApiResponse;

// Стандартный JSON-ответ
ApiResponse::create($data);                    // 200 с {data: ...}
ApiResponse::created($data, $locationUrl);     // 201 с заголовком Location
ApiResponse::noContent();                      // 204, пустое тело
ApiResponse::paginated($data, $total, $page, $perPage, $baseUrl);  // С метаданными пагинации
```

### 6. Обработка исключений

Выбрасывайте типизированные исключения — они автоматически перехватываются и преобразуются в ответы ошибок RFC 7807:

```php
use Grav\Plugin\Api\Exceptions\NotFoundException;
use Grav\Plugin\Api\Exceptions\ValidationException;
use Grav\Plugin\Api\Exceptions\ForbiddenException;
use Grav\Plugin\Api\Exceptions\ConflictException;
use Grav\Plugin\Api\Exceptions\ApiException;

throw new NotFoundException("Item not found.");           // 404
throw new ValidationException("Invalid input.", $errors); // 422
throw new ForbiddenException("Not allowed.");             // 403
throw new ConflictException("Resource modified.");        // 409
throw new ApiException(503, 'Unavailable', 'Detail');     // Пользовательский статус
```

### 7. Обеспечение автозагрузки

У вашего плагина **должен** быть метод `autoload()`, чтобы Grav загружал ваши классы:

```php
public function autoload(): \Composer\Autoload\ClassLoader
{
    return require __DIR__ . '/vendor/autoload.php';
}
```

И `composer.json` с PSR-4-сопоставлением:

```json
{
    "autoload": {
        "psr-4": {
            "Grav\\Plugin\\MyPlugin\\": "classes/"
        }
    }
}
```

Запустите `composer dump-autoload` после добавления новых классов.

## Документация API

### Документация, совместимая с Helios

Создайте в плагине папку `api-docs/` со страницами эндпоинтов в формате Helios:

```
my-plugin/
├── api-docs/
│   ├── chapter.md              # Обзорная страница
│   ├── 01.list-items/
│   │   └── api-endpoint.md     # Документация эндпоинта
│   ├── 02.create-item/
│   │   └── api-endpoint.md
│   └── grav-my-plugin-api.postman_collection.json
```

Каждая страница эндпоинта использует шаблон `api-endpoint`:

```yaml
---
title: 'List Items'
template: api-endpoint
api:
  method: GET
  path: /my-resource
  description: 'List all items with pagination.'
  parameters:
    - name: page
      type: integer
      required: false
      description: 'Page number (default: 1)'
  request_example: ''
  response_example: '{"data": [...]}'
  response_codes:
    - code: '200'
      description: 'Success'
    - code: '401'
      description: 'Unauthorized'
---

## Usage Notes

Additional documentation in markdown...
```

### Коллекция Postman

Включите JSON-файл коллекции Postman v2.1, использующий стандартные переменные окружения Grav API:

- `{{base_url}}` — URL сайта
- `{{api_prefix}}` — префикс API (по умолчанию: `/api/v1`)
- `{{api_key}}` — ключ API
- `{{grav_environment}}` — окружение Grav

Каждый запрос должен включать явные заголовки:

```json
{
    "header": [
        {"key": "X-API-Key", "value": "{{api_key}}"},
        {"key": "X-Grav-Environment", "value": "{{grav_environment}}"},
        {"key": "Content-Type", "value": "application/json"}
    ]
}
```

## События

Ваши эндпоинты API могут вызывать собственные события, чтобы другие плагины могли подключиться:

```php
// В вашем контроллере
$this->fireEvent('onMyPluginItemCreated', ['item' => $item]);
```

## Разрешения

Используйте существующую систему разрешений Grav. Распространённые шаблоны:

- **Эндпоинты чтения** → `api.system.read` или пользовательское разрешение
- **Эндпоинты записи** → `api.system.write` или пользовательское разрешение
- **Специфичные для плагина** → Зарегистрируйте собственные разрешения через `permissions.yaml`

## Интеграция с отчётами {: #reports-integration }

API предоставляет расширяемую плагинами систему отчётов на вкладке **Инструменты > Отчёты**. Плагины могут добавлять диагностические отчёты, слушая событие `onApiGenerateReports` и при необходимости предоставляя веб-компонент для пользовательского рендеринга.

### Как это работает

1. Эндпоинт `GET /reports` собирает встроенные отчёты (Security Check, YAML Linter)
2. Он вызывает событие `onApiGenerateReports`, чтобы плагины могли добавить свои отчёты
3. Каждый отчёт содержит структурированные данные и необязательную ссылку `component`
4. Если `component` задан, фронтенд Admin2 загружает веб-компонент плагина через `GET /gpm/plugins/{slug}/report-script/{reportId}`

### 1. Подписка на событие

```php
public static function getSubscribedEvents(): array
{
    return [
        // ... ваши существующие события ...
        'onApiGenerateReports' => ['onApiGenerateReports', 0],
    ];
}
```

### 2. Добавление данных отчёта

Прочитайте массив отчётов из события, добавьте свой отчёт и запишите его обратно. **Не используйте ссылки** — класс `Event` использует `ArrayAccess`, который не поддерживает ссылки `&`.

```php
public function onApiGenerateReports(Event $e): void
{
    // Запустить диагностические проверки
    $checker = new ProblemChecker();
    $problems = $checker->getProblems();

    $items = [];
    $hasCritical = false;

    foreach ($problems as $problem) {
        $item = $problem->toArray();
        unset($item['class'], $item['order']);
        $items[] = $item;

        if (!$problem->getStatus() && $problem->getLevel() === 'critical') {
            $hasCritical = true;
        }
    }

    // Прочитать, изменить, записать — НЕ используйте &$e['reports']
    $reports = $e['reports'];
    $reports[] = [
        'id'        => 'problems',
        'title'     => 'Grav Potential Problems',
        'provider'  => 'problems',           // слаг вашего плагина
        'component' => 'problems-report',    // ID веб-компонента (или null для рендеринга по умолчанию)
        'status'    => $hasCritical ? 'error' : 'success',
        'message'   => $hasCritical
            ? 'Critical problems found that need attention.'
            : 'No critical problems detected.',
        'items'     => $items,
    ];
    $e['reports'] = $reports;
}
```

### Структура данных отчёта

Каждый отчёт в массиве должен включать:

| Поле | Тип | Описание |
|-------|------|-------------|
| `id` | string | Уникальный идентификатор отчёта |
| `title` | string | Отображаемый заголовок над отчётом |
| `provider` | string | Слаг вашего плагина — используется для формирования URL скрипта веб-компонента |
| `component` | string\|null | ID веб-компонента или `null`, чтобы использовать рендерер по умолчанию |
| `status` | string | Общий статус: `success`, `warning` или `error` |
| `message` | string | Сводное сообщение в цветном баннере статуса |
| `items` | array | Элементы детализации отчёта (структура определяется вашим плагином) |

Когда `component` равен `null`, фронтенд Admin2 отрисовывает элементы встроенным рендерером по умолчанию. Когда задан, фронтенд загружает ваш пользовательский веб-компонент.

### 3. Создание веб-компонента (необязательно)

Если отчёту нужен пользовательский рендеринг, разместите JavaScript-файл по адресу:

```
your-plugin/admin-next/reports/{component-id}.js
```

Например, плагин Problems использует `admin-next/reports/problems-report.js`.

Скрипт должен определить пользовательский элемент с именем тега, переданным через `window.__GRAV_REPORT_TAG`:

```javascript
const TAG = window.__GRAV_REPORT_TAG || 'grav-problems--problems-report';

class ProblemsReportElement extends HTMLElement {
    #report = null;

    set report(val) {
        this.#report = val;
        this.render();
    }

    get report() {
        return this.#report;
    }

    connectedCallback() {
        if (this.#report) this.render();
    }

    render() {
        const report = this.#report;
        if (!report) return;

        const shadow = this.shadowRoot || this.attachShadow({ mode: 'open' });
        shadow.innerHTML = '';

        // Добавить стили
        const style = document.createElement('style');
        style.textContent = `
            :host { display: block; font-family: inherit; }
            .status-bar {
                padding: 10px 16px;
                font-size: 13px;
                font-weight: 600;
                color: #fff;
            }
            .status-bar.success { background: #22c55e; }
            .status-bar.error { background: #ef4444; }
            .status-bar.warning { background: #8b5cf6; }
            /* ... дополнительные стили ... */
        `;
        shadow.appendChild(style);

        // Отрисовать элементы отчёта
        for (const item of report.items) {
            // Построить собственный DOM для каждого элемента
            const section = document.createElement('div');
            // ... заполнить section ...
            shadow.appendChild(section);
        }
    }
}

customElements.define(TAG, ProblemsReportElement);
```

Ключевые моменты:
- Элемент получает полный объект отчёта через сеттер свойства `report`
- Используйте Shadow DOM (`attachShadow`) для изоляции стилей
- Имя тега внедряется как `window.__GRAV_REPORT_TAG` — всегда используйте его
- Для вызовов API из компонента используйте доступные глобальные переменные: `window.__GRAV_API_SERVER_URL`, `window.__GRAV_API_PREFIX`, `window.__GRAV_API_TOKEN`

### Соглашение о структуре файлов

```
your-plugin/
├── your-plugin.php               # Обработчик события с onApiGenerateReports
├── admin-next/
│   └── reports/
│       └── {component-id}.js     # Скрипт веб-компонента
```

## Текст справки в формах админки

Каждый пользователь админки выбирает в настройках, показывать ли текст справки к полю под меткой или во всплывающей подсказке за маленькой иконкой информации. Поля, созданные из схемы, следуют этому выбору автоматически. Плагин, который создаёт собственную разметку формы в админке, должен помещать справку в элемент `<grav-help>`, который админка регистрирует при загрузке, чтобы его справка тоже следовала выбору:

```html
<label>Store name <grav-help>Shown on receipts and emails.</grav-help></label>
```

Используйте его вместо абзаца-подсказки под меткой. Он работает внутри shadow root, применяет настройку на лету, а в старой админке, не знающей этого элемента, браузер просто показывает справку как обычный текст. Плагины, которые выводят справку иначе, могут читать `document.documentElement.dataset.helpMode` (`inline` или `tooltip`) и слушать событие `grav:help-mode` на `document`. [Документация admin-next](https://github.com/getgrav/grav-admin-next/blob/main/docs/help-element.md) описывает атрибуты элемента, событие и то, как перенести существующий плагин.

## Реальные примеры

- **Email Plugin** — регистрирует эндпоинты `/email/send` и `/email/test`
- **License Manager** — регистрирует полный CRUD для `/licenses` с проверкой формата
- **Problems Plugin** — добавляет диагностические отчёты через `onApiGenerateReports` с пользовательским веб-компонентом для отображения проверок версии PHP и статуса модулей

Все они следуют описанным выше шаблонам, и их можно найти в соответствующих репозиториях плагинов.
