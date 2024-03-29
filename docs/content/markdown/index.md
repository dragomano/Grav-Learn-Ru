---
description: Небольшая памятка об основах синтаксиса Markdown, используемого в Grav CMS.
---

# Синтаксис Markdown

Посмотрим правде в глаза: написание контента для Интернета утомительно. Редакторы WYSIWYG облегчают эту задачу, но обычно они приводят к ужасному коду или, что ещё хуже, уродливым веб-страницам.

**Markdown** — лучший способ написать **HTML**, без всех сложностей и уродства, которые обычно сопровождают его.

Некоторые из ключевых преимуществ:

1. Markdown прост в освоении, с минимальным количеством дополнительных символов, поэтому писать контент также быстрее.
2. Меньше вероятность ошибок при записи.
3. Создает действительный вывод XHTML.
4. Содержит контент и визуальный дисплей отдельно, поэтому вы не можете испортить внешний вид вашего сайта.
5. Пишите в любом текстовом редакторе или приложении Markdown, которое вам нравится.
6. Использование Markdown — одно удовольствие!

Джон Грубер, автор Markdown, говорит об этом так:

> Основная цель разработки синтаксиса форматирования Markdown - сделать его максимально читаемым. Идея состоит в том, что отформатированный с помощью Markdown документ должен быть опубликован как есть, как обычный текст, не выглядя так, как будто он размечен тегами или инструкциями по форматированию. Несмотря на то, что на синтаксис Markdown влияют несколько существующих фильтров преобразования текста в HTML, единственным основным источником вдохновения для синтаксиса Markdown является формат простой текстовой электронной почты.
> -- <cite>Джон Грубер</cite>

