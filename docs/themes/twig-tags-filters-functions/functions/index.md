---
title: "Функции Twig"
description: "Функции Twig вызываются напрямую, а параметры передаются в скобках."
tags:
  - "Темы"
  - "Twig"
---

# Функции Twig

Функции Twig вызываются напрямую, а параметры передаются в скобках.

### array

Приводит значение к массиву.

```twig title="Twig"
{% set value = array(value) %}
```

```text title="Вывод"
(значение приведено к массиву)
```

### array_diff {: #array-diff }

Вычисляет разницу массивов.

```twig title="Twig"
{% set diff = array_diff(array1, array2...) %}
```

```text title="Вывод"
(массив, содержащий все элементы array1, которых нет в других массивах)
```

### array_key_value {: #array-key-value }

Функция `array_key_value` позволяет добавить пару ключ/значение в ассоциативный массив.

```twig title="Twig"
{% set my_array = {fruit: 'apple'} %}
{% set my_array = array_key_value('meat','steak', my_array) %}
{{ print_r(my_array)}}
```

```text title="Вывод"
[
    "fruit" => "apple"
    "meat" => "steak"
]
```

### array_key_exists {: #array-key-exists }

Обёртка для функции PHP `array_key_exists`, возвращающая, существует ли ключ в ассоциативном массиве.

```twig title="Twig"
{% set my_array = {fruit: 'apple', meat: 'steak'} %}
{{ array_key_exists('meat', my_array) }}
```

```text title="Вывод"
true
```

### array_intersect {: #array-intersect }

Функция `array_intersect` возвращает пересечение двух массивов или коллекций Grav.

```twig title="Twig"
{% set array_1 = {fruit: 'apple', meat: 'steak'} %}
{% set array_2 = {fish: 'tuna', meat: 'steak'} %}
{{ print_r(array_intersect(array_1, array_2)) }}
```

```text title="Вывод"
[
    "meat" => "steak"
]
```

### array_unique {: #array-unique }

Обёртка для PHP `array_unique()`, удаляющая дубликаты из массива.

```twig title="Twig"
{{ array_unique(['foo', 'bar', 'foo', 'baz']) }}
```

```text title="Вывод"
['foo', 'bar', 'baz']
```

### authorize

Авторизует аутентифицированного пользователя для просмотра ресурса. Принимает строку разрешения или массив строк разрешений.

```twig title="Twig"
{{ authorize(['admin.statistics', 'admin.super']) }}
```

```text title="Вывод"
true/false (в зависимости от разрешений пользователя)
```

### body_class {: #body-class }

Принимает массив классов и, если они не заданы в `body_classes`, проверяет, заданы ли они в конфигурации текущей темы.

```twig title="Twig"
set body_classes = body_class(['header-fixed', 'header-animated', 'header-dark', 'header-transparent', 'sticky-footer'])
```

```text title="Вывод"
header-fixed header-animated header-dark
```

### cron

Создаёт объект «Cron» из синтаксиса cron.

```twig title="Twig"
{{ cron("3 * * * *").getNextRunDate()|date(config.date_format.default) }}
```

```text title="Вывод"
2024-01-15 14:03:00
```

### dump

Принимает допустимую переменную Twig и выводит её в [панель отладки Grav](../../../advanced/debugging/index.md). Чтобы увидеть значения на вкладке сообщений, отладчик должен быть **включён**.

```twig title="Twig"
{% do dump(page.header) %}
```

```text title="Вывод"
(выводит переменную в панель отладки)
```

### debug

То же, что `dump()`.

### evaluate

Функцию evaluate можно использовать, чтобы вычислить строку как Twig:

```twig title="Twig"
{{ evaluate('grav.language.getLanguage') }}
```

```text title="Вывод"
en
```

### evaluate_twig {: #evaluate-twig }

Похожа на evaluate, но вычисляет и обрабатывает с помощью Twig.

