---
title: "Настройка мультисайта"
description: "Мультисайт позволяет создать сеть из нескольких сайтов и управлять ею, причём все они работают на одной установке."
tags:
  - "Для продвинутых"
  - "Grav 1.7"
  - "Мультисайт"
---

# Настройка мультисайта

> [!CAUTION] Внимание
> В Grav есть предварительная поддержка мультисайта. Однако плагин Admin всё ещё нужно обновить, чтобы он полностью поддерживал конфигурации мультисайта. Мы продолжим работать над этим в следующих версиях Grav.

### Что такое мультисайт?

Мультисайт позволяет создать сеть из нескольких сайтов и управлять ею, причём все они работают на одной установке.

В Grav есть встроенная поддержка мультисайта. Эта возможность расширяет [базовую конфигурацию окружения](../environment-config/index.md), которая позволяет определять пользовательские окружения для продакшен- и dev-сайтов.

Полноценный мультисайт даёт вам власть над тем, как и откуда Grav загружает все свои файлы.

### Требования для мультисайта Grav

Самое важное, что понадобится для работы сети мультисайта Grav, — хороший веб-хостинг. Если вы не планируете создавать много сайтов и не ожидаете много посетителей, можно обойтись общим хостингом. Однако из-за особенностей мультисайтов по мере роста сайтов вам, скорее всего, понадобится VPS или выделенный сервер.

### Настройка и установка

Прежде чем начать, убедитесь, что ваш веб-сервер способен обслуживать несколько сайтов, то есть у вас есть доступ к корневой папке Grav.

Это необходимо, поскольку обслуживание нескольких сайтов с одной установки основано на файле `setup.php`, расположенном в корне Grav.

#### Быстрый старт (для начинающих)

После создания `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Чтобы обслуживать несколько сайтов с одной установки, этот скрипт (грубо говоря) должен сообщить Grav, где находятся файлы (конфигурации, темы, плагины, страницы и т. д.) для конкретного подсайта.

Приведённые ниже фрагменты настраивают вашу установку Grav так, что запрос вида

```text
https://<subsite>.example.com   -->   user/env/<subsite>.example.com
```

или

```text
https://example.com/<subsite>   -->   user/env/<subsite>
```

будет использовать папку `user/env` как базовый путь «user» вместо папки `user`.

Если вы выбираете подпапки или URL на основе путей для подсайтов, то всё, что нужно, — создать папку для каждого подсайта в папке `user/env`, содержащую как минимум обязательные папки `config`, `pages`, `plugins` и `themes`.

Если вы выбираете поддомены для построения сети сайтов, то помимо настройки подсайтов в папке `user/env` вам придётся настроить на сервере (wildcard-)поддомены.

В любом случае решите, какой вариант настройки подходит вам больше.

> [!NOTE] Примечание
> Серверные правила Grav, защищающие `config`, `accounts` и `data`, действуют и внутри каждой папки `user/env/<name>/`, а темы, плагины и ресурсы подсайта остаются публично доступными. Сайты на Apache получают эти правила в `.htaccess` при обновлении до Grav 2.2.4. На Nginx, Caddy, lighttpd и IIS конфигурацию сервера нужно обновить самостоятельно, поскольку эти серверы не читают `.htaccess`. Старые правила блокировали всё внутри `user/env`, поэтому на сервере, который всё ещё использует их, файлы темы, плагинов и ресурсов подсайта возвращают 403.

На Nginx строки с учётом `user/env` выглядят так:

```nginx
location ~* ^/user/(env/[^/]+/)?config/.*$ { return 403; }
location ~* ^/user/(env/[^/]+/)?accounts/[^/]+/[^/]+\.(jpe?g|png|gif|webp|avif|bmp|ico)$ { try_files $uri =404; }
location ~* ^/user/(env/[^/]+/)?accounts/.*$ { return 403; }
location ~* ^/user/(env/[^/]+/)?data/.*\.(jpe?g|png|gif|webp|avif|bmp|ico|mp4|webm|ogg|ogv|mov|mp3|wav|m4a|flac|pdf)$ { try_files $uri =404; }
location ~* ^/user/(env/[^/]+/)?data/.*$ { return 403; }
```

Полные правила для каждого сервера приведены в разделе [Доступность папки user](../../security/user-folder-exposure/index.md).

##### Фрагменты

Для подсайтов, доступных по поддоменам, скопируйте файл `setup_subdomain.php`, а для подсайтов, доступных по подпапкам, — файл `setup_subdirectory.php` в ваш `setup.php`.

> [!NOTE] Примечание
> Файл `setup.php` нужно положить в корневую папку Grav — ту же папку, где находятся `index.php`, `README.md` и остальные файлы Grav.

**setup_subdomain.php**:

```php linenums="1"
<?php
/**
 * Настройка мультисайта для подсайтов, доступных через поддомены.
 *
 * НЕ РЕДАКТИРУЙТЕ, ЕСЛИ НЕ УВЕРЕНЫ, ЧТО ДЕЛАЕТЕ!
 */

