---
title: Фильтры и функции Twig
page-toc:
  active: true
  depth: 3
process:
    twig: true
taxonomy:
    category: docs
---

Хотя Twig уже предоставляет обширный список [фильтров, функций и тегов](https://twig.sensiolabs.org/documentation), Grav также предоставляет выбор полезных дополнений,чтобы облегчить процесс тематизации.

!! Для получения информации о разработке собственных фильтров Twig, ознакомьтесь с примером [Свои фильтр/функция Twig](/cookbook/twig-recipes/#custom-twig-filter-function) в разделе **Рецепты Twig** в главе **Поваренная книга**.


## Twig-фильтры Grav

Фильтры Twig применяются к переменным Twig с помощью символа `|`, за которым следует имя фильтра. Параметры можно передавать так же, как функции Twig, используя круглые скобки.

#### Абсолютный URL

Возьмите относительный путь и преобразуйте его в абсолютный формат URL, включая имя хоста

`'<img src="/some/path/to/image.jpg" />'|absolute_url` <i class="fa fa-long-arrow-right"></i> `{{ '<img src="/some/path/to/image.jpg" />'|absolute_url }}`

#### Array Unique

Оболочка для PHP 'array_unique()', которая удаляет дубликаты из массива.

`['foo', 'bar', 'foo', 'baz']|array_unique` <i class="fa fa-long-arrow-right"></i> **{{ print_r(['foo', 'bar', 'foo', 'baz']|array_unique) }}**

#### Base32 Encode

Выполняет кодировку base32 для переменной
`'some variable here'|base32_encode` <i class="fa fa-long-arrow-right"></i> **{{ 'some variable here'|base32_encode }}**

#### Base32 Decode

Выполняет декодирование base32 на переменной
`'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK'|base32_decode` <i class="fa fa-long-arrow-right"></i> **{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK'|base32_decode }}**

#### Base64 Encode

Выполняет кодировку base64 для переменной
`'some variable here'|base64_encode` <i class="fa fa-long-arrow-right"></i> **{{ 'some variable here'|base64_encode }}**

#### Base64 Decode

Выполняет декодирование base64 на переменной
`'c29tZSB2YXJpYWJsZSBoZXJl'|base64_decode` <i class="fa fa-long-arrow-right"></i> **{{ 'c29tZSB2YXJpYWJsZSBoZXJl'|base64_decode }}**

#### Basename

Возвращает базовое имя пути.

`'/etc/sudoers.d'|basename` <i class="fa fa-long-arrow-right"></i> **{{ '/etc/sudoers.d'|basename }}**

#### Camelize

Преобразует строку в формат "CamelCase"

`'send_email'|camelize` <i class="fa fa-long-arrow-right"></i> **{{ 'send_email'|camelize }}**

[version=16,17]
#### Chunk Split

Разбивает строку на более мелкие куски определенного размера.

`'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGKA'|chunk_split(6, '-')` <i class="fa fa-long-arrow-right"></i> **{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGKA'|chunk_split(6, '-') }}**
[/version]

#### Contains

Определите, содержит ли конкретная строка другую строку

`'some string with things in it'|contains('things')` <i class="fa fa-long-arrow-right"></i> **{{ 'some string with things in it'|contains('things') }}**

#### Преобразование значений

PHP 7 получает более строгие проверки типов, а это означает, что передача значения неправильного типа теперь может вызвать исключение. Чтобы избежать этого, вы должны использовать фильтры, которые гарантируют, что значение, переданное методу, является допустимым:

**|string**

Преобразование значения в строку.

**|int**

Преобразование значения в целое число.

**|bool**

Преобразование значения в логическое.

**|float**

Преобразование значения в число с плавающей запятой.

**|array**

Приведение значения к массиву.

#### Defined

Иногда вы хотите проверить, определена ли какая-либо переменная, а если нет, укажите значение по умолчанию. Например:

`set header_image_width  = page.header.header_image_width|defined(900)`

При этом переменная `header_image_width` будет установлена в значение `900`, если она не определена в заголовке страницы.

#### Dirname

Вернуть имя каталога пути.

`'/etc/sudoers.d'|dirname` <i class="fa fa-long-arrow-right"></i> **{{ '/etc/sudoers.d'|dirname }}**


#### Ends-With

Берет иголку и стог сена и определяет, заканчивается ли стог иглой. Также теперь работает с массивом игл и будет возвращать `true`, если **любой** стог сена заканчивается иглой.

`'the quick brown fox'|ends_with('fox')` <i class="fa fa-long-arrow-right"></i> {{  'the quick brown fox'|ends_with('fox') ? 'true' : 'false' }}

#### FieldName

Фильтрация имени поля путем замены записи через точку на запись массива

`'field.name|fieldName`


[version=16,17]
#### Get Type

Получает тип переменной:

`page|get_type` <i class="fa fa-long-arrow-right"></i> **{{ page|get_type }}**
[/version]

#### Humanize

Преобразует строку в более удобный для чтения формат.

`'something_text_to_read'|humanize` <i class="fa fa-long-arrow-right"></i> **{{ 'something_text_to_read'|humanize }}**

#### Hyphenize

Преобразует строку в версию с дефисом.

`'Something Text to Read'|hyphenize` <i class="fa fa-long-arrow-right"></i> **{{ 'Something Text to Read'|hyphenize }}**

#### JSON Decode

Вы можете декодировать JSON, просто применив этот фильтр:

`array|json_decode` {% verbatim %}
[prism classes="language-twig line-numbers"]
{% set array = '{"first_name": "Guido", "last_name":"Rossum"}'|json_decode %}
{{ print_r(array) }}
[/prism]
{% endverbatim %}

{% set array = '{"first_name": "Guido", "last_name":"Rossum"}'|json_decode %}
[prism classes="language-text"]
{{ print_r(array) }}
[/prism]

#### Ksort

Сортировать карту массива по каждому ключу

`array|ksort` {% verbatim %}
[prism classes="language-twig line-numbers"]
{% set ritems = {'orange':1, 'apple':2, 'peach':3}|ksort %}
{% for key, value in ritems %}{{ key }}:{{ value }}, {% endfor %}
[/prism]
{% endverbatim %}

{% set ritems = {'orange':1, 'apple':2, 'peach':3}|ksort %}
[prism classes="language-twig line-numbers"]
{% for key, value in ritems %}{{ key }}:{{ value }}, {% endfor %}
[/prism]

#### Left Trim

`'/strip/leading/slash/'|ltrim('/')` <i class="fa fa-long-arrow-right"></i> {{ '/strip/leading/slash/'|ltrim('/') }}

Удаляет пробелы в начале строки. Он также может удалить другие символы, установив маску символов (см. [https://php.net/manual/ru/function.ltrim.php](https://php.net/manual/ru/function.ltrim.php)).

#### Markdown

Возьмите произвольную строку, содержащую разметку, и конвертируйте её в HTML, используя парсер Markdown Grav. Необязательный `boolean`-параметр:

* `true` (по умолчанию): обрабатывать как блок (текстовый режим, содержимое будет заключено в теги `<p>`)
* `false`: обрабатывать как строку (содержимое не будет обернуто никакими тегами)

```
string|markdown($is_block)
```

{% verbatim %}
[prism classes="language-twig line-numbers"]
<div class="div">
{{ 'A paragraph with **markdown** and [a link](http://www.cnn.com)'|markdown }}
</div>

<p class="paragraph">{{'A line with **markdown** and [a link](http://www.cnn.com)'|markdown(false) }}</p>
[/prism]
{% endverbatim %}

{% set var %}
<div class="div">
{{ 'A paragraph with **markdown** and [a link](http://www.cnn.com)'|markdown }}
</div>

<p class="paragraph">{{'A line with **markdown** and [a link](http://www.cnn.com)'|markdown(false) }}</p>
{% endset %}
[prism classes="language-text"]
{{ var|e }}
[/prism]

#### MD5

Создает md5 хэш для строки

`'anything'|md5` <i class="fa fa-long-arrow-right"></i> **{{ 'anything'|md5 }}**

#### Modulus

Выполняет ту же функциональность, что и символ Modulus `%` в PHP. Он работает с числом, передавая в числовом делителе и необязательный массив элементов для выбора.

`7|modulus(3, ['red', 'blue', 'green'])` <i class="fa fa-long-arrow-right"></i> **{{ 7|modulus(3, ['red', 'blue', 'green']) }}**

#### Monthize

Преобразовывает целое число дней в число месяцев

`'181'|monthize` <i class="fa fa-long-arrow-right"></i> **{{ '181'|monthize }}**

[version=16,17]
#### NiceCron

Получает читаемый человеком выходной сигнал для синтаксиса cron

`"2 * * * *"|nicecron` <i class="fa fa-long-arrow-right"></i> **{{ '2 * * * *'|nicecron }}**

#### NiceFilesize

Выводите размер файла в удобочитаемом для человека формате.

`612394|nicefilesize` <i class="fa fa-long-arrow-right"></i> **{{ 612394|nicefilesize }}**

#### NiceNumber

Вывод числа в удобочитаемом для человека формате красивых чисел.

`12430|nicenumber` <i class="fa fa-long-arrow-right"></i> **{{ 12430|nicenumber }}**
[/version]

#### NiceTime

Вывод даты в удобочитаемом человеком формате времени.

`page.date|nicetime(false)` <i class="fa fa-long-arrow-right"></i> **{{ page.date|nicetime(false) }}**

[version=16,17]
#### Of Type

Проверяет тип переменной на параметр:

`page|of_type('string')` <i class="fa fa-long-arrow-right"></i> **{{ page|of_type('string') ? 'true' : 'false' }}**
[/version]

#### Ordinalize

Добавляет порядковый номер к целому числу (например, 1-е, 2-е, 3-е, 4-е).

`'10'|ordinalize` <i class="fa fa-long-arrow-right"></i> **{{ '10'|ordinalize }}**

#### Pad

Заполняет строку до определенной длины другим символом. Это обёртка для PHP-функции [str_pad()](https://php.net/manual/ru/function.str-pad.php).

`'foobar'|pad(10, '-')` <i class="fa fa-long-arrow-right"></i> **{{ 'foobar'|pad(10, '-') }}**

#### Pluralize

Преобразовывает строку в английскую множественную версию

`'person'|pluralize` <i class="fa fa-long-arrow-right"></i> **{{ 'person'|pluralize }}**

[version=17]
#### Preg Match

Обертка для PHP [preg_match()](https://www.php.net/manual/ru/function.preg-match.php), которая соответствует тексту по шаблону, заданному в качестве регулярного выражения. Возвращает соответствующие значения, если есть хотя бы одно соответствие или `false`, если нет.
[/version]

[version=17]
#### Preg Split

Обертка для PHP [preg_split()](https://www.php.net/manual/ru/function.preg-split.php), которая расщепляет текст по шаблону, заданному в качестве регулярного выражения.
[/version]

[version=16,17]
#### Print Variable

Печатает читабельную информацию о переменной

`page.header|print_r`

[prism classes="language-text"]
{{ page.header|print_r }}
[/prism]
[/version]

#### Randomize

Рандомизирует предоставленный список.  Если значение задано в качестве параметра, оно пропустит эти значения и сохранит их в порядке.

`array|randomize` {% verbatim %}
[prism classes="language-twig line-numbers"]
{% set ritems = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten']|randomize(2) %}
{% for ritem in ritems %}{{ ritem }}, {% endfor %}
[/prism]
{% endverbatim %}

<strong>
{% set ritems = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten']|randomize(2) %}
{% for ritem in ritems %}{{ ritem }}, {% endfor %}
</strong>

#### Regex Replace

Полезная оболочка для PHP [preg_replace()](https://php.net/manual/ru/function.preg-replace.php) метод, вы можете выполнять сложные замены регулярных выражений в тексте с помощью этого фильтра:

`'The quick brown fox jumps over the lazy dog.'|regex_replace(['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle'])` <i class="fa fa-long-arrow-right"></i> **{{ 'The quick brown fox jumps over the lazy dog.'|regex_replace(['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) }}**

! По возможности используйте разделитель `~` вместо `/`. В противном случае вам, скорее всего, придется [удвоить определенные символы](https://github.com/getgrav/grav/issues/833). Например, `~\/\#.*~~, а не `'/\\/\\#.*/'`, что более соответствует [PCRE-синтаксису](https://www.php.net/manual/ru/regexp.reference.delimiters.php), используемому PHP.

#### Right Trim

`'/strip/trailing/slash/'|rtrim('/')` <i class="fa fa-long-arrow-right"></i> {{ '/strip/trailing/slash/'|rtrim('/') }}

Удаляет пробелы в конце строки. Он также может удалить другие символы, установив маску символов (см. [https://php.net/manual/ru/function.rtrim.php](https://php.net/manual/ru/function.rtrim.php)).

#### Singularize

Преобразует строку в английскую единственную версию

`'shoes'|singularize` <i class="fa fa-long-arrow-right"></i> **{{ 'shoes'|singularize }}**

#### Safe Email

Фильтр безопасной почты преобразует адрес электронной почты в символы ASCII, что затрудняет распознавание и захват спам-ботов.

`"someone@domain.com"|safe_email` <i class="fa fa-long-arrow-right"></i> {{ "someone@domain.com"|safe_email }}

Пример использования со ссылкой mailto:

[prism classes="language-html line-numbers"]
<a href="mailto:{{'your.email@server.com'|safe_email}}">
  Email me
</a>
[/prism]

Сначала вы можете и не заметить разницы, но изучение исходного текста страницы (не используя Инструменты разработчика браузера, фактического исходного текста страницы) выявит кодировку, лежащую в основе символов.

#### Сортировка по ключу

Можно сортировать массив по определенному ключу

`array|sort_by_key` {% verbatim %}
[prism classes="language-twig line-numbers"]
{% set people = [{'email':'fred@yahoo.com', 'id':34}, {'email':'tim@exchange.com', 'id':21}, {'email':'john@apple.com', 'id':2}]|sort_by_key('id') %}
{% for person in people %}{{ person.email }}:{{ person.id }}, {% endfor %}
[/prism]
{% endverbatim %}

<strong>
{% set people = [{'email':'fred@yahoo.com', 'id':34}, {'email':'tim@exchange.com', 'id':21}, {'email':'john@apple.com', 'id':2}]|sort_by_key('id') %}
{% for person in people %}{{ person.email }}:{{ person.id }}, {% endfor %}
</strong>

#### Starts-With

Берет иглу и стог сена и определяет, начинается ли стог сена с иглы. Также теперь работает с массивом игл и вернет `true`, если **ни** стог сена начинается с иглы.

`'the quick brown fox'|starts_with('the')` <i class="fa fa-long-arrow-right"></i> {{  'the quick brown fox'|starts_with('the') ? 'true' : 'false' }}


#### Titleize

Преобразует строку в формат «Заголовок»

`'welcome page'|titleize` <i class="fa fa-long-arrow-right"></i> **{{ 'welcome page'|titleize }}**


#### Translate

Перевести строку на текущий язык

`'MY_LANGUAGE_KEY_STRING'|t` <i class="fa fa-long-arrow-right"></i> 'Some Text in English'

Предполагается, что на вашем сайте переведены эти языковые строки и включена поддержка нескольких языков. Пожалуйста, обратитесь к [многоязычной документации](/content/multi-language) для получения более подробной информации.

#### Translate Admin

Перевести строку на текущий язык, установленный в пользовательских настройках интерфейса администратора.

`'MY_LANGUAGE_KEY_STRING'|tu` <i class="fa fa-long-arrow-right"></i> 'Some Text in English'

При этом используется языковое поле, установленное в пользовательском yaml.

#### Translate Array

Переводит массив на язык с помощью фильтра `|ta`. См. Подробный пример в [многоязычной документации](/content/multi-language).

`'MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1)` <i class="fa fa-long-arrow-right"></i> **{{ now|date('F') }}**

#### Translate Language

Переводит строку на определенный язык. Для получения более подробной информации ознакомьтесь с [многоязычной документацией](/content/multi-language#complex-translations).

`'SIMPLE_TEXT'|tl(['fr'])`

#### Truncate a String

Вы можете легко создать сокращенную, усеченную версию строки, используя этот фильтр. Единственное обязательное поле - это количество символов, но есть и другие параметры:

`'one sentence. two sentences'|truncate(5)` <i class="fa fa-long-arrow-right"></i> {{ 'one sentence. two sentences'|truncate(5) }}

Просто обрезается до 5 символов.

`'one sentence. two sentences'|truncate(5, true)` <i class="fa fa-long-arrow-right"></i> {{ 'one sentence. two sentences'|truncate(5, true) }}

Обрезается до ближайшего конца предложения после 5 символов.

Вы также можете обрезать текст HTML, но сначала следует использовать фильтр `striptags`, чтобы удалить любое форматирование HTML, которое может быть нарушено, если вы остановитесь между тегами:

`'<p>one <strong>sentence<strong>. two sentences</p>'|striptags|truncate(5)` <i class="fa fa-long-arrow-right"></i> {{ '<p>one <strong>sentence<strong>. two sentences</p>'|striptags|truncate(5) }}

##### Специализированные версии:

**|safe_truncate**

Обрезать текст по количеству символов, делая текст «безопаснее».

**|truncate_html**

Обрезать HTML по количеству символов, не делая текст «безопаснее»!

**|safe_truncate_html**

Обрезать HTML по количеству символов, делая текст «безопаснее».

#### Underscoreize

Преобразует строку в формат "under_scored"

`'CamelCased'|underscorize` <i class="fa fa-long-arrow-right"></i> **{{ 'CamelCased'|underscorize }}**

[version=16,17]
#### Yaml Encode

Дамп/кодирование переменной в синтаксис YAML

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set array = {foo: [0, 1, 2, 3], baz: 'qux' } %}
{{ array|yaml_encode }}
[/prism]
{% endverbatim %}

{% set array = {foo: [0, 1, 2, 3], baz: 'qux' } %}
[prism classes="language-yaml"]
{{ array|yaml_encode|e }}
[/prism]

#### Yaml Decode

Декодирование/парсинг переменной из синтаксиса YAML

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set yaml = "foo: [0, 1, 2, 3]\nbaz: qux" %}
{{ yaml|yaml_decode|var_dump }}
[/prism]
{% endverbatim %}

{% set yaml = "foo: [0, 1, 2, 3]\nbaz: qux" %}
[prism]
{{ yaml|yaml_decode|var_dump }}
[/prism]
[/version]

## Twig-функции Grav

Функции Twig вызываются непосредственно с любыми параметрами, передаваемыми через скобки.

#### Array

Привести значение к массиву

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set value = array(value) %}
[/prism]
{% endverbatim %}

#### Array Difference

Вычисляет разницу массивов.

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set diff = array_diff(array1, array2...) %}
[/prism]
{% endverbatim %}

#### Array Key Value

Функция `array_key_value` позволяет добавить пару ключ/значение в ассоциированный массив

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set my_array = {fruit: 'apple'} %}
{% set my_array = array_key_value('meat','steak', my_array) %}
{{ print_r(my_array)}}
[/prism]
{% endverbatim %}

{% set my_array = {fruit: 'apple'} %}
{% set my_array = array_key_value('meat','steak', my_array) %}
outputs: ** {{ print_r(my_array) }} **

#### Array Key Exists

Обертка для функции `array_key_exists` PHP, которая возвращает, существует ли ключ в ассоциативном массиве или нет.

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set my_array = {fruit: 'apple', meat: 'steak'} %}
{{ array_key_exists('meat', my_array) }}
[/prism]
{% endverbatim %}

{% set my_array = {fruit: 'apple', meat: 'steak'} %}
outputs: **{{ array_key_exists('meat', my_array) }}**

#### Array Intersect

Функция `array_intersect` обеспечивает пересечение двух массивов или коллекций Grav.

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set array_1 = {fruit: 'apple', meat: 'steak'} %}
{% set array_2 = {fish: 'tuna', meat: 'steak'} %}
{{ print_r(array_intersect(array_1, array_2)) }}
[/prism]
{% endverbatim %}

{% set array_1 = {fruit: 'apple', meat: 'steak'} %}
{% set array_2 = {fish: 'tuna', meat: 'steak'} %}

outputs: **{{ print_r(array_intersect(array_1, array_2)) }}**

#### Array Unique

Обертка для PHP `array_unique()`, которая удаляет дубликаты из массива.

`array_unique(['foo', 'bar', 'foo', 'baz'])` <i class="fa fa-long-arrow-right"></i> **{{ print_r(array_unique(['foo', 'bar', 'foo', 'baz'])) }}**

#### Authorize

Уполномочивает авторизованного пользователя на просмотр ресурса. Принимает одну строку разрешения или массив строк разрешений.

`authorize(['admin.statistics', 'admin.super'])`

[version=16,17]
#### Body Class

Берет массив классов, и если они не установлены на `body_classes`, посмотрим, установлены ли они в текущей конфигурации темы.

`set body_classes = body_class(['header-fixed', 'header-animated', 'header-dark', 'header-transparent', 'sticky-footer'])`

#### Cron

Создать объект "Cron" из синтаксиса cron

`cron("3 * * * *").getNextRunDate()|date(config.date_format.default)`

[/version]


#### Dump

Берет корректную переменную Twig и выбрасывает ее в [панели отладчика Grav](/advanced/debugging). Для просмотра значений на вкладке "Сообщения" отладчик должен быть **включен**.

`dump(page.header)`

#### Debug

То же, что и `dump()`

#### Evaluate

Функцию оценки можно использовать для оценки строки как Twig:

`evaluate('grav.language.getLanguage')`

#### Evaluate Twig

Подобно оценке, но будет оценивать и обрабатывать с помощью Twig

{% verbatim %}
`evaluate_twig('This is a twig variable: {{ foo }}', {foo: 'bar'})`)  <i class="fa fa-long-arrow-right"></i> **This is a twig variable: bar**
{% endverbatim %}

#### EXIF

Вывод EXIF-данных с изображения на основе его файловой траектории. Для этого нужны эти "медиа": auto_metadata_exif: true` установлен в `system.yaml`. Например, в шаблоне Twig:

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set image = page.media['sample-image.jpg'] %}
{% set exif = exif(image.filepath, true) %}
{{ exif.MaxApertureValue }}
[/prism]
{% endverbatim %}

Это позволило бы записать в камеру значение `MaxApertureValue`, например, "40/10". Вы всегда можете использовать `{% verbatim %}{{dump(exif)}}{% endverbatim %}` для отображения всех доступных данных в отладчике.

#### Получение куки-файла

Восстановление значения куки-файла с помощью этой функции:

`get_cookie('your_cookie_key')`

[version=16,17]
#### Функция получения типа переменной

Получает тип переменной:

`get_type(page)` <i class="fa fa-long-arrow-right"></i> **{{ get_type(page) }}**
[/version]

#### Gist

Принимает Github Gist ID и создает соответствующий код для встраивания Gist.

`gist('bc448ff158df4bc56217')` <i class="fa fa-long-arrow-right"></i> {{ gist('bc448ff158df4bc56217')}}

[version=16,17]
#### Код ответа HTTP

Если предоставлен код response_code, то будет возвращен предыдущий код статуса. Если код response_code не предоставлен, то будет возвращен код текущего состояния. Оба эти значения по умолчанию будут иметь код состояния 200, если они используются в среде веб-сервера.

`http_response_code(404)`
[/version]

#### Is Ajax Request

Функция `isajaxrequest()` может быть использована для проверки, установлена ли опция заголовка `HTTP_X_REQUESTED_WITH`:


#### Функция декодирования JSON

Вы можете декодировать JSON, просто применив этот фильтр:

`json_decode({"first_name": "Guido", "last_name":"Rossum"})`

#### Media Directory

Возвращает медиа-объект для произвольной директории.  После получения изображениями можно манипулировать так же, как и страницами.

`media_directory('theme://images')['some-image.jpg'].cropResize(200,200).html`

[version=16,17]
#### NiceFilesize Function

Выводите размер файла в удобочитаемом для человека формате.

`nicefilesize(612394)` <i class="fa fa-long-arrow-right"></i> **{{ nicefilesize(612394) }}**

#### NiceNumber Function

Вывод числа в удобочитаемом для человека формате красивых чисел.

`nicenumnber(12430)` <i class="fa fa-long-arrow-right"></i> **{{ nicenumber(12430)}}**

#### NiceTime Function

Вывод даты в удобочитаемом человеком формате времени.

`nicetime(page.date)` <i class="fa fa-long-arrow-right"></i> **{{ nicetime(page.date) }}**
[/version]

#### Nonce Field

Сгенерируйте Grav security nonce поле для формы с требуемым `action`:

`nonce_field('action')` <i class="fa fa-long-arrow-right"></i> **{{ nonce_field('action')|e }}**

[version=16,17]
#### Of Type Function

Проверяет тип переменной на параметр:

`of_type(page, 'string')` <i class="fa fa-long-arrow-right"></i> **{{ of_type(page, 'string') ? 'true' : 'false' }}**
[/version]

#### Pathinfo

Разбирает путь в массив.

{% verbatim %}
[prism classes="language-twig"]
{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}
{{ print_r(parts) }}
[/prism]
{% endverbatim %}

{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}

outputs: **{{ print_r(parts) }}**

[version=16,17]
#### Функция вывода переменной

Печатает переменную в читаемом формате

`print_r(page.header)`

[prism classes="language-twig"]
{{ print_r(page.header) }}
[/prism]

[/version]

#### Генерация случайной строки

Сгенерирует произвольную строку из требуемого количества символов.  Особенно полезно при создании уникального идентификатора или ключа.

`random_string(10)` <i class="fa fa-long-arrow-right"></i> **{{ random_string(10) }}**

#### Range

Генерирует массив, содержащий ряд элементов, опционально ступенчатых

`range(25, 300, 50)` <i class="fa fa-long-arrow-right"></i> **{{ print_r(range(25, 300, 50)) }}**

[version=16,17]
#### Read File

Простая функция для чтения файла на основе пути к файлу и его вывода.

`read_file('plugins://admin/README.md')|markdown`

[prism classes="language-markdown line-numbers"]
# Стандартный плагин админки Grav

Это плагин **админки** для [Grav](https://github.com/getgrav/grav), он представляет собой HTML-интерфейс пользователя, который обеспечивает удобный способ настройки Grav, с возможностью легко создавать и изменять страницы....
[/prism]

[/version]

#### Redirect Me

Переадресация на URL-адрес по вашему выбору.

`redirect_me('http://google.com', 304)`

[version=16,17]
#### Функция фильтра регулярных выражений

Выполняет `preg_grep` на массиве с регрессионным паттерном.

`regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/")`

{% set var = regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/") %}
[prism classes="language-text"]
{{ print_r(var) }}
[/prism]

#### Функция замены регулярного выражения

Полезная обертка для PHP-функции [preg_replace()](https://php.net/manual/ru/function.preg-replace.php), с помощью этого фильтра вы можете выполнять сложные regex-замены текста:

{% verbatim %}
`regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle'])`
{% endverbatim %}

{% set var = regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) %}
[prism classes="language-text"]
{{var }}
[/prism]

[/version]

#### Repeat

Будет повторять все, что передано, определенное количество раз.

`repeat('blah ', 10)` <i class="fa fa-long-arrow-right"></i> **{{ repeat('blah ', 10) }}**

#### String

Возвращает строку из значения. Если значение является массивом, верните его в кодировке json

`string(23)` => **"23"**

`string(['test' => 'x'])` => **{"test":"x"}**

[version=17]
#### Изображение SVG

Возвращает содержимое SVG-изображения и добавляет дополнительные классы по мере необходимости. Обеспечивает преимущества встроенного svg без необходимости вставлять код непосредственно на страницу. Полезно для многократно используемых изображений, таких как значки социальных сетей.

{% verbatim %}
`{{ svg_image(path, classes, strip_style) }}`
{% endverbatim %}

strip_style = удалить встроенный стиль SVG - полезно для стилизации с помощью CSS-классов.

пример: `{{ svg_image('theme://images/something.svg', 'my-class-here mb-10', true/false) }}`

[/version]

[version=16,17]
#### Переменная темы

Получите переменную темы из заголовка страницы, если она существует, в противном случае используйте конфигурацию темы:

`theme_var('grid-size')`

Сначала попробуйте `page.header.grid_size`, если это не установлено, попробуйте `theme.grid_size` из файла конфигурации темы:

`theme_var('grid_size', 1024)`
[/version]

#### Функция перевода строки

Переведите строку, как фильтр `|t`.

`t('SITE_NAME')` <i class="fa fa-long-arrow-right"></i> **Site Name**

#### Функция перевода массива

Функция, связанная с фильтром `|ta`.

#### Функция перевода

Переводит строку на определенный язык. Для получения более подробной информации ознакомьтесь с [многоязычной документацией](/content/multi-language#complex-translations).

`tl('SIMPLE_TEXT', ['fr'])`

#### Url

Создает URL и преобразует любые потоки PHP URL в полноценные HTML ресурсы. Значение по умолчанию может быть передано в случае, если URL не может быть разрешен.

`url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4')` <i class="fa fa-long-arrow-right"></i> **{{ url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4') }}**

#### VarDump

Функция `vardump()` выводит на экран текущую переменную (а не в отладчике, как в `dump()`)

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set my_array = {foo: 'bar', baz: 'qux'} %}
{{ vardump(my_array) }}
[/prism]
{% endverbatim %}

{% set my_array = {foo: 'bar', baz: 'qux'} %}

[prism classes="language-twig"]
{{ vardump(my_array) }}
[/prism]

[version=16,17]
#### XSS

Разрешить ручную проверку строки на наличие уязвимостей XSS

`xss('this string contains a <script>alert("hello");</script> XSS vulnerability')` <i class="fa fa-long-arrow-right"></i> **{{ xss('this string contains a <script>alert("hello");</script> XSS vulnerability') }}**

[/version]
