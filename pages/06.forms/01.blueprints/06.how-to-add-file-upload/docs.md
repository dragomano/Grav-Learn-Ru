---
title: 'Как добавить поле для загрузки файлов'
taxonomy:
    category: docs
---

### Загрузка файлов

Вы можете добавить функцию загрузки файлов в чертежи страниц, конфигураций, плагинов и тем. Загрузка файлов всегда основана на Ajax и позволяет перетаскивать их с рабочего стола или выбирать их как обычные поля файла. Каждый раз, когда файл добавляется в поле, он автоматически загружается во временную папку и будет сохранен только при выполнении действия «Сохранить» (или «Отправить»).

Пример использования:

[prism classes="language-yaml line-numbers"]
custom_file:
  name: myfile
  type: file
  label: A Label
  destination: 'user/plugins/my-plugin/assets'
  multiple: true
  autofocus: false
  accept:
    - image/*
[/prism]

! Чтобы добавить загрузку файла, вы должны иметь нижнюю команду рендеринга javascript в вашем базовом шаблоне Twig: `{{ assets.js('bottom') }}`

## Параметры

Поле файла имеет несколько доступных опций, от принятого типа или расширения MIME до разрешенного размера файла:

#### Значения по умолчанию

[prism classes="language-yaml line-numbers"]
custom_file:
  type: file
  label: A Label
  multiple: false
  destination: 'self@'
  random_name: false
  avoid_overwriting: false
  limit: 10
  accept:
    - image/*
[/prism]

#### `multiple`

[prism classes="language-yaml"]
multiple: false # [false | true]
[/prism]

Как и в случае с обычным полем файла HTML5, когда включена опция `multiple`, оно позволяет загружать более одного файла. Этот параметр также связан с параметром [`limit`](#limit), который определяет, сколько максимум разрешено выбирать файлов за раз.

#### `destination`

``` yaml
destination: 'self@' # [<path> | self@ | page@:<path> | theme@:<path>]
```

Место назначения - это место, где должны храниться загруженные файлы. Это может быть обычный `path` (относительно корня Grav),`self@` или один из специальных префиксов `page@:` и `theme@:`.

!! `self@` не допускается за пределами области страниц, будет выдана ошибка. Если вы используете поле файла вне страницы, вы всегда должны изменять настройку `destination`.

##### Примеры

1. Если нужно загрузить файлы в папку плагина `testing` (`user/plugins/testing`), местом назначения будет:

  [prism classes="language-yaml"]
  destination: 'user/plugins/testing'
  [/prism]

2. Предполагая, что у нас есть элемент блога на маршруте `/blog/ajax-upload` (физическое местоположение - `user/pages/02.blog/ajax-upload`), с префиксом `page@:` адресатом будет:

  [prism classes="language-yaml"]
  destination: 'page@:/blog/ajax-upload'
  [/prism]
3. Предположим, что текущая тема - это `antimatter`, и мы хотим загрузить в папку с ресурсами (физическое местоположение - `user/themes/antimatter/assets`), с префиксом `theme@:` место назначения будет:

   [prism classes="language-yaml"]
   destination: 'theme@:/assets'
   [/prism]

#### `random_name`

[prism classes="language-yaml"]
random_name: false # [false | true]
[/prism]

Когда включено `random_name`, загруженный файл будет переименован случайной строкой длиной **15** символов. Это полезно, если вы хотите хешировать свои загруженные файлы или если вы ищете способ уменьшить коллизию имен.

##### Пример
[prism classes="language-php"]
'my_file.jpg' => 'y5bqsGmE1plNTF2.jpg'
[/prism]

#### `avoid_overwriting`

[prism classes="language-yaml"]
avoid_overwriting: false # [false | true]
[/prism]

Когда включен параметр `prevent_overwriting` и файл с таким же именем, как и загруженный, уже существует в `destination`, он будет переименован. Перед вновь загруженным файлом будет стоять префикс текущей даты и времени, соединенный тире.

##### Пример
[prism classes="language-php"]
'my_file.jpg' => '20160901130509-my_file.jpg'
[/prism]

#### `limit`

[prism classes="language-yaml"]
limit: 10 # [1...X | 0 (unlimited)]
[/prism]

Когда включен параметр [`multiple`](#multiple), `limit` позволяет ограничить количество разрешенных файлов для отдельного поля. Если `multiple` не включен (не включен по умолчанию), `limit` автоматически возвращается к **1**.

Если для параметра `limit` установлено значение **0**, это означает, что нет ограничений на количество разрешенных файлов, которые могут быть загружены.

!! Рекомендуется всегда проверять установленный лимит разрешенных файлов, которые могут быть загружены. Таким образом, у вас будет больше контроля над использованием ресурсов вашего сервера.

#### `accept`

[prism classes="language-yaml line-numbers"]
accept:
  - 'image/*' # Array of MIME types and/or extensions. ['*'] for allowing any file.
[/prism]

Параметр accept позволяет использовать массив типа MIME, а также определения расширений. Все расширения должны начинаться с `.` (точки) плюс само расширение.

Кроме того, вы также можете разрешить любой файл, просто используя обозначение __*__ (звездочка) `accept: ['*']`.

##### Примеры

1. Чтобы разрешить только файлы `yaml` и` json`:
   [prism classes="language-yaml line-numbers"]
     accept:
       - .yaml
       - .json
   [/prism]
2. Чтобы разрешить только изображения и видео:
   [prism classes="language-yaml line-numbers"]
     accept:
       - 'image/*'
       - 'video/*'
   [/prism]
3. Чтобы разрешить любое изображение, любое видео и только файлы mp3:
   [prism classes="language-yaml line-numbers"]
     accept:
       - 'image/*'
       - 'video/*'
       - .mp3
   [/prism]
4. Чтобы разрешить любой файл:
   [prism classes="language-yaml line-numbers"]
     accept:
       - '*'
   [/prism]

#### `filesize`

Максимальный размер файла ограничен:

1. полем `filesize:`...

2. Настройкой `files: filesize:` плагина формы `user/plugins/form.yaml`...

3. Конфигурацией PHP для параметра `upload_max_filesize` для отдельных загружаемых файлов и `post_max_size` для максимального общего размера сообщения формы.

##### Примеры

1. Чтобы ограничить конкретное поле до `5M`
   [prism classes="language-yaml line-numbers"]
   custom_file:
     name: myfile
     type: file
     label: A Label
     destination: 'user/plugins/my-plugin/assets'
     filesize: 5
     accept:
       - image/*
   [/prism]

2. Чтобы ограничить все поля файла значением `5M`, отредактируйте файл `user/config/form.yaml`:
   [prism classes="language-yaml line-numbers"]
   files:
     multiple: false
     limit: 10
     destination: '@self'
     avoid_overwriting: false
     random_name: false
     filesize: 5
     accept:
       - 'image/*
   [/prism]
