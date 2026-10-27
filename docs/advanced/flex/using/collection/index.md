---
title: "Коллекция Flex"
description: "Коллекция Flex — это упорядоченная карта объектов, которую также можно использовать как список."
tags:
  - "Flex-объекты"
  - "Twig"
---

# Коллекция Flex

**Коллекция Flex** — это **упорядоченная карта объектов**, которую также можно использовать как список.

Коллекция предоставляет несколько полезных методов для рендеринга вывода, получения объектов, фильтрации, сортировки и так далее.

> [!WARNING] Предупреждение
> **Совет:** коллекция Flex расширяет **[Doctrine Collections](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/index.html)**.

## Рендеринг коллекции

## render()

`render( [layout], [context] ): Block` Рендерит коллекцию.

Параметры:
- **layout** имя макета (`string`)
- **context** дополнительные переменные, доступные внутри файла шаблона Twig (`array`)

Возвращает:
- **Block** (`object`) класс HtmlBlock, содержащий вывод

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** в Twig есть тег `{% render %}`, который следует использовать вместо прямого вызова метода. Это позволит корректно работать JS/CSS-ресурсам коллекции.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% set page = 2 %}
{% set limit = 10 %}
{% set start = (page - 1) * limit %}

<h2>Contacts:</h2>

{% render contacts.limit(start, limit) layout: 'cards' with { background: 'gray', color: 'white' } %}
```

```php title="PHP"
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

## Работа с коллекцией

Все эти методы возвращают **изменённую копию** коллекции. Исходная коллекция остаётся неизменной.

## sort()

`sort( orderings ): Collection` Сортирует коллекцию по списку пар `property: direction`.

Параметры:
- **orderings** пары `property: direction`, где direction — `ASC` или `DESC` (`array`)

Возвращает:
- **[коллекцию](./)** (`object`) новый отсортированный экземпляр коллекции

> [!WARNING] Предупреждение
> **Совет:** порядок сортировки по умолчанию для фронтенда можно задать в схеме **типа Flex**.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.sort({last_name: 'ASC', first_name: 'ASC'}) %}

<div>Displaying all contacts in alphabetical order:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->sort(['last_name' => 'ASC', 'first_name' => 'ASC']);
    // Коллекция теперь отсортирована по фамилии, затем по имени...

}
```

## limit()

`limit( start, limit ): Collection` Возвращает подмножество коллекции, начиная с `start` и включая только `limit` объектов.

Параметры:
- **start** начальный индекс, начиная с 0 (`int`)
- **limit** максимальное число объектов (`int`)

Возвращает:
- **[коллекцию](./)** (`object`) новый отфильтрованный экземпляр коллекции

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% set page = 3 %}
{% set limit = 6 %}
{% set start = (page - 1) * limit %}

{% set contacts = contacts.limit(start, limit) %}

<div>Displaying page {{ page|e }}:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->limit($start, $limit);
    // Коллекция содержит только объекты текущей страницы...

}
```

## filterBy()

`filterBy( filters ): Collection` Фильтрует коллекцию по списку пар `property: value`.

Параметры:
- **filters** пары `property: value`, используемые для фильтрации коллекции (`array`)

Возвращает:
- **[коллекцию](./)** (`object`) новый отфильтрованный экземпляр коллекции

> [!WARNING] Предупреждение
> **Совет:** фильтрацию по умолчанию для фронтенда можно задать в схеме **типа Flex**.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.filterBy({'published': true}) %}

<div>Displaying only published contacts:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->filterBy(['published' => true]);
    // Коллекция содержит только опубликованные объекты...

}
```

## reverse()

`reverse(): Collection`  Меняет порядок объектов в коллекции на обратный.

Возвращает:
- **[коллекцию](./)** (`object`) новый экземпляр коллекции в обратном порядке

> [!WARNING] Предупреждение
> **Совет:** если вы используете `sort()`, рекомендуется задать обратный порядок прямо в нём — так вы сэкономите лишний шаг.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.reverse() %}

<div>Displaying contacts in reverse ordering:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$start = 0;
$limit = 6;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->reverse();
    // Коллекция теперь в обратном порядке...

}
```

## shuffle()

`shuffle(): Collection`  Перемешивает объекты в случайном порядке.

Возвращает:
- **[коллекцию](./)** (`object`) новый экземпляр коллекции в случайном порядке

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.shuffle().limit(0, 6) %}

