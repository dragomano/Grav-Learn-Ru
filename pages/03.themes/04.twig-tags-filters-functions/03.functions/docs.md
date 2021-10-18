---
title: Функции Twig
metadata:
    description: 'Используемые функции Twig в Grav CMS.'
body_classes: twig__headers
page-toc:
  active: true
  start: 3
  depth: 1
process:
    twig: true
taxonomy:
    category: docs
---

Функции Twig вызываются непосредственно с любыми параметрами, передаваемыми через скобки.

### `array`

Привести значение к массиву

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set value = array(value) %}
[/prism]
{% endverbatim %}

### `array_diff`

Вычисляет разницу массивов.

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set diff = array_diff(array1, array2...) %}
[/prism]
{% endverbatim %}

### `array_key_value`

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
выводит: ** {{ print_r(my_array) }} **

### `array_key_exists`

Обертка для функции `array_key_exists` PHP, которая возвращает, существует ли ключ в ассоциативном массиве или нет.

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set my_array = {fruit: 'apple', meat: 'steak'} %}
{{ array_key_exists('meat', my_array) }}
[/prism]
{% endverbatim %}

{% set my_array = {fruit: 'apple', meat: 'steak'} %}
выводит: **{{ array_key_exists('meat', my_array) }}**

### `array_intersect`

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

выводит: **{{ print_r(array_intersect(array_1, array_2)) }}**

### `array_unique`

Обертка для PHP `array_unique()`, которая удаляет дубликаты из массива.

`array_unique(['foo', 'bar', 'foo', 'baz'])` <i class="fa fa-long-arrow-right"></i> **{{ print_r(array_unique(['foo', 'bar', 'foo', 'baz'])) }}**

### `authorize`

Уполномочивает авторизованного пользователя на просмотр ресурса. Принимает одну строку разрешения или массив строк разрешений.

`authorize(['admin.statistics', 'admin.super'])`

### `body_class`

Берет массив классов, и если они не установлены на `body_classes`, посмотрим, установлены ли они в текущей конфигурации темы.

`set body_classes = body_class(['header-fixed', 'header-animated', 'header-dark', 'header-transparent', 'sticky-footer'])`

### `cron`

Создать объект "Cron" из синтаксиса cron

`cron("3 * * * *").getNextRunDate()|date(config.date_format.default)`

### `dump`

Берет корректную переменную Twig и выбрасывает ее в [панели отладчика Grav](/advanced/debugging). Для просмотра значений на вкладке "Сообщения" отладчик должен быть **включен**.

`dump(page.header)`

### `debug`

То же, что и `dump()`

### `evaluate`

Функцию `evaluate` можно использовать для анализа строки как Twig:

`evaluate('grav.language.getLanguage')`

### `evaluate_twig`

То же, что и `evaluate`, но будет анализировать и обрабатывать с помощью Twig

{% verbatim %}
`evaluate_twig('This is a twig variable: {{ foo }}', {foo: 'bar'})`)  <i class="fa fa-long-arrow-right"></i> **This is a twig variable: bar**
{% endverbatim %}

### `exif`

