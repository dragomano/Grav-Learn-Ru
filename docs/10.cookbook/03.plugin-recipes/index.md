# Рецепты плагинов

> Список проблем и решений, связанных с плагинами Grav CMS.

Эта страница содержит целый ряд проблем и их соответствующих решений, связанных с плагинами Grav.

## На выходе какой-то PHP-код в шаблон Twig

#### Цель:

Вы хотите обработать некоторый пользовательский PHP-код и сделать результат доступным на странице.

#### Решение:

Вы создаете новый плагин, который создает расширение Twig и делает часть содержимого PHP доступной в ваших шаблонах Twig.

Создайте новую папку плагина в `user/plugins/example` и добавьте эти файлы:

`user/plugins/example/example.php`
`user/plugins/example/example.yaml`
`user/plugins/example/twig/ExampleTwigExtension.php`

В `twig/ExampleTwigExtension.php` вы выполните свою собственную обработку и вернете ее в виде строки в `exampleFunction()`.

Затем в вашем файле шаблона Twig (или в файле Markdown страницы, если вы включили обработку Twig в Pages) визуализируйте вывод, используя: `{{ example() }}`.

Обзор окончен, давайте посмотрим на код:

`example.php`:

```php
<?php
namespace Grav\Plugin;
use \Grav\Common\Plugin;
class ExamplePlugin extends Plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onTwigExtensions' => ['onTwigExtensions', 0]
        ];
    }
    public function onTwigExtensions()
    {
        require_once(__DIR__ . '/twig/ExampleTwigExtension.php');
        $this->grav['twig']->twig->addExtension(new ExampleTwigExtension());
    }
}
```

`ExampleTwigExtension.php`:

```php
<?php
namespace Grav\Plugin;
use Grav\Common\Twig\Extension\GravExtension;

class ExampleTwigExtension extends GravExtension
{
    public function getName()
    {
        return 'ExampleTwigExtension';
    }
    public function getFunctions(): array
    {
        return [
            new \Twig_SimpleFunction('example', [$this, 'exampleFunction'])
        ];
    }
    public function exampleFunction()
    {
        return 'something';
    }
}
```

`example.yaml`:

```yaml
enabled: true
```

Теперь плагин установлен и включен, и всё должно работать.

## Фильтрация таксономий с помощью плагина taxonomylist

#### Цель:

