---
title: "Рецепты плагинов"
description: "Сборник задач и их решений, связанных с плагинами Grav."
tags:
  - "Рецепты"
  - "Разработка"
  - "Плагины"
  - "PHP"
---

# Рецепты плагинов

На этой странице собраны задачи и их решения, связанные с плагинами Grav.

## Вывод результата PHP-кода в шаблоне Twig

<h4 id="goal">Цель:</h4>

Вы хотите обработать какой-то собственный PHP-код и сделать результат доступным на странице.

<h4 id="solution">Решение:</h4>

Вы создаёте новый плагин, который добавляет расширение Twig и делает некий PHP-контент доступным в шаблонах Twig.

Создайте папку нового плагина `user/plugins/example` и добавьте в неё файлы:

`user/plugins/example/example.php`
`user/plugins/example/example.yaml`
`user/plugins/example/twig/ExampleTwigExtension.php`

В `twig/ExampleTwigExtension.php` вы выполняете свою обработку и возвращаете результат строкой в `exampleFunction()`.

Затем в файле шаблона Twig (или в Markdown-файле страницы, если вы включили обработку Twig для страниц) выводите результат так: `{{ example() }}`.

Обзор закончен, перейдём к коду:

`example.php`:

```php linenums="1"
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

```php linenums="1"
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

Плагин установлен и включён, и всё должно работать.

## Фильтрация таксономий с помощью плагина taxonomylist

<h4 id="goal-1">Цель:</h4>

