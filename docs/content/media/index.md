---
description: Список поддерживаемых медиа-файлов в Grav CMS. Режимы отображения. Ссылки и лайтбоксы.
---

# Медиа

При создании контента в **Grav** вам часто требуется отображать различные типы мультимедиа, такие как **изображения**, **видео** и различные другие **файлы**. Эти файлы автоматически обнаруживаются и обрабатываются Grav, и они становятся доступными для использования на любой странице. Это особенно удобно, потому что затем вы можете использовать встроенные функции страницы для использования миниатюр, доступа к метаданным и динамического изменения мультимедиа (например, изменения размера изображений, установки размера отображения для видео и т. д.) По мере необходимости.

Grav использует **систему интеллектуального кэширования**, которая при необходимости автоматически создает в кэше копии динамически генерируемых носителей. Таким образом, все последующие запросы могут использовать кэшированную версию вместо того, чтобы создавать мультимедиа заново.

## Поддерживаемые медиафайлы

Следующие типы медиафайлов изначально поддерживаются Grav. Дополнительная поддержка мультимедийных файлов и потоковых вставок может быть добавлена ​​через плагины.

| Тип медиа                 | Тип файла                                                                                                                                                              |
| :------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Изображения               | jpg, jpeg, png                                                                                                                                                         |
| Аудио                     | mp3, wav, wma, ogg, m4a, aiff, aif                                                                                                                                     |
| Анимированные изображения | gif                                                                                                                                                                    |
| Векторные изображения     | svg                                                                                                                                                                    |
| Видео                     | mp4, mov, m4v, swf, flv, webm, ogv                                                                                                                                     |
| Данные / Информация       | txt, doc, docx, html, htm, pdf, zip, gz, 7z, tar, css, js, json, xml, xls, xlt, xlm, xlsm, xld, xla, xlc, xlw, xll, ppt, pps, rtf, bmp, tiff, mpeg, mpg, mpe, avi, wmv |

Полный список поддерживаемых типов MimeType можно найти в файле `system/config/media.yaml`. Если есть тип mime, который в настоящее время не поддерживается, вы можете просто создать свой собственный `user/config/media.yaml` и добавить его туда. Просто убедитесь, что вы используете тот же формат, что и исходный файл `system`. Самый простой подход — скопировать весь исходный файл и внести необходимые изменения.

## Месторасположение медиафайлов

Обычно вы используете мультимедийный файл на странице, поэтому просто поместите файл в папку страницы, и вы можете ссылаться на него в разметке страницы, например:

```markdown
![my image](image.jpg)
```

Если вы хотите поместить все изображения в одну папку, вы можете поместить их в папку `user/pages/images`. Таким образом, в Twig вы сможете связаться с ними через:

```twig
{% set my_image = page.find('/images').media['my-image.jpg'] %}
```

а также вы можете легко найти их с помощью Markdown и выполнить с ними операции:

```markdown
![my image](/images/my-image.jpg?cropResize=300,300)
```

В качестве альтернативы вы можете поместить их в свою тему, так как это легко доступно через ссылки CSS или из файла разметки с помощью потока `theme://`:

```markdown
![my image](theme://images/theme-image.jpg)
```

Другой вариант - `user/images`, где вы можете использовать поток `image://` для доступа к ним:

```markdown
![my image](image://my-image.jpg)
```

Фактически вы можете использовать любой поток, включая любую папку внутри `user/` через поток `user://`:

```markdown
![my image](user://themes/mytheme/images/my-image.jpg)
```

Вы также можете делать то же самое с помощью объекта Twig `Media`:

```twig
{{ media['user://themes/mytheme/images/my-image.jpg'].html()|raw }}
```

!!! danger ""

    В Grav есть папка `/images`. Не помещайте в эту папку свои собственные изображения, поскольку в ней хранятся автоматически сгенерированные Grav кэшированные изображения.

Вы также можете поместить все медиафайлы в отдельную папку, чтобы все они были доступны за один раз. Например, вы можете захотеть сохранить все свои MP3-файлы в папке `user/pages/mp3s` (не отображается) и поместить имя MP3-файла, связанного с определенной страницей, в поле заголовка, называемое `thistrack`. Если вы затем захотите получить доступ к файлу для определенной страницы и воспроизвести его с помощью аудиоэлемента HTML5, вам понадобится следующий код:

```twig
<audio controls>
  <source src="{{ page.find('/mp3s').media[page.header.thistrack~'.mp3']|e }}">
</audio>
```

## Режимы отображения

Grav предоставляет несколько различных режимов отображения для каждого типа медиа-объекта.

| Режим     | Объяснение                                                                                  |
| :-------- | :------------------------------------------------------------------------------------------ |
| source    | Визуальное представление самого носителя, то есть фактического изображения, видео или файла |
| text      | Текстовое представление медиа                                                               |
| thumbnail | Миниатюра для этого медиа-объекта                                                           |

