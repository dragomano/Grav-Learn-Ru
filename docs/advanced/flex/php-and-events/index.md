---
title: "Расширение Flex на PHP"
description: "Схемы дают очень многое, но рано или поздно любому типу Flex встречается задача, которую нельзя выразить одним YAML: вычисляемое свойство, директория, поставляемая плагином, собственный маршрут фронтенда или побочный эффект, который…"
tags:
  - "Flex-объекты"
  - "PHP"
  - "События"
---

# Расширение Flex на PHP

Схемы дают очень многое, но рано или поздно любому типу Flex встречается задача, которую нельзя выразить одним YAML: вычисляемое свойство, директория, поставляемая плагином, собственный маршрут фронтенда или побочный эффект, который должен срабатывать при каждом сохранении объекта. Вот тут и нужен PHP.

Эта страница проходит по пяти точкам расширения, к которым вы, скорее всего, обратитесь:

1. Наследование классов объекта, коллекции и индекса с подключением их в схему.
2. Регистрация типа директории из плагина через `FlexRegisterEvent`.
3. Подписка на событие маршрутизации фронтенда `flex.router.{name}`.
4. Контракт событий для сохранения и удаления.
5. Команда CLI `convert-data`.

Всё описанное ниже относится к основному фреймворку (`Grav\Framework\Flex` и `Grav\Common\Flex`) или к плагину flex-objects, поэтому доступно любому плагину или теме.

## Свои классы объекта, коллекции и индекса

Каждый тип Flex опирается на три класса:

| Класс          | Роль                                                                                     |
| :------------- | :--------------------------------------------------------------------------------------- |
| **Object**     | Отдельный элемент. Здесь живут логика отдельного элемента и вычисляемые свойства.                |
| **Collection** | Набор загруженных объектов. Здесь живут помощники для всего списка (фильтрация, группировка).     |
| **Index**      | Лёгкий кэшируемый поиск по всем ключам директории, используемый до загрузки объектов. |

Если вы не задаёте свои классы, Flex использует универсальные значения по умолчанию: `Grav\Common\Flex\Types\Generic\GenericObject`, `GenericCollection` и `GenericIndex`. Для простых данных они вполне подходят, но как только вам нужен собственный метод, вы наследуете нужный класс и указываете его в схеме.

### Написание класса

Минимальный подкласс объекта добавляет один метод и больше ничего. Поскольку Flex-объект — обычный объект PHP, любой добавленный публичный метод сразу можно вызывать из Twig.

```php
<?php
namespace Grav\Plugin\MyPlugin\Flex\Types\Product;

use Grav\Common\Flex\Types\Generic\GenericObject;

class ProductObject extends GenericObject
{
    /**
     * Цена с учётом налога, готовая к выводу.
     */
    public function getPriceWithTax(float $rate = 0.2): float
    {
        $price = (float) $this->getProperty('price');

        return round($price * (1 + $rate), 2);
    }
}
```

`getProperty()` читает значение из сохранённых данных объекта, а возвращать вы можете что угодно. Методы фреймворка тоже можно переопределять (`getFormValue()`, `prepareStorage()`, `save()`), но начинайте с обычных вспомогательных методов — они покрывают большинство случаев.

> [!TIP] Совет
> Размещайте классы там, откуда их можно автозагрузить. В плагине добавьте запись PSR-4 в `composer.json` плагина (затем запустите `composer dump-autoload`) либо зарегистрируйте пространство имён из метода `autoload()` плагина. Файл класса из примера выше лежит по пути `user/plugins/my-plugin/classes/Flex/Types/Product/ProductObject.php`.

### Подключение в схему

Укажите в `config.data` схемы написанные вами классы. Любой пропущенный ключ откатывается к универсальному значению по умолчанию, поэтому переопределить только класс объекта вполне нормально.

```yaml
title: Products
type: flex-objects

config:
    data:
        object: 'Grav\Plugin\MyPlugin\Flex\Types\Product\ProductObject'
        collection: 'Grav\Common\Flex\Types\Generic\GenericCollection'
        index: 'Grav\Common\Flex\Types\Generic\GenericIndex'
        storage:
            class: 'Grav\Framework\Flex\Storage\FolderStorage'
            options:
                folder: 'user-data://products'
                formatter:
                    class: 'Grav\Framework\File\Formatter\MarkdownFormatter'
```

