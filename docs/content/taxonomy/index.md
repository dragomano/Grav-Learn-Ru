---
title: "Таксономия"
description: "Узнайте о таксономии в Grav."
tags:
  - "Таксономия"
---

# Таксономия

В **Grav** возможность группировать страницы или назначать им теги встроена в систему с помощью **таксономии**.

---

> **Таксономия (в общем смысле)** — практика и наука (изучение) классификации вещей или понятий, включая принципы, лежащие в основе такой классификации.
>
> <cite>Wikipedia</cite>

В использовании таксономии на сайте есть несколько ключевых частей:

1. Определите список типов таксономии в файле [`site.yaml`](../../basics/grav-configuration/index.md).
2. Назначьте страницам соответствующие типы `taxonomy` со значениями.

## Пример таксономии

Эту концепцию лучше всего объяснить на примере. Допустим, вы хотите создать простой блог. В этом блоге вы будете создавать записи, которые, возможно, захотите привязать к определённым тегам для отображения **облака тегов**. Кроме того, у вас может быть несколько авторов, и каждую запись, возможно, нужно привязать к её автору.

В Grav это делается просто. Grav предоставляет файл `site.yaml` по умолчанию, расположенный в папке `system/config`. По умолчанию эта конфигурация определяет два типа таксономии: `category` и `tag`:

```yaml
taxonomies: [category,tag]
```

Поскольку `tag` уже определён, нужно просто добавить `authors`. Для этого создайте новый файл `site.yaml` в папке `user/config` и добавьте следующую строку:

```yaml
taxonomies: [category,tag,author]
```

Это переопределит таксономии, известные Grav, чтобы страницы можно было назначать любой из этих трёх таксономий.

Следующий шаг — создать несколько страниц, использующих эти типы таксономии. Например, страница может выглядеть так:

```markdown linenums="1"
---
title: Post 1
taxonomy:
    tag: [animal, dog]
    author: ksmith
---

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
```

…и другая страница, которая выглядит так:

```markdown linenums="1"
---
title: Post 2
taxonomy:
    tag: [animal, cat]
    author: jdoe
---

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
```

Как видно из конфигурации YAML, каждая страница назначает **значения** **типам таксономии**, которые мы определили в пользовательской конфигурации `site.yaml`. Эта информация используется Grav при обработке страниц и создаёт внутреннюю **карту таксономии**, по которой можно находить страницы на основе заданной таксономии.

> [!TIP] Совет
> Страницы не обязаны использовать каждую таксономию, определённую в `site.yaml`, но любую используемую таксономию нужно определить.

В теме можно легко вывести список страниц, написанных `ksmith`, используя `taxonomy.findTaxonomy()` для их поиска и перебора:

```twig linenums="1"
<h2>Kevin Smith's Posts</h2>
<ul>
{% for post in taxonomy.findTaxonomy({'author':'ksmith'}) %}
    <li>{{ post.title|e }}</li>
{% endfor %}
</ul>
```

Можно также выполнять сложные поиски по нескольким таксономиям с помощью массивов/хешей, например:

```twig linenums="1"
{% for post in taxonomy.findTaxonomy({'tag':['animal','cat'],'author':'jdoe'}) %}
```

Это найдёт все записи, у которых `tag` равен `animal` **и** `cat`, **и** `author` равен `jdoe`. Фактически это найдёт именно **Post 2**.

Если нужна коллекция, включающая один термин **или** другой, просто добавьте параметр `'or'` после массива, например:

```twig linenums="1"
{% for post in taxonomy.findTaxonomy({'tag':['dog','cat']},'or') %}
```

Это найдёт все записи, у которых `tag` равен `dog` **или** `cat`.

## Коллекции на основе таксономии

Мы рассматривали это в предыдущей главе, но важно помнить, что таксономии можно использовать в [заголовках страниц](../headers/index.md) для фильтрации коллекции страниц, связанных с родительской страницей. Если нужно освежить эту тему, вернитесь к [главе о заголовках коллекций таксономии](../collections/index.md).

## Добавление собственных значений таксономии в Default и Options {: #adding-custom-taxonomy-values-in-default-and-optio }

В схемах (blueprints) можно использовать приведённый ниже формат, чтобы переопределить таксономии `Default` и/или `Options`. Важно: если вы используете этот способ для переопределения обоих атрибутов, добавьте `validate: type: commalist`, иначе это может работать не так, как нужно.

```yaml linenums="1"
taxonomies:
  fields:
    header.taxonomy:
      default:
        category: ['blog','page']
        tag: ['test']
      options:
        category: ['grav']
      validate:
        type: commalist
```
