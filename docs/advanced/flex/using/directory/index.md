---
title: "Директория Flex"
description: "getTitle(): string — возвращает заголовок директории."
tags:
  - "Flex-объекты"
  - "Twig"
---

# Директория Flex

## getTitle()

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

## getDescription()

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

    /** @var string $title */
    $description = $directory->getDescription();

}
```

## getObject()

`getObject( id ): Object | null` Возвращает объект или null, если он не найден.

Параметры:
- **id** идентификатор объекта (`string`)

Возвращает:
- **[объект](../object/index.md)** (`object`)
- `null` объект не найден

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

{% set contact = directory.object('ki2ts4cbivggmtlj') %}

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

> [!NOTE] Примечание
> Посмотрите, что можно делать с **[Flex-объектом](../object/index.md)**.

## getCollection()

`getCollection(): Collection` Возвращает коллекцию или null, если она не найдена.

Возвращает:
- **[коллекцию](../collection/index.md)** (`object`)

```twig title="Twig"
{% set directory = grav.get('flex').directory('contacts') %}

{% set contacts = directory.collection() %}

{# Сделать что-нибудь #}
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

> [!NOTE] Примечание
> Посмотрите, что можно делать с **[коллекцией Flex](../collection/index.md)**.
