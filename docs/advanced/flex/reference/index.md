---
title: "Справочник по Twig и PHP"
description: "После регистрации типа Flex вы работаете с его данными во время выполнения через четыре объекта: сервис Flex, директорию, коллекцию и объект. Одни и те же методы доступны и из Twig, и из PHP, поэтому шаблон…"
tags:
  - "Flex-объекты"
  - "PHP"
  - "Twig"
  - "Справочник API"
---

# Справочник по Twig и PHP

После регистрации типа Flex вы работаете с его данными во время выполнения через четыре объекта: сервис **Flex**, **директорию**, **коллекцию** и **объект**. Одни и те же методы доступны и из Twig, и из PHP, поэтому шаблон и класс плагина читаются почти одинаково. Эта страница — справочник методов для всех четырёх.

Точка входа всегда — сервис Flex, зарегистрированный в контейнере Grav под ключом `flex`. В Twig вы получаете его через `grav.get('flex')`, в PHP — через `Grav::instance()->get('flex')`. Оттуда вы запрашиваете директорию по имени, затем коллекцию или отдельный объект, а затем работаете с отдельными объектами и их свойствами.

> [!NOTE] Примечание
> Методы работы с коллекциями и объектами (`sort`, `filterBy`, `limit` и подобные) возвращают **изменённую копию**. Исходный экземпляр остаётся нетронутым, поэтому можно безопасно строить цепочки вызовов, не меняя общее состояние.

## Flex

Сервис **Flex** — это реестр верхнего уровня. Он знает о каждой включённой директории (см. [Регистрация директории](../index.md)) и выдаёт вам директории, коллекции и объекты по имени.

### count()

`count(): int` Подсчитывает количество директорий, зарегистрированных в Flex.

