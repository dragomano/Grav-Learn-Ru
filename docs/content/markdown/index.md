---
title: "Синтаксис Markdown"
description: "Узнайте о синтаксисе Markdown в Grav."
tags:
  - "Контент"
  - "Markdown"
---

# Синтаксис Markdown

Скажем честно: писать контент для веба утомительно. Визуальные редакторы (WYSIWYG) немного облегчают задачу, но обычно в результате получается ужасный код или, что ещё хуже, некрасивые веб-страницы.

**Markdown** — лучший способ писать **HTML**, без всей сложности и уродства, которые обычно его сопровождают.

---

Некоторые из главных преимуществ:

1. Markdown легко освоить, дополнительных символов минимум, поэтому контент пишется быстрее.
2. Меньше вероятность ошибок при написании на Markdown.
3. Даёт корректный вывод XHTML.
4. Разделяет контент и визуальное отображение, поэтому вы не испортите внешний вид сайта.
5. Можно писать в любом текстовом редакторе или Markdown-приложении.
6. Markdown — одно удовольствие!

Джон Грубер, автор Markdown, описывает это так:

> Главная цель дизайна синтаксиса форматирования Markdown — сделать его максимально читаемым. Идея в том, что документ, отформатированный в Markdown, должен публиковаться как есть, в виде обычного текста, не выглядя при этом размеченным тегами или инструкциями форматирования. Хотя на синтаксис Markdown повлияли несколько существовавших фильтров текст-в-HTML, главный источник вдохновения — формат обычного текстового письма.
> -- <cite>Джон Грубер</cite>

