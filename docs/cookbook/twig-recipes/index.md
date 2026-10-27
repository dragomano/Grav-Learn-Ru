---
title: "Рецепты Twig"
description: "Сборник задач и их решений, связанных с шаблонизатором Twig."
tags:
  - "Рецепты"
  - "Разработка"
  - "Twig"
---

# Рецепты Twig

На этой странице собраны задачи и их решения, связанные с шаблонизатором Twig.

## Вывод 5 последних записей блога

<h5 id="problem">Проблема:</h5>

Вы хотите показывать в боковой панели сайта последние 5 записей блога, чтобы читатель видел свежую активность.

<h5 id="solution">Решение:</h5>

Найдите страницу `/blog`, получите её дочерние страницы, отсортируйте их по дате по убыванию, а затем возьмите первые 5 для вывода в списке:

```twig linenums="1"
<ul>
{% for post in page.find('/blog').children.published.order('date', 'desc').slice(0, 5) %}
    <li class="recent-posts">
        <strong><a href="{{ post.url }}">{{ post.title }}</a></strong>
    </li>
{% endfor %}
</ul>
```

при использовании внутри страниц не забудьте добавить в заголовки страницы такую конфигурацию:

```yaml linenums="1"
twig_first: true
process:
    twig: true
```

## Добавление ссылок навигации для немодульных страниц

<h5 id="problem-1">Проблема:</h5>

Вы хотите выводить ссылки навигации для немодульных страниц.

<h5 id="solution-1">Решение:</h5>

```twig linenums="1"
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
```

## Список записей блога за год

<h5 id="problem-2">Проблема:</h5>

Вы хотите показать все записи блога, опубликованные в текущем календарном году.

<h5 id="solution-2">Решение:</h5>

Найдите страницу `/blog`, получите её дочерние страницы, отфильтруйте подходящим `dateRange()` и отсортируйте их по дате по убыванию:

```twig linenums="1"
<ul>
{% set this_year = "now"|date('Y') %}
{% for post in page.find('/blog').children.dateRange('01/01/' ~ this_year, '12/31/' ~ this_year).order('date', 'desc') %}
    <li class="recent-posts">
        <strong><a href="{{ post.url }}">{{ post.title }}</a></strong>
    </li>
{% endfor %}
</ul>
```

## Вывод названия месяца на нужном языке

<h5 id="problem-3">Проблема:</h5>

В некоторых шаблонах страниц используется фильтр Twig `date`, а он не учитывает локали и многоязычность. Поэтому даже если страница на языке, отличном от английского, месяц может выводиться по-английски, если шаблон показывает его название.

<h5 id="solution-3">Решение:</h5>

У этой проблемы есть два решения.

###### Первый способ

Первый — использовать расширение Twig intl.

Установите https://github.com/Perlkonig/grav-plugin-twig-extensions. Убедитесь, что у вас установлено расширение PHP intl.

В шаблоне Twig вместо, например, `{{ page.date|date("M") }}` (как в теме Antimatter) используйте `{{ page.date|localizeddate('long', 'none', 'it', 'Europe/Rome', 'MMM') }}` (здесь укажите свой язык и часовой пояс).

###### Второй способ

Предположим, у вас в папке `user/languages/` есть файлы переводов, и в `en.yaml` есть запись:

```yaml
MONTHS_OF_THE_YEAR: [January, February, March, April, May, June, July, August, September, October, November, December]
```

А в `fr.yaml`:

```yaml
MONTHS_OF_THE_YEAR: [Janvier, Février, Mars, Avril, Mai, Juin, Juillet, Août, Septembre, Octobre, Novembre, Décembre]
```

Тогда в Twig:

```html linenums="1"
<li>
    <a href='{{ post.url }}'><aside class="dates">{{ 'GRAV.MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1) }} {{ post.date|date('d') }}</aside></a>
    <a href='{{ post.url }}'>{{ post.title }}</a>
</li>
```

Здесь используется пользовательский фильтр Twig `|ta` из Grav, что означает **Translate Array**. В английской версии вывод может выглядеть примерно так:

```text
An Example Post  July 2015
```

А во французской:

```text
Un exemple d’article Juillet 2015
```

## Вывод содержимого страницы без аннотации

<h5 id="problem-4">Проблема:</h5>

Вы хотите вывести содержимое страницы без аннотации в начале.

<h5 id="solution-4">Решение:</h5>