Возвращает:
- `int` количество [директорий](#directory)

```twig title="Twig"
{% set flex = grav.get('flex') %}

Flex has {{ flex.count() }} enabled directories.
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var int $count */
$count = $flex->count();
```

### getDirectories()

`getDirectories( [names] ): array` Возвращает список директорий.

Параметры:
- **names** необязательно: список имён директорий (`array`)

Возвращает:
- `array` список [директорий](#directory)

> [!NOTE] Примечание
> Если список имён не передан, метод возвращает все директории, зарегистрированные в Flex.

```twig title="Twig"
{% set flex = grav.get('flex') %}

{# Получить все директории #}
{% set directories = flex.directories() %}

{# Получить перечисленные директории #}
{% set listed_directories = flex.directories(['contacts', 'phonebook']) %}

{# Сделать что-нибудь с директориями #}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexDirectoryInterface[] $directories */
$directories = $flex->getDirectories();
// = ['contacts' => FlexDirectory, ...]

/** @var FlexDirectoryInterface[] $listedDirectories */
$listedDirectories = $flex->getDirectories(['contacts', 'phonebook']);
// = ['contacts' => FlexDirectory]

/** @var array<FlexDirectoryInterface|null> $listedDirectoriesWithMissing */
$listedDirectoriesWithMissing = $flex->getDirectories(['contacts', 'phonebook'], true);
// = ['contacts' => FlexDirectory, 'phonebook' => null]
```

### hasDirectory()

`hasDirectory( name ): bool` Проверяет, существует ли директория.

Параметры:
- **name** имя директории (`string`)

Возвращает:
- `bool` true, если найдена, иначе false

```twig title="Twig"
{% set flex = grav.get('flex') %}

Flex has {{ not flex.hasDirectory('contacts') ? 'no' }} contacts directory.
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var bool $exists */
$exists = $flex->hasDirectory('contacts');
```

### getDirectory()

`getDirectory( name ): Directory | null` Возвращает директорию или null, если она не найдена.

Параметры:
- **name** имя директории (`string`)

Возвращает:
- [директорию](#directory) (`object`)
- `null` директория не найдена

```twig title="Twig"
{% set flex = grav.get('flex') %}

{# Получить директорию contacts (null, если не найдена) #}
{% set directory = flex.directory('contacts') %}

{# Сделать что-нибудь с директорией contacts #}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexDirectoryInterface|null $directory */
$directory = $flex->getDirectory('contacts');
if ($directory) {
    // Директория существует, делаем что-нибудь с ней...
}
```

### getObject()

`getObject( id, directory ): Object | null` Возвращает объект или null, если он не найден.

Параметры:
- **id** идентификатор объекта (`string`)
- **directory** имя директории (`string`)

Возвращает:
- [объект](#object) (`object`)
- `null` объект не найден

```twig title="Twig"
{% set flex = grav.get('flex') %}

{% set contact = flex.object('ki2ts4cbivggmtlj', 'contacts') %}

{# Сделать что-нибудь #}
{% if contact %}
  {{ contact.first_name|e }} {{ contact.last_name|e }} has a website: {{ contact.website|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexObjectInterface|null $object */
$object = $flex->getObject('ki2ts4cbivggmtlj', 'contacts');
if ($object) {
    // Объект существует, делаем что-нибудь с ним...
}
```

### getCollection()

`getCollection( directory ): Collection | null` Возвращает коллекцию или null, если она не найдена.

Параметры:
- **directory** имя директории (`string`)

Возвращает:
- [коллекцию](#collection) (`object`)
- `null` директория не найдена

```twig title="Twig"
{% set flex = grav.get('flex') %}

{% set contacts = flex.collection('contacts') %}

<h2>Ten random contacts:</h2>
<ul>
  {% for contact in contacts.filterBy({published: true}).shuffle().limit(0, 10) %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var FlexCollectionInterface|null $collection */
$collection = $flex->getCollection('contacts');
if ($collection) {
    // Коллекция существует, делаем что-нибудь с ней...
}
```

## Директория {: #directory }

**Директория Flex** представляет один зарегистрированный тип Flex. Она несёт заголовок и описание типа, и именно через неё вы запрашиваете коллекцию типа или отдельный объект по id.

### getTitle()

`getTitle(): string` Возвращает заголовок директории.

Возвращает:
- `string` заголовок

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

<h2>{{ directory.title|e }}</h2>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var string $title */
    $title = $directory->getTitle();

}
```

### getDescription()

`getDescription(): string` Возвращает описание директории.

Возвращает:
- `string` описание

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

<p>{{ directory.description|e }}</p>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var string $description */
    $description = $directory->getDescription();

}
```

### getObject() {: #getobject-1 }

`getObject( [id] ): Object | null` Возвращает объект из директории или null, если он не найден. Вызванный без id, возвращает новый пустой объект.

Параметры:
- **id** идентификатор объекта (`string`)

Возвращает:
- [объект](#object) (`object`)
- `null` объект не найден

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

{% set contact = directory.object('ki2ts4cbivggmtlj') %}

{# Сделать что-нибудь #}
{% if contact %}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var FlexObjectInterface|null $object */
    $object = $directory->getObject('ki2ts4cbivggmtlj');
    if ($object) {
        // Объект существует, делаем что-нибудь с ним...
    }

}
```

### getCollection() {: #getcollection-1 }

`getCollection(): Collection` Возвращает коллекцию всех объектов директории.

Возвращает:
- [коллекцию](#collection) (`object`)

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

{% set contacts = directory.collection() %}

<h2>Ten first contacts:</h2>
<ul>
  {% for contact in contacts.filterBy({published: true}).limit(0, 10) %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var FlexCollectionInterface $collection */
    $collection = $directory->getCollection();

    // Делаем что-нибудь с коллекцией...

}
```

Директория также предоставляет несколько методов для самоанализа, к которым вы изредка будете обращаться из PHP: `getFlexType()` (имя директории), `getObjectClass()` и `getCollectionClass()` (классы, настроенные в `config.data`), `getConfig()` (итоговая конфигурация схемы) и `getBlueprint()` (схема директории). Они особенно полезны, когда вы пишете собственные классы объектов или коллекций.

## Коллекция {: #collection }

**Коллекция Flex** — это **упорядоченная карта объектов**, которую также можно использовать как список. Она предоставляет методы для рендеринга вывода, получения объектов по ключу, а также сортировки, фильтрации, группировки и извлечения срезов набора. Коллекции Flex построены на [Doctrine Collections](https://www.doctrine-project.org/projects/doctrine-collections/en/2.2/index.html), поэтому стандартные помощники в стиле массивов (`first()`, `last()`, `getKeys()`, `toArray()`, `isEmpty()`, `contains()` и так далее) тоже доступны.

> [!NOTE] Примечание
> Каждый метод ниже возвращает **изменённую копию** коллекции. Исходная никогда не меняется, что делает безопасными длинные цепочки вроде `collection.filterBy(...).sort(...).limit(...)`.

### render()

`render( [layout], [context] ): Block` Рендерит коллекцию.

Параметры:
- **layout** имя макета (`string`)
- **context** дополнительные переменные, доступные внутри файла шаблона Twig (`array`)

Возвращает:
- **Block** (`object`) экземпляр HtmlBlock, содержащий вывод

> [!TIP] Совет
> В Twig используйте тег `{% render %}`, а не прямой вызов метода. Тег позволяет корректно зарегистрировать JS/CSS-ресурсы коллекции.

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
$start = ($page - 1) * $limit;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->limit($start, $limit);

    /** @var HtmlBlock $block */
    $block = $collection->render('cards', ['background' => 'gray', 'color' => 'white']);

}
```

### sort()

`sort( orderings ): Collection` Сортирует коллекцию по списку пар `property: direction`.

Параметры:
- **orderings** пары `property: direction`, где direction — `ASC` или `DESC` (`array`)

Возвращает:
- [коллекцию](#collection) (`object`) новый отсортированный экземпляр

> [!NOTE] Примечание
> Порядок сортировки по умолчанию для фронтенда можно задать в схеме типа Flex.

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

### limit()

`limit( start, limit ): Collection` Возвращает подмножество коллекции, начиная с `start` и включая только `limit` объектов.

Параметры:
- **start** начальный индекс, начиная с 0 (`int`)
- **limit** максимальное число объектов (`int`)

Возвращает:
- [коллекцию](#collection) (`object`) новый отфильтрованный экземпляр

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
    // Коллекция теперь содержит только объекты текущей страницы...

}
```

### filterBy()

`filterBy( filters ): Collection` Фильтрует коллекцию по списку пар `property: value`.

Параметры:
- **filters** пары `property: value`, используемые для фильтрации коллекции (`array`)

Возвращает:
- [коллекцию](#collection) (`object`) новый отфильтрованный экземпляр

> [!NOTE] Примечание
> Фильтрацию по умолчанию для фронтенда можно задать в схеме типа Flex.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.filterBy({'published': true}) %}

<div>Displaying only published contacts:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->filterBy(['published' => true]);
    // Коллекция теперь содержит только опубликованные объекты...

}
```

### reverse()

`reverse(): Collection` Меняет порядок объектов в коллекции на обратный.

Возвращает:
- [коллекцию](#collection) (`object`) новый экземпляр в обратном порядке

> [!TIP] Совет
> Если вы уже используете `sort()`, задайте обратный порядок прямо в нём. Так вы сэкономите лишний шаг.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set contacts = contacts.reverse() %}

<div>Displaying contacts in reverse order:</div>
{% render contacts layout: 'cards' %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexCollectionInterface|null $collection */
$collection = Grav::instance()->get('flex')->getCollection('contacts');
if ($collection) {

    $collection = $collection->reverse();
    // Коллекция теперь в обратном порядке...

}
```

### shuffle()

`shuffle(): Collection` Перемешивает объекты в случайном порядке.

Возвращает:
- [коллекцию](#collection) (`object`) новый экземпляр в случайном порядке

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

    $collection = $collection->shuffle()->limit(0, 6);
    // Коллекция теперь содержит 6 случайных контактов...

}
```

### select()

`select( keys ): Collection` Оставляет только объекты, ключи которых есть в заданном списке.

Параметры:
- **keys** список ключей объектов для выбора (`array`)

Возвращает:
- [коллекцию](#collection) (`object`) новый экземпляр

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

### unselect()

`unselect( keys ): Collection` Удаляет объекты, ключи которых есть в заданном списке.

Параметры:
- **keys** список ключей объектов для удаления (`array`)

Возвращает:
- [коллекцию](#collection) (`object`) новый экземпляр

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

### search()

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
- [коллекцию](#collection) (`object`) новый отфильтрованный экземпляр

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
    // Коллекция теперь содержит все совпавшие результаты...

}
```

### copy()

`copy(): Collection` Создаёт копию коллекции, клонируя каждый объект в ней.

Возвращает:
- [коллекцию](#collection) (`object`) новый экземпляр с клонированными объектами

> [!WARNING] Предупреждение
> Если вы собираетесь изменять объекты в коллекции, всегда сначала работайте с `copy()`. Массовые помощники `setProperty()` / `defProperty()` / `unsetProperty()` изменяют экземпляры объектов, общие для всех коллекций, которые на них ссылаются.

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

    $collection = $collection->shuffle()->limit(0, 10);

    /** @var FlexCollectionInterface $fakes */
    $fakes = $collection->copy();
    $fakes->setProperty('first_name', 'JACK');
    // $collection не изменена; изменились только клонированные объекты в $fakes...

}
```

### chunk()

`chunk( size ): array` Разбивает коллекцию на блоки по `size` объектов.

Параметры:
- **size** размер каждого блока (`int`)

Возвращает:
- `array` двухмерный список пар `key: Object`

> [!TIP] Совет
> Удобно для раскладки контента по строкам и колонкам.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set columns = contacts.limit(0, 10).chunk(5) %}

<div>Displaying two columns of 5 emails each:</div>
<div class="columns">
{% for column, list in columns %}
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

    /**
     * @var int $column
     * @var array<string, FlexObjectInterface> $objects
     */
    foreach ($columns as $column => $objects) {
        // Делаем что-нибудь с объектами...
    }

}
```

### group()

`group( property ): array` Группирует объекты по свойству и возвращает их в виде ассоциативного массива.

Параметры:
- **property** имя свойства, по которому группируются объекты (`string`)

Возвращает:
- `array` двухмерный список пар `key: Object`, где значение свойства является ключом первого уровня

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{% set by_name = contacts.sort({last_name: 'ASC', first_name: 'ASC'}).group('last_name') %}

<div>Displaying contacts grouped by last name:</div>
<div>
{% for last_name, list in by_name %}
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

    /**
     * @var string $lastName
     * @var array<string, FlexObjectInterface> $objects
     */
    foreach ($byName as $lastName => $objects) {
        // Делаем что-нибудь с объектами...
    }

}
```

### Получение одного объекта

Вы можете вытащить один объект из коллекции по его ключу — либо через доступ как к массиву, либо методом `get()`. Оба возвращают `null`, если ключа нет.

```twig title="Twig"
{% set contacts = grav.get('flex').collection('contacts') %}

{# Доступ как к массиву #}
{% set contact = contacts['ki2ts4cbivggmtlj'] %}

{# Или эквивалентный метод get() #}
{% set contact = contacts.get('ki2ts4cbivggmtlj') %}

{% if contact %}
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
    $object = $collection->get('ki2ts4cbivggmtlj'); // или $collection['ki2ts4cbivggmtlj']
    if ($object) {
        // Объект существует, делаем что-нибудь с ним...
    }

}
```

### getKeys() и getObjectKeys()

`getKeys(): array` Возвращает все ключи коллекции. `getObjectKeys(): array` — псевдоним `getKeys()`.

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

## Объект {: #object }

**Flex-объект** представляет одну запись в директории: один контакт, один заказ, одну запись. Он оборачивает данные этой записи типизированным доступом к свойствам (`getProperty`, `setProperty` и подобными), знает свой ключ и тип, рендерит себя через макет и может отвечать на вопросы авторизации. Объекты также несут методы сохранения (`save()`, `delete()`, `update()`, `create()`), когда нужно записать данные обратно в хранилище.

### render() {: #render-1 }

`render( [layout], [context] ): Block` Рендерит объект.

Параметры:
- **layout** имя макета (`string`)
- **context** дополнительные переменные, доступные внутри файла шаблона Twig (`array`)

Возвращает:
- **Block** (`object`) экземпляр HtmlBlock, содержащий вывод

> [!TIP] Совет
> В Twig используйте тег `{% render %}`, а не прямой вызов метода, чтобы JS/CSS-ресурсы объекта зарегистрировались корректно.

```twig title="Twig"
{% set contact = grav.get('flex').object('gizwsvkyo5xtms2s', 'contacts') %}

{% render contact layout: 'details' with { my_variable: true } %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\ContentBlock\HtmlBlock;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexObjectInterface|null $object */
$object = Grav::instance()->get('flex')->getObject('gizwsvkyo5xtms2s', 'contacts');
if ($object) {

    /** @var HtmlBlock $block */
    $block = $object->render('details', ['my_variable' => true]);

}
```

### getKey()

`getKey(): string` Возвращает ключ объекта.

Возвращает:
- `string` ключ объекта

### hasKey()

`hasKey(): bool` Возвращает true, если ключ объекта задан.

Возвращает:
- `true`, если у объекта есть ключ, иначе `false`

### getFlexType()

`getFlexType(): string` Возвращает тип объекта.

Возвращает:
- `string` имя директории Flex, к которой относится объект

### hasProperty()

`hasProperty( property ): bool` Возвращает true, если свойство определено и имеет значение (не null).

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- `true`, если у свойства есть значение, иначе `false`

### getProperty()

`getProperty( property, [default] ): mixed` Возвращает значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)
- **default** необязательное значение, возвращаемое, когда свойство не задано (`mixed`)

Возвращает:
- `mixed` значение свойства
- `null`, если свойство не определено или не имеет значения

### setProperty()

`setProperty( property, value ): Object` Задаёт новое значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)
- **value** новое значение (`mixed`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов

> [!WARNING] Предупреждение
> Этот метод изменяет экземпляр объекта, общий для всех коллекций, которые на него ссылаются. Если это не задумано, сначала сделайте `clone` объекта.

### defProperty()

`defProperty( property, default ): Object` Определяет значение свойства объекта по умолчанию (применяется, только когда свойство ещё не задано).

Параметры:
- **property** имя свойства (`string`)
- **default** значение по умолчанию (`mixed`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов

> [!WARNING] Предупреждение
> Этот метод изменяет общий экземпляр объекта. Если это не задумано, сначала сделайте `clone` объекта.

### unsetProperty()

`unsetProperty( property ): Object` Удаляет значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов

> [!WARNING] Предупреждение
> Этот метод изменяет общий экземпляр объекта. Если это не задумано, сначала сделайте `clone` объекта.

### isAuthorized()

`isAuthorized( action, [scope], [user] ): bool | null` Проверяет, авторизован ли пользователь для выполнения действия над объектом.

Параметры:
- **action** (`string`)
  - одно из: `create`, `read`, `update`, `delete`, `list`
- **scope** необязательно (`string`)
  - обычно `admin` или `site`
- **user** необязательно, объект пользователя (`object`); по умолчанию текущий пользователь

Возвращает:
- `true` действие разрешено
- `false` действие запрещено
- `null` не задано (трактуется как запрет)

> [!NOTE] Примечание
> Есть два «запрещающих» значения: явный запрет (`false`) и «не задано» (`null`). Различать их нужно, чтобы можно было объединять несколько правил в цепочку, и тогда более позднее правило может решить случай, который не совпал с более ранним.

```twig title="Twig"
{% set contact = grav.get('flex').object('gizwsvkyo5xtms2s', 'contacts') %}

{% if contact.isAuthorized('update', 'admin') %}
  <a href="...">Edit this contact</a>
{% endif %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexObjectInterface|null $object */
$object = Grav::instance()->get('flex')->getObject('gizwsvkyo5xtms2s', 'contacts');
if ($object && $object->isAuthorized('update', 'admin')) {
    // Текущий пользователь может редактировать этот объект...
}
```

Кроме них, объекты предоставляют помощники сохранения и работы с формами, которые понадобятся вам при записи данных: `getFlexKey()` и `getStorageKey()` (уникальные идентификаторы), `getTimestamp()`, `exists()`, `update( data, files )`, `create( [key] )`, `createCopy( [key] )`, `save()`, `delete()`, `getBlueprint()` и `getForm()`. Сохранение и удаление вызывают события `onFlexObjectAfterSave` и `onFlexObjectAfterDelete`, поэтому плагины могут реагировать на записи (см. [REST API Flex-объектов](../../../api/endpoints/flex-objects/index.md) и справочник событий для полной картины).