Вы хотите использовать [плагин Grav для списка таксономии](https://github.com/getgrav/grav-plugin-taxonomylist) для перечисления тегов, которые используются в ваших сообщениях в блоге, но вместо того, чтобы перечислять их все, вы только хотите перечислить наиболее часто используемые элементы в данной таксономии (например, пять основных тегов).

#### Решение:

Это пример того, как гибкость плагинов Grav действительно пригодится. Первый шаг - убедиться, что у вас установлен [плагин Grav для списка таксономии](https://github.com/getgrav/grav-plugin-taxonomylist) в вашем пакете Grav. После установки убедитесь, что вы скопировали `/yoursite/user/plugins/taxonomylist/templates/partials/taxonomylist.html.twig` в `/yoursite/user/themes/yourtheme/templates/partials/taxonomylist.html.twig`, поскольку мы будем вносить изменения в этот файл.

Чтобы эта работа работала, мы собираемся ввести три новые переменные: `filter`, `filterstart` и `filterend`, где

* **filter** - это логическое значение, которому будет присвоено значение `true`, если мы хотим иметь возможность перечислять только несколько верхних тегов (или любую другую таксономию, которую вы хотите использовать).
* **filterstart** - произвольное целое число, но обычно его следует установить равным нулю. Это индекс в массиве таксономии, с которого вы хотите начать.
* **filterend** - произвольное целое число и индекс в массиве таксономии, которым вы хотите закончить. Обратите внимание, что если вы хотите перечислить пять лучших элементов в своей таксономии, вы должны установить это значение в 5, так как наш цикл будет повторяться до `filterend -1`.

Следующим шагом будет вызов `taxonomylist.html.twig` в шаблоне, в котором мы хотим перечислить основные элементы в нашей таксономии. Как обычно, мы будем делать это с помощью `{% include %}`, как показано в следующем примере фрагмента:

```twig
{% if config.plugins.taxonomylist.enabled %}
<div class="sidebar-content">
    <h4>Popular Tags</h4>
    {% include 'partials/taxonomylist.html.twig' with {'taxonomy':'tag', filter: true, filterstart: 0, filterend: 5} %}
</div>
{% endif %}
```
В этом примере мы собираемся перечислить пять самых популярных тегов.

Теперь давайте обратим наше внимание на `taxonomylist.html.twig`. Для справки, вот код по умолчанию для этого файла при первоначальной установке:

```twig
{% set taxlist = taxonomylist.get() %}

{% if taxlist %}
    <span class="tags">
        {% for tax,value in taxlist[taxonomy] %}
            <a href="{{ base_url }}/{{ taxonomy }}{{ config.system.param_sep }}{{ tax|e('url') }}">{{ tax }}</a>
        {% endfor %}
    </span>
{% endif %}
```

Чтобы это работало с нашими новыми переменными (например, `filter`, `filterstart` и `filterend`), нам нужно будет включить их в этот файл следующим образом:

```twig
{% set taxlist = taxonomylist.get %}

{% if taxlist %}
    {% set taxlist_taxonomy = taxlist[taxonomy] %}

    {% if filter %}
        {% set taxlist_taxonomy = taxlist_taxonomy|slice(filterstart,filterend) %}
    {% endif %}

    <span class="tags">
        {% for tax,value in taxlist_taxonomy %}
            <a href="{{ base_url }}/{{ taxonomy }}{{ config.system.param_sep }}{{ tax|e('url') }}">{{ tax }}</a>
        {% endfor %}
    </span>
{% endif %}
```

Здесь файл сначала проверяет, установлен ли `filter` в значение `true`. Если это так, цикл for выполняется так же, как и в исходном файле `taxonomylist.html.twig`, но на этот раз он использует фильтр Twig `slice`. Этот `filter`, в нашем случае, извлечет подмножество массива от начального индекса (в нашем случае `filterstart`) до конечного индекса (в нашем случае `filterend -1`).

Если, с другой стороны, для переменной `filter` установлено значение `false` или она не найдена, будут перечислены все элементы в вашей таксономии.

## Добавление кнопки поиска в плагин SimpleSearch

#### Цель:

Вам действительно нравится [плагин Grav SimpleSearch](https://github.com/getgrav/grav-plugin-simplesearch), но вы хотите добавить кнопку поиска в дополнение к текстовому полю. Одна из причин для добавления этой кнопки заключается в том, что пользователю может быть не очевидно, что ему нужно нажать клавишу «Enter», чтобы инициировать свой поисковый запрос.

#### Решение:

Сначала убедитесь, что вы установили [плагин Grav SimpleSearch](https://github.com/getgrav/grav-plugin-simplesearch). Затем убедитесь, что вы скопировали `/yoursite/user/plugins/simplesearch/templates/partials/simplesearch-searchbox.html.twig` в `/yoursite/user/themes/yourtheme/templates/partials/simplesearch-searchbox.html.twig`, так как нам нужно будет внести изменения в этот файл.

Прежде чем идти дальше, давайте рассмотрим, что делает этот файл:

```twig
<input type="text" placeholder="Search..." value="{{ query }}" data-search-input="{{ base_url }}{{ config.plugins.simplesearch.route}}/query" />
<script>
jQuery(document).ready(function($){
    var input = $('[data-search-input]');
    input.on('keypress', function(event) {
        if (event.which == 13 && input.val().length > 3) {
            event.preventDefault();
            window.location.href = input.data('search-input') + '{{ config.system.param_sep }}' + input.val();
        }
    });
});
</script>
```

Первая строка просто вставляет поле ввода текста в ваш шаблон Twig. Атрибут `data-search-input` хранит базовый URL страницы результирующего запроса. По умолчанию `http://yoursite/search/query`.

Теперь перейдем к jQuery ниже. Здесь тег, содержащий атрибут `data-search-input`, назначается переменной input. Затем к `input` применяется метод jQuery `.on()`. Метод `.on()` применяет обработчики событий к выбранным элементам (в данном случае к текстовому полю `<input>`). Итак, когда пользователь нажимает (`keypress`) клавишу, чтобы начать поиск, оператор `if` проверяет, что следующие элементы являются `истинными`:

1. Была нажата клавиша `Enter`: `event.which == 13`, где 13 - числовое значение клавиши `Enter` на клавиатуре.
2. Количество символов, введенных в поле поиска, больше трех. Вы можете настроить это по своему вкусу, поскольку в вашей организации может быть много сокращений, состоящих из трех или менее символов.

Если они верны, то `event.preventDefault();` гарантирует, что действие браузера по умолчанию для клавиши `Enter` игнорируется, так как это помешает нашему поиску. Наконец, создается полный URL-адрес поискового запроса. По умолчанию это `http://yoursite/search/query:yourquery`. Отсюда `/yoursite/user/plugins/simplesearch/simplesearch.php` выполняет фактический поиск, а другие файлы Twig в подключаемом модуле выводят результаты.

Нет возврата к нашему решению! Если мы хотим добавить кнопку поиска, мы должны:

1. Добавьте кнопку
2. Обязательно примените к кнопке метод `.on()`, но на этот раз с помощью `click` вместо `keypress`

Это достигается с помощью следующего кода, использующего [Turret CSS Framework](http://bigfishtv.github.io/turret/docs/index.html). Фрагменты кода для других фреймворков будут перечислены в конце.

```
<div class="input-group input-group-search">
	<input type="search" placeholder="Search" value="{{ query }}" data-search-input="{{ base_url }}{{ config.plugins.simplesearch.route}}/query" >
	<span class="input-group-button">
		<button class="button" type="submit">Search</button>
	</span>
</div>

<script>
jQuery(document).ready(function($){
    var input = $('[data-search-input]');
    var searchButton = $('.button.search');

    input.on('keypress', function(event) {
        if (event.which == 13 && input.val().length > 3) {
            event.preventDefault();
            window.location.href = input.data('search-input') + '{{ config.system.param_sep }}' + input.val();
        }
    });

    searchButton.on('click', function(event) {
        if (input.val().length > 3) {
            event.preventDefault();
            window.location.href = input.data('search-input') + '{{ config.system.param_sep }}' + input.val();
        }
    });
});
</script>
```

Атрибуты HTML и class специфичны для Turret, но конечный результат будет [что-то вроде этого](http://bigfishtv.github.io/turret/docs/index.html#input-group). Мы также можем видеть, что метод `.on()` также был назначен кнопке поиска, но он проверяет только то, что количество символов, введенных в поле поиска, больше трех перед выполнением кода в операторе `if`.

Вот HTML-код по умолчанию для текстового поля и кнопка поиска для нескольких других фреймворков:

[**Bootstrap**](http://getbootstrap.com/)
```
<div class="input-group">
    <input type="text" class="form-control" placeholder="Search for...">
    <span class="input-group-btn">
        <button class="btn btn-default" type="button">Go!</button>
    </span>
</div>
```

[**Materialize**](http://materializecss.com/)
```
<div class="input-field">
    <input id="search" type="search" required>
    <label for="search"><i class="material-icons">search</i></label>
</div>
```

[**Pure CSS**](http://purecss.io)
```
<form class="pure-form">
    <input type="text" class="pure-input-rounded">
    <button type="submit" class="pure-button">Search</button>
</form>
```

[**Semantic UI**](http://semantic-ui.com/)
```
<div class="ui action input">
  <input type="text" placeholder="Search...">
  <button class="ui button">Search</button>
</div>
```

## Итерации по страницам и медиа

#### Цель:

Вы хотите получить доступ ко всем страницам и связанным с каждой страницей носителям через PHP и/или Twig, чтобы над ними можно было зацикливаться или иным образом манипулировать плагином.

#### Решение:

Используйте возможности Grav для создания рекурсивного индекса всех страниц, а при индексировании также собирайте медиа-файлы для каждой страницы. Плагин [DirectoryListing](https://github.com/OleVik/grav-plugin-directorylisting/blob/v2.0.0-rc.2/Utilities.php#L64-L105) делает именно это и создает HTML- список с использованием созданной древовидной структуры. Для этого мы создадим рекурсивную функцию - или метод, который может иметь место в классе плагина, - который просматривает каждую страницу и сохраняет ее в массиве. Метод рекурсивен, потому что он снова вызывает себя для каждой найденной страницы, имеющей дочерние элементы.

Но обо всем по порядку, метод принимает три параметра: первый - это `$route` к странице, который сообщает Grav, где его найти; второй - `$mode`, который сообщает методу, следует ли выполнять итерацию по самой странице или по ее дочерним элементам; третий - `$depth`, который отслеживает, на каком уровне находится страница. Метод сначала создает экземпляр объекта Page, затем работает с глубиной и режимом и создает коллекцию. По умолчанию мы упорядочиваем страницы по дате, по убыванию, но вы можете сделать это настраиваемым. Затем мы создаем массив `$paths` для хранения каждой страницы. Поскольку маршруты в Grav уникальны, они используются как ключи в этом массиве для идентификации каждой страницы.

Теперь мы перебираем страницы, добавляя глубину, заголовок и маршрут (также сохраняемые как значение для облегчения доступа). В цикле foreach мы также пытаемся получить дочерние страницы и добавить их, если они найдены. Также мы находим все медиа, связанные со страницей, и добавляем их. Поскольку метод рекурсивен, он будет продолжать поиск страниц и дочерних страниц до тех пор, пока больше не будет найдено.

Возвращаемые данные представляют собой древовидную структуру или многомерный массив, говоря языком PHP, содержащий все страницы и их носители. Это можно передать в Twig или использовать в самом плагине. Обратите внимание, что с очень большими структурами папок PHP может выйти из строя или выйти из строя из-за ограничений рекурсии, например. папки глубиной 100 или более уровней.

```php
/**
 * Creates page-structure recursively
 * @param string $route Route to page
 * @param integer $depth Reserved placeholder for recursion depth
 * @return array Page-structure with children and media
 */
public function buildTree($route, $mode = false, $depth = 0)
{
    $page = Grav::instance()['page'];
    $depth++;
    $mode = '@page.self';
    if ($depth > 1) {
        $mode = '@page.children';
    }
    $pages = $page->evaluate([$mode => $route]);
    $pages = $pages->published()->order('date', 'desc');
    $paths = array();
    foreach ($pages as $page) {
        $route = $page->rawRoute();
        $path = $page->path();
        $title = $page->title();
        $paths[$route]['depth'] = $depth;
        $paths[$route]['title'] = $title;
        $paths[$route]['route'] = $route;
        if (!empty($paths[$route])) {
            $children = $this->buildTree($route, $mode, $depth);
            if (!empty($children)) {
                $paths[$route]['children'] = $children;
            }
        }
        $media = new Media($path);
        foreach ($media->all() as $filename => $file) {
            $paths[$route]['media'][$filename] = $file->items()['type'];
        }
    }
    if (!empty($paths)) {
        return $paths;
    } else {
        return null;
    }
}
```

## Плагин пользовательских шаблонов Twig

#### Цель:

Вместо использования наследования тем можно создать очень простой плагин, который позволит вам использовать настраиваемое расположение для предоставления настраиваемых шаблонов Twig.

#### Решение:

Единственное, что вам нужно в этом плагине, - это событие, чтобы указать местоположение для ваших шаблонов. Самый простой способ создать плагин - использовать плагин devtools. Итак, установите это с помощью:

```bash
$ bin/gpm install devtools
```

После установки создайте новый плагин с помощью команды:

```bash
$ bin/plugin devtools newplugin
```

Заполните данные для имени, автора и т. д. Скажем, мы называем это `Custom Templates`, и плагин будет создан в `/user/plugins/custom-templates`. Все, что вам нужно сделать сейчас, это отредактировать файл `custom-templates.php` и поместить этот код:

```php
<?php
namespace Grav\Plugin;

use \Grav\Common\Plugin;

class CustomTemplatesPlugin extends Plugin
{
    /**
     * Subscribe to required events
     *
     * @return array
     */
    public static function getSubscribedEvents()
    {
        return [
            'onTwigTemplatePaths' => ['onTwigTemplatePaths', 0]
        ];
    }

    /**
     * Add current directory to twig lookup paths.
     */
    public function onTwigTemplatePaths()
    {
        $this->grav['twig']->twig_paths[] = __DIR__ . '/templates';
    }
}
```

Этот плагин просто подписывается на событие `onTwigTemplatePaths()`, а затем в этом методе события он добавляет папку `user/plugins/custom-templates/templates` к этому из путей, которые Twig будет проверять.

Это позволяет вам добавить шаблон Twig под названием `foo.html.twig`, и тогда любая страница с именем `foo.md` сможет использовать этот шаблон.

?> ПРИМЕЧАНИЕ. Это добавит путь настраиваемого шаблона плагина к **концу** массива путей к шаблону Twig. Это означает, что тема (которая всегда стоит первой) будет иметь приоритет над одноименными шаблонами плагина. Чтобы решить эту проблему, просто поместите путь шаблона плагина перед массивом, изменив метод события:

```twig
    /**
     * Add current directory to twig lookup paths.
     */
    public function onTwigTemplatePaths()
    {
        array_unshift($this->grav['twig']->twig_paths, __DIR__ . '/templates');
    }
```

## Использование кэша в ваших собственных плагинах

#### Цель:

При разработке собственных плагинов часто бывает полезно использовать кэш Grav для кэширования данных для повышения производительности. К счастью, это очень простой процесс использования кэша в вашем собственном коде.

#### Решение:

Это некоторый базовый код, который показывает вам, как работает кэширование:

```php
    $cache = Grav::instance()['cache'];
    $id = 'myplugin-data'
    $list = [];

    if ($data = $cache->fetch($id)) {
        return $data;
    } else {
        $data = $this->gatherData();
        $cache->save($hash, $data);
        return $data;
    }
```

Сначала мы получаем объект кэша Grav, а затем пытаемся проверить, существуют ли наши данные уже в кэше (`$data = $cache->fetch($id)`). Если `$data` существует, просто верните его без необходимости дополнительной работы.

Однако, если выборка кэша возвращает null, то есть он не кэшируется, сделайте некоторую _работу_ и получите данные (`$data = $this->gatherData()`), а затем просто сохраните данные для следующего раза (`$cache->save($hash, $data)`).



## Обучение на примере

Учитывая обилие доступных в настоящее время плагинов, есть вероятность, что вы найдете свои ответы где-то в их исходном коде. Проблема в том, чтобы знать, на какие из них смотреть. Эта страница пытается перечислить общие проблемы плагинов, а затем перечисляет конкретные плагины, которые демонстрируют, как их решать.

Прежде чем продолжить, убедитесь, что вы ознакомились с [основной документацией](/04.plugins/index), особенно с главой [Жизненный цикл Grav](/04.plugins/05.grav-lifecycle/index)!

### Как я могу читать и записывать данные в файловую систему?

Гравитация может быть плоским файлом, но плоский файл &#8800; статичен! Существует множество способов чтения и записи данных в файловую систему.

  * Если вам просто нужен доступ для чтения к данным YAML, проверьте плагин [Import plugin](https://github.com/Deester4x4jr/grav-plugin-import).

  * Предпочтительный интерфейс - через встроенный интерфейс [RocketTheme\Toolbox\File](/14.api/index).

  * Ничто не мешает вам использовать [SQLite](https://sqlite.org/).

  * Самый простой пример - это, вероятно, [плагин комментариев](https://github.com/getgrav/grav-plugin-comments).

  * Другие включают в себя

    * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

    * [Thumb Ratings](https://github.com/iusvar/grav-plugin-thumb-ratings)

    * [Webmention](https://github.com/Perlkonig/grav-plugin-webmention)

### Как сделать данные из плагина доступными для Twig?

Один из способов — через пространство имён `config.plugins.X`. Просто сделайте `$this->config->set()`, как показано в одном из следующих примеров:

  * [ipLocate](https://github.com/Perlkonig/grav-plugin-iplocate/blob/master/iplocate.php#L82)
  * [Count Views](https://github.com/Perlkonig/grav-plugin-count-views/blob/master/count-views.php#L88)

Затем вы можете получить доступ к этому в шаблоне Twig через `{{ config.plugins.X.whatever.variable }}`.

Кроме того, вы можете передавать переменные через `grav['twig']`:

  * [Blogroll](https://github.com/Perlkonig/grav-plugin-blogroll/blob/master/blogroll.php#L43), к которому вы затем можете получить прямой доступ [через ваш шаблон](https://github.com/Perlkonig/grav-plugin-blogroll/blob/master/templates/partials/blogroll.html.twig#L32).

Наконец, вы можете ввести данные непосредственно в заголовок страницы, как показано в [плагине импорта](https://github.com/Deester4x4jr/grav-plugin-import).

### Как я могу ввести Markdown на страницу?

В соответствии с [жизненным циклом Grav](/04.plugins/05.grav-lifecycle/index), последний хук события, в который вы можете ввести необработанный Markdown, — `onPageContentRaw`. Самый ранний из них, вероятно, `onPageInitialized`. Вы можете просто захватить `$this->grav['page']->rawMarkdown()`, изменить его, а затем записать обратно с помощью `$this->grav['page']->setRawContent()`. Следующие плагины демонстрируют это:

  * [Page Inject](https://github.com/getgrav/grav-plugin-page-inject)
  * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

### Как мне внедрить HTML в конечный результат?

Последний раз вы можете ввести HTML и всё ещё кэшировать свои выходные данные во время события `onOutputGenerated`. Вы можете просто захватить и изменить `$this->grav->output`.

  * Многие распространенные задачи могут быть выполнены с помощью [Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core) infrastructure.
  * Плагины [Pubmed](https://github.com/Perlkonig/grav-plugin-pubmed) и [Tablesorter](https://github.com/Perlkonig/grav-plugin-tablesorter) используют более грубый подход.

### Как мне внедрить такие ресурсы, как файлы JavaScript и CSS?

Это делается через интерфейс [Grav\Common\Assets](/14.api/index?id=class-gravcommonassets).

  * [Google Analytics](https://github.com/escopecz/grav-ganalytics)
  * [Bootstrapper](https://github.com/getgrav/grav-plugin-bootstrapper)
  * [Gravstrap](https://github.com/giansi/gravstrap)
  * [Tablesorter](https://github.com/Perlkonig/grav-plugin-tablesorter)

### Как я могу повлиять на заголовки ответов и коды ответов?

Вы можете использовать команду php `header()` для установки заголовков ответов. Последнее, что вы можете сделать, - это во время события onOutputGenerated, после чего выходные данные фактически отправляются клиенту. Сам код ответа может быть задан только в заголовке YAML соответствующей страницы (`http_response_code`).

  * Плагин [Graveyard](https://github.com/Perlkonig/grav-plugin-graveyard) плагин заменяет ответы `404 NOT FOUND` на ответы `410 GONE` через заголовок YAML.
  * Плагин [Webmention](https://github.com/Perlkonig/grav-plugin-webmention) устанавливает заголовок `Location` в ответе `201 CREATED`.

### Как включить сторонние библиотеки в свой плагин?

Обычно вы включаете другие полные библиотеки во вложенную папку `vendor` и подключаете его `autoload.php` там, где это необходимо, в вашем плагине. (Если вы используете Git, подумайте об использовании [subtrees](https://help.github.com/articles/about-git-subtree-merges/).)

  * [Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core)
  * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

### Как мне расширить Twig?

Самый простой способ - следовать примеру [Custom Twig Filter/Function](/10.cookbook/02.twig-recipes/index?id=Пользовательский-фильтрфункция-twig) в разделе **Рецепты Twig**.

Также [прочтите документацию Twig](https://twig.sensiolabs.org/doc/advanced.html) и разработайте свое расширение. Затем посмотрите на плагин [TwigPCRE](https://github.com/kesslernetworks/grav-plugin-twigpcre), чтобы узнать, как включить его в Grav.

### Как мне взаимодействовать с внешними API?

Grav предоставляет объект [Grav\Common\GPM\Response](/14.api/index?id=class-gravcommongpmresponse), но ничто не мешает вам сделать это напрямую, если вы того пожелаете.

  * [ipLocate](https://github.com/Perlkonig/grav-plugin-iplocate)
  * [Pubmed](https://github.com/Perlkonig/grav-plugin-pubmed)