Grav поставляется со встроенной поддержкой [Markdown](https://daringfireball.net/projects/markdown/) и [Markdown Extra](https://michelf.ca/projects/php-markdown/extra/). Чтобы использовать **Markdown Extra**, включите её в файле конфигурации `system.yaml`.

> [!NOTE] Примечание
> **Новое в Grav 2.0** — рендеринг Markdown получил более полный набор фич [GitHub Flavored Markdown](https://github.github.com/gfm/), **включённый по умолчанию**: [списки задач](#task-lists), [подсветка](#highlight)/[нижний индекс](#subscript)/[верхний индекс](#superscript), автоматическое [связывание голых адресов `www.` и email](#automatic-links), а также экранирование запрещённых необработанных HTML-тегов. Также доступен набор [расширений для таблиц](#enhanced-tables), включаемых по желанию: объединение ячеек, таблицы без заголовка, подписи, атрибуты и многострочные ячейки. Всё это настраивается; см. [Конфигурация](#configuration).

Не будем откладывать: рассмотрим основные элементы Markdown и то, как выглядит получающийся HTML:

> [!CAUTION] Внимание
> :fontawesome-solid-bookmark: Сохраните эту страницу в закладки, чтобы легко обращаться к ней в будущем!

## Заголовки

Заголовки от `h1` до `h6` создаются с помощью `#` для каждого уровня:

```markdown
# h1 Heading
## h2 Heading
### h3 Heading
#### h4 Heading
##### h5 Heading
###### h6 Heading
```

Рендерится так:

# h1 Heading
## h2 Heading
### h3 Heading
#### h4 Heading
##### h5 Heading
###### h6 Heading

HTML:

```html
<h1>h1 Heading</h1>
<h2>h2 Heading</h2>
<h3>h3 Heading</h3>
<h4>h4 Heading</h4>
<h5>h5 Heading</h5>
<h6>h6 Heading</h6>
```

## Комментарии

Комментарии должны быть совместимы с HTML.

```html
<!--
This is a comment
-->
```

Комментарий ниже **НЕ** должен отображаться:

<!--
This is a comment
-->

## Горизонтальные линии

HTML-элемент `<hr>` создаёт «тематический разрыв» между элементами уровня абзаца. В Markdown `<hr>` можно создать любым из следующих способов:

* `___`: три подряд идущих символа подчёркивания
* `---`: три подряд идущих дефиса
* `***`: три подряд идущих звёздочки

рендерится так:

___

---

***

## Основной текст

Обычный текст оборачивается тегами `<p></p>` в отрендеренном HTML.

Например, такой текст:

```markdown
Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.
```

рендерится в такой HTML:

```html
<p>Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.</p>
```

**Перенос строки** делается двумя пробелами и одним переводом строки.

## Встроенный HTML

Если нужен определённый HTML-тег (с классом), просто используйте HTML:

```html
Paragraph in Markdown.

<div class="class">
    This is <b>HTML</b>
</div>

Paragraph in Markdown.
```

> [!NOTE] Примечание
> **Grav 2.0** экранирует небольшой список запрещённых тегов «необработанного HTML» в вашем контенте — `title`, `textarea`, `style`, `xmp`, `iframe`, `noembed`, `noframes`, `script` и `plaintext`, — поэтому они выводятся как видимый текст, а не как активная разметка, что соответствует `tagfilter` в GitHub Flavored Markdown. Это удобство, помогающее избежать случайных поломок, а не граница безопасности; широкая очистка HTML остаётся задачей слоя безопасности вашего сайта. Это можно отключить в разделе [Конфигурация](#configuration).

### Разрешение iframe и другого необработанного HTML {: #allowing-iframes-and-other-raw-html }

Если вставка, работавшая в Grav 1.x (YouTube-`<iframe>`, встроенная карта, виджет `<script>`), теперь отображается на странице как обычный текст, причина в GFM `tagfilter`: он экранировал открывающий `<` тега, поэтому браузер печатает тег вместо того, чтобы выполнить его.

Стоит прояснить два момента, потому что их легко перепутать:

- **Список «опасных HTML-тегов»** в разделе *Конфигурация → Безопасность* — другая фича. Он управляет *сканером* безопасности, который предупреждает вас при сохранении контента с рискованной разметкой. Он никогда не управлял тем, рендерится ли тег, поэтому его изменение не влияет на исчезающий iframe.
- **Список запрещённых тегов `tagfilter` фиксирован** (`title`, `textarea`, `style`, `xmp`, `iframe`, `noembed`, `noframes`, `script`, `plaintext`) и не редактируется. Он соответствует спецификации GitHub Flavored Markdown.

Есть два хороших способа вернуть вставку:

1. **Рекомендуется: добавить её через шорткод или шаблон страницы**, а не вставлять необработанный HTML в Markdown. Так `tagfilter` остаётся включённым везде, а вставку можно разместить осознанно. См. [Создание шорткода](../creating-a-shortcode/index.md) и [подход с шаблоном страницы](../twig-in-content/index.md#use-a-page-template).
2. **Отключить `tagfilter`**, если вы предпочитаете разрешить необработанные HTML-теги на всём сайте. Это настраивается в разделе *Конфигурация → Система → Markdown* или в `user/config/system.yaml`:

```yaml
pages:
  markdown:
    gfm:
      tagfilter: false
```

> [!NOTE] Примечание
> Отключение `tagfilter` **не** равносильно отключению очистки. Оно лишь перестаёт экранировать этот короткий список запрещённых тегов в выводе Markdown. Настоящая защита вашего сайта от XSS (слой `security.xss_*`, который охватывает опасные протоколы, атрибуты обработчиков событий, встроенные стили и многое другое) в любом случае остаётся полностью активной. Оставлять `tagfilter` выключенным разумно на сайте, где контент пишете вы и другие доверенные авторы; включайте его там, где вы рендерите недоверенный или присланный пользователями Markdown, и используйте в этом случае вариант 1.

## Выделение

### Жирный

Для выделения фрагмента текста более жирным начертанием.

Следующий фрагмент текста **rendered as bold text**.

```markdown
**rendered as bold text**
```

рендерится так:

**rendered as bold text**

и такой HTML:

```html
<strong>rendered as bold text</strong>
```

### Курсив

Для выделения фрагмента текста курсивом.

Следующий фрагмент текста _rendered as italicized text_.

```markdown
_rendered as italicized text_
```

рендерится так:

_rendered as italicized text_

и такой HTML:

```html
<em>rendered as italicized text</em>
```

### Зачёркивание {: #strikethrough }

В GFM (GitHub Flavored Markdown) можно зачёркивать текст.

```markdown
~~Strike through this text.~~
```

Что рендерится так:

~~Strike through this text.~~

HTML:

```html
<del>Strike through this text.</del>
```

### Подсветка {: #highlight }

> [!NOTE] Примечание
> Подсветка, нижний и верхний индексы — **новое в Grav 2.0**, включено по умолчанию.

Заключите текст в двойные знаки равенства, чтобы пометить его как подсвеченный; он рендерится как HTML-элемент `<mark>`.

```markdown
Markdown makes ==certain words stand out==.
```

Рендерится так:

Markdown makes ==certain words stand out==.

HTML:

```html
<p>Markdown makes <mark>certain words stand out</mark>.</p>
```

### Нижний индекс {: #subscript }

Заключите текст в одиночные тильды для нижнего индекса; он рендерится как элемент `<sub>`. Одиночное `~text~` — нижний индекс, а двойное `~~text~~` остаётся [зачёркиванием](#strikethrough), поэтому эти возможности никогда не конфликтуют.

```markdown
The chemical formula for water is H~2~O.
```

Рендерится так:

The chemical formula for water is H~2~O.

HTML:

```html
<p>The chemical formula for water is H<sub>2</sub>O.</p>
```

### Верхний индекс {: #superscript }

Заключите текст в одиночные символы каретки для верхнего индекса; он рендерится как элемент `<sup>`.

```markdown
Einstein's famous equation is E = mc^2^.
```

Рендерится так:

Einstein's famous equation is E = mc^2^.

HTML:

```html
<p>Einstein's famous equation is E = mc<sup>2</sup>.</p>
```

## Цитаты

Для цитирования блоков контента из другого источника в документе.

Добавьте `>` перед текстом, который хотите процитировать.

```markdown
> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.
```

Рендерится так:

> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.

и такой HTML:

```html
<blockquote>
  <p><strong>Fusion Drive</strong> combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.</p>
</blockquote>
```

Цитаты можно вкладывать друг в друга:

```markdown
> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.
```

Рендерится так:

> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.

## Уведомления

> [!WARNING] Предупреждение
> Старый механизм уведомлений, переопределявший синтаксис цитат (`>>>`), устарел. Теперь уведомления обрабатываются специальным плагином [Markdown Notices](https://github.com/getgrav/grav-plugin-markdown-notices).

## Списки

### Неупорядоченные

Список элементов, порядок которых явно не важен.

Для маркеров каждого элемента можно использовать любой из следующих символов:

```markdown
* valid bullet
- valid bullet
+ valid bullet
```

Например:

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

Рендерится так:

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

И такой HTML:

```html
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

### Упорядоченные

Список элементов, порядок которых явно важен.

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

Рендерится так:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

И такой HTML:

```html
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

**Совет**: если использовать `1.` для каждого пункта, Markdown автоматически пронумерует элементы. Например:

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

Рендерится так:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

### Списки задач {: #task-lists }

> [!NOTE] Примечание
> Списки задач — **новое в Grav 2.0**, включены по умолчанию.

Добавьте `[ ]` (не выполнено) или `[x]` (выполнено) сразу после маркера списка, чтобы элемент отобразился как флажок, согласно GitHub Flavored Markdown.

```markdown
- [x] Write the documentation
- [x] Review the examples
- [ ] Ship it
```

Рендерится так:

- [x] Write the documentation
- [x] Review the examples
- [ ] Ship it

HTML:

```html
<ul>
  <li class="task-list-item"><input type="checkbox" disabled="" checked="" /> Write the documentation</li>
  <li class="task-list-item"><input type="checkbox" disabled="" checked="" /> Review the examples</li>
  <li class="task-list-item"><input type="checkbox" disabled="" /> Ship it</li>
</ul>
```

Флажки отображаются только для чтения (`disabled`), а каждый `<li>` получает класс `task-list-item`, чтобы списки задач можно было оформить в теме.

## Код

### Встроенный код

Заключайте встроенные фрагменты кода в обратные кавычки `` ` ``.

```markdown
In this example, I50 should be wrapped as **code**.
```

Рендерится так:

In this example, `<section></section>` should be wrapped with **code**.

HTML:

```html
<p>In this example, <code>&lt;section&gt;&lt;/section&gt;</code> should be wrapped with <strong>code</strong>.</p>
```

### Код с отступом

Либо сделайте отступ в несколько строк кода минимум в четыре пробела, как здесь:

<pre>
  // Комментарии
  line 1 of code
  line 2 of code
  line 3 of code
</pre>

Рендерится так:

```text
// Комментарии
line 1 of code
line 2 of code
line 3 of code
```

HTML:

```html
<pre>
  <code>
    // Комментарии
    line 1 of code
    line 2 of code
    line 3 of code
  </code>
</pre>
```

### Блоки кода «fences»

Используйте «fences» ```` ``` ````, чтобы оградить несколько строк кода и указать язык.

<pre>
```
Sample text here...
```
</pre>

HTML:

```html
<pre language-html>
  <code>Sample text here...</code>
</pre>
```

### Подсветка синтаксиса

GFM, или «GitHub Flavored Markdown», также поддерживает подсветку синтаксиса. Чтобы включить её, просто добавьте расширение нужного языка сразу после первого «забора» кода — ` ```js `, — и подсветка синтаксиса автоматически применится в отрендеренном HTML. Например, чтобы включить подсветку для JavaScript:

<pre>
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
</pre>

Рендерится так:

```javascript
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

> [!NOTE] Примечание
> Чтобы подсветка синтаксиса работала, нужно установить и включить [плагин Highlight](https://github.com/getgrav/grav-plugin-highlight). Он, в свою очередь, использует jQuery-плагин, поэтому jQuery также должен быть загружен в вашей теме.

## Таблицы

Таблицы создаются добавлением вертикальных черт как разделителей между ячейками и строки дефисов (тоже разделённых чертами) под заголовком. Обратите внимание: вертикальные черты не нужно выравнивать.

```markdown
| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

Рендерится так:

| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

И такой HTML:

```html
<table>
  <thead>
    <tr>
      <th>Option</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>data</td>
      <td>path to data files to supply the data that will be passed into templates.</td>
    </tr>
    <tr>
      <td>engine</td>
      <td>engine to be used for processing templates. Handlebars is the default.</td>
    </tr>
    <tr>
      <td>ext</td>
      <td>extension to be used for dest files.</td>
    </tr>
  </tbody>
</table>
```

### Выравнивание по правому краю

Добавление двоеточия справа от дефисов под заголовком выровняет текст в этом столбце по правому краю.

```markdown
| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

### Расширенные таблицы {: #enhanced-tables }

Grav 2.0 добавляет пять расширений таблиц, **включаемых по желанию**. Все они **по умолчанию выключены**, поэтому стандартная таблица никогда не меняется, пока вы не включите расширение — глобально в разделе [Конфигурация](#configuration) или для отдельной страницы в её метаданных:

```yaml
markdown:
  tables:
    colspan: true
    headerless: true
    captions: true
    attributes: true
    multiline: true
```

#### Colspan

Оставьте ячейку пустой, и она объединится с ячейкой слева, увеличив её `colspan` (соглашение MultiMarkdown).

```markdown
| Item    | Jan | Feb | Mar |
| :--     | --: | --: | --: |
| Widgets | 10  | 12  | 9   |
| Total for the quarter |   |   | 57 |
```

Метка последней строки занимает первые три столбца:

```html
<tr>
  <td style="text-align: left;" colspan="3">Total for the quarter</td>
  <td style="text-align: right;">57</td>
</tr>
```

#### Без заголовка

Начните таблицу сразу со строки-разделителя, чтобы получить таблицу только с `<tbody>` и без заголовка.

```markdown
| --- | --- |
| Mercury | Closest to the Sun |
| Neptune | Furthest from the Sun |
```

```html
<table>
  <tbody>
    <tr><td>Mercury</td><td>Closest to the Sun</td></tr>
    <tr><td>Neptune</td><td>Furthest from the Sun</td></tr>
  </tbody>
</table>
```

#### Подписи

Строка `[Caption]` сразу после таблицы становится `<caption>` (рендерится первым дочерним элементом таблицы). Встроенный Markdown внутри подписи обрабатывается.

```markdown
| Symbol | Element |
| --- | --- |
| H | Hydrogen |
| He | Helium |
[The first two elements of the periodic table]
```

```html
<table>
  <caption>The first two elements of the periodic table</caption>
  <thead>...</thead>
  <tbody>...</tbody>
</table>
```

#### Атрибуты

Строка `{.class #id}` сразу после таблицы задаёт `class` и `id` для элемента `<table>` — удобно для CSS-фреймворков, которые оформляют классы вроде `.striped` или `.responsive`. Также принимается форма в стиле kramdown `{:.class}` (с ведущим двоеточием). Распознаются только `class` и `id`; при любом другом токене строка остаётся без изменений.

```markdown
| Name | Role |
| --- | --- |
| Ada | Engineer |
| Grace | Architect |
{.striped .responsive #team-table}
```

```html
<table class="striped responsive" id="team-table">
  ...
</table>
```

#### Многострочные ячейки

Завершите строку обратной косой чертой (`\`), и следующая строка продолжит её: ячейки следующей строки объединяются по столбцам с предыдущей строкой, каждая через `<br>`. Пустая ячейка продолжения ничего не добавляет, поэтому на следующую строку можно перенести только один столбец.

```markdown
| Feature | Description |
| --- | --- |
| Tables | Alignment, escaped pipes, \
| | and inline cell content |
| Lists | Ordered, unordered and task lists |
```

```html
<tr>
  <td>Tables</td>
  <td>Alignment, escaped pipes,<br>and inline cell content</td>
</tr>
```

## Ссылки

### Простая ссылка

```markdown
[Assemble](https://assemble.io)
```

Рендерится так (наведите на ссылку — подсказки нет):

[Assemble](https://assemble.io)

HTML:

```html
<a href="https://assemble.io">Assemble</a>
```

### Добавление заголовка

```markdown
[Upstage](https://github.com/upstage/ "Visit Upstage!")
```

Рендерится так (наведите на ссылку — должна появиться подсказка):

[Upstage](https://github.com/upstage/ "Visit Upstage!")

HTML:

```html
<a href="https://github.com/upstage/" title="Visit Upstage!">Upstage</a>
```

### Именованные якоря

Именованные якоря позволяют перейти к указанной точке на той же странице. Например, каждая из этих глав:

```markdown
# Table of Contents
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)
```

будет вести к этим разделам:

```markdown
## Chapter 1 <a id="chapter-1"></a>
Content for chapter one.

## Chapter 2 <a id="chapter-2"></a>
Content for chapter one.

## Chapter 3 <a id="chapter-3"></a>
Content for chapter one.
```

**Обратите внимание**, что конкретное расположение тега якоря кажется произвольным. Здесь они размещены в строке, поскольку так они выглядят ненавязчиво, и это работает.

### Автоматические ссылки {: #automatic-links }

> [!NOTE] Примечание
> Автоматическое связывание голых URL `www.` и адресов email — **новое в Grav 2.0** (автоссылки GitHub Flavored Markdown), включено по умолчанию. Голые URL `http(s)://` обрабатываются отдельно параметром **Auto URL Links**, который по умолчанию остаётся выключенным.

Голые URL `www.` и адреса email автоматически превращаются в ссылки без какой-либо разметки:

```markdown
Visit www.getgrav.org or email hello@getgrav.org.
```

Рендерится так:

Visit www.getgrav.org or email hello@getgrav.org.

HTML:

```html
<p>Visit <a href="http://www.getgrav.org">www.getgrav.org</a> or email <a href="mailto:hello@getgrav.org">hello@getgrav.org</a>.</p>
```

## Изображения

Изображения имеют синтаксис, похожий на ссылки, но с восклицательным знаком в начале.

```markdown
![Minion](https://octodex.github.com/images/minion.png)
```

![Minion](https://octodex.github.com/images/minion.png)

или:

```markdown
![Alt text](https://octodex.github.com/images/stormtroopocat.jpg "The Stormtroopocat")
```

![Alt text](https://octodex.github.com/images/stormtroopocat.jpg "The Stormtroopocat")

Как и у ссылок, у изображений есть синтаксис в стиле сносок:

```markdown
![Alt text][id]
```

![Alt text][id]

С определением расположения URL позже в документе:

[id]: https://octodex.github.com/images/dojocat.jpg  "The Dojocat"

    [id]: https://octodex.github.com/images/dojocat.jpg  "The Dojocat"

## Конфигурация {: #configuration }

Рендеринг Markdown настраивается в разделе **Конфигурация → Система → Markdown** админки или в `user/config/system.yaml` по ключу `pages.markdown`. Любой параметр также можно переопределить для отдельной страницы в её метаданных по ключу `markdown:`. Значения по умолчанию:

```yaml
pages:
  markdown:
    extra: false              # Включить Markdown Extra
    auto_line_breaks: false   # Считать одиночный перевод строки <br>
    auto_url_links: false     # Автоматически связывать URL http(s)://
    escape_markup: false      # Экранировать встроенный HTML в сущности
    gfm:                      # GitHub Flavored Markdown — включён по умолчанию
      task_lists: true        # флажки - [ ] / - [x]
      marks: true             # ==подсветка==, ~нижний индекс~, ^верхний индекс^
      tagfilter: true         # Экранировать запрещённые необработанные HTML-теги
      autolinks: true         # Автоматически связывать голые URL www. и email
    tables:                   # Расширения таблиц — по умолчанию выключены
      colspan: false          # Пустая ячейка объединяется с ячейкой слева
      headerless: false       # Таблица может начинаться со строки-разделителя
      captions: false         # Строка [Caption] после таблицы становится <caption>
      attributes: false       # Строка {.class #id} после таблицы задаёт атрибуты
      multiline: false        # Строка, оканчивающаяся на \, продолжается на следующей
```

Параметры `gfm` меняют то, как рендерится существующий контент (флажки, `<mark>`/`<sub>`/`<sup>`, автоссылки URL, экранирование необработанного HTML), поэтому каждый из них можно отключить отдельно, если нужно прежнее поведение. Расширения `tables` все выключены по умолчанию и никогда не влияют на стандартную таблицу, пока вы их не включите.

> [!TIP] Совет
> Разработчики плагинов могут добавлять собственный синтаксис Markdown — настраиваемые блоки и встроенные метки — через [API расширений Markdown](../../plugins/markdown-extensions/index.md).

## Twig в контенте

**Twig** также можно обрабатывать внутри Markdown-контента страницы. В Grav 2.0 эта фича **по умолчанию выключена** и, когда включена, выполняется в песочнице безопасности — изменение по сравнению с предыдущими версиями, где `process.twig: true` на странице запускал Twig в контенте без ограничений. О том, как это работает, как включить и какие есть рекомендуемые альтернативы ([шаблоны страниц](../twig-in-content/index.md#use-a-page-template) и [шорткоды](../creating-a-shortcode/index.md)), см. [Twig в контенте](../twig-in-content/index.md).
