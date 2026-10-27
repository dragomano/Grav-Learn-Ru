---
title: "Система"
description: "Эндпоинты для операций уровня системы, включая информацию, управление кэшем, окружения, журналы, резервные копии, переводы и отчёты."
tags:
  - "REST API"
---

# Система

Эндпоинты для операций уровня системы, включая информацию, управление кэшем, окружения, журналы, резервные копии, переводы и отчёты.

## Информация о системе {: #system-info }

<span class="api-method api-method-get">GET</span> `/system/info`

Подробная информация о системе: версии Grav и PHP, загруженные расширения, серверное ПО, текущее окружение, установленные плагины и темы, а также структурированная сводка конфигурации PHP (загрузки, память, обработка ошибок, пути, сессии, OPcache, безопасность, локаль). Предназначен для заполнения страницы «Информация о системе» и страниц поддержки в Admin2. Требуется `api.system.read`.

**Ответ**

```json
{"data": {"grav_version": "2.1.0", "php_version": "8.3.14", "php_extensions": ["Core", "date", "pcre"], "server_software": "Apache/2.4.58", "environment": "localhost", "plugins": [{"name": "API", "version": "1.0.38", "enabled": true}], "themes": [{"name": "Quark", "version": "2.0.4", "active": true}], "php_config": {"Upload & POST": {"file_uploads": "1", "upload_max_filesize": "64M", "max_file_uploads": "20", "post_max_size": "64M"}}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Информация возвращена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.read`. |

Для демо-учётных записей `server_software` и значения путей в `php_config` (журнал ошибок, open_basedir, временная папка, корень документов, include path, путь сохранения сессий) заменяются на `(hidden in demo mode)`.

---

## Очистка кэша {: #clear-cache }

<span class="api-method api-method-delete">DELETE</span> `/cache`

Очищает системный кэш. По умолчанию используется область `standard` (как у `bin/grav cache clear`), что соответствует типичной кнопке «сбросить кэш». Используйте `scope=all`, чтобы также удалить кэши изображений и ресурсов. Требуется `api.system.write`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `scope` | string | Нет | Одно из значений: `all`, `standard`, `images`, `assets`, `tmp`. По умолчанию `standard`. `images`, `assets` и `tmp` очищают только соответствующую папку. |

**Ответ**

```json
{"data": {"scope": "standard", "message": "Cache cleared successfully (scope: standard).", "details": ["Cleared:  /var/www/grav/cache/*"]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Кэш очищен. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.write`. |
| `422` | Неверная область кэша. |

`details` — список строк результатов по папкам от очистки кэша Grav.

---

## Получение переводов {: #get-translations }

<span class="api-method api-method-get">GET</span> `/translations/{lang}`

Возвращает все строки перевода для языка в виде плоского отображения ключ/значение. Это публичный маршрут: аутентификация не нужна. Ключи, которые язык не переводит, заполняются из английского (`en-US`), поэтому отображение полно для любого языка.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `lang` | string | Да | Код языка (например, `en`, `fr`, `pt-BR`). Короткие коды нормализуются до полной локали (`en` становится `en-US`, `fr` — `fr-FR`); значение, не являющееся кодом языка, заменяется языком сайта по умолчанию. Итоговый код возвращается в `lang`. |
| `prefix` | string | Нет | Возвращать только ключи, начинающиеся с этого префикса, за которым идёт точка (без учёта регистра), например `PLUGIN_ADMIN`. |

**Ответ**

