---
title: "Расширенные фичи схем"
description: "В схемах есть расширенные фичи, которые позволяют расширять их и использовать динамические поля."
tags:
  - "Формы"
  - "Схемы"
---

# Расширенные фичи схем

В схемах есть расширенные фичи, которые позволяют расширять их и использовать динамические поля.

## Определение правил валидации

Если одни и те же правила валидации нужны несколько раз, можно создать собственное правило.

```yaml linenums="1"
rules:
  slug:
    pattern: "[a-z][a-z0-9_\-]+"
    min: 2
    max: 80
form:
  fields:
    folder:
      type: text
      label: Folder Name
      validate:
        rule: slug
```

Пример выше создаёт правило `slug`, которое затем используется в поле folder формы.

## Расширение базового типа (extends@) {: #extending-base-type-extendsat }

Можно расширять существующую схему — это позволяет добавлять новые поля, а также изменять существующие из базовой схемы.

```yaml linenums="1"
extends@: default
```

В расширенном формате можно указать контекст поиска базового файла:

```yaml linenums="1"
extends@:
  type: default
  context: blueprints://pages
```

Схему можно расширять и саму по себе, если есть несколько версий одной и той же схемы.

```yaml linenums="1"
extends@: parent@
```

Число схем, которые можно расширять, не ограничено. Поля, определённые в первой схеме, будут заменены последующими схемами в списке.

```yaml linenums="1"
extends@:
  - parent@
  - type: default
    context: blueprints://pages
```

### О свойствах type и context