<div>Displaying 6 random contacts:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->shuffle()->limit(0,6);
    // Коллекция содержит 6 случайных контактов...

}
```

## select()

`select( keys ): Collection` Выбирает объекты (по их ключам) из коллекции.

Параметры:
- **keys** список ключей объектов для выбора (`array`)

Возвращает:
- **[коллекцию](./)** (`object`) новый экземпляр коллекции

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% set selected = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'] %}

{% set contacts = contacts.select(selected) %}

<div>Displaying 4 selected contacts:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$selected = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'];

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->select($selected);
    // Коллекция теперь содержит 4 выбранных контакта...

}
```

## unselect()

`unselect( keys ): Collection` Удаляет объекты (по их ключам) из коллекции.

Параметры:
- **keys** список ключей объектов для удаления (`array`)

Возвращает:
- **[коллекцию](./)** (`object`) новый экземпляр коллекции

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% set ignore = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'] %}

{% set contacts = contacts.unselect(ignore) %}

<div>Displaying all but 4 ignored contacts:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$ignore = ['gizwsvkyo5xtms2s', 'gjmva53uoncdo4sb', 'mfzwwtcugv5hkocd', 'k5nfctkeoftwi4zu'];

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->unselect($ignore);
    // Коллекция теперь содержит все контакты, кроме 4 исключённых...

}
```

## search()

`search( string, [properties], [options] ): Collection` Ищет строку в коллекции.

Параметры:
- **string** строка поиска (`string`)
- **properties** свойства для поиска; если null (или не переданы), используются значения по умолчанию (`array` или `null`)
- **options** дополнительные параметры поиска (`array`)
  - starts_with: `bool`
  - ends_with: `bool`
  - contains: `bool`
  - case_sensitive: `bool`

Возвращает:
- **[коллекцию](./)** (`object`) новый отфильтрованный экземпляр коллекции

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.search('Jack', ['first_name', 'last_name', 'email'], {'contains': true}) %}

<div>Displaying all search results for 'Jack':</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->search('Jack', ['first_name', 'last_name', 'email'], ['contains' => true]);
    // Коллекция теперь содержит все результаты поиска...

}
```

## copy()

`copy(): Collection` Создаёт копию коллекции, клонируя все объекты в ней.

Возвращает:
- **[коллекцию](./)** (`object`) новый экземпляр коллекции, теперь с клонированными объектами

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** если вы изменяете объекты в коллекции, всегда используйте копии!

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.shuffle().limit(0, 10) %}
{% set fakes = contacts.copy() %}

{% do fakes.setProperty('first_name', 'JACK') %}

<h2>Fake cards</h2>
{% render fakes layout: 'cards' %}

<h2>Original cards</h2>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->search('Jack', ['first_name', 'last_name', 'email'], ['contains' => true]);
    // Коллекция теперь содержит все результаты поиска...

}
```

## Перебор коллекции

По **коллекциям** можно итерироваться.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

<h2>All contacts:</h2>
<ul>
  {% for contact in contacts %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface $object */
    foreach ($collection as $object) {
        // Делаем что-нибудь с объектом...
    }

}
```

## first()

`first(): Object | false` Переводит итератор на первый объект коллекции и возвращает этот объект.

Возвращает:
- **[объект](../object/index.md)** (`object`) первый объект
- `false` в коллекции нет объектов

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.first() %}

{% if contact %}
    <h2>First contact:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    $object = $collection->first();
    if ($object) {
        // Делаем что-нибудь с объектом...
    }

}
```

## last()

`last(): Object | false` Переводит итератор на последний объект коллекции и возвращает этот объект.

Возвращает:
- **[объект](../object/index.md)** (`object`) последний объект
- `false` в коллекции нет объектов

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.last() %}

{% if contact %}
    <h2>Last contact:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    $object = $collection->last();
    if ($object) {
        // Делаем что-нибудь с объектом...
    }

}
```

## next()

`next(): object | false` Перемещает итератор к следующему объекту и возвращает этот элемент.

Возвращает:
- **[объект](../object/index.md)** (`object`) следующий объект
- `false` в коллекции больше нет объектов

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% set first = contacts.first() %}
...

{% set contact = contacts.next() %}

