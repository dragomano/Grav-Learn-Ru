---
title: Коллекция Flex
page-toc:
  active: true
taxonomy:
    category: docs
---

**Коллекция Flex** — это **упорядоченная карта объектов**, которую также можно использовать как список.

Коллекция предоставляет несколько полезных методов, которые можно использовать для визуализации вывода, выборки объектов, фильтрации и сортировки и так далее.

! **СОВЕТ:** Коллекция Flex расширяет **[Коллекции Doctrine](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/index.html)**.

# Визуализация коллекции

## render()

`render( [layout], [context] ): Block` Визуализирует коллекцию.

Параметры:
- **layout** Название макета (`string`)
- **context** Дополнительные переменные, которые можно использовать внутри файла шаблона Twig (`array`)

Возвращает:
- **Block** (`object`) HTML-класс блока, содержащего вывод

!!! **ПРИМЕЧАНИЕ:** В twig есть тег `{% render %}`, который следует использовать вместо прямого вызова метода. Это позволит активам JS/CSS из коллекции работать правильно.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% set page = 2 %}
{% set limit = 10 %}
{% set start = (page - 1) * limit %}

<h2>Contacts:</h2>

{% render contacts.limit(start, limit) layout: 'cards' with { background: 'gray', color: 'white' } %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\ContentBlock\HtmlBlock;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$page = 2;
$limit = 10;
$start = ($page-1)*$limit;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->limit($start, $limit);

    /** @var HtmlBlock $block */
    $block = $collection->render('cards', ['background' =>'gray', 'color' => 'white']);

}
```
[/ui-tab]
[/ui-tabs]

# Манипуляции с коллекциями

Все эти методы возвращают **измененную копию** коллекции. Исходная коллекция остается без изменений.

## sort()

`sort( orderings ): Collection` Отсортируйте коллекцию по списку пар `свойство: направление` (`property: direction`).

Параметры:
- **orderings** Пары `property: direction`, где направление либо `ASC`, либо `DESC` (`array`)

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый отсортированный экземпляр коллекции

! **СОВЕТ:** Порядок сортировки по умолчанию может быть установлен для внешнего интерфейса в чертежах **Flex-типа**.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.sort({last_name: 'ASC', first_name: 'ASC'}) %}

<div>Displaying all contacts in alphabetical order:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->sort(['last_name' => 'ASC', 'first_name' => 'ASC']);
    // Collection has now be sorted by last name, first name...

}
```
[/ui-tab]
[/ui-tabs]

## limit()

`limit( start, limit ): Collection` Вернуть подмножество коллекции, начиная с `start` и включая только `limit` объектов.

Параметры:
- **start** Начальный индекс начиная с 0 (`int`)
- **limit** Максимальное количество объектов (`int`)

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый отфильтрованный экземпляр коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% set page = 3 %}
{% set limit = 6 %}
{% set start = (page - 1) * limit %}

{% set contacts = contacts.limit(start, limit) %}

<div>Displaying page {{ page|e }}:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->limit($start, $limit);
    // Collection contains only the objects in the current page...

}
```
[/ui-tab]
[/ui-tabs]

## filterBy()

`filterBy( filters ): Collection` Фильтрация коллекции по списку пар `свойство: значение` (`property: value`).

Параметры:
- **filters** Пары `property: value`, которые используются для фильтрации коллекции (`array`)

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый отфильтрованный экземпляр коллекции

! **СОВЕТ:** Фильтрацию по умолчанию можно установить для внешнего интерфейса в чертежах **Flex-типа**.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.filterBy({'published': true}) %}

<div>Displaying only published contacts:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->filterBy(['published' => true]);
    // Collection contains only published objects...

}
```
[/ui-tab]
[/ui-tabs]

## reverse()

`reverse(): Collection`  Поменять порядок объектов в коллекции на обратный.

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый перевернутый экземпляр коллекции

! **СОВЕТ:** Если вы используете `sort()`, рекомендуется изменить порядок в нем, поскольку это экономит дополнительный шаг.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.reverse() %}

