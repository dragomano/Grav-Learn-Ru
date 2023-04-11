---
description: Учимся создавать плагины для Grav CMS. Краткое руководство.
---

# Создание плагинов

Плагины обычно разрабатываются, потому что есть задача, которая не может быть выполнена с помощью основной функциональности Grav.

В этом уроке мы создадим плагин, который поможет Grav доставить случайную страницу пользователю.  Вы, вероятно, видели подобную функциональность на сайтах блогов как способ предоставления случайного сообщения в блоге, когда вы нажимаете кнопку.

!!! info ""

    Поскольку уже существует подключаемый модуль, выполняющий эту работу с именем `Random`, мы назовем этот тестовый модуль `Randomizer`.

Эта функция недоступна **из коробки**, но **легко** предоставляется через плагин. Как и в случае со многими аспектами Grav, для этого не существует _единого_ пути. Вместо этого у вас есть много вариантов. Мы рассмотрим только один подход...

## Обзор плагина Randomizer

Для нашего плагина мы воспользуемся следующим подходом:

1. Активируем плагин, если URI соответствует настроенному нами `триггерному маршруту`. (Например, `/random`)

2. Создадим фильтр, чтобы в пул случайных страниц попадали только настроенные таксономии. (Например, `category: blog`)

3. Найдём случайную страницу из нашего отфильтрованного пула и скажем Grav использовать её для содержимого страницы.

ХОРОШО! Звучит достаточно просто, правда? Итак, приступим!


## Шаг 1 - Установка плагина DevTools

!!! warning ""

    Предыдущие версии этого руководства требовали создания плагина вручную. Весь этот процесс можно пропустить благодаря нашему новому **плагину DevTools**

Первый шаг в создании нового плагина — это **установить плагин DevTools**. Это можно сделать двумя способами.

#### Установка через CLI GPM

* Перейдите в командной строке к корню вашей установки Grav

```bash
bin/gpm install devtools
```

#### Установка через плагин админки

* После входа в систему просто перейдите в раздел **Плагины** на боковой панели.
* Нажмите кнопку :material-plus: **Добавить** в правом верхнем углу.
* Найдите **DevTools** в списке и нажмите кнопку :material-plus: **Установить**.

## Шаг 2 - Создание плагина Randomizer

Для этого следующего шага вам действительно нужно находиться в [командной строке](/cli-console/command-line-intro), поскольку DevTools предоставляет несколько команд CLI, чтобы значительно упростить процесс создания нового плагина!

В корне вашей установки Grav введите следующую команду:

```bash
bin/plugin devtools new-plugin
```

Этот процесс задаст вам несколько вопросов, которые необходимы для создания нового плагина:

```bash
bin/plugin devtools new-plugin
Enter Plugin Name: Randomizer
Enter Plugin Description: Sends the user to a random page
Enter Developer Name: Acme Corp
Enter Developer Email: contact@acme.co

SUCCESS plugin Randomizer -> Created Successfully

Path: /www/user/plugins/randomizer

Make sure to run `composer update` to initialize the autoloader
```

!!! info ""

    На этом этапе вам **нужно запустить** `composer update` во вновь созданной папке плагина.

Команда DevTools сообщает вам, где был создан этот новый плагин. Созданный плагин полностью функционален, но не будет автоматически иметь логику для выполнения желаемой функции. Нам придется изменить его в соответствии с нашими потребностями.

## Шаг 3 - Основы плагина

Теперь мы создали новый плагин, который можно изменять и развивать. Давайте разберемся и посмотрим, из чего состоит плагин. Если вы заглянете в папку `user/plugins/randomizer`, вы увидите:

```text
.
├── CHANGELOG.md
├── LICENSE
├── README.md
├── blueprints.yaml
├── randomizer.php
└── randomizer.yaml
```

Это примерная структура, но необходимы некоторые вещи:

### Необходимые элементы для работы

Эти элементы очень важны, и ваш плагин не будет работать надежно, если вы не включите их в свой плагин.