```json
{"data": {"lang": "fr-FR", "dir": "ltr", "count": 1868, "checksum": "5d41402abc4b2a76b9719d911017c592", "strings": {"PLUGIN_ADMIN.SAVE": "Enregistrer"}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |

`dir` — направление текста (`ltr` или `rtl`), а `checksum` — MD5 возвращённых строк, поэтому клиент может кэшировать отображение и запрашивать его заново только при изменении контрольной суммы. Строки, которые добавляют только выключенные плагины, не включаются.

---

## Проверка доступности {: #ping }

<span class="api-method api-method-get">GET</span> `/ping`

Облегчённая проверка работоспособности. Это публичный маршрут: учётные данные не нужны и разрешения не проверяются, поэтому он лишь подтверждает, что API доступен.

**Ответ**

```json
{"data": {"pong": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |

---

## Список окружений {: #list-environments }

<span class="api-method api-method-get">GET</span> `/system/environments`

Выводит окружения, которые можно выбрать в качестве цели записи конфигурации, а также окружение, определённое Grav для этого запроса. Требуется `api.system.read`.

**Ответ**

```json
{"data": {"detected": "localhost", "environments": [{"name": "", "label": "Default", "exists": true, "hasOverrides": false}, {"name": "staging.example.com", "label": "staging.example.com", "exists": true, "hasOverrides": true}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.system.read` |

Первый элемент всегда является базовой конфигурацией (`user/config`) с `name: ""` и `label: "Default"`; его `hasOverrides` всегда `false`. Каждый остальной элемент — существующая папка `user/env/<name>/`, причём включаются и устаревшие макеты `user/<host>/config/`. `hasOverrides` сообщает, содержит ли окружение какие-либо файлы конфигурации.

---

## Получение журналов {: #get-logs }

<span class="api-method api-method-get">GET</span> `/system/logs`

Читает записи журнала из файла журнала (по умолчанию `grav.log`) с постраничной разбивкой, фильтрацией по уровню и текстовым поиском. Записи возвращаются сначала новые. Требуется `api.system.read`; демо-учётные записи получают 403.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `file` | string | Нет | Файл журнала для чтения (по умолчанию `grav.log`). Должен быть одним из имён, перечисленных в `GET /system/logs/files`. |
| `page` | integer | Нет | Номер страницы для постраничной разбивки (по умолчанию: 1) |
| `per_page` | integer | Нет | Число результатов на странице (по умолчанию: 20, максимум: 1000) |
| `level` | string | Нет | Фильтр по уровню журнала без учёта регистра (например, DEBUG, INFO, NOTICE, WARNING, ERROR, CRITICAL) |
| `search` | string | Нет | Текст без учёта регистра, который должно содержать сообщение |

**Ответ**

```json
{"data": [{"date": "2026-09-21T10:30:00.123456+00:00", "logger": "grav", "level": "WARNING", "message": "Plugin X deprecated method"}], "meta": {"pagination": {"page": 1, "per_page": 20, "total": 150, "total_pages": 8}}, "links": {"self": "/api/v1/system/logs?page=1&per_page=20", "next": "/api/v1/system/logs?page=2&per_page=20", "last": "/api/v1/system/logs?page=8&per_page=20"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех. Зарегистрированный, но ещё не записанный журнал возвращает пустую страницу. |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.system.read` или это демо-учётная запись |
| `422` | Неизвестный файл журнала |

---

## Отчёты {: #reports }

<span class="api-method api-method-get">GET</span> `/reports`

Возвращает расширяемые плагинами диагностические отчёты. Встроенные отчёты — проверка безопасности (сканирование XSS), Twig в контенте и линтер YAML. Плагины могут добавлять собственные отчёты через событие onApiGenerateReports. Требуется `api.reports.read`.

**Ответ**

```json
{
    "data": [
        {
            "id": "security-check",
            "title": "Grav Security Check",
            "provider": "core",
            "component": null,
            "status": "success",
            "message": "Security Scan complete: No issues found.",
            "items": []
        },
        {
            "id": "twig-content",
            "title": "Twig in Content",
            "provider": "core",
            "component": null,
            "status": "success",
            "message": "Twig in Content is disabled. No pages are leaking raw Twig.",
            "meta": { "gate": false, "sandbox": true, "editor_enabled": false, "leak_count": 0, "event_count": 0, "global_request_gated": false, "frontmatter_request_gated": false },
            "items": []
        },
        {
            "id": "yaml-linter",
            "title": "Grav Yaml Linter",
            "provider": "core",
            "component": null,
            "status": "success",
            "message": "YAML Linting: No errors found.",
            "items": []
        },
        {
            "id": "problems",
            "title": "Grav Potential Problems",
            "provider": "problems",
            "component": "problems-report",
            "status": "success",
            "message": "No critical problems detected.",
            "items": [
                {
                    "id": "PHP Minimum Version",
                    "level": "critical",
                    "status": true,
                    "msg": "Your PHP 8.5.4 is greater than the minimum of 8.3.0 required"
                }
            ]
        }
    ]
}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Отчёты успешно возвращены |
| `401` | Не авторизован |
| `403` | Запрещено — требуется разрешение `api.reports.read` |

### Структура отчёта

Каждый отчёт в массиве ответа содержит:

| Поле | Тип | Описание |
|-------|------|-------------|
| `id` | string | Уникальный идентификатор отчёта |
| `title` | string | Читаемое название отчёта |
| `provider` | string | Источник — `core` для встроенных или slug плагина |
| `component` | string\|null | Идентификатор веб-компонента для пользовательского рендеринга или `null` для стандартного |
| `status` | string | Общий статус: `success`, `warning` или `error` |
| `message` | string | Краткое сообщение |
| `meta` | object | Необязательные сводные значения, специфичные для отчёта |
| `items` | array | Элементы подробностей, специфичные для отчёта |

### Встроенные отчёты

- **Проверка безопасности** — сканирует все страницы на потенциальные уязвимости XSS. Элементы содержат `route` и `field` для каждой найденной проблемы.
- **Twig в контенте** (`twig-content`): показывает, включён ли Twig в контенте страниц и работает ли он в песочнице, какие страницы выводят посетителям необработанные маркеры Twig и какие токены недавно блокировала песочница. В его `meta` есть `gate`, `sandbox`, `editor_enabled`, `leak_count`, `event_count`, `global_request_gated` и `frontmatter_request_gated`; элементы — строки с `kind: leak` и `kind: event`. Эндпоинты под `/reports/twig-content/` работают с этим отчётом.
- **Линтер YAML** — проверяет все YAML-файлы на синтаксические ошибки. Элементы содержат `file` и `error` для каждой найденной проблемы.

### Отчёты плагинов

Плагины добавляют отчёты, прослушивая событие `onApiGenerateReports`. Когда отчёт указывает `component`, фронтенд админки загружает веб-компонент плагина из `GET /gpm/plugins/{provider}/report-script/{component}`.

Подробности реализации см. в руководстве [Интеграция плагинов с API](../../../plugins/plugin-api-integration/index.md#reports-integration).

---

## Создание резервной копии {: #create-backup }

<span class="api-method api-method-post">POST</span> `/system/backup`

Запускает резервное копирование сайта. Использует встроенный класс `Backups` Grav, записывает zip-архив с датой в настроенное место хранения резервных копий и возвращает имя файла, путь, размер и метку времени создания. Admin2 использует его для действия «Создать резервную копию» на панели управления. Требуется `api.system.backup`, так как архивы содержат хеши паролей учётных записей и секреты конфигурации; демо-учётные записи получают 403.

**Ответ**

```json
{"data": {"filename": "default_site_backup--20260917120000.zip", "path": "/var/www/grav/backup/default_site_backup--20260917120000.zip", "size": 1048576, "date": "2026-09-17T12:00:00+00:00"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `201` | Резервная копия создана; заголовок Location указывает на `/system/backups`. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.backup` или это демо-учётная запись. |

---

## Список резервных копий {: #list-backups }

<span class="api-method api-method-get">GET</span> `/system/backups`

Выводит существующие файлы резервных копий (имя файла, название, дата, размер), а также настроенную политику очистки и число настроенных профилей резервного копирования. Требуется `api.system.backup`; демо-учётные записи получают 403.

**Ответ**

```json
{"data": {"backups": [{"filename": "default_site_backup--20260917120000.zip", "title": "Default Site Backup", "date": "2026-09-17T12:00:00+00:00", "size": 1048576}], "purge": {"trigger": "space", "max_backups_count": 25, "max_backups_space": 5, "max_backups_time": 365}, "profiles_count": 1}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Список резервных копий возвращён. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.backup` или это демо-учётная запись. |

`date` — метка времени ISO 8601 в том же формате, что возвращает [Создание резервной копии](#create-backup). `purge` — конфигурация сайта `backups.purge`.

---

## Удаление резервной копии {: #delete-backup }

<span class="api-method api-method-delete">DELETE</span> `/system/backups/{filename}`

Удаляет zip-архив резервной копии. Имя файла проверяется, чтобы предотвратить обход пути — это должно быть простое базовое имя, оканчивающееся на `.zip`. Требуется `api.system.backup`; демо-учётные записи получают 403.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `filename` | string | Да | Имя файла резервной копии (простое базовое имя, например `default_site_backup--20260917120000.zip`). |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `204` | Резервная копия удалена. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.backup` или это демо-учётная запись. |
| `404` | Резервная копия не найдена. |
| `422` | Неверное имя файла. |

---

## Скачивание резервной копии {: #download-backup }

<span class="api-method api-method-get">GET</span> `/system/backups/{filename}/download`

Передаёт zip-файл резервной копии вызывающей стороне. Возвращает `application/zip` с заголовком `Content-Disposition: attachment`. Требуется `api.system.backup`; демо-учётные записи получают 403.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `filename` | string | Да | Имя файла резервной копии (простое базовое имя, оканчивающееся на `.zip`). |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Резервная копия передана. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.backup` или это демо-учётная запись. |
| `404` | Резервная копия не найдена. |
| `422` | Неверное имя файла. |

---

## Отчёт о системе {: #systeminfo-report }

<span class="api-method api-method-get">GET</span> `/systeminfo`

Формирует компактный отчёт о системе: сведения PHP (версия, SAPI, расширения, лимиты памяти и загрузки), версия Grav, свободное и общее место на диске, число плагинов и состояние кэша. Легче, чем `/system/info`, и предназначен для виджета «Состояние системы» на панели управления, а не для полной страницы информации о системе в Admin2. Требуется `api.system.read`.

**Ответ**

```json
{"data": {"php": {"version": "8.3.14", "sapi": "fpm-fcgi", "extensions": ["Core", "date"], "memory_limit": "256M", "max_execution_time": "60", "upload_max_filesize": "64M", "post_max_size": "64M"}, "grav": {"version": "2.1.0", "php_version": "8.3.14"}, "disk": {"free_space": 10737418240, "total_space": 53687091200}, "plugins": {"total": 30, "enabled": 24, "disabled": 6}, "cache": {"enabled": true, "driver": "auto"}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Отчёт возвращён. |
| `401` | Не авторизован. |
| `403` | Нет разрешения `api.system.read`. |

`disk.free_space` и `disk.total_space` указаны в байтах или равны `false`, когда PHP не может их прочитать.

---

## Создание окружения {: #create-environment }

<span class="api-method api-method-post">POST</span> `/system/environments`

Создаёт пустую папку окружения в `user/env/<name>/config`, которую затем можно выбрать в качестве цели записи конфигурации. Требуется `api.config.write`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `name` | string | Да | Имя окружения, обычно имя хоста. Должно начинаться с буквы или цифры и содержать только буквы, цифры, `.`, `_` и `-`. `default` и `base` зарезервированы. |

**Запрос**

```json
{"name": "staging.example.com"}
```

**Ответ**

```json
{"data": {"name": "staging.example.com", "label": "staging.example.com", "exists": true, "hasOverrides": false}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `201` | Окружение создано |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.config.write` |
| `422` | Неверное или зарезервированное имя, либо окружение уже существует |

---

## Удаление окружения {: #delete-environment }

<span class="api-method api-method-delete">DELETE</span> `/system/environments/{name}`

Удаляет папку окружения `user/env/<name>/` и всё её содержимое. Требуется суперадминистратор, так как папка может содержать переопределения `system` и `security`, которые вправе записывать только суперадминистратор.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `name` | string | Да | Имя окружения (параметр пути) |

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `204` | Окружение удалено |
| `401` | Не авторизован |
| `403` | Не суперадминистратор |
| `404` | Папка `user/env/<name>/` не существует |
| `422` | Неверное имя, это окружение, которое Grav использует для текущего запроса, или устаревший макет `user/<name>/` (удалите его вручную) |

---

## Список файлов журналов {: #list-log-files }

<span class="api-method api-method-get">GET</span> `/system/logs/files`

Выводит файлы журналов, которые может открыть просмотрщик журналов админки. Эти имена — единственные значения, которые `GET /system/logs` и `DELETE /system/logs` принимают для `file`. Требуется `api.system.read`; демо-учётные записи получают 403.

**Ответ**

```json
{"data": {"files": [{"file": "grav.log", "label": "Grav System Log"}, {"file": "security.log", "label": "Security Log"}, {"file": "email.log", "label": "Email Log"}, {"file": "scheduler.log", "label": "Scheduler Log"}, {"file": "custom-plugin.log", "label": "Custom Plugin Log"}], "default": "grav.log"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.system.read` или это демо-учётная запись |

Четыре основных журнала всегда выводятся первыми, даже до того, как в них что-то записано. Затем идут любые другие файлы `*.log` в папке `log://`, а после них — файлы, которые добавляют плагины через событие `onApiLogFiles`. Записи дедуплицируются по имени файла.

---

## Очистка журнала {: #clear-log }

<span class="api-method api-method-delete">DELETE</span> `/system/logs`

Очищает один из файлов журналов, перечисленных в `GET /system/logs/files`. Файл усекается, а не удаляется, и на его месте остаётся одна запись о том, кто его очистил. Требуется суперадминистратор; демо-учётные записи получают 403. Вызывает `onApiLogCleared`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `file` | string | Нет | Файл журнала для очистки (по умолчанию `grav.log`). Можно передать в теле JSON или параметром запроса `?file=`. |

**Запрос**

```json
{"file": "grav.log"}
```

**Ответ**

```json
{"data": {"file": "grav.log", "cleared_bytes": 450507, "message": "Log cleared: grav.log"}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Журнал очищен или уже был пуст (`cleared_bytes: 0`) |
| `401` | Не авторизован |
| `403` | Не суперадминистратор или это демо-учётная запись |
| `422` | Неизвестный файл журнала или файл недоступен для записи |

---

## Список языков админки {: #admin-languages }

<span class="api-method api-method-get">GET</span> `/admin/languages`

Выводит языки, на которых может отображаться сам интерфейс админки, по одному на каждый файл `languages/*.yaml`, поставляемый плагином Admin2. Это отличается от `GET /languages`, который перечисляет языки контента сайта. Сортировка по родному названию; пусто, если Admin2 не установлен. Требуется `api.system.read`.

**Ответ**

```json
{"data": {"languages": [{"code": "en-US", "name": "English", "native_name": "English", "rtl": false}, {"code": "fr-FR", "name": "French", "native_name": "Français", "rtl": false}]}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.system.read` |

---

## Разрешение заблокированного токена Twig {: #twig-content-allowlist }

<span class="api-method api-method-post">POST</span> `/reports/twig-content/allowlist`

Добавляет один токен, который заблокировала песочница Twig для контента страниц, в соответствующий список `security.twig_sandbox.allowed_*`, сохраняемый в `user/config/security.yaml`. Недавние события блокировки этого токена удаляются из отчёта «Twig в контенте». Требуется суперадминистратор (ключу API нужно `admin.super` в его областях действия).

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `rule` | string | Да | Правило песочницы, заблокировавшее токен: `tag`, `filter`, `function`, `method` или `property`. |
| `token` | string | Да | Имя тега, фильтра, функции, метода или свойства, которое нужно разрешить. |
| `class` | string | Нет | Класс-владелец. Обязателен для `method` и `property`, в остальных случаях игнорируется. |

**Запрос**

```json
{"rule": "filter", "token": "markdown"}
```

**Ответ**

```json
{"data": {"rule": "filter", "key": "allowed_filters", "value": ["markdown"], "resolved": 2}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Токен добавлен (или уже присутствовал) и сохранён |
| `401` | Не авторизован |
| `403` | Не суперадминистратор |
| `422` | Неизвестное правило, пустой токен или отсутствующий класс для `method`/`property` |

`key` — записанный ключ `security.twig_sandbox`, а `value` — его полное новое значение: список имён для `tag`, `filter` и `function` или список строк `{class, methods}` для `method` и `property`. `resolved` — число удалённых записанных событий блокировки. Поставляемые значения по умолчанию живут в коде, поэтому файл содержит только собственные дополнения сайта. Вызывает `onApiConfigUpdated` с областью `security`.

---

## Очистка событий Twig в контенте {: #twig-content-clear-events }

<span class="api-method api-method-delete">DELETE</span> `/reports/twig-content/events`

Очищает список недавно заблокированных или вырезанных токенов Twig в контенте, который читают отчёт «Twig в контенте» и баннер редактора страниц. `logs/security.log` не затрагивается. Требуется `api.system.write`.

**Ответ**

```json
{"data": {"cleared": true}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Очищено. `cleared` равно false, только если сохранённый список не удалось удалить. |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.system.write` |

---

## Состояние Twig в контенте для страницы {: #twig-content-page }

<span class="api-method api-method-get">GET</span> `/reports/twig-content/page`

Состояние «Twig в контенте» для одной страницы, используется баннером редактора страниц: покажет ли страница посетителям необработанные маркеры `{{ }}` / `{% %}` и какие недавние события блокировки записаны для её маршрута. Требуется `api.reports.read`.

**Параметры**

| Имя | Тип | Обязательный | Описание |
| ---- | ---- | -------- | ----------- |
| `route` | string | Да | Маршрут страницы с ведущей косой чертой. |

**Ответ**

```json
{"data": {"route": "/blog/my-first-post", "gate": false, "sandbox": true, "leak": {"route": "/blog/my-first-post", "requested": true, "gate": false, "reason": "gate_off"}, "events": []}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.reports.read` |
| `422` | Отсутствует параметр `route` |

`gate` — это `security.twig_content.process_enabled`, а `sandbox` — `security.twig_sandbox.enabled`. `leak` равен `null`, когда на странице нет маркеров Twig, она использует модульный или тематический Twig, её Twig будет отрендерен как обычно или её не удаётся найти (отсутствующая страница не является ошибкой). Его `reason` равен `gate_off`, когда общий переключатель сайта выключен, или `page_off`, когда переключатель включён, но страница не запрашивает обработку Twig. `events` перечисляет недавние блокировки именно для этого маршрута, сначала новые.

---

## Сканирование Twig в контенте {: #twig-content-scan }

<span class="api-method api-method-get">GET</span> `/reports/twig-content/scan`

Сканирует все страницы (пропуская модульный и тематический Twig) на теги, фильтры и функции Twig в контенте, которые не разрешены текущими списками песочницы, то есть на то, что потребуется контенту до включения общего переключателя. Требуется `api.reports.read`.

**Ответ**

```json
{"data": {"tags": {"include": ["/blog/my-first-post"]}, "filters": {"raw": ["/about"]}, "functions": {"source": ["/about", "/contact"]}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.reports.read` |

Каждый список сопоставляет токен маршрутам, которые его используют. Список без находок приходит как пустой массив (`[]`), а не как `{}`. Это текстовое сканирование и приближение; события блокировки в отчёте, записанные во время рендеринга, — надёжный сигнал. Стоимость растёт с числом страниц.

---

## Политика песочницы Twig {: #twig-content-sandbox-policy }

<span class="api-method api-method-get">GET</span> `/reports/twig-content/sandbox-policy`

Разбивка песочницы Twig для контента страниц только для чтения: для каждого списка — встроенные значения по умолчанию, собственные дополнения сайта `allowed_*`, его удаления `denied_*` и итоговый результат. Требуется `api.reports.read`.

**Ответ**

```json
{"data": {"enabled": true, "config_access": false, "lists": {"tags": {"defaults": ["if", "for", "set"], "added": [], "denied": [], "effective": ["if", "for", "set"]}, "filters": {"defaults": ["escape", "upper"], "added": ["markdown"], "denied": [], "effective": ["escape", "upper", "markdown"]}, "functions": {"defaults": ["range"], "added": [], "denied": [], "effective": ["range"]}, "methods": {"defaults": {"Grav\\Common\\Page\\Page": ["title", "url"]}, "added": [], "denied": [], "effective": {"Grav\\Common\\Page\\Page": ["title", "url"]}}, "properties": {"defaults": [], "added": [], "denied": [], "effective": []}}, "config_denied_paths": {"defaults": ["plugins", "streams", "security", "backups", "scheduler", "system.cache.redis.password", "system.debugger.token"], "added": [], "effective": ["plugins", "streams", "security", "backups", "scheduler", "system.cache.redis.password", "system.debugger.token"]}}}
```

**Коды ответов**

| Код | Описание |
| ---- | ----------- |
| `200` | Успех |
| `401` | Не авторизован |
| `403` | Нет разрешения `api.reports.read` |

Списки в примере сокращены. `enabled` — это `security.twig_sandbox.enabled`, а `config_access` — `security.twig_content.config_access`. В `methods` и `properties` каждая запись сопоставляет класс разрешённым именам его членов, а пустое отображение приходит как `[]`. `config_denied_paths` перечисляет пути конфигурации, скрытые от Twig в контенте. Политика строится только из конфигурации; дополнения, которые делают плагины через `onBuildTwigSandboxPolicy`, в неё не входят.
