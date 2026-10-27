---
title: "Flex-объект"
description: "render( , ): Block — рендерит объект."
tags:
  - "Flex-объекты"
  - "Twig"
---

# Flex-объект {: #render-object }

## render()

`render( [layout], [context] ): Block` Рендерит объект.

Параметры:
- **layout** имя макета (`string`)
- **context** дополнительные переменные, доступные внутри файла шаблона Twig (`array`)

Возвращает:
- **Block** (`object`) класс HtmlBlock, содержащий вывод

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** в Twig есть тег `{% render %}`, который следует использовать вместо прямого вызова метода. Это позволит корректно работать JS/CSS-ресурсам объекта.

```twig title="Twig"
{% set contact = grav.get('flex').object('gizwsvkyo5xtms2s', 'contacts') %}

{% render contact layout: 'details' with { my_variable: true } %}
```

```php title="PHP"
use Grav\Common\Grav;
use Grav\Framework\ContentBlock\HtmlBlock;
use Grav\Framework\Flex\Interfaces\FlexObjectInterface;

/** @var FlexObjectInterface|null $collection */
$object = Grav::instance()->get('flex')->getObject('gizwsvkyo5xtms2s', 'contacts');
if ($object) {

    /** @var HtmlBlock $block */
    $block = $object->render('details', ['my_variable' => true]);

}
```

## Прочее

## getKey()

`getKey(): string` Возвращает ключ объекта.

Возвращает:
- `string` ключ объекта

## hasKey()

`hasKey(): bool` Возвращает true, если ключ объекта задан.

Возвращает:
- `true`, если у объекта есть ключ, иначе `false`

## getFlexType()

`getFlexType(): string` Возвращает тип объекта.

Возвращает:
- `string` имя директории Flex, к которой относится объект

## hasProperty()

`hasProperty( property ): bool` Возвращает true, если свойство объекта определено и имеет значение (не null).

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- `true`, если у свойства есть значение, иначе `false`.

## getProperty()

`getProperty( property, default ): mixed` Возвращает значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- `mixed` значение свойства
- `null`, если свойство не определено или не имеет значения

## setProperty()

