---
title: "Справочник по схемам"
description: "Тип Flex определяется одним YAML-файлом схемы. В начале файла описывается тип (title, description, type), а затем он делится на три раздела: form (поля, которые редактирует администратор), config (как хранятся данные…"
tags:
  - "Flex-объекты"
  - "Схемы"
---

# Справочник по схемам

**Тип Flex** определяется одним YAML-файлом схемы. В начале файла описывается тип (`title`, `description`, `type`), а затем он делится на три раздела: `form` (поля, которые редактирует администратор), `config` (как хранятся данные, как ведут себя интерфейс [Admin Next](../../../../admin-panel/index.md) и REST API и как рендерит фронтенд) и `blueprints` (параметры конфигурации уровня всей директории).

Схемы лежат в `user/blueprints/flex-objects/<type>.yaml` либо могут поставляться плагином или темой. Чтобы активировать схему, включите её в **Admin Next** в разделе **Plugins > Flex Objects** (поле `directories`), что дописывает путь к схеме в конфигурацию плагина. Полный разбор активации см. в разделе [Пользовательские типы](../index.md).

Минимальный скелет схемы (здесь `contacts.yaml`) выглядит так:

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

Чтобы создать свою директорию, начните с имени `type` (имени файла), заполните `title` и `description` и оставьте `type: flex-objects` точно как показано.

> [!NOTE] Примечание
> Мы предполагаем, что вы уже умеете строить форму. Если нет, сначала прочитайте **[формы](../../../../forms/index.md)**, а затем возвращайтесь сюда.

## Форма

Раздел `form` — основная схема для каждого объекта в вашей директории. Он должен содержать все поля, определённые у объекта. Считайте его формой, которая показывается администратору при создании или редактировании элемента.

В примере с контактами форма выглядит так:

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

Режим `validation` (`loose` или `strict`) управляет тем, насколько строго отправленные данные проверяются по определениям полей. Форма пишется одинаково независимо от того, пришла ли она со страницы, из файла конфигурации, из плагина или из схемы темы.