<div>Displaying contacts in reverse ordering:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->reverse();
    // Collection is now in reverse ordering...

}
```
[/ui-tab]
[/ui-tabs]

## shuffle()

`shuffle(): Collection`  Перемешать объекты в случайном порядке.

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый случайно упорядоченный экземпляр коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.shuffle().limit(0, 6) %}

<div>Displaying 6 random contacts:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->shuffle()->limit(0,6);
    // Collection contains 6 random contacts...

}
```
[/ui-tab]
[/ui-tabs]

## select()

`select( keys ): Collection` Выбрать объекты (по их ключам) из коллекции.

Параметры:
- **keys** Список ключей объекта, используемых для выбора объектов (`array`)

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый экземпляр коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% set selected = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'] %}

{% set contacts = contacts.select(selected) %}

<div>Displaying 4 selected contacts:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$selected = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'];

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->select($selected);
    // Collection now contains the 4 selected contacts...

}
```
[/ui-tab]
[/ui-tabs]

## unselect()

`unselect( keys ): Collection` Удалять предметы (по их ключам) из коллекции.

Параметры:
- **keys** Список ключей объектов, используемых для удаления объектов (`array`)

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый экземпляр коллекции


[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% set ignore = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'] %}

{% set contacts = contacts.unselect(ignore) %}

<div>Displaying all but 4 ignored contacts:</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$ignore = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'];

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->unselect($ignore);
    // Collection now contains all but 4 ignored contacts...

}
```
[/ui-tab]
[/ui-tabs]

## search()

`search( string, [properties], [options] ): Collection` Поиск строки в коллекции.

Параметры:
- **string** Строка для поиска (`string`)
- **properties** Свойства для поиска, если `null` (или не указано), используйте значения по умолчанию (`array` или `null`)
- **options** Дополнительные параметры, используемые при поиске (`array`)
  - starts_with: `bool`
  - ends_with: `bool`
  - contains: `bool`
  - case_sensitive: `bool`

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый отфильтрованный экземпляр коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.search('Jack', ['first_name', 'last_name', 'email'], {'contains': true}) %}

<div>Displaying all search results for 'Jack':</div>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->search('Jack', ['first_name', 'last_name', 'email'], ['contains' => true]);
    // Collection now contains all search results...

}
```
[/ui-tab]
[/ui-tabs]

## copy()

`copy(): Collection` Создать копию из коллекции, клонировав все объекты в коллекции.

Возвращает:
- **[Collection](/advanced/flex/using/collection)** (`object`) Новый экземпляр коллекции, теперь с клонированными объектами

!! **ПРЕДУПРЕЖДЕНИЕ:** Если вы изменяете объекты в своей коллекции, вы всегда должны использовать копии!

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.shuffle().limit(0, 10) %}
{% set fakes = contacts.copy() %}

{% do fakes.setProperty('first_name', 'JACK') %}

<h2>Fake cards</h2>
{% render fakes layout: 'cards' %}

<h2>Original cards</h2>
{% render contacts layout: 'cards' %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->search('Jack', ['first_name', 'last_name', 'email'], ['contains' => true]);
    // Collection now contains all search results...

}
```
[/ui-tab]
[/ui-tabs]

# Итерация по коллекции

**Коллекции** можно перебирать.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

<h2>All contacts:</h2>
<ul>
  {% for contact in contacts %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface $object */
    foreach ($collection as $object) {
        // Do something with the object...
    }

}
```
[/ui-tab]
[/ui-tabs]

## first()

`first(): Object | false` Устанавливает итератор на первый объект в коллекции и возвращает этот объект.

Возвращает:
- **[Object](/advanced/flex/using/object)** (`object`) Первый объект
- `false` Никаких предметов в коллекции нет

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.first() %}

{% if contact %}
    <h2>First contact:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    $object = $collection->first();
    if ($object) {
        // Do something with the object...
    }

}
```
[/ui-tab]
[/ui-tabs]

## last()

`last(): Object | false` Устанавливает итератор на последний объект в коллекции и возвращает этот объект.

