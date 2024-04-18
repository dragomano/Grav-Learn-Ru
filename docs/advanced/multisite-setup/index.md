---
description: Как настроить несколько сайтов Grav CMS. Сниппеты и потоки. Расширенная конфигурация для экспертов.
---

# Настройка мультисайта

!!! warning ""

    В Grav есть поддержка нескольких сайтов. Однако плагин админки по-прежнему нуждается в обновлении для полной поддержки мультисайтовых конфигураций. Мы продолжим работать над этим в следующих выпусках Grav.

## Что такое мультисайтовая установка?

Мультисайтовая установка позволяет вам создавать и управлять сетью из нескольких веб-сайтов, работающих на одной установке.

Grav имеет встроенную поддержку нескольких сайтов. В отличие от [автоматической конфигурации среды](/advanced/environment-config), которая позволяет вам определять пользовательские среды для поддержки различных конфигураций и сценариев.

Полная мультисайтовая настройка дает вам возможность изменить способ, как и откуда Grav загружает все свои файлы.

## Требования к установке мультисайта Grav

Самое главное, что вам понадобится для работы в мультисайтовой сети Grav — это хороший хостинг веб-сайтов. Если вы не планируете создавать много сайтов и не ожидаете большого количества посетителей, то вы можете обойтись виртуальным хостингом. Однако из-за природы мультисайтов вам, вероятно, понадобится VPS или выделенный сервер по мере роста ваших сайтов.

## Настройка и установка

Перед тем, как начать, вы должны убедиться, что ваш веб-сервер поддерживает работу нескольких веб-сайтов, т. е. у вас есть доступ к корневому каталогу Grav.

Это важно, поскольку обслуживание нескольких веб-сайтов из одной и той же установки основано на файле `setup.php`, расположенном в вашем корне Grav.

### Быстрый старт (для начинающих)

После создания `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Чтобы обслуживать несколько веб-сайтов из одной установки, этот сценарий (грубо говоря) должен сообщать Grav, где находятся файлы (для конфигураций, тем, плагинов, страниц и т. д.) Для определенного дочернего сайта.

Приведенные ниже фрагменты настраивают вашу установку Grav таким образом, чтобы запрос вроде

```text
https://<subsite>.example.com   -->   user/sites/<subsite>.example.com
```

или

```text
https://example.com/<subsite>   -->   user/sites/<subsite>
```

будет использовать каталог `user/env` в качестве базового пути для «user» вместо каталога `user`.

Если вы выбираете подкаталоги или URL-адреса на основе пути для дочерних сайтов, то единственное, что вам нужно, это создать каталог для каждого дочернего сайта в каталоге `user/env`, содержащий как минимум необходимые папки `config`, `pages`, `plugins` и `themes`.

Если вы выберете субдомены для структурирования сети своего веб-сайта, вам придется настроить (подстановочные) субдомены на своем сервере в дополнение к настройке ваших дочерних сайтов в каталоге `user/env`.

В любом случае, решите, какая установка вам больше подходит.

#### Сниппеты

Для дочерних сайтов, доступных через поддомены, скопируйте файл `setup_subdomain.php`, в противном случае для дочерних сайтов, доступных через подкаталоги, файл `setup_subdirectory.php` в ваш `setup.php`.

!!! tip ""

    Файл `setup.php` должен быть помещен в корневую папку Grav, ту же папку, где вы можете найти `index.php`, `README.md` и другие файлы Grav.

```php title="setup_subdomain.php"
<?php
/**
 * Multisite setup for subsites accessible via sub-domains.
 *
 * DO NOT EDIT UNLESS YOU KNOW WHAT YOU ARE DOING!
 */

use Grav\Common\Utils;

// Get subsite name from sub-domain
$environment = isset($_SERVER['HTTP_HOST'])
    ? $_SERVER['HTTP_HOST']
    : (isset($_SERVER['SERVER_NAME']) ? $_SERVER['SERVER_NAME'] : 'localhost');
// Remove port from HTTP_HOST generated $environment
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