Используйте фильтр `slice`, чтобы убрать аннотацию из содержимого страницы:

```twig linenums="1"
{% set content = page.content|slice(page.summary|length) %}
{{ content|raw }}
```

## Как спрятать email от спам-ботов

<h5 id="problem-5">Проблема:</h5>

Вы хотите спрятать email от спам-ботов.

<h5 id="solution-5">Решение:</h5>

Включите обработку Twig в заголовках страницы:

```yaml linenums="1"
process:
    twig: true
```

Затем используйте фильтр Twig `safe_email`:

```html linenums="1"
<a href="mailto:{{'your.email@server.com'|safe_email}}">
  Email me
</a>
```

## Выбор случайного элемента из переведённого массива

<h5 id="problem-6">Проблема:</h5>

Вы хотите выбрать случайный элемент из массива, переведённого на определённый язык. Предполагается, что у вас [настроен и сконфигурирован многоязычный сайт](../../content/multi-language/index.md), как описано в документации.

<h5 id="solution-6">Решение:</h5>

Также предположим, что у вас в папке `user/languages/` есть файлы переводов, и в `en.yaml` есть запись:

```text
FRUITS: [Banana, Cherry, Lemon, Lime, Strawberry, Raspberry]
```

А в `fr.yaml`:

```text
FRUITS: [Banane, Cerise, Citron, Citron Vert, Fraise, Framboise]
```

Тогда в Twig:

```twig linenums="1"
{% set langobj  = grav['language'] %}
{% set curlang  = langobj.getLanguage() %}
{% set fruits   = langobj.getTranslation(curlang,'FRUITS',true) %}
<span data-ticker="{{ fruits|join(',') }}">{{ random(fruits) }}</span>
```

## Вывод изображения, загруженного через поле file

<h5 id="problem-7">Проблема:</h5>

Вы добавили поле `file` в свою схему и хотите вывести изображение, добавленное в это поле.

<h5 id="solution-7">Решение:</h5>

Поскольку поле `file` позволяет загружать несколько изображений, оно создаёт в метаданных страницы два вложенных объекта: первый — список загруженных изображений, а вложенный в него объект — набор свойств и значений для конкретного изображения.

_Учтите: если нужно, чтобы пользователь выбрал только одно изображение, проще использовать поле `filepicker`, которое хранит один объект со свойствами выбранного изображения._

Если изображение одно, вывести его в шаблоне можно так:

```twig
{{ page.media[header.yourfilefield|first.name] }}
```

Если вы разрешили загружать несколько изображений, Twig может выглядеть так:

```twig linenums="1"
{% for imagesuploaded in page.header.yourfilefield %}
{{ page.media[imagesuploaded.name] }}
{% endfor %}
```

## Вывод изображения, выбранного в поле mediapicker

<h5 id="problem-8">Проблема:</h5>

Вы добавили поле `mediapicker` в свою схему и хотите вывести выбранное изображение.

<h5 id="solution-8">Решение:</h5>

Поле `mediapicker` можно добавить в схему так:

```yaml linenums="1"
header.myimage:
  type: mediapicker
  folder: 'self@'
  label: Select a file
  preview_images: true
```

Поле `mediapicker` хранит путь к изображению строкой, например `/home/background.jpg`.
Чтобы обратиться к этому изображению через возможности работы с медиафайлами страницы, нужно разбить строку и получить:
 - путь к странице, где хранится изображение;
 - имя изображения.

Сделать это в Twig можно так:

```twig linenums="1"
{% set image_parts = pathinfo(header.myimage) %}
{% set image_basename = image_parts.basename %}
{% set image_page = image_parts.dirname == '.' ? page : page.find(image_parts.dirname) %}

{{ image_page.media[image_basename].html()|raw }}
```

## Пользовательский фильтр/функция Twig {: #custom-twig-filter-function }

<h5 id="problem-9">Проблема:</h5>

Иногда в Twig нужна логика, которую можно реализовать только на PHP, и тогда лучшее решение — создать пользовательский фильтр или функцию Twig. Фильтр обычно применяется к строке в формате `"some string"|custom_filter`, а функция может принимать строку или переменную любого другого типа: `custom_function("some string")`. По сути они очень похожи.

Можно также передавать дополнительные параметры: `"some string"|custom_filter('foo', 'bar')`, где эти параметры используются внутри фильтра. Вариант с функцией будет таким: `custom_function("some string", 'foo', 'bar')`.

