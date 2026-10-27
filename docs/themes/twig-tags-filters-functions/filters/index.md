---
title: "Фильтры Twig"
description: "Фильтры Twig применяются к переменным Twig с помощью символа `|` и имени фильтра. Параметры передаются так же, как в функциях Twig, в скобках."
tags:
  - "Twig"
---

# Фильтры Twig

Фильтры Twig применяются к переменным Twig с помощью символа `|` и имени фильтра. Параметры передаются так же, как в функциях Twig, в скобках.

### absolute_url {: #absolute-url }

Принимает HTML-фрагмент с атрибутом `src` или `href`, содержащим относительный путь. Преобразует строку пути в абсолютный URL, включая имя хоста.

```twig title="Twig"
{{ '<img src="/some/path/to/image.jpg" />'|absolute_url }}
```

```html title="Вывод"
<img src="https://learn.getgrav.org/some/path/to/image.jpg">
```

### array_group_by {: #array-group-by }

Группирует элементы массива или коллекции по указанному свойству или callback-функции. Особенно полезен для организации записей блога, товаров или любых коллекций по общим атрибутам: дате, категории, автору или пользовательским критериям.

**Базовое использование с именем свойства:**

`collection|array_group_by('category')` :fontawesome-solid-arrow-right-long: Группирует элементы по их свойству 'category'

**Пример: группировка по категории таксономии**

```twig
{# Подготовка коллекции с категорией #}
{% set collection_with_category = [] %}
{% for post in page.collection() %}
    {% set category = post.taxonomy.category|first ?: 'uncategorized' %}
    {% set collection_with_category = collection_with_category|merge([{
        post: post,
        category: category
    }]) %}
{% endfor %}

{# Группировка по категории #}
{% set posts_by_category = collection_with_category|array_group_by('category') %}

{# Вывод сгруппированных записей #}
{% for category, posts in posts_by_category %}
    <h3>{{ category|capitalize }}</h3>
    {% for item in posts %}
        <div>{{ item.post.title }}</div>
    {% endfor %}
{% endfor %}
```

### array_unique {: #array-unique }

Обёртка для PHP `array_unique()`, удаляющая дубликаты из массива.

```twig title="Twig"
{{ ['foo', 'bar', 'foo', 'baz']|array_unique }}
```

```text title="Вывод"
['foo', 'bar', 'baz']
```

### base32_encode {: #base32-encode }

Выполняет кодирование base32 переменной.

```twig title="Twig"
{{ 'some variable here'|base32_encode }}
```

```text title="Вывод"
ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK
```

### base32_decode {: #base32-decode }

Выполняет декодирование base32 переменной.

```twig title="Twig"
{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK'|base32_decode }}
```

```text title="Вывод"
some variable here
```

### base64_encode {: #base64-encode }

Выполняет кодирование base64 переменной.

```twig title="Twig"
{{ 'some variable here'|base64_encode }}
```

```text title="Вывод"
c29tZSB2YXJpYWJsZSBoZXJl
```

### base64_decode {: #base64-decode }

Выполняет декодирование base64 переменной.

```twig title="Twig"
{{ 'c29tZSB2YXJpYWJsZSBoZXJl'|base64_decode }}
```

```text title="Вывод"
some variable here
```

### basename

Возвращает базовое имя пути.

```twig title="Twig"
{{ '/etc/sudoers.d'|basename }}
```

```text title="Вывод"
sudoers.d
```

### camelize

Преобразует строку в формат «CamelCase».

```twig title="Twig"
{{ 'send_email'|camelize }}
```

```text title="Вывод"
SendEmail
```

### chunk_split {: #chunk-split }

Разбивает строку на меньшие части заданного размера.

```twig title="Twig"
{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGKA'|chunk_split(6, '-') }}
```

```text title="Вывод"
ONXW2Z-JAOZQX-E2LBMJ-WGKIDI-MVZGKA-
```

### contains

Определяет, содержит ли одна строка другую.

```twig title="Twig"
{{ 'some string with things in it'|contains('things') }}
```

```text title="Вывод"
true
```

#### Приведение значений

В PHP 7 проверка типов стала строже, поэтому передача значения неправильного типа теперь может выбросить исключение. Чтобы избежать этого, используйте фильтры, гарантирующие допустимость значения, передаваемого методу:

### string

Используйте `|string`, чтобы привести значение к строке.

### int

