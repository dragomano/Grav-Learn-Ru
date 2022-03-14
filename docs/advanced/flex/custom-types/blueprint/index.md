---
description: Руководство по созданию чертежей Flex в Grav CMS.
---

# Чертёж

Базовая структура **чертежа Flex** содержит `title`,` description` и `type`, описывающие тип, и три раздела: `config`, `blueprints` и `form`, описывающие различные аспекты типа каталога.

Основная структура `contacts.yaml`:

```yaml
title: Contacts
description: Simple contact directory with tags.
type: flex-objects  # do not change

# Flex Configuration
config: {}

# Flex Directory Forms
blueprints: {}

# Flex Object Form
form: {}
```

Чтобы создать свой собственный каталог, вам нужно начать с присвоения имени вашему `type` (имя файла) и заполнения` title` и `description`.

После создания файла и заполнения основной информации следующим шагом будет либо копирование существующей формы, либо добавление некоторых полей в файл.

!!! tip ""

	Мы предполагаем, что вы уже знаете, как создавать свои собственные **[формы и чертежи](/forms)**.

!!! warning ""

	Лучше не использовать простой формат списка для описания полей, как указано в секции **[Создание простой формы](/forms/forms/#sozdanie-prostoi-formy)**. Также не передавайте в этот файл раздел формы `process`, он не будет использоваться Flex.

## Секция Form

В нашем примере с контактами раздел формы выглядит так:

```yaml
# Flex Object Form
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

Форма выглядит одинаково независимо от того, была ли она взята со страницы или из файла конфигурации, плагина или схемы темы. Это основной план для каждого объекта в вашем каталоге, и он должен содержать все поля, определенные в объекте. Подумайте об этом как о форме, отображаемой администратору.

!!! warning ""

	Будьте осторожны при изменении чертежа для существующего типа Flex. Убедитесь, что объекты, которые вы уже сохранили, совместимы с новой версией чертежа — это означает, что вы должны иметь возможность как сохранять, так и отображать старые объекты.

Мы ещё не закончили. Есть ещё две вещи, которые необходимо сделать, чтобы каталог заработал: нам нужно настроить уровень хранения данных и определить поля для отображения в представлении «Список администраторов». Мы можем сделать и то, и другое в секции `config`.

## Секция Config

Раздел конфигурации — самая сложная часть чертежа Flex, хотя большая часть его предназначена только для настройки. Он содержит разделы `data`, `admin` и `site`.

```yaml
# Flex Configuration
config:

  # Data Settings
  data: {}

  # Admin Settings
  admin: {}

  # Site Settings
  site: {}
```

Минимальная конфигурация выглядит примерно так:

```yaml
# Flex Configuration
config:

  # Data Settings
  data:
    storage: user-data://flex-objects/contacts.json

  # Admin Settings
  admin:
    # List view
    list:
      # List of fields to display
      fields:
        last_name:
          link: edit # Edit link
        first_name:
          link: edit # Edit link
        email:
        website:
```

В конфигурации есть два обязательных раздела: `config.data.storage` и `config.admin.list.fields`. Последний определяет поля, отображаемые в представлении списка администраторов. С другой стороны, хранилище данных определяет, как данные будут храниться.

### Config > Data

**Flex-каталог** легко настраиваемый. Вы можете использовать свои классы `object`, `collection` и `index` для добавления собственного поведения. Кроме того, вы можете настроить уровень хранилища (`storage`) в соответствии с вашими потребностями. Справочник также имеет стандартные функции «упорядочивания» (`ordering`) и «поиска» (`search`).

```yaml
config:
  data:
    # Flex Object Class
    object: CLASSNAME
    # Flex Collection Class
    collection: CLASSNAME
    # Flex Index Class
    index: CLASSNAME
    # Storage Options
    storage: {}
    # Ordering Options
    ordering: {}
    # Search Options
    search: {}
```

Объект, коллекция и индекс принимают имена классов. Если они не указаны, Grav будет использовать следующую конфигурацию по умолчанию:

```yaml
config:
  data:
    object: 'Grav\Common\Flex\Types\Generic\GenericObject'
    collection: 'Grav\Common\Flex\Types\Generic\GenericCollection'
    index: 'Grav\Common\Flex\Types\Generic\GenericIndex'
```

Эти классы вместе будут определять поведение вашего типа. Если вы хотите настроить свой собственный тип, это можно сделать, расширив эти классы и передав здесь свои собственные классы.

Одна из наиболее важных частей - определить, где и как хранятся данные:

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

Выше частный случай, который также можно записать в краткой форме:

```yaml
config:
  data:
    storage: user-data://flex-objects/contacts.json
```

Grav 1.7 предоставляет 3 различных стратегии хранения, хотя вы можете легко создать свою собственную:


| Название | Класс | Описание |
|------|-------|-------------|
| Simple Storage | Grav\Framework\Flex\Storage\SimpleStorage | Все объекты хранятся в одном файле. Не поддерживает медиа. |
| File Storage | Grav\Framework\Flex\Storage\FileStorage | Объекты хранятся в отдельных файлах в одной папке. |
| Folder Storage | Grav\Framework\Flex\Storage\FolderStorage | Каждый объект хранится в отдельной папке. |


Кроме того, вы можете указать формат файла с `options.formatter.class`:


| Название | Класс | Описание |
|------|-------|-------------|
| JSON | Grav\Framework\File\Formatter\JsonFormatter | Использовать формат файла JSON. |
| YAML | Grav\Framework\File\Formatter\YamlFormatter | Использовать формат файла YAML. |
| Markdown | Grav\Framework\File\Formatter\MarkdownFormatter | Использовать формат файла Grav Markdown с обложкой YAML. |
| Serialize | Grav\Framework\File\Formatter\SerializeFormatter | Использовать сериализатор PHP. Быстро, но не читается человеком. |
| INI | Grav\Framework\File\Formatter\IniFormatter | Использовать формат файла INI. Не рекомендуется. |
| CSV | Grav\Framework\File\Formatter\CsvFormatter | Использовать формат файла CSV. Не рекомендуется. |


Параметры конфигурации (со значениями по умолчанию) для средств форматирования по умолчанию можно найти ниже во вкладках:

=== "JSON"

	```yaml
	# JSON
	formatter:
	class: 'Grav\Framework\File\Formatter\JsonFormatter'
	options:
		file_extension: '.json'
		encode_options: '' # See https://www.php.net/manual/en/function.json-encode.php (separate options with space)
		decode_assoc: true # Decode objects as arrays
		decode_depth: 512  # Decode up to 512 levels
		decode_options: '' # See https://www.php.net/manual/en/function.json-decode.php (separate options with space)
	```

=== "Yaml"

	```yaml
	# YAML
	formatter:
	class: 'Grav\Framework\File\Formatter\YamlFormatter'
	options:
		file_extension: '.yaml'
		inline: 5           # Save with up to 4 expanded levels
		indent: 2           # Indent with 2 spaces
		native: true        # Use native YAML decoder if available
		compat: true        # If YAML cannot be decoded, use compatibility mode (SLOW)
	```

=== "Markdown"

	```yaml
	# Markdown
	formatter:
	class: 'Grav\Framework\File\Formatter\MarkdownFormatter'
	options:
		file_extension: '.md'
		header: 'header'    # Header variable eg. header.title
		body: 'markdown'    # Body variable
		raw: 'frontmatter'  # RAW YAML variable
		yaml:
		inline: 20        # YAML options, see YAML formatter from above
	```

=== "Serialize"

	```yaml
	# PHP Serialize
	formatter:
	class: 'Grav\Framework\File\Formatter\SerializeFormatter'
	options:
		file_extension: '.ser'
		decode_options:
		allowed_classes: ['stdClass'] # List of allowed / safe classes during unserialize
	```

=== "INI"

	```yaml
	# INI
	formatter:
	class: 'Grav\Framework\File\Formatter\IniFormatter'
	options:
		file_extension: '.ini'
	```

=== "CSV"

	```yaml
	# CSV
	formatter:
	class: 'Grav\Framework\File\Formatter\CsvFormatter'
	options:
		file_extension: ['.csv', '.tsv']
		delimiter: ','      # Delimiter to separate the values
		mime: 'text/x-csv'  # MIME type for downloading file
	```

Вы также можете установить порядок по умолчанию, который определяется парами `key: ASC|DESC`:

```yaml
config:
  data:
    # Ordering Options
    ordering:
      key: ASC
      timestamp: ASC
```

Наконец, вы можете добавить поля поиска, которые просматриваются при вызове `collection.search()`:

```yaml
config:
  data:
    search:
      # Fields to be searched
      fields:
        - last_name
        - first_name
        - email
      # Search Options
      options:
        - contains: 1   # If field contains the search string, assign weight 1 to the object
```

**Fields** содержат список полей для поиска.

Возможные варианты поиска:

| Название | Класс | Описание |
|------|-------|-------------|
| case_sensitive | `true` или `false` | Если true, все проверки чувствительны к регистру, по умолчанию false |
| same_as | 0 ... 1 | Значение поля должно совпадать со строкой поиска |
| starts_with | 0 ... 1 | Значение поля должно начинаться со строки поиска |
| ends_with | 0 ... 1 | Значение поля должно заканчиваться строкой поиска |
| contains | 0 ... 1 | Значение поля должно содержать строку поиска |


Функция поиска возвращает 0, если поле не совпадает, и вес от 0 до 1, Если есть совпадение. Вес используется для упорядочения результатов поиска. Объект, который получает самое высокое ядро, имеет лучшее совпадение, чем объект с более низким баллом.

!!! success ""

	В настоящее время поиск не поддерживает наличие различных весов или стратегий для каждого поля.

### Config > Admin

Раздел Admin содержит различные параметры конфигурации для настройки администрирования каталога. Он содержит несколько основных разделов: `router`, `actions`, `permissions`, `menu`, `template` и `views`.

```yaml
config:
  # Admin Settings
  admin:
    # Admin router
    router: {}
    # Allowed admin actions
    actions: {}
    # Permissions
    permissions: {}
    # Admin menu
    menu: {}
    # Admin template type
    template: pages
    # Admin views
    views: {}
```

Дополнительный раздел `router` можно использовать для настройки маршрутов администрирования **Flex-каталога**. Маршрутизация поддерживает базовый путь, настраиваемые маршруты для каждого действия, а также перенаправление для обработки, например, обратной совместимости. Все пути относятся к базовому URL-адресу администратора.

```yaml
config:
  admin:
    # Admin router
    router:
      path: '/contacts' # Custom path to the directory
      actions:
        configure: # Action name
          path: '/contacts/configure' # New path to the action.
      redirects: # List of redirects (from: to)
        '/flex-objects/contacts': '/contacts'
```

Иногда вы хотите ограничить администрирование только отображением записей или, например, разрешить редактировать только существующие. Для этого `actions` вы можете настроить разрешенные операции CRUD, чтобы они лучше соответствовали вашим потребностям.

```yaml
config:
  admin:
    # Allowed admin actions (for all users, including super user)
    actions:
      list: true   # Needs to be true (may change in the future)
      create: true # Set to false to disable creating new objects
      read: true   # Set to false to disable link to edit / details of the objects
      update: false # Set to false to disable saving existing objects
      delete: false # Set to false to disable deleting objects
```

Приведенный выше пример предотвратит сохранение существующих объектов и их удаление для каждого пользователя, включая суперадминистратора.

Раздел разрешений позволяет вам добавлять новые правила разрешений для Grav. Эти правила появятся в администраторе пользователя/группы. Вы можете создать столько правил разрешений, сколько захотите, но вам нужно добавить свою собственную логику или правила авторизации в этот файл, чтобы использовать их.

```yaml
config:
  admin:
    # Permissions
    permissions:
      # Primary permissions (used for the objects)
      admin.contacts:
        type: crudl # Create, Read, Update, Delete, List
        label: Contacts Directory
      # Secondary permissions (you need to assign these to a view, otherwise these will not be used)
      admin.configuration.contacts:
        type: default # Simple permission
        label: Contacts Configuration
```

Если вы не хотите отображать свой каталог в администрировании `Flex Objects`, один из вариантов — отобразить пункт меню в основной навигации. Вы можете сделать это в секции `menu`.

```yaml
config:
  admin:
    # Admin Menu
    menu:
      list:
        hidden: false # If true, hide the menu item.
        route: '/contacts' # Alias to `config.admin.router.path` if router path is not set.
        title: Контакты
        icon: fa-address-card-o
        authorize: ['admin.contacts.list', 'admin.super'] # Authorization needed to access the menu item.
        priority: 2 # Priority -10 .. 10 (highest goes up)
```

В приведенном выше примере создается пункт меню **Контакты**, указывающий на `/admin/contacts`.

Когда вы создаете свои собственные каталоги Flex, вам иногда может потребоваться использовать одни и те же шаблоны для всех ваших пользовательских каталогов. Вы можете сделать это с помощью `template`:

```yaml
config:
  admin:
    # Admin template type (folder)
    template: contacts
```

**Flex Admin** имеет несколько представлений для объектов. По умолчанию поддерживаются следующие представления: `list`, `edit`, `configure` и необязательные `preview` и `export`. Также возможно добавить свои собственные представления.

```yaml
config:
  admin:
    views:
      # List view
      list: {}
      # Edit view
      edit: {}
      # Configure view
      configure: {}
      # Preview
      preview: {}
      # Data Export
      export: {}
```

#### Просмотр списка

Первое представление, которое вам понадобится — это то, в котором перечислены все ваши объекты. По умолчанию представление `list` использует *VueTable* и *AJAX* для разбивки объектов на страницы. Ему нужен список «полей» (`fields`) для отображения, а также «параметры» (`options`), чтобы определить, сколько элементов отображается одновременно, а также поле по умолчанию, используемое для упорядочивания.

```yaml
config:
  admin:
    views:
      # List view
      list:
        icon: fa-address-card-o
        title: Site Contacts
        fields: {}        # See below
        options:
          per_page: 20    # Default number of items per page
          order:
            by: last_name # Default field used for ordering
            dir: asc      # Default ordering direction
```

**Icon** и **Title** используются для настройки значка и заголовка страницы со списком. **Title** также поддерживает использование шаблона Twig в следующем формате:

```yaml
        title:
          template: "{{ 'PLUGIN_CONTACTS.CONTACTS_TITLE'|tu }}"
```

**Fields** содержит поля, которые вы хотите отобразить в списке каталогов. У каждого поля есть ключ, который представляет собой имя поля. Значение можно не указывать или оно может содержать следующие параметры конфигурации:


| Название | Значение | Пример | Описание |
|------|-------|-------------|
| width | `integer` | 8 | Ширина поля в пикселях |
| alias | `string` | 'header.published' | Имя используемого поля формы. VueTable не любит точки в именах, поэтому установите псевдоним для вложенных переменных. |
| field | `array` |  | Переопределение поля формы. Написано так же, как и любое поле формы, но без ключа. |
| link | `string` | 'edit' | Добавляет ссылку для редактирования в текст. |
| search | `boolean` | true | Делает поле доступным для поиска в списке администрирования. |
| sort | `array` | field: 'first_name' | Вы можете указать другое значение, если вы используете другое имя поля при запросе данных на стороне сервера, например first_name. |
| title_class | `string` | 'center' | Классы CSS, используемые в заголовках. |
| data_class | `string` |  'left' | Классы CSS, используемые в столбцах данных. |


#### Редактирование

Режим редактирования имеет те же основные параметры конфигурации, что и режим просмотра списка:

```yaml
config:
  admin:
    views:
      # Edit view
      edit:
        icon: fa-address-card-o
        title:
          template: '{{ object.last_name ?? ''Last'' }}, {{ object.first_name ?? ''First Name'' }}'
```

#### Настройка

Можно добавлять параметры конфигурации каталога в широком диапазоне, которые затем могут быть использованы в файлах шаблонов.

```yaml
config:
  admin:
    views:
      # Configure view
      configure:
        hidden: false # Configuration button can be hidden, for example if you have custom tab to replace it, like in Accounts.
        authorize: 'admin.configuration.contacts' # Optional custom authorize rule for this view.
        file: 'config://flex/contacts.yaml' # Optional file where the configuration is saved.

        icon: fa-cog
        title:
          template: "{{ directory.title }} {{ 'PLUGIN_ADMIN.CONFIGURATION'|tu }}"
```

#### Предварительный просмотр

Flex также поддерживает предварительный просмотр, хотя сейчас он работает путем рендеринга страницы из внешнего интерфейса, которая может быть определена в файле blueprint.

```yaml
    # Preview View
    preview:
      enabled: true
      route:
        template: '/contacts' # Twig template to create URL. In this case we use the list view

       icon: fa-address-card-o
        title:
          template: "{{ object.form.getValue('title') ?? object.title ?? key }}"
```

#### Экспорт

Все объекты могут быть экспортированы в один файл. Вот пример конфигурации, как экспортировать данные в файл YAML:

```yaml
    # Data Export
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
  # Site Settings
  site:
    templates:
      collection:
        # Lookup for the template layout files for collections of objects
        paths:
          - 'flex/{TYPE}/collection/{LAYOUT}{EXT}'
      object:
        # Lookup for the template layout files for objects
        paths:
          - 'flex/{TYPE}/object/{LAYOUT}{EXT}'
      defaults:
        # Default template variable {TYPE}; overridden by filename of this blueprint if template folder exists
        type: contacts
        # Default template variable {LAYOUT}; can be overridden in render calls (usually Twig in templates)
        layout: default
```

Настройки шаблона позволяют настроить пути поиска шаблона и установить тип по умолчанию и имя макета во внешнем интерфейсе.

## Чертежи

Раздел чертежи определяет общие параметры конфигурации для всего каталога. Эти параметры позволяют настроить общий каталог в соответствии с потребностями сайта, не требуя ручного редактирования файлов.

```yaml
blueprints:
  # Blueprint for configure view.
  configure:
    # We are inside TABS field.
    fields:
      # Add our own tab
      compatibility:
        type: tab
        title: Compatibility
        fields:
          # Fields should be prefixed with object, collection etc..
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

!!! tip ""

	Эти параметры конфигурации могут быть изменены в секции **[Настройка](/advanced/flex/administration/configuration)** **[Администрирования Flex-каталога](/advanced/flex/administration)**.

!!! note ""

	В настоящее время единственные используемые параметры конфигурации находятся внутри раздела кэша. Для ваших пользовательских настроек вам нужно добавить логику, чтобы использовать их самостоятельно.