```twig title="Twig"
{{ evaluate_twig('This is a twig variable: {% verbatim %}{{ foo }}{% endverbatim %}', {foo: 'bar'}) }}
```

```text title="Вывод"
This is a twig variable: bar
```

### exif

Выводит EXIF-данные изображения по пути к файлу. Для этого в `system.yaml` должно быть задано `media: auto_metadata_exif: true`. Например, в шаблоне Twig:

```twig title="Twig"
{% set image = page.media['sample-image.jpg'] %}
{% set exif = exif(image.filepath, true) %}
{{ exif.MaxApertureValue }}
```

```text title="Вывод"
40/10
```

Это выведет значение `MaxApertureValue`, заданное в камере, например «40/10». Всегда можно использовать `{{ dump(exif) }}`, чтобы показать все доступные данные в отладчике.

### get_cookie {: #get-cookie }

Получает значение cookie с помощью этой функции:

```twig title="Twig"
{{ get_cookie('your_cookie_key') }}
```

```text title="Вывод"
cookie_value
```

### get_type {: #get-type }

Возвращает тип переменной:

```twig title="Twig"
{{ get_type(page) }}
```

```text title="Вывод"
Grav\Common\Page\Page
```

### gist

Принимает ID Github Gist и создаёт подходящий код встраивания Gist.

```twig title="Twig"
{{ gist('bc448ff158df4bc56217') }}
```

```html title="Вывод"
<script src="https://gist.github.com/bc448ff158df4bc56217.js"></script>
```

### header_var {: #header-var }

`header_var($variable, $pages = null)`

Возвращает `page.header.<variable>`.

> [!WARNING] Предупреждение
> **ПРИМЕЧАНИЕ:** устарело с Grav 1.7. Следует использовать `theme_var`.

> [!WARNING] Предупреждение
> Логика поиска переменной изменилась, что может привести к неожиданным результатам:
> - Если вторым параметром передан массив страниц для поиска, будет использована только первая страница.
> - Если `<variable>` не определена в заголовке страницы, Grav будет искать переменную в дереве родительских страниц.
> - Если переменная всё ещё не найдена, Grav будет искать её в файле конфигурации темы.

При таких метаданных страницы:

```
---
title: Home
---
```

```twig title="Twig"
{{ header_var('title') }}
```

```text title="Вывод"
Home
```

### http_response_code {: #http-response-code }

Если передан `response_code`, возвращается предыдущий код состояния. Если `response_code` не передан, возвращается текущий код состояния. Оба значения по умолчанию равны 200, если функция используется в среде веб-сервера.

```twig title="Twig"
{% do http_response_code(404) %}
```

```text title="Вывод"
(устанавливает код ответа HTTP 404)
```

### isajaxrequest

Функцию `isajaxrequest()` можно использовать, чтобы проверить, установлен ли параметр заголовка `HTTP_X_REQUESTED_WITH`:

```twig title="Twig"
{{ isajaxrequest() }}
```

```text title="Вывод"
true/false
```

### json_decode {: #json-decode }

JSON можно декодировать, просто вызвав эту функцию:

```twig title="Twig"
{{ json_decode('{"first_name": "Guido", "last_name":"Rossum"}') }}
```

```text title="Вывод"
[
    "first_name" => "Guido"
    "last_name" => "Rossum"
]
```

### markdown_body {: #markdown-body }

Тело [Markdown-вывода](../../../content/markdown-output/index.md) страницы: основная область контента страницы в том виде, в каком её рендерит тема, с добавленным `# Title`, только если тема не вывела ни одного заголовка. По умолчанию — текущая страница. **Grav 2.1+**

```twig title="Twig"
{{ markdown_body(page) }}
```

```markdown title="Вывод"
# My Post

The post content, converted from the rendered HTML.
```

### markdown_frontmatter {: #markdown-frontmatter }

YAML-блок, открывающий [Markdown-вывод](../../../content/markdown-output/index.md) страницы: title, канонический URL, URL `.md`, язык, дата, описание и таксономия. По умолчанию — текущая страница. **Grav 2.1+**

