---
description: Знакомимся с понятием таксономии в Grav CMS. Изучаем коллекции таксономий.
---

# Таксономия

Благодаря **Grav** возможность группировать или помечать страницы встроена прямо в систему с **таксономией**.

> **Таксономия** — практика и наука (изучение) классификации вещей или понятий, включая принципы, лежащие в основе такой классификации.
>
> <cite>Википедия</cite>

Использование таксономии на вашем сайте состоит из нескольких ключевых моментов:

1. Определите список типов таксономии в [`site.yaml`](/basics/grav-configuration).
2. Назначьте своим страницам соответствующие типы `таксономии` со значениями.

## Пример таксономии

Эту концепцию лучше всего пояснить на примере. Допустим, вы хотите создать простой блог. В этом блоге вы будете создавать сообщения, которым, возможно, захотите назначить определенные теги, чтобы обеспечить отображение **облака тегов**. Кроме того, у вас может быть несколько авторов, и вы можете назначить каждое сообщение этому автору.

Достичь этого в Grav — простая процедура. Grav предоставляет файл `site.yaml` по умолчанию, который находится в папке `system/config`. По умолчанию эта конфигурация определяет два таксономических типа `category` и `tag`:

```yaml
taxonomies: [category,tag]
```

Так как `tag` уже определен, нужно просто добавить `authors`. Для этого просто создайте новый `site.yaml` файл в папке `пользователь/config` и добавьте следующую строку:

```yaml
taxonomies: [category,tag,author]
```

Это переопределит таксономии, о которых знает Grav, так что страницы могут быть отнесены к любой из этих трех таксономий.

Следующим шагом является создание нескольких страниц, использующих эти типы таксономии. Например, у вас может быть страница, которая выглядит так:

```markdown
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

... и ещё одну страницу, которая выглядит так:

```markdown
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


Как вы можете видеть в конфигурации YAML, каждая страница присваивает **значения** **таксономическим типам**, которые мы определили в нашей пользовательской конфигурации `site.yaml`. Эта информация используется Grav, когда страницы обрабатываются, и создает внутреннюю **таксономическую карту**, которая может быть использована для поиска страниц на основе таксономии, которую вы определили.

!!! danger ""

	Ваши страницы не обязаны использовать каждую таксономию, которую вы определили в своем `site.yaml`, но вы должны определить любую таксономию, которую вы используете.

В вашей теме вы можете легко отобразить список страниц, написанных `ksmith` с помощью `taxonomy.findTaxonomy()`, чтобы найти их и провести над ними итерацию:

```twig
<h2>Kevin Smith's Posts</h2>
<ul>
	{% for post in taxonomy.findTaxonomy({'author':'ksmith'}) %}
		<li>{{ post.title }}</li>
	{% endfor %}
</ul>
```

Вы также можете выполнять сложный поиск на основе нескольких таксономий, используя массивы/хэши, например:

```twig
{% for post in taxonomy.findTaxonomy({'tag':['animal','cat'],'author':'jdoe'}) %}
```

Здесь вы найдете все сообщения с `tag`, установленным в `animal` **и** `cat`, **и** `author`, установленным в `jdoe`. В основном, это специально для **Пост 2**.

Если вам нужна коллекция, которая включает в себя один термин **или** другой, просто добавьте параметр `or` после массива, например:

```twig
{% for post in taxonomy.findTaxonomy({'tag':['dog','cat']},'or') %}
```

Будут найдены все сообщения с тегом `dog` **или** `cat`.


## Коллекции на основе таксономии

Мы рассматривали это в одной из предыдущих глав, но важно помнить, что вы также можете использовать таксономию в [заголовках страниц](/content/headers), чтобы отфильтровать коллекцию страниц, связанных с родительской страницей. Если вам необходимо освежить эту тему, пожалуйста, обратитесь к этой [главе о заголовках сборников таксономии](/content/collections).

## Добавление пользовательских значений таксономии в значениях по умолчанию и параметрах

Вы можете использовать формат, приведенный ниже в чертежах, чтобы отменить таксономию `Default` и/или `Options`. Важным замечанием здесь является то, что если вы используете этот метод для переопределения обоих этих атрибутов, вы должны добавить `validate: type: commalist`, в противном случае он может не функционировать должным образом.

```yaml
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