Используйте `|int`, чтобы привести значение к целому числу.

### bool

Используйте `|bool`, чтобы привести значение к логическому типу.

### float

Используйте `|float`, чтобы привести значение к числу с плавающей точкой.

### array

Используйте `|array`, чтобы привести значение к массиву.

### defined

Иногда нужно проверить, определена ли переменная, и если нет — задать значение по умолчанию. Например:

```twig
set header_image_width = page.header.header_image_width|defined(900)
```

Это задаст переменной `header_image_width` значение `900`, если она не определена в заголовке страницы.

### dirname

Возвращает имя папки из пути.

```twig title="Twig"
{{ '/etc/sudoers.d'|dirname }}
```

```text title="Вывод"
/etc
```

### ends_with {: #ends-with }

Принимает искомое значение и строку, в которой ищут, и определяет, заканчивается ли эта строка искомым значением. Теперь также работает с массивом искомых значений и вернёт `true`, если строка заканчивается **любым** из них.

```twig title="Twig"
{{ 'the quick brown fox'|ends_with('fox') }}
```

```text title="Вывод"
true
```

### fieldName

Фильтрует имя поля, преобразуя точечную нотацию в нотацию массива.

```twig title="Twig"
{{ 'field.name'|fieldName }}
```

```text title="Вывод"
field[name]
```

### get_type {: #get-type }

Возвращает тип переменной:

```twig title="Twig"
{{ page|get_type }}
```

```text title="Вывод"
Grav\Common\Page\Page
```

### html_to_markdown {: #html-to-markdown }

Преобразует фрагмент отрендеренного HTML в Markdown — то же преобразование, которое Grav использует для [Markdown-вывода](../../../content/markdown-output/index.md) страницы: заголовки ATX, блоки кода с указанием языка, таблицы, абсолютные URL и без форм, скриптов и встроенного SVG. **Grav 2.1+**

```twig title="Twig"
{{ '<h2>Install</h2><p>Run <code>composer install</code> first.</p>'|html_to_markdown }}
```

```markdown title="Вывод"
## Install

Run I13 first.
```

### humanize

Преобразует строку в более «человекочитаемый» формат.

```twig title="Twig"
{{ 'something_text_to_read'|humanize }}
```

```text title="Вывод"
Something text to read
```

### hyphenize

Преобразует строку в версию через дефисы.

```twig title="Twig"
{{ 'Something Text to Read'|hyphenize }}
```

```text title="Вывод"
something-text-to-read
```

### json_decode {: #json-decode }

JSON можно декодировать, просто применив этот фильтр:

```twig title="Twig"
{% set array = '{"first_name": "Guido", "last_name":"Rossum"}'|json_decode %}
{{ print_r(array) }}
```

```text title="Вывод"
[
    "first_name" => "Guido"
    "last_name" => "Rossum"
]
```

### ksort

Сортирует карту массива по каждому ключу.

```twig title="Twig"
{% set items = {'orange':1, 'apple':2, 'peach':3}|ksort %}
{{ print_r(items) }}
```

```text title="Вывод"
[
    "apple" => 2
    "orange" => 1
    "peach" => 3
]
```

### ltrim