```twig title="Twig"
{{ markdown_frontmatter(page) }}
```

```yaml title="Вывод"
---
title: 'My Post'
url: 'https://example.com/blog/my-post'
markdown: 'https://example.com/blog/my-post.md'
date: '2026-09-10'
---
```

### markdown_links {: #markdown-links }

Раздел навигации, закрывающий [Markdown-вывод](../../../content/markdown-output/index.md) страницы: родительская, предыдущая и следующая страницы и дочерние страницы, каждая по своему URL `.md`. Пусто, если у страницы их нет. По умолчанию — текущая страница. **Grav 2.1+**

```twig title="Twig"
{{ markdown_links(page) }}
```

```markdown title="Вывод"
---

## Navigation

- Parent: [Blog](https://example.com/blog.md)
- Next: [A Newer Post](https://example.com/blog/newer-post.md)
```

### markdown_output {: #markdown-output }

Весь документ [Markdown-вывода](../../../content/markdown-output/index.md) страницы: метаданные, тело и навигация в том виде, в каком их рендерит шаблон ядра `default.md.twig`. По умолчанию — текущая страница. **Grav 2.1+**

```twig title="Twig"
{%- autoescape false -%}
{{ markdown_output(page) }}
{%- endautoescape -%}
```

### markdown_url {: #markdown-url }

Абсолютный URL `.md` страницы, по которому отдаётся её [Markdown-вывод](../../../content/markdown-output/index.md). Главная страница ссылается по своему реальному маршруту, а не `/.md`. По умолчанию — текущая страница. **Grav 2.1+**

```twig title="Twig"
<link rel="alternate" type="text/markdown" href="{{ markdown_url(page)|e }}" />
```

```html title="Вывод"
<link rel="alternate" type="text/markdown" href="https://example.com/blog/my-post.md" />
```

### media_directory {: #media-directory }

Возвращает объект медиафайлов для произвольной папки. После получения с изображениями можно работать так же, как со страницами.

```twig title="Twig"
{{ media_directory('theme://images')['some-image.jpg'].cropResize(200,200).html }}
```

```html title="Вывод"
<img src="/user/themes/mytheme/images/some-image.jpg" width="200" height="200">
```

### nicefilesize

Выводит размер файла в удобном человекочитаемом формате.

```twig title="Twig"
{{ nicefilesize(612394) }}
```

```text title="Вывод"
598.04 KB
```

### nicenumber

Выводит число в удобном человекочитаемом формате.

```twig title="Twig"
{{ nicenumber(12430) }}
```

```text title="Вывод"
12K
```

### nicetime

Выводит дату в удобном человекочитаемом формате времени.

```twig title="Twig"
{{ nicetime(page.date) }}
```

```text title="Вывод"
1 month ago
```

### nonce_field {: #nonce-field }

Создаёт поле nonce безопасности Grav для формы с обязательным `action`:

```twig title="Twig"
{{ nonce_field('action') }}
```

```html title="Вывод"
<input type="hidden" name="nonce" value="abc123def456">
```

### of_type {: #of-type }

Проверяет тип переменной по параметру:

```twig title="Twig"
{{ of_type(page, 'string') }}
```

```text title="Вывод"
false
```

### pathinfo

Разбирает путь в массив.

```twig title="Twig"
{% set parts = pathinfo('/www/htdocs/inc/lib.inc.php') %}
{{ print_r(parts) }}
```

```text title="Вывод"
[
    "dirname" => "/www/htdocs/inc"
    "basename" => "lib.inc.php"
    "extension" => "php"
    "filename" => "lib.inc"
]
```

### print_r {: #print-r }

Выводит переменную в читаемом формате.

```twig title="Twig"
{{ print_r(page.header) }}
```

```text title="Вывод"
[
    "title" => "My Page"
    "published" => true
]
```

### random_string {: #random-string }