{% if contact %}
    <h2>Next contact is:</h2>
    <div>{{ contact.first_name|e }} {{ contact.last_name|e }}</div>
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|false $object */
    while ($object = $collection->next()) {
        // Делаем что-нибудь с объектом...
    }

}
```

## current()

`current(): object | false` Возвращает объект коллекции на текущей позиции итератора.

Возвращает:
- **[объект](../object/index.md)** (`object`) текущий объект
- `false` в коллекции больше нет объектов

```twig title="Twig"
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

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {
    while ($collection->next()) {

        /** @var FlexObjectInterface|false $object */
        $object = $collection->current();
        // Делаем что-нибудь с объектом...

    }
}
```

## key()

`key(): key | null` Возвращает ключ объекта на текущей позиции итератора.

Возвращает:
- **key** (`string`) ключ объекта
- `null` в коллекции больше нет объектов

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}
{% do contacts.next() %}
{% do contacts.next() %}
...

{% set key = contacts.key() %}

{% if key %}
    Current contact key is: <strong>{{ key|e }}</strong>
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {
    while ($collection->next()) {

        $key = $collection->key();
        // Делаем что-нибудь с ключом...

    }
}
```

## Получение объекта и ключа

## Доступ как к массиву {: #array-access }

К **коллекциям** можно обращаться так же, как к ассоциативным массивам или картам.

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** возвращается `null`, если объекта с указанным ключом нет в коллекции.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts['ki2ts4cbivggmtlj']

{# Сделать что-нибудь #}
{% if contact %}
  {# Получили Bruce Day #}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|null $object */
    $object = $collection['ki2ts4cbivggmtlj'];
    if ($object) {
        // Объект существует, делаем что-нибудь с ним...
    }

}
```

## get()

`get( key ): Object | null` Возвращает объект с указанным ключом.

Параметры:
- **key**  ключ объекта (`string`)

Возвращает:
- **Object** (`object`)
- `null` объекта с указанным ключом нет в коллекции

Альтернатива [доступу как к массиву](#array-access)

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contact = contacts.get('ki2ts4cbivggmtlj')

{# Сделать что-нибудь #}
{% if contact %}
  {# Получили Bruce Day #}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface|null $object */
    $object = $collection->get('ki2ts4cbivggmtlj');
    if ($object) {
        // Объект существует, делаем что-нибудь с ним...
    }

}
```

## Коллекция как массив

## getKeys()

`getKeys(): array` Возвращает все ключи коллекции.

Возвращает:
- `array` список ключей

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set keys = contacts.keys() %}

Keys are: {{ keys|join(', ')|e }}
```

```php title="PHP"
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

## GetObjectKeys()

`GetObjectKeys(): array` Псевдоним метода `getKeys()`.

Возвращает:
- `array` список ключей

## getValues()

`getValues(): array` Возвращает все объекты коллекции.

Преобразует коллекцию в массив. Ключи не сохраняются.

Возвращает:
- список **объектов** (`array`)

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.values() %}
<ol>
{% for i,object in list %}
    <li>#{{ (i+1)|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var FlexObjectInterface[] $objects */
    $objects = $collection->getValues();
    foreach ($objects as $pos => $object) {
        // Делаем что-нибудь с объектом и его позицией...
    }

}
```

## toArray()

`toArray(): array` Возвращает нативное представление коллекции в виде массива PHP.

Аналогично `getValues()`, но сохраняет ключи.

Возвращает:
- `array` список пар `key: Object`

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.toArray() %}
<ol>
{% for key,object in list %}
    <li>ID: {{ key|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array<string, FlexObjectInterface> $objects */
    $objects = $collection->toArray();
    foreach ($objects as $key => $object) {
        // Делаем что-нибудь с объектом и его ключом...
    }

}
```

## slice()

`slice( offset, length ): array` Извлекает из коллекции срез из `length` элементов, начиная с позиции `offset`.

Параметры:
- **offset** начальное смещение, начиная с 0 (`int`)
- **length** максимальное число объектов (`int`)

Возвращает:
- `array` список пар `key: Object`

> [!WARNING] Предупреждение
> **Совет:** этот метод можно использовать для постраничного вывода.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set list = contacts.slice(10, 5) %}