use Grav\Common\Utils;

// Получаем имя подсайта из поддомена
$environment = isset($_SERVER['HTTP_HOST'])
    ? $_SERVER['HTTP_HOST']
    : (isset($_SERVER['SERVER_NAME']) ? $_SERVER['SERVER_NAME'] : 'localhost');
// Убираем порт из $environment, полученного из HTTP_HOST
$environment = strtolower(Utils::substrToString($environment, ':'));
$folder = "env/{$environment}";

if ($environment === 'localhost' || !is_dir(ROOT_DIR . "user/{$folder}")) {
    return [];
}

return [
    'environment' => $environment,
    'streams' => [
        'schemes' => [
            'user' => [
               'type' => 'ReadOnlyStream',
               'prefixes' => [
                   '' => ["user/{$folder}"],
               ]
            ]
        ]
    ]
];
```

**setup_subdirectory.php**:

```php linenums="1"
<?php
/**
 * Настройка мультисайта для подсайтов, доступных через
 * подпапки или URL на основе пути.
 *
 * НЕ РЕДАКТИРУЙТЕ, ЕСЛИ НЕ УВЕРЕНЫ, ЧТО ДЕЛАЕТЕ!
 */

use Grav\Common\Filesystem\Folder;

// Получаем относительный путь от корня Grav.
$path = isset($_SERVER['PATH_INFO'])
   ? $_SERVER['PATH_INFO']
   : Folder::getRelativePath($_SERVER['REQUEST_URI'], ROOT_DIR);

// Извлекаем имя подсайта из пути
$name = Folder::shift($path);
$folder = "env/{$name}";
$prefix = "/{$name}";

if (!$name || !is_dir(ROOT_DIR . "user/{$folder}")) {
    return [];
}

// Добавляем имя подсайта как префикс ко всем страницам
$container['pages']->base($prefix);

return [
    'environment' => $name,
    'streams' => [
        'schemes' => [
            'user' => [
               'type' => 'ReadOnlyStream',
               'prefixes' => [
                   '' => ["user/{$folder}"],
               ]
            ]
        ]
    ]
];
```

При использовании подпапок для переключения языковых контекстов может понадобиться загружать разные конфигурации в зависимости от языка.
Вы можете разместить конфигурации для конкретного языка в `config/<lang-context>/site.yaml`, как показано в примере `setup_subdir_config_switch.php` ниже.
Так `example.com/de-AT/index.html` загрузит `config/de-AT/site.yaml`, `example.com/de-CH/index.html` загрузит `config/de-CH/site.yaml` и так далее.

**setup_subdir_config_switch.php**:

```php linenums="1"
<?php
/**
 * Переключение конфигурации в зависимости от подпапки языкового контекста
 *
 * НЕ РЕДАКТИРУЙТЕ, ЕСЛИ НЕ УВЕРЕНЫ, ЧТО ДЕЛАЕТЕ!
 */

use Grav\Common\Filesystem\Folder;

$languageContexts = [
    'de-AT',
    'de-CH',
    'de-DE',
];

// Получаем относительный путь от корня Grav.
$path = isset($_SERVER['PATH_INFO'])
    ? $_SERVER['PATH_INFO']
    : Folder::getRelativePath($_SERVER['REQUEST_URI'], ROOT_DIR);

// Извлекаем имя подпапки из пути
$name = Folder::shift($path);

if (in_array($name, $languageContexts)) {
    return [
        'streams' => [
            'schemes' => [
                'config' => [
                    'type' => 'ReadOnlyStream',
                    'prefixes' => [
                        '' => [
                            'environment://config',
                            'user://config/' . $name,
                            'user://config',
                            'system/config',
                        ],
                    ],
                ],
            ],
        ],
    ];
}