В примерах выше `type` ссылается на файл, а `context` — на путь. Свойство `context` использует [потоки](https://learn.getgrav.org/advanced/multisite-setup#streams), то есть разрешается в физическое расположение.

`context: blueprints://` по умолчанию даёт `/user/plugins/admin/blueprints` — папку схем Admin. `type: default` при поиске файлов даёт `default.yaml`. Поскольку эти два свойства используются вместе, они дают полный путь, понятный Grav: `/user/plugins/admin/blueprints/default.yaml`.

Если в этой документации вы видите синтаксис `://`, почти наверняка речь о потоке. А при использовании `context` этот поток должен указывать на существующую папку, иначе ничего не сработает.

## Встраивание формы (import@) {: #embedding-form-importat }

Иногда нужно совместно использовать часть полей или подформ в нескольких формах.

Создадим `blueprints://partials/gallery.yaml`, который хотим встроить в форму:

```yaml linenums="1"
form:
  fields:
    gallery.images:
      type: list
      label: Images
      fields:
        .src:
          type: text
          label: Image
```

В нашей форме тогда есть раздел, куда мы хотим встроить изображения галереи:

```yaml linenums="1"
form:
  fields:
    images:
        type: section
        title: Images
        underline: true
        import@:
          type: partials/gallery
          context: blueprints://
```

Хотя YAML не позволяет использовать один и тот же ключ `import@` несколько раз, можно импортировать несколько схем, добавив после `@` уникальное число, например `import@1`, `import@2` и так далее. Число не имеет другого смысла, кроме предотвращения ошибки парсера YAML:

```yaml linenums="1"
form:
  fields:
    images:
        type: section
        title: Images
        underline: true
        import@1:
          type: partials/gallery
          context: blueprints://
        import@2:
          type: partials/another-gallery
          context: blueprints://
```

По умолчанию `blueprints://` разрешается в `/user/plugins/admin/blueprints/`, поэтому учтите: если вы работаете в контексте темы, контекст инструкции импорта нужно скорректировать:

```yaml linenums="1"
form:
  fields:
    images:
        type: section
        title: Images
        underline: true
        import@:
          type: partials/gallery
          context: theme://blueprints
```

## Удаление полей и свойств (unset-*@) {: #removing-fields-properties-unset-at }

Если нужно удалить поле, добавьте внутрь него `unset@: true`.
Если нужно удалить свойство поля, просто добавьте имя свойства, например `unset-options@` удаляет все options.

## Замена полей и свойств (replace-*@) {: #replacing-fields-properties-replace-at }

По умолчанию схемы глубоко объединяют свойства. Иногда вместо объединения содержимого поля нужно начать с чистой таблицы.
Если нужно заменить всё поле, новое поле должно начинаться с `replace@`:

```yaml linenums="1"
author.name:
  replace@: true
  type: text
  label: Author name
```

В результате у `author.name` будет только два свойства: `type` и `label`, независимо от того, что было в форме раньше.
То же можно сделать для отдельных свойств:

```yaml linenums="1"
summary.enabled:
  replace-options@: true
  options:
    0: Yeah
    1: Nope
    2: Do not care
```

Примечание: `replace-*@` — псевдоним для `unset-*@`.

## Использование конфигурации (config-*@) {: #using-configuration-config-at }

Иногда нужно получить значение по умолчанию из конфигурации Grav. Например, можно сделать так, чтобы поле автора по умолчанию содержало автора сайта:

```yaml linenums="1"
form:
  fields:
    author:
      type: text
      label: Author
      config-default@: site.author.name
```

Если имя автора сайта — `John Doe`, форма эквивалентна:

```yaml linenums="1"
form:
  fields:
    author:
      type: text
      label: Author
      default: "John Doe"
```

`config-*@` можно использовать для любого поля; например, чтобы менять `type` поля, достаточно указать `config-type@: site.forms.author.type` и менять тип поля ввода из конфигурации.

## Использование вызовов функций (data-*@) {: #using-function-calls-data-at }

Можно вызывать функции с параметрами из схем, чтобы динамически получать значение для любого свойства поля. Для этого используется запись `data-*@:` в качестве ключа, где `*` — имя поля, которое нужно заполнить результатом вызова функции.

Например, мы редактируем страницу и хотим иметь поле, позволяющее менять её родителя, иными словами перемещать страницу в другое место. Для этого нужно значение по умолчанию, указывающее на текущее расположение, а также список вариантов из всех возможных расположений. Нам нужен способ спросить Grav:

```yaml linenums="1"
form:
  fields:
    route:
      type: select
      label: Parent
      classes: fancy
      data-default@: '\Grav\Plugin\Admin::route'
      data-options@: '\Grav\Common\Page\Pages::parentsRawRoutes'
      options:
        '/': '- Root -'
```

Если вы редактировали страницу участника команды, получившаяся форма выглядела бы примерно так:

```yaml linenums="1"
form:
  fields:
    route:
      type: select
      label: Parent
      classes: fancy
      default: /team
      options:
        '/': '- Root -'
        '/home': 'Home'
        '/team': 'Team'
        '/team/ceo': '  Meet Our CEO'
        ...
```

Хотя `data-default@:` и `data-options@:` — вероятно, самые используемые динамические свойства полей, ими дело не ограничивается. Нет ограничений на то, какие свойства можно получать, включая `type`, `label`, `validation` и даже `fields` текущего поля.

Кроме того, в вызов функции можно передавать параметры, используя массив, где первое значение — имя функции, а дальше идут параметры:

```yaml linenums="1"
data-default@: ['\Grav\Theme\ImaginaryClass::getMyDefault', 'default', false]
```

### Регистрация собственных поставщиков данных (Grav 2.0) {: #registering-custom-data-providers-grav-2-0 }

> [!IMPORTANT] Важно
> Начиная с Grav 2.0.11, вызываемые объекты `Class::method`, используемые в директивах `data-*@`, проверяются по списку разрешённых поставщиков данных. Собственные поставщики ядра Grav (например, `\Grav\Common\Page\Pages::pageTypes` и `\Grav\Common\Utils::timezones`) предварительно разрешены, но собственный поставщик в вашем плагине или теме должен быть зарегистрирован, прежде чем Grav его вызовет. Это особенно важно в Admin 2.0, который получает динамические параметры через [эндпоинт Resolve Data API](../../../api/endpoints/blueprints/index.md): незарегистрированный вызываемый объект отклоняется с ошибкой `Callable '...' is not an approved data provider.`

Чтобы зарегистрировать поставщика, один раз при запуске вызовите `Blueprint::addAllowedDynamicCallable()`. В теме естественное место — `onThemeInitialized`:

```php linenums="1"
<?php
namespace Grav\Theme;

use Grav\Common\Data\Blueprint;
use Grav\Common\Theme;

class MyTheme extends Theme
{
    public static function getSubscribedEvents(): array
    {
        return [
            'onThemeInitialized' => ['onThemeInitialized', 0],
        ];
    }

    public function onThemeInitialized(): void
    {
        Blueprint::addAllowedDynamicCallable('Grav\Theme\MyTheme\Utils::getIcons');
    }
}
```

В плагине сделайте то же самое в обработчике `onPluginsInitialized`. После регистрации вызываемый объект работает везде, где может ссылаться директива `data-*@`: в схемах, разрешаемых на сервере, и в полях select Admin 2.0, которые получают параметры через API.

Список разрешённых применяется только к вызываемым объектам, указанным из недоверенных источников, например к форме, определённой во frontmatter страницы, или к объекту, переданному в API как параметр запроса. Файлы схем, поставляемые с плагином или темой, считаются доверенными для серверного рендеринга во фронтенде, но для того, чтобы Admin мог их разрешить, регистрация поставщика всё равно требуется, поэтому всегда регистрируйте собственных поставщиков.

> [!NOTE] Примечание
> Класс поставщика также должен быть автозагружаемым в момент разрешения вызываемого объекта, иначе API ответит `Callable '...' not found.` Admin 2.0 разрешает данные схем через фронтенд-конвейер `/api` сайта, где `isAdmin()` всегда `false`, поэтому загрузка или регистрация класса, обёрнутая в условие «только для админки», там никогда не выполнится. Настройте правильную PSR-4-автозагрузку темы или плагина (файл `composer.json` с разделом `autoload` плюс метод `autoload()` в основном классе, возвращающий `require __DIR__ . '/vendor/autoload.php';`) и никогда не привязывайте настройку, связанную со схемами, к `isAdmin()`.

## Изменение порядка полей

При расширении схемы или импорте файла новые поля по умолчанию добавляются в конец списка. Иногда это не то, что нужно: возможно, элемент должен идти первым или после какого-то существующего поля.

Порядок поля можно задать свойством `ordering@`. Оно может содержать имя поля или целое число (-1 = первый элемент).

Вот пример:

```yaml linenums="1"
form:
  fields:
    route:
      ordering@: -1
      type: select
      label: Parent
      classes: fancy
      default: /team
      options:
        '/': '- Root -'
        '/home': 'Home'
        '/team': 'Team'
        '/team/ceo': '  Meet Our CEO'
        ...
```

Так поле route гарантированно будет первым в форме. Это позволяет легко импортировать и/или расширять существующее поле и размещать дополнительные поля там, где нужно.

Ещё один пример:

```yaml linenums="1"
form:
  fields:
    author:
      ordering@: header.title
      type: text
      label: Author
      default: "John Doe"
```

В примере выше мы использовали имя другого поля, чтобы задать порядок. Здесь сделано так, чтобы поле `author` шло после поля `title` в форме.

> [!CAUTION] Внимание
> При задании порядка полей в схеме страницы по-прежнему нужно ссылаться на имена полей с префиксом `header.`, например `header.title`, чтобы порядок работал.

## Создание нового типа поля формы

Если вы создаёте особый тип поля формы, которому нужна специальная обработка в схемах, можно использовать функцию плагина.

```php linenums="1"
/**
 * Получить список типов полей формы, указанных в этом плагине.
 * Перечислять нужно только особые типы.
 *
 * @return array
 */
public function getFormFieldTypes()
{
    return [
        'display' => [
            'input@' => false
        ],
        'spacer' => [
            'input@' => false
        ]
    ];
}
```

Регистрировать эту функцию не нужно: это не совсем событие, она вызывается при создании объекта плагина.
Её назначение — дать дополнительные инструкции по обработке поля; например, код выше делает типы display и spacer виртуальными, то есть они не существуют в реальных данных.

Можно добавлять любые пары `key: value`, включая динамические свойства вроде `data-options@`, которые автоматически добавляются к полям.

## Переопределение или расширение схемы плагина {: #override-or-extend-a-plugin-s-blueprint }

Иногда нужно изменить схему, предоставляемую плагином: добавить, переместить или удалить параметры. Это не так просто: схема плагина содержит не только свойство `form` и не объявлена расширяемой. Однако при создании плагинов стоит предусмотреть такую возможность для [пользовательских схем](../../../basics/folder-structure/index.md#user-blueprints).

- Во-первых, плагин должен объявить, что поддерживает схемы, добавив публичное свойство в свой PHP-файл: `public $features = ['blueprints' => 10];`
- Во-вторых, плагин должен `import@` свои поля формы из файла, например:

```yaml linenums="1"
form:
  validation: strict
  fields:
    tabs:
      type: tabs
      active: 1
      fields:
        import@:
          type: options
          context: blueprints://plugins/yourpluginname
```

Так импортируется `user/plugins/yourpluginname/blueprints/plugins/yourpluginname/options.yaml`.

- В-третьих, этот файл должен объявлять части формы по умолчанию:

```yaml linenums="1"
form:
  options:
    type: tab
    title: PLUGIN_ADMIN.OPTIONS
    fields:
      enabled:
        type: toggle
        label: PLUGIN_ADMIN.PLUGIN_STATUS
        default: 1
        options:
          1: PLUGIN_ADMIN.ENABLED
          0: PLUGIN_ADMIN.DISABLED
        validate:
          type: bool
```

> [!CAUTION] Внимание
> `context` и `type` должны быть в таком виде, чтобы избежать возможных конфликтов файлов и имён и оставаться легко идентифицируемыми — отсюда и кажущийся излишним длинный путь выше.

Затем пользователь может добавить свои изменения в `user/blueprints/plugins/yourpluginname/options.yaml`:

```yaml linenums="1"
form:
  options:
    fields:
      category:
        type: selectize
        label: Category
        validate:
          type: commalist
```

И они будут подхвачены на странице конфигурации плагина.

## onBlueprintCreated или доступ к данным схемы

Поскольку схемы состоят из полей с точками, получение вложенного поля из схемы использует нотацию `/` вместо `.`.

```php
$tabs = $blueprint->get('form/fields/tabs');
```

Это позволяет обращаться к особым полям данных, например:

```php
$name = $blueprint->get('form/fields/content.name');
$name = $blueprint->get('form/fields/content/fields/.name');
```

Для обратной совместимости можно указать разделитель в последнем (третьем) параметре `set()` и `get()`:

```php
$tabs = $blueprint->get('form/fields/tabs', null, '/');
```