Возвращает:
- **[Object](/advanced/flex/using/object)** (`object`) Последний объект
- `false` Никаких предметов в коллекции нет


[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.last() %}

{% if contact %}
    <h2>Last contact:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    $object = $collection->last();
    if ($object) {
        // Do something with the object...
    }

}
```
[/ui-tab]
[/ui-tabs]

## next()

`next(): object | false` Перемещает позицию итератора к следующему объекту и возвращает этот элемент.

Возвращает:
- **[Object](/advanced/flex/using/object)** (`object`) Следующий объект
- `false` Больше никаких элементов в коллекции


[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% set first = contacts.first() %}
...

{% set contact = contacts.next() %}

{% if contact %}
    <h2>Next contact is:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    while ($object = $collection->next()) {
        // Do something with the object...
    }

}
```
[/ui-tab]
[/ui-tabs]

## current()

`current(): object | false` Возвращает объект коллекции в текущей позиции итератора.

Возвращает:
- **[Object](/advanced/flex/using/object)** (`object`) Текущий объект
- `false` Больше никаких элементов в коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% do contacts.next() %}
{% do contacts.next() %}
...

{% set contact = contacts.current() %}

{% if contact %}
    <h2>Current contact is:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {
    while ($collection->next()) {

        /** @var FlexObjectInterface|false $object */
        $object = $collection->current();
        // Do something with the object...

    }
}
```
[/ui-tab]
[/ui-tabs]

## key()

`key(): key | null` Возвращает ключ объекта в текущей позиции итератора.

Возвращает:
- **key** (`string`) Ключ объекта
- `null` Больше никаких элементов в коллекции

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}
{% do contacts.next() %}
{% do contacts.next() %}
...

{% set key = contacts.key() %}

{% if key %}
    Current contact key is: <strong>{{ key|e }}</strong>
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {
    while ($collection->next()) {

        $key = $collection->key();
        // Do something with the key...

    }
}
```
[/ui-tab]
[/ui-tabs]

# Получение объекта/ключа

## Доступ к массиву

**Коллекции** доступны так же, как ассоциативные массивы или карты.

!!! **ПРИМЕЧАНИЕ:** если объекта с данным ключом нет в коллекции, возвращается `null`.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts['ki2ts4cbivggmtlj']

{# Do something #}
{% if contact %}
  {# Got Bruce Day #}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|null $object */
    $object = $collection['ki2ts4cbivggmtlj'];
    if ($object) {
        // Object exists, do something with it...
    }

}
```
[/ui-tab]
[/ui-tabs]

## get()

`get( key ): Object | null` Получает объект с указанным ключом.

Параметры:
- **key**  Ключ объекта (`string`)

Возвращает:
- **Object** (`object`)
- `null` Объект с данным ключом отсутствует в коллекции

Альтернатива — [Доступ к массиву](#array-access)

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.get('ki2ts4cbivggmtlj')

{# Do something #}
{% if contact %}
  {# Got Bruce Day #}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|null $object */
    $object = $collection->get('ki2ts4cbivggmtlj');
    if ($object) {
        // Object exists, do something with it...
    }

}
```
[/ui-tab]
[/ui-tabs]

# Коллекция как массив

## getKeys()

`getKeys(): array` Получает все ключи коллекции.

Возвращает:
- `array` Список ключей

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set keys = contacts.keys() %}

Keys are: {{ keys|join(', ')|e }}
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var string[] $keys */
    $keys = $collection->getKeys();
    $keysList = implode(', ', $keys);

}
```
[/ui-tab]
[/ui-tabs]

## GetObjectKeys()

`GetObjectKeys(): array` Псевдоним метода `getKeys()`.

Возвращает:
- `array` Список ключей

## getValues()

`getValues(): array` Получает все объекты коллекции.

Преобразует коллекцию в массив. Ключи не сохраняются.

Возвращает:
- Список **объектов** (`array`)

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.values() %}
<ol>
{% for i,object in list %}
    <li>#{{ (i+1)|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface[] $objects */
    $objects = $collection->getValues();
    foreach ($objects as $pos => $object) {
        // Do something with the object and its position...
    }

}
```
[/ui-tab]
[/ui-tabs]