!!! danger ""

    Носители типа **Data/Information** не поддерживают режим «источник», по умолчанию они будут использовать режим `text`, если другой режим не выбран явно.

## Расположение эскиза

Grav будет искать ваши эскизы в трех местах.

1. В той же папке, что и мультимедийный файл: `[media-name].[media-extension].thumb.[thumb-extension]` где `media-name` и `media-extension` - это соответственно имя и расширение исходного медиафайла, а `thumb-extension` - это любое расширение, поддерживаемое типом медиафайла `image`. Примеры: `my_video.mp4.thumb.jpg` и `my-image.jpg.thumb.png`.
   **Только для изображений!** Сама картинка будет использована в качестве эскиза.
2. Ваша пользовательская папка: `user/images/media/thumb-[media-extension].png`, где `media-extension` - это расширение исходного медиафайла. Примеры: thumb-mp4.png и thumb-jpg.jpg.
3. Системная папка: `system/images/media/thumb-[media-extension].png`, где `media-extension` - это расширение исходного медиафайла. **Эскизы в системных папках предоставлены Grav заранее.**

!!! warning ""

    Вы также можете вручную выбрать нужный эскиз с помощью действий, описанных ниже.

## Ссылки и лайтбоксы

Вышеупомянутые режимы отображения также можно использовать в сочетании со ссылками и лайтбоксов, которые будут объяснены более подробно позже. Однако важно отметить следующее:

!!! danger ""

    Grav не предоставляет функциональность лайтбоксов из коробки, для этого вам понадобится плагин. Для этого вы можете использовать [плагин FeatherLight](https://github.com/getgrav/grav-plugin-featherlight).

Когда вы используете мультимедийные функции Grav для рендеринга лайтбокса, всё, что делает Grav, выводит тег **якоря**, который имеет некоторые атрибуты для чтения плагином лайтбокса. Если вы заинтересованы в использовании библиотеки лайтбоксов, которой нет в нашем репозитории плагинов, или вы хотите создать свой собственный плагин, вы можете использовать приведенную ниже таблицу в качестве справки.

| Атрибут     | Объяснение                                                                                                            |
| :---------- | :-------------------------------------------------------------------------------------------------------------------- |
| rel         | Простой индикатор того, что это не обычная ссылка, а ссылка лайтбокса. Значение всегда будет `lightbox`.              |
| href        | URL-адрес самого медиаобъекта.                                                                                        |
| data-width  | Ширина, которую пользователь запросил для этого лайтбокса.                                                            |
| data-height | Высота, которую пользователь запросил для этого лайтбокса.                                                            |
| data-srcset | В случае графических носителей это содержит строку `srcset`. ([Дополнительная информация](#adaptivnye-izobrazheniia)) |

## Общие действия

Grav использует паттерн **Строитель** при работе с медиа, поэтому вы можете выполнять **несколько действий** с определённым медиа. Некоторые действия доступны для каждого вида среды, в то время как другие зависят от конкретной среды.

Эти действия доступны для всех типов медиа.

#### url

!!! warning ""

    Этот метод предназначен только для использования в шаблонах **Twig**, отсюда отсутствие синтаксиса Markdown.

Это возвращает **необработанный URL-путь** к медиа.

=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].url|e }}
	`
=== "HTML"
`
	/images/a/f/2/8/f/af28f2ad724f1e05248ac8dd518b2a5789c6cd41-sample-image.jpg
	`

#### html

!!! warning ""

    В Markdown этот метод неявно вызывается при использовании синтаксиса `![]`.

Действие `html` выведет действительный HTML-тег для мультимедиа в зависимости от текущего режима отображения.

=== "Markdown"
`markdown
	![Some ALT text](sample-image.jpg?classes=myclass "My title")
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].html('My title', 'Some ALT text', 'myclass')|raw }}
	`
=== "HTML"
`
	<img title="My title" alt="Some ALT text" class="myclass" src="/images/a/f/2/8/f/af28f2ad724f1e05248ac8dd518b2a5789c6cd41-sample-image.jpg" />
	`

#### display

Используйте это действие для переключения между различными режимами отображения, которые предоставляет Grav. Как только вы переключите режим отображения, все предыдущие действия будут сброшены. Исключениями из этого правила являются действия `lightbox` и `link`, а также любые действия, которые использовались до этих двух.

Например, эскиз, полученный в результате вызова `page.media['sample-image.jpg'].sepia().display('thumbnail').html()`, не будет иметь примененного действия `sepia()`, но `page.media['sample-image.jpg'].display('thumbnail').sepia().html()` будет применено действие `sepia()`.

!!! info ""

    После того, как вы переключитесь в режим эскизов, вы будете манипулировать изображением. Это означает, что даже если текущим медиафайлом является видео, вы можете использовать все действия с изображениями на миниатюре.

#### link

Превратите свой медиа-объект в ссылку. Все действия, которые вы вызываете до `link()`, будут применены к цели ссылки, в то время как любые действия, вызываемые после, будут применены к тому, что отображается на вашей странице.

!!! warning ""

    После вызова `link()`, Grav автоматически переключит режим отображения на **thumbnail**.

В следующем примере будет отображена текстовая ссылка (`display('text')`) на версию файла `sample-image.jpg` сепии:

=== "Markdown"
`markdown
	![Image link](sample-image.jpg?sepia&link&display=text)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].sepia().link().display('text').html('Image link')|raw }}
	`
=== "HTML"
`
	<a href="/images/3/7/7/1/c/3771c3fabb6d7bc0035dd119281718f9143c4653-sample-image.jpg"><img title="Image link" alt="" src="/images/a/f/2/8/f/af28f2ad724f1e05248ac8dd518b2a5789c6cd41-sample-image.jpg" /></a>
	`

#### Только Cache

Grav можно настроить на кэширование всех файлов изображений, это может увеличить скорость обслуживания файлов. Тем не менее, изображения будут проходить через систему манипулирования изображениями Grav, что может привести к значительно большему размеру файла для изображений, которые уже были оптимизированы до Grav. Манипуляции с изображениями можно обойти.

Включите `cache_all` в `system/config/system.yaml`

```yaml
images:
  default_image_quality: 85
  cache_all: false
