---
title: "Интеграция с Admin2"
description: "Эндпоинты, обеспечивающие расширяемость и предпочтения Admin2: элементы панели меню, плавающие виджеты, выезжающие контекстные панели, панели страниц настроек, реестр пользовательских компонентов полей от плагинов, кнопки панели инструментов редактора markdown…"
tags:
  - "REST API"
  - "Admin Next"
---

# Интеграция с Admin2

Эндпоинты, обеспечивающие расширяемость и предпочтения Admin2: элементы панели меню, плавающие виджеты, выезжающие контекстные панели, панели страниц настроек, реестр пользовательских компонентов полей от плагинов, кнопки панели инструментов редактора markdown, а также пользовательские и сайтовые предпочтения Admin2 и брендинг.

Все эндпоинты требуют `api.access` и предназначены для вызова из Admin2, а не из конечных приложений. Запись сайтовых предпочтений и брендинга также требует суперпользователя. Реестры интерфейса работают на событиях (`onApiMenubarItems`, `onApiFloatingWidgets`, `onApiContextPanels`, `onApiAdminSettingsPanels`, `onApiMarkdownEditorButtons`), которые плагины перехватывают для регистрации элементов. Полный рецепт интеграции см. в [Руководстве разработчика](../../developer-guide/index.md).

## Элементы панели меню {: #menubar-items }

<span class="api-method api-method-get">GET</span> `/menubar/items`

Собирает элементы меню панели инструментов, зарегистрированные плагинами через событие `onApiMenubarItems`. Каждый элемент объявляет id, slug плагина-владельца, метку, иконку, ключ действия и необязательный запрос подтверждения. Admin2 отображает их как кнопки в верхней панели инструментов и отправляет POST на `/menubar/actions/{plugin}/{action}` при нажатии. Необязательные ключи представления управляют внешним видом: `variant` (default|primary|success|warning|danger), `showLabel` и `size` (sm|md). Необязательные ключи размещения управляют тем, где находится кнопка: `placement` (`start`, значение по умолчанию, — свободное место слева, вдали от действия «Очистить кэш»; `end` — рядом с основными действиями за разделителем) и `priority` (большее значение отображается раньше в пределах зоны). Вместо этого элемент может объявить клиентское намерение, которое выполняется в Admin2 без обращения к серверу: `route` выполняет навигацию в SPA (например, `/pages/new?parent=/blog&template=item`, чтобы открыть форму новой страницы с заданным родителем и зафиксированным шаблоном), а `modal` открывает один из собственных модальных веб-компонентов плагина (`{component, title?, props?, size?, useStandardHeader?}` — см. `GET /gpm/plugins/{slug}/modal-script/{modalId}`). Если присутствует `route` или `modal`, они имеют приоритет над `action`; `confirm` всё равно выполняется первым, если задан. Элемент может объявить `authorize` (строка разрешения или массив для проверки «любое из»): элементы, которые вызывающая сторона не проходит, не включаются, а `authorize` удаляется из ответа. Элементы возвращаются в порядке событий; Admin2 сортирует их по `priority`.

**Ответ**

