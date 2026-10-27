---
title: "Site Toolbox"
description: "Узнайте, как использовать плагины из Site Toolbox."
tags:
  - "Grav Premium"
  - "Site Toolbox"
  - "Grav 1.7"
---

# Site Toolbox

> [!IMPORTANT] Важно
> Продукты Premium требуют бесплатного плагина [License Manager](../license-manager/index.md). Установите его и добавьте лицензию продукта, прежде чем устанавливать этот продукт.

## Установка

Сначала убедитесь, что у вас установлена последняя версия **Grav 1.7** (`-f` принудительно обновляет индекс GPM).

```
$ bin/gpm selfupgrade -f
```

Эти плагины доступны через GPM, и установить их все сразу можно из CLI:

```
$ bin/gpm install svg-icons warm-cache zapier-rss mega-frontmatter
```

> [!IMPORTANT] Важно
> Плагин **Warm Cache** использует плагин **sitemap** для правильной работы, поэтому разрешите установку, когда появится запрос.

Установить тему также можно через раздел **Плагины** в плагине Admin.

## SVG Icons

Плагин **SVG Icons** — это расширение для [Grav CMS](http://github.com/getgrav/grav), которое предоставляет более 8000 SVG-иконок из различных наборов. Их можно использовать в контенте и шаблонах Twig с помощью уникального шорткода (для контента) или функции Twig (для шаблонов).

Сейчас этот пакет содержит 6 основных наборов SVG-иконок:

* [Tabler Icons (2664)](https://tabler-icons.io/) → [DEFAULT] Разработано [Csaba Kissi](https://twitter.com/csaba_kiss) (v1.95.1)
* [Hero Icons (292)](https://heroicons.dev/) → Разработано [Steve Schoger](https://twitter.com/steveschoger), есть варианты `outline` и `solid` (v2.0.11)
* [Simple Icon Brands (2340)](https://simpleicons.org/) → Более 2100 популярных брендовых иконок (v7.11.0)
* [Social Icons (6)](./) → Несколько базовых согласованных иконок социальных сетей (v1.0)
* **NEW** [Bootstrap Icons (1811)](https://icons.getbootstrap.com/) → Более 1500 иконок (v1.9.1)
* **NEW** [Iconsax icons (1792)](https://iconsax.io/) →  варианты `outline` и `bold` (v1.0)

#### Конфигурация

* **Статус плагина** → Включает или отключает весь плагин.

#### Шорткод для контента

Доступные наборы иконок:

* `tabler`
* `heroicons\solid`
* `heroicons\outline`
* `brands`
* `bootstrap`
* `iconsax\bold`
* `iconsax\outline`
* `social`

Когда нужно использовать SVG-иконку в контенте, можно применить шорткод `[svg-icon]`. Вот несколько примеров:

```bbcode
[raw]
[svg-icon="alien" /]
[/raw]
```

<kbd>[svg-icon="alien" /]</kbd>

Это самый быстрый и простой подход. Используется набор SVG-иконок `tabler` по умолчанию. Обратите внимание: расширение в имени иконки не требуется, так как все они — SVG.

```bbcode
[raw]
[svg-icon=award set="tabler" /]
[/raw]
```

<kbd>[svg-icon="award" set="tabler" /]</kbd>

Ещё один пример: набор указан явно, а завершающей косой черты нет.

```bbcode
[raw]
[svg-icon icon="atom" /]
[/raw]
```

<kbd>[svg-icon icon="atom" /]</kbd>

Более распространённый подход: иконка указана явно.

```bbcode
[raw]
[svg-icon icon="battery-4" set="tabler" /]
[/raw]
```

<kbd>[svg-icon icon="battery-4" set="tabler" /]</kbd>

Иконка и набор указаны, но завершающей косой черты нет.

```bbcode
[raw]
[svg-icon icon="clipboard-document-check" class="w-12" set="heroicons|solid" /]
[/raw]
```

<kbd>[svg-icon icon="clipboard-document-check" class="w-12" set="heroicons|solid" /]</kbd>

Пример из HeroIcons / Solid с классом TailwindCSS `w-12` для задания ширины.

```bbcode
[raw]
[svg-icon icon="shield-check" class="w-12 h-12 text-blue-600" /]
[/raw]
```

[svg-icon icon="grav" class="w-8" set="brands" /]
[svg-icon icon="apple" class="w-8" set="brands" /]
[svg-icon icon="ferrari" class="w-8" set="brands" /]
[svg-icon icon="spacex" class="w-8" set="brands" /]

Пример нескольких популярных брендов из Simple Icons.

```bbcode
[raw]
[svg-icon icon="grav" class="w-8" set="brands" /]
[svg-icon icon="apple" class="w-8" set="brands" /]
[svg-icon icon="ferrari" class="w-8" set="brands" /]
[svg-icon icon="spacex" class="w-8" set="brands" /]
[/raw]
```

<kbd>[svg-icon icon="shield-check" class="w-12 h-12 text-blue-600" /]</kbd>

Более сложный пример с классами TailwindCSS для ширины и высоты, а также для цвета.

```bbcode
[raw]
[svg-icon icon="shield-check" class="w-24 h-24 text-red-500 stroke-1 transform rotate-90" /]
[/raw]
```

<kbd>[svg-icon icon="shield-check" class="w-24 h-24 text-red-500 stroke-1 transform rotate-90" /]</kbd>

А здесь просто демонстрация: изменён векторный контур и добавлен свой поворот.

#### Функция Twig для шаблонов

Использовать плагин напрямую из шаблонов Twig немного иначе. Есть функция Twig `svg_icon()`, но она принимает только путь к SVG-иконке и классы. Примеры:

```twig
{{ svg_icon('tabler/plus.svg', 'h-6 w-6 text-gray-600 stroke-3/2')|raw }}
```

Здесь используется иконка `tabler/plus.svg` с различными классами TailwindCS для ширины, высоты, цвета и контура. Обратите внимание на фильтр `|raw` в конце. Он важен, потому что вывод — это необработанный встроенный HTML SVG.

```twig
{{ svg_icon('heroicons/outline/star.svg', 'current-color h-8 w-8')|raw }}
```

Здесь используется `star.svg` из HeroIcons в стиле Outline. Классы используют текущий цвет.

#### Свои иконки

Если вы хотите добавить свои иконки, их следует почистить: убрать любые жёстко заданные цвета и использовать вместо них `currentColor`. Например, найдите и замените:

```
stroke=\"#(?:[0-9a-fA-F]{3}){1,2}\"
stroke="currentColor"

fill=\"#(?:[0-9a-fA-F]{3}){1,2}\"
fill="currentColor"
```

## Mega-Frontmatter

Mega-Frontmatter позволяет хранить указанные поля метаданных страницы и загружать их из внешнего YAML-файла, что может повысить производительность, когда сайт использует большие объёмы данных, хранящихся в метаданных страницы.

Внешний файл метаданных хранится рядом с самим файлом страницы и не включается в метаданные страницы при её инициализации. Он загружается только при просмотре или редактировании страницы, что может заметно улучшить производительность.

#### Фичи

* Сохраняет определённые поля метаданных страницы во внешний файл
* Без швов загружает внешний файл и объединяет его с метаданными страницы при просмотре или редактировании страницы
* Полностью совместим с многоязычностью
* Автоматически создаёт файл метаданных, если его нет, или объединяет данные, если он есть
* Команда CLI для автоматического перебора всех страниц с извлечением метаданных во внешний файл или обратного возврата из отдельного файла в метаданные страницы

#### Конфигурация {: #configuration-1 }

* **Статус плагина** → Включает или отключает весь плагин.
* **Имя mega-файла** → Имя YAML-файла, в котором будут храниться ваши поля метаданных страницы рядом с файлом страницы `.md`. По умолчанию `mega-frontmatter.yaml`.
* **Поля mega** → Поля верхнего уровня из метаданных страницы, которые будут храниться в mega-файле. По умолчанию `field1, field2, field3, feild4`. Их можно изменить под свои нужды.

#### Использование

Проще всего объяснить использование на примере. Допустим, у вас есть страница такого вида:

```yaml
---
title: My Custom Page
published: true
taxonomy:
    category: [cat1, cat2]
    tag: [tag1, tag2]
hero:
    title: Hero Title
    subtitle: Hero Subtitle
    image: /images/hero.jpg
    content: Hero Content
features:
  - name: Feature 1
    type: foo
    list: [a, b, c]
    content: Aliquam erat volutpat. Donec volutpat diam quis finibus finibus. Etiam sodales finibus mauris eget feugiat. Sed egestas nibh a massa tincidunt finibus. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. In commodo lobortis sapien et facilisis. Cras mattis arcu in ipsum euismod scelerisque. Vestibulum consequat consectetur nibh at euismod. Praesent sed cursus ante, ac sagittis mauris. Maecenas id egestas justo, id ullamcorper lorem. Duis ornare fringilla volutpat. In massa est, sodales non cursus ut, gravida id diam.
  - name: Feature 3
    type: bar
    list: [d, e, f]
    content: Curabitur eu cursus enim. In sodales tempus tempor. Quisque auctor facilisis congue. Nunc et est sagittis, ullamcorper ligula a, dictum dolor. Proin sed tempor neque, scelerisque hendrerit nibh. Quisque pharetra, sem eu efficitur hendrerit, nisi erat tristique erat, at placerat eros purus a elit. Mauris ut facilisis quam. Ut risus tellus, volutpat a porta mattis, consectetur et magna. Curabitur congue aliquam mattis. Curabitur rhoncus interdum ipsum, eu laoreet velit egestas sit amet. In hac habitasse platea dictumst.
    ...
---

Your page content here...
```

Я сократил пример для краткости, но представьте, что у вас 30 таких блоков features, и из-за этого метаданные страницы становятся довольно большими. Если таких страниц много, вы можете обнаружить, что ваш сайт на Grav работает медленнее, чем ожидалось. Вероятно, это связано с большим объёмом метаданных, которые приходится обрабатывать для каждой страницы. Представьте, что можно указать определённые метаданные, которые сейчас обрабатываются для каждой страницы, но обрабатывались бы только тогда, когда это нужно. Именно это делает плагин.

Просто укажите поля `hero` и `features` в параметре конфигурации `mega_fields`: если они найдены в метаданных страницы, они будут перенесены во внешний mega-файл метаданных.

Если вы используете админку, это произойдёт автоматически при **редактировании** и **сохранении** страницы. Для вас это прозрачно: данные объединяются при загрузке и извлекаются при сохранении. После добавления полей `hero` и `features`, редактирования и сохранения страницы вы не заметите никакой разницы, но если заглянуть в папку страницы, вы найдёте файл `mega-frontmatter.yaml`. У вас останется markdown-файл страницы такого вида:

```yaml
---
title: My Custom Page
published: true
taxonomy:
    category: [cat1, cat2]
    tag: [tag1, tag2]
---

Your page content here...
```

и файл `mega-frontmatter.yaml` такого вида:

```yaml
hero:
    title: Hero Title
    subtitle: Hero Subtitle
    image: /images/hero.jpg
    content: Hero Content
features:
  - name: Feature 1
    type: foo
    list: [a, b, c]
    content: Aliquam erat volutpat. Donec volutpat diam quis finibus finibus. Etiam sodales finibus mauris eget feugiat. Sed egestas nibh a massa tincidunt finibus. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. In commodo lobortis sapien et facilisis. Cras mattis arcu in ipsum euismod scelerisque. Vestibulum consequat consectetur nibh at euismod. Praesent sed cursus ante, ac sagittis mauris. Maecenas id egestas justo, id ullamcorper lorem. Duis ornare fringilla volutpat. In massa est, sodales non cursus ut, gravida id diam.
  - name: Feature 3
    type: bar
    list: [d, e, f]
    content: Curabitur eu cursus enim. In sodales tempus tempor. Quisque auctor facilisis congue. Nunc et est sagittis, ullamcorper ligula a, dictum dolor. Proin sed tempor neque, scelerisque hendrerit nibh. Quisque pharetra, sem eu efficitur hendrerit, nisi erat tristique erat, at placerat eros purus a elit. Mauris ut facilisis quam. Ut risus tellus, volutpat a porta mattis, consectetur et magna. Curabitur congue aliquam mattis. Curabitur rhoncus interdum ipsum, eu laoreet velit egestas sit amet. In hac habitasse platea dictumst.
    ...
```

#### Команда CLI

> [!NOTE] Примечание
> Если вы используете команды CLI, убедитесь, что ваша конфигурация верна.

Доступны две команды. Первая использует ваши настройки конфигурации, перебирает все страницы сайта и **извлекает** настроенные заголовки метаданных страницы во внешний файл _mega_filename_.

```shell
bin/plugin mega-frontmatter morph extract
```

Вторая команда делает обратное: она перебирает все страницы и, если находит _mega_filename_, **вставляет** данные обратно в метаданные страницы:

```shell
bin/plugin mega-frontmatter morph insert
```

## Warm Cache

#### Конфигурация {: #configuration-2 }

`![](warm-cache.png?class=image-shadow)`

* **Статус плагина** → Включает или отключает весь плагин. Когда плагин отключён, значок на панели не отображается.

* **URL карты сайта** → По умолчанию используется значение, заданное в плагине **Sitemap**

* **Включить быструю панель** → Включает или отключает иконку на быстрой панели для запуска прогрева кэша сайта через Ajax.

* **Иконка быстрой панели** → Позволяет заменить иконку `tachometer` по умолчанию на другую иконку Fontawesome

* **События onCacheClear** → Если включено, **очистка кэша** через админку или CLI будет запускать прогрев кэша.

* **Показывать количество в результатах** → По умолчанию в результатах запуска показывается количество прогретых страниц. Это можно отключить.

* **Журналировать результаты** → Результаты очистки кэша можно записывать в журнал для отладки ошибок.

* **User Agent** → Это строка, которую задаёт сборщик, обращающийся к каждой странице. Её можно настроить или использовать, чтобы отфильтровать эти вызовы прогрева кэша из данных аналитики.

#### Использование {: #usage-1 }

##### Плагин Admin

Чтобы использовать плагин warm-cache в админке, просто нажмите иконку на быстрой панели. Иконку можно настроить, но по умолчанию это **тахометр**. После завершения в правом верхнем углу должно появиться всплывающее уведомление с сообщением.

`![Быстрый запуск](warm-cache-quicktray.png?class=image-shadow&style=max-width:300px;)`

##### Команда CLI {: #cli-command-1 }

Плагин warm-cache также включает полезную команду CLI, которую можно запускать из скрипта или использовать через задание cron либо планировщик по мере необходимости.

Чтобы использовать её из CLI, нужно вызвать команду и передать URL карты сайта. Например:

```shell
bin/plugin warm-cache warm https://my-site.example/sitemap.json
```

Что интересно, эту команду можно запустить из любого экземпляра Grav с установленным warm-cache и прогреть кэш любого другого сервера Grav с установленным плагином sitemap. Это потому, что warm-cache просто перебирает записи в карте сайта и заставляет Grav отвечать запрошенной страницей, тем самым заставляя Grav прогревать кэш для этой страницы.

##### Скрипт

Самый простой способ превратить процесс прогрева кэша в скрипт для сайта, который у нас находится в `/Useres/joe/workspace/grav-example-site`, — создать shell-скрипт, запускающий команду CLI. В этом примере мы создадим файл `/Users/joe/bin/warm-cache.sh` с содержимым:

```shell
#!/bin/sh
cd /Users/joe/workspace/grav-example-site
bin/plugin warm-cache warm https://localhost/grav-example-site/sitemap.json
```

Затем убедимся, что `warm-cache.sh` можно выполнять, с помощью:

```shell
chmod +x warm-cache.sh
```

Если папка `bin/` есть в пути у Джо, то в любое время, находясь в CLI, мы можем просто выполнить:

```shell
warm-cache.sh
```

##### Задание cron

Когда скрипт `warm-cache.sh` создан, его можно запускать ежедневно через crontab. Отредактируйте файл crontab с помощью `crontab -e` и добавьте следующую строку:

```crontab
* 3 * * * /Users/joe/bin/warm-cache.sh
```

##### Планировщик Grav

Чтобы использовать планировщик Grav, простой подход — сначала создать скрипт `warm-cache.sh`, как показано выше, а затем настроить его в планировщике в разделе «Custom Schedular Jobs».

Сначала убедитесь, что планировщик работает, затем можно добавить пользовательскую запись в файл конфигурации `user/config/scheduler.yaml`:

```yaml
...
custom_jobs:
  warm-cache:
    command: /Users/joe/bin/warm-cache.sh
    args: null
    at: '* 3 * * *'
    output: logs/warm-cache.log
    output_mode: append
    email: null
```

или через админку:

`![Планировщик Warm Cache](warm-cache-scheduler.png?class=image-shadow)`

Это будет запускать скрипт warm-cache ежедневно в 3 часа ночи и сохранять вывод в файл `logs/warm-cache.log` Grav, каждый раз дописывая вывод в конец.

## Zapier RSS

#### Конфигурация {: #configuration-3 }

Прежде чем настраивать этот плагин, скопируйте `user/plugins/zapier-rss/zapier-rss.yaml` в `user/config/plugins/zapier-rss.yaml` и редактируйте только эту копию.

Ниже приведена конфигурация по умолчанию и пояснения к доступным параметрам:

```yaml
enabled: true                                                 # плагин включён
active: false                                                 # состояние zapier по умолчанию
template: zapier-feed.zrss.twig                               # используемый шаблон TWIG
description: For Zapier RSS integration with other services   # Описание
limit: 20                                                     # Количество элементов для включения
length: 1000                                                  # Количество символов на элемент
order:                                                        # Порядок элементов
  by: date
  dir: desc
```

Учтите: если вы используете плагин Admin, файл с вашей конфигурацией с именем zapier-rss.yaml будет сохранён в папке `user/config/plugins/` после сохранения конфигурации в админке.

#### Использование {: #usage-2 }

Просто установите плагин, и URL, который вы будете использовать для интеграции, — это существующая страница блога с добавленным расширением `.zrss`:

```
http://my-site.example/your/blog.zrss
```

Это гарантирует, что для обработки ленты используется пользовательский файл `templates/zapier-feed.zrss.twig`. Чтобы изменить этот файл Twig, просто скопируйте его в папку `templates/` вашей темы или плагина и измените по своему усмотрению.

Вы можете переопределить обычные свойства коллекции или любые другие параметры конфигурации на уровне блога, изменив метаданные страницы, содержащей коллекцию. Например:

```yaml
---
title: 'My Blog'
header_image: custom_header.jpg
content:
    items:
        - self@.children
    order:
        by: date
        dir: desc
    limit: 8

zapier-rss:
    active: true
    description: 'My Custom Zapier ZRSS Feed'
    template: 'zapier-custom-feed.rss.twig'
    limit: 20
    length: 500
---
```

Также можно управлять тем, попадает ли конкретный элемент коллекции в эту ленту, пропустив его:

```yaml
---
title: 'My Blog Post"

zapier-rss:
    skip: true
---
```