Вывод EXIF-данных с изображения на основе его файловой траектории. Для этого нужны эти "медиа": auto_metadata_exif: true` установлен в `system.yaml`. Например, в шаблоне Twig:

{% verbatim %}
[prism classes="language-twig line-numbers"]
{% set image = page.media['sample-image.jpg'] %}
{% set exif = exif(image.filepath, true) %}
{{ exif.MaxApertureValue }}
[/prism]
{% endverbatim %}

Это позволило бы записать в камеру значение `MaxApertureValue`, например, "40/10". Вы всегда можете использовать `{% verbatim %}{{dump(exif)}}{% endverbatim %}` для отображения всех доступных данных в отладчике.

### `get_cookie`

Восстановление значения куки-файла с помощью этой функции:

`get_cookie('your_cookie_key')`

### `get_type`

Получает тип переменной:

`get_type(page)` <i class="fa fa-long-arrow-right"></i> **{{ get_type(page) }}**

### `gist`

Принимает Github Gist ID и создает соответствующий код для встраивания Gist.

`gist('bc448ff158df4bc56217')` <i class="fa fa-long-arrow-right"></i> {{ gist('bc448ff158df4bc56217')}}

### `http_response_code`

Если предоставлен код response_code, то будет возвращен предыдущий код статуса. Если код response_code не предоставлен, то будет возвращен код текущего состояния. Оба эти значения по умолчанию будут иметь код состояния 200, если они используются в среде веб-сервера.

`http_response_code(404)`

### `isajaxrequest`

Функция `isajaxrequest()` может быть использована для проверки, установлена ли опция заголовка `HTTP_X_REQUESTED_WITH`:

### `json_decode`

Вы можете декодировать JSON, просто применив этот фильтр:

`json_decode({"first_name": "Guido", "last_name":"Rossum"})`

### `media_directory`

Возвращает медиа-объект для произвольной директории.  После получения изображениями можно манипулировать так же, как и страницами.

`media_directory('theme://images')['some-image.jpg'].cropResize(200,200).html`

### `nicefilesize`

Выводите размер файла в удобочитаемом для человека формате.

`nicefilesize(612394)` <i class="fa fa-long-arrow-right"></i> **{{ nicefilesize(612394) }}**

### `nicenumber`

Вывод числа в удобочитаемом для человека формате красивых чисел.

`nicenumnber(12430)` <i class="fa fa-long-arrow-right"></i> **{{ nicenumber(12430)}}**

### `nicetime`

Вывод даты в удобочитаемом человеком формате времени.

`nicetime(page.date)` <i class="fa fa-long-arrow-right"></i> **{{ nicetime(page.date) }}**

### `nonce_field`

Генерирует поле с кодом безопасности Grav для формы с требуемым `action`:

`nonce_field('action')` <i class="fa fa-long-arrow-right"></i> **{{ nonce_field('action')|e }}**

### `of_type`

Проверяет тип переменной на параметр:

`of_type(page, 'string')` <i class="fa fa-long-arrow-right"></i> **{{ of_type(page, 'string') ? 'true' : 'false' }}**

### `pathinfo`

Разбирает путь в массив.

{% verbatim %}
[prism classes="language-twig"]
{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}
{{ print_r(parts) }}
[/prism]
{% endverbatim %}

{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}

выводит: **{{ print_r(parts) }}**

### `print_r`

Печатает переменную в читаемом формате

`print_r(page.header)`

[prism classes="language-twig"]
{{ print_r(page.header) }}
[/prism]

### `random_string`

Сгенерирует произвольную строку из требуемого количества символов.  Особенно полезно при создании уникального идентификатора или ключа.

`random_string(10)` <i class="fa fa-long-arrow-right"></i> **{{ random_string(10) }}**

### `unique_id`

Генерирует случайную строку с настраиваемой длиной, префиксом и суффиксом. В отличие от встроенной функции PHP `uniqid()` и утилиты `random_string`, эта строка будет сгенерирована действительно уникальной.

`unique_id(9)` <i class="fa fa-long-arrow-right"></i> **{{ unique_id(9) }}**
`unique_id(11, { prefix: 'user_' })` <i class="fa fa-long-arrow-right"></i> **unique_id(11, { prefix: 'user_' }) }}**
`unique_id(13, { suffix: '.json' })` <i class="fa fa-long-arrow-right"></i> **unique_id(13, { suffix: '.json' }) }}**

### `range`

Генерирует массив, содержащий ряд элементов, опционально ступенчатых

`range(25, 300, 50)` <i class="fa fa-long-arrow-right"></i> **{{ print_r(range(25, 300, 50)) }}**

### `read_file`

Простая функция для чтения файла на основе пути к файлу и его вывода.

`read_file('plugins://admin/README.md')|markdown`

[prism classes="language-markdown line-numbers"]
# Стандартный плагин админки Grav

Это плагин **админки** для [Grav](https://github.com/getgrav/grav), он представляет собой HTML-интерфейс пользователя, который обеспечивает удобный способ настройки Grav, с возможностью легко создавать и изменять страницы....
[/prism]

### `redirect_me`

Переадресация на URL-адрес по вашему выбору.

`redirect_me('http://google.com', 304)`

### `regex_filter`

Выполняет `preg_grep` на массиве с регрессионным паттерном.

`regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/")`