## toArray()

`toArray(): array` Получает собственное представление коллекции в виде массива PHP.

То же, что и `getValues​​()`, но с сохранением ключей.

Возвращает:
- `array` Список пар `key: Object`

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.toArray() %}
<ol>
{% for key,object in list %}
    <li>ID: {{ key|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array<string, FlexObjectInterface> $objects */
    $objects = $collection->toArray();
    foreach ($objects as $key => $object) {
        // Do something with the object and its key...
    }

}
```
[/ui-tab]
[/ui-tabs]

## slice()

`slice( offset, length ): array` Извлекает из коллекции `length` элементов, начиная с позиции `offset`.

Параметры:
- **offset** Начальное смещение, начиная с 0 (`int`)
- **length** Максимальное количество объектов (`int`)

Возвращает:
- `array` Список пар `key: Object`

! **СОВЕТ:** Этот метод можно использовать для нумерации страниц.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.slice(10, 5) %}

<div>Displaying 5 emails starting from offset 10:</div>
<ol>
{% for key,object in list %}
    <li>ID: {{ key|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array<string, FlexObjectInterface> $objects */
    $objects = $collection->slice(10, 5);

    // Do something with the object and its key...

}
```
[/ui-tab]
[/ui-tabs]

## chunk()

`chunk( size ): array` Разбить коллекцию на части с максимальной вместимостью `size` объектов в каждой.

Параметры:
- **size** Размер частей (`int`)

Возвращает:
- `array` Двумерный список пар `key: Object`

! **СОВЕТ:** Этот метод можно использовать для разделения содержимого на строки и столбцы.

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set columns = contacts.limit(0, 10).chunk(5) %}

<div>Displaying two columns of 5 emails each:</div>
<div class="columns">
{% for column,list in columns %}
    <div class="column">
    {% for object in list %}
        <div>{{ object.email|e }}</div>
    {% endfor %}
    </div>
{% endfor %}
</div>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array $columns */
    $columns = $collection->limit(0, 10)->chunk(5);
    /** @var
        int $column
        array<string, FlexObjectInterface> $objects
    */
    foreach ($columns as $column => $objects) {
        // Do something with the objects...
    }
}

}
```
[/ui-tab]
[/ui-tabs]

## group()

`group( property ): array` Сгруппировать объекты в коллекции по свойству и вернуть их в виде связанного массива.

Параметры:
- **property** Имя свойства, которое используется для группировки объектов (`string`)

Возвращает:
- `array` Двумерный список пар `key: Object`, где значение свойства является ключом первого уровня

[ui-tabs]
[ui-tab title="Twig"]
```twig
{% set contacts = grav.get('flex').collection('contacts') %}

{% set by_name = contacts.sort({last_name: 'ASC', first_name: 'ASC'}).group('last_name') %}

<div>Displaying contacts grouped by last name:</div>
<div>
{% for last_name,list in by_name %}
    {{ last_name|e }}:
    <ul>
    {% for object in list %}
        <li>{{ object.first_name|e }}</li>
    {% endfor %}
    </ul>
{% endfor %}
</div>
```
[/ui-tab]
[ui-tab title="PHP"]
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array $byName */
    $byName = $collection->group('last_name');
    /** @var
        string $lastName
        array<string, FlexObjectInterface> $objects
    */
    foreach ($byName as $lastName => $objects) {
        // Do something with the objects...
    }
}

}
```
[/ui-tab]
[/ui-tabs]

# Добавление и удаление объектов

## add()

`add( Object )` Добавляет объект в конец коллекции.

Параметры:
- **[Object](/advanced/flex/using/object)** Объект для добавления (`object`)

## remove()

`remove( key ): Object | null` Удаляет элемент с указанным индексом из коллекции.

Параметры:
- **key** Ключ объекта для удаления (`object`)

