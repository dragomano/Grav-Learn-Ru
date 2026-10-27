---
title: "Создание блога"
description: "Начнём с простого: выберите тему, в которой уже есть шаблон страницы блога. Например, Antimatter, TwentyFifteen, Deliver, Lingonberry, Afterburner2 и многие другие."
tags:
  - "Разработка"
  - "Блогинг"
  - "Twig"
---

# Создание блога

> [!CAUTION] Внимание
> Скачайте и установите локально скелет сайта Blog Site с [https://getgrav.org/downloads/skeletons](https://getgrav.org/downloads/skeletons) или хотя бы держите под рукой для сверки репозиторий [https://github.com/getgrav/grav-skeleton-blog-site](https://github.com/getgrav/grav-skeleton-blog-site). Это пример сайта на теме Antimatter. Готовый работающий сайт Grav со структурой блога наверняка выручит вас, если вы застрянете или не поймёте, что делать дальше.

## Убедитесь, что тема предоставляет шаблоны страниц Blog и Item {: #check-your-theme-provides-the-blog-and-item-page-t }

Начнём с простого: выберите тему, в которой уже есть шаблон страницы блога. Например, Antimatter, TwentyFifteen, Deliver, Lingonberry, Afterburner2 и многие другие.
Как проверить, есть ли в теме шаблон страницы блога? Зайдите в папку `/user/themes/[yourtheme]/templates` и проверьте наличие файлов `blog.html.twig` и `item.html.twig`.

Если вы уже выбрали тему, а этих файлов в ней нет, скопируйте их из Antimatter: [https://github.com/getgrav/grav-theme-antimatter/tree/develop/templates](https://github.com/getgrav/grav-theme-antimatter/tree/develop/templates) (если берёте файлы из этой темы, понадобятся ещё два фрагмента шаблона — `blog_item.html.twig` и `sidebar.html.twig`. Их нужно положить в папку `templates/partials`.)

Возможно, разметку придётся подправить под свою тему. Если вы только начинаете, лучший вариант — взять тему, где всё это уже есть.

## Создание структуры страниц блога
Структурировать страницы можно по-разному. По умолчанию и проще всего — сделать родительскую страницу типа Blog и дочерние страницы для записей блога.

### С помощью плагина Admin
Создайте страницу типа Blog. Это будет «домашняя страница» блога со списком записей.

Создайте одну или несколько дочерних страниц типа `Item`. Это и будут записи блога.

### Вручную
Зайдите в папку `pages/`, создайте страницу `01.blog` (число меняйте под свою структуру меню) и добавьте в неё файл `blog.md`.
В этот файл добавьте такое содержимое:

```yaml linenums="1"
---
content:
    items: '@self.children'
---
```

Это говорит Grav, что нужно перебрать дочерние страницы (записи блога).

Создайте подпапку для каждой записи, которую хотите добавить, и в каждой папке разместите файл `item.md` с содержимым записи блога.

## URL

Описанная выше структура создаст записи блога с `/blog/` в URL. Возможно, вам нужно не это. Например, если на сайте есть только блог, а список записей — это домашняя страница. В таких случаях вы захотите открывать этот контент по корневому домену, а не отправлять посетителей в дочернюю папку.

В этом случае в system.yaml (системная конфигурация в админке) установите параметр `home.hide_in_urls` (Hide Home in URLs в админке) в true.

## Как это устроено

Возможно, вам интересно, как это работает. Шаблон Blog — содержимое файла `blog.html.twig` из папки темы `templates/` — просто перебирает свои дочерние страницы.

В простейшем виде:

```twig linenums="1"
{% set collection = page.collection() %}

{% for child in collection %}
        {% include 'partials/blog_item.html.twig' with {'blog':page, 'page':child, 'truncate':true} %}
{% endfor %}
```

page.collection() по умолчанию берёт свойство `content.items` из метаданных страницы в YAML и возвращает массив с элементами, которые соответствуют этому определению.

Если страница содержит:

```yaml linenums="1"
---
content:
    items: '@self.children'
---
```

то `collection` будет массивом дочерних страниц текущей страницы.

В этом случае тема подключает фрагмент шаблона `partials/blog_item.html.twig`, отвечающий за рендеринг отдельной записи блога, и передаёт ему объект `child` с той записью, которую нужно отрендерить.

### Подробнее

- Коллекции: [https://learn.getgrav.org/content/collections](https://learn.getgrav.org/content/collections)
- Страница-список: [https://learn.getgrav.org/content/content-pages#listing-page](https://learn.getgrav.org/content/content-pages#listing-page)
- Папки: [https://learn.getgrav.org/content/content-pages#folders](https://learn.getgrav.org/content/content-pages#folders)
- Таксономия: [https://learn.getgrav.org/content/taxonomy#taxonomy-example](https://learn.getgrav.org/content/taxonomy#taxonomy-example)