{% set var = regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/") %}
[prism classes="language-text"]
{{ print_r(var) }}
[/prism]

### `regex_replace`

Полезная обертка для PHP-функции [preg_replace()](https://php.net/manual/ru/function.preg-replace.php), с помощью этого фильтра вы можете выполнять сложные regex-замены текста:

{% verbatim %}
`regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle'])`
{% endverbatim %}

{% set var = regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) %}
[prism classes="language-text"]
{{var }}
[/prism]

[/version]

[version=17]
### `regex_match`

Полезная обертка для PHP-метода [preg_match()](https://php.net/manual/en/function.preg-math.php), с помощью этого фильтра можно выполнить сложное сопоставление регулярных выражений с текстом:

{% verbatim %}
`regex_match('http://www.php.net/index.html', '@^(?:http://)?([^/]+)@i')`
{% endverbatim %}

{% set var = regex_match('http://www.php.net/index.html', '@^(?:http://)?([^/]+)@i') %}
[prism classes="language-text"]
{{ print_r(var) }}
[/prism]

### `regex_split`

Полезная обертка для PHP-метода [preg_split()](https://php.net/manual/en/function.preg-split.php). Разбиение строки регулярным выражением на текст с помощью этого фильтра:

{% verbatim %}
`regex_split('hypertext language, programming', '/\\s*,\\s*/u')`
{% endverbatim %}

{% set var = regex_split('hypertext language    ,    programming', '/\\s*,\\s*/u') %}
[prism classes="language-text"]
{{ print_r(var) }}
[/prism]

### `repeat`

Будет повторять всё, что передано, определенное количество раз.

`repeat('blah ', 10)` <i class="fa fa-long-arrow-right"></i> **{{ repeat('blah ', 10) }}**

### `string`

Возвращает строку из значения. Если значение является массивом, верните его в кодировке json

`string(23)` => **"23"**

`string(['test' => 'x'])` => **{"test":"x"}**

### `svg_image`

Возвращает содержимое SVG-изображения и добавляет дополнительные классы по мере необходимости. Обеспечивает преимущества встроенного svg без необходимости вставлять код непосредственно на страницу. Полезно для многократно используемых изображений, таких как значки социальных сетей.

{% verbatim %}
`{{ svg_image(path, classes, strip_style) }}`
{% endverbatim %}

strip_style = удалить встроенный стиль SVG - полезно для стилизации с помощью CSS-классов.

пример:

{% verbatim %}
`{{ svg_image('theme://images/something.svg', 'my-class-here mb-10', true) }}`
{% endverbatim %}

### `theme_var`

Получите переменную темы из заголовка страницы, если она существует, в противном случае используйте конфигурацию темы:

`theme_var('grid-size')`

Сначала попробуйте `page.header.grid_size`, если это не установлено, попробуйте `theme.grid_size` из файла конфигурации темы:

`theme_var('grid_size', 1024)`

### `t`

Переведите строку, как фильтр `|t`.

`t('SITE_NAME')` <i class="fa fa-long-arrow-right"></i> **Site Name**

### `ta`

Функция, связанная с фильтром `|ta`.

### `tl`

Переводит строку на определенный язык. Для получения более подробной информации ознакомьтесь с [документацией](/content/multi-language#complex-translations).

`tl('SIMPLE_TEXT', ['fr'])`

### `url`

Создает URL и преобразует любые потоки PHP URL в полноценные HTML ресурсы. Значение по умолчанию может быть передано в случае, если URL не может быть разрешен.

`url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4')` <i class="fa fa-long-arrow-right"></i> **{{ url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4') }}**

### `vardump`

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

### `xss`

Разрешить ручную проверку строки на наличие уязвимостей XSS

`xss('this string contains a <script>alert("hello");</script> XSS vulnerability')` <i class="fa fa-long-arrow-right"></i> **{{ xss('this string contains a <script>alert("hello");</script> XSS vulnerability') }}**