Возвращает:
- **[Object](/advanced/flex/using/object)** Удаленный объект (`object`) или `null`, если объекта не существует

## removeElement()

`removeElement( Object ): bool` Удаляет указанный объект из коллекции, если он найден.

Параметры:
- **[Object](/advanced/flex/using/object)** Объект, подлежащий удалению (`object`)

Возвращает:
- `true` если объект был в коллекции, `false` если нет

## clear()

`clear()` Очищает коллекцию, удаляя все элементы.

# Тесты

## containsKey()

`containsKey( key ): bool` Проверяет, содержит ли коллекция объект с указанным ключом.

Параметры:
- **key** Ключ для проверки (`string`)

Возвращает:
- `true` если объект находится в коллекции, `false` если нет

## contains()

`contains( object ): bool` Проверяет, содержится ли элемент в коллекции.

Параметры:
- **[Object](/advanced/flex/using/object)** Объект для тестирования (`object`)

Возвращает:
- `true` если объект находится в коллекции, `false` если нет

## indexOf()

`indexOf( object ): string | false` Возвращает индекс/ключ данного объекта.

Параметры:
- **[Object](/advanced/flex/using/object)** Объект для тестирования (`object`)

Возвращает:
- `string` индекс/ключ объекта, `false` если объект не был найден

## isEmpty()

`isEmpty(): bool` Проверяет, пуста ли коллекция (не содержит объектов).

Возвращает:
- `true` если коллекция пуста, `false` в противном случае

## count()

`count(): int`

Возвращает:
- `int` Количество объектов в коллекции

# Массовые действия для объектов

## hasProperty()

`hasProperty( property ): array` Возвращает список пар `key: boolean` независимо от того, имеет ли объект с ключом определенное свойство или нет.

Параметры:
- **property** Имя свойства (`string`)

Возвращает:
- `array` пар `key: bool`, где `key` — ключ объекта, а `bool` либо `true`, либо `false`.

## getProperty()

`getProperty( property, default ): array` Возвращает список `key: value` для каждого объекта.

Параметры:
- **property** Имя свойства (`string`)

Возвращает:
- `array` пар `key: value`, где `key` — ключ объекта, а `value` — значение свойства.

## setProperty()

`setProperty( property, value ): Collection` Установить новое значение свойства для каждого объекта в коллекции.

Параметры:
- **property** Имя свойства (`string`)
- **value** Новое значение (`mixed`)

Возвращает:
- **Collection** (`object`) Коллекция для цепочки вызовов методов.