Создаёт случайную строку нужного количества символов. Особенно полезна для создания уникального id или ключа.

```twig title="Twig"
{{ random_string(10) }}
```

```text title="Вывод"
aBc123XyZ9
```

### unique_id {: #unique-id }

Создаёт случайную строку с настраиваемой длиной, префиксом и суффиксом. В отличие от встроенной функции PHP `uniqid()` и утилиты `random_string`, эта строка будет действительно уникальной и не конфликтующей.

```twig title="Twig"
{{ unique_id(9) }}
{{ unique_id(11, { prefix: 'user_' }) }}
{{ unique_id(13, { suffix: '.json' }) }}
```

```text title="Вывод"
a1b2c3d4e
user_a1b2c3d4e5f
a1b2c3d4e5f6g.json
```

### range

Создаёт массив, содержащий диапазон элементов с необязательным шагом.

```twig title="Twig"
{{ range(25, 300, 50) }}
```

```text title="Вывод"
[25, 75, 125, 175, 225, 275]
```

### read_file {: #read-file }

Простая функция для чтения файла по пути и вывода его содержимого.

```twig title="Twig"
{{ read_file('theme://README.md')|markdown }}
```

```html title="Вывод"
<h1>My Theme</h1>
<p>This <strong>theme</strong> for Grav...</p>
```

В целях безопасности `read_file` читает только файлы, проходящие три проверки; все они настраиваются в `security.read_file` в `system/config/security.yaml`:

- **`allowed_streams`** — схемы потоков, которые она разрешает. По умолчанию `theme`, `themes`, `page` и `user-data`. Путь в любом другом потоке (например, `plugins://` или `user://`) возвращает `false`, пока вы не добавите этот поток сюда.
- **`allowed_extensions`** — расширения файлов, которые она читает. По умолчанию только текстовые и контентные форматы: `md`, `markdown`, `txt`, `html`, `htm`, `css`, `json`, `csv`, `xml` и `svg`.
- **`max_size`** — максимальный размер файла в байтах (по умолчанию `1048576`, то есть 1 МБ). Файлы большего размера возвращают `false`, а не читаются в память. Установите `0`, чтобы отключить ограничение.

> [!NOTE] Примечание
> Добавлять исполняемые или секретные форматы (например, `php`, `yaml`, `env` или `htaccess`) в `allowed_extensions` не рекомендуется — это превращает `read_file` в способ утечки конфигурации и учётных данных в отрендеренный вывод.

### redirect_me {: #redirect-me }

Перенаправляет на выбранный URL.

```twig title="Twig"
{% do redirect_me('http://google.com', 304) %}
```

```text title="Вывод"
(перенаправляет на http://google.com со статусом 304)
```

### regex_filter {: #regex-filter }

Выполняет `preg_grep` по массиву с шаблоном регулярного выражения.

```twig title="Twig"
{{ regex_filter(['pasta', 'fish', 'steak', 'potatoes'], "/p.*/") }}
```

```text title="Вывод"
['pasta', 'potatoes']
```

### regex_replace {: #regex-replace }

