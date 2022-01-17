# Фильтры Twig

> Используемые фильтры Twig в Grav CMS.

Фильтры Twig применяются к переменным Twig с помощью символа `|`, за которым следует имя фильтра. Параметры можно передавать так же, как функции Twig, используя круглые скобки.

#### `absolute_url`

Принимает фрагмент HTML, содержащий атрибут `src` или `href`, который использует относительный путь. Преобразует строку пути в абсолютный формат URL, включая имя хоста.

`'<img src="/some/path/to/image.jpg" />'|absolute_url` <i class="fa fa-long-arrow-right"></i> `{{ '<img src="/some/path/to/image.jpg" />'|absolute_url }}`

#### `array_unique`

Оболочка для PHP 'array_unique()', которая удаляет дубликаты из массива.

`['foo', 'bar', 'foo', 'baz']|array_unique` <i class="fa fa-long-arrow-right"></i> **{{ print_r(['foo', 'bar', 'foo', 'baz']|array_unique) }}**

#### `base32_encode`

Выполняет кодировку base32 для переменной
`'some variable here'|base32_encode` <i class="fa fa-long-arrow-right"></i> **{{ 'some variable here'|base32_encode }}**

#### `base32_decode`

Выполняет декодирование base32 на переменной
`'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK'|base32_decode` <i class="fa fa-long-arrow-right"></i> **{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGK'|base32_decode }}**

#### `base64_encode`

Выполняет кодировку base64 для переменной
`'some variable here'|base64_encode` <i class="fa fa-long-arrow-right"></i> **{{ 'some variable here'|base64_encode }}**

#### `base64_decode`

Выполняет декодирование base64 на переменной
`'c29tZSB2YXJpYWJsZSBoZXJl'|base64_decode` <i class="fa fa-long-arrow-right"></i> **{{ 'c29tZSB2YXJpYWJsZSBoZXJl'|base64_decode }}**

#### `basename`

Возвращает базовое имя пути.

`'/etc/sudoers.d'|basename` <i class="fa fa-long-arrow-right"></i> **{{ '/etc/sudoers.d'|basename }}**

#### `camelize`

Преобразует строку в формат "CamelCase"

`'send_email'|camelize` <i class="fa fa-long-arrow-right"></i> **{{ 'send_email'|camelize }}**

#### `chunk_split`

Разбивает строку на более мелкие куски определенного размера.

`'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGKA'|chunk_split(6, '-')` <i class="fa fa-long-arrow-right"></i> **{{ 'ONXW2ZJAOZQXE2LBMJWGKIDIMVZGKA'|chunk_split(6, '-') }}**

#### `contains`

Определите, содержит ли конкретная строка другую строку

`'some string with things in it'|contains('things')` <i class="fa fa-long-arrow-right"></i> **{{ 'some string with things in it'|contains('things') }}**

#### Преобразование значений

PHP 7 получает более строгие проверки типов, а это означает, что передача значения неправильного типа теперь может вызвать исключение. Чтобы избежать этого, вы должны использовать фильтры, которые гарантируют, что значение, переданное методу, является допустимым:

### `string`

Используйте `|string` для приведения значения к строке.

### `int`

Используйте `|int` для приведения значения к целому числу.

### `bool`

Используйте `|bool` для приведения значения к логическому.

### `float`

Используйте `|float` для приведения значения к числу с плавающей запятой.

### `array`

Используйте `|array` для приведения значения к массиву.

### `defined`

Иногда вы хотите проверить, определена ли какая-либо переменная, а если нет, укажите значение по умолчанию. Например:

`set header_image_width  = page.header.header_image_width|defined(900)`

При этом переменная `header_image_width` будет установлена в значение `900`, если она не определена в заголовке страницы.

### `dirname`

Вернуть имя каталога пути.

`'/etc/sudoers.d'|dirname` <i class="fa fa-long-arrow-right"></i> **{{ '/etc/sudoers.d'|dirname }}**

### `ends_with`

Берет иголку и стог сена и определяет, заканчивается ли стог иглой. Также теперь работает с массивом игл и будет возвращать `true`, если **любой** стог сена заканчивается иглой.

`'the quick brown fox'|ends_with('fox')` <i class="fa fa-long-arrow-right"></i> {{  'the quick brown fox'|ends_with('fox') ? 'true' : 'false' }}

### `fieldName`

Фильтрация имени поля путем замены записи через точку на запись массива

`'field.name|fieldName`

### `get_type`

Получает тип переменной:

`page|get_type` <i class="fa fa-long-arrow-right"></i> **{{ page|get_type }}**

### `humanize`

Преобразует строку в более удобный для чтения формат.

`'something_text_to_read'|humanize` <i class="fa fa-long-arrow-right"></i> **{{ 'something_text_to_read'|humanize }}**

### `hyphenize`

Преобразует строку в версию с дефисом.

`'Something Text to Read'|hyphenize` <i class="fa fa-long-arrow-right"></i> **{{ 'Something Text to Read'|hyphenize }}**

### `json_decode`

Вы можете декодировать JSON, просто применив этот фильтр:

`array|json_decode`
```twig
{% set array = '{"first_name": "Guido", "last_name":"Rossum"}'|json_decode %}
{{ print_r(array) }}
```


{% set array = '{"first_name": "Guido", "last_name":"Rossum"}'|json_decode %}
```text
{{ print_r(array) }}
```

### `ksort`

Сортировать карту массива по каждому ключу

`array|ksort`
```twig
{% set ritems = {'orange':1, 'apple':2, 'peach':3}|ksort %}
{% for key, value in ritems %}{{ key }}:{{ value }}, {% endfor %}
```


{% set ritems = {'orange':1, 'apple':2, 'peach':3}|ksort %}
```twig
{% for key, value in ritems %}{{ key }}:{{ value }}, {% endfor %}
```

### `ltrim`

`'/strip/leading/slash/'|ltrim('/')` <i class="fa fa-long-arrow-right"></i> {{ '/strip/leading/slash/'|ltrim('/') }}

Удаляет пробелы в начале строки. Он также может удалить другие символы, установив маску символов (см. [https://php.net/manual/ru/function.ltrim.php](https://php.net/manual/ru/function.ltrim.php)).

### `markdown`

Возьмите произвольную строку, содержащую разметку, и конвертируйте её в HTML, используя парсер Markdown Grav. Необязательный `boolean`-параметр:

* `true` (по умолчанию): обрабатывать как блок (текстовый режим, содержимое будет заключено в теги `<p>`)
* `false`: обрабатывать как строку (содержимое не будет обернуто никакими тегами)

```
string|markdown($is_block)
```

```twig
<div class="div">
{{ 'A paragraph with **markdown** and [a link](http://www.cnn.com)'|markdown }}
</div>

<p class="paragraph">{{'A line with **markdown** and [a link](http://www.cnn.com)'|markdown(false) }}</p>
```

{% set var %}
<div class="div">
{{ 'A paragraph with **markdown** and [a link](http://www.cnn.com)'|markdown }}
</div>

<p class="paragraph">{{'A line with **markdown** and [a link](http://www.cnn.com)'|markdown(false) }}</p>
{% endset %}
```text
{{ var|e }}
```

### `md5`

Создает md5 хэш для строки

`'anything'|md5` <i class="fa fa-long-arrow-right"></i> **{{ 'anything'|md5 }}**

### `modulus`

Выполняет ту же функциональность, что и символ Modulus `%` в PHP. Он работает с числом, передавая в числовом делителе и необязательный массив элементов для выбора.

`7|modulus(3, ['red', 'blue', 'green'])` <i class="fa fa-long-arrow-right"></i> **{{ 7|modulus(3, ['red', 'blue', 'green']) }}**

### `monthize`

Преобразовывает целое число дней в число месяцев

`'181'|monthize` <i class="fa fa-long-arrow-right"></i> **{{ '181'|monthize }}**

### `nicecron`

Получает читаемый человеком выходной сигнал для синтаксиса cron

`"2 * * * *"|nicecron` <i class="fa fa-long-arrow-right"></i> **{{ '2 * * * *'|nicecron }}**

### `nicefilesize`

Выводите размер файла в удобочитаемом для человека формате.

`612394|nicefilesize` <i class="fa fa-long-arrow-right"></i> **{{ 612394|nicefilesize }}**

### `nicenumber`

Вывод числа в удобочитаемом для человека формате красивых чисел.

`12430|nicenumber` <i class="fa fa-long-arrow-right"></i> **{{ 12430|nicenumber }}**

### `nicetime`

Вывод даты в удобочитаемом человеком формате времени.

`page.date|nicetime(false)` <i class="fa fa-long-arrow-right"></i> **{{ page.date|nicetime(false) }}**

Первый аргумент указывает, следует ли использовать описание даты в полном формате. По умолчанию это `true`.

Вы можете указать второй аргумент `false`, если хотите удалить относительный дескриптор времени (например, «назад» или «с этого момента» на вашем языке) из результата.

### `of_type`

Проверяет тип переменной на параметр:

`page|of_type('string')` <i class="fa fa-long-arrow-right"></i> **{{ page|of_type('string') ? 'true' : 'false' }}**

### `ordinalize`

Добавляет порядковый номер к целому числу (например, 1-е, 2-е, 3-е, 4-е).

`'10'|ordinalize` <i class="fa fa-long-arrow-right"></i> **{{ '10'|ordinalize }}**

### `pad`

Заполняет строку до определенной длины другим символом. Это обёртка для PHP-функции [str_pad()](https://php.net/manual/ru/function.str-pad.php).

`'foobar'|pad(10, '-')` <i class="fa fa-long-arrow-right"></i> **{{ 'foobar'|pad(10, '-') }}**

### `pluralize`

Преобразовывает строку в английскую множественную версию

`'person'|pluralize` <i class="fa fa-long-arrow-right"></i> **{{ 'person'|pluralize }}**

`pluralize` также принимает необязательный числовой параметр, который можно ввести, если заранее неизвестно, на сколько элементов будет ссылаться существительное. Значение по умолчанию равно 2, поэтому при опущении будет указана форма множественного числа. Например:

```twig
<p>We have {{ num_vacancies }} {{ 'vacancy'|pluralize(num_vacancies) }} right now.</p>
```

### `print_r`

Печатает читабельную информацию о переменной

`page.header|print_r`

```text
{{ page.header|print_r }}
```

### `randomize`

Рандомизирует предоставленный список.  Если значение задано в качестве параметра, оно пропустит эти значения и сохранит их в порядке.

`array|randomize`

```twig
{% set ritems = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten']|randomize(2) %}
{% for ritem in ritems %}{{ ritem }}, {% endfor %}
```

<strong>
{% set ritems = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten']|randomize(2) %}
{% for ritem in ritems %}{{ ritem }}, {% endfor %}
</strong>

### `regex_replace`

Полезная оболочка для PHP [preg_replace()](https://php.net/manual/ru/function.preg-replace.php) метод, вы можете выполнять сложные замены регулярных выражений в тексте с помощью этого фильтра:

`'The quick brown fox jumps over the lazy dog.'|regex_replace(['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle'])` <i class="fa fa-long-arrow-right"></i> **{{ 'The quick brown fox jumps over the lazy dog.'|regex_replace(['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) }}**

?> По возможности используйте разделитель `~` вместо `/`. В противном случае вам, скорее всего, придется [удвоить определенные символы](https://github.com/getgrav/grav/issues/833). Например, `~\/\#.*~~, а не `'/\\/\\#.*/'`, что более соответствует [PCRE-синтаксису](https://www.php.net/manual/ru/regexp.reference.delimiters.php), используемому PHP.

### `rtrim`

`'/strip/trailing/slash/'|rtrim('/')` <i class="fa fa-long-arrow-right"></i> {{ '/strip/trailing/slash/'|rtrim('/') }}

Удаляет пробелы в конце строки. Он также может удалить другие символы, установив маску символов (см. [https://php.net/manual/ru/function.rtrim.php](https://php.net/manual/ru/function.rtrim.php)).

### `singularize`

Преобразует строку в английскую единственную версию

`'shoes'|singularize` <i class="fa fa-long-arrow-right"></i> **{{ 'shoes'|singularize }}**

### `safe_email`

Фильтр безопасной почты преобразует адрес электронной почты в символы ASCII, что затрудняет распознавание и захват спам-ботов.

`"someone@domain.com"|safe_email` <i class="fa fa-long-arrow-right"></i> {{ "someone@domain.com"|safe_email }}

Пример использования со ссылкой mailto:

```
<a href="mailto:{{'your.email@server.com'|safe_email}}">
  Email me
</a>
```

Сначала вы можете и не заметить разницы, но изучение исходного текста страницы (не используя Инструменты разработчика браузера, фактического исходного текста страницы) выявит кодировку, лежащую в основе символов.

### `sort_by_key`

Можно сортировать массив по определенному ключу

`array|sort_by_key`

```twig
{% set people = [{'email':'fred@yahoo.com', 'id':34}, {'email':'tim@exchange.com', 'id':21}, {'email':'john@apple.com', 'id':2}]|sort_by_key('id') %}
{% for person in people %}{{ person.email }}:{{ person.id }}, {% endfor %}
```

<strong>
{% set people = [{'email':'fred@yahoo.com', 'id':34}, {'email':'tim@exchange.com', 'id':21}, {'email':'john@apple.com', 'id':2}]|sort_by_key('id') %}
{% for person in people %}{{ person.email }}:{{ person.id }}, {% endfor %}
</strong>

### `starts_with`

Берет иглу и стог сена и определяет, начинается ли стог сена с иглы. Также теперь работает с массивом игл и вернет `true`, если **ни** стог сена начинается с иглы.

`'the quick brown fox'|starts_with('the')` <i class="fa fa-long-arrow-right"></i> {{  'the quick brown fox'|starts_with('the') ? 'true' : 'false' }}

### `titleize`

Преобразует строку в формат «Заголовок»

`'welcome page'|titleize` <i class="fa fa-long-arrow-right"></i> **{{ 'welcome page'|titleize }}**

### `t`

Перевести строку на текущий язык

`'MY_LANGUAGE_KEY_STRING'|t` <i class="fa fa-long-arrow-right"></i> 'Some Text in English'

Предполагается, что на вашем сайте переведены эти языковые строки и включена поддержка нескольких языков. Пожалуйста, обратитесь к [документации](/02.content/11.multi-language/index) для получения более подробной информации.

### `tu`

Перевести строку на текущий язык, установленный в пользовательских настройках интерфейса администратора.

`'MY_LANGUAGE_KEY_STRING'|tu` <i class="fa fa-long-arrow-right"></i> 'Some Text in English'

При этом используется языковое поле, установленное в пользовательском yaml.

### `ta`

Переводит массив на язык с помощью фильтра `|ta`. См. Подробный пример в [документации](/02.content/11.multi-language/index).

`'MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1)` <i class="fa fa-long-arrow-right"></i> **{{ now|date('F') }}**

### `tl`

Переводит строку на определенный язык. Для получения более подробной информации ознакомьтесь с [документацией](/02.content/11.multi-language/index?id=Комплексные-переводы).

`'SIMPLE_TEXT'|tl(['fr'])`

### `truncate`

Вы можете легко создать сокращенную, усеченную версию строки, используя этот фильтр. Единственное обязательное поле - это количество символов, но есть и другие параметры:

`'одно предложение. два предложения'|truncate(5)|raw` <i class="fa fa-long-arrow-right"></i> {{ 'одно предложение. два предложения'|truncate(5)|raw }}

Просто обрезается до 5 символов.

`'одно предложение. два предложения'|truncate(5, true)|raw` <i class="fa fa-long-arrow-right"></i> {{ 'одно предложение. два предложения'|truncate(5, true)|raw }}

> Фильтр `|raw` следует использовать с элементом заполнения по умолчанию `&helip;` (многоточие), чтобы он отображался с автоматическим экранированием Twig

Обрезается до ближайшего конца предложения после 5 символов.

Вы также можете обрезать текст HTML, но сначала следует использовать фильтр `|striptags`, чтобы удалить любое форматирование HTML, которое может быть нарушено, если вы остановитесь между тегами:

`'<span>одно <strong>предложение</strong>. два предложения</span>'
|raw|striptags|truncate(25)` <i class="fa fa-long-arrow-right"></i> **{{ '<span>одно <strong>предложение</strong>. два предложения</span>'|raw|striptags|truncate(25) }}**

#### Специализированные версии:

### `safe_truncate`

Используйте `|safe_truncate` для обрезки текста по количеству символов, с сохранением полных слов.

### `truncate_html`

Используйте `|truncate_html` для обрезки HTML по количеству символов, без сохранения полных слов!

### `safe_truncate_html`

Используйте `|safe_truncate_html` для обрезки HTML по количеству символов, с сохранением полных слов.

### `underscorize`

Преобразует строку в формат «under_scored»

`'CamelCased'|underscorize` <i class="fa fa-long-arrow-right"></i> **{{ 'CamelCased'|underscorize }}**

### `yaml_encode`

Дамп/кодирование переменной в синтаксис YAML

```twig
{% set array = {foo: [0, 1, 2, 3], baz: 'qux' } %}
{{ array|yaml_encode }}
```

{% set array = {foo: [0, 1, 2, 3], baz: 'qux' } %}

```yaml
{{ array|yaml_encode|e }}
```

### `yaml_decode`

Декодирование/парсинг переменной из синтаксиса YAML

```twig
{% set yaml = "foo: [0, 1, 2, 3]\nbaz: qux" %}
{{ yaml|yaml_decode|var_dump }}
```

{% set yaml = "foo: [0, 1, 2, 3]\nbaz: qux" %}

```yaml
{{ yaml|yaml_decode|var_dump }}
```