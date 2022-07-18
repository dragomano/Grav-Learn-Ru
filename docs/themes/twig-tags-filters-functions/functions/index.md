---
description: Используемые функции Twig в Grav CMS.
---

# Функции Twig

Функции Twig вызываются непосредственно с любыми параметрами, передаваемыми через скобки.

### `array`

Приводит значение к массиву:

```twig
{% set value = array(value) %}
```

### `array_diff`

Вычисляет разницу массивов:

```twig
{% set diff = array_diff(array1, array2...) %}
```

### `array_key_value`

Позволяет добавить пару ключ/значение в ассоциированный массив:

```twig
{% set my_array = {fruit: 'apple'} %}
{% set my_array = array_key_value('meat','steak', my_array) %}
{{ print_r(my_array) }}
```

### `array_key_exists`

Обертка для функции `array_key_exists` PHP, которая проверяет, существует ли ключ в ассоциативном массиве или нет:

```twig
{% set my_array = {fruit: 'apple', meat: 'steak'} %}
{{ array_key_exists('meat', my_array) }}
```

### `array_intersect`

Функция `array_intersect` обеспечивает пересечение двух массивов или коллекций Grav:

```twig
{% set array_1 = {fruit: 'apple', meat: 'steak'} %}
{% set array_2 = {fish: 'tuna', meat: 'steak'} %}
{{ print_r(array_intersect(array_1, array_2)) }}
```

### `array_unique`

Обертка для PHP `array_unique()`, которая удаляет дубликаты из массива:

```twig
{{ print_r(array_unique(['foo', 'bar', 'foo', 'baz'])) }}
```

```text
Array
(
  0 => "foo"
  1 => "bar"
  3 => "baz"
)
```

### `authorize`

Уполномочивает авторизованного пользователя на просмотр ресурса. Принимает одну строку разрешения или массив строк разрешений.

```twig
{{ print_r(authorize(['admin.statistics', 'admin.super'])) }}
```

### `body_class`

Берет массив классов, и если они не установлены на `body_classes`, посмотрим, установлены ли они в текущей конфигурации темы.

```twig
{% set body_classes = body_class(['header-fixed', 'header-animated', 'header-dark', 'header-transparent', 'sticky-footer']) %}
```

### `cron`

Создаёт объект "Cron" из синтаксиса cron:

```twig
{{ cron("3 * * * *").getNextRunDate()|date(config.date_format.default) }}
```

### `dump`

Берет корректную переменную Twig и выбрасывает её в [панели отладчика Grav](/advanced/debugging). Для просмотра значений на вкладке "Сообщения" отладчик должен быть **включен**.

```twig
{{ dump(page.header) }}
```

### `debug`

То же, что и `dump()`

### `evaluate`

Функцию `evaluate` можно использовать для анализа строки как Twig:

```twig
{{ evaluate('grav.language.getLanguage') }}
```

### `evaluate_twig`

То же, что и `evaluate`, но будет анализировать и обрабатывать с помощью Twig

```twig
{{ evaluate_twig('This is a twig variable: {{ foo }}', {foo: 'bar'}) }}
```

### `exif`

Вывод EXIF-данных с изображения на основе его файловой траектории. Для этого нужны эти "медиа": auto_metadata_exif: true` установлен в `system.yaml`. Например, в шаблоне Twig:

```twig
{% set image = page.media['sample-image.jpg'] %}
{% set exif = exif(image.filepath, true) %}
{{ exif.MaxApertureValue }}
```

Это позволило бы записать в камеру значение `MaxApertureValue`, например, "40/10". Вы всегда можете использовать `{{dump(exif)}}` для отображения всех доступных данных в отладчике.

### `get_cookie`

Восстановление значения куки-файла с помощью этой функции:

```twig
{{ get_cookie('your_cookie_key') }}
```

### `get_type`

Получает тип переменной:

```twig
{{ get_type(page) }}
```

### `gist`

Принимает Github Gist ID и создает соответствующий код для встраивания Gist:

```twig
{{ gist('bc448ff158df4bc56217') }}
```

### `header_var`

Вспомогательная функция. Возвращает `page.header.<variable>`.

С учётом frontmatter
```
---
title: Home
---
```

`header_var('title')` :fontawesome-solid-arrow-right: **About**

### `http_response_code`

Если предоставлен код response_code, то будет возвращен предыдущий код статуса. Если код response_code не предоставлен, то будет возвращен код текущего состояния. Оба эти значения по умолчанию будут иметь код состояния 200, если они используются в среде веб-сервера.

```twig
{{ http_response_code(404) }}
```

### `isajaxrequest`

Функция `isajaxrequest()` может быть использована для проверки, установлена ли опция заголовка `HTTP_X_REQUESTED_WITH`:

### `json_decode`

Вы можете декодировать JSON, просто применив этот фильтр:

```twig
{{ json_decode({"first_name": "Guido", "last_name":"Rossum"}) }}
```

### `media_directory`

Возвращает медиа-объект для произвольной директории. После получения изображениями можно манипулировать так же, как и страницами.

```twig
{{ media_directory('theme://images')['some-image.jpg'].cropResize(200,200).html }}
```

### `nicefilesize`

Выводите размер файла в удобочитаемом для человека формате.

```twig
{{ nicefilesize(612394) }}
```

### `nicenumber`

Вывод числа в удобочитаемом для человека формате красивых чисел.

```twig
{{ nicenumnber(12430) }}
```

### `nicetime`

Вывод даты в удобочитаемом человеком формате времени.

```twig
{{ nicetime(page.date) }}
```

### `nonce_field`

Генерирует поле с кодом безопасности Grav для формы с требуемым `action`:

```twig
{{ nonce_field('action') }}
```

### `of_type`

Проверяет тип переменной на параметр:

```twig
{{ of_type(page, 'string') }}
```

### `pathinfo`

Разбирает путь в массив:

```twig
{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}
{{ print_r(parts) }}
```

```text
Array
(
  [dirname] => /www/htdocs/inc
  [basename] => lib.inc.php
  [extension] => php
  [filename] => lib.inc
)
```

### `print_r`

Печатает переменную в читаемом формате:

```twig
{{ print_r(page.header) }}
```

### `random_string`

Сгенерирует произвольную строку из требуемого количества символов. Особенно полезно при создании уникального идентификатора или ключа.

```twig
{{ random_string(10) }}
```

### `unique_id`

Генерирует случайную строку с настраиваемой длиной, префиксом и суффиксом. В отличие от встроенной функции PHP `uniqid()` и утилиты `random_string`, эта строка будет сгенерирована действительно уникальной.

```twig
{{ unique_id(9) }}
{{ unique_id(11, { prefix: 'user_' }) }}
{{ unique_id(13, { suffix: '.json' }) }}
```

### `range`

Генерирует массив, содержащий ряд элементов, опционально ступенчатых

```twig
{{ range(25, 300, 50) }}
```

### `read_file`

Простая функция для чтения файла на основе пути к файлу и его вывода.

```twig
{{ read_file('plugins://admin/README.md')|markdown }}
```

```markdown
# Стандартный плагин админки Grav