> [!WARNING] Предупреждение
> **Не** используйте простой формат списка для описания полей (тот, что описан в разделе [Создание простой одиночной формы](../../../../forms/forms/index.md#create-a-simple-single-form)). Flex нужна полная карта `fields:` с именованными ключами. Также **не** добавляйте в форму Flex раздел `process:`; Flex его игнорирует, поэтому любая обработка, которую вы там ожидаете, молча не выполнится.

> [!WARNING] Предупреждение
> Будьте осторожны, меняя схему типа Flex, у которого уже есть сохранённые объекты. Убедитесь, что уже сохранённые объекты остаются совместимы с новой версией схемы, то есть вы должны уметь и сохранять, и отображать старые объекты.

Прежде чем директория заработает, нужно сделать ещё две вещи: настроить слой хранения и определить, какие поля появляются в виде списка Admin Next. И то, и другое находится в разделе `config`.

## Config

`config` — самая большая часть схемы Flex, хотя большая часть существует для настройки и может оставаться со значениями по умолчанию. Он содержит три подраздела: `data`, `admin` и `site`.

```yaml
# Конфигурация Flex
config:

  # Настройки данных (хранилище, классы, сортировка, поиск)
  data: {}

  # Настройки Admin Next (список, редактирование, меню, разрешения, экспорт)
  admin: {}

  # Настройки фронтенда (пути поиска шаблонов)
  site: {}
```

Самая маленькая рабочая конфигурация — это просто путь к хранилищу плюс набор полей списка:

```yaml
# Конфигурация Flex
config:

  # Настройки данных
  data:
    storage: user-data://flex-objects/contacts.json

  # Настройки Admin Next
  admin:
    list:
      fields:
        last_name:
          link: edit  # привязать ячейку к виду редактирования
        first_name:
          link: edit
        email:
        website:
```

Фактически обязательны две вещи: `config.data.storage` (где и как хранятся объекты) и `config.admin.list.fields` (колонки, показываемые в списке Admin Next). Всё остальное необязательно.

## Config: Data {: #config-data }
Раздел `data` очень гибко настраивается. Можно предоставить собственные PHP-классы `object`, `collection` и `index`, чтобы добавить поведение, настроить слой `storage` под свои нужды и задать поведение `ordering` и `search` по умолчанию.

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

### Классы object, collection и index

`object`, `collection` и `index` принимают полные имена классов PHP. Если их опустить, Grav использует значения Generic по умолчанию:

```yaml
config:
  data:
    object: 'Grav\Common\Flex\Types\Generic\GenericObject'
    collection: 'Grav\Common\Flex\Types\Generic\GenericCollection'
    index: 'Grav\Common\Flex\Types\Generic\GenericIndex'
```

Вместе эти три класса определяют поведение вашего типа. Чтобы настроить поведение сверх того, что позволяют схемы, расширьте эти классы и укажите в конфигурации свои подклассы. О том, как их регистрировать и использовать, см. [PHP-классы](../../php-and-events/index.md).

### Хранилище {: #storage }
Самое важное решение о данных — где и как хранятся объекты. Длинная форма описывает класс хранилища, форматтер файлов и целевую папку или файл:

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

Пример выше — обычный случай, поэтому его можно записать и короткой формой. Одна строка интерпретируется как JSON-файл `SimpleStorage` по указанному пути потока:

```yaml
config:
  data:
    storage: user-data://flex-objects/contacts.json
```

Grav поставляется с тремя стратегиями хранения (а свою можно написать, расширив базовый класс хранилища):

| Название | Класс | Описание |
|------|-------|-------------|
| Simple Storage | `Grav\Framework\Flex\Storage\SimpleStorage` | Все объекты хранятся в **одном файле**. **Не** поддерживает медиафайлы для отдельных объектов. |
| File Storage | `Grav\Framework\Flex\Storage\FileStorage` | Каждый объект хранится в **своём файле** внутри одной папки. |
| Folder Storage | `Grav\Framework\Flex\Storage\FolderStorage` | Каждый объект хранится в **своей папке**, что поддерживает медиафайлы для отдельных объектов. |

> [!IMPORTANT] Важно
> Если вашим объектам нужны изображения или загрузка файлов, используйте `FolderStorage`. `SimpleStorage` держит всё в одном файле и не имеет места для медиафайлов отдельных объектов, поэтому медиаэндпоинты для таких директорий возвращают ошибку.

Формат на диске выбирается через `options.formatter.class`:

| Название | Класс | Описание |
|------|-------|-------------|
| JSON | `Grav\Framework\File\Formatter\JsonFormatter` | Использовать формат JSON. |
| YAML | `Grav\Framework\File\Formatter\YamlFormatter` | Использовать формат YAML. |
| Markdown | `Grav\Framework\File\Formatter\MarkdownFormatter` | Использовать формат Markdown-файлов Grav с YAML-frontmatter. |
| Serialize | `Grav\Framework\File\Formatter\SerializeFormatter` | Использовать сериализатор PHP. Быстро, но не читаемо человеком. |
| INI | `Grav\Framework\File\Formatter\IniFormatter` | Использовать формат INI. Не рекомендуется. |
| CSV | `Grav\Framework\File\Formatter\CsvFormatter` | Использовать формат CSV. Не рекомендуется. |

Параметры конфигурации (со значениями по умолчанию) для каждого форматтера приведены на вкладках ниже:

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

### Сортировка

Сортировку по умолчанию можно задать парами `key: ASC|DESC`. Она применяется, когда коллекция загружается без явной сортировки:

```yaml
config:
  data:
    # Параметры сортировки
    ordering:
      key: ASC
      timestamp: ASC
```

### Поиск
Раздел `search` определяет, по каким полям идёт поиск при вызове `collection.search()`, и также обеспечивает работу параметра поиска в REST-эндпоинте списка.

```yaml
config:
  data:
    search:
      # Поля для поиска
      fields:
        - last_name
        - first_name
        - email
      # Параметры поиска (веса)
      options:
        contains: 1   # если поле содержит строку поиска, назначить объекту вес 1
```

`fields` — список полей, по которым идёт поиск. `options` сопоставляет стратегию совпадения с весом. Доступные стратегии:

| Название | Значение | Описание |
|------|-------|-------------|
| case_sensitive | `true` или `false` | Если true, все проверки чувствительны к регистру. По умолчанию false. |
| same_as | 0 ... 1 | Значение поля должно точно совпадать со строкой поиска. |
| starts_with | 0 ... 1 | Значение поля должно начинаться со строки поиска. |
| ends_with | 0 ... 1 | Значение поля должно заканчиваться строкой поиска. |
| contains | 0 ... 1 | Значение поля должно содержать строку поиска. |

Поиск возвращает 0, если поле не совпало, и вес от 0 до 1, если совпало. Вес используется для сортировки результатов, поэтому объект с более высоким баллом совпадает лучше, чем с меньшим.

> [!NOTE] Примечание
> Сейчас поиск не поддерживает разные веса или стратегии для разных полей; веса применяются ко всем полям из списка `fields`.

## Config: Admin
Раздел `admin` управляет интерфейсом **Admin Next** (одностраничным приложением на SvelteKit) и REST API, который его обслуживает. Классическая админка на Vue была удалена в flex-objects v1.4.2, поэтому описанные здесь ключи читает плагин API (главным образом `GET /flex-objects/{type}`), а рендерит Admin Next, а не какое-либо серверное представление Twig.

Основные ключи — `list`, `edit`, `menu`, `permissions`, `export` и `template`.

```yaml
config:
  admin:
    # Вид списка (колонки, постраничный вывод, вложенные строки подробностей)
    list: {}
    # Вид редактирования (шаблон заголовка)
    edit: {}
    # Пункт меню боковой панели
    menu: {}
    # Права доступа (см. отдельный раздел ниже)
    permissions: {}
    # Экспорт данных
    export: {}
    # Тип (папка) шаблона админки
    template: default
```

### List: fields

`list.fields` определяет колонки, показываемые в списке Admin Next. У каждой записи есть ключ (имя поля) и необязательная карта параметров. Если значение опущено, показывается сырое значение поля как есть.

В старой классической админке эти параметры существовали, чтобы обойти ограничения VueTable. Эта причина исчезла. Admin Next читает ту же конфигурацию через `GET /flex-objects/{type}`, который возвращает определения колонок вместе со строками с постраничным выводом, и рендерит собственную таблицу.

```yaml
config:
  admin:
    list:
      title: name       # необязательная подпись для страницы списка
      fields:
        published:
          field:
            type: toggle
            label: Publ
          width: 8
        last_name:
          link: edit
        first_name:
          link: edit
        email:
        website:
        tags:
```

Параметры отдельного поля:

| Название | Значение | Пример | Описание |
|------|-------|---------|-------------|
| width | `integer` | `8` | Подсказка ширины колонки. |
| alias | `string` | `header.published` | Имя поля формы, из которого читается значение. Полезно для вложенных значений, например `header.published`. |
| field | `array` | | Встроенное переопределение поля формы, написанное как поле формы, но без ключа. Управляет тем, как ячейка рендерится (и редактируется). |
| link | `string` | `edit` | Превращает ячейку в ссылку на вид редактирования. |
| search | `boolean` | `true` | Включает поле в поиск по списку Admin Next. |
| sort | `array` | `field: first_name` | Сопоставление с другим именем поля на бэкенде при сортировке, если запрос использует другое имя, чем отображаемое поле. |
| title_class | `string` | `center` | CSS-классы для заголовка колонки. **Только для классической админки, в Admin Next игнорируется.** |
| data_class | `string` | `left` | CSS-классы для ячеек данных. **Только для классической админки, в Admin Next игнорируется.** |

### List: options

`list.options` управляет постраничным выводом и сортировкой списка по умолчанию:

```yaml
config:
  admin:
    list:
      options:
        per_page: 20      # число элементов на странице по умолчанию
        order:
          by: last_name   # поле сортировки по умолчанию
          dir: asc        # направление сортировки по умолчанию (asc | desc)
```

### List: detail (вложенные связанные строки)

`list.detail` позволяет строке списка раскрываться и показывать связанные объекты из другой директории Flex, например заказы, принадлежащие клиенту. Admin Next разрешает связь, проверяет разрешение `list` связанной директории для текущего пользователя и рендерит дочерние строки прямо в списке.

```yaml
config:
  admin:
    list:
      detail:
        enabled: true
        label: Orders
        icon: fa-list
        limit: 10          # максимум дочерних строк (по умолчанию per_page дочернего типа)
        actions: true      # разрешить редактирование / удаление дочерних строк (нужны разрешения update / delete)
        relation:
          type: orders           # связанный тип Flex
          local_key: id          # поле ЭТОГО объекта
          foreign_key: customer  # поле связанного объекта, указывающее обратно
          sort:
            created: desc
        fields:
          # Опустите, чтобы использовать собственные поля списка дочернего типа, или переопределите по ключу:
          #   true  -> включить поле списка дочернего типа как есть
          #   false -> скрыть его
          #   map   -> наложить переопределения поверх поля списка дочернего типа
          number: true
          total: true
```

Если отсутствует `relation.type`, `local_key` или `foreign_key`, либо связанная директория не включена, либо у пользователя нет разрешения `list` на неё, блок подробностей тихо пропускается, и строка просто не раскрывается. Переключатель `actions` включает редактирование и удаление только тогда, когда у пользователя есть соответствующие разрешения `update` / `delete` на связанную директорию.

### Edit: шаблон заголовка

Вид `edit` принимает `icon` и `title`. Заголовок может быть шаблоном Twig, вычисляемым по текущему `object`, поэтому вкладка редактирования показывает осмысленную подпись:

```yaml
config:
  admin:
    edit:
      icon: fa-address-card
      title:
        template: '{{ object.last_name ?? ''Last'' }}, {{ object.first_name ?? ''First Name'' }}'
```

Заголовок списка `title` и заголовок меню `title` могут использовать ту же форму `template:` (например, `{{ 'PLUGIN_CONTACTS.CONTACTS_TITLE'|tu }}`), когда нужна переведённая или вычисляемая подпись вместо статической строки.

> [!NOTE] Примечание
> Admin Next рендерит заголовок редактирования небольшим подмножеством Twig, а не полным движком. Он поддерживает заполнители `{{ object.<field> ?? 'fallback' }}` (сколько угодно, в сочетании с обычным текстом) и необязательную цепочку фильтров, например `|tu` или `|t`. Значение по умолчанию переводится так же, как ведёт себя `|tu`: языковой ключ вроде `'PLUGIN_NEWS.CREATE_NEWS'` переводится или очеловечивается, а обычная строка используется как есть. Всё более сложное — условия, вложенные свойства, другие переменные — не вычисляется и отрендерилось бы буквально.

### Меню

По умолчанию включённая директория автоматически регистрируется в боковой панели Admin Next со своими `title` и `icon`. Добавьте блок `menu.list`, чтобы настроить пункт или изменить его положение:

```yaml
config:
  admin:
    menu:
      list:
        route: '/contacts'   # маршрут для пункта боковой панели
        title: Contacts
        icon: fa-address-card
        # Разрешение, необходимое для отображения пункта
        authorize: ['admin.contacts.list', 'admin.super']
        # Приоритет от -10 до 10 (чем больше, тем выше)
        priority: 2
        # Исключить эту директорию из общей авторегистрации Flex,
        # например если плагин поставляет собственный пункт боковой панели.
        hidden_in_admin_next: false
```

> [!NOTE] Примечание
> Задайте `hidden_in_admin_next: true`, когда плагин регистрирует собственный пункт боковой панели Admin Next для директории и вы не хотите, чтобы появлялся ещё и общий пункт Flex Objects. Директории также пропускаются в автоматической боковой панели, если пользователь не проходит проверку разрешения `list` для этой директории.

### Экспорт

Каждый объект директории можно экспортировать в один файл. Настройте метод, форматтер и имя файла:

```yaml
config:
  admin:
    export:
      enabled: true
      method: 'jsonSerialize'
      formatter:
        class: 'Grav\Framework\File\Formatter\YamlFormatter'
      filename: 'contacts'
```

Это обеспечивает работу эндпоинта `GET /flex-objects/{type}/export` (см. [справочник REST API](../../../../api/endpoints/flex-objects/index.md)).

### Только для классической админки, в Admin Next игнорируется

Некоторые ключи формата схем Grav 1.7 были нацелены на старую серверную админку на Vue. Они по-прежнему разбираются без ошибок для обратной совместимости, но Admin Next их не читает. Не полагайтесь на них:

| Ключ | Что делал в классической админке | Состояние в Admin Next |
|-----|------------------------|----------------------|
| `admin.router` (`path`, `actions`, `redirects`) | Пользовательские URL админки и перенаправления для директории. | Игнорируется. Маршруты Admin Next фиксированы: `/flex-objects/{type}`. |
| `admin.views.preview` | Рендерил страницу фронтенда как предпросмотр в админке. | Игнорируется. Используйте вместо этого тип страницы фронтенда или шорткод `[flex-objects]`. |
| `list.fields.*.title_class` / `data_class` | Подсказки CSS-стилей VueTable для заголовка и ячеек данных. | Игнорируется. Admin Next рендерит собственные стили таблицы. |

## Права доступа {: #permissions }
> [!IMPORTANT] Важно
> Разрешения — самая частая причина обращений в поддержку вида «моя директория возвращает 403». Прочитайте весь этот раздел, прежде чем выпускать директорию.

Директория объявляет свои разрешения в `config.admin.permissions` с ключом по **префиксу**. Каждый префикс разворачивается в разрешения для отдельных действий (`<prefix>.list`, `<prefix>.create`, `<prefix>.read`, `<prefix>.update`, `<prefix>.delete`), которые появляются в редакторах пользователей и групп и проверяются Admin Next и REST API.

Объявляйте **оба** префикса: `admin.<type>` (устаревший префикс Grav 1.7 для перенесённых учётных записей и классической админки) **и** `api.<type>` (префикс Grav 2.0, по которому фактически предоставляют доступ Admin Next и REST API). Слой API проверяет каждый объявленный префикс по логике ИЛИ, поэтому разрешение по любому из префиксов авторизует действие.

```yaml
config:
  admin:
    permissions:
      # Устаревший префикс (Grav 1.7 / классические учётные записи). Соответствует api.* один к одному
      admin.contacts:
        type: crudpl   # создание, чтение, обновление, удаление, публикация, список
        label: Contacts
      # Префикс Grav 2.0 (разрешения Admin Next + REST API)
      api.contacts:
        type: crudpl
        label: Contacts API
```

Сокращение `type` разворачивается в набор действий: `crudl` даёт create, read, update, delete и list; `crudpl` добавляет publish. Flex проверяет действия `list`, `create`, `read`, `update` и `delete`.

### Отказ по умолчанию начиная с flex-objects 1.4.3 {: #default-deny-since-flex-objects-1-4-3 }

> [!WARNING] Предупреждение
> Начиная с flex-objects **1.4.3** (исправление безопасности [GHSA-23vq-365v-qcmh](https://github.com/trilbymedia/grav-plugin-flex-objects/security/advisories)) директория, схема которой не определяет **блок** `permissions`, **закрыта для всех, кроме суперадминистраторов, по REST API** (отказ по умолчанию). Если ваша директория возвращает 403 редакторам, но работает у вас, почти всегда причина — отсутствующий блок `permissions`. Добавьте префиксы `admin.<type>` + `api.<type>`, показанные выше, и разрешения начнут работать.

### Суперадминистраторы

Суперадминистраторы полностью обходят проверки для отдельных директорий. Есть две суперобласти, и любая из них даёт полный доступ:

* Сессии **Admin Next** несут `api.super`.
* **Классические** учётные записи несут `admin.super`.

Перенесённый классический суперадминистратор может иметь `admin.super`, но не `api.super`, и наоборот — поэтому API принимает любой из них.

### Предоставление доступа редактору

Чтобы дать не-супер-редактору доступ к директории, назначьте ему конкретные действия, которые ему нужны, по префиксу `api.<type>`. Для доступа только на чтение:

```yaml
access:
  api:
    contacts:
      list: true
      read: true
```

Добавьте `create`, `update` и `delete` по мере необходимости для роли. Пользователь также должен иметь `api.access` (общее разрешение «эта учётная запись может использовать REST API»); без него каждый запрос Flex отклоняется ещё до проверки для отдельной директории.

> [!NOTE] Примечание
> Можно также объявлять вторичные разрешения (например, префикс `admin.configuration.contacts` для вкладки настроек уровня директории). Они не проверяются автоматически; на них нужно ссылаться самостоятельно из правила `authorize` соответствующего вида.

## Config: Site

Раздел `site` управляет тем, как фронтенд ищет шаблоны Twig для этого типа. По умолчанию макеты коллекций и объектов разрешаются из `flex/{TYPE}/collection/{LAYOUT}` и `flex/{TYPE}/object/{LAYOUT}`:

```yaml
config:
  site:
    templates:
      collection:
        # Поиск файлов макета коллекции (списка)
        paths:
          - 'flex/{TYPE}/collection/{LAYOUT}{EXT}'
      object:
        # Поиск файлов макета одного объекта
        paths:
          - 'flex/{TYPE}/object/{LAYOUT}{EXT}'
      defaults:
        # {TYPE} по умолчанию; переопределяется именем файла схемы, если существует соответствующая папка шаблонов
        type: contacts
        # {LAYOUT} по умолчанию; может переопределяться в вызовах render (обычно из Twig)
        layout: default
```

`{TYPE}` по умолчанию равен типу директории, а `{LAYOUT}` — `default`. Оба можно переопределить во время рендеринга, например тегом `render` или шорткодом `[flex-objects]`. О том, как используются эти шаблоны, см. [Фронтенд](../../frontend/index.md).

## Blueprints

Раздел `blueprints` определяет параметры конфигурации уровня всей директории, редактируемые на вкладке конфигурации директории без ручного редактирования файлов. Поля имеют префикс пути конфигурации, в который они пишут (`object`, `collection` и т. д.):

```yaml
blueprints:
  # Схема для вида конфигурации
  configure:
    # Мы внутри поля TABS
    fields:
      compatibility:
        type: tab
        title: Compatibility
        fields:
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

Переключатель `object.compat.events` выше — настоящая настройка: он управляет тем, срабатывают ли `onAdminSave` / `onAdminAfterSave` при записи объектов, от чего зависят старые плагины. Полный список событий, которые может перехватывать автор плагина, см. в разделе [События](../../php-and-events/index.md).
