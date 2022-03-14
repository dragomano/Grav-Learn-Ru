---
description: Список методов для работы с Flex-каталогами в Grav CMS.
---

# Каталог Flex

## getTitle()

`getTitle(): string` Получить название каталога

Возвращает:

* `string` Заголовок

=== "Twig"

	```twig
	{% set directory = grav.get('flex').directory('contacts') %}

	<h2>{{ directory.title|e }}</h2>
	```

=== "PHP"

	```php
	<?php

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

`getDescription(): string` Получить описание каталога

Возвращает:

* `string` Описание

=== "Twig"

	```twig
	{% set directory = grav.get('flex').directory('contacts') %}

	<p>{{ directory.description|e }}</p>
	```

=== "PHP"

	```php
	<?php

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

`getObject( id ): Object | null` Получить объект или `null`, если объект не был найден.

Параметры:

* **id** ID объекта (`string`)

Возвращает:

* **[Object](../object)** (`object`)
* `null` Объект не найден

=== "Twig"

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

=== "PHP"

	```php
	<?php

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

!!! tip ""

	Проверьте, что вы можете сделать с **[Объектом Flex](../object)**

## getCollection()

`getCollection(): Collection` Получить коллекцию или `null`, если коллекция не была найдена.

Возвращает:

* **[Collection](../collection)** (`object`)

=== "Twig"

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

=== "PHP"

	```php
	<?php

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

!!! tip ""

	Проверьте, что вы можете сделать с **[Коллекцией Flex](../collection)**