```

Отключите манипуляцию с изображениями с помощью параметра `cache`.

=== "Markdown"
`markdown
	![](sample-image.jpg?cache)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cache.html()|raw }}
	`
=== "HTML"
`
	<img alt="" src="/images/a/f/2/8/f/af28f2ad724f1e05248ac8dd518b2a5789c6cd41-sample-image.jpg" />
	`

#### lightbox

Действие лайтбокса по сути такое же, как действие ссылки, но с некоторыми дополнениями. Как объяснялось выше ([Ссылки и лайтбоксы](#Ссылки-и-лайтбоксы)), действие лайтбокса не будет делать ничего, кроме создания ссылки с некоторыми дополнительными атрибутами. Он отличается от действия ссылки тем, что добавляет атрибут `rel="lightbox"` и принимает атрибуты `width` и `height`.

Если возможно (в настоящее время только в случае изображений), Grav изменит размер вашего мультимедиа до требуемой ширины и высоты. В противном случае он просто добавит к ссылке атрибуты `data-width` и `data-height`.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?lightbox=600,400&resize=200,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].lightbox(600,400).resize(200,200).html('Sample Image')|raw }}
	`
=== "HTML"
`
	<a rel="lightbox" data-width="600" data-height="400" href="/images/b/5/e/b/3/b5eb31744b96349b1a711697692b897624202cb1-sample-image.jpg"><img title="Sample Image" alt="" src="/images/4/5/5/e/4/455e41587c2cd25f34cfdccd8ab5078707aabe6b-sample-image.jpg" /></a>
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/6/e/3/8/8/6e388bab69f2d1bc5c0d3cd80ef317e9d1a343ed-sample-image.jpg)

#### thumbnail

Вручную выберите миниатюру, которую должен использовать Grav. Вы можете выбрать между `page` и `default` для любого типа медиа, а также `media` для графических медиа, если вы хотите использовать сам медиаобъект в качестве миниатюры.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?thumbnail=default&display=thumbnail)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].thumbnail('default').display('thumbnail').html('Sample Image')|raw }}
	`
=== "HTML"
`
	<img title="Sample Image" alt="" src="/system/images/media/thumb-jpg.png" />
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/system/images/media/thumb-jpg.png)

#### attribute

Это добавляет дополнительный атрибут HTML к выходным данным.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?attribute=myattribute,myvalue)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].attribute('myattribute', 'myvalue').html('Sample Image')|raw }}
	`
=== "HTML"
`
	<img myattribute="myvalue" title="Sample Image" alt="" src="/images/a/f/2/8/f/af28f2ad724f1e05248ac8dd518b2a5789c6cd41-sample-image.jpg" />
	`

## Действия с изображениями

#### resize

Изменение размера делает именно то, что вы ожидаете. `resize` позволяет вам создать новое изображение на основе «ширины» и «высоты». Соотношение сторон сохраняется, и новое изображение будет содержать пустые области **необязательного** цвета фона, представленного как «шестнадцатеричное значение», например `0xffffff`. Параметр фона является необязательным, и, если он не указан, по умолчанию будет установлено значение **прозрачное**, если изображение в формате PNG, или **белое**, если оно в формате JPEG.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?resize=400,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].resize(400, 200).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/f/8/2/e/8/f82e84dcb05f4e4abc0337d77f766863a4864658-sample-image.jpg)

#### forceResize

