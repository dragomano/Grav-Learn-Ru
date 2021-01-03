---
title: Настройка мультисайта
taxonomy:
    category: docs
---

!! Предварительная поддержка мультисайтов теперь доступна в Grav 1.0. Однако команды интерфейса командной строки, а также подключаемый модуль администратора по-прежнему нуждаются в обновлении для полной поддержки многосайтовых конфигураций. Мы продолжим работать над этим в следующих выпусках Grav.

### Что такое многосайтовая установка?

> Многосайтовая установка позволяет вам создавать и управлять сетью из нескольких веб-сайтов, работающих на одной установке.

Grav имеет встроенную поддержку нескольких сайтов. В отличие от [автоматической конфигурации среды](../environment-config), которая позволяет вам определять пользовательские среды для поддержки различных конфигураций и сценариев, многосайтовая настройка дает вам возможность изменять способ и откуда Grav загружает все свои файлы.

### Требования к установке Grav Multisite

Самое главное, что вам понадобится для работы в многосайтовой сети Grav - это хороший хостинг веб-сайтов. Если вы не планируете создавать много сайтов и не ожидаете большого количества посетителей, то вы можете обойтись виртуальным хостингом. Однако из-за природы мультисайтов вам, вероятно, понадобится VPS или выделенный сервер по мере роста ваших сайтов.

### Настройка и установка

Перед тем, как начать, вы должны убедиться, что ваш веб-сервер поддерживает работу нескольких веб-сайтов, т. е. у вас есть доступ к корневому каталогу Grav.

Это важно, поскольку обслуживание нескольких веб-сайтов из одной и той же установки основано на файле `setup.php`, расположенном в вашем корне Grav.

#### Быстрый старт (для начинающих)

После создания `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Чтобы обслуживать несколько веб-сайтов из одной установки, этот сценарий (грубо говоря) должен сообщать Grav, где находятся файлы (для конфигураций, тем, плагинов, страниц и т. д.) Для определенного дочернего сайта.

Приведенные ниже фрагменты настраивают вашу установку Grav таким образом, чтобы запрос вроде

[prism classes="language-text"]
http://<subsite>.example.com   -->   user/sites/<subsite>.example.com
[/prism]
or
[prism classes="language-text"]
http://example.com/<subsite>   -->   user/sites/<subsite>
[/prism]

будет использовать каталог `user/sites` в качестве базового пути для "user" вместо каталога `user`.

Если вы выбираете подкаталоги или URL-адреса на основе пути для дочерних сайтов, то единственное, что вам нужно, это создать каталог для каждого дочернего сайта в каталоге `user/sites`, содержащий как минимум необходимые папки `config`, `pages`, `plugins` и `themes`.

Если вы выберете субдомены для структурирования сети своего веб-сайта, вам придется настроить (подстановочные) субдомены на своем сервере в дополнение к настройке ваших дочерних сайтов в каталоге `user/sites`.

В любом случае, решите, какая установка вам больше подходит.

##### Фрагменты

Для дочерних сайтов, доступных через поддомены, скопируйте файл `setup_subdomain.php`, в противном случае для дочерних сайтов, доступных через подкаталоги, файл `setup_subdirectory.php` в ваш `setup.php`.

!!! Файл `setup.php` должен быть помещен в корневую папку Grav, ту же папку, где вы можете найти `index.php`, `README.md` и другие файлы Grav.

**setup_subdomain.php**:
[prism classes="language-php line-numbers"]
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
$folder = "sites/{$environment}";

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
[/prism]

**setup_subdirectory.php**:
[prism classes="language-php line-numbers"]
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
$folder = "sites/{$name}";
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
[/prism]

#### Расширенная конфигурация (для экспертов)

После создания `setup.php` получите доступ к двум важным переменным: (i) `$container`, который является еще не инициализированным должным образом [экземпляр Grav](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Grav.php) и (ii) `$self`, который является экземпляром [класса ConfigServiceProvider](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Service/ConfigServiceProvider.php).

Внутри этого скрипта вы можете делать что угодно, но имейте в виду, что файл `setup.php` вызывается каждый раз, когда пользователь запрашивает страницу. Это означает, что критические для памяти или требующие много времени операции инициализации приводят к замедлению работы всей системы, и поэтому их следует избегать.

В конце концов, setup.php должен возвращать ассоциативный массив с необязательным именем среды **environment** и коллекцией потоков **streams**.
(подробнее и о том, как правильно их настроить, см. раздел [Потоки](#streams)):

[prism classes="language-php line-numbers"]
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

[/prism]

!!!! Имейте в виду, что на этом очень раннем этапе у вас нет доступа к конфигурации или экземпляру URI, и поэтому любой вызов неинициализированного класса может закончиться зависанием системы, неожиданными ошибками или (полной) потерей данных.

#### Потоки

В Grav потоки - это объекты, отображающие набор физических каталогов системы на логическое устройство. Они классифицируются с помощью атрибута `type`. Для потоков, доступных только для чтения, это тип ReadOnlyStream, а для потоков с возможностью чтения и записи - это тип Stream. Вы можете зарегистрировать любой настраиваемый тип потока и указывать на него, если он является экземпляром класса интерфейса [StreamInterface](https://github.com/rockettheme/toolbox/blob/develop/StreamWrapper/src/StreamInterface.php).

Сопоставление физических каталогов с логическим устройством может быть выполнено двумя способами: путем установки «путей» или «префиксов». Первый можно понимать как сопоставление 1 к 1, тогда как последний (как следует из названия) позволяет объединить несколько физических путей в один логический поток. Допустим, вы хотите зарегистрировать поток с именем «изображение». Затем вы можете с потоком `images://` list с

[prism classes="language-php line-numbers"]
'image' => [
    'type' => 'ReadOnlyStream',
    'paths' => [
        'user/images',
        'system/images'
    ]
];
[/prism]

все изображения находятся в папках `user/images` и `system/images`. Для **префиксов** рассмотрим пример

[prism classes="language-php line-numbers"]
'cache' => [
    'type' => 'Stream',
    'prefixes' => [
        '' => ['cache'],
        'images' => ['images']
    ]
];
[/prism]

В этом случае `cache://` преобразуется в `cache`, а `cache://images` преобразуется в `images`.

И последнее, но не менее важное: потоки можно использовать в других потоках. Например, при наличии потока «пользователь» и потока «система» вышеуказанный поток «изображений» также может быть записан как

[prism classes="language-php line-numbers"]
'image' => [
    'type' => 'ReadOnlyStream',
    'paths' => [
        'user://images',
        'system://images'
    ]
];

[/prism]
