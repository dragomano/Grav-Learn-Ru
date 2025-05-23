---
description: Небольшое руководство по добавлению поля для загрузки файлов в Grav CMS.
---

# Как добавить поле для загрузки файлов

## Загрузка файлов

Вы можете добавить функцию загрузки файлов в чертежи страниц, конфигураций, плагинов и тем. Загрузка файлов всегда основана на Ajax и позволяет перетаскивать их с рабочего стола или выбирать их как обычные поля файла. Каждый раз, когда файл добавляется в поле, он автоматически загружается во временную папку и будет сохранен только при выполнении действия «Сохранить» (или «Отправить»).

Пример использования:

```yaml
custom_file:
  name: myfile
  type: file
  label: A Label
  destination: 'plugins://my-plugin/assets'
  multiple: true
  autofocus: false
  accept:
    - image/*
```

!!! info ""

    Чтобы добавить загрузку файла, вы должны иметь нижнюю команду рендеринга javascript в вашем базовом шаблоне Twig: `{{ assets.js('bottom') }}`

## Параметры

Поле файла имеет несколько доступных опций, от принятого типа или расширения MIME до разрешенного размера файла:

### Значения по умолчанию

```yaml
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
```

### `multiple`

```yaml
multiple: false # [false | true]
```

Как и в случае с обычным полем файла HTML5, когда включена опция `multiple`, оно позволяет загружать более одного файла. Этот параметр также связан с параметром [`limit`](#limit), который определяет, сколько максимум разрешено выбирать файлов за раз.

### `destination`

```yaml
destination: 'self@' # [<path> | <stream> | self@ | page@:<path>]
```

Место назначения — это место, где должны храниться загруженные файлы. Это может быть обычный `path` (относительно корня Grav), `stream` (такой, как `theme://images`), `self@`, или специальный префикс `page@:`. Вы также можете ссылаться на вложенную папку относительно текущей страницы с помощью `self@/path`.

!!! warning ""

    `self@` не допускается за пределами области страниц или объектов Flex, будет выдана ошибка. Если вы используете поле файла вне страницы или объекта Flex, вы всегда должны изменять настройку `destination`.

#### Примеры

1. Если нужно загрузить файлы в папку плагина `testing` (`user/plugins/testing`), местом назначения будет:
   ```yaml
   destination: 'plugins://testing'
   ```
2. Предполагая, что у нас есть элемент блога на маршруте `/blog/ajax-upload` (физическое местоположение — `user/pages/02.blog/ajax-upload`), с префиксом `page@:` адресатом будет:
   ```yaml
   destination: 'page@:/blog/ajax-upload'
   ```
3. Предположим, что текущая тема — это `antimatter`, и мы хотим загрузить в папку с ресурсами (физическое местоположение — `user/themes/antimatter/assets`), с потоком `theme` место назначения будет:
   ```yaml
   destination: 'theme://assets'
   ```

### `random_name`

```yaml
random_name: false # [false | true]
```

Когда включено `random_name`, загруженный файл будет переименован случайной строкой длиной **15** символов. Это полезно, если вы хотите хешировать свои загруженные файлы или если вы ищете способ уменьшить коллизию имен.

#### Пример

```php
'my_file.jpg' => 'y5bqsGmE1plNTF2.jpg'
```

### `avoid_overwriting`

```yaml
avoid_overwriting: false # [false | true]
```

Когда включен параметр `prevent_overwriting` и файл с таким же именем, как и загруженный, уже существует в `destination`, он будет переименован. Перед вновь загруженным файлом будет стоять префикс текущей даты и времени, соединенный тире.

#### Пример

```php
'my_file.jpg' => '20160901130509-my_file.jpg'
```

### `limit`

```yaml
limit: 10 # [1...X | 0 (unlimited)]
```

Когда включен параметр [`multiple`](#multiple), `limit` позволяет ограничить количество разрешенных файлов для отдельного поля. Если `multiple` не включен (не включен по умолчанию), `limit` автоматически возвращается к **1**.

Если для параметра `limit` установлено значение **0**, это означает, что нет ограничений на количество разрешенных файлов, которые могут быть загружены.

!!! tip ""

    Рекомендуется всегда проверять установленный лимит разрешенных файлов, которые могут быть загружены. Таким образом, у вас будет больше контроля над использованием ресурсов вашего сервера.

### `accept`

```yaml
accept:
  - 'image/*' # Array of MIME types and/or extensions. ['*'] for allowing any file.
```

Параметр accept позволяет использовать массив типа MIME, а также определения расширений. Все расширения должны начинаться с `.` (точки) плюс само расширение.

Кроме того, вы также можете разрешить любой файл, просто используя обозначение **\*** (звездочка) `accept: ['*']`.

#### Примеры

1. Чтобы разрешить только файлы `yaml` и` json`:
   ```yaml
   	accept:
   	- .yaml
   	- .json
   ```
2. Чтобы разрешить только изображения и видео:
   ```yaml
   	accept:
   	- 'image/*'
   	- 'video/*'
   ```
3. Чтобы разрешить любое изображение, любое видео и только файлы mp3:
   ```yaml
   	accept:
   	- 'image/*'
   	- 'video/*'
   	- .mp3
   ```
4. Чтобы разрешить любой файл:
   ```yaml
   	accept:
   	- '*'
   ```

### `filesize`

Максимальный размер файла ограничен:

1. полем `filesize:`...
2. Настройкой `files: filesize:` плагина формы `user/plugins/form.yaml`...
3. Конфигурацией PHP для параметра `upload_max_filesize` для отдельных загружаемых файлов и `post_max_size` для максимального общего размера сообщения формы.

#### Примеры

1. Чтобы ограничить конкретное поле до `5M`
   ```yaml
   custom_file:
   	name: myfile
   	type: file
   	label: A Label
   	destination: 'plugins://my-plugin/assets'
   	filesize: 5
   	accept:
   	- image/*
   ```
2. Чтобы ограничить все поля файла значением `5M`, отредактируйте файл `user/config/form.yaml`:
   ```yaml
   files:
   	multiple: false
   	limit: 10
   	destination: 'self@'
   	avoid_overwriting: false
   	random_name: false
   	filesize: 5
   	accept:
   	- 'image/*
   ```

### Обработка загрузки файлов в устаревшем режиме и ручное управление

Для базовой обработки файлов достаточно только определить поле. Файлы загружаются во временное расположение с помощью виджета Dropzone через XHR-запрос к серверу. При отправке формы файлы автоматически перемещаются из временного расположения в конечное. Однако вы можете использовать действие `upload: true` в блоке `process:`, чтобы вручную указать, на каком этапе рабочего процесса следует переместить эти файлы.

#### Примеры:

```yaml
process:
   upload: true
   message: 'Благодарим за ваши файлы'
   reset: true
```
