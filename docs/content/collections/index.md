---
title: "Коллекции страниц"
description: "Узнайте о коллекциях страниц в Grav."
tags:
  - "Контент"
  - "Grav 1.7"
  - "Коллекции"
  - "Страницы"
---

# Коллекции страниц

В Grav самый распространённый тип коллекции — список страниц, который можно определить в метаданных страницы или непосредственно в Twig. Чаще всего коллекция определяется в метаданных страницы. Определённая коллекция становится доступна в Twig этой страницы, и с ней можно делать что угодно.

---

Используя методы коллекций страниц или перебирая каждый [объект `Page`](https://learn.getgrav.org/themes/theme-vars#page-object) и применяя его методы или свойства, можно делать очень многое. Типичные примеры — вывод списка записей блога или отображение модульных дочерних страниц для создания сложного дизайна страницы.

## Объект Collection

Когда вы определяете коллекцию в заголовке страницы, вы динамически создаёте [коллекцию Grav](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Page/Collection.php), доступную в Twig этой страницы. Этот объект Collection **итерируемый**, с ним можно обращаться как с **массивом**, что позволяет делать, например, так:

```twig linenums="1"
{{ dump(page.collection[page.path]) }}
```

## Пример определения коллекции

Пример коллекции, определённой в метаданных страницы:

```yaml linenums="1"
content:
    items: '@self.children'
    order:
        by: date
        dir: desc
    limit: 10
    pagination: true
```

Значение `content.items` в метаданных страницы указывает Grav собрать коллекцию элементов, а переданная информация определяет, как эта коллекция будет построена.

Это определение создаёт для страницы коллекцию, состоящую из всех **дочерних страниц**, отсортированных по **дате** **по убыванию**, с **пагинацией** по **10 элементов** на страницу.

Ссылки пагинации добавляются только если установлен и включён [плагин Pagination](https://github.com/getgrav/grav-plugin-pagination).

## Доступ к коллекциям в Twig

Когда коллекция определена в заголовке, Grav создаёт коллекцию **`page.collection`**, к которой можно обратиться в Twig-шаблоне так:

```twig linenums="1"
{% for p in page.collection %}
<h2>{{ p.title|e }}</h2>
{{ p.summary|raw }}
{% endfor %}
```

Этот код просто перебирает [страницы](https://learn.getgrav.org/themes/theme-vars#page-object) в коллекции и выводит заголовок и аннотацию.

Можно также передать параметр сортировки, чтобы изменить порядок страниц по умолчанию:

```twig linenums="1"
{% for p in page.collection.order('folder','asc') %}
<h2>{{ p.title|e }}</h2>
{{ p.summary|raw }}
{% endfor %}
```

## Заголовки коллекций

Чтобы указать Grav, что конкретная страница должна быть страницей-списком и содержать дочерние страницы, можно использовать ряд переменных:

### Сводка параметров коллекции

|                   Строка                      |                           Результат                          |
|-----------------------------------------------|-----------------------------------------------------------|
| `'@root.pages'`                               | Получить страницы верхнего уровня                                   |
| `'@root.descendants'`                         | Получить все страницы сайта                             |
| `'@root.all'`                                 | Получить все страницы и модули сайта                 |
|                                               |                                                           |
| `'@self.page'`                                | Получить коллекцию только с текущей страницей               |
| `'@self.parent'`                              | Получить коллекцию только с родительской страницей текущей страницы |
| `'@self.siblings'`                            | Получить соседние страницы текущей страницы                          |
| `'@self.children'`                            | Получить дочерние страницы текущей страницы                          |
| `'@self.modules'`                             | Получить модули текущей страницы                          |
| `'@self.all'`                                 | Получить и дочерние страницы, и модули текущей страницы         |
| `'@self.descendants'`                         | Рекурсивно обойти всех потомков текущей страницы      |
|                                               |                                                           |
| `'@page.page': '/fruit'`                      | Получить коллекцию только со страницей `/fruit`              |
| `'@page.parent': '/fruit'`                    | Получить коллекцию только с родительской страницей страницы `/fruit`|
| `'@page.siblings': '/fruit'`                  | Получить соседние страницы для страницы `/fruit`                     |
| `'@page.children': '/fruit'`                  | Получить дочерние страницы для страницы `/fruit`                     |
| `'@page.modules': '/fruit'`                   | Получить модули страницы `/fruit`                          |
| `'@page.all': '/fruit'`                       | Получить и дочерние страницы, и модули страницы `/fruit`        |
| `'@page.descendants': '/fruit'`               | Получить и рекурсивно обойти всех потомков страницы `/fruit` |
|                                               |                                                           |
| `'@taxonomy.tag': photography`                | таксономия с тегом `photography`                           |
| `'@taxonomy': {tag: birds, category: blog}`   | таксономия с тегом `birds` && категорией `blog`              |

> [!WARNING] Предупреждение
> В этом документе описано использование `@page`, `@taxonomy.category` и т. п., но более безопасный для YAML формат — `page@`, `taxonomy@.category`. Все команды с `@` можно записывать как в префиксном, так и в постфиксном формате.

> [!CAUTION] Внимание
> Параметры коллекций были улучшены и изменены начиная с **Grav 1.6**. Старые варианты по-прежнему работают, но использовать их не рекомендуется.

Мы рассмотрим их подробнее.

## Корневые коллекции

##### @root.pages — страницы верхнего уровня {: #atroot-pages-top-level-pages }

Позволяет получить **опубликованные страницы** верхнего/корневого уровня сайта. Особенно полезно для получения элементов, из которых состоит основная навигация, например:

```yaml linenums="1"
content:
    items: '@root.pages'
```

Псевдоним `'@root.children'` также допустим. Использование `'@root'` устарело, поскольку его значение может измениться в будущем.

##### @root.descendants — все страницы {: #atroot-descendants-all-the-pages }

Фактически получает каждую страницу сайта: Grav рекурсивно проходит по всем дочерним страницам начиная с корневой и строит коллекцию **всех** **опубликованных страниц** сайта:

```yaml linenums="1"
content:
    items: '@root.descendants'
```

##### @root.all — все страницы и модули {: #atroot-all-all-the-pages-and-modules }

Делает то же самое, что и выше, но включает **все** **опубликованные страницы и модули** сайта.

```yaml linenums="1"
content:
    items: '@root.all'
```

## Коллекции относительно текущей страницы

##### @self.page — только текущая страница {: #atself-page-current-page-only }

Возвращает коллекцию, содержащую только текущую страницу.

```yaml linenums="1"
content:
    items: '@self.page'
```

Псевдоним `'@self.self'` также допустим.

> [!WARNING] Предупреждение
> Если страница не опубликована, будет возвращена пустая коллекция.

##### @self.parent — родительская страница текущей страницы {: #atself-parent-the-parent-page-of-the-current-page }

Это особый случай коллекции, потому что она всегда возвращает только **родительскую** страницу текущей страницы:

```yaml linenums="1"
content:
    items: '@self.parent'
```

> [!WARNING] Предупреждение
> Если страница находится на верхнем уровне, будет возвращена пустая коллекция.

##### @self.siblings — соседние страницы текущей страницы {: #atself-siblings-siblings-of-the-current-page }

Эта коллекция собирает все **опубликованные страницы** того же уровня, что и текущая, исключая саму текущую страницу:

```yaml linenums="1"
content:
    items: '@self.siblings'
```

##### @self.children — дочерние страницы текущей страницы {: #atself-children-children-of-the-current-page }

Используется для вывода **опубликованных дочерних страниц** текущей страницы:

```yaml linenums="1"
content:
    items: '@self.children'
```

Псевдоним `'@self.pages'` также допустим. Использование `'@self'` устарело, поскольку его значение может измениться в будущем.

##### @self.modules — модули текущей страницы {: #atself-modules-modules-of-the-current-page }

Этот метод получает только **опубликованные модули** текущей страницы (`_features`, `_showcase` и т. д.):

```yaml linenums="1"
content:
    items: '@self.modules'
```

Использование псевдонима `'@self.modular'` устарело.

##### @self.all — дочерние страницы и модули текущей страницы {: #atself-all-children-and-modules-of-the-current-pag }

Этот метод получает только **опубликованные дочерние страницы и модули** текущей страницы:

```yaml linenums="1"
content:
    items: '@self.all'
```

##### @self.descendants — дочерние страницы и все потомки текущей страницы {: #atself-descendants-children-all-descendants-of-the }

Аналогично `.children`, коллекция `.descendants` получает всех **опубликованных дочерних страниц**, но продолжает рекурсивно обходить всех их потомков:

```yaml linenums="1"
content:
    items: '@self.descendants'
```

## Коллекции конкретных страниц {: #page-collections-1 }

##### @page.page — коллекция только с указанной страницей {: #atpage-page-collection-of-just-the-specific-page }

Эта коллекция принимает в качестве аргумента маршрут-слаг страницы и возвращает коллекцию, содержащую эту страницу (если она **опубликована**):

```yaml linenums="1"
content:
    items:
      '@page.page': '/blog'
```

Псевдоним `'@page.self': '/blog'` также допустим.

> [!WARNING] Предупреждение
> Если страница не опубликована, будет возвращена пустая коллекция.

##### @page.parent — родительская страница указанной страницы {: #atpage-parent-the-parent-page-of-a-specific-page }

Это особый случай коллекции, потому что она всегда возвращает только **родительскую** страницу текущей страницы:

```yaml linenums="1"
content:
    items:
      '@page.parent': '/blog'
```

@
> [!WARNING] Предупреждение
> Если страница находится на верхнем уровне, будет возвращена пустая коллекция.

##### @page.siblings — соседние страницы указанной страницы {: #atpage-siblings-siblings-of-a-specific-page }

Эта коллекция собирает все **опубликованные страницы** того же уровня, что и указанная страница, исключая саму страницу:

```yaml linenums="1"
content:
    items:
        '@page.siblings': '/blog'
```

##### @page.children — дочерние страницы указанной страницы {: #atpage-children-children-of-a-specific-page }

Эта коллекция принимает в качестве аргумента маршрут-слаг страницы и возвращает всех **опубликованных дочерних страниц** этой страницы:

```yaml linenums="1"
content:
    items:
      '@page.children': '/blog'
```

Псевдоним `'@page.pages': '/blog'` также допустим. Использование `'@page': '/blog'` устарело, поскольку его значение может измениться в будущем.

##### @page.modules — модули указанной страницы {: #atpage-modules-modules-of-a-specific-page }

Эта коллекция принимает в качестве аргумента маршрут-слаг страницы и возвращает все **опубликованные модули** этой страницы:

```yaml linenums="1"
content:
    items:
      '@page.modules': '/blog'
```

Использование псевдонима `'@page.modular': '/blog'` устарело.

##### @page.all — дочерние страницы и модули указанной страницы {: #atpage-all-children-and-modules-of-a-specific-page }

Этот метод получает только **опубликованные дочерние страницы и модули** указанной страницы:

```yaml linenums="1"
content:
    items:
      '@page.all': '/blog'
```

##### @page.descendants — коллекция дочерних страниц и всех потомков указанной страницы {: #atpage-descendants-collection-of-children-all-desc }

Эта коллекция принимает в качестве аргумента маршрут-слаг страницы и возвращает всех **опубликованных дочерних страниц** и всех их потомков:

```yaml linenums="1"
content:
    items:
      '@page.descendants': '/blog'
```

## Коллекции таксономии

```yaml linenums="1"
content:
   items:
      '@taxonomy.tag': foo
```

С помощью параметра `@taxonomy` можно использовать мощные возможности таксономии Grav. Здесь в игру вступает переменная `taxonomy` из файла [конфигурации сайта](../../basics/grav-configuration/index.md#site-configuration). Чтобы Grav считал ссылку на таксономию допустимой, **должно** существовать её определение в этом файле конфигурации.

Задав `@taxonomy.tag: foo`, вы заставите Grav найти все **опубликованные страницы** в папке `/user/pages`, у которых в переменной таксономии задано `tag: foo`:

```yaml linenums="1"
content:
    items:
       '@taxonomy.tag': [foo, bar]
```

Переменная `content.items` может принимать массив таксономий и собирает все страницы, удовлетворяющие этим правилам. Будут собраны опубликованные страницы, у которых есть **одновременно** теги `foo` **и** `bar`. Подробнее эта концепция рассматривается в главе [Таксономия](../taxonomy/index.md).

> [!CAUTION] Внимание
> Если нужно указать несколько переменных в одной строке, разделяйте подпеременные от родителей фигурными скобками `{}`. Отдельные переменные на этом уровне разделяются запятой. Например: `'@taxonomy': {category: [blog, featured], tag: [foo, bar]}`. В этом примере подпеременные `category` и `tag` размещены под `@taxonomy` в иерархии, каждая со списком значений в квадратных скобках `[]`. Чтобы страница была найдена, она должна удовлетворять **всем** этим требованиям.

Если в одном родителе нужно задать несколько переменных, можно использовать встроенный способ, но для простоты мы рекомендуем стандартный. Пример:

```yaml linenums="1"
content:
  items:
    '@taxonomy':
      category: [blog, featured]
      tag: [foo, bar]
```

Каждый уровень иерархии добавляет два пробела перед переменной. YAML допускает любое количество пробелов, но два — стандартная практика. В примере выше переменные `category` и `tag` заданы под `@taxonomy`.

Коллекции страниц автоматически фильтруются по таксономии, если она задана в URL (например, `/archive/category:news`). Это позволяет создать один шаблон архива блога и динамически фильтровать коллекцию через URL. Если коллекция страницы должна игнорировать таксономию из URL, используйте флаг `url_taxonomy_filters:false`, чтобы отключить эту фичу.

### Сложные коллекции

Можно также задать несколько сложных определений коллекций, и итоговая коллекция будет объединением всех страниц, найденных по каждому определению. Например:

```yaml linenums="1"
content:
  items:
    - '@self.children'
    - '@taxonomy':
         category: [blog, featured]
```

Кроме того, коллекцию можно фильтровать с помощью `filter: type: value`. Тип может быть любым из следующих: `published`, `visible`, `page`, `module`, `routable`. Они соответствуют [методам, специфичным для коллекции](#collection-object-methods), и можно использовать несколько фильтров одновременно. Все они принимают значение `true` или `false`. Кроме того, есть `type`, принимающий одно имя шаблона, `types`, принимающий массив имён шаблонов, и `access`, принимающий массив уровней доступа. Например:

```yaml linenums="1"
content:
 items: '@self.siblings'
 filter:
   visible: true
   type: 'blog'
   access: ['site.login']
```

Тип может быть и отрицательным: `non-published`, `non-visible`, `non-page` (= модуль), `non-module` (= страница) и `non-routable`, но предпочтительнее использовать положительный вариант со значением `false`.

```yaml linenums="1"
content:
 items: '@self.children'
 filter:
   published: false
```

> [!CAUTION] Внимание
> Фильтры коллекций были упрощены начиная с **Grav 1.6**. Старые варианты `modular` и `non-modular` по-прежнему работают, но использовать их не рекомендуется. Вместо них используйте `module` и `page`.

## Параметры порядка {: #ordering-options }

```yaml linenums="1"
content:
    order:
        by: date
        dir: desc
    limit: 5
    pagination: true
```

Порядок дочерних страниц подчиняется тем же правилам, что и порядок папок. Доступны следующие варианты:

| Порядок     | Описание                                                                                                                                            |
| :----------  | :----------                                                                                                                                        |
| `default`    | Порядок основан на файловой системе, то есть `01.home` идёт перед `02.advark`                                                                              |
| `title`      | Порядок основан на заголовке, определённом на каждой странице                                                                                            |
| `basename`   | Порядок основан на алфавитном имени папки после обработки PHP-функцией `basename()`                                      |
| `date`       | Порядок основан на дате, определённой на каждой странице                                                                                                |
| `modified`   | Порядок основан на отметке времени последнего изменения страницы                                                                                              |
| `folder`     | Порядок основан на имени папки без числового префикса, то есть `01.`, удалённого                                                                  |
| `header.x`   | Порядок основан на любом поле заголовка страницы, то есть `header.taxonomy.year`. Значение по умолчанию можно добавить через вертикальную черту, то есть `header.taxonomy.year|2015`    |
| `random`     | Порядок случайный                                                                                                                            |
| `custom`     | Порядок основан на переменной `content.order.custom`                                                                                                                             |
| `manual`     | Порядок основан на переменной `order_manual`, заданной в метаданных **родительской** страницы                                                                       |
| `sort_flags` | Позволяет переопределить флаги сортировки для порядка на основе заголовков страниц или порядка по умолчанию. Если загружено PHP-расширение `intl`, доступны только [эти флаги](https://secure.php.net/manual/en/collator.asort.php). Иначе можно использовать PHP [стандартные флаги сортировки](https://secure.php.net/manual/en/array.constants.php). |

Переменная `content.order.dir` управляет направлением сортировки. Допустимые значения — `desc` или `asc`.

```yaml linenums="1"
content:
    order:
        by: default
        custom:
            - _showcase
            - _highlights
            - _callout
            - _features
    limit: 5
    pagination: true
```

В приведённой конфигурации `content.order.custom` задаёт **собственный ручной порядок**, чтобы страница строилась со секцией **showcase** первой, **highlights** второй и т. д. Обратите внимание: если страница не указана в списке ручного порядка, для неё Grav использует резервный вариант из `content.order.by`.

Если у страницы есть собственный слаг, в списке `content.order.custom` нужно использовать именно его.

> [!NOTE] Примечание
> `order_manual` и `content.order.custom` выполняют разные задачи и не взаимозаменяемы. `order_manual` задаётся в метаданных **родительской** страницы, перечисляет слаги дочерних страниц в нужном порядке и меняет порядок `page.children`, то есть порядок ваших меню и всего остального, что строится из дерева страниц. `content.order.custom` меняет порядок только коллекции, возвращаемой `page.collection()`; на `page.children` он не влияет и навигацию не меняет. Используйте `order_manual`, когда нужно изменить порядок самого дерева, и `content.order.custom`, когда нужно упорядочить только одну коллекцию. Учтите, что `order_manual` не может изменить порядок меню верхнего уровня, потому что у корня страниц нет родительской страницы, из метаданных которой Grav мог бы его прочитать.

`content.pagination` — простой логический флаг, который плагины и т. п. используют, чтобы понять, нужно ли инициализировать **пагинацию** для этой коллекции. `content.limit` — количество элементов, отображаемых на странице при включённой пагинации.

### Диапазон дат

Есть также возможность фильтровать страницы по диапазону дат:

```yaml linenums="1"
content:
    items: '@self.children'
    dateRange:
        start: 1/1/2014
        end: 1/1/2015
```

Можно использовать любой строковый формат даты, поддерживаемый [`strtotime()`](https://php.net/manual/en/function.strtotime.php), например `-6 weeks` или `last Monday`, а также более традиционные даты вроде `01/23/2014` или `23 January 2014`. `dateRange` отфильтрует страницы, дата которых выходит за пределы указанного диапазона. Обе даты, **start** и **end**, необязательны, но хотя бы одну нужно указать.

### Несколько коллекций {: #multiple-collections }

Когда вы создаёте коллекцию с помощью `content: items:` в YAML, вы определяете одну коллекцию на основе нескольких условий. Однако Grav позволяет создать на странице произвольный набор коллекций — достаточно создать ещё одну:

```yaml linenums="1"
content:
    items: '@self.children'
    order:
        by: date
        dir: desc
    limit: 10
    pagination: true

fruit:
    items:
       '@taxonomy.tag': [fruit]
```

Так для страницы создаются **2 коллекции**: первая использует коллекцию `content` по умолчанию, а вторая определяет коллекцию на основе таксономии с именем `fruit`. Чтобы обратиться к этим двум коллекциям из Twig, используйте такой синтаксис:

```yaml linenums="1"
{% set default_collection = page.collection %}
{% set fruit_collection = page.collection('fruit') %}
```

## Методы объекта Collection {: #collection-object-methods }

К итерируемым методам относятся:

| Свойство | Описание |
| -------- | ----------- |
| `Collection::append($items)` | Добавить другую коллекцию или массив |
| `Collection::first()` | Получить первый элемент коллекции |
| `Collection::last()` | Получить последний элемент коллекции |
| `Collection::random($num)` | Выбрать `$num` случайных элементов коллекции |
| `Collection::reverse()` | Изменить порядок коллекции на обратный |
| `Collection::shuffle()` | Перемешать всю коллекцию случайным образом |
| `Collection::slice($offset, $length)` | Обрезать список |

Есть также несколько полезных методов, специфичных для `Collection`:

| Свойство | Описание |
| -------- | ----------- |
| `Collection::addPage($page)` | Добавить другую страницу в коллекцию |
| `Collection::copy()` | Создаёт копию текущей коллекции |
| `Collection::current()` | Получает текущий элемент коллекции |
| `Collection::key()` | Возвращает слаг текущего элемента |
| `Collection::remove($path)` | Удаляет указанную страницу из коллекции или текущую, если `$path = null` |
| `Collection::order($by, $dir, $manual)` | Сортирует текущую коллекцию |
| `Collection::intersect($collection2)` | Объединяет две коллекции, оставляя элементы, которые есть в обеих (как условие «И») |
| `Collection::merge($collection2)` | Объединяет две коллекции, оставляя элементы, которые есть в любой из них (как условие «ИЛИ») |
| `Collection::isFirst($path)` | Определяет, является ли страница, указанная путём, первой |
| `Collection::isLast($path)` | Определяет, является ли страница, указанная путём, последней |
| `Collection::prevSibling($path)` | Возвращает предыдущую соседнюю страницу, если она есть |
| `Collection::nextSibling($path)` | Возвращает следующую соседнюю страницу, если она есть |
| `Collection::currentPosition($path)` | Возвращает текущий индекс |
| `Collection::dateRange($startDate, $endDate, $field)` | Фильтрует текущую коллекцию по датам |
| `Collection::visible()` | Фильтрует текущую коллекцию, оставляя только видимые страницы |
| `Collection::nonVisible()` | Фильтрует текущую коллекцию, оставляя только невидимые страницы |
| `Collection::pages()` | Фильтрует текущую коллекцию, оставляя только страницы (без модулей) |
| `Collection::modules()` | Фильтрует текущую коллекцию, оставляя только модули (без страниц) |
| `Collection::published()` | Фильтрует текущую коллекцию, оставляя только опубликованные страницы |
| `Collection::nonPublished()` | Фильтрует текущую коллекцию, оставляя только неопубликованные страницы |
| `Collection::routable()` | Фильтрует текущую коллекцию, оставляя только маршрутизируемые страницы |
| `Collection::nonRoutable()` | Фильтрует текущую коллекцию, оставляя только немаршрутизируемые страницы |
| `Collection::ofType($type)` | Фильтрует текущую коллекцию, оставляя только страницы, где template = `$type` |
| `Collection::ofOneOfTheseTypes($types)` | Фильтрует текущую коллекцию, оставляя только страницы, чей шаблон входит в массив `$types` |
| `Collection::ofOneOfTheseAccessLevels($levels)` | Фильтрует текущую коллекцию, оставляя только страницы, чей уровень доступа входит в массив `$levels` |

> [!CAUTION] Внимание
> В **Grav 1.7** устарели следующие методы: `Collection::modular()` и `Collection::nonModular()`. Используйте вместо них `Collection::modules()` и `Collection::pages()` соответственно.

Вот пример из файла **docs.html.twig** темы **Learn2**: он определяет коллекцию на основе таксономии (и, если они есть, тегов) и использует методы `Collection::isFirst` и `Collection::isLast`, чтобы условно добавить навигацию по страницам:

```twig linenums="1"
{% set tags = page.taxonomy.tag %}
{% if tags %}
    {% set progress = page.collection({'items':{'@taxonomy':{'category': 'docs', 'tag': tags}},'order': {'by': 'default', 'dir': 'asc'}}) %}
{% else %}
    {% set progress = page.collection({'items':{'@taxonomy':{'category': 'docs'}},'order': {'by': 'default', 'dir': 'asc'}}) %}
{% endif %}

{% block navigation %}
        <div id="navigation">
        {% if not progress.isFirst(page.path) %}
            <a class="nav nav-prev" href="{{ progress.nextSibling(page.path).url|e }}"> <i class="fa fa-chevron-left"></i></a>
        {% endif %}

        {% if not progress.isLast(page.path) %}
            <a class="nav nav-next" href="{{ progress.prevSibling(page.path).url|e }}"><i class="fa fa-chevron-right"></i></a>
        {% endif %}
        </div>
{% endblock %}
```

`nextSibling()` ведёт выше по списку, а `prevSibling()` — ниже. Работает это так:

Предположим, у вас есть страницы:

```text linenums="1"
Project A
Project B
Project C
```

Вы находитесь на странице Project A, предыдущая страница — Project B.
Если вы находитесь на странице Project B, предыдущая — Project C, а следующая — Project A.

## Коллекции в коде

Вы можете полностью управлять коллекциями прямо из PHP в плагинах Grav, темах или даже из Twig. Это более жёстко заданный подход по сравнению с определением в метаданных страницы, но он позволяет реализовать более сложную и гибкую логику коллекций.

### Коллекции в PHP

В PHP можно реализовать сложную логику коллекций, например:

```php linenums="1"
$collection = new Collection($pages);
$collection->setParams(['taxonomies' => ['tag' => ['dog', 'cat']]])->dateRange('01/01/2016', '12/31/2016')->published()->ofType('blog-item')->order('date', 'desc');

$titles = [];

foreach ($collection as $page) {
    $titles[] = $page->title();
}
```

Функция `order()` помимо параметров `by` и `dir` может принимать параметры `manual` и `sort_flags`. Они [описаны выше](#ordering-options). Можно также использовать тот же метод `evaluate()`, который применяется в коллекциях страниц на основе метаданных:

```php linenums="1"
$page = Grav::instance()['page'];
$collection = $page->evaluate(['@page.children' => '/blog', '@taxonomy.tag' => 'photography']);
$ordered_collection = $collection->order('date', 'desc');
```

Ещё один пример собственного порядка:

```php linenums="1"
$ordered_collection = $collection->order('header.price','asc',null,SORT_NUMERIC);
```

Подобное можно делать непосредственно в **Twig-шаблонах**:

```twig linenums="1"
{% set collection = page.evaluate([{'@page.children':'/blog', '@taxonomy.tag':'photography'}]) %}
{% set ordered_collection = collection.order('date','desc') %}
```

#### Продвинутые коллекции

По умолчанию, когда вы вызываете `page.collection()` в Twig страницы, у которой коллекция определена в заголовке, Grav ищет коллекцию с именем `content`. Это позволяет определять [несколько коллекций](#multiple-collections), но можно пойти ещё дальше.

Если нужно программно сгенерировать коллекцию, вызовите `page.collection()` и передайте массив в том же формате, что и определение коллекции в заголовке страницы. Например:

```twig linenums="1"
{% set options = { items: {'@page.children': '/my/pages'}, 'limit': 5, 'order': {'by': 'date', 'dir': 'desc'}, 'pagination': true } %}
{% set my_collection = page.collection(options) %}

<ul>
{% for p in my_collection %}
<li>{{ p.title|e }}</li>
{% endfor %}
</ul>
```

Генерация меню для всего сайта (нужно задать свойство *menu* в метаданных страницы):

```yaml linenums="1"
---
title: Home
menu: Home
---
```

```twig linenums="1"
{% set options = { items: {'@root.descendants':''}, 'order': {'by': 'folder', 'dir': 'asc'}} %}
{% set my_collection = page.collection(options) %}

{% for p in my_collection %}
{% if p.header.menu %}
	<ul>
	{% if page.slug == p.slug %}
		<li class="{{ p.slug|e }} active"><span>{{ p.menu|e }}</span></li>
	{% else %}
		<li class="{{ p.slug|e }}"><a href="{{ p.url|e }}">{{ p.menu|e }}</a></li>
	{% endif %}
	</ul>
{% endif %}
{% endfor %}
```

#### Пагинация для сложных коллекций

Частый вопрос — как включить пагинацию для собственных коллекций. Pagination — плагин, который можно установить через GPM под именем `pagination`. После установки он работает «из коробки» с коллекциями, настроенными на странице, но ничего не знает о собственных коллекциях, созданных в Twig. Чтобы упростить задачу, Pagination предоставляет собственную Twig-функцию `paginate()`, которая обеспечивает нужную функциональность пагинации.

После передачи коллекции и лимита в функцию `paginate()` нужно также передать информацию о пагинации непосредственно в шаблон `partials/pagination.html.twig`, чтобы он отрендерился правильно.

```twig linenums="1"
{% set options = { items: {'@root.descendants':''}, 'order': {'by': 'folder', 'dir': 'asc'}} %}
{% set my_collection = page.collection(options) %}
{% do paginate( my_collection, 5 ) %}

{% for p in my_collection %}
    <ul>
        {% if page.slug == p.slug %}
            <li class="{{ p.slug|e }} active"><span>{{ p.menu|e }}</span></li>
        {% else %}
            <li class="{{ p.slug|e }}"><a href="{{ p.url|e }}">{{ p.menu|e }}</a></li>
        {% endif %}
    </ul>
{% endfor %}

{% include 'partials/pagination.html.twig' with {'base_url':page.url, 'pagination':my_collection.params.pagination} %}
```

#### Собственная обработка коллекции с событием `onCollectionProcessed()` {: #custom-collection-handling-with-oncollectionproces }

Иногда возможностей параметров события просто не хватает. Бывает, что нужно получить коллекцию, а затем дополнительно изменить её по какому-то очень специфичному правилу. Представьте сценарий: у вас обычный, казалось бы, список блога, но клиент хочет тонко контролировать, что отображается в списке. Он хочет, чтобы у каждой записи блога был собственный переключатель, позволяющий убрать её из списка, но при этом оставить опубликованной и доступной по прямой ссылке.

Для этого достаточно добавить в заголовок страницы записи собственный параметр `display_in_listing: false`:

```yaml linenums="1"
---
title: 'My Story'
date: '13:34 04/14/2020'
taxonomy:
    tag:
        - journal
display_in_listing: false
---
...
```

Проблема в том, что этот фильтр невозможно определить или включить при определении коллекции на странице-списке. Скорее всего, коллекция определяется примерно так:

```yaml linenums="1"
---
menu: News
title: 'My Blog'
content:
    items:
        - self@.children
    order:
        by: date
        dir: desc
    limit: 8
    pagination: true
    url_taxonomy_filters: true
---
...
```

То есть коллекция просто задаётся директивой `self@.children`, чтобы получить всех опубликованных дочерних страниц текущей страницы. А как быть со страницами, у которых задано `display_in_listing: false`? Нужно дополнительно обработать коллекцию перед возвратом, чтобы удалить ненужные элементы. Для этого можно использовать событие `onCollectionProcessed()` в собственном плагине. Нужно добавить слушатель:

```php linenums="1"
public static function getSubscribedEvents(): array
{
    return [
        ['autoload', 100000],
        'onPluginsInitialized' => ['onPluginsInitialized', 0],
        'onCollectionProcessed' => ['onCollectionProcessed', 10]
    ];
}
```

Затем нужно определить метод и перебрать элементы коллекции, находя страницы с заданным полем `display_in_listing:` и удаляя их, если значение `false`:

```php linenums="1"
/**
 * Удаляет из коллекции страницы с display_in_listing: false|0
 *
 * @param Event $event
 */
public function onCollectionProcessed(Event $event): void
{
    /** @var Collection $collection */
    $collection = $event['collection'];

    foreach ($collection as $item) {
        $display_in_listing = $item->header()->display_in_listing ?? true;
        if ((bool) $display_in_listing === false) {
            $collection->remove($item->path());
        }
    }

}
```

Теперь ваша коллекция содержит правильные элементы, и все другие плагины или Twig-шаблоны, использующие эту коллекцию, увидят изменённую коллекцию, поэтому пагинация и т. п. будут работать как ожидается.