`setProperty( property, value ): Object` Задаёт новое значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)
- **value** новое значение (`mixed`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов методов

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляр объекта, общий для всех коллекций. Если это не задумано, перед использованием метода сделайте `clone` объекта.

## defProperty()

`defProperty( property, default ): Object` Определяет значение свойства объекта по умолчанию.

Параметры:
- **property** имя свойства (`string`)
- **default** значение по умолчанию (`mixed`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов методов

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляр объекта, общий для всех коллекций. Если это не задумано, перед использованием метода сделайте `clone` объекта.

## unsetProperty()

`unsetProperty( property ): Object` Удаляет значение свойства объекта.

Параметры:
- **property** имя свойства (`string`)

Возвращает:
- **Object** (`object`) объект для цепочки вызовов методов

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** этот метод изменяет экземпляр объекта, общий для всех коллекций. Если это не задумано, перед использованием метода сделайте `clone` объекта.

## isAuthorized()

`isAuthorized( action, [scope], [user] ): bool | null` Проверяет, авторизован ли пользователь для действия.

Параметры:
- **action** (`string`)
  - одно из: `create`, `read`, `update`, `delete`, `list`
- **scope** необязательно (`string`)
  - обычно `admin` или `site`
- **user** необязательно, объект пользователя (`object`)

Возвращает:
- `true` действие разрешено
- `false` действие запрещено
- `null` не задано (действует как запрет)

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** есть два значения запрета: запрещено (false) и не задано (null). Это позволяет объединять несколько правил в цепочку, если предыдущие правила не совпали.

## getFlexDirectory()

`getFlexDirectory(): Directory`

Возвращает:
- **[директорию](../directory/index.md)** (`object`)

## getTimestamp()

`getTimestamp(): int` Возвращает временную метку последнего изменения объекта.

Возвращает:
- `int` временная метка.

## search()

`search(string, [properties], [options] ): float` Ищет строку в объекте, возвращает вес от 0 до 1.

Параметры:
- **string** строка поиска (`string`)
- **properties** свойства для поиска; если null (или не переданы), используются значения по умолчанию (`array` или `null`)
- **options** дополнительные параметры поиска (`array`)
  - starts_with: `bool`
  - ends_with: `bool`
  - contains: `bool`
  - case_sensitive: `bool`

Возвращает:
- `float` вес совпадения от 0 до 1, используется для сортировки результатов
- `0` объект не совпадает с поиском

> [!NOTE] Примечание
> **ПРИМЕЧАНИЕ:** если вы переопределяете эту функцию, убедитесь, что возвращаемое значение лежит в диапазоне 0...1!

## getFlexKey()

`getFlexKey(): string` Возвращает уникальный ключ объекта.

Возвращает:
- `string` **ключ Flex** объекта

Ключи Flex можно использовать, не зная, к какой директории относится объект.

## getStorageKey()

`getStorageKey(): string` Возвращает уникальный ключ хранения (в пределах директории), который используется для определения имени файла или идентификатора в базе данных.

Возвращает:
- `string` **ключ хранения** объекта

## exists()

`exists(): bool` Возвращает true, если объект существует в хранилище.

Возвращает:
- `true` объект существует в хранилище
- `false` объект ещё не сохранён

## update()

`update( data, files ): Object` Обновляет объект в памяти.

Параметры:
- **data** (`array`) вложенные массивы свойств с их значениями
- **files** (`array`) массив объектов `Psr\Http\Message\UploadedFileInterface`

Возвращает:
- **Object** (`object`) объект для цепочки вызовов методов

> [!WARNING] Предупреждение
> **Совет:** после вызова этого метода объект нужно сохранить.

## create()

`create( [key] ): Object` Создаёт новый объект в хранилище.

Параметры:
- **key** (`string`) необязательный ключ

Возвращает:
- **Object** (`object`) сохранённый объект

## createCopy()

`createCopy( [key] ): Object` Создаёт новый объект на основе текущего и сохраняет его в хранилище.

Параметры:
- **key** (`string`) необязательный ключ

Возвращает:
- **Object** (`object`) сохранённый объект

## save()

`save(): Object` Сохраняет объект в хранилище.

Возвращает:
- **Object** (`object`) сохранённый объект

## delete()

`delete(): Object` Удаляет объект из хранилища.

Возвращает:
- **Object** (`object`) удалённый объект

## getBlueprint()

`getBlueprint( [name] ): Blueprint` Возвращает схему объекта.

Параметры:
- **name** (`string`) необязательное имя схемы

Возвращает:
- **Blueprint** (`object`)

## getForm()

`getForm( [name], [options] ): Form` Возвращает экземпляр формы для объекта.

Параметры:
- **name** (`string`) необязательное имя формы
- **options** (`array`) необязательные параметры формы

Возвращает:
- **Form** (`object`)

## getDefaultValue()

`getDefaultValue( name, [separator] ): mixed` Возвращает значение по умолчанию, подходящее для использования в форме для указанного свойства.

Параметры:
- **name** (`string`) имя свойства
- **separator** (`string`) необязательный символ-разделитель для вложенных свойств, по умолчанию `.` (точка)

Возвращает:
- `mixed` значение свойства по умолчанию

## getDefaultValues()

`getDefaultValues(): array` Возвращает значения по умолчанию, подходящие для использования в форме для указанного свойства.

Возвращает:
- `array` все значения по умолчанию

## getFormValue()

`getFormValue( name, [default], [separator] ): mixed` Возвращает сырое значение, подходящее для использования в форме для указанного свойства.

Параметры:
- **name** (`string`) имя свойства
- **default** (`mixed`) необязательное значение поля по умолчанию, `null`, если не задано
- **separator** (`string`) необязательный символ-разделитель для вложенных свойств, по умолчанию `.` (точка)

Возвращает:
- `mixed` значение поля формы

## triggerEvent()

`triggerEvent( name, [Event] ): Object` Запускает событие по вашему выбору.

Параметры:
- **name** (`string`) имя события
- **Event** (`object`) необязательный класс события

Возвращает:
- **Object** (`object`) объект для цепочки вызовов методов