### Вызов своего метода из Twig

Когда класс подключён, каждый объект директории становится экземпляром `ProductObject`, поэтому ваш метод доступен везде, где вы перебираете коллекцию.

```twig
{% set products = flex_objects('products') %}
{% for product in products %}
    <li>{{ product.title }} ({{ product.getPriceWithTax|number_format(2) }})</li>
{% endfor %}
```

> [!IMPORTANT] Важно
> Пользовательский класс вступает в силу только после пересборки кэша директории. Запустите `bin/grav clear` после добавления или изменения классов в `config.data`, чтобы Flex подхватил новый тип объекта.

## Регистрация типа директории из плагина

Большинство директорий активируются в Admin Next включением файла схемы, что дописывает его путь в конфигурацию плагина. Плагин может зарегистрировать директорию сам, тогда тип существует сразу после активации плагина, без ручного шага для владельца сайта.

Фреймворк вызывает `Grav\Events\FlexRegisterEvent` при первом обращении к `$grav['flex']`. Событие несёт экземпляр `Flex` в свойстве `flex`. Подпишитесь на него, а затем вызовите `addDirectoryType()` с именем типа и путём к схеме.

```php
<?php
namespace Grav\Plugin;

use Grav\Common\Plugin;
use Grav\Events\FlexRegisterEvent;

class MyPluginPlugin extends Plugin
{
    public static function getSubscribedEvents(): array
    {
        return [
            FlexRegisterEvent::class => [
                ['onRegisterFlex', 0],
            ],
        ];
    }

    public function onRegisterFlex(FlexRegisterEvent $event): void
    {
        $flex = $event->flex;

        $flex->addDirectoryType(
            'products',
            'plugin://my-plugin/blueprints/flex-objects/products.yaml'
        );
    }
}
```

Первый аргумент — имя типа (ключ, который вы передаёте в `flex_objects('products')` и используете в маршрутах REST). Второй — путь потока к схеме. `addDirectoryType()` также принимает необязательный третий аргумент `array $config`, который накладывается поверх собственной конфигурации схемы; это полезно для переопределения путей хранения в разных окружениях.

Именно так плагин регистрирует директории, включённые пользователем: в `flex-objects.php` метод `onRegisterFlex()` читает настроенный список директорий и передаёт каждую схему в `registerDirectories()`, который вызывает `addDirectoryType()` для каждого типа, ещё не включённого.

> [!IMPORTANT] Важно
> `FlexRegisterEvent` вызывается только один раз, при первом обращении к Flex. Регистрируйте тип здесь, а не в `onPluginsInitialized`, иначе к моменту выполнения вашего кода директория может быть уже построена.

## Маршрутизация фронтенда с `flex.router.{name}` {: #frontend-routing-with-flex-router-name }

Тип страницы flex-objects позволяет странице работать как фронт-контроллер для директории (см. [Отображение на фронтенде](../frontend/index.md)). В основе этого механизма лежит событие с динамическим именем.

Когда разрешается страница, в заголовке которой есть блок `flex` со значением `router`, обработчик `onPagesInitialized` плагина собирает событие маршрутизации и вызывает `flex.router.{router}`, где `{router}` — строка из заголовка. Это позволяет любому плагину иметь именованный маршрутизатор, не правя ядро.

```yaml
# В frontmatter страницы
flex:
    directory: products
    router: default
```

Событие, передаваемое в `flex.router.default`, несёт всё, что нужно маршрутизатору: `flex`, `directory`, владеющую `page` (она же `parent`), базовый маршрут `base`, оставшийся `path`, объект `route`, разобранные `options`, входящий `request` и `response` по ссылке.

```php
<?php
namespace Grav\Plugin;

use Grav\Common\Plugin;
use RocketTheme\Toolbox\Event\Event;

class MyRouterPlugin extends Plugin
{
    public static function getSubscribedEvents(): array
    {
        return [
            'flex.router.default' => [
                ['onFlexRouterDefault', 0],
            ],
        ];
    }

    public function onFlexRouterDefault(Event $event): Event
    {
        $directory = $event['directory'];
        $path      = $event['path']; // оставшиеся сегменты URL после базовой страницы

        // Находим объект по пути и рендерим его напрямую.
        $object = $path ? $directory->getObject($path) : null;
        if ($object) {
            // $object->render($layout) возвращает отрендеренный HTML объекта.
            $html = (string) $object->render('default');
            $event['response'] = new \Grav\Framework\Psr7\Response(200, [], $html);
        }

        return $event;
    }
}
```