* **`blueprints.yaml`** - файл конфигурации, используемый Grav для получения информации о вашем плагине. Он также может определять форму, которую администратор может отображать при просмотре сведений о плагине. Эта форма позволит вам сохранить настройки плагина. [Этот файл задокументирован в главе «Формы»](/forms/blueprints).
* **`randomizer.php`** - этот файл будет называться в соответствии с вашим плагином, но может использоваться для размещения любой логики, необходимой вашему плагину. Вы можете использовать любой [перехват событий плагина](../event-hooks) для выполнения логики практически в любой точке [жизненного цикла Grav](../grav-lifecycle).
* **`randomizer.yaml`** - это конфигурация, используемая плагином для установки параметров, которые плагин может использовать. Он должен называться так же, как и файл `.php`.

### Необходимые элементы для выпуска

Эти элементы необходимы, если вы хотите выпустить свой плагин через GPM.

* **`CHANGELOG.md`** - Файл, соответствующий [формату журнала изменений Grav](/advanced/grav-development/#format-spiska-izmenenii) для отображения изменений в выпусках.
* **`LICENSE`** - файл лицензии, вероятно, должен быть MIT, если у вас нет особой потребности в чем-то еще.
* **`README.md`** - "Readme", который должен содержать любую документацию для плагина. Как установить, настроить и использовать.

## Шаг 4 - Настройка плагина

Как мы описали в **Обзоре плагина**, нам нужно иметь несколько параметров конфигурации для нашего плагина, поэтому файл `randomizer.yaml` должен выглядеть примерно так:

```yaml
enabled: true
active: true
route: /random
filters:
    category: blog
```

Это позволяет нам иметь несколько фильтров, если мы хотим, но пока мы просто хотим, чтобы весь контент с таксономией `category: blog` имел право на случайный выбор.

Все плагины должны иметь опцию «включен». Если в конфигурации всего сайта установлено значение false, ваш плагин
никогда не будет инициализирован Grav. Все плагины также имеют опцию `active`. Если это `false` во всем сайте
конфигурации, каждая страница должна будет активировать ваш плагин. Обратите внимание, что несколько плагинов также поддерживают `enabled`/`active` в
обложку страницы с помощью `mergeConfig`, подробно описанного ниже.

!!! info ""

    При установке Grav по умолчанию таксономия определена для `category` и `tag` по умолчанию. Эту конфигурацию можно изменить в вашем файле `user/config/site.yaml`.

Конечно, как и в случае со всеми другими конфигурациями в Grav, рекомендуется не трогать эту конфигурацию по умолчанию для повседневного управления. Вместо этого вам следует создать переопределение в файле с именем `/user/config/plugins/randomizer.yaml` для размещения любых пользовательских настроек. Этот предоставляемый плагином `randomizer.yaml` действительно предназначен для установки некоторых разумных значений по умолчанию для вашего плагина.

## Шаг 5 - Базовая структура плагина

Структура базового класса плагина уже будет выглядеть примерно так:

```php
<?php

namespace Grav\Plugin;

use Composer\Autoload\ClassLoader;
use Grav\Common\Plugin;
use RocketTheme\Toolbox\Event\Event;

/**
 * Class RandomizerPlugin
 * @package Grav\Plugin
 */
class RandomizerPlugin extends Plugin
{
    /**
     * Composer autoload.
     *
     * @return ClassLoader
     */
    public function autoload(): ClassLoader
    {
        return require __DIR__ . '/vendor/autoload.php';
    }
}
```

Нам нужно добавить несколько операторов `use`, потому что мы собираемся использовать эти классы в нашем плагине, и это экономит место и делает код более читабельным, если нам не нужно помещать полное пространство имен для каждого класса в строку.

Измените операторы `use`, чтобы они выглядели так:

```php
<?php

use Composer\Autoload\ClassLoader;
use Grav\Common\Plugin;
use Grav\Common\Page\Collection;
use Grav\Common\Uri;
use Grav\Common\Taxonomy;
```

Двумя ключевыми частями этой структуры классов являются:

1. Плагины должны иметь `namespace Grav\Plugin` в верхней части PHP-файла.
2. Плагины должны быть названы в **CamelCase** на основе имени плагина со строкой `Plugin`, добавленной в конец, и должен расширять `Plugin`, отсюда и название класса `RandomizerPlugin`.

## Шаг 6 - События с подпиской

Grav использует сложную систему событий, и для обеспечения оптимальной производительности все плагины проверяются Grav, чтобы определить, на какие события плагин подписан.

```php
<?php

public static function getSubscribedEvents(): array
{
    return [
        'onPluginsInitialized' => [
            ['autoload', 100000], // TODO: Remove when plugin requires Grav >=1.7
            ['onPluginsInitialized', 0]
        ]
    ];
}
```

В этом плагине мы собираемся сообщить Grav, что мы подписываемся на событие `onPluginsInitialized`. Таким образом, мы можем использовать это событие (которое является первым событием, доступным для плагинов), чтобы определить, следует ли нам подписываться на другие события.

!!! note ""

    Первый прослушиватель событий `autoload` нужен только в Grav 1.6. Grav 1.7 автоматически вызывает этот метод.

## Шаг 7 - Определение необходимости запуска плагина

Следующим шагом является добавление метода в наш класс `RandomizerPlugin` для обработки события `onPluginsInitialized`, поэтому он активируется только тогда, когда пользователь пытается перейти к маршруту, который мы настроили в нашем файле `randomizer.yaml`. Замените текущую логику плагина 'Example' на следующую:


```php
<?php

public function onPluginsInitialized(): void
{
    // Don't proceed if we are in the admin plugin
    if ($this->isAdmin()) {
        return;
    }

    /** @var Uri $uri */
    $uri = $this->grav['uri'];
    $config = $this->config();

    $route = $config['route'] ?? null;
    if ($route && $route == $uri->path()) {
        $this->enable([
            'onPageInitialized' => ['onPageInitialized', 0]
        ]);
    }
}
```

Во-первых, мы получаем объект Uri **из контейнера инъекции зависимостей**. Он содержит всю информацию о текущем URI, включая информацию о маршруте.

Объект **config** уже является частью базового **Plugin**, поэтому мы можем просто использовать его, чтобы получить значение конфигурации для нашего настроенного `route`.

Затем мы сравниваем настроенный маршрут с текущим путем URI. Если они равны, мы сообщаем диспетчеру, что наш плагин также будет прослушивать новое событие: `onPageInitialized`.

Используя этот подход, мы гарантируем, что не будем запускать какой-либо дополнительный код, если в этом нет необходимости. Практика, подобная этой, гарантирует, что ваш сайт будет работать как можно быстрее.

## Шаг 8 — Отображение случайной страницы

Последним шагом нашего плагина является отображение случайной страницы, и мы можем сделать это, добавив следующий метод:

```php
<?php

/**
 * Send user to a random page
 */
public function onPageInitialized(): void
{
    /** @var Taxonomy $uri */
    $taxonomy_map = $this->grav['taxonomy'];
    $config = $this->config();

    $filters = (array)($config['filters'] ?? []);
    $operator = $config['filter_combinator'] ?? 'and';

    if (count($filters) > 0) {
        $collection = new Collection();
        $collection->append($taxonomy_map->findTaxonomy($filters, $operator)->toArray());
        if (count($collection) > 0) {
            unset($this->grav['page']);
            $this->grav['page'] = $collection->random()->current();
        }
    }
}
```

Этот метод немного сложнее, поэтому мы перейдем к тому, что происходит:

1. Во-первых, мы получаем объект **Taxonomy** из контейнера **Grav DI** и присваиваем его переменной `$taxonomy_map`.

2. Затем мы извлекаем массив фильтров из нашей конфигурации плагина. В нашей конфигурации это массив с 1 элементом: ['category'=>'blog'].

3. Убедитесь, что у нас есть фильтры, а затем создайте новую `коллекцию` в переменной `$collection` для хранения наших страниц.

4. Добавьте все страницы, соответствующие фильтру, в нашу переменную `$collection`.

5. Снимите значение текущего объекта `page`, о котором знает Grav.

6. Установите текущую `page` на случайный элемент в коллекции.


## Шаг 9 - Очистка

Пример плагина, созданного с помощью плагина **DevTools**, использовал событие под названием `onPageContentRaw()'. Это событие не используется в нашем новом плагине, поэтому мы можем безопасно удалить всю функцию.

## Шаг 10 - Итог

И это все, что в нем есть! Теперь плагин завершен. Ваш полный класс плагинов должен выглядеть примерно так:

```php
<?php

namespace Grav\Plugin;

use Composer\Autoload\ClassLoader;
use Grav\Common\Plugin;
use Grav\Common\Page\Collection;
use Grav\Common\Uri;
use Grav\Common\Taxonomy;

/**
 * Class RandomizerPlugin
 * @package Grav\Plugin
 */
class RandomizerPlugin extends Plugin
{
    /**
     * @return array
     *
     * The getSubscribedEvents() gives the core a list of events
     *     that the plugin wants to listen to. The key of each
     *     array section is the event that the plugin listens to
     *     and the value (in the form of an array) contains the
     *     callable (or function) as well as the priority. The
     *     higher the number the higher the priority.
     */
    public static function getSubscribedEvents(): array
    {
    return [
        'onPluginsInitialized' => [
            ['autoload', 100000], // TODO: Remove when plugin requires Grav >=1.7
            ['onPluginsInitialized', 0]
        ]
    ];
    }

    /**
     * Composer autoload.
     *
     * @return ClassLoader
     */
    public function autoload(): ClassLoader
    {
        return require __DIR__ . '/vendor/autoload.php';
    }

    public function onPluginsInitialized(): void
    {
        // Don't proceed if we are in the admin plugin
        if ($this->isAdmin()) {
            return;
        }

        /** @var Uri $uri */
        $uri = $this->grav['uri'];
        $config = $this->config();

        $route = $config['route'] ?? null;
        if ($route && $route == $uri->path()) {
            $this->enable([
                'onPageInitialized' => ['onPageInitialized', 0]
            ]);
        }
    }

    /**
     * Send user to a random page
     */
    public function onPageInitialized(): void
    {
        /** @var Taxonomy $uri */
        $taxonomy_map = $this->grav['taxonomy'];
        $config = $this->config();

        $filters = (array)($config['filters'] ?? []);
        $operator = $config['filter_combinator'] ?? 'and';

        if (count($filters) > 0) {
            $collection = new Collection();
            $collection->append($taxonomy_map->findTaxonomy($filters, $operator)->toArray());
            if (count($collection) > 0) {
                unset($this->grav['page']);
                $this->grav['page'] = $collection->random()->current();
            }
        }
    }
}
```

Если вы последуете этому примеру, у вас должен быть полнофункциональный плагин **Randomizer**, включенный для вашего сайта. Просто укажите ваш браузер на `https://yoursite.com/random`, и вы должны увидеть случайную страницу. Вы также можете загрузить оригинальный плагин **Random** прямо с сайта [getgrav.org](https://getgrav.org/downloads/plugins).

## Слияние плагина и конфигурации страницы

Одной из популярных методик, которая используется в различных плагинах, является концепция объединения конфигурации плагина (либо по умолчанию, либо переопределенной конфигурации пользователя) с конфигурацией на уровне страниц. Это означает, что вы можете установить конфигурацию **site-wide**, а затем иметь определенную конфигурацию для данной страницы по мере необходимости. Это обеспечивает много мощности и гибкости для вашего плагина.

В последних версиях Grav был добавлен вспомогательный метод для автоматического выполнения этой функции, а не для того, чтобы вам приходилось кодировать эту логику самому. Плагин **SmartyPants** является хорошим примером этой функциональности в действии:

```php
<?php

public function onPageContentProcessed(Event $event)
{
    $page = $event['page'];
    $config = $this->mergeConfig($page);

    if ($config->get('process_content')) {
        $page->setRawContent(\Michelf\SmartyPants::defaultTransform(
            $page->getRawContent(),
            $config->get('options')
        ));
    }
}
```

## Реализация CLI в вашем плагине

Плагины также имеют возможность интеграции с командной строкой `bin/plugin` для выполнения задач. Вы можете следовать [расширенному руководству](/cli-console/grav-cli-plugin), если хотите реализовать такую ​​функциональность.