```php
<?php
/**
 * Multisite setup for sub-directories or path based
 * URLs for subsites.
 *
 * DO NOT EDIT UNLESS YOU KNOW WHAT YOU ARE DOING!
 */

use Grav\Common\Filesystem\Folder;

// Get relative path from Grav root.
$path = isset($_SERVER['PATH_INFO'])
   ? $_SERVER['PATH_INFO']
   : Folder::getRelativePath($_SERVER['REQUEST_URI'], ROOT_DIR);

// Extract name of subsite from path
$name = Folder::shift($path);
$folder = "env/{$name}";
$prefix = "/{$name}";

if (!$name || !is_dir(ROOT_DIR . "user/{$folder}")) {
    return [];
}

// Prefix all pages with the name of the subsite
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

При использовании подкаталогов для переключения языковых контекстов вам может потребоваться загружать различные конфигурации в зависимости от языка.
Вы можете поместить свои языковые конфигурации в `config/<lang-context>/site.yaml`, используя пример для `setup_subdir_config_switch.php` ниже.
Таким образом `yoursite.com/de-AT/index.html` будет загружать `config/de-AT/site.yaml`, `yoursite.com/de-CH/index.html` будет загружать `config/de-CH/site.yaml` и так далее.

**setup_subdir_config_switch.php**:

```php
<?php
/**
 * Switch config based on the language context subdir
 *
 * DO NOT EDIT UNLESS YOU KNOW WHAT YOU ARE DOING!
 */

use Grav\Common\Filesystem\Folder;

$languageContexts = [
    'de-AT',
    'de-CH',
    'de-DE',
];

// Get relative path from Grav root.
$path = isset($_SERVER['PATH_INFO'])
    ? $_SERVER['PATH_INFO']
    : Folder::getRelativePath($_SERVER['REQUEST_URI'], ROOT_DIR);

// Extract name of subdir from path
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

#### GPM (менеджер пакетов Grav) и несколько настроек

