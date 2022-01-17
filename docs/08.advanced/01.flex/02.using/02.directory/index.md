# Каталог Flex

> Список методов для работы с Flex-каталогами в Grav CMS.

## getTitle()

`getTitle(): string` Получить название каталога

Возвращает:
- `string` Заголовок

<!-- tabs:start -->
<!-- tab:Twig-->
```twig
{% set directory = grav.get('flex').directory('contacts') %}

<h2>{{ directory.title|e }}</h2>
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var string $title */
    $title = $directory->getTitle();

}
```
<!-- tabs:end -->

## getDescription()

`getDescription(): string` Получить описание каталога

Возвращает:
- `string` Описание

<!-- tabs:start -->
<!-- tab:Twig-->
```twig
{% set directory = grav.get('flex').directory('contacts') %}

<p>{{ directory.description|e }}</p>
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var string $title */
    $description = $directory->getDescription();

}
```
<!-- tabs:end -->

## getObject()

`getObject( id ): Object | null` Получить объект или `null`, если объект не был найден.

Parameters:
- **id** ID объекта (`string`)

Возвращает:
- **[Object](/08.advanced/01.flex/02.using/04.object/index)** (`object`)
- `null` Объект не найден

<!-- tabs:start -->
<!-- tab:Twig-->
```twig
{% set directory = grav.get('flex').directory('contacts') %}

{% set contact = directory.object('ki2ts4cbivggmtlj') %}

{# Do something #}
{% if contact %}
  {# Got Bruce Day #}
  Email for {{ contact.first_name|e }} {{ contact.last_name|e }} is {{ contact.email|e }}
{% else %}
  Oops, contact has been removed!
{% endif %}
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var FlexObjectInterface|null $object */
    $object = $directory->getObject('ki2ts4cbivggmtlj');
    if ($object) {
        // Object exists, do something with it...
    }

}
```
<!-- tabs:end -->

!> Проверьте, что вы можете сделать с **[Объектом Flex](/08.advanced/01.flex/02.using/04.object/index)**

## getCollection()

`getCollection(): Collection` Получить коллекцию или `null`, если коллекция не была найдена.

Возвращает:
- **[Collection](/08.advanced/01.flex/02.using/03.collection/index)** (`object`)

<!-- tabs:start -->
<!-- tab:Twig-->
```twig
{% set directory = grav.get('flex').directory('contacts') %}

{% set contacts = directory.collection() %}

{# Do something #}
<h2>Ten first contacts:</h2>
<ul>
  {% for contact in contacts.filterBy({published: true}).limit(0, 10) %}
    <li>{{ contact.first_name|e }} {{ contact.last_name|e }}</li>
  {% endfor %}
</ul>
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexDirectoryInterface;
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

/** @var FlexDirectoryInterface|null $directory */
$directory = Grav::instance()->get('flex')->getDirectory('contacts');
if ($directory) {

    /** @var FlexCollectionInterface $collection */
    $collection = $directory->getCollection();

    // Do something with the collection...

}
```
<!-- tabs:end -->

!> Проверьте, что вы можете сделать с **[Коллекцией Flex](/08.advanced/01.flex/02.using/03.collection/index)**
