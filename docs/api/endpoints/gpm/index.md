---
title: "Менеджер пакетов"
description: "Эндпоинты для управления пакетами Grav (плагинами и темами) через GPM, включая установку, удаление, обновления и поиск по репозиторию, а также эндпоинты, которые отдают Admin2 страницы админки плагинов и скрипты веб-компонентов…"
tags:
  - "REST API"
  - "Справочник API"
  - "GPM"
---

# Менеджер пакетов

Эндпоинты для управления пакетами Grav (плагинами и темами) через GPM, включая установку, удаление, обновления и поиск по репозиторию, а также эндпоинты, которые отдают Admin2 страницы админки плагинов и скрипты веб-компонентов.

## Список плагинов {: #list-plugins }

<span class="api-method api-method-get">GET</span> `/gpm/plugins`

Выводит все установленные плагины с флагами включения, состоянием обновления и сведениями о том, установлен ли каждый плагин через символическую ссылку (это важно для контроля обновлений — пакеты, подключённые символической ссылкой, не следует перезаписывать). Требуется `api.gpm.read`.

**Ответ**

```json
{"data": [{"slug": "simplesearch", "name": "SimpleSearch", "version": "3.0.0", "type": "plugin", "description": "A simple search plugin for Grav.", "description_html": "<p>A simple search plugin for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-simplesearch", "enabled": true, "is_symlink": false, "available_version": "3.0.1", "updatable": true}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Плагины возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |

Каждый элемент — объект пакета. `version` — установленная версия; `available_version` появляется только когда `updatable` равно `true`. Необязательные поля схемы (`docs`, `bugs`, `dependencies`, `compatibility`, `keywords`, `icon`) появляются только если плагин их задаёт, а премиум-плагины добавляют `premium`, `licensed`, а когда известно — `vendor` и `purchase_url`. Плагин, настройки которого выводятся на странице админки, добавляет `settings_route` (маршрут-хеш), а если эта страница принадлежит другому плагину — ещё и `settings_page`. `name` или `description`, записанные как ключ перевода, переводятся на язык админки вызывающей стороны.

---

## Получение плагина {: #get-plugin }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}`

Возвращает сведения об одном установленном плагине: тот же объект пакета, что и в разделе «Список плагинов», плюс `custom_fields`, если плагин поставляет компоненты полей Admin Next. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |

**Ответ**