<div>Displaying 5 emails starting from offset 10:</div>
<ol>
{% for key,object in list %}
    <li>ID: {{ key|e }}: {{ object.email|e }}</li>
{% endfor %}
</ol>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    /** @var array<string, FlexObjectInterface> $objects */
    $objects = $collection->slice(10, 5);

    // Делаем что-нибудь с объектом и его ключом...

}
```

## chunk()

`chunk( size ): array` Разбивает коллекцию на блоки по `size` объектов.

Параметры:
- **size** размер блоков (`int`)

Возвращает:
- `array` двухмерный список пар `key: Object`

> [!WARNING] Предупреждение
> **Совет:** этот метод можно использовать, чтобы разбить контент на строки и колонки.

```twig title="Twig"
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

```php title="PHP"
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
        // Делаем что-нибудь с объектами...
    }
}

}
```

## group()

`group( property ): array` Группирует объекты коллекции по свойству и возвращает их в виде ассоциативного массива.

Параметры:
- **property** имя свойства, по которому группируются объекты (`string`)

Возвращает:
- `array` двухмерный список пар `key: Object`, где значение свойства является ключом первого уровня

```twig title="Twig"
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

```php title="PHP"
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
        // Делаем что-нибудь с объектами...
    }
}

}
```

## Добавление и удаление объектов

## add()

`add( Object )` Добавляет объект в конец коллекции.

Параметры:
- **[объект](../object/index.md)** объект для добавления (`object`)

## remove()

`remove( key ): Object | null` Удаляет элемент по указанному индексу из коллекции.

Параметры:
- **key** ключ удаляемого объекта (`object`)

Возвращает:
- **[объект](../object/index.md)** удалённый объект (`object`) или `null`, если он не найден

## removeElement()

`removeElement( Object ): bool` Удаляет указанный объект из коллекции, если он найден.

Параметры:
- **[объект](../object/index.md)** объект для удаления (`object`)

Возвращает:
- `true`, если объект был в коллекции, `false`, если нет

## clear()

`clear()` Очищает коллекцию, удаляя все элементы.

## Проверки

## containsKey()

`containsKey( key ): bool` Проверяет, содержит ли коллекция объект с указанным ключом.

Параметры:
- **key** проверяемый ключ (`string`)

Возвращает:
- `true`, если объект есть в коллекции, `false`, если нет

## contains()

`contains( object ): bool` Проверяет, содержится ли элемент в коллекции.

Параметры:
- **[объект](../object/index.md)** проверяемый объект (`object`)

Возвращает:
- `true`, если объект есть в коллекции, `false`, если нет

## indexOf()

`indexOf( object ): string | false` Возвращает индекс/ключ указанного объекта.

Параметры:
- **[объект](../object/index.md)** проверяемый объект (`object`)

Возвращает:
- `string` индекс/ключ объекта, `false`, если объект не найден

## isEmpty()

`isEmpty(): bool` Проверяет, пуста ли коллекция (не содержит объектов).

Возвращает:
- `true`, если коллекция пуста, иначе `false`

## count()

`count(): int`

Возвращает:
- `int` число объектов в коллекции

## Массовые действия с объектами

## hasProperty()

`hasProperty( property ): array` Возвращает список пар `key: boolean` — определено ли у объекта с данным ключом свойство.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- `array` пар `key: bool`, где `key` — ключ объекта, а `bool` — true или false.

## getProperty()

`getProperty( property, default ): array` Возвращает список `key: value` для каждого объекта.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- `array` пар `key: value`, где `key` — ключ объекта, а `value` — значение свойства.

## setProperty()

`setProperty( property, value ): Collection` Задаёт новое значение свойства для каждого объекта в коллекции.

Параметры:
- **property** имя свойства (`string`)
- **value** новое значение (`mixed`)

Возвращает:
- **Collection** (`object`) коллекцию для цепочки вызовов методов.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляры объектов, общие для всех коллекций. Если это не задумано, перед использованием метода [скопируйте](#copy) коллекцию.

## defProperty()

`defProperty( property, default ): Collection` Определяет значение свойства по умолчанию для каждого объекта в коллекции.

Параметры:
- **property** имя свойства (`string`)
- **default** значение по умолчанию (`mixed`)

Возвращает:
- **Collection** (`object`) коллекцию для цепочки вызовов методов.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляры объектов, общие для всех коллекций. Если это не задумано, перед использованием метода [скопируйте](#copy) коллекцию.

## unsetProperty()

`unsetProperty( property ): Collection` Удаляет значение свойства у каждого объекта в коллекции.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- **Collection** (`object`) коллекцию для цепочки вызовов методов.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляры объектов, общие для всех коллекций. Если это не задумано, перед использованием метода [скопируйте](#copy) коллекцию.

## call()

`call( method, arguments): array` Вызывает метод для каждого объекта в коллекции. Возвращает результаты каждого вызова.

Параметры:
- **method** имя метода (`string`)
- **arguments** список аргументов (`array`)

Возвращает:
- список пар `key: result` (`array`)

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** если метод изменяет объект, перед использованием метода [скопируйте](#copy) коллекцию.

## getTimestamps()

`getTimestamps(): array`  Возвращает список `key: timestamp` для каждого объекта.

Возвращает:
- список пар `key: timestamp`, где timestamp — целое число (`array`)

## getStorageKeys()

`getStorageKeys(): array`  Возвращает список `key: storage_key` для каждого объекта.

Возвращает:
- список пар `key: storage_key` (`array`)

## getFlexKeys()

`getFlexKeys(): array`  Возвращает список `key: flex_key` для каждого объекта.

Возвращает:
- список пар `key: flex_key` (`array`)

## withKeyField()

`withKeyField( field ): Collection` Возвращает новую коллекцию с другим ключом.

Параметры:
- **field** поле ключа (`string`)
  - 'key': ключ по умолчанию, используемый директорией
  - 'storage_key': ключ слоя хранения
  - 'flex_key': уникальный ключ, который можно использовать, не зная директории

Возвращает:
- **Collection** (`object`) коллекцию, но проиндексированную новым ключом.

## Проверки с замыканиями (только PHP)

## exists()

`exists( Closure ): bool` Проверяет, существует ли объект, удовлетворяющий заданному предикату.

Параметры:
- **Closure** метод, используемый для проверки каждого объекта.

Возвращает:
- `bool` true, если ваша функция обратного вызова возвращает true хотя бы для одного объекта.

## forAll()

`forAll( Closure ): bool` Проверяет, выполняется ли заданный предикат для всех объектов коллекции.

Параметры:
- **Closure** метод, используемый для проверки каждого объекта.

Возвращает:
- `bool` true, если ваша функция обратного вызова возвращает true для всех объектов.

## Фильтрация с замыканиями (только PHP)

## filter()

`filter( Closure ): Collection` Возвращает все объекты коллекции, удовлетворяющие предикату.

Порядок элементов сохраняется.

Параметры:
- **Closure** метод, используемый для проверки одного объекта.

Возвращает:
- **Collection** (`object`) новую коллекцию со всеми объектами, для которых ваша функция обратного вызова возвращает `true`.

## map()

`map( Closure ): Collection` Применяет заданную функцию к каждому объекту коллекции и возвращает новую коллекцию с объектами, возвращёнными функцией.

Параметры:
- **Closure** метод, используемый для проверки одного объекта.

Возвращает:
- **Collection** (`object`) новую коллекцию с объектами, возвращёнными функцией обратного вызова.

## collectionGroup()

`collectionGroup( property ): Collection[]` Группирует объекты коллекции по полю и возвращает их в виде ассоциативного массива коллекций.

Параметры:
- **property** (`string`) свойство, по которому группируются объекты.

Возвращает:
- `array` несколько коллекций в массиве, где ключ — значение свойства.

## matching()

`matching( Criteria ): Collection` Выбирает все объекты, соответствующие выражению, и возвращает новую коллекцию с этими объектами.

Параметры:
- **[Criteria](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expression-builder.html#expression-builder)** выражение

Возвращает:
- **Collection** (`object`) новую коллекцию с объектами, соответствующими критерию.

> [!WARNING] Предупреждение
> **Совет:** см. документацию Doctrine по **[Expression Builder](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expression-builder.html#expression-builder)** и **[Expressions](https://www.doctrine-project.org/projects/doctrine-collections/en/1.6/expressions.html#expressions)**.

## orderBy()

`orderBy( array ): Collection` Переупорядочивает коллекцию по списку пар свойство/значение.

Параметры:
- `array`

Возвращает:
- **Collection** (`object`) новую коллекцию с новым порядком.

## partition()

`partition( Closure ): array` Разделяет коллекцию на две коллекции по предикату.

Ключи сохраняются в результирующих коллекциях.

Параметры:
- **Closure** метод, используемый для разделения одного объекта. Возвращает true или false.

Возвращает:
- `array` разделённые объекты `[[a, b], [c, d, e]]`