Если вам необходимо управлять плагинами и темами ваших подсайтов с помощью [GPM](https://learn.getgrav.org/17/cli-console/grav-cli-gpm), храните `user/themes` и `user/plugins`, чтобы GPM получал и обновлял их в одном месте. Затем создайте символические ссылки на `user/env/my.site.com/themes` или `user/env/my.site.com/plugins`. Затем настройте отдельные yaml-конфигурации `user/env/my.site.com/config/plugins` для каждого подсайта.

### Расширенная конфигурация (для экспертов)

После создания `setup.php` получите доступ к двум важным переменным: (i) `$container`, который является ещё не инициализированным должным образом [экземпляр Grav](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Grav.php) и (ii) `$self`, который является экземпляром [класса ConfigServiceProvider](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Service/ConfigServiceProvider.php).

Внутри этого скрипта вы можете делать что угодно, но имейте в виду, что файл `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Это означает, что критические для памяти или требующие много времени операции инициализации приводят к замедлению работы всей системы, и поэтому их следует избегать.

В конце концов, `setup.php` должен возвращать ассоциативный массив с необязательным именем среды **environment** и коллекцией потоков **streams** (подробнее и о том, как правильно их настроить, см. раздел [Потоки](#potoki)):

```php
<?php

return [
  'environment' => '<name>',            // Название окружения
  'streams' => [
    'schemes' => [
      '<stream_name>' => [              // Название потока
        'type' => 'ReadOnlyStream',     // Объект потока, например: 'ReadOnlyStream' или 'Stream'
        'prefixes' => [
          '<prefix>' => [
            '<path1>',
            '<path2>',
            '<etc>'
          ]
        ],
        'paths' => [                    // Пути (не обязательно)
          '<paths1>',
          '<paths2>',
          '<etc>'
        ]
      ]
    ]
  ]
]

```

!!! warning ""

    Имейте в виду, что на этом очень раннем этапе у вас нет доступа к конфигурации или экземпляру URI, и поэтому любой вызов неинициализированного класса может закончиться зависанием системы, неожиданными ошибками или (полной) потерей данных.

### Потоки

Grav использует URI-подобные потоки для определения всех путей к файлам в Grav. Использование потоков позволяет очень легко настроить пути поиска для любого файла.

По умолчанию потоки были настроены следующим образом:

- `user://` - папка пользователя, например `user/`
- `page://` - папка страниц, например `user://pages/`
- `image://` - папка изображений, например `user://images/`, `system://images/`
- `account://` - папка учётных записей, например `user://accounts/`
- `environment://` - текущее местоположение нескольких сайтов.
- `asset://` - папка скомпилированных активов JS/CSS, например `assets/`
- `blueprints://` - папка чертежей, например `environment://blueprints/`, `user://blueprints/`, `system://blueprints/`
- `config://` - папка конфигураций, например `environment://config/`, `user://config/`, `system://config/`
- `plugins://` - папка плагинов, например `user://plugins/`
- `themes://` - папка тем, например `user://themes/`
- `theme://` - папка текущей темы, например `themes://antimatter/`
- `languages://` - папка языков, например `environment://languages/`, `user://languages/`, `system://languages/`
- `user-data://` - папка данных, например `user/data/`
- `system://` - папка файлов движка, например `system/`
- `cache://` - папка кэша, например `cache/`, `images/`
- `log://` - папка логов, например `logs/`
- `backup://` - папка резервных копий, например `backups/`
- `tmp://` - папка временных файлов, например `tmp/`

При настройке нескольких сайтов некоторые из этих параметров по умолчанию могут быть не теми, которые вам нужны. Grav обеспечивает простой способ, чтобы настроить потоки от конфигурации среды, используя `config/streams.yaml`. Кроме того, вы можете создавать и использовать свои собственные потоки, когда это необходимо.

Сопоставление физических каталогов с логическим устройством может быть выполнено путем настройки префиксов. Вот пример, где мы отделяем страницы, изображения, учетные записи, данные, кэш и журналы от остальных сайтов, но заставляем все остальное искать в местоположении по умолчанию:

`user/env/domain.com/config/streams.yaml`:

```yaml
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

In Grav streams are objects, mapping a set of physical directories of the system to a logical device. They are classified via their `type` attribute. For read-only streams that's the `ReadOnlyStream` type and for read-writeable streams that's the `Stream` type.

For example, if you use `image://mountain.jpg` stream, Grav looks up `environment://images` (`user/env/domain.com/images`) and `system://images` (`system/images`). This means that streams can be used to define other streams.

Prefixes allows you to combine several physical paths into one logical stream. If you look carefully at `cache` stream definition, it is a bit different. In this case `cache://` resolves to `cache`, but `cache://images` resolves to `images`.

## Серверная мультисайтовая конфигурация

Grav 1.7 добавляет поддержку настройки начальной среды из конфигурации вашего сервера.

Эта функция удобна, если вы хотите использовать, например, контейнеры docker и хотите сделать их независимыми от домена, который вы используете. Или если вы не хотите хранить секреты в конфигурации, но хранить их в настройках Вашего сервера.

Следующие переменные среды можно использовать для настройки путей по умолчанию, которые Grav использует для настройки среды. После инициализации потоки могут указывать на другое местоположение.

!!! note ""

    Вы можете использовать либо переменные среды, либо константы PHP, но они должны быть установлены перед запуском Grav.

| Переменная           | Значение по умолчанию | Описание                                                                                                                                                      |
| -------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GRAV_SETUP_PATH**  | AUTO DETECT           | Пользовательский путь к `setup.php`, включая имя файла. По умолчанию Grav просматривает файл из `GRAV_ROOT/setup.php` и `GRAV_ROOT/GRAV_USER_PATH/setup.php`. |
| **GRAV_USER_PATH**   | `user`                | Относительный путь для потока `user://`.                                                                                                                      |
| **GRAV_CACHE_PATH**  | `cache`               | Относительный путь для потока `cache://`.                                                                                                                     |
| **GRAV_LOG_PATH**    | `logs`                | Относительный путь для потока `log://`.                                                                                                                       |
| **GRAV_TMP_PATH**    | `tmp`                 | Относительный путь для потока `tmp://`.                                                                                                                       |
| **GRAV_BACKUP_PATH** | `backup`              | Относительный путь для потока `backup://`.                                                                                                                    |

Кроме того, есть переменные для настройки окружения. Лучшую документацию по ним можно найти в разделе [Конфигурация среды на основе сервера](/advanced/environment-config/#konfiguratsiia-sredy-na-osnove-servera).

!!! note ""

    Они также работают из файла `setup.php`. Вы можете сделать их константами, используя `define()`, или переменные среды с `putenv()`. Константы предпочтительнее переменных среды.

| Переменная                 | Значение по умолчанию    | Описание                                                                                                                                                                   |
| -------------------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GRAV_ENVIRONMENT**       | DOMAIN NAME              | Имя среды. Может использоваться, например, в контейнерах докеров для установки настраиваемой среды, которая не зависит от имени домена, например `production` и `develop`. |
| **GRAV_ENVIRONMENTS_PATH** | `user://env`             | Путь поиска для всех сред, если вы предпочитаете что-то вроде `user://sites`. Может быть потоком или относительным путем от `GRAV_ROOT`.                                   |
| **GRAV_ENVIRONMENT_PATH**  | `user://env/ENVIRONMENT` | Иногда может быть полезно указать настраиваемое местоположение для вашей среды.                                                                                            |

### Переопределения конфигурации на основе сервера

Если вы не хотите хранить секретные учетные данные внутри конфигурации, вы также можете предоставить их, используя переменные среды с вашего сервера.

Поскольку переменные среды имеют строгие требования к именованию (они могут содержать только A-Z, a-z, 0-9 и \_), необходимы некоторые уловки, чтобы заставить работать переопределения конфигурации.

Вот пример простого переопределения конфигурации с использованием формата YAML для презентации:

```yaml
GRAV_CONFIG: true # If false, the configuration here will be ignored.
GRAV_CONFIG_ALIAS__GITHUB: plugins.github # Create alias GITHUB='plugins.github' to shorten the variable names below
GRAV_CONFIG__GITHUB__auth__method: api # Override config.plugins.github.auth.method = api
GRAV_CONFIG__GITHUB__auth__token: xxxxxxxx # Override config.plugins.github.auth.token = xxxxxxxx
```

В приведенном выше примере `__` (двойное подчеркивание) представляет вложенную переменную, которая в Twig представлена ​​с помощью точки (`.`).

Вы также можете использовать переменные среды в `setup.php`. Это позволяет, например, хранить секреты вне конфигурации:

`user/setup.php`:

```php
<?php

// Use following environment variables in your server configuration:
//
// DYNAMODB_SESSION_KEY: DynamoDb server key for the PHP session storage
// DYNAMODB_SESSION_SECRET: DynamoDb server secret
// DYNAMODB_SESSION_REGION: DynamoDb server region
// GOOGLE_MAPS_KEY: Google Maps secret key

return [
    'plugins' => [
        // This plugin does not exist
        'dynamodb_session' => [
            'credentials' => [
                'key' => getenv('DYNAMODB_SESSION_KEY') ?: null,
                'secret' => getenv('DYNAMODB_SESSION_SECRET') ?: null
            ],
            'region' => getenv('DYNAMODB_SESSION_REGION') ?: null
        ],
        // This plugin does not exist
        'google_maps' => [
            'key' => getenv('GOOGLE_MAPS_KEY') ?: null
        ]
    ]
];
```

!!! warning ""

    `setup.php` используется для установки начальной конфигурации. Если плагин или ваша конфигурация позже переопределят эти настройки, исходные значения будут потеряны.

После определения переменных в `setup.php` вы можете установить их на своем сервере:

=== "Apache 2"

````apacheconf
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

=== "NGINX php-fpm"
```nginx
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

=== "NGINX php-cgi"
```nginx
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

=== "Docker"
`yaml
    web:
    environment:
        - GRAV_SETUP_PATH=user/setup.php
        - GRAV_ENVIRONMENT=production
        - DYNAMODB_SESSION_KEY=JBGARDQ06UNJV00DL0R9
        - DYNAMODB_SESSION_SECRET=CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7
        - DYNAMODB_SESSION_REGION=us-east-1
        - GOOGLE_MAPS_KEY=XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp
    `

=== "PHP"
`php
    <?php
    putenv('GRAV_SETUP_PATH', 'user/setup.php');
    putenv('GRAV_ENVIRONMENT', 'production');
    putenv('DYNAMODB_SESSION_KEY', 'JBGARDQ06UNJV00DL0R9');
    putenv('DYNAMODB_SESSION_SECRET', 'CVjwH+QkfnPhKgVvJvrG24s0ABi343cJ7WTPxvb7');
    putenv('DYNAMODB_SESSION_REGION', 'us-east-1');
    putenv('GOOGLE_MAPS_KEY', 'XWIozB2R2GmYInTqZ6jnKuUrdELounUb4BIxYmp');
    `

В этом примере сервер также будет использовать среду `production`, хранящуюся в папке `user/env/production`.
````