Это плагин **админки** для [Grav](https://github.com/getgrav/grav), он представляет собой HTML-интерфейс пользователя, который обеспечивает удобный способ настройки Grav, с возможностью легко создавать и изменять страницы....
```

### `redirect_me`

Переадресация на URL-адрес по вашему выбору.

```twig
{{ redirect_me('http://google.com', 304) }}
```

### `regex_filter`

Выполняет `preg_grep` на массиве с регрессионным паттерном:

```twig
{{ regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/") }}
```

```text
Array
(
  0 => "pasta"
  3 => "potatoes"
)
```

### `regex_replace`

Полезная обертка для PHP-функции [preg_replace()](https://php.net/manual/ru/function.preg-replace.php), с помощью этого фильтра вы можете выполнять сложные regex-замены текста:

```twig
{{ regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) }}
```

```text
The slow black bear jumps over the lazy turtle.
```

### `regex_match`

Полезная обертка для PHP-метода [preg_match()](https://php.net/manual/ru/function.preg-math.php), с помощью этого фильтра можно выполнить сложное сопоставление регулярных выражений с текстом:

```twig
{{ regex_match('http://www.php.net/index.html', '@^(?:http://)?([^/]+)@i') }}
```

```text
Array
(
  [0] => http://www.php.net
  [1] => www.php.net
)
```

### `regex_split`

Полезная обертка для PHP-метода [preg_split()](https://php.net/manual/ru/function.preg-split.php). Разбиение строки регулярным выражением на текст с помощью этого фильтра:

```twig
{{ regex_split('hypertext language, programming', '/\\s*,\\s*/u') }}
```

```text
Array
(
  [0] => hypertext language
  [1] => programming
)
```

### `repeat`

Будет повторять всё, что передано, определенное количество раз.

```twig
{{ repeat('blah ', 10) }}
```

### `string`

Возвращает строку из значения. Если значение является массивом, верните его в кодировке json

```twig
{{ string(23) }}
{{ string(['test' => 'x']) }}
```

### `svg_image`

Возвращает содержимое SVG-изображения и добавляет дополнительные классы по мере необходимости. Обеспечивает преимущества встроенного svg без необходимости вставлять код непосредственно на страницу. Полезно для многократно используемых изображений, таких как значки социальных сетей.

```twig
{{ svg_image(path, classes, strip_style) }}
```

`strip_style` - удалить встроенный стиль SVG - полезно для стилизации с помощью CSS-классов.

пример:

```twig
{{ svg_image('theme://images/something.svg', 'my-class-here mb-10', true) }}
```

### `theme_var`

Получите переменную темы из заголовка страницы, если она существует, в противном случае используйте конфигурацию темы:

```twig
{{ theme_var('grid-size') }}
```

Сначала попробуйте `page.header.grid_size`, если это не установлено, попробуйте `theme.grid_size` из файла конфигурации темы:

```twig
{{ theme_var('grid_size', 1024) }}
```

### `t`

Переведите строку, как фильтр `|t`.

```twig
{{ t('SITE_NAME') }}
```

### `ta`

Функция, связанная с фильтром `|ta`.

### `tl`

Переводит строку на определенный язык. Для получения более подробной информации ознакомьтесь с [документацией](/content/multi-language/#kompleksnye-perevody).

```twig
{{ tl('SIMPLE_TEXT', ['fr']) }}
```

### `url`

Создает URL и преобразует любые потоки PHP URL в полноценные HTML ресурсы. Значение по умолчанию может быть передано в случае, если URL не может быть разрешен.

```twig
{{ url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4') }}
```

### `vardump`

Функция `vardump()` выводит на экран текущую переменную (а не в отладчике, как в `dump()`)

```twig
{% set my_array = {foo: 'bar', baz: 'qux'} %}
{{ vardump(my_array) }}
```

```text
Array
(
  "foo" => "bar"
  "baz" => "qux"
)
```

### `xss`

Проверяет строку на наличие уязвимостей XSS:

```twig
{{ xss('this string contains a <script>alert("hello");</script> XSS vulnerability') }}
```