```json
{"data": {"slug": "simplesearch", "name": "SimpleSearch", "version": "3.0.0", "type": "plugin", "description": "A simple search plugin for Grav.", "description_html": "<p>A simple search plugin for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-simplesearch", "enabled": true, "is_symlink": false, "available_version": "3.0.1", "updatable": true, "custom_fields": {"simplesearch-preview": "simplesearch-preview"}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Плагин возвращён. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |
| `404` | Плагин не установлен. |

Ответ содержит заголовок `ETag`. Отправьте его обратно как `If-None-Match`, и API ответит `304 Not Modified` с пустым телом, если ничего не изменилось. `custom_fields` сопоставляет каждый тип поля, который плагин поставляет в `admin-next/fields/`, с самим собой.

---

## Установка пакета {: #install-package }

<span class="api-method api-method-post">POST</span> `/gpm/install`

Устанавливает плагин или тему из репозитория GPM вместе со всеми нужными зависимостями. Требуется `api.gpm.write`. Вызывает `onApiBeforePackageInstall` и `onApiPackageInstalled`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `package` | string | Да | Slug устанавливаемого пакета |
| `type` | string | Нет | Тип пакета: `plugin` (по умолчанию) или `theme` |
| `license` | string | Нет | Лицензионный ключ для премиум-пакета. Сохраняется только если slug указывает на реальный пакет, и удаляется снова, если установка не удалась. |

**Запрос**

```json
{"package": "sitemap", "type": "plugin"}
```

**Ответ**

```json
{"data": {"message": "Plugin 'sitemap' installed successfully.", "package": "sitemap", "type": "plugin", "dependencies": []}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `201` | Пакет установлен. `dependencies` перечисляет зависимости, установленные или обновлённые вместе с ним. |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.write` |
| `422` | Ошибка валидации (неверный тип, пакет уже установлен, некорректный лицензионный ключ, премиум-пакет без лицензии или не пройдена проверка зависимостей и требований) |
| `500` | Установка не удалась. В сообщении перечислены зависимости, установленные до сбоя. |

---

## Удаление пакета {: #remove-package }

<span class="api-method api-method-post">POST</span> `/gpm/remove`

Удаляет установленный плагин или тему. Требуется `api.gpm.write`. Вызывает `onApiBeforePackageRemove` и `onApiPackageRemoved`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `package` | string | Да | Slug удаляемого пакета |

**Запрос**

```json
{"package": "sitemap"}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `204` | Пакет удалён |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.write` |
| `404` | Пакет не установлен |
| `422` | Отсутствует поле `package` |
| `500` | Удаление не удалось. Сообщение об ошибке — простой текст с удалёнными цветовыми кодами консоли GPM. |

---

## Проверка обновлений {: #check-updates }

<span class="api-method api-method-get">GET</span> `/gpm/updates`

Проверяет наличие обновлений для плагинов, тем и ядра Grav. Перечисляются только пакеты, у которых есть обновление. `total` учитывает как сам Grav, так и обновления плагинов и тем. Объект `grav` содержит `is_symlink`, чтобы интерфейсы админки могли отключать действие обновления ядра, когда Grav подключён символической ссылкой. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `flush` | boolean | Нет | Обновить кэшированные данные репозитория GPM перед проверкой. |

**Ответ**

```json
{"data": {"grav": {"current": "2.1.0", "available": "2.1.1", "updatable": true, "date": "2026-09-18", "is_symlink": false}, "plugins": [{"slug": "form", "name": "Form", "version": "8.1.0", "type": "plugin", "description": "Enables forms handling and processing.", "description_html": "<p>Enables forms handling and processing.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-form", "enabled": true, "is_symlink": false, "available_version": "8.1.1", "updatable": true}], "themes": [], "total": 2, "installed": 14}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Сводка обновлений возвращена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |

В `grav` поле `current` — установленная версия, а `available` — последняя версия в репозитории (`null`, если репозиторий не удалось прочитать). В `plugins` и `themes` каждый элемент — объект пакета, где `version` — установленная версия, а `available_version` — новая. `installed` — общее число установленных плагинов и тем.

---

## Поиск по репозиторию {: #search-repository }

<span class="api-method api-method-get">GET</span> `/gpm/search`

Ищет плагины и темы в репозитории GPM. Плагины выводятся раньше тем, и каждый результат содержит флаг `installed`. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `q` | string | Да | Строка поискового запроса, сопоставляется с именем, slug и описанием |
| `page` | integer | Нет | Номер страницы для постраничной разбивки (по умолчанию: 1) |
| `per_page` | integer | Нет | Число результатов на странице (по умолчанию: 20, максимум: 1000) |

**Ответ**

```json
{"data": [{"slug": "simplesearch", "name": "SimpleSearch", "version": "3.0.1", "type": "plugin", "description": "A simple search plugin for Grav.", "description_html": "<p>A simple search plugin for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-simplesearch", "installed": true}], "meta": {"pagination": {"page": 1, "per_page": 20, "total": 1, "total_pages": 1}}, "links": {"self": "/api/v1/gpm/search?page=1&per_page=20"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.read` |
| `422` | Отсутствует параметр `q` |

---

## Список тем {: #list-themes }

<span class="api-method api-method-get">GET</span> `/gpm/themes`

Выводит все установленные темы с URL миниатюры и скриншота, состоянием обновления и флагом `is_symlink`. Для тем `enabled` означает, что тема активна. Требуется `api.gpm.read`.

**Ответ**

```json
{"data": [{"slug": "quark", "name": "Quark", "version": "2.0.4", "type": "theme", "description": "Quark is the default theme for Grav.", "description_html": "<p>Quark is the default theme for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-theme-quark", "enabled": true, "is_symlink": false, "updatable": false, "thumbnail": "/api/v1/thumbnails/8c1f2a9e4b.jpg", "screenshot": "/api/v1/thumbnails/0d3e7b61fa.jpg"}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Темы возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |

`thumbnail` (не больше 500px) и `screenshot` (не больше 2000px) — это URL вида `/thumbnails/{file}`, построенные из собственных изображений темы `thumbnail` и `screenshot`. Если у темы есть только одно из двух, оба поля используют его; если нет ни одного, оба равны `null`.

---

## Получение темы {: #get-theme }

<span class="api-method api-method-get">GET</span> `/gpm/themes/{slug}`

Возвращает сведения об одной установленной теме: тот же объект пакета, что и в разделе «Список тем», плюс `custom_fields`, если тема поставляет компоненты полей Admin Next. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug темы. |

**Ответ**

```json
{"data": {"slug": "quark", "name": "Quark", "version": "2.0.4", "type": "theme", "description": "Quark is the default theme for Grav.", "description_html": "<p>Quark is the default theme for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-theme-quark", "enabled": true, "is_symlink": false, "updatable": false, "thumbnail": "/api/v1/thumbnails/8c1f2a9e4b.jpg", "screenshot": "/api/v1/thumbnails/0d3e7b61fa.jpg"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Тема возвращена. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |
| `404` | Тема не установлена. |

Ответ содержит заголовок `ETag`. Отправьте его обратно как `If-None-Match`, и API ответит `304 Not Modified` с пустым телом, если ничего не изменилось.

---

## Обновление пакета {: #update-package }

<span class="api-method api-method-post">POST</span> `/gpm/update`

Обновляет конкретный установленный плагин или тему до последней версии, сначала устанавливая или обновляя нужные зависимости. Требуется `api.gpm.write`. Вызывает `onApiBeforePackageUpdate` и `onApiPackageUpdated`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `package` | string | Да | Slug обновляемого пакета |

**Запрос**

```json
{"package": "form"}
```

**Ответ**

```json
{"data": {"message": "Package 'form' updated successfully.", "package": "form", "type": "plugin", "dependencies": []}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Пакет обновлён |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.write` |
| `404` | Пакет не установлен |
| `422` | Пакет уже обновлён до последней версии или не пройдена проверка зависимостей и требований |
| `500` | Обновление не удалось. В сообщении перечислены зависимости, установленные до сбоя. |

---

## Обновление всех пакетов {: #update-all }

<span class="api-method api-method-post">POST</span> `/gpm/update-all`

Обновляет все обновляемые плагины и темы за один запрос, используя свежие данные репозитория. Каждый пакет обрабатывается независимо — один сбой не прерывает пакет. Требуется `api.gpm.write`.

**Ответ**

```json
{"data": {"updated": ["simplesearch", "seo-magic"], "failed": [{"package": "broken-plugin", "error": "Dependency unmet"}], "skipped": [{"package": "form", "reason": "already up to date (installed as a dependency)"}], "cascaded_dependencies": ["form"]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Пакет обработан; результат по каждому пакету в `updated[]` / `failed[]` / `skipped[]`. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.write`. |

Ответ содержит четыре массива:

- `updated`: slug'и, которые были обновлены.
- `failed`: объекты с `package` и `error` для пакетов, чьи требования не выполнены или обновление которых не удалось.
- `skipped`: объекты с `package` и `reason` для пакетов, уже обновлённых как зависимость более раннего пакета в рамках этой операции.
- `cascaded_dependencies`: slug'и, установленные или обновлённые попутно как зависимости.

---

## Просмотр тем репозитория {: #browse-themes }

<span class="api-method api-method-get">GET</span> `/gpm/repository/themes`

Выводит все темы, доступные в репозитории GPM, с флагом `installed` у каждой. Поддерживает постраничную разбивку и текстовый поиск `q`. 502, если репозиторий недоступен. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `q` | string | Нет | Поисковый запрос для фильтрации тем (сопоставляется с name / slug / description). |
| `page` | integer | Нет | Номер страницы (по умолчанию 1). |
| `per_page` | integer | Нет | Элементов на странице (по умолчанию из конфигурации; максимум 2000, чтобы модальное окно установки могло получить полный список). |

**Ответ**

```json
{"data": [{"slug": "quark", "name": "Quark", "version": "2.0.4", "type": "theme", "description": "Quark is the default theme for Grav.", "description_html": "<p>Quark is the default theme for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-theme-quark", "screenshot": "https://getgrav.org/images/quark-screenshot.jpg", "installed": true}], "meta": {"pagination": {"page": 1, "per_page": 20, "total": 120, "total_pages": 6}}, "links": {"self": "/api/v1/gpm/repository/themes?page=1&per_page=20", "next": "/api/v1/gpm/repository/themes?page=2&per_page=20", "last": "/api/v1/gpm/repository/themes?page=6&per_page=20"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Темы репозитория возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |
| `502` | Репозиторий GPM недоступен. |

В результатах репозитория `version` — последняя версия в репозитории, а `screenshot` — полный URL скриншота из репозитория. Премиум-темы добавляют `premium`, `licensed`, а когда известно — `vendor` и `purchase_url`.

---

## Получение README {: #get-readme }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/readme`

Возвращает необработанное содержимое README.md установленного плагина или темы. Также доступно по адресу `/gpm/themes/{slug}/readme`. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug пакета |

**Ответ**

```json
{"data": {"content": "# My Plugin\n\nThis plugin does..."}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.read` |
| `404` | Пакет или README не найден |
| `422` | Неверный slug пакета |

---

## Получение журнала изменений {: #get-changelog }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/changelog`

Возвращает необработанное содержимое CHANGELOG.md установленного плагина или темы. Также доступно по адресу `/gpm/themes/{slug}/changelog`. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug пакета |

**Ответ**

```json
{"data": {"content": "# v1.2.0\n## 01/15/2025\n\n* Added feature X..."}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.read` |
| `404` | Пакет или журнал изменений не найден |
| `422` | Неверный slug пакета |

---

## Получение страницы плагина {: #plugin-page }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/page`

Возвращает описание страницы админки для плагина: тип страницы, ссылку на схему, эндпоинты данных и сохранения, а также кнопки действий. Порядок поиска: 1) событие onApiPluginPageInfo, 2) `admin-next/pages/{slug}.yaml`, 3) `admin-next/pages/{slug}.js` (режим выведенного компонента). Требуется только `api.access`; данные, лежащие в основе страницы, подчиняются собственным разрешениям плагина.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина |

**Ответ**

```json
{"data": {"id": "license-manager", "plugin": "license-manager", "title": "License Manager", "icon": "fa-key", "page_type": "blueprint", "blueprint": "licenses", "data_endpoint": "/licenses/form-data", "save_endpoint": "/licenses", "actions": [{"id": "save", "label": "Save", "icon": "fa-check", "primary": true}], "has_custom_component": false}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.access` |
| `404` | Для плагина не найдена страница админки |

Описание может также содержать `settings_route` — маршрут-хеш внутри страницы, где выводятся настройки плагина, и `settings_page` — slug другого установленного плагина, страница админки которого их выводит. `settings_page` сохраняется только вместе с действительным `settings_route`.

---

## Получение скрипта страницы плагина {: #plugin-page-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/page-script`

Отдаёт JavaScript-файл веб-компонента уровня страницы для плагина. Файл загружается из `admin-next/pages/{slug}.js` в папке плагина.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Плагин не установлен или не включён, либо нет компонента страницы. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение журнала изменений Grav {: #grav-changelog }

<span class="api-method api-method-get">GET</span> `/gpm/grav/changelog`

Возвращает журнал изменений ядра Grav для всех выпусков новее установленной версии, объединённый в один Markdown-документ с заголовком `# v{version} ({date})` для каждого выпуска. `content` — пустая строка, когда Grav обновлён до последней версии. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `flush` | boolean | Нет | Сначала обновить кэшированные данные репозитория GPM. |

**Ответ**

```json
{"data": {"content": "# v2.1.1 (09/18/2026)\n\n1. [](#bugfix)\n    * Fixed ..."}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.gpm.read` |

---

## Обновление ядра Grav {: #upgrade-core }

<span class="api-method api-method-post">POST</span> `/gpm/upgrade`

Обновляет ядро Grav. Отказывается работать, если Grav установлен через символическую ссылку (типично для сред разработки и монорепозиториев). Сначала выполняет предварительные проверки обновления. Требуется `api.gpm.write`. Вызывает `onApiBeforeGravUpgrade` и `onApiGravUpgraded` вокруг обновления.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `override` | boolean | Нет | Обновлять, даже если предварительные проверки сообщают о блокирующих проблемах. |

**Запрос**

```json
{"override": false}
```

**Ответ**

```json
{"data": {"message": "Grav upgraded successfully.", "previous_version": "2.1.0", "new_version": "2.1.1"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Ядро обновлено. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.write`. |
| `409` | Заблокировано предварительными проверками. Тело перечисляет проблемы; повторите с `override: true`, чтобы всё равно обновиться. |
| `422` | Grav уже последней версии или установлен через символическую ссылку. |
| `500` | Обновление прервалось на середине. |

Когда предварительные проверки блокируют обновление, ответ `409` — это обычные данные `data`, а не ошибка:

```json
{
    "data": {
        "status": "preflight_failed",
        "message": "Upgrade blocked by preflight checks.",
        "blocking": [
            "Some enabled plugins/themes have not been marked as compatible with Grav 2.2. Disable them before continuing."
        ],
        "warnings": [],
        "incompatible_packages": { "target": "2.2" },
        "can_override": true
    }
}
```

---

## Прямая установка {: #direct-install }

<span class="api-method api-method-post">POST</span> `/gpm/direct-install`

Устанавливает плагин или тему из URL или загруженного zip-архива в обход поиска в репозитории GPM. Принимает либо JSON-тело с `url`, либо multipart-загрузку с полем `file`. Удобно для приватных и неопубликованных пакетов, предварительных сборок или локальной разработки. Требуется `api.gpm.write`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `url` | string | Нет | URL zip-архива пакета с `http://` или `https://` (взаимоисключающе с `file`). Пути на сервере отклоняются; вместо этого загрузите локальный архив как `file`. |
| `file` | file | Нет | Загруженный zip-архив (multipart; взаимоисключающе с `url`). |

**Запрос**

```json
{"url": "https://downloads.example.com/builds/my-plugin-1.0.0.zip"}
```

**Ответ**

```json
{"data": {"message": "Package installed successfully via direct install."}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `201` | Пакет установлен. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.write`. |
| `422` | Не передан ни `url`, ни действительный `file`, либо `url` не является адресом http(s). |
| `500` | Установка не удалась. |

---

## Список плагинов репозитория {: #repository-plugins }

<span class="api-method api-method-get">GET</span> `/gpm/repository/plugins`

Выводит все плагины, доступные в репозитории GPM, с флагом `installed` у каждого. Поддерживает постраничную разбивку (`page`, `per_page` — максимум 2000, чтобы модальное окно установки могло получить полный список) и текстовый поиск `q`. 502, если репозиторий недоступен. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `page` | integer | Нет | Номер страницы (по умолчанию 1). |
| `per_page` | integer | Нет | Элементов на странице (по умолчанию из конфигурации; максимум 2000). |
| `q` | string | Нет | Поисковый фильтр, сопоставляемый с name / slug / description. |

**Ответ**

```json
{"data": [{"slug": "simplesearch", "name": "SimpleSearch", "version": "3.0.1", "type": "plugin", "description": "A simple search plugin for Grav.", "description_html": "<p>A simple search plugin for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-simplesearch", "keywords": ["search", "plugin"], "installed": true}], "meta": {"pagination": {"page": 1, "per_page": 50, "total": 250, "total_pages": 5}}, "links": {"self": "/api/v1/gpm/repository/plugins?page=1&per_page=50", "next": "/api/v1/gpm/repository/plugins?page=2&per_page=50", "last": "/api/v1/gpm/repository/plugins?page=5&per_page=50"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Плагины репозитория возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |
| `502` | Репозиторий GPM недоступен. |

В результатах репозитория `version` — последняя версия в репозитории. Чтобы сравнить с установленным, используйте [Список плагинов](#list-plugins) или [Проверку обновлений](#check-updates). Премиум-плагины добавляют `premium`, `licensed`, а когда известно — `vendor` и `purchase_url`.

---

## Получение пакета репозитория {: #repository-package }

<span class="api-method api-method-get">GET</span> `/gpm/repository/{slug}`

Возвращает полные сведения из репозитория о плагине или теме (в зависимости от того, что соответствует slug), плюс флаг `installed`. Требуется `api.gpm.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug пакета. |

**Ответ**

```json
{"data": {"slug": "simplesearch", "name": "SimpleSearch", "version": "3.0.1", "type": "plugin", "description": "A simple search plugin for Grav.", "description_html": "<p>A simple search plugin for Grav.</p>", "author": {"name": "Team Grav", "email": "devs@getgrav.org", "url": "https://getgrav.org"}, "homepage": "https://github.com/getgrav/grav-plugin-simplesearch", "keywords": ["search", "plugin"], "installed": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Пакет найден. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.gpm.read`. |
| `404` | Пакет не найден в репозитории GPM. |

---

## Получение скрипта пользовательского поля {: #custom-field-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/field/{type}`

Отдаёт JavaScript-веб-компонент для пользовательского типа поля схемы плагина. Также доступно по адресу `/gpm/themes/{slug}/field/{type}` для полей, предоставляемых темой. Admin2 загружает их по требованию при рендеринге схемы, использующей неизвестный тип поля. Соглашение: файл находится в `admin-next/fields/{type}.js` внутри пакета. В отличие от остальных эндпоинтов скриптов, этот отдаёт и поля выключенного плагина, чтобы его форму настроек можно было отредактировать до включения.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина (или темы). |
| `type` | string | Да | Идентификатор типа поля (совпадает со значением `type:` в схеме). |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Пакет или компонент поля не найден. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение скрипта отчёта {: #report-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/report-script/{reportId}`

Отдаёт веб-компонент для пользовательского отчёта, показанного на странице отчётов Admin2. Плагины поставляют файл в `admin-next/reports/{reportId}.js` и регистрируют отчёт через событие `onApiGenerateReports`. Admin2 загружает скрипт только при рендеринге соответствующего отчёта.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |
| `reportId` | string | Да | Идентификатор отчёта (совпадает с `id` из `onApiGenerateReports`). |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Плагин не установлен или не включён, либо компонент отчёта не найден. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение скрипта плавающего виджета {: #widget-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/widget-script`

Отдаёт веб-компонент для плавающего виджета, предоставленного плагином. Соглашение: файл находится в `admin-next/widgets/{slug}.js`. Регистрируется через событие `onApiFloatingWidgets` и выводится в `GET /floating-widgets`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Плагин не установлен или не включён, либо компонент виджета не найден. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение скрипта контекстной панели {: #panel-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/panel-script`

Отдаёт веб-компонент для контекстной панели, предоставленной плагином (панель в правой панели, показываемая при редактировании страниц или другого контента). Соглашение: файл находится в `admin-next/panels/{slug}.js`. Регистрируется через событие `onApiContextPanels` и выводится в `GET /context-panels`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Плагин не установлен или не включён, либо компонент панели не найден. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение скрипта модального окна {: #modal-script }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/modal-script/{modalId}`

Отдаёт веб-компонент для модального диалога, предоставленного плагином. Соглашение: файл находится в `admin-next/modals/{modalId}.js` и монтируется как пользовательский элемент `grav-{slug}--modal-{modalId}`. Открывается из веб-компонента плагина через `window.__GRAV_DIALOGS.open()` или напрямую из элемента панели меню с намерением `modal`. Плагин может поставлять несколько разных модальных окон, каждое со своим `modalId`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина. |
| `modalId` | string | Да | Идентификатор компонента модального окна — базовое имя файла в `admin-next/modals/`. |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | JavaScript-файл отдан. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Плагин не установлен или не включён, либо компонент модального окна не найден. |

Требуется только `api.access`, поэтому любой администратор, имеющий доступ к экранам плагина, может загрузить их без прав менеджера пакетов. Скрипт отдаётся как `application/javascript` с `Cache-Control: private, no-cache` и `ETag`, построенным по времени изменения и размеру файла, поэтому повторный запрос с `If-None-Match` получает пустой `304`, пока файл не изменится.

---

## Получение набора пользовательских полей {: #custom-field-bundle }

<span class="api-method api-method-get">GET</span> `/gpm/plugins/{slug}/fields`

Отдаёт все веб-компоненты пользовательских полей пакета одним ответом в виде JSON-объекта, сопоставляющего каждый тип поля с его исходным кодом JavaScript. Admin2 запрашивает его один раз на пакет вместо одного запроса на каждый тип поля. Также доступно по адресу `/gpm/themes/{slug}/fields`. Отдаётся и для выключенных плагинов, чтобы их формы настроек оставались редактируемыми.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `slug` | string | Да | Slug плагина (или темы). |

**Ответ**

```json
{"seo-magic-preview": "class SeoMagicPreview extends HTMLElement { ... }", "seo-magic-keywords": "class SeoMagicKeywords extends HTMLElement { ... }"}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Набор отдан. Пустой объект, если пакет не поставляет компонентов полей. |
| `304` | Не изменено. Заголовок `If-None-Match` совпадает с текущим ETag. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |
| `404` | Пакет не найден. |
| `422` | Неверный slug пакета. |

Тело — это чистый JSON-объект, без обычной обёртки `data`. Он отдаётся с `Cache-Control: private, no-cache` и `ETag`, построенным по имени, времени изменения и размеру каждого файла, поэтому повторная проверка стоит только `304`, пока какой-либо файл поля не изменится. Требуется только `api.access`.