Grav поставляется со встроенной поддержкой [Markdown](https://daringfireball.net/projects/markdown/) и [Markdown Extra](https://michelf.ca/projects/php-markdown/extra/). Вы должны включить **Markdown Extra** в конфигурационном файле `system.yaml`.

Давайте рассмотрим основные элементы Markdown и то, как выглядит полученный HTML:

!!! tip ""

	:material-bookmark: Добавьте эту страницу в закладки для удобства использования в будущем!

## Заголовки

Заголовки от `h1` до `h6` состоят из символа `#` для каждого уровня:

```markdown
# Заголовок h1
## Заголовок h2
### Заголовок h3
#### Заголовок h4
##### Заголовок h5
###### Заголовок h6
```

HTML:

```
<h1>Заголовок h1</h1>
<h2>Заголовок h2</h2>
<h3>Заголовок h3</h3>
<h4>Заголовок h4</h4>
<h5>Заголовок h5</h5>
<h6>Заголовок h6</h6>
```

## Комментарии

Комментарии должны быть совместимы с HTML

```
<!--
Это комментарий
-->
```

Комментарий ниже **не должен** быть виден:

<!--
Это комментарий
-->

## Горизонтальные линии

Элемент HTML `<hr>` предназначен для создания «тематического разрыва» между элементами уровня абзаца. В Markdown вы можете создать <hr> любым из способов ниже:

* `___`: три последовательных подчеркивания
* `---`: три последовательных тире
* `***`: три звездочки подряд

преобразуется в:

___

---

***

## Обычный текст

Обычный текст будет заключен в теги `<p> </p>` в обработанном HTML.

Итак, это обычный текст:

```markdown
Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.
```

преобразуется в такой HTML:

```
<p>Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.</p>
```

**Разрыв строки** может быть выполнен с 2 пробелами, за которыми следует 1 перенос строки.

## Встроенный HTML

Если вам нужен определённый HTML-тег (с классом), вы можете использовать обычный HTML:

```
Абзац в Markdown.

<div class="class">
    Это <b>HTML</b>
</div>

Абзац в Markdown.
```

## Акцент

### Жирный

Для выделения фрагмента текста с большим весом шрифта.

Следующий фрагмент текста **выделен жирным шрифтом**.

```markdown
**преобразуется в жирный текст**
```

преобразуется в:

**преобразуется в жирный текст**

и такой HTML:

```
<strong>преобразуется в жирный текст</strong>
```

### Курсив

Для выделения фрагмента текста курсивом.

Следующий фрагмент текста _представлен как курсив_.

```markdown
_преобразуется в курсив_
```

преобразуется в:

_преобразуется в курсив_

и такой HTML:

```
<em>преобразуется в курсив</em>
```

### Зачеркнутый

В GFM (GitHub Flavored Markdown) вы можете делать зачеркивания.

```markdown
~~Зачеркни этот текст.~~
```

Преобразуется в:

~~Зачеркни этот текст.~~

HTML:

```
<del>Зачеркни этот текст.</del>
```

## Цитаты

Для цитирования блоков контента из другого источника в вашем документе.

Добавьте `>` перед любым текстом, который вы хотите процитировать.

```markdown
> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.
```

Преобразуется в:

> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.

и такой HTML:

```
<blockquote>
  <p><strong>Fusion Drive</strong> combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.</p>
</blockquote>
```

Цитаты также могут быть вложенными:

```markdown
> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.
```

Преобразуется в:

> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.

## Уведомления

!!! info ""

	Старый механизм уведомлений, отменяющий синтаксис блочных кавычек (`>>>`), устарел. Уведомления теперь обрабатываются через специальный плагин под названием [Markdown Notices](https://github.com/getgrav/grav-plugin-markdown-notices)

## Списки

### Неупорядоченный

Список элементов, в котором порядок явно не имеет значения.

Вы можете использовать любой из следующих символов для обозначения маркеров для каждого элемента списка:

```markdown
* правильный маркер
- правильный маркер
+ правильный маркер
```

Пример

```markdown
+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem
```

Преобразуется в:

+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem

И такой HTML

```
<ul>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit
    <ul>
      <li>Phasellus iaculis neque</li>
      <li>Purus sodales ultricies</li>
      <li>Vestibulum laoreet porttitor sem</li>
      <li>Ac tristique libero volutpat at</li>
    </ul>
  </li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ul>
```

### Упорядоченный

Список элементов, в котором порядок элементов явно имеет значение.

```markdown
1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem
```

Преобразуется в:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

И такой HTML:

```
<ol>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit</li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ol>
```

!!! tip ""

	Если вы просто используете `1.` для каждого пункта списка, Markdown применит автоматическую нумерацию. Например:

```markdown
1. Lorem ipsum dolor sit amet
1. Consectetur adipiscing elit
1. Integer molestie lorem at massa
1. Facilisis in pretium nisl aliquet
1. Nulla volutpat aliquam velit
1. Faucibus porta lacus fringilla vel
1. Aenean sit amet erat nunc
1. Eget porttitor lorem
```

Преобразуется в:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

## Код

### Встроенный код

Оберните встроенные фрагменты кода в `` ` ``.

```markdown
В этом примере блок `<section> </section>` должен быть заключен в **код**.
```

Преобразуется в:

В этом примере блок `<section> </section>` должен быть заключен в **код**.

HTML:

```
<p>В этом примере блок <code>&lt;section&gt;&lt;/section&gt;</code> должен быть заключен в <strong>code</strong>.</p>
```

### Код с отступом

Или сделайте отступ в нескольких строках кода как минимум на четыре пробела, например:

````
```
  // Комментарий
  line 1 of code
  line 2 of code
  line 3 of code
```
````

Преобразуется в:

```text
// Комментарий
line 1 of code
line 2 of code
line 3 of code
```

HTML:

```
<pre>
  <code>
    // Комментарий
    line 1 of code
    line 2 of code
    line 3 of code
  </code>
</pre>
```

### Блок кода с «заборчиками»

Используйте тройные кавычки ```` ``` ````, чтобы вывести текст в нескольких строках кода с атрибутом языка

````
```
Образец текста...
```
````

HTML:

```
<pre language-html>
  <code>Образец текста...</code>
</pre>
```

### Подсветка синтаксиса

GFM или «GitHub Flavored Markdown» также поддерживает подсветку синтаксиса. Чтобы активировать её, просто добавьте расширение файла на языке, который вы хотите использовать, сразу после открывающего блока с тройными кавычками, ` ```js `, и выделение синтаксиса будет автоматически применяться в выходном HTML. Например, чтобы применить подсветку синтаксиса к коду JavaScript:

````
```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```
````

Преобразуется в:

```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```

!!! success ""

	Чтобы подсветка синтаксиса работала, нужно установить и включить [плагин Highlight](https://github.com/getgrav/grav-plugin-highlight). Он, в свою очередь, использует плагин jquery, поэтому jquery также должен подключаться в вашем шаблоне.

## Таблицы

Таблицы создаются путем добавления вертикальных черточек в качестве разделителей между каждой ячейкой и добавления линий тире (также разделенных полосами) под заголовком. Обратите внимание, что вертикальные черточки не нужно выравнивать по вертикали.

```markdown
| Параметр | Описание |
| ------ | ----------- |
| data   | путь к файлам данных для предоставления данных, которые будут переданы в шаблоны. |
| engine | движок, который будет использоваться для обработки шаблонов. Handlebars по умолчанию. |
| ext    | расширение, которое будет использоваться для файлов назначения. |
```

Преобразуется в:

| Параметр | Описание |
| ------ | ----------- |
| data   | путь к файлам данных для предоставления данных, которые будут переданы в шаблоны. |
| engine | движок, который будет использоваться для обработки шаблонов. Handlebars по умолчанию. |
| ext    | расширение, которое будет использоваться для файлов назначения. |

И такой HTML:

```
<table>
  <thead>
    <tr>
      <th>Параметр</th>
      <th>Описание</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>data</td>
      <td>путь к файлам данных для предоставления данных, которые будут переданы в шаблоны.</td>
    </tr>
    <tr>
      <td>engine</td>
      <td>движок, который будет использоваться для обработки шаблонов. Handlebars по умолчанию.</td>
    </tr>
    <tr>
      <td>ext</td>
      <td>расширение, которое будет использоваться для файлов назначения.</td>
    </tr>
  </tbody>
</table>
```

### Текст с выравниванием по правому краю

Добавление двоеточия справа от тире под любым заголовком выравнивает текст этого столбца по правому краю.

```markdown
| Параметр | Описание |
| ------:| -----------:|
| data   | путь к файлам данных для предоставления данных, которые будут переданы в шаблоны. |
| engine | движок, который будет использоваться для обработки шаблонов. Handlebars по умолчанию. |
| ext    | расширение, которое будет использоваться для файлов назначения. |
```

| Параметр | Описание |
| ------:| -----------:|
| data   | путь к файлам данных для предоставления данных, которые будут переданы в шаблоны. |
| engine | движок, который будет использоваться для обработки шаблонов. Handlebars по умолчанию. |
| ext    | расширение, которое будет использоваться для файлов назначения. |

## Ссылки

### Обычная ссылка

```markdown
[Assemble](https://assemble.io)
```

Рендерится так:

[Assemble](https://assemble.io)

HTML:

```
<a href="https://assemble.io">Assemble</a>
```

### Добавление подсказки при наведении

```markdown
[Upstage](https://github.com/upstage/ "Visit Upstage!")
```

Наведите курсор на ссылку, должна появиться подсказка:

[Upstage](https://github.com/upstage/ "Visit Upstage!")

HTML:

```
<a href="https://github.com/upstage/" title="Visit Upstage!">Upstage</a>
```

### Именованные якоря

Именованные привязки позволяют перейти к указанной точке привязки на той же странице. Например, каждая из этих глав:

```markdown
# Table of Contents
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)
```

ведёт к соответствующим заголовкам на странице:

```markdown
## Chapter 1 <a id="chapter-1"></a>
Content for chapter one.

## Chapter 2 <a id="chapter-2"></a>
Content for chapter one.

## Chapter 3 <a id="chapter-3"></a>
Content for chapter one.
```

!!! note ""

	Конкретное размещение тега привязки кажется произвольным. Они размещены здесь встроенными, так как это кажется ненавязчивым, и это работает.

## Изображения

Изображения имеют синтаксис, аналогичный ссылкам, но содержат предшествующий восклицательный знак.

```markdown
![Minion](https://octodex.github.com/images/minion.png)
```
![Minion](https://octodex.github.com/images/minion.png)

или:

```markdown
![Alt text](https://octodex.github.com/images/stormtroopocat.jpg "The Stormtroopocat")
```
![Alt text](https://octodex.github.com/images/stormtroopocat.jpg "The Stormtroopocat")

Как и ссылки, изображения также имеют синтаксис стиля сносок:

```markdown
![Alt text][id]
```
![Alt text][id]

Со ссылкой позже в документе, определяющем расположение URL:

[id]: https://octodex.github.com/images/dojocat.jpg  "The Dojocat"


    [id]: https://octodex.github.com/images/dojocat.jpg  "The Dojocat"