Изменяет размер изображения на ширину и высоту, как указано. `forceResize` не будет учитывать исходное соотношение сторон и растянет изображение по мере необходимости, чтобы оно соответствовало новому размеру изображения.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?forceResize=200,300)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].forceResize(200, 300).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/4/5/a/6/d/45a6d20510669bf09fc1f89b470ec8508851f1e3-sample-image.jpg)

#### cropResize

`cropResize` изменяет размер изображения до меньшего или большего размера в зависимости от ширины и высоты. Соотношение сторон сохраняется, и размер нового изображения будет изменен, чтобы поместиться в ограничивающей рамке, как описано с помощью предоставленных `width` и `height`. Другими словами, любая область фона, которую вы видите при обычном изменении размера, обрезается.

Например, если у вас есть изображение размером `640` x `480` и вы выполняете над ним действие `cropResize(100, 100)`, вы получите изображение размером `100` x` 75`.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropResize=300,300)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropResize(300, 300).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/c/6/f/2/a/c6f2a4797dd65154696266da9292f3280240de79-sample-image.jpg)

#### crop

`crop` не изменит размер изображения вообще, он просто обрежет исходное изображение так, чтобы только часть ограничивающего прямоугольника, описываемая значениями `width` и `height`, происходила из местоположений `x` и `y` и использовалась для создания нового изображения.

Например, изображение размером `640` x `480` с `crop(0, 0, 400, 100)` создаст изображение шириной `400` и высотой `100`, исходящее из левого верхнего угла (`0, 0`).

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?crop=100,100,300,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].crop(100,100,300,200).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/f/1/8/9/a/f189a48ef158968a6e707cb65f6fa4167e3513ab-sample-image.jpg)

#### cropZoom

Подобно обычному `cropResize`, `cropZoom` также принимает значения `width` и `height`, но **изменяет размер и обрезает** изображение, чтобы в результате был получен именно тот размер, который вы запрашивали. Соотношение сторон сохраняется, но части изображения могут быть обрезаны, однако результирующее изображение будет центрировано.

!!! warning ""

    Основное различие между **cropResize** и **cropZoom** заключается в том, что в параметре cropResize размер изображения изменяется с сохранением соотношения сторон, так что отображается всё изображение, а любое дополнительное пространство считается фоном.

При использовании **cropZoom** размер изображения изменяется так, что фон не виден, а дополнительная область изображения за пределами нового размера изображения обрезается.

Например, если у вас есть изображение размером `640` x` 480` и вы выполняете действие `cropZoom(400, 100)`, результирующее изображение будет изменено до `400` x `300`, а затем будет обрезана высота. В результате получается изображение `400` x `100`.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=600,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(600,200).html()|raw }}
	`

!!! warning ""

    Люди, знакомые с использованием `zoomCrop` для этой цели, обнаружат, что он также работает в Grav.

##### Результат:

![Sample Image](https://learn.getgrav.org/images/d/9/2/b/f/d92bfdaee70366a7390266cc163e6c1cbf085b6b-sample-image.jpg)

#### quality

Динамически позволяет установить значение **процента сжатия** для изображения от `0` до `100`. Меньшее число означает меньшее качество, где `100` означает максимальное качество.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&quality=25)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).quality(25).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/4/d/3/7/a/4d37ada8680de55cf46d1cb9b78217463755f765-sample-image.jpg)

#### negate

Применяет **отрицательный фильтр** к изображению с инвертированными цветами.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&negate)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).negate.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/3/4/f/8/0/34f801552d1959142ff0a8af90d1ef0b7ffca31b-sample-image.jpg)

#### brightness

Применяет **фильтр яркости** к изображению со значением от `-255` до `+255`. Большие отрицательные числа сделают изображение темнее, а большие положительные числа сделают изображение ярче.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&brightness=-100)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).brightness(-100).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/7/4/2/3/b/7423b7e6ff834ef259c41a05134bf6dd954a943c-sample-image.jpg)

#### contrast

Это применяет **фильтр контраста** к изображению со значением от `-100` до `+100`. Большие отрицательные числа увеличивают контраст, а большие положительные числа уменьшают контраст.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&contrast=-50)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).contrast(-50).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/3/1/4/1/e/3141e4a436373d0d14b30c81f5fd67e420de4546-sample-image.jpg)

#### grayscale

Это обрабатывает изображение с помощью **фильтра оттенков серого**.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&grayscale)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).grayscale.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/9/b/8/d/f/9b8dfb148124c5623c59d1a63c006e2f7eb066ce-sample-image.jpg)

#### emboss

Это обрабатывает изображение с помощью **фильтра тиснения**.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&emboss)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).emboss.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/4/c/e/2/1/4ce21377a2ee4a7749535dc82bd3521f48e4e117-sample-image.jpg)

#### smooth

Это применяет **фильтр сглаживания** к изображению на основе параметра сглаживания `value` от `-10` до `10`.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&smooth=5)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).smooth(5).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/5/5/8/0/3/55803bc6242fe0f0eb7602c4376c3643880beca9-sample-image.jpg)

#### sharp

Это применяет **фильтр повышения резкости** к изображению.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&sharp)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).sharp.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/d/1/c/b/8/d1cb87797f40ca52e4c8010d4e95e2ed63e2f664-sample-image.jpg)

#### edge

Это применяет **фильтр обнаружения краев** к изображению.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&edge)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).edge.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/2/c/3/4/7/2c3473fc46d715d58d6d54978c896f63a6c737e6-sample-image.jpg)

#### colorize

Вы можете раскрасить изображение на основе настройки значений `red`, `green` и `blue` для изображения от `-255` до `+255` для каждого цвета.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&colorize=100,-100,40)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).colorize(100,-100,40).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/8/6/5/b/9/865b9c011421966a2e11320c3b84b7c98139c87a-sample-image.jpg)

#### sepia

Это применяет **фильтр сепии** к изображению для создания винтажного вида.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&sepia)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).sepia.html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/b/2/1/5/3/b2153a55ad179282856a085973af657bde607e95-sample-image.jpg)

#### gaussianBlur

**Размывает** изображение с помощью фактора, который определяет, как часто фильтр размытия применяется к изображению. По умолчанию 1 раз.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?gaussianBlur=3)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].gaussianBlur(3).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/8/f/d/7/c/8fd7ce172594ef49d2307c1b405c68360d714380-sample-image.jpg)

#### rotate

**Поворачивает** изображение на заданный `угол` против часовой стрелки, отрицательные значения вращают по часовой стрелке.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&rotate=-90)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).rotate(-90).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/5/0/f/7/f/50f7f806aedb26a4bdba9c9b4b8615b900cd0098-sample-image.jpg)

#### flip

**Переворачивает** изображение в заданном направлении. Оба параметра могут иметь значение `0|1`. Оба значения `0` эквивалентны отсутствию переворачивания в любом направлении.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?cropZoom=300,200&flip=0,1)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].cropZoom(300,200).flip(0,1).html()|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/8/b/2/d/7/8b2d7799726efac9caceeb2e867a5b49c327eb63-sample-image.jpg)

#### fixOrientation

Исправляет ориентацию изображения при повороте через данные EXIF ​​(применимо к изображениям jpeg, снятым с помощью телефонов и камер).

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?fixOrientation)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].fixOrientation().html()|raw }}
	`

