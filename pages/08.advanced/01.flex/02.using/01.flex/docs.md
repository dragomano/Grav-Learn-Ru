---
title: Flex
metadata:
    description: 'Список общих методов для работы с Flex-объектами в Grav CMS.'
page-toc:
  active: true
taxonomy:
  category: docs
---

! **СОВЕТ:** Полный список методов можно найти в разделе **Настройка объектов Flex**.

## count()

`count(): int` Подсчёт количества каталогов, зарегистрированных во Flex.

Возвращает:
- `int` Количество **[Каталогов](/advanced/flex/using/directory)**

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

Flex has {{ flex.count() }} enabled directories.
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var int $count */
$count = $flex->count();
```
[/ui-tab]
[/ui-tabs]

## getDirectories()

`getDirectories( [names] ): array` Получить список каталогов.

Параметры:
- **names** Необязательно: список имен каталогов (`array`)

Возвращает:
- `array` список **[Каталогов](/advanced/flex/using/directory)**

! **СОВЕТ:** Если список имен не был предоставлен, метод возвращает все каталоги, зарегистрированные во Flex.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

{# Get all directories #}
{% set directories = flex.directories() %}

{# Get listed directories #}
{% set listed_directories = flex.directories(['contacts', 'phonebook']) %}

{# Do something with the directories #}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexDirectoryInterface[] $directories */
$directories = $flex->getDirectories();
// = ['contacts' => FlexDirectory, ...]

/** @var FlexDirectoryInterface[] $directories */
$listedDirectories = $flex->getDirectories(['contacts', 'phonebook']);
// = ['contacts' => FlexDirectory]

/** @var array<FlexDirectoryInterface|null> $directories */
$listedDirectoriesWithMissing = $flex->getDirectories(['contacts', 'phonebook'], true);
// = ['contacts' => FlexDirectory, 'phonebook' => null]
```
[/ui-tab]
[/ui-tabs]

! **СОВЕТ:** Вы можете убедиться, что возвращаете только те каталоги, которые хотите.

## hasDirectory()

`hasDirectory( name ): bool`: Проверить, существует ли каталог.

Параметры:
- **name** Имя каталога (`string`)

Возвращает:
- `bool` `true`, если найден, иначе `false`

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

Flex has {{ not flex.hasDirectory('contacts') ? 'not' }} contacts directory.
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var bool $exists */
$exists = $flex->hasDirectory('contacts');
```
[/ui-tab]
[/ui-tabs]

## getDirectory()

`getDirectory( name ): Directory | null` Получить каталог, возвращает `null`, если он не был найден.

Параметры:
- **name** Имя каталога (`string`)

Возвращает:
- **[Каталог](/advanced/flex/using/directory)** (`object`)
- `null` Каталог не найден

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

{# Get contacts directory (null if not found) #}
{% set directory = flex.directory('contacts') %}

{# Do something with the contacts directory #}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexDirectoryInterface|null $directory */
$directory = $flex->getDirectory('contacts');
if ($directory) {
    // Directory exists, do something with it...
}
```
[/ui-tab]
[/ui-tabs]

!!! Проверьте, что вы можете сделать с **[каталогом Flex](/advanced/flex/using/directory)**

## getObject()

`getObject( id, directory ): Object | null` Получить объект, возвращает `null`, если он не был найден.

Параметры:
- **id** ID объекта (`string`)
- **directory** Имя каталога (`string`)

Возвращает:
- **[Объект](/advanced/flex/using/object)** (`object`)
- `null` Объект не найден

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

{% set contact = flex.object('ki2ts4cbivggmtlj', 'contacts') %}

{# Do something #}
{% if contact %}
  {# Got Bruce Day #}
  {{ contact.first_name|e }} {{ contact.last_name|e }} has a website: {{ contact.websize|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexObjectInterface|null $object */
$object = $flex->getObject('ki2ts4cbivggmtlj', 'contacts');
if ($object) {
    // Object exists, do something with it...
}
```
[/ui-tab]
[/ui-tabs]

!!! Проверьте, что вы можете сделать с **[объектом Flex](/advanced/flex/using/object)**

## getCollection()

`getCollection( directory ): Collection | null` Получить коллекцию, возвращает `null`, если она не найдена.

Параметры:
- `directory` Имя каталога (`string`)

Возвращает:
- **[Коллекция](/advanced/flex/using/collection)** (`object`)
- `null` Каталог не найден

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set flex = grav.get('flex') %}

{% set contacts = flex.collection('contacts') %}

{# Do something #}
<h2>Ten random contacts:</h2>
<ul>
  {% for contact in contacts.filterBy({published: true}).shuffle().limit(0, 10) %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexCollectionInterface|null $collection */
$collection = $flex->getCollection('contacts');
if ($collection) {
    // Collection exists, do something with it...
}
```
[/ui-tab]
[/ui-tabs]

!!! Проверьте, что вы можете сделать с **[коллекцией Flex](/advanced/flex/using/collection)**
