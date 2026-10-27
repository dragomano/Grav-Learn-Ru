---
title: "Схема"
description: "Базовая структура схемы Flex содержит title, description и type, которые описывают тип, и три раздела: config, blueprints и form, которые описывают разные аспекты типа директории."
tags:
  - "Grav 1.7"
  - "Flex-объекты"
  - "Схемы"
---

# Схема

Базовая структура **схемы Flex** содержит `title`, `description` и `type`, которые описывают тип, и три раздела: `config`, `blueprints` и `form`, которые описывают разные аспекты типа директории.

Основная структура `contacts.yaml`:
```yaml
title: Contacts
description: Simple contact directory with tags.
type: flex-objects  # не меняйте

# Конфигурация Flex
config: {}

# Формы директории Flex
blueprints: {}

# Форма объекта Flex
form: {}
```

Чтобы создать собственную директорию, начните с имени `type` (имени файла) и заполните `title` и `description`.

После создания файла и заполнения базовой информации следующий шаг — скопировать существующую форму или добавить в файл несколько полей.

> [!WARNING] Предупреждение
> **Совет:** мы предполагаем, что вы уже умеете создавать свои **[формы и схемы](../../../../forms/index.md)**.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** лучше не использовать простой формат списка для описания полей, как рассказано в разделе **[Создание простой одиночной формы](../../../../forms/forms/index.md#create-a-simple-single-form)**. Также не переносите в этот файл раздел формы `process`: Flex его не использует.

## Форма

В нашем примере с контактами раздел формы выглядит так:

```yaml
# Форма объекта Flex
form:
  validation: loose

  fields:
    published:
      type: toggle
      label: Published
      highlight: 1
      default: 1
      options:
        1: PLUGIN_ADMIN.YES
        0: PLUGIN_ADMIN.NO
      validate:
        type: bool
        required: true

    last_name:
      type: text
      label: Last Name
      validate:
        required: true

    first_name:
      type: text
      label: First Name
      validate:
        required: true

    email:
      type: email
      label: Email Address
      validate:
        required: true

    website:
      type: url
      label: Website URL

    tags:
      type: selectize
      size: large
      label: Tags
      classes: fancy
      validate:
        type: commalist
```

Форма выглядит одинаково независимо от того, взята ли она со страницы, из конфигурации или из файла схемы плагина либо темы. Это основная схема для каждого объекта в вашей директории, и она должна содержать все поля, определённые у объекта. Считайте её формой, которая показывается администратору.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** будьте осторожны, меняя схему существующего типа Flex. Убедитесь, что уже сохранённые объекты совместимы с новой версией схемы — то есть вы должны уметь и сохранять, и отображать старые объекты.

Мы ещё не закончили. Чтобы директория заработала, нужно сделать ещё две вещи: настроить слой хранения данных и определить поля, отображаемые в списке админки. И то, и другое делается в разделе `config`.

## Config

Раздел config — самая сложная часть схемы Flex, хотя многое в нём существует просто для настройки. Он содержит разделы `data`, `admin` и `site`.

```yaml
# Конфигурация Flex
config:

  # Настройки данных
  data: {}

  # Настройки админки
  admin: {}

  # Настройки сайта
  site: {}
```

Минимальная конфигурация выглядит примерно так:

```yaml
# Конфигурация Flex
config:

  # Настройки данных
  data:
    storage: user-data://flex-objects/contacts.json

  # Настройки админки
  admin:
    # Вид списка
    list:
      # Список полей для отображения
      fields:
        last_name:
          link: edit # Ссылка на редактирование
        first_name:
          link: edit # Ссылка на редактирование
        email:
        website:
```

В конфигурации есть два обязательных раздела: `config.data.storage` и `config.admin.list.fields`. Второй определяет поля, отображаемые в списке админки. Хранилище данных, в свою очередь, определяет, как данные будут храниться.

### Config > Data

**Директория Flex** очень гибко настраивается. Можно использовать собственные PHP-классы `object`, `collection` и `index`, чтобы добавить своё поведение. Кроме того, слой `storage` можно настроить под свои нужды. У директории также есть функциональность `ordering` и `search` по умолчанию.

```yaml
config:
  data:
    # Класс объекта Flex
    object: CLASSNAME
    # Класс коллекции Flex
    collection: CLASSNAME
    # Класс индекса Flex
    index: CLASSNAME
    # Параметры хранилища
    storage: {}
    # Параметры сортировки
    ordering: {}
    # Параметры поиска
    search: {}
```

Object, collection и index принимают имена классов. Если они не указаны, Grav использует такую конфигурацию по умолчанию:

```yaml
config:
  data:
    object: 'Grav\Common\Flex\Types\Generic\GenericObject'
    collection: 'Grav\Common\Flex\Types\Generic\GenericCollection'
    index: 'Grav\Common\Flex\Types\Generic\GenericIndex'
```

Вместе эти классы определяют поведение вашего типа. Если хотите настроить собственный тип, это можно сделать, расширив эти классы и указав здесь свои.

Одна из самых важных частей — определить, где и как хранятся данные:

```yaml
config:
  data:
    storage:
      class: 'Grav\Framework\Flex\Storage\SimpleStorage'
      options:
        formatter:
          class: 'Grav\Framework\File\Formatter\JsonFormatter'
        folder: user-data://flex-objects/contacts.json
```

Выше особый случай, который можно записать и короткой формой:

```yaml
config:
  data:
    storage: user-data://flex-objects/contacts.json
```

Grav 1.7 предлагает 3 разные стратегии хранения, хотя вы легко можете создать свою:

| Название | Класс | Описание |
|------|-------|-------------|
| Simple Storage | Grav\Framework\Flex\Storage\SimpleStorage | Все объекты хранятся в одном файле. Не поддерживает медиафайлы. |
| File Storage | Grav\Framework\Flex\Storage\FileStorage | Объекты хранятся в отдельных файлах в одной папке. |
| Folder Storage | Grav\Framework\Flex\Storage\FolderStorage | Каждый объект хранится в отдельной папке. |

Кроме того, можно указать формат файла через `options.formatter.class`:

| Название | Класс | Описание |
|------|-------|-------------|
| JSON | Grav\Framework\File\Formatter\JsonFormatter | Использовать формат JSON. |
| YAML | Grav\Framework\File\Formatter\YamlFormatter | Использовать формат YAML. |
| Markdown | Grav\Framework\File\Formatter\MarkdownFormatter | Использовать формат Markdown-файлов Grav с YAML-frontmatter. |
| Serialize | Grav\Framework\File\Formatter\SerializeFormatter | Использовать сериализатор PHP. Быстро, но не читаемо человеком. |
| INI | Grav\Framework\File\Formatter\IniFormatter | Использовать формат INI. Не рекомендуется. |
| CSV | Grav\Framework\File\Formatter\CsvFormatter | Использовать формат CSV. Не рекомендуется. |

Параметры конфигурации (со значениями по умолчанию) для форматтеров по умолчанию приведены ниже на вкладках:

```yaml title="JSON"
# JSON
formatter:
  class: 'Grav\Framework\File\Formatter\JsonFormatter'
  options:
    file_extension: '.json'
    encode_options: '' # см. https://www.php.net/manual/en/function.json-encode.php (параметры через пробел)
    decode_assoc: true # декодировать объекты как массивы
    decode_depth: 512  # декодировать до 512 уровней
    decode_options: '' # см. https://www.php.net/manual/en/function.json-decode.php (параметры через пробел)
```

```yaml title="YAML"
# YAML
formatter:
  class: 'Grav\Framework\File\Formatter\YamlFormatter'
  options:
    file_extension: '.yaml'
    inline: 5           # сохранять до 4 раскрытых уровней
    indent: 2           # отступ в 2 пробела
    native: true        # использовать нативный декодер YAML, если доступен
    compat: true        # если YAML не декодируется, использовать режим совместимости (МЕДЛЕННО)
```

```yaml title="Markdown"
# Markdown
formatter:
  class: 'Grav\Framework\File\Formatter\MarkdownFormatter'
  options:
    file_extension: '.md'
    header: 'header'    # переменная заголовка, например header.title
    body: 'markdown'    # переменная тела
    raw: 'frontmatter'  # переменная RAW YAML
    yaml:
      inline: 20        # параметры YAML, см. форматтер YAML выше
```

```yaml title="Serialize"
# PHP Serialize
formatter:
  class: 'Grav\Framework\File\Formatter\SerializeFormatter'
  options:
    file_extension: '.ser'
    decode_options:
      allowed_classes: ['stdClass'] # список разрешённых / безопасных классов при unserialize
```

```yaml title="INI"
# INI
formatter:
  class: 'Grav\Framework\File\Formatter\IniFormatter'
  options:
    file_extension: '.ini'
```

```yaml title="CSV"
# CSV
formatter:
  class: 'Grav\Framework\File\Formatter\CsvFormatter'
  options:
    file_extension: ['.csv', '.tsv']
    delimiter: ','      # разделитель значений
    mime: 'text/x-csv'  # MIME-тип для скачивания файла
```

Можно также задать сортировку по умолчанию, которая определяется парами `key: ASC|DESC`:

```yaml
config:
  data:
    # Параметры сортировки
    ordering:
      key: ASC
      timestamp: ASC
```

Наконец, можно добавить поля поиска, которые учитываются при вызове `collection.search()`:

```yaml
config:
  data:
    search:
      # Поля для поиска
      fields:
        - last_name
        - first_name
        - email
      # Параметры поиска
      options:
        - contains: 1   # если поле содержит строку поиска, назначить объекту вес 1
```

**Fields** содержит список полей, по которым идёт поиск.

Параметры поиска могут быть такими:

| Название | Значение | Описание |
|------|-------|-------------|
| case_sensitive | `true` или `false` | Если true, все проверки чувствительны к регистру; по умолчанию false |
| same_as | 0 ... 1 | Значение поля должно точно совпадать со строкой поиска |
| starts_with | 0 ... 1 | Значение поля должно начинаться со строки поиска |
| ends_with | 0 ... 1 | Значение поля должно заканчиваться строкой поиска |
| contains | 0 ... 1 | Значение поля должно содержать строку поиска |

Функция поиска возвращает 0, если поле не совпало, и вес от 0 до 1, если совпадение есть. Вес используется для сортировки результатов поиска. Объект с наибольшим баллом совпадает лучше, чем объект с меньшим баллом.

> [!NOTE] Примечание
> Сейчас поиск не поддерживает разные веса или стратегии для разных полей.

### Config > Admin

Раздел admin содержит различные параметры для настройки администрирования директории. В нём есть несколько основных разделов: `router`, `actions`, `permissions`, `menu`, `template` и `views`.

```yaml
config:
  # Настройки админки
  admin:
    # Маршрутизатор админки
    router: {}
    # Разрешённые действия в админке
    actions: {}
    # Права доступа
    permissions: {}
    # Меню админки
    menu: {}
    # Тип шаблона админки
    template: pages
    # Виды админки
    views: {}
```

Необязательный раздел `router` позволяет настроить маршруты админки **директории Flex**. Маршрутизация поддерживает базовый путь, настраиваемые маршруты для каждого действия, а также перенаправления, например для обратной совместимости. Все пути относительны базового URL админки.

```yaml
config:
  admin:
    # Маршрутизатор админки
    router:
      path: '/contacts' # собственный путь к директории
      actions:
        configure: # имя действия
          path: '/contacts/configure' # новый путь к действию.
      redirects: # список перенаправлений (откуда: куда)
        '/flex-objects/contacts': '/contacts'
```

Иногда нужно ограничить администрирование так, чтобы только показывать записи или, например, разрешить только редактирование существующих. Для этого есть `actions`, где можно подогнать разрешённые операции CRUD под свои нужды.

```yaml
config:
  admin:
    # Разрешённые действия в админке (для всех пользователей, включая суперпользователя)
    actions:
      list: true   # должно быть true (может измениться в будущем)
      create: true # false запрещает создание новых объектов
      read: true   # false запрещает ссылку на редактирование / подробности объектов
      update: false # false запрещает сохранение существующих объектов
      delete: false # false запрещает удаление объектов
```

Пример выше запретит сохранение и удаление существующих объектов для всех пользователей, включая суперадминистратора.

Раздел permissions позволяет добавлять новые правила разрешений для Grav. Эти правила появятся в админке пользователей и групп. Можно создать сколько угодно правил разрешений, но чтобы они работали, в этом файле нужно добавить свою логику или правила `authorize`.

```yaml
config:
  admin:
    # Права доступа
    permissions:
      # Основные разрешения (используются для объектов)
      admin.contacts:
        type: crudl # создание, чтение, обновление, удаление, список
        label: Contacts Directory
      # Вторичные разрешения (их нужно назначить виду, иначе они не используются)
      admin.configuration.contacts:
        type: default # простое разрешение
        label: Contacts Configuration
```

Если вы не хотите отображать директорию в администрировании `Flex Objects`, один из вариантов — показать пункт меню в основной навигации. Это делается в разделе конфигурации `menu`.

```yaml
config:
  admin:
    # Меню админки
    menu:
      list:
        hidden: false # если true, скрыть пункт меню.
        route: '/contacts' # псевдоним для `config.admin.router.path`, если путь маршрутизатора не задан.
        title: Contacts
        icon: fa-address-card-o
        authorize: ['admin.contacts.list', 'admin.super'] # разрешение, необходимое для доступа к пункту меню.
        priority: 2 # приоритет от -10 до 10 (чем больше, тем выше)
```

Пример выше создаёт пункт меню **Contacts**, ведущий на `/admin/contacts`.

Когда вы создаёте свои директории Flex, иногда хочется использовать одни и те же шаблоны во всех пользовательских директориях. Это делается с помощью `template`:

```yaml
config:
  admin:
    # Тип (папка) шаблона админки
    template: contacts
```

У **админки Flex** есть несколько видов объектов. По умолчанию поддерживаются виды `list`, `edit`, `configure` и, необязательно, `preview` и `export`. Можно добавлять и собственные виды.

```yaml
config:
  admin:
    views:
      # Вид списка
      list: {}
      # Вид редактирования
      edit: {}
      # Вид конфигурации
      configure: {}
      # Предпросмотр
      preview: {}
      # Экспорт данных
      export: {}
```

#### Вид списка

Первый нужный вид — тот, что выводит все объекты. По умолчанию вид `list` использует *VueTable* и *AJAX* для постраничного вывода объектов. Ему нужен список `fields` для отображения и `options`, определяющие, сколько элементов показывается сразу, а также поле сортировки по умолчанию.

```yaml
config:
  admin:
    views:
      # Вид списка
      list:
        icon: fa-address-card-o
        title: Site Contacts
        fields: {}        # см. ниже
        options:
          per_page: 20    # число элементов на странице по умолчанию
          order:
            by: last_name # поле сортировки по умолчанию
            dir: asc      # направление сортировки по умолчанию
```

**Icon** и **title** используются для настройки иконки и заголовка страницы списка. **Title** также поддерживает шаблон Twig в таком формате:

```yaml
        title:
          template: "{{ 'PLUGIN_CONTACTS.CONTACTS_TITLE'|tu }}"
```

**Fields** содержит поля, которые вы хотите отображать в списке директории. У каждого поля есть ключ — имя поля. Значение можно опустить, либо оно может содержать следующие параметры:

| Название | Значение | Пример | Описание |
|------|-------|-------------|-------------|
| width | `integer` | 8 | Ширина поля в пикселях |
| alias | `string` | 'header.published' | Имя поля формы, которое нужно использовать. VueTable не любит точки в именах, поэтому задайте псевдоним для вложенных переменных. |
| field | `array` |  | Переопределение поля формы. Пишется как любое поле формы, но без ключа. |
| link | `string` | 'edit' | Добавляет ссылку на редактирование к тексту. |
| search | `boolean` | true | Делает поле доступным для поиска в списке админки. |
| sort | `array` | field: 'first_name' | Можно указать другое значение, если при запросе данных на сервере используется другое имя поля, например first_name. |
| title_class | `string` | 'center' | CSS-классы, используемые в заголовках. |
| data_class | `string` |  'left' | CSS-классы, используемые в колонках данных. |

#### Вид редактирования

У вида редактирования те же базовые параметры, что и у вида списка:

```yaml
config:
  admin:
    views:
      # Вид редактирования
      edit:
        icon: fa-address-card-o
        title:
          template: '{{ object.last_name ?? ''Last'' }}, {{ object.first_name ?? ''First Name'' }}'
```

#### Вид конфигурации

Вид конфигурации позволяет добавлять параметры уровня всей директории, которые затем можно использовать в файлах шаблонов.

```yaml
config:
  admin:
    views:
      # Вид конфигурации
      configure:
        hidden: false # кнопку конфигурации можно скрыть, например если у вас есть своя вкладка, как в Accounts.
        authorize: 'admin.configuration.contacts' # необязательное правило authorize для этого вида.
        file: 'config://flex/contacts.yaml' # необязательный файл, в котором сохраняется конфигурация.

        icon: fa-cog
        title:
          template: "{{ directory.title }} {{ 'PLUGIN_ADMIN.CONFIGURATION'|tu }}"
```

#### Вид предпросмотра

Flex также поддерживает предпросмотр, хотя сейчас он работает через рендеринг страницы фронтенда, которую можно задать в файле схемы.

```yaml
    # Вид предпросмотра
    preview:
      enabled: true
      route:
        template: '/contacts' # Twig-шаблон для создания URL. Здесь используется вид списка

       icon: fa-address-card-o
        title:
          template: "{{ object.form.getValue('title') ?? object.title ?? key }}"
```

#### Вид экспорта

Все объекты можно экспортировать в один файл. Вот пример конфигурации для экспорта данных в YAML-файл:

```yaml
    # Экспорт данных
    export:
      enabled: true
      method: 'jsonSerialize'
      formatter:
        class: 'Grav\Framework\File\Formatter\YamlFormatter'
      filename: 'contacts'
```

### Config > Site

```yaml
config:
  # Настройки сайта
  site:
    templates:
      collection:
        # Поиск файлов макета шаблона для коллекций объектов
        paths:
          - 'flex/{TYPE}/collection/{LAYOUT}{EXT}'
      object:
        # Поиск файлов макета шаблона для объектов
        paths:
          - 'flex/{TYPE}/object/{LAYOUT}{EXT}'
      defaults:
        # Переменная шаблона {TYPE} по умолчанию; переопределяется именем файла этой схемы, если папка шаблонов существует
        type: contacts
        # Переменная шаблона {LAYOUT} по умолчанию; может переопределяться в вызовах render (обычно в Twig-шаблонах)
        layout: default
```

Настройки шаблонов позволяют настроить пути поиска шаблонов и задать имя типа и макета по умолчанию на фронтенде.

## Blueprints

Раздел blueprints определяет общие параметры конфигурации для всей директории. Они позволяют настроить общую директорию под нужды сайта, не требуя ручного редактирования файлов.

```yaml
blueprints:
  # Схема для вида конфигурации.
  configure:
    # Мы внутри поля TABS.
    fields:
      # Добавляем собственную вкладку
      compatibility:
        type: tab
        title: Compatibility
        fields:
          # Поля должны иметь префикс object, collection и т. д.
          object.compat.events:
            type: toggle
            toggleable: true
            label: Admin event compatibility
            help: Enables onAdminSave and onAdminAfterSave events for plugins
            highlight: 1
            default: 1
            options:
              1: PLUGIN_ADMIN.ENABLED
              0: PLUGIN_ADMIN.DISABLED
            validate:
              type: bool
```

> [!WARNING] Предупреждение
> **Совет:** эти параметры конфигурации можно изменить в разделе **[Конфигурация](../../administration/configuration/index.md)** раздела **[Администрирование директории Flex](../../administration/index.md)**.

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** сейчас используются только параметры конфигурации из раздела кэша. Для своих настроек вам нужно самостоятельно добавить логику их использования.