return [];
```

##### GPM (Grav Package Manager) и несколько установок
Если вам нужно управлять плагинами и темами подсайтов с помощью [GPM](https://learn.getgrav.org/17/cli-console/grav-cli-gpm),
держите и `user/themes`, и `user/plugins`, чтобы GPM скачивал и обновлял их в одном месте. Затем создайте символические ссылки на нужные элементы в `user/env/my.site.com/themes` или `user/env/my.site.com/plugins`. После этого настройте отдельные yaml-конфигурации `user/env/my.site.com/config/plugins` для каждого подсайта.

#### Продвинутая конфигурация (для экспертов)

После создания `setup.php` вам доступны две важные переменные: (i) `$container` — ещё не до конца инициализированный [экземпляр Grav](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Grav.php), и (ii) `$self` — экземпляр [класса ConfigServiceProvider](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Service/ConfigServiceProvider.php).

Внутри этого скрипта можно делать что угодно, но учтите: `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Это значит, что критичные к памяти или долгие операции инициализации замедляют всю систему, и их следует избегать.

В конце `setup.php` должен вернуть ассоциативный массив с необязательным именем окружения **environment** и коллекцией потоков **streams**
(подробнее и о том, как настроить их правильно, см. раздел [Потоки](#streams)):

```php linenums="1"
return [
  'environment' => '<name>',            // имя окружения
  'streams' => [
    'schemes' => [
      '<stream_name>' => [              // имя потока
        'type' => 'ReadOnlyStream',     // объект потока, например 'ReadOnlyStream' или 'Stream'
        'prefixes' => [
          '<prefix>' => [
            '<path1>',
            '<path2>',
            '<etc>'
          ]
        ],
        'paths' => [                    // пути (необязательно)
          '<paths1>',
          '<paths2>',
          '<etc>'
        ]
      ]
    ]
  ]
]
```

> [!TIP] Совет
> Учтите, что на этом очень раннем этапе у вас нет доступа ни к конфигурации, ни к экземпляру URI, поэтому любой вызов неинициализированного класса может привести к зависанию системы, неожиданным ошибкам или (полной) потере данных.

#### Потоки {: #streams }

Grav использует потоки вида URI для определения всех путей к файлам. Потоки сильно упрощают настройку путей поиска для любого файла.

По умолчанию потоки настроены так:

* `user://` — папка user. Например, `user/`
* `page://` — папка pages. Например, `user://pages/`
* `image://` — папка images. Например, `user://images/`, `system://images/`
* `account://` — папка accounts. Например, `user://accounts/`
* `environment://` — текущее расположение мультисайта.
* `asset://` — папка скомпилированных JS/CSS. Например, `assets/`
* `blueprints://` — папка blueprints. Например, `environment://blueprints/`, `user://blueprints/`, `system://blueprints/`
* `config://` — папка конфигурации. Например, `environment://config/`, `user://config/`, `system://config/`
* `plugins://` — папка plugins. Например, `user://plugins/`
* `themes://` — текущая тема. Например, `user://themes/`
* `theme://` — текущая тема. Например, `themes://antimatter/`
* `languages://` — папка languages. Например, `environment://languages/`, `user://languages/`, `system://languages/`
* `user-data://` — папка data. Например, `user/data/`
* `system://` — системная папка. Например, `system/`
* `cache://` — папка кэша. Например, `cache/`, `images/`
* `log://` — папка журналов. Например, `logs/`
* `backup://` — папка резервных копий. Например, `backups/`
* `tmp://` — временная папка. Например, `tmp/`

В конфигурациях мультисайта некоторые из этих настроек по умолчанию могут вам не подходить. Grav даёт простой способ настраивать потоки из конфигурации окружения с помощью `config/streams.yaml`. Кроме того, при необходимости можно создавать и использовать собственные потоки.

Сопоставить физические папки логическому устройству можно, настроив `prefixes`. Вот пример, в котором мы отделяем страницы, изображения, учётные записи, данные, кэш и журналы от остальных частей сайтов, а всё остальное ищется в местах по умолчанию:

`user/env/domain.com/config/streams.yaml`:

```yaml linenums="1"
schemes:
  account:
    type: ReadOnlyStream
    prefixes:
      '': ['environment://accounts']
  page:
    type: ReadOnlyStream
    prefixes:
      '': ['environment://user']
  image:
    type: Stream
    prefixes:
      '': ['environment://images', 'system://images/']
  'user-data':
    type: Stream
    prefixes:
      '': ['environment://data']
  cache:
    type: Stream
    prefixes:
      '': ['cache/domain.com']
      images: ['images/domain.com']
  log:
    type: Stream
    prefixes:
      '': ['logs/domain.com']
```

В Grav потоки — это объекты, сопоставляющие набор физических папок системы логическому устройству. Они классифицируются по атрибуту `type`. Для потоков только для чтения это тип `ReadOnlyStream`, а для потоков с чтением и записью — `Stream`.

Например, если вы используете поток `image://mountain.jpg`, Grav ищет в `environment://images` (`user/env/domain.com/images`) и `system://images` (`system/images`). Это значит, что потоки можно использовать для определения других потоков.

Префиксы позволяют объединить несколько физических путей в один логический поток. Если внимательно посмотреть на определение потока `cache`, оно немного отличается. В этом случае `cache://` разрешается в `cache`, а `cache://images` — в `images`.

### Конфигурация мультисайта на основе сервера {: #server-based-multi-site-configuration }

Grav 1.7 добавляет поддержку настройки начального окружения из конфигурации сервера.

Эта фича пригодится, если вы хотите, например, использовать docker-контейнеры и сделать их независимыми от домена, который вам доведётся использовать. Или если вы не хотите хранить секреты в конфигурации, а хотите хранить их в настройках сервера.

Следующие переменные окружения позволяют настроить пути по умолчанию, которые Grav использует для настройки окружения. После инициализации потоки могут указывать на другое место.

> [!NOTE] Примечание
> **Примечание:** можно использовать переменные окружения или PHP-константы, но задать их нужно до запуска Grav.

| Переменная | По умолчанию | Описание |
| -------- | ------- | ----------- |
| **GRAV_SETUP_PATH** | АВТООПРЕДЕЛЕНИЕ | Пользовательский путь к файлу `setup.php` вместе с именем файла. По умолчанию Grav ищет файл в `GRAV_ROOT/setup.php` и `GRAV_ROOT/GRAV_USER_PATH/setup.php`. |
| **GRAV_USER_PATH** | `user` | Относительный путь для потока `user://`. |
| **GRAV_CACHE_PATH** | `cache` | Относительный путь для потока `cache://`. |
| **GRAV_LOG_PATH** | `logs` | Относительный путь для потока `log://`. |
| **GRAV_TMP_PATH** | `tmp` | Относительный путь для потока `tmp://`. |
| **GRAV_BACKUP_PATH** | `backup` | Относительный путь для потока `backup://`. |

Кроме того, есть переменные для настройки окружений. Более подробная документация о них приведена в разделе [Конфигурация окружения на основе сервера](../environment-config/index.md#server-based-environment-configuration).

> [!NOTE] Примечание
> **Примечание:** они также работают из файла `setup.php`. Их можно сделать константами с помощью `define()` или переменными окружения с помощью `putenv()`. Константы предпочтительнее переменных окружения.

| Переменная | По умолчанию | Описание |
| -------- | ------- | ----------- |
| **GRAV_ENVIRONMENT** | ИМЯ ДОМЕНА | Имя окружения. Можно использовать, например, в docker-контейнерах, чтобы задать пользовательское окружение, не зависящее от имени домена, например `production` и `develop`. |
| **GRAV_ENVIRONMENTS_PATH** | `user://env` | Путь поиска для всех окружений, если вы предпочитаете что-то вроде `user://sites`. Может быть потоком или относительным путём от `GRAV_ROOT`. |
| **GRAV_ENVIRONMENT_PATH** | `user://env/ENVIRONMENT` | Иногда бывает полезно задать отдельное расположение для вашего окружения. |

#### Переопределения конфигурации на основе сервера

Если вы не хотите хранить секретные учётные данные в конфигурации, их также можно передать через переменные окружения с вашего сервера.

Поскольку к именам переменных окружения предъявляются строгие требования (они могут содержать только A-Z, a-z, 0-9 и _), чтобы переопределения конфигурации заработали, нужны некоторые ухищрения.

Вот пример простого переопределения конфигурации в формате YAML для наглядности:

```yaml
GRAV_CONFIG: true                           # если false, конфигурация ниже будет проигнорирована.

GRAV_CONFIG_ALIAS__GITHUB: plugins.github   # создаёт псевдоним GITHUB='plugins.github', чтобы сократить имена переменных ниже

GRAV_CONFIG__GITHUB__auth__method: api      # переопределяет config.plugins.github.auth.method = api
GRAV_CONFIG__GITHUB__auth__token: xxxxxxxx  # переопределяет config.plugins.github.auth.token = xxxxxxxx
```

В приведённом примере `__` (двойное подчёркивание) обозначает вложенную переменную, которая в twig представляется точкой `.`.

Переменные окружения можно также использовать в `setup.php`. Это позволяет, например, хранить секреты вне конфигурации:

`user/setup.php`:
```php
<?php

// Используйте следующие переменные окружения в конфигурации сервера:
//
// DYNAMODB_SESSION_KEY: ключ сервера DynamoDb для хранения сессий PHP
// DYNAMODB_SESSION_SECRET: секрет сервера DynamoDb
// DYNAMODB_SESSION_REGION: регион сервера DynamoDb
// GOOGLE_MAPS_KEY: секретный ключ Google Maps

return [
    'plugins' => [
        // Этот плагин не существует
        'dynamodb_session' => [
            'credentials' => [
                'key' => getenv('DYNAMODB_SESSION_KEY') ?: null,
                'secret' => getenv('DYNAMODB_SESSION_SECRET') ?: null
            ],
            'region' => getenv('DYNAMODB_SESSION_REGION') ?: null
        ],
        // Этот плагин не существует
        'google_maps' => [
            'key' => getenv('GOOGLE_MAPS_KEY') ?: null
        ]
    ]
];
```

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** `setup.php` используется для задания начальной конфигурации. Если плагин или ваша конфигурация позже переопределят эти настройки, начальные значения будут потеряны.

Определив переменные в `setup.php`, их можно задать на сервере:

```apache title="Apache 2" linenums="1"
<VirtualHost 127.0.0.1:80>
    ...

    SetEnv GRAV_SETUP_PATH         user/setup.php
    SetEnv GRAV_ENVIRONMENT        production
    SetEnv DYNAMODB_SESSION_KEY    JBGARDQ06UNJV00DL0R9
    SetEnv DYNAMODB_SESSION_SECRET CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7
    SetEnv DYNAMODB_SESSION_REGION us-east-1
    SetEnv GOOGLE_MAPS_KEY         XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp
</VirtualHost>
```

```nginx title="NGINX php-fpm" linenums="1"
location / {
    ...

    fastcgi_param GRAV_SETUP_PATH         user/setup.php;
    fastcgi_param GRAV_ENVIRONMENT        production;
    fastcgi_param DYNAMODB_SESSION_KEY    JBGARDQ06UNJV00DL0R9;
    fastcgi_param DYNAMODB_SESSION_SECRET CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7;
    fastcgi_param DYNAMODB_SESSION_REGION us-east-1;
    fastcgi_param GOOGLE_MAPS_KEY         XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp;
}
```

```nginx title="NGINX php-cgi" linenums="1"
location / {
...

    env[GRAV_SETUP_PATH]          = user/setup.php
    env[GRAV_ENVIRONMENT]         = production
    env[DYNAMODB_SESSION_KEY]     = JBGARDQ06UNJV00DL0R9
    env[DYNAMODB_SESSION_SECRET]  = CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7
    env[GDYNAMODB_SESSION_REGION] = us-east-1
    env[GGOOGLE_MAPS_KEY]         = XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp
}
```

```yaml title="Docker" linenums="1"
web:
  environment:
    - GRAV_SETUP_PATH=user/setup.php
    - GRAV_ENVIRONMENT=production
    - DYNAMODB_SESSION_KEY=JBGARDQ06UNJV00DL0R9
    - DYNAMODB_SESSION_SECRET=CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7
    - DYNAMODB_SESSION_REGION=us-east-1
    - GOOGLE_MAPS_KEY=XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp
```

```php title="PHP" linenums="1"
putenv('GRAV_SETUP_PATH', 'user/setup.php');
putenv('GRAV_ENVIRONMENT', 'production');
putenv('DYNAMODB_SESSION_KEY', 'JBGARDQ06UNJV00DL0R9');
putenv('DYNAMODB_SESSION_SECRET', 'CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7');
putenv('DYNAMODB_SESSION_REGION', 'us-east-1');
putenv('GOOGLE_MAPS_KEY', 'XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp');
```

В этом примере сервер также будет использовать окружение `production`, хранящееся в папке `user/env/production`.
