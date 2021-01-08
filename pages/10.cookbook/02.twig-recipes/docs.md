---
title: Рецепты Twig
page-toc:
  active: true
  depth: 1
taxonomy:
    category: docs
---

Эта страница содержит набор проблем и их соответствующих решений, связанных с шаблонизацией Twig.

## Перечисление последних 5 сообщений в блоге

##### Проблема:

Вы хотите отобразить последние 5 сообщений в блоге на боковой панели вашего сайта, чтобы читатель мог видеть последние действия в блоге.

##### Решение:

Просто найдите страницу `/blog`, получите её дочерние элементы, упорядочите их по дате в порядке убывания, а затем получите первые 5 для отображения в списке:

[prism classes="language-twig line-numbers"]
<ul>
{% for post in page.find('/blog').children.order('date', 'desc').slice(0, 5) %}
    <li class="recent-posts">
        <strong><a href="{{ post.url }}">{{ post.title }}</a></strong>
    </li>
{% endfor %}
</ul>
[/prism]

при использовании в пределах страницы, убедитесь, что вы добавили следующую конфигурацию заголовка страницы:

[prism classes="language-yaml line-numbers"]
twig_first: true
process:
    twig: true
[/prism]


## Добавление немодульных навигационных ссылок

##### Проблема:

Вы хотите отображать навигационные ссылки с немодульных страниц.

##### Решение

[prism classes="language-twig line-numbers"]
<div class="desktop-nav__navigation">
    {% for page in pages.children %}
        {% if page.visible %}
            {% set current_page = (page.active or page.activeChild) ? 'active' : '' %}
            <a class="desktop-nav__nav-link {{ current_page }}" href="{{ page.url }}">
                {{ page.menu }}
            </a>
        {% endif %}
    {% endfor %}
</div>
[/prism]

## Перечисление постов в блоге за год

##### Проблема:

Вы хотите отобразить все сообщения в блоге, которые произошли в этом календарном году.

##### Решение:

Просто найдите страницу `/blog`, получите ее дочерние элементы, отфильтруйте по соответствующему `dateRange()` и упорядочите их по дате в порядке убывания:

[prism classes="language-twig line-numbers"]
<ul>
{% set this_year = "now"|date('Y') %}
{% for post in page.find('/blog').children.dateRange('01/01/' ~ this_year, '12/31/' ~ this_year).order('date', 'desc') %}
    <li class="recent-posts">
        <strong><a href="{{ post.url }}">{{ post.title }}</a></strong>
    </li>
{% endfor %}
</ul>
[/prism]

## Отображение переведенного месяца

##### Проблема:

В некоторых шаблонах страниц используется фильтр Twig `date`, который не обрабатывает языковые стандарты/мультиязычность. Таким образом, даже если ваша страница на языке, отличном от английского, она может отображать месяц на английском языке, если шаблон выберет отображение названия месяца.

##### Решение:

Есть два решения этой проблемы.

###### Первый подход

Первый предполагает использование расширения Twig intl.

Установите https://github.com/Perlkonig/grav-plugin-twig-extensions. Убедитесь, что у вас установлено расширение PHP intl.

В вашем шаблоне замените, например (как в теме Antimatter) `{{ page.date | date (" M ") }}` на `{{ page.date | localizeddate ('long', 'none', 'it', 'Europe / Rome', 'MMM')}}` (укажите здесь свой язык и часовой пояс)

###### Второй подход

Предположим, у вас есть настройки языковых переводов в папке `user/languages​​/` под названием `en.yaml`, которая содержит запись:
[prism classes="language-yaml"]
MONTHS_OF_THE_YEAR: [January, February, March, April, May, June, July, August, September, October, November, December]
[/prism]

И `ru.yaml`:
[prism classes="language-yaml"]
MONTHS_OF_THE_YEAR: [Январь, Февраль, Март, Апрель, Май, Июнь, Июль, Август, Сентябрь, Октябрь, Ноябрь, Декабрь]
[/prism]

Тогда можно использовать Twig:

[prism classes="language-html line-numbers"]
<li>
    <a href='{{ post.url }}'><aside class="dates">{{ 'GRAV.MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1) }} {{ post.date|date('d') }}</aside></a>
    <a href='{{ post.url }}'>{{ post.title }}</a>
</li>
[/prism]

Это использует настраиваемый фильтр Grav Twig `|ta`, который означает **Translate Array**. В английской версии результат может быть примерно таким:

[prism classes="language-text"]
An Example Post  July 2015
[/prism]

А на русском:

[prism classes="language-text"]
Статья-пример  Июль 2015
[/prism]

## Отображение содержимого страницы без тизера

##### Проблема:

Вы хотите отображать содержимое страницы без тизера вверху.

##### Решение:

Используйте фильтр `slice`, чтобы удалить тизер из содержимого страницы:

[prism classes="language-twig line-numbers"]
{% set content = page.content|slice(page.summary|length) %}
{{ content }}
[/prism]


## Скрытие имейла от спам-ботов

##### Проблема:

Вы хотите скрыть имейл от спам-ботов

##### Решение:

Включите обработку Twig в шапке страницы:

[prism classes="language-yaml line-numbers"]
process:
    twig: true
[/prism]

Затем используйте фильтр Twig `safe_email`:

[prism classes="language-html line-numbers"]
<a href="mailto:{{'your.email@server.com'|safe_email}}">
  Email me
</a>
[/prism]

## Выбор случайного элемента из переведенного массива

##### Проблема:

Вы хотите выбрать случайный элемент из массива, переведенного на определенный язык. Для этого предполагается, что у вас есть [многоязычный сайт](/content/multi-language), настроенный, как указано в документации.

##### Решение:

Предположим также, что у вас есть настройки языковых переводов в папке `user/languages​​/` под названием `en.yaml`, которая содержит запись:

[prism classes="language-text"]
FRUITS: [Banana, Cherry, Lemon, Lime, Strawberry, Raspberry]
[/prism]

И `ru.yaml`:

[prism classes="language-text"]
FRUITS: [Банан, Вишня, Лимон, Лайм, Клубника, Малина]
[/prism]

Тогда можно использовать Twig:

[prism classes="language-twig line-numbers"]
{% set langobj  = grav['language'] %}
{% set curlang  = langobj.getLanguage() %}
{% set fruits   = langobj.getTranslation(curlang,'FRUITS',true) %}
<span data-ticker="{{ fruits|join(',') }}">{{ random(fruits) }}</span>
[/prism]

## Отображение изображения, загруженного в поле файла

##### Проблема:

Вы добавили поле типа `file` в свой собственный чертеж и хотите отобразить изображение, добавленное в это поле.

##### Решение:

Поскольку поле `file` позволяет загружать несколько изображений, оно генерирует два вложенных объекта в вашем frontmatter, первый объект - это список загруженных изображений, вложенный объект внутри - это группа свойств/значений для данного изображения.

_Обратите внимание, что в случае, когда вы хотите, чтобы ваш пользователь выбирал только одно изображение, было бы проще использовать поле `filepicker`, которое хранит один объект со свойствами выбранных изображений._

Если у вас есть одно изображение, вы можете отобразить его в своем шаблоне, используя:

[prism classes="language-twig"]
{{ page.media[header.yourfilefield|first.name] }}
[/prism]

Если вы разрешили пользователю загружать несколько изображений, ваша разметка на Twig могла бы выглядеть так:

[prism classes="language-twig line-numbers"]
{% for imagesuploaded in page.header.yourfilefield %}
{{ page.media[imagesuploaded.name] }}
{% endfor %}
[/prism]

## Отображение изображения, выбранного в поле `mediapicker`

##### Проблема:

Вы добавили поле `mediapicker` в свой собственный чертеж и хотите отобразить выбранное изображение.

##### Решение:

Поле `mediapicker` может быть добавлено к вашему чертежу, как показано ниже:

[prism classes="language-yaml line-numbers"]
header.myimage:
  type: mediapicker
  folder: 'self@'
  label: Select a file
  preview_images: true
[/prism]

Поле `mediapicker` хранит путь к изображению в виде строки, такой как `/home/background.jpg`.
Чтобы получить доступ к этому изображению с помощью функции мультимедиа страницы, вы должны разделить эту строку и получить:
 - путь к странице, где хранится это изображение
 - название изображения.

Вы можете сделать это через Twig, используя приведенный ниже фрагмент:

[prism classes="language-twig line-numbers"]
{% set image_parts = pathinfo(header.myimage) %}
{% set image_basename = image_parts.basename %}
{% set image_page = image_parts.dirname == '.' ? page : page.find(image_parts.dirname) %}

{{ image_page.media[image_basename].html() }}
[/prism]

## Пользовательский фильтр/функция Twig

##### Проблема:

Иногда вам нужна некоторая логика в Twig, которая может быть реализована только в PHP, поэтому лучшим решением является создание пользовательского фильтра или функции Twig. Фильтр обычно добавляется к строке в формате: `"некоторая строка"|custom_filter`, и функция может принимать строку или любой другой тип переменной: `custom_function("some string")`, но по сути они очень похожи.

Вы также можете передать дополнительные параметры, например: `"some string"|custom_filter('foo', 'bar')`, где дополнительные параметры можно использовать внутри фильтра. И вариант функции будет: `custom_function("some string", 'foo', 'bar')`.

В этом примере мы создадим простой фильтр Twig для подсчета дублей строки и разделения её на фрагменты, разделенные разделителем. Это особенно полезно для таких вещей, как номера кредитных карт, лицензионные ключи и т. д.

##### Решение:

Лучший способ добавить эту дополнительную функциональность - добавить логику в свой собственный плагин, хотя добавление её в php-файл вашей темы также является вариантом. В этом примере мы будем использовать плагин для простоты. Сначала вам нужно установить плагин DevTools, чтобы сделать создание плагина простым процессом на основе мастера:

[prism classes="language-bash command-line"]
bin/gpm install devtools
[/prism]

Затем вам нужно создать свой новый настраиваемый плагин, а затем ввести свои данные при появлении запроса.

[prism classes="language-bash command-line" cl-output="2-12"]
bin/plugin devtools new-plugin

Enter Plugin Name: ACME Twig Filters
Enter Plugin Description: Plugin for custom Twig filters
Enter Developer Name: ACME, Inc.
Enter GitHub ID (can be blank):
Enter Developer Email: hello@acme.com

SUCCESS plugin ACME Twig Filters -> Created Successfully

Path: /Users/joe/grav/user/plugins/acme-twig-filters
[/prism]

По умолчанию этот скелетный каркас для нового плагина добавит на вашу страницу какой-то фиктивный тест через событие `onPageContentRaw()`. Сначала вам нужно заменить эту функциональность на код, который прослушивает событие `onTwigInitialized()`:

[prism classes="language-php line-numbers"]
    public function onPluginsInitialized()
    {
        // Don't proceed if we are in the admin plugin
        if ($this->isAdmin()) {
            return;
        }

        // Enable the main event we are interested in
        $this->enable([
            'onTwigInitialized' => ['onTwigInitialized', 0]
        ]);
    }

    /**
     * @param Event $e
     */
    public function onTwigInitialized(Event $e)
    {

    }
[/prism]

Сначала нам нужно зарегистрировать фильтр в методе `onTwigInitialized()`:

[prism classes="language-php line-numbers"]
    /**
     * @param Event $e
     */
    public function onTwigInitialized(Event $e)
    {
        $this->grav['twig']->twig()->addFilter(
            new \Twig_SimpleFilter('chunker', [$this, 'chunkString'])
        );
    }
[/prism]

Первый параметр метода регистрирует `chunker` как имя фильтра, а `chunkString` как метод PHP, в котором возникает логика. Итак, нам нужно создать следующее:

[prism classes="language-php line-numbers"]
    /**
     * Break a string up into chunks
     */
    public function chunkString($string, $chunksize = 4, $delimiter = '-')
    {
        return (trim(chunk_split($string, $chunksize, $delimiter), $delimiter));
    }
[/prism]

Теперь вы можете попробовать это в своих шаблонах Twig следующим образом:

[prism classes="language-twig"]
{{ "ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5"|chunker }}
[/prism]

Что даст:

[prism classes="language-text"]
ER27-XV3O-CCDP-RJK5-IVSD-ME6D-6OT6-QHK5
[/prism]

или вы можете передать дополнительные параметры:

[prism classes="language-twig"]
{{ "ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5"|chunker(8, '|') }}
[/prism]

который даст:

[prism classes="language-text"]
ER27XV3O|CCDPRJK5|IVSDME6D|6OT6QHK5
[/prism]

Наконец, если вы хотите, чтобы это было доступно через функцию, а не только через фильтр, вы можете просто зарегистрировать функцию Twig с тем же именем в методе `onTwigInitialized()`:

[prism classes="language-php line-numbers"]
    /**
     * @param Event $e
     */
    public function onTwigInitialized(Event $e)
    {
        $this->grav['twig']->twig()->addFilter(
            new \Twig_SimpleFilter('chunker', [$this, 'chunkString'])
        );
        $this->grav['twig']->twig()->addFunction(
            new \Twig_SimpleFunction('chunker', [$this, 'chunkString'])
        );
    }
[/prism]

И тогда вы можете использовать синтаксис функции:

[prism classes="language-twig line-numbers"]
{{ chunker("ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5", 8, '|') }}
[/prism]

## Расширение базового шаблона унаследованной темы

##### Проблема

Иногда нужно расширить сам базовый шаблон. Это может произойти, когда нет простого и очевидного способа расширить блоки, уже присутствующие в шаблоне. Давайте использовать Quark в качестве примера для родительской темы, и вы хотите расширить файл `themes/quark/templates/partials/base.html.twig` вашей темой `myTheme`.

##### Решение

Вы можете добавить Quark в качестве пространства имен Twig, используя тему `my-theme.php` для прослушивания события `onTwigLoader` и добавления каталога шаблона Quark. Содержание класса должно быть примерно таким:

[prism classes="language-php line-numbers"]
    <?php
    namespace Grav\Theme;

    use Grav\Common\Grav;
    use Grav\Common\Theme;

    class MyTheme extends Quark {
        public static function getSubscribedEvents() {
            return [
                'onTwigLoader' => ['onTwigLoader', 10]
            ];
        }

        public function onTwigLoader() {
            parent::onTwigLoader();

            // add quark theme as namespace to twig
            $quark_path = Grav::instance()['locator']->findResource('themes://quark');
            $this->grav['twig']->addPath($quark_path . DIRECTORY_SEPARATOR . 'templates', 'quark');
        }
    }
[/prism]


Теперь в файле `themes/my-theme/templates/partials/base.html.twig` можно расширить базовый шаблон Quark следующим образом:

[prism classes="language-twig line-numbers"]
    {% extends '@quark/partials/base.html.twig' %}

    {% block header %}
    This is a new extended header.
    {% endblock %}
[/prism]