В этом примере мы создадим простой фильтр Twig, который принимает строку и разбивает её на части через разделитель. Это особенно удобно для номеров банковских карт, лицензионных ключей и т. п.

<h5 id="solution-9">Решение:</h5>

Лучший способ добавить такую функциональность — реализовать логику в своём плагине, хотя можно добавить её и в PHP-файл темы. Для простоты в этом примере мы используем плагин. Сначала установите плагин devtools, чтобы создавать плагины в простом мастер-режиме:

```bash
bin/gpm install devtools
```

Затем создайте свой плагин и заполните данные, которые мастер запросит.

```bash
bin/plugin devtools new-plugin

Enter Plugin Name: ACME Twig Filters
Enter Plugin Description: Plugin for custom Twig filters
Enter Developer Name: ACME, Inc.
Enter GitHub ID (can be blank):
Enter Developer Email: hello@acme.com

SUCCESS plugin ACME Twig Filters -> Created Successfully

Path: /Users/joe/grav/user/plugins/acme-twig-filters
```

По умолчанию этот каркас нового плагина добавляет на страницу немного тестового кода через событие `onPageContentRaw()`. Сначала замените эту функциональность кодом, который слушает событие `onTwigInitialized()`:

```php linenums="1"
public function onPluginsInitialized()
{
    // Не продолжаем, если мы в плагине админки
    if ($this->isAdmin()) {
        return;
    }

    // Включаем основное событие, которое нам нужно
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
```

Сначала зарегистрируем фильтр в методе `onTwigInitialized()`:

```php linenums="1"
/**
 * @param Event $e
 */
public function onTwigInitialized(Event $e)
{
    $this->grav['twig']->twig()->addFilter(
        new \Twig_SimpleFilter('chunker', [$this, 'chunkString'])
    );
}
```

Первый параметр метода регистрирует `chunker` как имя фильтра, а `chunkString` — как PHP-метод, где выполняется логика. Значит, дальше нужно создать этот метод:

```php linenums="1"
/**
 * Разбивает строку на части
 */
public function chunkString($string, $chunksize = 4, $delimiter = '-')
{
    return (trim(chunk_split($string, $chunksize, $delimiter), $delimiter));
}
```

Теперь можно попробовать в шаблонах Twig:

```twig
{{ "ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5"|chunker }}
```

В результате получится:

```text
ER27-XV3O-CCDP-RJK5-IVSD-ME6D-6OT6-QHK5
```

или можно передать дополнительные параметры:

```twig
{{ "ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5"|chunker(8, '|') }}
```

и тогда получится:

```text
ER27XV3O|CCDPRJK5|IVSDME6D|6OT6QHK5
```

Наконец, если вы хотите, чтобы это было доступно не только как фильтр, но и как функция, просто зарегистрируйте функцию Twig с тем же именем в методе `onTwigInitialized()`:

```php linenums="1"
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
```

И тогда можно использовать синтаксис функции:

```twig linenums="1"
{{ chunker("ER27XV3OCCDPRJK5IVSDME6D6OT6QHK5", 8, '|') }}
```

## Расширение базового шаблона унаследованной темы

<h5 id="problem-10">Проблема:</h5>

Иногда нужно расширить сам базовый шаблон. Это бывает, когда нет простого и очевидного способа расширить уже имеющиеся в шаблоне блоки. Возьмём Quark как пример родительской темы, и вы хотите расширить `themes/quark/templates/partials/base.html.twig` в своей теме `myTheme`.

<h5 id="solution-10">Решение:</h5>

Вы можете добавить Quark как пространство имён Twig: в файле темы `my-theme.php` слушайте событие `onTwigLoader` и добавьте папку шаблонов темы Quark. Содержимое класса должно быть примерно таким:

```php linenums="1"
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

        // добавляем тему quark как пространство имён для twig
        $quark_path = Grav::instance()['locator']->findResource('themes://quark');
        $this->grav['twig']->addPath($quark_path . DIRECTORY_SEPARATOR . 'templates', 'quark');
    }
}
```

Теперь в `themes/my-theme/templates/partials/base.html.twig` можно расширить базовый шаблон Quark так:

```twig linenums="1"
{% extends '@quark/partials/base.html.twig' %}

{% block header %}
This is a new extended header.
{% endblock %}
```
