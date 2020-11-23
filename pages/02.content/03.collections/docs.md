---
title: Коллекции страниц
page-toc:
  active: true
taxonomy:
    category: docs
---

Коллекции значительно выросли со времен ранних бета-версий Grav. Мы начали с очень ограниченного набора коллекций на основе страниц, но с помощью нашего сообщества мы расширили эти возможности, чтобы сделать коллекции ещё более мощными! Настолько, что теперь у них есть свой раздел в документации.

## Основы коллекций

В Grav наиболее распространенным типом коллекции является список страниц, который может быть определен либо в frontmatter страницы, либо в самой Twig-разметке. Наиболее распространенным является определение коллекции в frontmatter. Если коллекция определена, она доступна в Twig страницы, чтобы делать с ней всё, что вы пожелаете. Используя методы коллекции страниц или циклически проходя через каждый [объект страницы](https://learn.getgrav.org/themes/theme-vars#page-object) и используя методы или свойства страницы, вы можете делать мощные вещи. Распространенные примеры этого включают отображение списка записей в блоге или отображение модульных подстраниц для визуализации сложного дизайна страницы.

## Объект коллекции

Когда вы определяете коллекцию в заголовке страницы, вы динамически создаете [коллекцию Grav](https://github.com/getgrav/grav/blob/develop/system/src/Grav/Common/Page/Collection.php), доступную в Twig страницы. Этот объект коллекции является **итерабельным** и может рассматриваться как **массив**, который позволяет вам делать такие вещи, как:

[prism classes="language-twig line-numbers"]
{{ dump(page.collection[page.path]) }}
[/prism]

## Пример определения коллекции

Пример коллекции, определенной в frontmatter страницы:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.children'
    order:
        by: date
        dir: desc
    limit: 10
    pagination: true
[/prism]

Значение `content.items` в frontmatter страницы указывает Grav собирать коллекцию элементов, и информация, передаваемая ей, определяет, как должна быть построена коллекция.

Это определение создает коллекцию для страницы, которая состоит из всех **дочерних страниц**, отсортированных по **дате** и **по убыванию**, отображаемых вместе с **нумерацией страниц**, по **10 элементов** на страницу.

## Доступ к коллекциям в Twig

Когда эта коллекция определена в заголовке, Grav создает коллекцию **page.collection**, к которой можно получить доступ в шаблоне Twig:

[prism classes="language-twig line-numbers"]
{% for p in page.collection %}
<h2>{{ p.title }}</h2>
{{ p.summary }}
{% endfor %}
[/prism]

Это просто перебирает [страницы](https://learn.getgrav.org/themes/theme-vars#page-object) в коллекции с отображением заголовка и резюме.

Вы также можете включить параметр порядка, чтобы изменить порядок страниц по умолчанию:

[prism classes="language-twig line-numbers"]
{% for p in page.collection.order('folder','asc') %}
<h2>{{ p.title }}</h2>
{{ p.summary }}
{% endfor %}
[/prism]

## Заголовки коллекций

Чтобы сообщить Grav, что определенная страница должна быть страницей списков и содержать дочерние страницы, есть ряд переменных, которые можно использовать:

### Сводка параметров коллекции

[div class="table-keycol"]
|                   Строка                  |                           Результат                          |
|-------------------------------------------|-----------------------------------------------------------|
| `'@root'`                                   | Получить дочерние страницы от корня                                     |
| `'@root.children'`                          | Получить дочерние страницы от корня (альтернативный способ)                       |
| `'@root.descendants'`                       | Получить корневой элемент и рекурсивно пройтись по всем дочерним страницам             |
|                                           |                                                           |
| `'@self.parent'`                            | Получить родителя текущей страницы                        |
| `'@self.siblings'`                          | Коллекция всех страниц того же уровня, что и текущая             |
| `'@self.modular'`                           | Получить только модульные дочерние страницы                             |
| `'@self.children'`                          | Получить немодульные дочерние  страницы                              |
| `'@self.descendants'`                       | Рекурсивно пройтись по всем немодульным дочерним страницам              |
|                                           |                                                           |
| `'@page': '/fruit'`                         | Получить все дочерние страницы `/fruit`                     |
| `'@page.children': '/fruit'`                | Альтернативный способ (см. выше)                                      |
| `'@page.self': '/fruit'`                    | Получить коллекцию с элементом `/fruit`              |
| `'@page.page': '/fruit'`                    | Альтернативный способ (см. выше)                                     |
| `'@page.descendants': '/fruit'`             | Получить и рекурсивно пройтись по всем дочерним страницам `/fruit` |
| `'@page.modular': '/fruit'`                | Получить коллекцию всех модульных дочерних страниц `/fruit`      |
|                                           |                                                           |
| `'@taxonomy.tag': photography`              | таксономия с tag=`photography`                           |
| `'@taxonomy': {tag: birds, category: blog}` | таксономия с tag=`birds` и category=`blog`              |
[/div]

! Этот документ описывает использование `@page`, `@taxonomy.category` и т. д., но для YAML безопасней использовать альтернативный формат `page@`, `taxonomy@.category`.  Все команды `@` могут быть записаны либо в префиксном, либо в постфиксном формате.

Мы рассмотрим их более подробно.

## Корневые коллекции

##### @root - Потомки верхнего уровня

Это может быть использовано для извлечения верхнего/корневого уровня **опубликованных немодульных дочерних элементов** сайта. Особенно полезно для получения элементов, составляющих основную навигацию, например:

[prism classes="language-yaml line-numbers"]
content:
    items: '@root'
[/prism]

альтернативный способ:

[prism classes="language-yaml line-numbers"]
content:
    items: '@root.children'
[/prism]

##### @root - Потомки верхнего уровня + все потомки

Это позволит эффективно получить каждую страницу вашего сайта, поскольку он рекурсивно перемещается по всем дочерним элементам от корневой страницы вниз и создает коллекцию **всех опубликованных немодульных дочерних элементов** сайта:

[prism classes="language-yaml line-numbers"]
content:
    items: '@root.descendants'
[/prism]

## Самостоятельные коллекции

##### @self.children - Дочерние элементы текущей страницы

Это используется для перечисления **опубликованных немодульных дочерних элементов** текущей страницы:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.children'
[/prism]

##### @self.descendants - Немодульные потомки + все потомки текущей страницы

Подобно `.children`, коллекция `.descendants` получит все **опубликованные немодульные дочерние страницы**, но вдобавк ещё и рекурсивно получит всех потомков:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.descendants'
[/prism]

##### @self.modular - Модульные дочерние элементы текущей страницы

В отличие от `.children`, этот метод получает только **опубликованные модульные дочерние страницы** текущей (`_features`, `_showcase` и т. д.):

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.modular'
[/prism]

##### @self.parent - Родительская страница текущей страницы

Это особый случай коллекции, потому что он всегда будет возвращать только один **родитель** текущей страницы:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.parent'
[/prism]

##### @self.siblings - Все родственные страницы

Эта коллекция соберет все **опубликованные** страницы на том же уровне текущей страницы, исключая текущую страницу:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.siblings'
[/prism]

## Коллекции страниц

##### @page или @page.children - Коллекция дочерних элементов указанной страницы

Эта коллекция принимает маршрут указанной страницы в качестве аргумента и возвращает все **опубликованные немодульные** дочерние элементы этой страницы:

[prism classes="language-yaml line-numbers"]
content:
    items:
      '@page': '/blog'
[/prism]

или:

[prism classes="language-yaml line-numbers"]
content:
    items:
      '@page.children': '/blog'
[/prism]

##### @page.self или @page.page - Коллекция только конкретной страницы

Эта коллекция принимает маршрут указанной страницы в качестве аргумента и возвращает коллекцию, содержащую эту страницу (если она **опубликована и немодульна**):

[prism classes="language-yaml line-numbers"]
content:
    items:
      '@page.self': '/blog'
[/prism]

##### @page.descendants - Коллекция детей + все потомки определенной страницы

Эта коллекция принимает маршрут указанной страницы в качестве аргумента и возвращает все **опубликованные немодульные** дочерние элементы и всех их потомков:

[prism classes="language-yaml line-numbers"]
content:
    items:
      '@page.descendants': '/blog'
[/prism]

##### @page.modular - Коллекция модульных дочерних элементов определенной страницы

Эта коллекция принимает маршрут указанной страницы в качестве аргумента и возвращает все **опубликованные модульные** дочерние элементы этой страницы:

[prism classes="language-yaml line-numbers"]
content:
    items:
      '@page.modular': '/blog'
[/prism]


## Коллекции таксономии

[prism classes="language-yaml line-numbers"]
content:
   items:
      '@taxonomy.tag': foo
[/prism]

С опцией `@taxonomy` вы можете использовать мощную функциональность таксономии Grav. Именно здесь вступает в действие переменная `taxonomy` в файле [конфигурации сайта](../../basics/grav-configuration#site-configuration). **Должно** быть определение для таксономии, определенной в этом файле конфигурации для Grav, чтобы интерпретировать ссылку на страницу как действительную.

Установив `@taxonomy.tag: foo`, Grav найдет все **опубликованные страницы** в папке `/user/pages`, имеющие `tag: foo` в своей переменной таксономии:

[prism classes="language-yaml line-numbers"]
content:
    items:
       '@taxonomy.tag': [foo, bar]
[/prism]

Переменная `content.items` может принимать массив таксономий и собирать все страницы, удовлетворяющие этим правилам. Будут собраны опубликованные страницы, которые имеют **как** тег `foo`, **так и** тег `bar`. В главе [Таксономия](../taxonomy) эта концепция рассмотрена более подробно.

!! Если вы хотите поместить несколько переменных в строку, вам нужно отделить подэлементы от их родителей с помощью угловых скобок `{}`. Затем вы можете разделить отдельные переменные на этом уровне запятой. Например: `'@taxonomy': {category: [blog, featured], tag: [foo, bar]}`. В этом примере суб-переменные `category` и `tag` помещаются в `@taxonomy` в иерархии, каждая из которых содержит перечисленные значения в квадратных скобках `[]`. Страницы должны соответствовать **всем** этим требованиям.

Если у вас есть несколько переменных в одном родителе для установки, вы можете сделать это с помощью встроенного метода, но для простоты мы рекомендуем использовать стандартный метод. Вот пример:

[prism classes="language-yaml line-numbers"]
content:
  items:
    '@taxonomy':
      category: [blog, featured]
      tag: [foo, bar]
[/prism]

На каждом уровне иерархии перед переменной добавляются два пробела. YAML позволит вам использовать столько пробелов, сколько вы хотите, но два - стандартная практика. В приведенном выше примере переменные `category` и `tag` установлены в `@taxonomy`.

Коллекции страниц будут автоматически отфильтрованы по таксономии, если она была указана в URL (например, `/archive/category:news`). Это позволяет создать один шаблон архива блога, но динамически фильтровать коллекцию с помощью URL. Если ваша коллекция страниц должна игнорировать таксономию, установленную в URL, используйте флаг `url_taxonomy_filters: false`, чтобы отключить эту функцию.

### Сложные коллекции

Вы также можете предоставить несколько сложных определений коллекции, и результирующая коллекция будет суммой всех страниц, найденных из каждого из определений коллекции. Например:

[prism classes="language-yaml line-numbers"]
content:
  items:
    - '@self.children'
    - '@taxonomy':
         category: [blog, featured]
[/prism]

Кроме того, вы можете фильтровать коллекцию, используя `filter: type: value`. Тип может быть любым из следующих: `published`, `non-published`, `visible`, `non-visible`, `modular`, `non-modular`, `routable`, `non-routable`, `type`, `types`, `access`. Они соответствуют [методам коллекции](#collection-object-methods), и вы можете использовать несколько для фильтрации вашей коллекции. Все они являются `true` или `false`, за исключением `type`, который принимает одно имя шаблона, `types`, который принимает массив имен шаблона, и `access`, который принимает массив уровней доступа. Например:

[prism classes="language-yaml line-numbers"]
 content:
  items: '@self.siblings'
  filter:
    published: true
    type: 'blog'
[/prism]

### Варианты сортировки

[prism classes="language-yaml line-numbers"]
content:
    order:
        by: date
        dir: desc
    limit: 5
    pagination: true
[/prism]

Порядок подстраниц соответствует тем же правилам, что и порядок папок. Доступные опции:

[div class="table-keycol"]
| Порядок    | Подробности                                                                                                                                          |
| :----------  | :----------                                                                                                                                        |
| `default`    | Порядок зависит от файловой системе, например `01.home` идет перед `02.advark`                                                                              |
| `title`      | Порядок зависит от названии, определенном на каждой странице                                                                                            |
| `basename`   | Порядок зависит от алфавитном имени папки после того, как она была обработана PHP-функцией `basename()`                                      |
| `date`       | Порядок зависит от дате, определенной на каждой странице                                                                                                |
| `modified`   | Порядок зависит от времени изменения страницы                                                                                             |
| `folder`     | Порядок зависит от имени папки с любым числовым префиксом, например: `01.`                                                                  |
| `header.x`   | Порядок зависит от конкретном поле `header` страницы. Например, `header.taxonomy.year`. Кроме того, значение по умолчанию может быть добавлено через вертикальную черту. Например, `header.taxonomy.year|2015`    |
| `manual`     | Порядок зависит от переменной `order_manual`                                                                                                    |
| `random`     | Случайный порядок                                                                                                                            |
| `custom`     | Порядок зависит от переменной `content.order.custom`                                                                                                    |
| `sort_flags` | Разрешить переопределение флагов сортировки для упорядочения на основе заголовка страницы или по умолчанию. Если загружено расширение PHP `intl`, то доступны только [эти флаги](https://secure.php.net/manual/en/collator.asort.php). В противном случае вы можете использовать [стандартные флаги сортировки](https://secure.php.net/manual/en/array.constants.php) PHP. |
[/div]

Переменная `content.order.dir` определяет направление сортировки. Допустимые значения: `desc` или `asc`.

[prism classes="language-yaml line-numbers"]
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
[/prism]

В приведенной выше конфигурации вы можете видеть, что `content.order.custom` определяет **пользовательский порядок** для указания того, что на странице сначала идет **витрина**, затем секция **основных моментов** и т. д. Обратите внимание, что если страница не указана в списке пользовательской сортировки, то Grav возвращается к `content.order.by` для неуказанных страниц.

Если на странице есть настраиваемый ярлык, вы должны использовать его в списке `content.order.custom`.

`content.pagination` — это простой логический флаг, который используется плагинами и т. д. Чтобы знать, следует ли инициализировать **разбиение на страницы** для этой коллекции. `content.limit` — количество элементов, отображаемых на странице при включенной нумерации страниц.

### Диапазон дат

Начиная с **Grav 0.9.13** — возможность фильтрации по диапазону дат:

[prism classes="language-yaml line-numbers"]
content:
    items: '@self.children'
    dateRange:
        start: 1/1/2014
        end: 1/1/2015
[/prism]

Вы можете использовать любой строковый формат даты, поддерживаемый [strtotime()](https://php.net/manual/ru/function.strtotime.php), например `6 weeks` или `last Monday`, а также более традиционные даты, такие как `01/23/2014` или `23 января 2014`. DateRange отфильтрует любые страницы, у которых есть дата за пределами предоставленного диапазона. Даты **начала** и **конца** являются необязательными, но следует указать хотя бы одну.

### Несколько коллекций

Когда вы создаете коллекцию с `content: items:` в вашем YAML, вы определяете одну коллекцию на основе нескольких условий. Однако Grav позволяет создавать произвольный набор коллекций на странице, вам просто нужно создать ещё одну:

[prism classes="language-yaml line-numbers"]
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
[/prism]

Это устанавливает **2 коллекции** для этой страницы, первая использует коллекцию по умолчанию `content`, а вторая определяет коллекцию на основе таксономии под названием `fruit`. Для доступа к этим двум коллекциям через Twig вы можете использовать следующий синтаксис:

[prism classes="language-yaml line-numbers"]
{% set default_collection = page.collection %}
{% set fruit_collection = page.collection('fruit') %}
[/prism]

## Методы объекта коллекции

Доступные итерируемые методы:

[div class="table-keycol"]
| Параметр | Описание |
| -------- | ----------- |
| `Collection::append($items)` | Добавить другую коллекцию или массив |
| `Collection::first()` | Полуить первый элемент коллекции |
| `Collection::last()` | Получить последний элемент коллекции |
| `Collection::random($num)` | Получить `$num` случайных элементов коллекции |
| `Collection::reverse()` | Обратный порядок коллекции |
| `Collection::shuffle()` | Перемешать всю коллекцию |
| `Collection::slice($offset, $length)` | Обрезать список |
[/div]

Также есть несколько полезных методов, специфичных для коллекции:

[div class="table-keycol"]
| Параметр | Описание |
| -------- | ----------- |
| `Collection::addPage($page)` | Можно добавить другую страницу в коллекцию |
| `Collection::copy()` | Создать копию текущей коллекции |
| `Collection::current()` | Получить текущий элемент коллекции |
| `Collection::key()` | Получить слаг текущего элемента |
| `Collection::remove($path)` | Удалить указанную страницу из коллекции, или текущую, если `$path = null` |
| `Collection::order($by, $dir, $manual)` | Сортировка текущей коллекции |
| `Collection::intersect($collection2)` | Объединить две коллекции, сохранив элементы, которые встречаются в обеих коллекциях (как условие "И") |
| `Collection::merge($collection2)` | Объединить две коллекции, сохранив элементы, которые встречаются в любой коллекции (как условие "ИЛИ") |
| `Collection::isFirst($path)` | Определить, является ли указанная страница первой |
| `Collection::isLast($path)` | Определить, является ли указанная страница последней |
| `Collection::prevSibling($path)` | Если возможно, возвращает предыдущую страницу того же уровня |
| `Collection::nextSibling($path)` | Если возможно, возвращает следующую страницу того же уровня |
| `Collection::currentPosition($path)` | Возвращает текущий индекс |
| `Collection::dateRange($startDate, $endDate, $field)` | Фильтрует текущую коллекцию по датам |
| `Collection::visible()` | Фильтрует текущую коллекцию, чтобы включить только видимые страницы |
| `Collection::nonVisible()` | Фильтрует текущую коллекцию, чтобы включить только невидимые страницы |
| `Collection::modular()` | Фильтрует текущую коллекцию, чтобы включить только модульные страницы |
| `Collection::nonModular()` | Фильтрует текущую коллекцию, чтобы включить только немодульные страницы |
| `Collection::published()` | Фильтрует текущую коллекцию, чтобы включить только опубликованные страницы |
| `Collection::nonPublished()` | Фильтрует текущую коллекцию для включения только неопубликованных страниц |
| `Collection::routable()` | Фильтрует текущую коллекцию, чтобы включить только маршрутизируемые страницы |
| `Collection::nonRoutable()` | Фильтрует текущую коллекцию, чтобы включить только немаршрутизируемые страницы |
| `Collection::ofType($type)` | Отфильтровывает текущую коллекцию, чтобы включить только страницы с шаблоном = `$type` |
| `Collection::ofOneOfTheseTypes($types)` | Фильтрует текущую коллекцию, чтобы включить только те страницы, где шаблон находится в массиве `$types` |
| `Collection::ofOneOfTheseAccessLevels($levels)` | Фильтрует текущую коллекцию, чтобы включить только страницы, доступ к которым осуществляется в массиве `$levels` |
[/div]

Вот пример из **docs.html.twig** темы **Learn2**, который определяет коллекцию на основе таксономии (и, возможно, тегов, если они существуют) и использует методы `Collection::isFirst` и `Collection::isLast` для навигации по страницам:

[prism classes="language-twig line-numbers"]
{% set tags = page.taxonomy.tag %}
{% if tags %}
    {% set progress = page.collection({'items':{'@taxonomy':{'category': 'docs', 'tag': tags}},'order': {'by': 'default', 'dir': 'asc'}}) %}
{% else %}
    {% set progress = page.collection({'items':{'@taxonomy':{'category': 'docs'}},'order': {'by': 'default', 'dir': 'asc'}}) %}
{% endif %}

{% block navigation %}
        <div id="navigation">
        {% if not progress.isFirst(page.path) %}
            <a class="nav nav-prev" href="{{ progress.nextSibling(page.path).url }}"> <i class="fa fa-chevron-left"></i></a>
        {% endif %}

        {% if not progress.isLast(page.path) %}
            <a class="nav nav-next" href="{{ progress.prevSibling(page.path).url }}"><i class="fa fa-chevron-right"></i></a>
        {% endif %}
        </div>
{% endblock %}
[/prism]

`nextSibling()` — вверх по списку, `prevSibling()` — вниз по списку, вот как это работает:

Предположим, что у вас есть страницы:

[prism classes="language-txt line-numbers"]
Project A
Project B
Project C
[/prism]

Вы находитесь на странице «Проект A», предыдущая страница - это «Проект B».
При переходе на страницу «Проект B» предыдущей станет «Проект C», а следующей — «Проект A».


## Программные коллекции

Вы можете получить полный контроль над коллекциями прямо из PHP в плагинах, темах Grav или даже из Twig. Это более жестко запрограммированный подход по сравнению с определением их во внешнем оформлении страницы, но он также позволяет использовать более сложную и гибкую логику коллекций.

### PHP-коллекции

Вы можете выполнить расширенную логику сбора с помощью PHP, например:

[prism classes="language-php line-numbers"]
$collection = new Collection($pages);
$collection->setParams(['taxonomies' => ['tag' => ['dog', 'cat']]])->dateRange('01/01/2016', '12/31/2016')->published()->ofType('blog-item')->order('date', 'desc');

$titles = [];

foreach ($collection as $page) {
    $titles[] = $page->title();
}
[/prism]

Функция `order()` может также, в дополнение к параметрам `by` и `dir`, принимать параметры `manual` и `sort_flags`. Это [задокументировано выше](#ordering-options). Вы также можете использовать тот же метод `evaluate()`, который используют коллекции страниц на основе frontmatter:

[prism classes="language-php line-numbers"]
$page = Grav::instance()['page'];
$collection = $page->evaluate(['@page.children' => '/blog', '@taxonomy.tag' => 'photography']);
$ordered_collection = $collection->order('date', 'desc');
[/prism]

И ещё один пример индивидуальной сортировки:

[prism classes="language-php line-numbers"]
$ordered_collection = $collection->order('header.price','asc',null,SORT_NUMERIC);
[/prism]

Вы также можете сделать то же самое непосредственно в **шаблонах Twig**:

[prism classes="language-twig line-numbers"]
{% set collection = page.evaluate([{'@page.children':'/blog', '@taxonomy.tag':'photography'}]) %}
{% set ordered_collection = collection.order('date','desc') %}
[/prism]

#### Расширенные коллекции

По умолчанию, когда вы вызываете `page.collection()` в Twig страницы, имеющей коллекцию, определенную в заголовке, Grav ищет коллекцию с названием `content`. Это позволяет определять [несколько коллекций](#multiple-collections), но вы можете пойти дальше.

Если вам нужно программно сгенерировать коллекцию, вы можете сделать это, вызвав `page.collection()` и передав массив в том же формате, что и определение коллекции заголовков страницы. Например:

[prism classes="language-twig line-numbers"]
{% set options = { items: {'@page.children': '/my/pages'}, 'limit': 5, 'order': {'by': 'date', 'dir': 'desc'}, 'pagination': true } %}
{% set my_collection = page.collection(options) %}

<ul>
{% for p in my_collection %}
<li>{{ p.title }}</li>
{% endfor %}
</ul>
[/prism]

Генерация меню для всего сайта (вам необходимо установить свойство **menu** во frontmatter страницы):


[prism classes="language-yaml line-numbers"]
---
title: Home
menu: Home
---
[/prism]

[prism classes="language-twig line-numbers"]
{% set options = { items: {'@root.descendants':''}, 'order': {'by': 'folder', 'dir': 'asc'}} %}
{% set my_collection = page.collection(options) %}

{% for p in my_collection %}
{% if p.header.menu %}
	<ul>
	{% if page.slug == p.slug %}
		<li class="{{ p.slug }} active"><span>{{ p.menu }}</span></li>
	{% else %}
		<li class="{{ p.slug }}"><a href="{{ p.url }}">{{ p.menu }}</a></li>
	{% endif %}
	</ul>
{% endif %}
{% endfor %}
[/prism]

#### Пагинация с расширенными коллекциями

Часто мы слышим вопрос о том, как включить разбиение на страницы для пользовательских коллекций. Пагинация - это плагин `pagination`, который можно установить через GPM. После установки он работает «из коробки» с коллекциями, настроенными на странице, но ничего не знает о пользовательских коллекциях, созданных в Twig. Чтобы упростить этот процесс, разбиение на страницы поставляется с собственной функцией Twig, называемой `paginate()`, которая обеспечит необходимую функциональность разбивки на страницы.

После того, как мы передаем коллекцию и ограничение функции `paginate()`, нам также необходимо передать информацию о разбиении на страницы непосредственно в шаблон `partials/pagination.html.twig` для правильного отображения.

[prism classes="language-twig line-numbers"]
{% set options = { items: {'@root.descendants':''}, 'order': {'by': 'folder', 'dir': 'asc'}} %}
{% set my_collection = page.collection(options) %}
{% do paginate( my_collection, 5 ) %}

{% for p in my_collection %}
    <ul>
        {% if page.slug == p.slug %}
            <li class="{{ p.slug }} active"><span>{{ p.menu }}</span></li>
        {% else %}
            <li class="{{ p.slug }}"><a href="{{ p.url }}">{{ p.menu }}</a></li>
        {% endif %}
    </ul>
{% endfor %}

{% include 'partials/pagination.html.twig' with {'base_url':page.url, 'pagination':my_collection.params.pagination} %}
[/prism]


[version=16]
#### Пользовательская обработка коллекции с событием `onCollectionProcessed()`

Бывают случаи, когда вариантов событий просто недостаточно. Когда вы хотите получить коллекцию, но затем продолжите манипулировать коллекцией на основе чего-то очень нестандартного. Представьте себе, если хотите, вариант использования, в котором у вас есть то, что кажется довольно стандартным списком блогов, но ваш клиент хочет иметь точный контроль над тем, что отображается в списке. Они хотят иметь настраиваемый переключатель для каждого элемента блога, который позволяет им удалить его из списка, но при этом опубликовать его и получить по прямой ссылке.

Чтобы это произошло, мы можем просто добавить настраиваемую опцию `display_in_listing: false` в заголовок страницы для элемента:

[prism classes="language-yaml line-numbers"]
---
title: 'My Story'
date: '13:34 04/14/2020'
taxonomy:
    tag:
        - journal
display_in_listing: false
---
...
[/prism]

Проблема в том, что нет способа определить или включить этот фильтр при определении коллекции на странице листинга. Вероятно, это определяется примерно так:

[prism classes="language-yaml line-numbers"]
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
[/prism]

Таким образом, коллекция просто определяется директивой `self@.chidren` для получения всех опубликованных дочерних элементов текущей страницы. Так что насчет тех страниц, у которых установлен параметр `display_in_listing: false`? Нам нужно проделать дополнительную работу с этой коллекцией, прежде чем она будет возвращена, чтобы убедиться, что мы удалим все элементы, которые мы не хотим видеть. Для этого мы можем использовать событие `onCollectionProcessed()` в настраиваемом плагине. Нам нужно добавить слушателя:

[prism classes="language-php line-numbers"]
    public static function getSubscribedEvents()
    {
        return [
            ['autoload', 100000],
            'onPluginsInitialized' => ['onPluginsInitialized', 0],
            'onCollectionProcessed' => ['onCollectionProcessed', 10]
        ];
    }
[/prism]

Затем нам нужно определить метод и перебрать элементы коллекции, ища любые страницы с этим набором полей `display_in_listing:`, а затем удалить его, если он равен `false`:

[prism classes="language-php line-numbers"]
    /**
     * Remove any page from collection with display_in_listing: false|0
     *
     * @param Event $event
     */
    public function onCollectionProcessed(Event $event)
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
[/prism]

Теперь в вашей коллекции есть правильные элементы, и все другие плагины или шаблоны Twig, которые полагаются на эту коллекцию, будут видеть эту измененную коллекцию, поэтому такие вещи, как разбиение на страницы, будут работать должным образом.
[/version]