#### loading

Атрибут `loading` на изображениях дает авторам контроль над тем, когда браузер должен начать загрузку ресурса. Значение этого атрибута может быть одним из следующих `auto` (по умолчанию), `lazy`, `eager`.
Значение может быть установлено в `system.images.defaults.load` в качестве значения по умолчанию, или в качестве значения для изображения в разметке Markdown с помощью параметра `?loading=lazy`
Когда выбрано значение `auto`, атрибут `loading` не добавляется и браузер определит, какую стратегию использовать.

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?loading=lazy)
	`
=== "Twig"
```twig
{# Using default value as defined in 'config.system.images.defaults.loading' #}
{{ page.media['sample-image.jpg'].loading.html('Sample Image')|raw }}

    {# Using explicit value #}
    {{ page.media['sample-image.jpg'].loading('lazy').html('Sample Image')|raw }}
    ```

=== "HTML"
`
	<img loading="lazy" title="Sample Image"  src="/images/e/f/1/0/5/ef10554cd3a99f2e65136e79dce170d4f8a7a1b9-sample-image.jpg" />
	`

## Действия с объектами

#### resize

Поскольку PHP не может обрабатывать динамическое изменение размера этих типов носителей, действие resize будет только гарантировать, что атрибут `width` и `height` или `data-width` и `data-height` установлены в вашем теге `<img>`/`<video>` или `<a>` соответственно. Это означает, что ваше изображение или видео будет отображаться в требуемом размере, но фактическое изображение или видеофайл не будут преобразованы каким-либо образом.

=== "Markdown"
`markdown
	![Sample Trailer](sample-trailer.mov?resize=400,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-trailer.mov'].resize(400, 200).html('Sample Trailer') }}
	`
=== "HTML"
`
	<video controls="1" style="width: 400px;height: 200px;" title="Sample Trailer" alt=""><source src="/user/pages/02.content/07.media/sample-trailer.mov?loading=auto">Your browser does not support the video tag.</video>
	`

#### Примеры

Некоторые примеры этого:

=== "Векторное изображение"
`markdown
	![Sample Vector](sample-vector.svg?resize=300,300)
	`
<img alt="Sample Vector" src="https://learn.getgrav.org/user/pages/02.content/07.media/sample-vector.svg" width="300" height="300" />
=== "Анимированное изображение"
`markdown
	![Animated Gif](sample-animated.gif?resize=300,300)
	`
<img alt="Animated Gif" src="https://learn.getgrav.org/user/pages/02.content/07.media/sample-animated.gif" width="300" height="300" />
=== "Видео"
`markdown
	![Sample Trailer](sample-trailer.mov?resize=400,200)
	`
<video controls="1" style="width: 400px;height: 200px;" title="Sample Trailer"><source src="https://learn.getgrav.org/user/pages/02.content/07.media/sample-trailer.mov?loading=auto">Your browser does not support the video tag.</video>

## Действия с аудио

Аудио медиа будет отображать аудио ссылку HTML5:

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].html()|raw }}
	`

##### Результат:

<audio controls="1" title="Hal 9000: I'm Sorry Dave"><source src="https://learn.getgrav.org/user/pages/02.content/07.media/hal9000.mp3">Your browser does not support the audio tag.</audio>

#### controls

Позволяет явно устанавливать или удалять элементы управления HTML5 по умолчанию. Передача `0` скрывает элементы управления браузером для воспроизведения, громкости и т. д.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?controls=0)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].controls(0)|raw }}
	`
=== "HTML"
`
	<audio controls="0" title="Hal 9000: I'm Sorry Dave"><source src="pages/02.content/07.media/hal9000.mp3">Your browser does not support the audio tag.</audio>
	`

#### preload

Отвечает за предварительную загрузку контента, по умолчанию `auto`. Допустимые параметры: `auto`, `metadata` и `none`.

!!! warning ""

    <q cite="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio#attr-preload">Если не установлен, его значение по умолчанию определяется браузером (т. е. каждый браузер может иметь собственное значение по умолчанию). В спецификации рекомендуется установить значение <code>metadata</code>.</q>

!!! warning ""

    Атрибут `preload` игнорируется, если присутствует `autoplay`.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?preload=metadata)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].preload('metadata')|raw }}
	`

