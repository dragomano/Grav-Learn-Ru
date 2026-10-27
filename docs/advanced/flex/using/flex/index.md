---
title: "Flex"
description: "count(): int — подсчитывает количество директорий, зарегистрированных в Flex."
tags:
  - "Flex-объекты"
  - "Twig"
---

# Flex

> [!WARNING] Предупреждение
> **Совет:** полный список методов приведён в разделе **Настройка Flex-объектов**.

## count()

`count(): int` Подсчитывает количество директорий, зарегистрированных в Flex.

Возвращает:
- `int` количество **[директорий](../directory/index.md)**

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

## getDirectories()

`getDirectories( [names] ): array` Возвращает список директорий.

Параметры:
- **names** Необязательно: список имён директорий (`array`)

Возвращает:
- `array` список **[директорий](../directory/index.md)**

> [!WARNING] Предупреждение
> **Совет:** если список имён не передан, метод возвращает все директории, зарегистрированные в Flex.

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

/** @var FlexDirectoryInterface[] $directories */
$listedDirectories = $flex->getDirectories(['contacts', 'phonebook']);
// = ['contacts' => FlexDirectory]

/** @var array<FlexDirectoryInterface|null> $directories */
$listedDirectoriesWithMissing = $flex->getDirectories(['contacts', 'phonebook'], true);
// = ['contacts' => FlexDirectory, 'phonebook' => null]
```

> [!WARNING] Предупреждение
> **Совет:** возможно, стоит убедиться, что вы возвращаете только нужные вам директории.

## hasDirectory()

`hasDirectory( name ): bool`: проверяет, существует ли директория.

Параметры:
- **name** имя директории (`string`)

Возвращает:
- `bool` true, если найдена, иначе false

```twig title="Twig"
{% set flex = grav.get('flex') %}

Flex has {{ not flex.hasDirectory('contacts') ? 'not' }} contacts directory.
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\Flex\Interfaces\FlexInterface;

/** @var FlexInterface $flex */
$flex = Grav::instance()->get('flex');

/** @var bool $exists */
$exists = $flex->hasDirectory('contacts');
```

## getDirectory()

`getDirectory( name ): Directory | null` Возвращает директорию или null, если она не найдена.

Параметры:
- **name** имя директории (`string`)

Возвращает:
- **[директорию](../directory/index.md)** (`object`)
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

> [!NOTE] Примечание
> Посмотрите, что можно делать с **[директорией Flex](../directory/index.md)**.

## getObject()

`getObject( id, directory ): Object | null` Возвращает объект или null, если он не найден.

Параметры:
- **id** идентификатор объекта (`string`)
- **directory** имя директории (`string`)

Возвращает:
- **[объект](../object/index.md)** (`object`)
- `null` объект не найден

```twig title="Twig"
{% set flex = grav.get('flex') %}

{% set contact = flex.object('ki2ts4cbivggmtlj', 'contacts') %}

{# Сделать что-нибудь #}
{% if contact %}
  {# Получили Bruce Day #}
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

> [!NOTE] Примечание
> Посмотрите, что можно делать с **[Flex-объектом](../object/index.md)**.

## getCollection()

`getCollection( directory ): Collection | null` Возвращает коллекцию или null, если она не найдена.

Параметры:
- `directory` имя директории (`string`)

Возвращает:
- **[коллекцию](../collection/index.md)** (`object`)
- `null` директория не найдена

```twig title="Twig"
{% set flex = grav.get('flex') %}

{% set contacts = flex.collection('contacts') %}

{# Сделать что-нибудь #}
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

> [!NOTE] Примечание
> Посмотрите, что можно делать с **[коллекцией Flex](../collection/index.md)**.