Удаляет пробелы в начале строки. Также может удалять другие символы, если задать маску символов (см. [https://php.net/manual/en/function.ltrim.php](https://php.net/manual/en/function.ltrim.php)).

```twig title="Twig"
{{ '/strip/leading/slash/'|ltrim('/') }}
```

```text title="Вывод"
strip/leading/slash/
```

### markdown

Принимает произвольную строку с Markdown и преобразует её в HTML с помощью парсера Markdown Grav. Необязательный параметр `boolean`:

* `true` (по умолчанию): обрабатывать как блок (текстовый режим, контент будет обёрнут в теги `<p>`)
* `false`: обрабатывать как строку (контент не будет обёрнут)

```
string|markdown($is_block)
```

```twig title="Twig"
<div class="div">
{{ 'A paragraph with **markdown** and [a link](http://www.cnn.com)'|markdown }}
</div>

<p class="paragraph">{{'A line with **markdown** and [a link](http://www.cnn.com)'|markdown(false) }}</p>
```

```html title="Вывод"
<div class="div">
<p>A paragraph with <strong>markdown</strong> and <a href="http://www.cnn.com">a link</a></p>
</div>

<p class="paragraph">A line with <strong>markdown</strong> and <a href="http://www.cnn.com">a link</a></p>
```

### md5

Создаёт md5-хеш строки.

```twig title="Twig"
{{ 'anything'|md5 }}
```

```text title="Вывод"
f0e166dc34d14d6c228ffac576c9a43c
```

### modulus

Работает так же, как символ `%` в PHP. Применяется к числу с числовым делителем и необязательным массивом элементов для выбора.

```twig title="Twig"
{{ 7|modulus(3, ['red', 'blue', 'green']) }}
```

```text title="Вывод"
blue
```

### monthize

Преобразует целое число дней в количество месяцев.

```twig title="Twig"
{{ '181'|monthize }}
```

```text title="Вывод"
6
```

### nicecron

Возвращает человекочитаемый вывод для синтаксиса cron.

```twig title="Twig"
{{ "2 * * * *"|nicecron }}
```

```text title="Вывод"
At 2 minutes past the hour
```

### nicefilesize

Выводит размер файла в удобном человекочитаемом формате.

```twig title="Twig"
{{ 612394|nicefilesize }}
```

```text title="Вывод"
598.04 KB
```

### nicenumber

Выводит число в удобном человекочитаемом формате.

```twig title="Twig"
{{ 12430|nicenumber }}
```

```text title="Вывод"
12K
```

### nicetime

Выводит дату в удобном человекочитаемом формате времени.

```twig title="Twig"
{{ page.date|nicetime(false) }}
```

```text title="Вывод"
1 month ago
```

Первый аргумент указывает, использовать ли полное описание даты. По умолчанию `true`.

Вторым аргументом можно передать `false`, если нужно убрать относительный временной дескриптор (например, «ago» или «from now» в вашем языке) из результата.

### of_type {: #of-type }

Проверяет тип переменной по параметру:

```twig title="Twig"
{{ page|of_type('string') }}
```

```text title="Вывод"
false
```

### ordinalize

Добавляет порядковый суффикс к целому числу (например, 1st, 2nd, 3rd, 4th).

```twig title="Twig"
{{ '10'|ordinalize }}
```

```text title="Вывод"
10th
```

### pad

Дополняет строку до определённой длины другим символом. Это обёртка для функции PHP [str_pad()](https://php.net/manual/en/function.str-pad.php).

```twig title="Twig"
{{ 'foobar'|pad(10, '-') }}
```

```text title="Вывод"
foobar----
```

### pluralize

Преобразует строку в английскую форму множественного числа.

```twig title="Twig"
{{ 'person'|pluralize }}
```

```text title="Вывод"
people
```

`pluralize` также принимает необязательный числовой параметр, который можно передать, когда заранее неизвестно, к скольким предметам будет относиться существительное. По умолчанию он равен 2, поэтому при отсутствии параметра будет использована форма множественного числа. Например:

```twig
<p>We have {{ num_vacancies }} {{ 'vacancy'|pluralize(num_vacancies) }} right now.</p>
```

### print_r {: #print-r }

Выводит человекочитаемую информацию о переменной.

```twig
page.header|print_r
```

### randomize

Перемешивает предоставленный список. Если передать значение параметром, оно пропустит первые n значений и сохранит их порядок.

```twig title="Twig"
{% set ritems = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten']|randomize(2) %}
{{ print_r(ritems) }}
```

```text title="Вывод"
['one', 'two', 'eight', 'four', 'ten', 'seven', 'nine', 'three', 'six', 'five']
```

Примечание: первые два элемента ('one', 'two') остаются в прежнем порядке, остальные перемешиваются.

### regex_replace {: #regex-replace }

Полезная обёртка для метода PHP [preg_replace()](https://php.net/manual/en/function.preg-replace.php), позволяющая выполнять сложные замены регулярными выражениями в тексте через этот фильтр:

```twig title="Twig"
{{ 'The quick brown fox jumps over the lazy dog.'|regex_replace(['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) }}
```

```text title="Вывод"
The slow black bear jumps over the lazy turtle.
```

> [!WARNING] Предупреждение
> По возможности используйте разделитель `~` вместо `/`. Иначе, скорее всего, придётся [дважды экранировать некоторые символы](https://github.com/getgrav/grav/issues/833). Например, `~\/\#.*~` вместо `'/\\/\\#.*/'`, что ближе к [синтаксису PCRE](https://www.php.net/manual/en/regexp.reference.delimiters.php), используемому в PHP.

### rtrim

Удаляет пробелы в конце строки. Также может удалять другие символы, если задать маску символов (см. [https://php.net/manual/en/function.rtrim.php](https://php.net/manual/en/function.rtrim.php)).

```twig title="Twig"
{{ '/strip/trailing/slash/'|rtrim('/') }}
```

```text title="Вывод"
/strip/trailing/slash
```

### singularize

Преобразует строку в английскую форму единственного числа.

```twig title="Twig"
{{ 'shoes'|singularize }}
```

```text title="Вывод"
shoe
```

### safe_email {: #safe-email }

Фильтр безопасного email преобразует адрес электронной почты в ASCII-символы, чтобы спам-ботам было сложнее распознать и собрать его.

```twig title="Twig"
{{ "someone@domain.com"|safe_email }}
```

```text title="Вывод"
&#115;&#111;&#109;&#101;&#111;&#110;&#101;&#64;&#100;&#111;&#109;&#97;&#105;&#110;&#46;&#99;&#111;&#109;
```

Пример использования со ссылкой mailto:

```html
<a href="mailto:{{ 'your.email@server.com'|safe_email }}">
  Email me
</a>
```

Сначала вы можете не заметить разницы, но если посмотреть исходный код страницы (не через инструменты разработчика браузера, а именно исходный код страницы), вы увидите лежащую в основе кодировку символов.

### sort_by_key {: #sort-by-key }

Сортирует карту массива по определённому ключу.

```twig title="Twig"
{% set people = [{'email':'fred@yahoo.com', 'id':34}, {'email':'tim@exchange.com', 'id':21}, {'email':'john@apple.com', 'id':2}]|sort_by_key('id') %}
{% for person in people %}{{ person.email }}:{{ person.id }}, {% endfor %}
```

```text title="Вывод"
john@apple.com:2, tim@exchange.com:21, fred@yahoo.com:34,
```

### starts_with {: #starts-with }

Принимает искомое значение и строку, в которой ищут, и определяет, начинается ли эта строка с искомого значения. Теперь также работает с массивом искомых значений и вернёт `true`, если строка начинается с **любого** из них.

```twig title="Twig"
{{ 'the quick brown fox'|starts_with('the') }}
```

```text title="Вывод"
true
```

### titleize

Преобразует строку в формат «Title Case».

```twig title="Twig"
{{ 'welcome page'|titleize }}
```

```text title="Вывод"
Welcome Page
```

### t

Переводит строку на текущий язык.

```twig title="Twig"
{{ 'MY_LANGUAGE_KEY_STRING'|t }}
```

```text title="Вывод"
Some Text in English
```

Предполагается, что эти языковые строки переведены на сайте и включена поддержка нескольких языков. Подробнее см. в [документации по многоязычности](../../../content/multi-language/index.md).

### tu

Переводит строку на текущий язык, заданный в настройках пользователя в админке.

```twig title="Twig"
{{ 'MY_LANGUAGE_KEY_STRING'|tu }}
```

```text title="Вывод"
Some Text in English
```

Используется поле языка, заданное в YAML пользователя.

### ta

Переводит массив с языком с помощью фильтра `|ta`. Подробный пример см. в [документации по многоязычности](../../../content/multi-language/index.md).

```twig title="Twig"
{{ 'MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1) }}
```

```text title="Вывод"
December
```

### tl

Переводит строку на конкретный язык. Подробнее см. в [документации по многоязычности](../../../content/multi-language/index.md#complex-translations).

```twig
'SIMPLE_TEXT'|tl(['fr'])
```

### truncate

С помощью этого фильтра легко создать сокращённую, усечённую версию строки. Единственный обязательный параметр — количество символов, но есть и другие опции:

```twig title="Twig"
'one not so long sentence. two sentences'|truncate(5)|raw
```

```text title="Вывод"
one n&hellip;
```

Просто обрезает до 5 символов.

```twig title="Twig"
'one not so long sentence. two sentences'|truncate(5, true)|raw
```

```text title="Вывод"
one not&hellip;
```

Если второй аргумент равен `true`, обрезка происходит по первому символу разрыва на указанной длине или после неё. Символ разрыва по умолчанию — пробел, поэтому пример заканчивается после `one not`, а не в конце предложения.

Третий аргумент задаёт другой символ разрыва. Например, `truncate(5, true, '.')` возвращает `one not so long sentence&hellip;` для того же ввода. Точка не включается в усечённый результат; эта опция не определяет границы предложений.

Если на указанной длине или после неё нет символа разрыва, строка обрезается по лимиту. `'abcdefgh'|truncate(5, true)` вернёт `abcde&hellip;`, поскольку пробела для разрыва нет.

> [!CAUTION] Внимание
> Фильтр Twig `|raw` следует использовать с элементом заполнения `&hellip;` (многоточие) по умолчанию, чтобы он рендерился при включённом автоэкранировании Twig.

Также можно усекать HTML-текст, но сначала следует применить фильтр `|striptags`, чтобы удалить HTML-разметку, которая может сломаться при обрезке между тегами:

```twig title="Twig"
'<span>one <strong>sentence</strong>. two sentences</span>'|raw|striptags|truncate(25)
```

```text title="Вывод"
one sentence. two sentenc&hellip;
```

#### Специализированные версии:

### safe_truncate {: #safe-truncate }

Используйте `|safe_truncate`, чтобы усекать текст по количеству символов «без разрыва слов». Он никогда не разрывает слово, поэтому результат может выйти за лимит до конца слова, в котором оказался: `'alpha bravo charlie'|safe_truncate(3)` вернёт `alpha&hellip;`.

### truncate_html {: #truncate-html }

Используйте `|truncate_html`, чтобы усекать HTML по количеству символов, сохраняя разметку. Он не «безопасен для слов», поэтому может разрезать слово. По умолчанию — 100 символов.

```twig title="Twig"
'<p>alpha bravo charlie delta echo</p>'|truncate_html(3)
```

```text title="Вывод"
<p>alp...</p>
```

### safe_truncate_html {: #safe-truncate-html }

Используйте `|safe_truncate_html`, чтобы усекать HTML по количеству **слов**, сохраняя разметку. По умолчанию — 25 слов. Обратите внимание: здесь считаются слова, тогда как `truncate_html` считает символы.

```twig title="Twig"
'<p>alpha bravo charlie delta echo</p>'|safe_truncate_html(3)
```

```text title="Вывод"
<p>alpha bravo charlie...</p>
```

### underscorize

Преобразует строку в формат «under_scored».

```twig title="Twig"
{{ 'CamelCased'|underscorize }}
```

```text title="Вывод"
camel_cased
```

### wordcount

Считает количество слов в текстовой строке с поддержкой нескольких языков и повышенной точностью для HTML-контента.

```twig title="Twig"
{{ page.content|wordcount }}
```

```text title="Вывод"
36
```

Фильтр `wordcount` также принимает необязательный параметр локали для корректной обработки разных языков. Для западных языков (английский, испанский, французский и т. д.) он считает отдельные слова, разделённые пробелами. Для азиатских языков, таких как китайский, японский и корейский, он считает символы вместо слов, что больше подходит для этих систем письма.

```twig
{# С конкретной локалью для английского контента #}
{{ page.content|wordcount('en') }}

{# Для китайского контента — считает символы вместо слов #}
{{ page.content|wordcount('zh') }}

{# Использование в структурированных данных JSON-LD #}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "wordCount": page.content|wordcount,
  "headline": page.title
}
</script>
```

> [!CAUTION] Внимание
> **Поддерживаемые локали:** `en` (английский, по умолчанию), `es` (испанский), `fr` (французский), `de` (немецкий) и другие западные языки используют подсчёт по словам. `zh`/`zh-cn`/`zh-tw`/`chinese` (китайский), `ja`/`japanese` (японский) и `ko`/`korean` (корейский) используют подсчёт по символам.

### yaml_encode {: #yaml-encode }

Сериализует/кодирует переменную в синтаксис YAML.

```twig title="Twig"
{% set array = {foo: [0, 1, 2, 3], baz: 'qux' } %}
{{ array|yaml_encode }}
```

```yaml title="Вывод"
foo:
    - 0
    - 1
    - 2
    - 3
baz: qux
```

### yaml_decode {: #yaml-decode }

Декодирует/разбирает переменную из синтаксиса YAML.

```twig title="Twig"
{% set yaml = "foo: [0, 1, 2, 3]\nbaz: qux" %}
{{ yaml|yaml_decode|var_dump }}
```

```text title="Вывод"
array(2) {
  ["foo"]=> array(4) { [0]=> int(0) [1]=> int(1) [2]=> int(2) [3]=> int(3) }
  ["baz"]=> string(3) "qux"
}
```
