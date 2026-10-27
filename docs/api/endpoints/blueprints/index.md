---
title: "Схемы"
description: "Эндпоинты для получения схем, используемых для шаблонов страниц, учётных записей пользователей, групп, плагинов, тем, конфигурации и определений разрешений, а также GET /blueprint-files, который перечисляет файлы для полей выбора папки схемы…"
tags:
  - "REST API"
  - "Схемы"
---

# Схемы

Эндпоинты для получения схем, используемых для шаблонов страниц, учётных записей пользователей, групп, плагинов, тем, конфигурации и определений разрешений, а также `GET /blueprint-files`, который перечисляет файлы для полей выбора папки схемы. Загрузка и удаление файлов для полей схемы с типом `file` описаны в разделе [Медиафайлы](../media/index.md): `POST /blueprint-upload` и `DELETE /blueprint-upload`.

## Список типов страниц {: #list-page-types }

<span class="api-method api-method-get">GET</span> `/blueprints/pages`

Выводит шаблоны страниц (схемы), доступные для новых страниц, после применения хуков `onAdminPageTypes` / `onAdminModularPageTypes` и настроенных скрытых типов. Требуется `api.pages.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `modular` | boolean | Нет | Параметр запроса: `true` (или `1` / `yes`) выводит модульные шаблоны вместо обычных шаблонов страниц |

**Ответ**

```json
{"data": [{"type": "default", "label": "Default"}, {"type": "blog", "label": "Blog"}, {"type": "post", "label": "Post"}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.pages.read` |

---

## Получение схемы страницы {: #get-page-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/pages/{template}`

Возвращает полностью разрешённую схему для шаблона страницы, включая унаследованные поля и импорты. Требуется `api.pages.read`. Вызывает `onApiBlueprintResolved` с `context: page`, чтобы плагины могли скорректировать сериализованные поля.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `template` | string | Да | Имя шаблона страницы (например, default, blog, post или modular/hero для модульного шаблона) |

**Ответ**

```json
{"data": {"name": "default", "title": "Default", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "header.title", "type": "text", "label": "Title"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.pages.read` |
| `404` | Схема шаблона не найдена |

---

## Получение схемы пользователя {: #get-user-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/users`

Возвращает схему учётной записи пользователя для рендеринга формы редактирования пользователя. Нужно только `api.access`, так как каждый вошедший пользователь рендерит с её помощью форму своего профиля. Вызывает `onApiBlueprintResolved` с `context: account`.

**Ответ**

```json
{"data": {"name": "account", "title": "Account", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "username", "type": "text", "label": "Username"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `404` | Схема пользователя не найдена |

---

## Получение разрешений {: #get-permissions }

<span class="api-method api-method-get">GET</span> `/blueprints/users/permissions`

Возвращает все зарегистрированные в системе действия разрешений, организованные иерархически с переведёнными метками. Требуется `api.users.read`. Ключ `children` появляется только у действий, у которых есть дочерние действия.

**Ответ**

```json
{"data": [{"name": "admin", "label": "Admin", "children": [{"name": "admin.login", "label": "Login"}]}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.users.read` |

---

## Получение схемы страницы плагина {: #plugin-page-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/plugins/{plugin}/pages/{pageId}`

Возвращает пользовательскую схему страницы для плагина. Требуется `api.config.read`. Загружает YAML-файл из admin/blueprints/{pageId}.yaml в папке плагина и возвращает его как сериализованную схему с разрешёнными полями. Если этот файл отсутствует, а `pageId` совпадает со slug плагина, вместо него используется основной `blueprints.yaml` плагина. Вызывает `onApiBlueprintResolved` с `context: plugin-page`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `plugin` | string | Да | Slug плагина |
| `pageId` | string | Да | Идентификатор схемы страницы (совпадает с именем файла без расширения) |

**Ответ**

```json
{"data": {"name": "licenses", "title": "Licenses", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "licenses", "type": "array", "label": "Licenses"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.config.read` |
| `404` | Плагин или схема страницы не найдены |

---

## Получение схемы плагина {: #plugin-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/plugins/{plugin}`

Возвращает `blueprints.yaml` плагина с разрешёнными полями (метки переведены, параметры данных раскрыты). Требуется `api.config.read`. Вызывает событие `onApiBlueprintResolved` с `context: plugin`, чтобы плагины могли изменить сериализованные поля (например, добавить динамические параметры).

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `plugin` | string | Да | Slug плагина. |

**Ответ**

```json
{"data": {"name": "simplesearch", "title": "SimpleSearch", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "enabled", "type": "toggle", "label": "Plugin status"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Схема возвращена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.config.read`. |
| `404` | Плагин или его `blueprints.yaml` не найдены. |

---

## Получение схемы темы {: #theme-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/themes/{theme}`

Возвращает `blueprints.yaml` темы с разрешёнными полями. Требуется `api.config.read`. Вызывает `onApiBlueprintResolved` с `context: theme`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `theme` | string | Да | Slug темы. |

**Ответ**

```json
{"data": {"name": "quark", "title": "Quark", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "dropdown.enabled", "type": "toggle", "label": "Dropdown in navbar"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Схема возвращена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.config.read`. |
| `404` | Тема или её `blueprints.yaml` не найдены. |

---

## Получение схемы конфигурации {: #config-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/config/{scope}`

Возвращает схему, описывающую область конфигурации system/site. Admin2 использует её вместе с `GET /config/{scope}` для рендеринга форм конфигурации. Ищет схему через поток `blueprints://`, чтобы учитывались переопределения плагинов (например, `media.yaml` плагина admin), а затем откатывается к `system://blueprints/config/{scope}.yaml`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `scope` | string | Да | Одна из основных областей `system`, `site`, `media`, `security`, `scheduler`, `backups` или пользовательская область конфигурации верхнего уровня, заданная сайтом (например, `user/blueprints/config/<scope>.yaml`). Другие системные схемы, такие как `streams`, отклоняются. |

**Ответ**

```json
{"data": {"name": "system", "title": "System Configuration", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "cache.enabled", "type": "toggle", "label": "Caching"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Схема возвращена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.config.read`. |
| `404` | Неизвестная область или файл схемы не найден. |

---

## Разрешение параметров данных {: #resolve-data }

<span class="api-method api-method-get">GET</span> `/data/resolve`

Разрешает директивы `data-options@: \Some\Class::method`, используемые в определениях полей схемы (например, список шаблонов страниц, список тем). Admin2 вызывает его при рендеринге схемы, поле select/checkboxes которой ссылается на вызываемый метод. Принимаются только вызываемые методы `Class::method` из белого списка утверждённых поставщиков данных (`Blueprint::isSafeDynamicCall()`) — белый список с точным совпадением предотвращает выполнение произвольного кода. Поставщики ядра утверждены заранее; плагины и темы регистрируют свои через `Blueprint::addAllowedDynamicCallable()` при запуске. Результаты нормализуются до `[{value, label}]` для использования с select.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `callable` | string | Да | Полное имя `\Class\Name::method` для вызова. Должно быть утверждённым поставщиком данных (из белого списка ядра или зарегистрированным через `Blueprint::addAllowedDynamicCallable()`). |
| `type` | string | Нет | Дополнительный аргумент, передаваемый, когда `callable` — это `Grav\Common\Page\Pages::pageTypes` (`standard` / `modular`). По умолчанию `standard`. |

**Ответ**

```json
{"data": [{"value": "default", "label": "Default"}, {"value": "blog", "label": "Blog"}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Параметры возвращены (пустой массив, если метод не возвращает массив). |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.pages.read`. |
| `404` | Класс или метод не существует. |
| `422` | Отсутствует или неверен `callable`, либо вызываемый метод не является утверждённым поставщиком данных, либо не соответствует формату `Class::method`. |

---

## Получение схемы группы {: #group-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/groups`

Возвращает схему редактирования группы пользователей (`user/group.yaml`, разрешается через поток `blueprints://`, поэтому переопределения сайта применяются). Требуется `api.users.read`. Вызывает `onApiBlueprintResolved` с `context: group`.

**Ответ**

```json
{"data": {"name": "group", "title": "Group", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "readableName", "type": "text", "label": "Display Name"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.users.read` |
| `404` | Схема группы не найдена |

---

## Получение схемы новой группы {: #new-group-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/groups/new`

Возвращает схему создания группы пользователей (`user/group_new.yaml`, разрешается через поток `blueprints://`, поэтому переопределения сайта применяются). Требуется `api.users.read`. Вызывает `onApiBlueprintResolved` с `context: group_new`.

**Ответ**

```json
{"data": {"name": "group_new", "title": "Add Group", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "groupname", "type": "text", "label": "Group Name"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.users.read` |
| `404` | Схема создания группы не найдена |

---

## Получение схемы конфигурации учётных записей {: #accounts-config-blueprint }

<span class="api-method api-method-get">GET</span> `/blueprints/config/accounts`

Возвращает схему конфигурации учётных записей Flex — форму на вкладке «Конфигурация» страницы «Пользователи». Требуется `api.config.read`. Строится по той же схеме директории Flex, что использует классическая админка, поэтому включает общую вкладку кэширования наряду с настройками учётных записей пользователей. Используйте её вместе с `GET /config/accounts`. Вызывает `onApiBlueprintResolved` с `context: config` и `template: accounts`.

**Ответ**

```json
{"data": {"name": "accounts", "title": "Accounts Configuration", "type": null, "child_type": null, "validation": "loose", "fields": [{"name": "object.cache.index.enabled", "type": "toggle", "label": "Enable Index Caching"}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.config.read` |
| `404` | Flex Objects недоступен или директория Flex `user-accounts` не зарегистрирована |

---

## Список файлов папки схемы {: #list-blueprint-files }

<span class="api-method api-method-get">GET</span> `/blueprint-files`

Выводит файлы в папке, на которую указывает поле схемы со своим параметром `folder:` (filepicker, mediapicker), как `taskGetFilesInFolder` в классической админке. Требуется `api.media.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `folder` | string | Да | Параметр запроса: папка для вывода. Поток Grav (`user://media`, `theme://images`, `account://`), токен `self@:subpath`, разрешаемый относительно `scope`, или путь относительно `user/` |
| `scope` | string | Нет | Параметр запроса: владелец схемы, используемый для разрешения `self@:`. Одно из значений: `plugins/<slug>`, `themes/<slug>`, `pages/<route>`, `users/<username>` |
| `accept` | string | Нет | Параметр запроса: фильтр расширений через запятую (`.pdf`, `*.jpg`) или шаблонов MIME (`image/*`). `*` совпадает со всем |

**Ответ**

```json
{"data": [{"filename": "logo.png", "url": "/user/themes/quark/images/logo.png", "type": "image/png", "size": 18432, "dimensions": {"width": 400, "height": 120}, "thumbnail_url": "/api/v1/thumbnails/3f2a9c1e4b5d6a7f8e9d0c1b2a394857.png", "modified": "2026-09-01T10:15:00+00:00"}], "meta": {"pagination": {"page": 1, "per_page": 1, "total": 1, "total_pages": 1}, "folder": "themes/quark/images", "scope": null, "exists": true}, "links": {"self": "/api/v1/blueprint-files?page=1&per_page=1"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех. Ещё не существующая папка возвращает пустой список с `meta.exists: false` |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.media.read` или используется область `users/<username>` другого аккаунта без `api.users.write` |
| `422` | Отсутствует `folder`, есть обход пути, неразрешимый поток или область, либо папка медиафайлов страницы `@self` / `self@` |

Результат использует обёртку с постраничной разбивкой, но на самом деле никогда не разбивается на страницы: все подходящие файлы приходят на странице 1.

Литералы медиафайлов страницы `@self` и `self@` (без подпути) здесь не обрабатываются, так как у клиента уже есть медиафайлы страницы из `GET /pages/{route}/media`. Они возвращают стандартную ошибку валидации 422, в записи `errors` для `folder` которой стоит сообщение `PAGE_MEDIA_ONLY`:

```json
{"status": 422, "title": "Unprocessable Entity", "detail": "Use /pages/{route}/media for @self / self@ folders.", "errors": [{"field": "folder", "message": "PAGE_MEDIA_ONLY"}]}
```