С тем, что вы возвращаете, плагин делает две вещи: если вы зададите `response` по ссылке как ответ PSR-7, Grav сразу отправит его и остановится. Иначе, если вы зададите в событии `page`, Grav подменит эту страницу активной и остановит дальнейшую маршрутизацию. Если не задать ни того, ни другого, продолжит работать обычная обработка «не найдено».

> [!NOTE] Примечание
> Плагин поставляется с маршрутизатором `default` для типа страницы flex-objects, поэтому большинству сайтов свой маршрутизатор не нужен. Беритесь за собственный `flex.router.{name}`, только когда нужно поведение маршрутизации, которого нет во встроенном маршрутизаторе.

## Контракт событий сохранения и удаления

Есть два семейства событий записи, и понимание разницы между ними сильно экономит путаницу.

### События объектов ядра (срабатывают всегда)

`Grav\Framework\Flex\FlexObject::save()` вызывает `triggerEvent('onBeforeSave')` перед записью и `triggerEvent('onAfterSave')` после. `delete()` делает то же с `onBeforeDelete` и `onAfterDelete`. Они сопоставлены публичным именам событий Grav:

| Внутренний триггер  | Событие Grav                  | Когда срабатывает                        |
| :---------------- | :-------------------------- | :--------------------------- |
| `onBeforeSave`    | `onFlexObjectBeforeSave`    | до записи объекта |
| `onAfterSave`     | `onFlexObjectAfterSave`     | после записи объекта  |
| `onBeforeDelete`  | `onFlexObjectBeforeDelete`  | до удаления объекта |
| `onAfterDelete`   | `onFlexObjectAfterDelete`   | после удаления объекта  |

Они срабатывают при каждом сохранении и удалении независимо от того, откуда пришла запись (админка, REST API или ваш собственный PHP), поэтому `onFlexObjectAfterSave` — правильное место для побочных эффектов вроде очистки кэша или отправки уведомления.

```php
<?php
public function onFlexObjectAfterSave(Event $event): void
{
    // Реагируем только на одну директорию.
    $object = $event['object'];
    if ($object->getFlexType() !== 'products') {
        return;
    }

    // ... переиндексация, уведомление, сброс кэша и т. д.
}
```

### События совместимости с админкой (можно отключить)

Для обратной совместимости с плагинами, написанными под классический поток данных, типы Flex Pages и Users при сохранении также вызывают общие события `onAdminSave` и `onAdminAfterSave`. Каждое несёт событие со значением `type` равным `flex`, а также `directory` и `object`. Они срабатывают только в админке и только когда конфигурация `object.compat.events` директории оставлена по умолчанию (`true`).

Если директории этот мостик совместимости не нужен (или старый плагин ведёт себя неправильно на ваших Flex-объектах), отключите его в схеме:

```yaml
config:
    object:
        compat:
            events: false
```

> [!WARNING] Предупреждение
> Во время `onAdminSave` нельзя заменять объект другим экземпляром. Ядро бросает `RuntimeException`, если объект подменён, потому что логика сохранения ниже всё ещё держит исходный. Изменяйте объект на месте.

## Преобразование сохранённых данных через CLI

Плагин поставляется с одной командой CLI для обслуживания данных: `convert-data`, которая переписывает файл данных из одного формата в другой (JSON и YAML).

```bash
bin/plugin flex-objects convert-data --in user/data/products.json --out yaml
```

Параметр `--in` (`-i`) — путь к исходному файлу, а `--out` (`-o`) — целевой формат. Это удобно, когда вы меняете форматтер хранения директории и нужно перенести существующий файл SimpleStorage в соответствующий вид.

## Что читать дальше

- [Хранилище и конфигурация](../custom-types/blueprint-reference/index.md#config-data) — о блоке `config.data`, в который подключаются эти классы.
- [Права доступа](../custom-types/blueprint-reference/index.md#permissions) — прежде чем открывать зарегистрированную плагином директорию через REST API.
- [Отображение на фронтенде](../frontend/index.md) — о типе страницы, на котором работает событие маршрутизации.