Вы хотите использовать [плагин taxonomylist для Grav](https://github.com/getgrav/grav-plugin-taxonomylist), чтобы выводить теги, которые используются в записях блога, но вместо всех тегов показывать только самые используемые элементы заданной таксономии (например, пять самых популярных тегов).

<h4 id="solution-1">Решение:</h4>

Это пример, где гибкость плагинов Grav действительно пригодится. Первый шаг — убедиться, что [плагин taxonomylist](https://github.com/getgrav/grav-plugin-taxonomylist) установлен в вашей сборке Grav. После установки скопируйте `user/plugins/taxonomylist/templates/partials/taxonomylist.html.twig` в `user/themes/yourtheme/templates/partials/taxonomylist.html.twig`, потому что мы будем вносить изменения в этот файл.

Чтобы всё заработало, мы введём три новые переменные: `filter`, `filterstart` и `filterend`, где

 * **filter** — логическое значение, которое устанавливается в `true`, если нужно вывести только несколько самых популярных тегов (или любую другую нужную вам таксономию).
 *  **filterstart** — произвольное целое число, но обычно его ставят равным нулю. Это индекс в массиве таксономии, с которого нужно начать.
 * **filterend** — произвольное целое число, индекс в массиве таксономии, на котором нужно закончить. Учтите: если нужно вывести пять первых элементов таксономии, установите это значение в 5, потому что цикл будет идти до `filterend -1`.

Следующий шаг — вызвать `taxonomylist.html.twig` в том шаблоне, где мы хотим вывести самые популярные элементы таксономии. Как обычно, сделаем это через `{% include %}`, как в примере ниже:

```twig linenums="1"
{% if config.plugins.taxonomylist.enabled %}
<div class="sidebar-content">
    <h4>Popular Tags</h4>
    {% include 'partials/taxonomylist.html.twig' with {'taxonomy':'tag', filter: true, filterstart: 0, filterend: 5} %}
</div>
{% endif %}
```

В этом примере мы выведем пять самых популярных тегов.

Теперь обратимся к `taxonomylist.html.twig`. Для справки: вот стандартный код этого файла сразу после установки:

```twig linenums="1"
{% set taxlist = taxonomylist.get() %}

{% if taxlist %}
    <span class="tags">
        {% for tax,value in taxlist[taxonomy] %}
            <a href="{{ base_url }}/{{ taxonomy }}{{ config.system.param_sep }}{{ tax|e('url') }}">{{ tax }}</a>
        {% endfor %}
    </span>
{% endif %}
```

Чтобы это работало с нашими новыми переменными (`filter`, `filterstart` и `filterend`), нужно добавить их в этот файл:

```twig linenums="1"
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

Здесь файл по умолчанию собирает все элементы таксономии в переменную `taxlist_taxonomy`.

Если `filter` задан, таксономия использует фильтр Twig `slice`. В нашем случае этот фильтр извлекает подмножество массива от начального индекса (`filterstart`) до конечного (`filterend`).

Цикл `for` выполняется так же, как в исходном `taxonomylist.html.twig`, но с содержимым `taxlist_taxonomy` — отфильтрованным или нет.

## Добавление кнопки поиска в плагин SimpleSearch

<h4 id="goal-2">Цель:</h4>

Вам очень нравится [плагин SimpleSearch для Grav](https://github.com/getgrav/grav-plugin-simplesearch), но вы хотите добавить кнопку поиска в дополнение к текстовому полю. Одна из причин добавить такую кнопку — пользователю может быть неочевидно, что для запуска поиска нужно нажать клавишу `Enter`.

<h4 id="solution-2">Решение:</h4>

Сначала убедитесь, что у вас установлен [плагин SimpleSearch для Grav](https://github.com/getgrav/grav-plugin-simplesearch). Затем скопируйте `user/plugins/simplesearch/templates/partials/simplesearch-searchbox.html.twig` в `user/themes/yourtheme/templates/partials/simplesearch-searchbox.html.twig`, потому что нам нужно будет изменить этот файл.

Прежде чем идти дальше, разберём, что делает этот файл:

```twig linenums="1"
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

Первая строка просто вставляет в шаблон Twig текстовое поле. Атрибут `data-search-input` хранит базовый URL страницы с результатами запроса. По умолчанию это `http://my-site.example/search/query`.

Теперь перейдём к jQuery ниже. Здесь тег с атрибутом `data-search-input` присваивается переменной `input`. Затем к `input` применяется метод jQuery `.on()`. Метод `.on()` назначает обработчики событий выбранным элементам (в данном случае текстовому полю `<input>`). Когда пользователь нажимает клавишу (`keypress`), чтобы запустить поиск, инструкция `if` проверяет, что выполняются следующие условия:

1. Нажата клавиша `Enter`: `event.which == 13`, где 13 — числовой код клавиши `Enter` на клавиатуре.
2. В поле поиска введено больше трёх символов. Возможно, это значение стоит подстроить под себя: в вашей организации может быть много аббревиатур из трёх символов и меньше.

Если условия выполняются, `event.preventDefault();` гарантирует, что стандартное действие браузера для клавиши `Enter` будет проигнорировано, иначе поиск не сработает. Наконец, строится полный URL поискового запроса. По умолчанию это `http://my-site.example/search/query:yourquery`. Дальше `user/plugins/simplesearch/simplesearch.php` выполняет сам поиск, а другие файлы Twig в плагине выводят результаты.

Но вернёмся к нашему решению! Если мы хотим добавить кнопку поиска, нужно:

1. Добавить кнопку.
2. Убедиться, что метод `.on()` применяется к кнопке, но на этот раз с событием `click` вместо `keypress`.

Это делается таким кодом с использованием [CSS-фреймворка Turret](http://bigfishtv.github.io/turret/docs/index.html). Фрагменты для других фреймворков приведены в конце.

```html linenums="1"
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

HTML и атрибуты классов специфичны для Turret, но в итоге получится [примерно так](http://bigfishtv.github.io/turret/docs/index.html#input-group). Видно также, что метод `.on()` назначен и кнопке поиска, но он проверяет только, что в поле поиска введено больше трёх символов, прежде чем выполнить код внутри `if`.

Вот стандартный HTML для текстового поля с кнопкой поиска для нескольких других фреймворков:

[**Bootstrap**](http://getbootstrap.com/)

```html linenums="1"
<div class="input-group">
    <input type="text" class="form-control" placeholder="Search for...">
    <span class="input-group-btn">
        <button class="btn btn-default" type="button">Go!</button>
    </span>
</div>
```

[**Materialize**](http://materializecss.com/)

```html linenums="1"
<div class="input-field">
    <input id="search" type="search" required>
    <label for="search"><i class="material-icons">search</i></label>
</div>
```

[**Pure CSS**](http://purecss.io)

```html linenums="1"
<form class="pure-form">
    <input type="text" class="pure-input-rounded">
    <button type="submit" class="pure-button">Search</button>
</form>
```

[**Semantic UI**](http://semantic-ui.com/)

```html linenums="1"
<div class="ui action input">
  <input type="text" placeholder="Search...">
  <button class="ui button">Search</button>
</div>
```

## Перебор страниц и медиафайлов

<h4 id="goal-3">Цель:</h4>

Вы хотите получать доступ ко всем страницам и связанным с каждой страницей медиафайлам через PHP и/или Twig, чтобы плагин мог перебирать их или как-то иначе обрабатывать.

<h4 id="solution-3">Решение:</h4>

Используйте возможности коллекций Grav, чтобы построить рекурсивный индекс всех страниц, попутно собирая медиафайлы каждой страницы. Плагин [DirectoryListing](https://github.com/OleVik/grav-plugin-directorylisting/blob/v2.0.0-rc.2/Utilities.php#L64-L105) делает именно это и строит HTML-список по полученной древовидной структуре. Для этого мы создадим рекурсивную функцию — или метод, если речь о классе плагина, — которая проходит по каждой странице и сохраняет её в массив. Метод рекурсивный, потому что он снова вызывает себя для каждой найденной страницы, у которой есть дочерние.

Сначала о параметрах: метод принимает три аргумента. Первый — `$route` страницы, он говорит Grav, где её искать; второй — `$mode`, который сообщает методу, перебирать ли саму страницу или её дочерние страницы; третий — `$depth`, который отслеживает уровень страницы. Сначала метод создаёт объект Page, затем обрабатывает глубину и режим и строит коллекцию. По умолчанию мы сортируем страницы по дате по убыванию, но это можно сделать настраиваемым. Затем мы создаём массив `$paths` для хранения страниц. Поскольку маршруты в Grav уникальны, они используются как ключи этого массива для идентификации каждой страницы.

Теперь перебираем страницы, добавляя глубину, заголовок и маршрут (он также сохраняется как значение для удобства доступа). Внутри цикла foreach мы также пытаемся получить дочерние страницы и добавляем их, если они найдены. Кроме того, мы находим все медиафайлы, связанные со страницей, и добавляем их. Поскольку метод рекурсивный, он продолжит искать страницы и дочерние страницы, пока они не закончатся.

Возвращаемые данные — это древовидная структура, или многомерный массив в терминах PHP, содержащий все страницы и их медиафайлы. Его можно передать в Twig или использовать внутри самого плагина. Учтите, что при очень большой структуре папок PHP может завершиться по тайм-ауту или упасть из-за ограничений рекурсии, например при глубине 100 уровней и больше.

```php linenums="1"
/**
 * Рекурсивно создаёт структуру страниц
 * @param string $route Маршрут к странице
 * @param integer $depth Зарезервированное место для глубины рекурсии
 * @return array Структура страниц с дочерними страницами и медиафайлами
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

<h4 id="goal-4">Цель:</h4>

Вместо наследования темы можно создать очень простой плагин, который позволяет использовать собственное расположение для пользовательских шаблонов Twig.

<h4 id="solution-4">Решение:</h4>

Всё, что нужно в этом плагине, — событие, которое задаёт расположение ваших шаблонов. Проще всего создать плагин с помощью плагина `devtools`. Установите его:

```bash
$ bin/gpm install devtools
```

После установки создайте новый плагин командой:

```bash
$ bin/plugin devtools newplugin
```

Заполните данные: имя, автора и т. д. Допустим, мы назовём его `Custom Templates`, и плагин будет создан в `/user/plugins/custom-templates`. Теперь достаточно отредактировать файл `custom-templates.php` и вставить такой код:

```php linenums="1"
<?php
namespace Grav\Plugin;

use \Grav\Common\Plugin;

class CustomTemplatesPlugin extends Plugin
{
    /**
     * Подписка на нужные события
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
     * Добавляет текущую папку в пути поиска twig.
     */
    public function onTwigTemplatePaths()
    {
        $this->grav['twig']->twig_paths[] = __DIR__ . '/templates';
    }
}
```

Этот плагин просто подписывается на событие `onTwigTemplatePaths()` и в его методе добавляет папку `user/plugins/custom-templates/templates` к путям, которые проверяет Twig.

Это позволяет положить шаблон Twig с именем `foo.html.twig`, после чего любая страница `foo.md` сможет использовать этот шаблон.

> [!WARNING] Предупреждение
> Примечание: путь к пользовательским шаблонам плагина добавляется в **конец** массива путей шаблонов Twig. Это значит, что тема (она всегда идёт первой) имеет приоритет над шаблонами плагина с тем же именем. Чтобы это исправить, поставьте путь шаблонов плагина в начало массива, изменив метод события:

```twig linenums="1"
/**
 * Добавляет текущую папку в пути поиска twig.
 */
public function onTwigTemplatePaths()
{
    array_unshift($this->grav['twig']->twig_paths, __DIR__ . '/templates');
}
```

## Использование кэша в своих плагинах

<h4 id="goal-5">Цель:</h4>

При разработке своих плагинов часто полезно кэшировать данные средствами кэша Grav, чтобы повысить производительность. К счастью, использовать кэш в своём коде очень просто.

<h4 id="solution-5">Решение:</h4>

Вот базовый код, показывающий, как работает кэширование:

```php linenums="1"
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

Сначала мы получаем объект кэша Grav и проверяем, нет ли уже наших данных в кэше (`$data = $cache->fetch($id)`). Если `$data` существует, просто возвращаем его без лишней работы.

Однако если выборка из кэша вернула null, значит, данных в кэше нет: выполняем нужную _работу_ и получаем данные (`$data = $this->gatherData()`), а затем просто сохраняем их на будущее (`$cache->save($hash, $data)`).

## Учимся на примерах

Плагинов сейчас так много, что ответ наверняка найдётся в исходном коде какого-нибудь из них. Проблема в том, чтобы понять, какие именно смотреть. Эта страница пытается перечислить типовые задачи плагинов и указать конкретные плагины, в которых показано, как их решать.

Прежде чем продолжать, убедитесь, что вы знакомы с [основной документацией](https://learn.getgrav.org/plugins), особенно с [жизненным циклом Grav](https://learn.getgrav.org/plugins/grav-lifecycle)!

### Как читать данные из файловой системы и записывать их? {: #how-do-i-read-from-and-write-data-to-the-file-syst }

Grav работает с файлами, но файлы &#8800; статика! Есть множество способов читать данные из файловой системы и записывать их.

  * Если нужен только доступ на чтение к данным YAML, посмотрите [плагин Import](https://github.com/Deester4x4jr/grav-plugin-import).

  * Предпочтительный интерфейс — встроенный [RocketTheme\Toolbox\File](https://learn.getgrav.org/api#class-RocketThemeToolboxFile).

  * Ничто не мешает использовать и [SQLite](https://sqlite.org/).

  * Самый простой пример — пожалуй, плагин [Comments](https://github.com/getgrav/grav-plugin-comments).

  * Другие примеры:

    * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

    * [Thumb Ratings](https://github.com/iusvar/grav-plugin-thumb-ratings)

    * [Webmention](https://github.com/Perlkonig/grav-plugin-webmention)

### Как сделать данные из плагина доступными в Twig?

Один из способов — пространство имён `config.plugins.X`. Просто вызовите `$this->config->set()`, как в примерах ниже:

  * [ipLocate](https://github.com/Perlkonig/grav-plugin-iplocate/blob/master/iplocate.php#L82)
  * [Count Views](https://github.com/Perlkonig/grav-plugin-count-views/blob/master/count-views.php#L88)

Затем к этим данным можно обратиться в шаблоне Twig через `{{ config.plugins.X.whatever.variable }}`.

Как вариант, переменные можно передавать через `grav['twig']`:

  * [Blogroll](https://github.com/Perlkonig/grav-plugin-blogroll/blob/master/blogroll.php#L43), после чего к ним можно обратиться напрямую [в своём шаблоне](https://github.com/Perlkonig/grav-plugin-blogroll/blob/master/templates/partials/blogroll.html.twig#L32).

Наконец, данные можно внедрить прямо в заголовки страницы, как это сделано в [плагине Import](https://github.com/Deester4x4jr/grav-plugin-import).

### Как внедрить Markdown в страницу?

Согласно [жизненному циклу Grav](https://learn.getgrav.org/plugins/grav-lifecycle), самый поздний хук события, где можно внедрить сырой Markdown, — `onPageContentRaw`. Самый ранний — вероятно, `onPageInitialized`. Вы можете просто взять `$this->grav['page']->rawMarkdown()`, обработать его и записать обратно через `$this->grav['page']->setRawContent()`. Это демонстрируют следующие плагины:

  * [Page Inject](https://github.com/getgrav/grav-plugin-page-inject)

  * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

### Как внедрить HTML в итоговый вывод?

Самый поздний момент, когда можно внедрить HTML и при этом сохранить вывод в кэше, — событие `onOutputGenerated`. Можно просто взять и изменить `$this->grav->output`.

  * Многие типовые задачи можно решить средствами [Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core).

  * Плагины [Pubmed](https://github.com/Perlkonig/grav-plugin-pubmed) и [Tablesorter](https://github.com/Perlkonig/grav-plugin-tablesorter) действуют более грубо.

### Как подключить ресурсы вроде JavaScript и CSS? {: #how-do-i-inject-assets-like-javascript-and-css-fil }

Это делается через интерфейс [Grav\Common\Assets](https://learn.getgrav.org/api#class-gravcommonassets).

  * [Google Analytics](https://github.com/escopecz/grav-ganalytics)

  * [Bootstrapper](https://github.com/getgrav/grav-plugin-bootstrapper)

  * [Gravstrap](https://github.com/giansi/gravstrap)

  * [Tablesorter](https://github.com/Perlkonig/grav-plugin-tablesorter)

### Как влиять на заголовки и коды ответа? {: #how-do-i-affect-the-response-headers-and-response- }

Заголовки ответа можно задать командой PHP `header()`. Самое позднее, когда это можно сделать, — событие `onOutputGenerated`, после которого вывод фактически отправляется клиенту. Сам код ответа можно задать только в YAML-заголовках соответствующей страницы (`http_response_code`).

  * Плагин [Graveyard](https://github.com/Perlkonig/grav-plugin-graveyard) заменяет ответы `404 NOT FOUND` на `410 GONE` через YAML-заголовки.

  * Плагин [Webmention](https://github.com/Perlkonig/grav-plugin-webmention) устанавливает заголовок `Location` в ответе `201 CREATED`.

### Как включить сторонние библиотеки в свой плагин? {: #how-do-i-incorporate-third-party-libraries-into-my }

Обычно другие готовые библиотеки помещают в подпапку `vendor`, а их `autoload.php` подключают через `require` в нужном месте плагина. (Если вы используете Git, подумайте о [поддеревьях](https://help.github.com/articles/about-git-subtree-merges/).)

  * [Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core)

  * [Table Importer](https://github.com/Perlkonig/grav-plugin-table-importer)

### Как расширить Twig?

Проще всего повторить пример [«Пользовательский фильтр/функция Twig»](../twig-recipes/index.md#custom-twig-filter-function) из раздела **Рецепты Twig**.

Кроме того, [прочитайте документацию Twig](https://twig.symfony.com/) и разработайте своё расширение. Затем посмотрите плагин [TwigPCRE](https://github.com/kesslernetworks/grav-plugin-twigpcre), чтобы узнать, как встроить его в Grav.

### Как работать с внешними API?

Grav предоставляет объект [Grav\Common\GPM\Response](https://learn.getgrav.org/api#class-grav-common-gpm-response), но ничто не мешает обращаться к API напрямую, если вам так удобнее.

  * [ipLocate](https://github.com/Perlkonig/grav-plugin-iplocate)

  * [Pubmed](https://github.com/Perlkonig/grav-plugin-pubmed)