#### autoplay

Позволяет установить, будет ли аудио воспроизводиться автоматически при загрузке страницы. Если не задано, по умолчанию используется `false`.

!!! warning ""

    Если в заданном элементе `audio` присутствуют и `autoplay`, и `preload`, `preload` игнорируется.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?autoplay=1)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].autoplay(1)|raw }}
	`

#### controlsList

Позволяет установить свойство `controlsList`, которое принимает одно или несколько из трех возможных значений: `nodownload`, `nofullscreen` и `noremoteplayback`.

!!! warning ""

    Если вы устанавливаете более одного параметра в markdown, разделяйте их тире (`-`). Они будут заменены пробелами в выходном HTML.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?controlsList=nodownload-nofullscreen-noremoteplayback)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].controlsList('nodownload nofullscreen noremoteplayback')|raw }}
	`

#### muted

Позволяет установить, будет ли звук отключен при загрузке. Если не задано, по умолчанию используется `false`.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?muted=1)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].muted(1)|raw }}
	`

#### loop

Позволяет установить, будет ли звук зацикливаться после завершения воспроизведения. Если не задано, по умолчанию используется `false`.

=== "Markdown"
`markdown
	![Hal 9000: I'm Sorry Dave](hal9000.mp3?loop=1)
	`
=== "Twig"
`twig
	{{ page.media['hal9000.mp3'].loop(1)|raw }}
	`

## Действия с файлами

На данный момент Grav не предоставляет никаких настраиваемых действий с файлами, и мы не планируем их добавлять. Если вы что-то придумаете, свяжитесь с нами.

=== "Markdown"
`markdown
	[Текстовый файл](acronyms.txt)
	`
=== "Twig"
`twig
	<a href="{{ page.media['acronyms.txt'].url()|raw }}">Текстовый файл</a>
	`

##### Результат:

[Текстовый файл](https://learn.getgrav.org/content/media/acronyms.txt)

### Комбинации

Как видите: Grav предоставляет несколько мощных функций для работы с изображениями, которые упрощают работу с изображениями! Настоящая сила приходит, когда вы комбинируете несколько эффектов и производите очень сложные динамические манипуляции с изображениями. Например, это абсолютно верно:

=== "Markdown"
`markdown
	![Sample Image](sample-image.jpg?negate&lightbox&cropZoom=200,200)
	`
=== "Twig"
`twig
	{{ page.media['sample-image.jpg'].negate.lightbox.cropZoom(200,200)|raw }}
	`

##### Результат:

![Sample Image](https://learn.getgrav.org/images/5/9/a/5/b/59a5bca6c32648f2e976b41e92f2b1b6cd4f40b6-sample-image.jpg)

#### Сброс нескольких вызовов на одно и то же изображение

Когда вы обращаетесь к одному и тому же изображению несколько раз на одной странице, действия, которые вы предоставили для изображения, не сбрасываются по умолчанию. Поэтому, если вы измените размер изображения и выведите HTML-код, а затем на той же странице просто выведите URL-адрес изображения, вы также получите URL-адрес измененного изображения. Вы, вероятно, ожидали URL-адрес исходного изображения.

Чтобы бороться с этим, вы можете сбросить действия с изображениями, передав `false` методу `url()`:

```twig
{% for item in page.header.gallery %}
    {% set image = page.media[item.src].cropZoom(800, 600).quality(70) %}
    <a href="{{ image.url(false) }}">
      <img src="{{ image.url }}" alt="{{ item.alt }}" title="{{ item.title }}" />
    </a>
{% endfor %}
```

### Адаптивные изображения

#### Дисплеи с более высокой плотностью

Grav имеет встроенную поддержку отзывчивых изображений для дисплеев с более высокой плотностью (например, экранов **Retina**). Grav выполняет это с помощью атрибута `srcset` в [HTML элементе изображения](https://html.spec.whatwg.org/multipage/embedded-content.html#the-picture-element). Хорошая статья для чтения, если вы хотите лучше понять это, - [сообщение в блоге Эрика Портиса](https://ericportis.com/posts/2014/srcset-sizes/).

!!! warning ""

    Grav по умолчанию устанавливает аргумент `sizes`, упомянутый в сообщениях выше, на полную ширину области просмотра. Используйте действие `sizes`, показанное ниже, чтобы выбрать желаемый размер.

Чтобы начать использовать адаптивные изображения, всё, что вам нужно сделать, это добавить на свои страницы изображения с более высокой плотностью, добавив суффикс к имени файла. Если вы предоставляете только изображения с более высокой плотностью, Grav автоматически сгенерирует для вас версии более низкого качества. Именование работает следующим образом: `[image-name]@[density-ratio]x.[Image-extension]`, поэтому, например, добавление `sample-image@3x.jpg` на вашу страницу приведет к тому, что Grav создаст `2x` и версию `1x` (обычного размера) по умолчанию.

!!! info ""

    Эти файлы, созданные Grav, будут храниться в папке кэша `images/`, а не в папке вашей страницы.

Предположим, у вас есть файл с именем `retina@2x.jpg`, вы на самом деле укажете его в своих ссылках как `retina.jpg`, и тогда Grav не найдет это изображение и начнет искать размеры изображения. Он найдет `retina@2x.jpg`, а затем поймет, что ему нужно создать вариант `@1x` и отобразить соответствующий вывод `srcset`:

=== "Markdown"
`markdown
	![Retina Image](retina.jpg?sizes=80vw)
	`
=== "Twig"
`twig
	{{ page.media['retina.jpg'].sizes('80vw').html()|raw }}
	`
=== "HTML"
`
	<img alt="Retina Image" src="/images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg" srcset="/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg 2880w, /images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg 1440w" sizes="80vw">
	`

##### Результат:

![Retina Image](https://learn.getgrav.org/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg)

!!! danger ""

    В зависимости от вашего дисплея и реализации вашего браузера и поддержки `srcset`, вы можете никогда не увидеть разницы.

##### Размеры с медиа-запросами

Grav также поддерживает медиа-запросы внутри атрибута `sizes`, что позволяет использовать разную ширину в зависимости от размера экрана устройства. В отличие от первого метода, вам не нужно создавать несколько изображений; они будут созданы автоматически. Резервное изображение - это текущее изображение, поэтому браузер без поддержки `srcset` будет отображать исходное изображение.

=== "Markdown"
`markdown
	![Retina Image](retina.jpg?sizes=%28max-width%3A26em%29+100vw%2C+50vw)
	`
=== "Twig"
`twig
	{{ page.media['retina.jpg'].sizes('(max-width:26em) 100vw, 50vw').html('Retina Image')|raw }}
	`
=== "HTML"
`
	<img alt="Retina Image" src="/images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg" srcset="/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg 2880w, /images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg 1440w" sizes="(max-width:26em)+100vw">
	`

##### Результат:

![Retina Image](https://learn.getgrav.org/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg)

!!! danger ""

    В зависимости от вашего дисплея и реализации вашего браузера и поддержки `srcset`, вы можете никогда не увидеть разницы.

##### Размеры с медиа-запросами с использованием производных

Если вы хотите настроить размеры автоматически созданных файлов, вы можете использовать метод `derivatives()` (как показано ниже). Первый параметр — это ширина наименьшего из сгенерированных изображений. Второй — это максимальная ширина генерируемых изображений. Третий и единственный необязательный параметр определяет интервалы, с которыми будут создаваться фотографии (по умолчанию-200). Например, если вы установите первый параметр равным `320`, а третий — `100`, Grav будет генерировать изображение для 320, 420, 520, 620 и так далее, пока не достигнет установленного максимума.

В нашем примере мы установили максимум в `1600`. В результате будет достигнуто увеличение на 300 с `320` до `1520`, так как `1620` будет выше порога.

!!! warning ""

    На данный момент он не работает внутри `markdown`, только в ваших файлах `twig`.

=== "Markdown"
`markdown
	![Retina Image](retina.jpg?derivatives=320,1600,300&sizes=%28max-width%3A26em%29+100vw%2C+50vw)
	`
=== "Twig"
`twig
	{{ page.media['retina.jpg'].derivatives(320,1600,300).sizes('(max-width:26em) 100vw, 50vw').html()|raw }}
	`
=== "HTML"
`
	<img alt="Retina Image" src="/images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg" srcset="/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg 2880w, /images/d/c/e/f/7/dcef77ec0cc8efd0a66851a7750b530d8bfe093a-retina320w.jpg 320w, /images/1/5/d/a/1/15da17d9989ac18738474b1fab5ff6104b96be41-retina620w.jpg 620w, /images/e/c/e/3/f/ece3fa30474b851808a485197b481d202d8f3811-retina920w.jpg 920w, /images/3/8/8/2/4/3882463d358fc22a189380da7b7d14db2a5b260a-retina1220w.jpg 1220w, /images/6/0/5/0/e/6050e7409d6040b3737b6562cdec89854cac3f9a-retina1520w.jpg 1520w, /images/3/7/f/0/c/37f0ca845b3eb054374d6a1ac2e36e13c59e14f8-retina1x.jpg 1440w" sizes="(max-width:26em)+100vw">
	`

##### Результат:

![Retina Image](https://learn.getgrav.org/images/b/a/c/1/9/bac199ed46f9188dafad759760afd27da935e564-retina2x.jpg)

!!! danger ""

    В зависимости от вашего дисплея и реализации и поддержки `srcset` вашего браузера, вы, возможно, никогда не увидите разницы. Мы включили HTML-разметку в четвертую вкладку, чтобы вы могли видеть, что происходит за экранами.

#### Ручное определение размера

Вместо того чтобы позволить Grav генерировать размеры в четных шагах между заданными границами, вы можете вручную определить, какие размеры Grav должен генерировать:

```markdown
![Retina Image](retina.jpg?derivatives=[360,720,1200])
```

Это сгенерирует уменьшенные версии изображения `retina.jpg` с разной шириной: 360, 720 и 1200px.

## Метафайлы

Каждый файл, на который вы ссылаетесь в Grav, например `image1.jpg`, `sample-trailer.mov`, или даже `archive.zip` имеет возможность устанавливать переменные или даже переопределять их с помощью **мета-файла**. Эти файлы принимают формат `<filename>.meta.yaml`. Например, для изображения с именем файла `image1.jpg` можно создать метафайл `image1.jpg.meta.yaml`.

С помощью этого метода вы можете добавить практически любой параметр или кусочек метаданных.

Содержимое этого файла должно быть в синтаксисе YAML, примером может быть:

```yaml
image:
  filters:
    default:
      - [cropResize, 300, 300]
      - sharp
alt_text: My Alt Text
```

Если вы используете этот метод для добавления стиля для конкретного файла или мета-тегов для одного файла, вы захотите поместить YAML-файл в ту же папку, что и файл, на который делается ссылка. Это гарантирует, что файл будет вытянут вместе с данными YAML. Это удобный способ даже задать метаданные для конкретного файла, так как вы не можете этого сделать с самой страницы.

Допустим, вы хотели просто вытащить значение `alt_text`, указанное для файла изображения `sample-image.jpg`. Затем вы создаете файл под названием `sample-image.jpg.meta.yaml` и помещаете его в папку с файлом изображения, на который есть ссылка. Затем вставьте данные, использованные в примере выше, и сохраните этот YAML-файл. В файле Markdown для страницы эти данные можно отобразить, используя следующую строку:

```yaml
{{ page.media['sample-image.jpg'].meta.alt_text }}
```

При этом вместо изображения будет вытаскиваться пример фразы `My Alt Text`. Это всего лишь простой пример. Вы можете использовать этот метод для ряда вещей, включая создание галереи с несколькими уникальными точками данных, на которые вы хотите ссылаться для каждого изображения. Ваши изображения, по сути, имеют набор уникальных для них данных, на которые можно легко ссылаться и вытягивать по мере необходимости.

## Параметры видео

Встроенные опции управления видео — это ещё одна из имеющихся возможностей Grav. Эти опции, добавленные в строку с именем файла, дают вам возможность определить настройки `autoplay`, `controls` и `loop`.

Вот пример:

```markdown
![video.mov](video.mov?loop=1&controls=0&autoplay=1&muted)
```

Опции следующие:

| Атрибут  | Объяснение                                                                                           |
| :------- | :--------------------------------------------------------------------------------------------------- |
| autoplay | Включает (`1`) или отключает (`0`) автоматическое воспроизведение видео при загрузке страницы.       |
| controls | Включает (`1`) или отключает (`0`) управление мультимедиа для встроенного видео.                     |
| loop     | Включает (`1`) или отключает (`0`) автоматическое петлевание для видео, проигрывая его по окончании. |
| muted    | Отключите видео и, как правило, разрешите его воспроизведение в автоматическом режиме.               |