```json
{"data": [{"id": "warm-cache", "plugin": "warm-cache", "label": "Warm Cache", "icon": "fa-tachometer", "action": "warm", "confirm": "Warm the cache?"}, {"id": "new-article", "plugin": "my-plugin", "label": "New Article", "icon": "fa-plus", "route": "/pages/new?parent=/blog&template=item"}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Элементы возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

---

## Выполнение действия панели меню {: #menubar-action }

<span class="api-method api-method-post">POST</span> `/menubar/actions/{plugin}/{action}`

Выполняет действие панели меню, зарегистрированное плагином. Вызывает `onApiMenubarAction` с ключами маршрутизации plugin/action, отправленным телом и текущим пользователем. Плагины, которым принадлежит slug `{plugin}`, задают `$event['result']`, например `{"status": "success", "message": "..."}` или `{"status": "error", "message": "..."}`. Результат возвращается как есть с HTTP 200 в любом случае, а Admin2 показывает всплывающее уведомление об успехе или ошибке в зависимости от `status`. Если зарегистрированный элемент панели меню соответствует этому плагину и действию, вызывающая сторона должна удовлетворять его `authorize`. Действие без соответствующего элемента требует только `api.access`, поэтому его обработчик должен сам проверять разрешения.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `plugin` | string | Да | Slug плагина-владельца (из регистрации элемента панели меню). |
| `action` | string | Да | Ключ действия (из регистрации элемента панели меню). |

**Запрос**

```json
{}
```

**Ответ**

```json
{"data": {"status": "success", "message": "Cache warmed successfully."}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Обработчик выполнен. Его результат возвращается как есть, включая результаты со `status: error`. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access` или вызывающая сторона не проходит `authorize`, объявленный в соответствующем элементе панели меню. |
| `404` | Ни один плагин не обработал эту пару plugin/action. |

---

## Плавающие виджеты {: #floating-widgets }

<span class="api-method api-method-get">GET</span> `/floating-widgets`

Собирает постоянные виджеты интерфейса (чат-ассистенты, панели уведомлений), зарегистрированные плагинами через `onApiFloatingWidgets`. Каждый виджет поставляет веб-компонент в `admin-next/widgets/{slug}.js`, который Admin2 загружает по требованию через `/gpm/plugins/{slug}/widget-script`. Виджет может задать `autoLoad`, чтобы загружать свой скрипт сразу, и `routes`, чтобы ограничить эту автозагрузку определёнными представлениями админки.

**Ответ**

```json
{"data": [{"id": "ai-pro-chat", "plugin": "ai-pro", "label": "AI Assistant", "icon": "bot", "priority": 10}, {"id": "users-enhancer", "plugin": "my-plugin", "label": "Users enhancer", "showFab": false, "autoLoad": true, "routes": ["/users"]}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Виджеты возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

Каждый виджет содержит `id`, `plugin`, `label`, `icon` и необязательный `priority`. Виджеты возвращаются в порядке событий; Admin2 сортирует их по `priority`. Виджет может объявить `authorize` (строка разрешения или массив для проверки «любое из»): виджеты, которые вызывающая сторона не проходит, не включаются, а `authorize` удаляется из ответа. Два необязательных флага управляют загрузкой:

- `autoLoad` — загружать скрипт виджета сразу, а не при первом нажатии на кнопку запуска. Используется расширениями полей и таблиц, которые регистрируют поведение, не показывая плавающую кнопку действия (обычно вместе с `showFab: false`).
- `routes` — список внутренних маршрутов SPA админки (`/users`, `/pages`, `/plugin/my-plugin`), к которым применяется автозагружаемый виджет. Admin2 загружает скрипт только при точном совпадении маршрута; опустите его, чтобы загружать на каждом маршруте. API нормализует каждую запись до пути с ведущей косой чертой и без завершающей и отбрасывает неверные. `routes` ограничивает только загрузку скрипта — это не граница безопасности; проверкой безопасности остаётся `authorize`.

Контракт `onApiFloatingWidgets` см. на странице [События API](../../events/index.md), а полный рецепт виджета — в [Руководстве разработчика](../../developer-guide/index.md#floating-widgets).

---

## Контекстные панели {: #context-panels }

<span class="api-method api-method-get">GET</span> `/context-panels`

Собирает регистрации выезжающих панелей. Контекстные панели вызываются кнопками панели инструментов внутри редакторов Admin2 — плагины перехватывают `onApiContextPanels`, чтобы зарегистрировать их, указывают, в каких контекстах редактора они появляются (например, `pages`), и поставляют веб-компонент в `admin-next/panels/{slug}.js`, отдаваемый через `/gpm/plugins/{slug}/panel-script`. Необязательный `badgeEndpoint` возвращает `{count: N}`, чтобы управлять значком на кнопке панели инструментов. Панель может объявить `authorize` (строка разрешения или массив для проверки «любое из»): панели, которые вызывающая сторона не проходит, не включаются, а `authorize` удаляется из ответа. Остальные сортируются по `priority`, сначала большее значение, при равенстве сохраняется порядок регистрации.

**Ответ**

```json
{"data": [{"id": "revisions-pro", "plugin": "revisions-pro", "label": "Revision History", "icon": "history", "contexts": ["pages"], "priority": 10, "width": 900, "badgeEndpoint": "/revisions-pro/badge"}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Панели возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

---

## Панели настроек {: #settings-panels }

<span class="api-method api-method-get">GET</span> `/settings/panels`

Собирает регистрации панелей настроек админки от плагинов через `onApiAdminSettingsPanels`. В отличие от полных страниц плагинов (регистрируемых через `onApiSidebarItems`), панели настроек отображаются как карточки внутри страницы настроек Admin2. Каждая панель следует формату страницы плагина в режиме схемы: файл схемы плюс эндпоинты данных и сохранения. Сортировка по `priority` по убыванию (сначала большее значение), при равенстве сохраняется порядок вставки. API не фильтрует панели по разрешениям, поэтому плагин должен проверить `$event['user']` перед добавлением панели.

**Ответ**

```json
{"data": [{"id": "login-settings", "plugin": "api", "label": "Login & Security", "description": "Authentication settings", "icon": "fa-shield-alt", "blueprint": "login-settings", "data_endpoint": "/login-settings/data", "save_endpoint": "/login-settings/save", "priority": 0}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Панели возвращены (отсортированы). |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

---

## Список пользовательских полей {: #custom-fields }

<span class="api-method api-method-get">GET</span> `/custom-fields`

Сопоставляет каждый пользовательский тип поля схемы плагину или теме, которые его поставляют, сканируя `admin-next/fields/*.js` в каждом включённом плагине и каждой установленной теме. Каждое значение — `{ slug, kind }`, где `kind` равно `plugins` или `themes`, поэтому Admin2 загружает скрипт каждого поля по правильному маршруту `/gpm/{kind}/{slug}/field/{type}`. Admin2 вызывает его один раз при загрузке, чтобы заполнить реестр пользовательских полей, и неизвестные типы полей в схемах корректно отображались с первого раза. Требуется только `api.access`, поэтому поля плагинов работают для редакторов, которые не могут управлять пакетами.

**Ответ**

```json
{"data": {"codeshtheme": {"slug": "codesh", "kind": "plugins"}, "products-status": {"slug": "license-manager", "kind": "plugins"}, "tinymce": {"slug": "mytheme", "kind": "themes"}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Реестр возвращён. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

---

## Кнопки панели инструментов редактора {: #editor-toolbar-buttons }

<span class="api-method api-method-get">GET</span> `/editor/toolbar-buttons`

Собирает кнопки, которые плагины добавляют в панель инструментов редактора markdown (CodeMirror) по умолчанию в Admin2 через событие `onApiMarkdownEditorButtons`. Кнопка либо открывает один из модальных веб-компонентов плагина (`modal`, отдаваемый через `/gpm/plugins/{plugin}/modal-script/{component}`), либо содержит данные `insert` (`{content, mode}`, где `mode` — `insert-at-cursor`, `append` или `replace`), которые редактор применяет напрямую. Кнопка может объявить `authorize` (строка разрешения или массив для проверки «любое из»): кнопки, которые вызывающая сторона не проходит, не включаются, а `authorize` удаляется из ответа. Кнопки возвращаются в порядке событий.

**Ответ**

```json
{"data": [{"id": "youtube", "plugin": "youtube", "label": "YouTube Video", "icon": "<svg viewBox=\"0 0 24 24\">...</svg>", "modal": {"component": "youtube-insert", "title": "Insert YouTube Video", "size": "md"}}, {"id": "hr", "plugin": "my-plugin", "label": "Horizontal Rule", "icon": "fa-minus", "insert": {"content": "---", "mode": "insert-at-cursor"}}]}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Кнопки возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

---

## Получение предпочтений Admin2 {: #get-preferences }

<span class="api-method api-method-get">GET</span> `/admin-next/preferences`

Возвращает полные данные предпочтений, которые Admin2 читает один раз при запуске: брендинг сайта и готовые URL логотипов (`branding_urls`), сайтовые значения по умолчанию (`site`), настройки только уровня сайта (`site_settings`), лимиты загрузки медиафайлов (`media_upload`), собственные сохранённые переопределения вызывающей стороны (`user`), объединённые действующие значения (`effective`) и `can_edit_site` (является ли вызывающая сторона суперпользователем, который может записывать сайтовые предпочтения и брендинг). Ответ содержит заголовок `ETag`.

**Ответ**

```json
{"data": {"branding": {"mode": "default", "text": "Grav", "logoLight": "", "logoDark": "", "logoHeight": 0, "title": "", "subtitle": "", "showPoweredBy": true, "favicon": ""}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271, "accentSaturation": 91, "fontFamily": "google-sans", "fontSize": "normal", "editorMode": "normal", "editorKeymap": "default", "editorStickyToolbar": true, "editorFixedHeight": 0, "adminLanguage": "en", "pagesPerPage": 20, "pagesViewMode": "miller", "usersViewMode": "cards", "groupsViewMode": "cards", "pluginsViewMode": "cards", "themesViewMode": "cards", "flexAfterSave": ""}, "site_settings": {"autoSaveEnabled": false, "autoSaveToolbarUndo": true, "autoSaveBatchWindowMs": 0, "collabEnabled": true, "menubarLinks": []}, "media_upload": {"resizeWidth": 0, "resizeHeight": 0, "resizeQuality": 0.8, "minWidth": 0, "minHeight": 0, "maxWidth": 0, "maxHeight": 0}, "user": {"colorMode": "dark"}, "effective": {"colorMode": "dark", "accentHue": 271, "accentSaturation": 91, "fontFamily": "google-sans", "fontSize": "normal", "editorMode": "normal", "editorKeymap": "default", "editorStickyToolbar": true, "editorFixedHeight": 0, "adminLanguage": "en", "pagesPerPage": 20, "pagesViewMode": "miller", "usersViewMode": "cards", "groupsViewMode": "cards", "pluginsViewMode": "cards", "themesViewMode": "cards", "flexAfterSave": "", "autoSaveEnabled": false, "autoSaveToolbarUndo": true, "autoSaveBatchWindowMs": 0, "collabEnabled": true, "menubarLinks": []}, "can_edit_site": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access`. |

Значения, которые может переопределить пользователь, разрешаются в таком порядке: встроенные значения по умолчанию, затем сайтовые значения по умолчанию (`ui.defaults` в `user/config/admin-next.yaml`), затем собственные переопределения пользователя (`admin_next.preferences` в файле его учётной записи). Если пользователь никогда не выбирал язык админки в Admin2, вместо него используется значение `language:` классической админки в учётной записи. Настройки только уровня сайта (`ui.settings`: автосохранение, совместная работа и ссылки в панели меню) применяются последними и не могут быть переопределены для отдельного пользователя.

---

## Обновление предпочтений пользователя {: #update-user-preferences }

<span class="api-method api-method-patch">PATCH</span> `/admin-next/preferences/user`

Сохраняет собственные переопределения предпочтений вызывающей стороны в `admin_next.preferences` в файле её учётной записи. Ключи, отсутствующие в теле, сохраняют своё значение, а ключ, установленный в `null`, удаляет это переопределение, и снова применяется сайтовое значение по умолчанию. Принимаются только ключи, переопределяемые пользователем (`colorMode`, `accentHue`, `accentSaturation`, `fontFamily`, `fontSize`, `editorMode`, `editorKeymap`, `editorStickyToolbar`, `editorFixedHeight`, `adminLanguage`, `pagesPerPage`, `pagesViewMode`, `usersViewMode`, `groupsViewMode`, `pluginsViewMode`, `themesViewMode`, `flexAfterSave`); ключи только уровня сайта, такие как `autoSaveEnabled` или `menubarLinks`, игнорируются. Неверные значения (неизвестный параметр, неверный тип) отбрасываются, а не отклоняются, а числа ограничиваются допустимым диапазоном. Возвращает полные разрешённые данные, как `GET /admin-next/preferences`. Демо-учётные записи отклоняются.

**Запрос**

```json
{"colorMode": "dark", "accentHue": 200, "fontSize": null}
```

**Ответ**

```json
{"data": {"branding": {"mode": "default", "text": "Grav"}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271}, "site_settings": {"autoSaveEnabled": false}, "media_upload": {"resizeWidth": 0}, "user": {"colorMode": "dark", "accentHue": 200}, "effective": {"colorMode": "dark", "accentHue": 200, "fontSize": "normal"}, "can_edit_site": false}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Переопределения сохранены; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access` или это демо-учётная запись. |

---

## Сброс предпочтений пользователя {: #reset-user-preferences }

<span class="api-method api-method-delete">DELETE</span> `/admin-next/preferences/user`

Удаляет все переопределения предпочтений, сохранённые вызывающей стороной (весь блок `admin_next.preferences` в файле её учётной записи), чтобы снова применялись сайтовые значения по умолчанию. Это основа действия «Сбросить к настройкам сайта» в Admin2. Макет панели управления (`admin_next.dashboard`) не затрагивается. Возвращает полные разрешённые данные, как `GET /admin-next/preferences`. Демо-учётные записи отклоняются.

**Ответ**

```json
{"data": {"branding": {"mode": "default", "text": "Grav"}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271}, "site_settings": {"autoSaveEnabled": false}, "media_upload": {"resizeWidth": 0}, "user": [], "effective": {"colorMode": "", "accentHue": 271}, "can_edit_site": false}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Переопределения очищены; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.access` или это демо-учётная запись. |

---

## Обновление сайтовых предпочтений {: #update-site-preferences }

<span class="api-method api-method-patch">PATCH</span> `/admin-next/preferences/site`

Сохраняет сайтовые предпочтения в `user/config/admin-next.yaml`. Плоское тело делится на две части. Ключи, переопределяемые пользователем (те же, что принимает `PATCH /admin-next/preferences/user`), сливаются в `ui.defaults`: меняются только отправленные ключи, ключ, установленный в `null`, удаляет сохранённое значение по умолчанию, и снова применяется встроенное значение, а неверные значения отбрасываются. Ключи только уровня сайта (`autoSaveEnabled`, `autoSaveToolbarUndo`, `autoSaveBatchWindowMs`, `collabEnabled`, `menubarLinks`) сливаются в `ui.settings`, также меняя только отправленные ключи. Любой другой ключ игнорируется. Требуется суперпользователь (с учётом ограничения области действия ключа API). Демо-учётные записи отклоняются. Возвращает полные разрешённые данные, как `GET /admin-next/preferences`.

**Запрос**

```json
{"accentHue": 271, "fontFamily": "inter", "colorMode": null, "autoSaveEnabled": true, "menubarLinks": [{"label": "Docs", "url": "https://learn.getgrav.org", "icon": "book", "external": true}]}
```

**Ответ**

```json
{"data": {"branding": {"mode": "default", "text": "Grav"}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271, "fontFamily": "inter"}, "site_settings": {"autoSaveEnabled": true, "autoSaveToolbarUndo": true, "autoSaveBatchWindowMs": 0, "collabEnabled": true, "menubarLinks": [{"label": "Docs", "url": "https://learn.getgrav.org", "icon": "book", "external": true}]}, "media_upload": {"resizeWidth": 0}, "user": [], "effective": {"colorMode": "", "accentHue": 271, "fontFamily": "inter", "autoSaveEnabled": true}, "can_edit_site": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Сайтовые предпочтения сохранены; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Не суперпользователь или это демо-учётная запись. |

Каждая запись `menubarLinks` требует непустых `label` и `url` и может содержать `icon` и `external`; записи без любого из обязательных полей отбрасываются.

---

## Обновление брендинга {: #update-branding }

<span class="api-method api-method-patch">PATCH</span> `/admin-next/branding`

Обновляет брендинг Admin2, хранящийся в `ui.branding` в `user/config/admin-next.yaml`. Тело сливается с текущим брендингом, поэтому отправляйте только поля, которые хотите изменить. Поля: `mode` (`default` показывает логотип Grav, `text` показывает `text`, `custom` показывает загруженные логотипы; неизвестные значения откатываются к `default`), `text` (до 64 символов), `logoHeight` (высота пользовательского логотипа в боковой панели в CSS-пикселях, ограничена от 16 до 44; `0` использует встроенные 28px), `title` (заголовок экрана входа и вкладки браузера, до 64 символов), `subtitle` (подзаголовок экрана входа, до 128 символов), `showPoweredBy` (показывать строку «Powered by Grav CMS» на экранах входа и настройки) и `logoLight`, `logoDark` и `favicon`, которые принимают только базовое имя внутри `user/media/admin-next/`. Загружайте и удаляйте эти файлы через `POST` и `DELETE /admin-next/branding/logo`. Требуется суперпользователь (с учётом ограничения области действия ключа API). Демо-учётные записи отклоняются. Возвращает полные разрешённые данные, как `GET /admin-next/preferences`.

**Запрос**

```json
{"mode": "text", "text": "Acme CMS", "title": "Acme Content Admin", "logoHeight": 32, "showPoweredBy": false}
```

**Ответ**

```json
{"data": {"branding": {"mode": "text", "text": "Acme CMS", "logoLight": "", "logoDark": "", "logoHeight": 32, "title": "Acme Content Admin", "subtitle": "", "showPoweredBy": false, "favicon": ""}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271}, "site_settings": {"autoSaveEnabled": false}, "media_upload": {"resizeWidth": 0}, "user": [], "effective": {"colorMode": "", "accentHue": 271}, "can_edit_site": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Брендинг сохранён; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Не суперпользователь или это демо-учётная запись. |

---

## Загрузка логотипа брендинга {: #upload-branding-logo }

<span class="api-method api-method-post">POST</span> `/admin-next/branding/logo`

Загружает светлый логотип, тёмный логотип или favicon (multipart, поле `file` или `logo`) в `user/media/admin-next/` и записывает его в брендинг сайта. Тип файла определяется по содержимому файла, а не по объявленному клиентом MIME-типу: PNG, JPEG, WebP и ICO принимаются как изображения, а всё остальное должно разбираться как документ SVG, который санируется (удаляются элементы script и foreignObject, атрибуты `on*` и URL `javascript:`/`data:`). Максимальный размер — 4 МБ. Файл сохраняется как `logo-{variant}-{hash}.{ext}` (или `favicon-favicon-{hash}.{ext}`), а предыдущий файл для этого варианта удаляется. Загрузка светлого или тёмного логотипа переключает `mode` брендинга на `custom`, если он не равен `text`. Требуется суперпользователь (с учётом ограничения области действия ключа API). Демо-учётные записи отклоняются. Возвращает полные разрешённые данные; `branding_urls` содержит URL нового файла.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `variant` | string | Да | Параметр запроса (или поле multipart): `light`, `dark` или `favicon`. Если переданы оба, побеждает параметр запроса. |
| `file` | file | Да | Файл изображения (поле multipart `file` или `logo`). |

**Ответ**

```json
{"data": {"branding": {"mode": "custom", "text": "Grav", "logoLight": "logo-light-3f9c2a7e5b.svg", "logoDark": "", "logoHeight": 0, "title": "", "subtitle": "", "showPoweredBy": true, "favicon": ""}, "branding_urls": {"light": "/user/media/admin-next/logo-light-3f9c2a7e5b.svg", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271}, "site_settings": {"autoSaveEnabled": false}, "media_upload": {"resizeWidth": 0}, "user": [], "effective": {"colorMode": "", "accentHue": 271}, "can_edit_site": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `201` | Логотип сохранён; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Не суперпользователь или это демо-учётная запись. |
| `422` | Отсутствует или неверен `variant`, файл не загружен, файл больше 4 МБ либо содержимое не является поддерживаемым изображением или действительным SVG. |

---

## Удаление логотипа брендинга {: #delete-branding-logo }

<span class="api-method api-method-delete">DELETE</span> `/admin-next/branding/logo`

Удаляет сохранённый файл для одного варианта брендинга и очищает его путь в брендинге сайта. Если оба варианта логотипа окажутся пустыми, а режим — `custom`, режим возвращается к `default`, чтобы Admin2 показывал встроенный логотип Grav. Завершается успешно, даже если для варианта не было задано файла. Требуется суперпользователь (с учётом ограничения области действия ключа API). Демо-учётные записи отклоняются. Возвращает полные разрешённые данные, как `GET /admin-next/preferences`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `variant` | string | Да | Параметр запроса: `light`, `dark` или `favicon`. |

**Ответ**

```json
{"data": {"branding": {"mode": "default", "text": "Grav", "logoLight": "", "logoDark": "", "logoHeight": 0, "title": "", "subtitle": "", "showPoweredBy": true, "favicon": ""}, "branding_urls": {"light": "", "dark": "", "favicon": ""}, "site": {"colorMode": "", "accentHue": 271}, "site_settings": {"autoSaveEnabled": false}, "media_upload": {"resizeWidth": 0}, "user": [], "effective": {"colorMode": "", "accentHue": 271}, "can_edit_site": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Вариант очищен; разрешённые предпочтения возвращены. |
| `401` | Не авторизован. |
| `403` | Не суперпользователь или это демо-учётная запись. |
| `422` | Отсутствует или неверен `variant`. |