Полезная обёртка для метода PHP [preg_replace()](https://php.net/manual/en/function.preg-replace.php), позволяющая выполнять сложные замены регулярными выражениями в тексте через эту функцию:

```twig title="Twig"
{{ regex_replace('The quick brown fox jumps over the lazy dog.', ['/quick/','/brown/','/fox/','/dog/'], ['slow','black','bear','turtle']) }}
```

```text title="Вывод"
The slow black bear jumps over the lazy turtle.
```

### regex_match {: #regex-match }

Полезная обёртка для метода PHP [preg_match()](https://php.net/manual/en/function.preg-match.php), позволяющая выполнять сложные совпадения регулярных выражений в тексте через эту функцию:

```twig title="Twig"
{{ regex_match('http://www.php.net/index.html', '@^(?:http://)?([^/]+)@i') }}
```

```text title="Вывод"
[
    0 => "http://www.php.net"
    1 => "www.php.net"
]
```

### regex_split {: #regex-split }

Полезная обёртка для метода PHP [preg_split()](https://php.net/manual/en/function.preg-split.php). Разбивает строку регулярным выражением через эту функцию:

```twig title="Twig"
{{ regex_split('hypertext language, programming', '/\\s*,\\s*/u') }}
```

```text title="Вывод"
['hypertext language', 'programming']
```

### repeat

Повторяет переданное значение определённое количество раз.

```twig title="Twig"
{{ repeat('blah ', 10) }}
```

```text title="Вывод"
blah blah blah blah blah blah blah blah blah blah
```

### string

Возвращает строку из значения. Если значение — массив, возвращает его в JSON-кодировке.

```twig title="Twig"
{{ string(23) }}
{{ string(['test' => 'x']) }}
```

```text title="Вывод"
"23"
{"test":"x"}
```

### svg_image {: #svg-image }

Возвращает содержимое SVG-изображения и при необходимости добавляет дополнительные классы. Даёт преимущества встроенного SVG без необходимости вставлять код прямо на страницу. Полезно для повторно используемых изображений, например иконок соцсетей.

```
{{ svg_image(path, classes, strip_style) }}
```

`strip_style` = удаляет встроенные стили SVG — полезно для стилизации через CSS-классы.

```twig title="Twig"
{{ svg_image('theme://images/something.svg', 'my-class-here mb-10', true) }}
```

```html title="Вывод"
<svg class="my-class-here mb-10" viewBox="0 0 24 24">...</svg>
```

### theme_var {: #theme-var }

`theme_var($variable, $default = null, $page = null)`

Получает переменную темы из заголовка страницы или, если она не найдена, из её родительских страниц, файла конфигурации темы или значения по умолчанию, если оно предоставлено:

```twig title="Twig"
{{ theme_var('grid-size') }}
```

```text title="Вывод"
1200
```

Сначала будет проверено `page.header.grid-size`; если там не задано, будет пройдено дерево родителей. Если всё ещё не найдено, будет проверено `theme.grid-size` из файла конфигурации темы.

Опционально можно передать значение по умолчанию как резервное:

```twig title="Twig"
{{ theme_var('grid-size', 1024) }}
```

```text title="Вывод"
1024 (если не найдено в другом месте)
```

### t

Переводит строку, как фильтр [`|t`](../filters/index.md#t).

```twig title="Twig"
{{ t('SITE_NAME') }}
```

```text title="Вывод"
Site Name
```

### ta

Работает так же, как фильтр [`|ta`](../filters/index.md#ta).

### tl

Переводит строку на конкретный язык. Подробнее см. в [документации по многоязычности](../../../content/multi-language/index.md#complex-translations).

```twig title="Twig"
{{ tl('SIMPLE_TEXT', ['fr']) }}
```

```text title="Вывод"
Texte simple
```

### url

Создаёт URL и преобразует любые URL-потоки PHP в допустимый HTML-ресурс. Можно передать значение по умолчанию на случай, если URL не удаётся разрешить.

```twig title="Twig"
{{ url('theme://images/logo.png')|default('http://www.placehold.it/150x100/f4f4f4') }}
```

```text title="Вывод"
/user/themes/mytheme/images/logo.png
```

### vardump

Функция `vardump()` выводит текущую переменную на экран (а не в отладчик, как `dump()`).

```twig title="Twig"
{% set my_array = {foo: 'bar', baz: 'qux'} %}
{{ vardump(my_array) }}
```

```text title="Вывод"
[
  "foo" => "bar"
  "baz" => "qux"
]
```

### xss

Позволяет вручную проверить строку на уязвимости XSS.

```twig title="Twig"
{{ xss('this string contains a <script>alert("hello");</script> XSS vulnerability') }}
```

```text title="Вывод"
this string contains a  XSS vulnerability
```

Функция `xss()` удаляет из строки вредоносный тег script.