!! **ПРЕДУПРЕЖДЕНИЕ:** Этот метод изменяет экземпляры объектов, общие для всех коллекций, если это не предусмотрено, пожалуйста, [copy()](#copy) коллекцию перед использованием этого метода.

## defProperty()

`defProperty( property, default ): Collection` Определить значение по умолчанию для свойства для каждого объекта в коллекции.

Параметры:
- **property** Имя свойства (`string`)
- **default** Значение по умолчанию (`mixed`)

Возвращает:
- **Collection** (`object`) Коллекция для цепочки вызовов методов.

!! **ПРЕДУПРЕЖДЕНИЕ:** Этот метод изменяет экземпляры объектов, общие для всех коллекций, если это не предусмотрено, пожалуйста, [copy()](#copy) коллекцию перед использованием этого метода.

## unsetProperty()

`unsetProperty( property ): Collection` Удалить значение свойства для каждого объекта в коллекции.

Параметры:
- **property** Имя свойства (`string`)

Возвращает:
- **Collection** (`object`) Коллекция для объединения вызовов методов.

!! **ПРЕДУПРЕЖДЕНИЕ:** Этот метод изменяет экземпляры объектов, общие для всех коллекций, если это не предусмотрено, пожалуйста, [copy()](#copy) коллекцию перед использованием этого метода.

## call()

`call( method, arguments): array` Вызывает метод для каждого объекта в коллекции. Возвращает результаты каждого вызова.

Параметры:
- **method** Имя метода (`string`)
- **arguments** Список аргументов (`array`)

Возвращает:
- Список пар `key: result` (`array`)

!! **ПРЕДУПРЕЖДЕНИЕ:** Если метод изменяет объект, пожалуйста, [copy()](#copy) коллекцию перед использованием этого метода.

## getTimestamps()

`getTimestamps(): array`  Возвращает список пар `key: timestamp` для каждого объекта.

Возвращает:
- Список пар `key: timestamp`, где timestamp — целое число (`array`)

## getStorageKeys()

`getStorageKeys(): array`  Возвращает список пар `key: storage_key` для каждого объекта.

Возвращает:
- Список пар `key: storage_key` (`array`)

## getFlexKeys()

`getFlexKeys(): array`  Возвращает список пар `key: flex_key` для каждого объекта.

Возвращает:
- Список пар `key: flex_key` (`array`)

## withKeyField()

`withKeyField( field ): Collection` Return new collection with a different key.

Параметры:
- **field** Ключ поля (`string`)
  - 'key': Ключ по умолчанию, используемый каталогом
  - 'storage_key': Storage layer key
  - 'flex_key': Уникальный ключ, который можно использовать, не зная каталога

Возвращает:
- **Collection** (`object`) Коллекция, проиндексированная новым ключом.

# Closure Tests (только PHP)

## exists()

`exists( Closure ): bool` Проверяет существование объекта, удовлетворяющего заданному предикату.

Параметры:
- **Closure** Метод, используемый для проверки каждого объекта.

Возвращает:
- `bool` `true`, если ваша функция обратного вызова возвращает `true` для всех объектов.

## forAll()

`forAll( Closure ): bool` Проверяет, выполняется ли данный предикат для всех объектов этой коллекции.

Параметры:
- **Closure** Метод, используемый для проверки каждого объекта.

Возвращает:
- `bool` `true`, если ваша функция обратного вызова возвращает `true` для всех объектов.

# Closure Filtering (PHP only)

## filter()

`filter( Closure ): Collection` Возвращает все объекты этой коллекции, удовлетворяющие предикату.

Порядок элементов сохраняется.

Параметры:
- **Closure** Метод, используемый для проверки отдельного объекта.

Возвращает:
- **Collection** (`object`) Новая коллекция со всеми объектами, для которых функция обратного вызова возвращает `true`.

## map()

`map( Closure ): Collection` Применяет данную функцию к каждому объекту в коллекции и возвращает новую коллекцию с объектами, возвращаемыми функцией.

Параметры:
- **Closure** Метод, используемый для проверки отдельного объекта.

Возвращает:
- **Collection** (`object`) Новая коллекция с объектами, возвращаемыми функцией обратного вызова.


## collectionGroup()

`collectionGroup( property ): Collection[]` Сгруппируйте объекты в коллекции по полю и верните их как связанный массив коллекций.

Параметры:
- **property** (`string`) Свойство, используемое для группировки объектов.

Возвращает:
- `array` Несколько коллекций в массиве, ключом является значение свойства.

## matching()

`matching( Criteria ): Collection` Выбирает все объекты, соответствующие выражению, и возвращает новую коллекцию, содержащую эти объекты.

Параметры:
- **[Criteria](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expression-builder.html#expression-builder)** Выражение

Возвращает:
- **Collection** (`object`) Новая коллекция с объектами, соответствующими критериям.

! **СОВЕТ:** См. документацию Doctrine о **[Конструкторе выражений](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expression-builder.html#expression-builder)** и ** [Выражениях](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expressions.html#expressions)**.

## orderBy()

`orderBy( array ): Collection` Изменить порядок коллекции по списку пар свойство/значение.

Параметры:
- `array`

Возвращает:
- **Collection** (`object`) Новая коллекция с новым порядком.

## partition()

`partition( Closure ): array` Разбивает эту коллекцию на две коллекции в соответствии с предикатом.

Ключи сохраняются в полученных коллекциях.

Параметры:
- **Closure** Метод, используемый для разделения одного объекта. Возвращает `true` или `false`.

Возвращает:
- `array` Секционированные объекты `[[a, b], [c, d, e]]`
