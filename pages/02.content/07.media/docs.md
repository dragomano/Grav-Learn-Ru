---
title: Медиа
page-toc:
  active: true
  depth: 3
markdown:
  extra: true
taxonomy:
    category: docs
process:
    twig: true
---

При создании контента в **Grav** вам часто требуется отображать различные типы мультимедиа, такие как **изображения**, **видео** и различные другие **файлы**. Эти файлы автоматически обнаруживаются и обрабатываются Grav, и они становятся доступными для использования на любой странице. Это особенно удобно, потому что затем вы можете использовать встроенные функции страницы для использования миниатюр, доступа к метаданным и динамического изменения мультимедиа (например, изменения размера изображений, установки размера отображения для видео и т. д.) По мере необходимости.

Grav использует **систему интеллектуального кэширования**, которая при необходимости автоматически создает в кэше копии динамически генерируемых носителей. Таким образом, все последующие запросы могут использовать кешированную версию вместо того, чтобы создавать мультимедиа заново.

## Поддерживаемые медиа-файлы

Следующие типы медиафайлов изначально поддерживаются Grav. Дополнительная поддержка мультимедийных файлов и потоковых вставок может быть добавлена ​​через плагины.

[div class="table-keycol"]
| Тип медиа          | Тип файла                                                                                                                                                             |
| :-----             | :-----                                                                                                                                                                |
| Изображения        | jpg, jpeg, png                                                                                                                                                        |
| Аудио              | mp3, wav, wma, ogg, m4a, aiff, aif                                                                                                                                    |
| Анимированные изображения | gif                                                                                                                                                                |
| Векторные изображения   | svg                                                                                                                                                                   |
| Видео              | mp4, mov, m4v, swf, flv, webm, ogv                                                                                                                                    |
| Данные / Информация | txt, doc, docx, html, htm, pdf, zip, gz, 7z, tar, css, js, json, xml, xls, xlt, xlm, xlsm, xld, xla, xlc, xlw, xll, ppt, pps, rtf, bmp, tiff, mpeg, mpg, mpe, avi, wmv |
[/div]

Полный список поддерживаемых типов MimeType можно найти в файле `system/config/media.yaml`. Если есть тип mime, который в настоящее время не поддерживается, вы можете просто создать свой собственный `user/config/media.yaml` и добавить его туда. Просто убедитесь, что вы используете тот же формат, что и исходный файл `system`. Самый простой подход - скопировать весь исходный файл и внести необходимые изменения.

## Месторасположение медиафайлов

Обычно вы используете мультимедийный файл на странице, поэтому просто поместите файл в папку страницы, и вы можете ссылаться на него в разметке страницы, например:

[prism classes="language-markdown"]
![my image](image.jpg)
[/prism]

Если вы хотите поместить все изображения в одну папку, вы можете поместить их в папку `user/pages/images`. Таким образом, в Twig вы сможете связаться с ними через

{% verbatim %}
```twig
{% set my_image = page.find('/images').media['my-image.jpg'] %}
```
{% endverbatim %}

а также вы можете легко найти их с помощью markdown и выполнить с ними операции:

```markdown
![my image](/images/my-image.jpg?cropResize=300,300)
```

В качестве альтернативы вы можете поместить их в свою тему, так как это легко доступно через ссылки CSS или из файла разметки с помощью потока `theme: //`:

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

{% verbatim %}
```twig
{{ media['user://themes/mytheme/images/my-image.jpg'].html() }}
```
{% endverbatim %}

!!!! В Grav есть папка `/images`. Не помещайте в эту папку свои собственные изображения, поскольку в ней хранятся автоматически сгенерированные Grav кэшированные изображения.

Вы также можете поместить все медиафайлы в отдельную папку, чтобы все они были доступны за один раз. Например, вы можете захотеть сохранить все свои MP3-файлы в папке `user/pages/mp3s` (не отображается) и поместить имя MP3-файла, связанного с определенной страницей, в поле заголовка, называемое `thistrack`. Если вы затем захотите получить доступ к файлу для определенной страницы и воспроизвести его с помощью аудиоэлемента HTML5, вам понадобится следующий код:

{% verbatim %}
[prism classes="language-twig line-numbers"]
<audio controls>
  <source src="{{ page.find('/mp3s').media[page.header.thistrack~'.mp3'] }}">
</audio>
[/prism]
{% endverbatim %}

## Режимы отображения

Grav предоставляет несколько различных режимов отображения для каждого типа медиа-объекта.

[div class="table-keycol"]
| Режим      | Объяснение                                                                     |
| :-----    | :-----                                                                          |
| source    | Визуальное представление самого носителя, то есть фактического изображения, видео или файла |
| text      | Текстовое представление медиа                                             |
| thumbnail | Миниатюра для этого медиа-объекта                                       |
[/div]

!!!! Носители типа **Data/Information** не поддерживают режим «источник», по умолчанию они будут использовать режим `text`, если другой режим не выбран явно.

## Расположение эскиза

Grav будет искать ваши эскизы в трех местах.

1. В той же папке, что и мультимедийный файл: `[media-name].[media-extension].thumb.[thumb-extension]` где `media-name` и `media-extension` - это соответственно имя и расширение исходного медиафайла, а `thumb-extension` - это любое расширение, поддерживаемое типом медиафайла `image`. Примеры: `my_video.mp4.thumb.jpg` и `my-image.jpg.thumb.png`.
**Только для изображений!** Сама картинка будет использована в качестве эскиза.
2. Ваша пользовательская папка: `user/images/media/thumb-[media-extension].png`, где `media-extension` - это расширение исходного медиафайла. Примеры: thumb-mp4.png и thumb-jpg.jpg.
3. Системная папка: `system/images/media/thumb-[media-extension].png`, где `media-extension` - это расширение исходного медиафайла. **Эскизы в системных папках предоставлены Grav заранее.**

!! Вы также можете вручную выбрать нужный эскиз с помощью действий, описанных ниже.

## Ссылки и лайтбоксы

Вышеупомянутые режимы отображения также можно использовать в сочетании со ссылками и лайтбоксов, которые будут объяснены более подробно позже. Однако важно отметить следующее:

!!!! Grav не предоставляет функциональность лайтбоксов из коробки, для этого вам понадобится плагин. Для этого вы можете использовать [плагин FeatherLight Grav](https://github.com/getgrav/grav-plugin-featherlight).

Когда вы используете мультимедийные функции Grav для рендеринга лайтбокса, все, что делает Grav, выводит тег **якоря**, который имеет некоторые атрибуты для чтения плагином лайтбокса. Если вы заинтересованы в использовании библиотеки лайтбоксов, которой нет в нашем репозитории плагинов, или вы хотите создать свой собственный плагин, вы можете использовать приведенную ниже таблицу в качестве справки.

[div class="table-keycol"]
| Атрибут   | Объяснение                                                                                                  |
| :-----      | :-----                                                                                                       |
| rel         | Простой индикатор того, что это не обычная ссылка, а ссылка лайтбокса. Значение всегда будет `lightbox`. |
| href        | URL-адрес самого медиаобъекта.                                                                             |
| data-width  | Ширина, которую пользователь запросил для этого лайтбокса.                                                             |
| data-height | Высота, которую пользователь запросил для этого лайтбокса.                                                            |
| data-srcset | В случае графических носителей это содержит строку `srcset`. ([Дополнительная информация](../media#responsive-images))         |
[/div]

## Действия

Grav использует паттерн **Строитель** при работе с медиа, поэтому вы можете выполнять **несколько действий** с определенным медиа. Некоторые действия доступны для каждого вида среды, в то время как другие специфичны для среды.

### Общие

Эти действия доступны для всех типов медиа.

#### url

!! Этот метод предназначен только для использования в шаблонах **Twig**, отсюда отсутствие синтаксиса Markdown.

Это возвращает **необработанный url-путь** к медиа.

[ui-tabs]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].url }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-image.jpg'].url|e }}
```
[/ui-tab]
[/ui-tabs]


#### html

!! В Markdown этот метод неявно вызывается при использовании синтаксиса `![]`.

Действие `html` выведет действительный HTML-тег для мультимедиа в зависимости от текущего режима отображения.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Some ALT text](sample-image.jpg?classes=myclass "My title")
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].html('My title', 'Some ALT text', 'myclass') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-image.jpg'].html('My title', 'Some ALT text', 'myclass')|e }}
```
[/ui-tab]
[/ui-tabs]


#### display

Используйте это действие для переключения между различными режимами отображения, которые предоставляет Grav. Как только вы переключите режим отображения, все предыдущие действия будут сброшены. Исключениями из этого правила являются действия `lightbox` и `link`, а также любые действия, которые использовались до этих двух.

Например, эскиз, полученный в результате вызова `page.media['sample-image.jpg'].sepia().display('thumbnail').html()`, не будет иметь примененного действия `sepia()`, но `page.media['sample-image.jpg'].display('thumbnail').sepia().html()` будет применено действие `sepia()`.

! После того, как вы переключитесь в режим эскизов, вы будете манипулировать изображением. Это означает, что даже если текущим медиафайлом является видео, вы можете использовать все действия с изображениями на миниатюре.

#### link

Превратите свой медиа-объект в ссылку. Все действия, которые вы вызываете до `link()`, будут применены к цели ссылки, в то время как любые действия, вызываемые после, будут применены к тому, что отображается на вашей странице.

!! После вызова `link()`, Grav автоматически переключит режим отображения на **thumbnail**.

В следующем примере будет отображена текстовая ссылка (`display('text')`) на версию файла `sample-image.jpg` сепии:

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Image link](sample-image.jpg?sepia&link&display=text)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].sepia().link().display('text').html('Image link') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-image.jpg'].sepia().link().display('text').html('Image link')|e }}
```
[/ui-tab]
[/ui-tabs]

#### lightbox

Действие лайтбокса по сути такое же, как действие ссылки, но с некоторыми дополнениями. Как объяснялось выше, действие лайтбокса не будет делать ничего, кроме создания ссылки с некоторыми дополнительными атрибутами. Он отличается от действия ссылки тем, что добавляет атрибут `rel="lightbox"` и принимает атрибуты `width` и `height`.

Если возможно (в настоящее время только в случае изображений), Grav изменит размер вашего мультимедиа до требуемой ширины и высоты. В противном случае он просто добавит к ссылке атрибуты `data-width` и `data-height`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?lightbox=600,400&resize=200,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].lightbox(600,400).resize(200,200).html('Sample Image') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-image.jpg'].lightbox(600,400).resize(200,200).html('Sample Image')|e }}
```
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?lightbox=600,400&resize=200,200)

#### thumbnail

Вручную выберите миниатюру, которую должен использовать Grav. Вы можете выбрать между `page` и `default` для любого типа медиа, а также `media` для графических медиа, если вы хотите использовать сам медиаобъект в качестве миниатюры.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?thumbnail=default&display=thumbnail)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].thumbnail('default').display('thumbnail').html('Sample Image') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-image.jpg'].thumbnail('default').display('thumbnail').html('Sample Image')|e }}
```
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?thumbnail=default&display=thumbnail)

## Действия с изображениями

#### resize

Изменение размера делает именно то, что вы ожидаете. `resize` позволяет вам создать новое изображение на основе «ширины» и «высоты». Соотношение сторон сохраняется, и новое изображение будет содержать пустые области цвета **необязательного** цвета фона, представленного как «шестнадцатеричное значение», например `0xffffff`. Параметр фона является необязательным, и, если он не указан, по умолчанию будет установлено значение **прозрачное**, если изображение в формате PNG, или **белое**, если оно в формате JPEG.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?resize=400,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].resize(400, 200).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?resize=400,200)

#### forceResize

Изменяет размер изображения на ширину и высоту, как указано. `forceResize` не будет учитывать исходное соотношение сторон и растянет изображение по мере необходимости, чтобы оно соответствовало новому размеру изображения.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?forceResize=200,300)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].forceResize(200, 300).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?forceResize=200,300)

#### cropResize

`cropResize` изменяет размер изображения до меньшего или большего размера в зависимости от ширины и высоты. Соотношение сторон сохраняется, и размер нового изображения будет изменен, чтобы поместиться в ограничивающей рамке, как описано с помощью предоставленных `width` и `height`. Другими словами, любая область фона, которую вы видите при обычном изменении размера, обрезается.

Например, если у вас есть изображение размером `640` x `480` и вы выполняете над ним действие `cropResize(100, 100)`, вы получите изображение размером `100` x` 75`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropResize=300,300)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropResize(300, 300).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropResize=300,300)

#### crop

`crop` не изменит размер изображения вообще, он просто обрежет исходное изображение так, чтобы только часть ограничивающего прямоугольника, описываемая значениями `width` и `height`, происходила из местоположений `x` и `y.` используется для создания нового изображения.

Например, изображение размером `640` x `480`, на которое действует действие `crop(0, 0, 400, 100)`, будет просто обрезано по ширине и высоте, так что в результате будет получено изображение. с шириной `400` и высотой `100` происходит от верхнего левого угла, как описано как `0, 0`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?crop=100,100,300,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].crop(100,100,300,200).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?crop=100,100,300,200)

#### cropZoom

Подобно обычному `cropResize`, `cropZoom` также принимает значения `width` и `height`, но **изменяет размер и обрезает** изображение, чтобы в результате был получен именно то размер, который вы запрашивали. Соотношение сторон сохраняется, но части изображения могут быть обрезаны, однако результирующее изображение будет центрировано.

!! Основное различие между **cropResize** и **cropZoom** заключается в том, что в параметре cropResize размер изображения изменяется с сохранением соотношения сторон, так что отображается все изображение, а любое дополнительное пространство считается фоном.

При использовании **cropZoom** размер изображения изменяется так, что фон не виден, а дополнительная область изображения за пределами нового размера изображения обрезается.

Например, если у вас есть изображение размером `640` x` 480` и вы выполняете действие `cropZoom(400, 100)`, результирующее изображение будет изменено до `400` x `300`, а затем будет обрезана высота. В результате получается изображение `400` x `100`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=600,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(600,200).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

!! Люди, знакомые с использованием `zoomCrop` для этой цели, обнаружат, что он также работает в Grav.

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=600,200)

#### quality

Динамически позволяет установить значение **процента сжатия** для изображения от `0` до `100`. Меньшее число означает меньшее качество, где `100` означает максимальное качество.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&quality=25)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).quality(25).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&quality=25)

#### negate

Применяет **отрицательный фильтр** к изображению с инвертированными цветами.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&negate)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).negate.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&negate)

#### brightness

Применяет **фильтр яркости** к изображению со значением от `-255` до `+255`. Большие отрицательные числа сделают изображение темнее, а большие положительные числа сделают изображение ярче.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&brightness=-100)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).brightness(-100).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&brightness=-100)

#### contrast

Это применяет **фильтр контраста** к изображению со значением от `-100` до `+100`. Большие отрицательные числа увеличивают контраст, а большие положительные числа уменьшают контраст.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&contrast=-50)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).contrast(-50).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

![Sample Image](sample-image.jpg?cropZoom=300,200&contrast=-50)

#### grayscale

Это обрабатывает изображение с помощью **фильтра оттенков серого**.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&grayscale)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).grayscale.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&grayscale)

#### emboss

Это обрабатывает изображение с помощью **фильтра тиснения**.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&emboss)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).emboss.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&emboss)

#### smooth

Это применяет **фильтр сглаживания** к изображению на основе параметра сглаживания `value` от `-10` до `10`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&smooth=5)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).smooth(5).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&smooth=5)

#### sharp

Это применяет **фильтр повышения резкости** к изображению.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&sharp)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).sharp.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&sharp)

#### edge

Это применяет **фильтр обнаружения краев** к изображению.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&edge)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).edge.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&edge)

#### colorize

Вы можете раскрасить изображение на основе настройки значений `red`, `green` и `blue` для изображения от `-255` до `+255` для каждого цвета.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&colorize=100,-100,40)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).colorize(100,-100,40).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&colorize=100,-100,40)

#### sepia

Это применяет **фильтр сепии** к изображению для создания винтажного вида.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&sepia)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).sepia.html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&sepia)

#### gaussianBlur

**Размывает** изображение с помощью фактора, который определяет, как часто фильтр размытия применяется к изображению. По умолчанию 1 раз.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?gaussianBlur=3)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].gaussianBlur(3).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?gaussianBlur=3)

#### rotate

**Поворачивает** изображение на заданный `угол` против часовой стрелки, отрицательные значения вращают по часовой стрелке.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&rotate=-90)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).rotate(-90).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&rotate=-90)

#### flip

**Переворачивает** изображение в заданном направлении. Оба параметра могут иметь значение `0|1`. Оба значения `0` эквивалентны отсутствию переворачивания в любом направлении.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?cropZoom=300,200&flip=0,1)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].cropZoom(300,200).flip(0,1).html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?cropZoom=300,200&flip=0,1)

#### fixOrientation

Исправляет ориентацию изображения при повороте через данные EXIF ​​(применимо к изображениям jpeg, снятым с помощью телефонов и камер).

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?fixOrientation)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].fixOrientation }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]


## Анимированные/векторизованные действия

#### resize

Поскольку PHP не может обрабатывать динамическое изменение размера этих типов носителей, действие resize будет только гарантировать, что атрибут `width` и `height` или `data-width` и `data-height` установлены в вашем теге `<img>`/`<video>` или `<a>` соответственно. Это означает, что ваше изображение или видео будет отображаться в требуемом размере, но фактическое изображение или видеофайл не будут преобразованы каким-либо образом.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Trailer](sample-trailer.mov?resize=400,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-trailer.mov'].resize(400, 200).html('Sample Trailer') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['sample-trailer.mov'].resize(400, 200).html('Sample Trailer')|e }}
```
[/ui-tab]
[/ui-tabs]


#### примеры

Некоторые примеры этого:

[ui-tabs]
[ui-tab title="Vector Image"]
```markdown
![Sample Vector](sample-vector.svg?resize=300,300)
```
![Sample Vector](sample-vector.svg?resize=300,300)
[/ui-tab]
[ui-tab title="Animated Image"]
```markdown
![Animated Gif](sample-animated.gif?resize=300,300)
```
![Animated Gif](sample-animated.gif?resize=300,300)
[/ui-tab]
[ui-tab title="Video"]
```markdown
![Sample Trailer](sample-trailer.mov?resize=400,200)
```
![Sample Trailer](sample-trailer.mov?resize=400,200)
[/ui-tab]
[/ui-tabs]

## Действия с аудио

Аудио медиа будет отображать аудио ссылку HTML5:

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].html() }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Hal 9000: I'm Sorry Dave](hal9000.mp3)

#### элементы управления

Позволяет явно устанавливать или удалять элементы управления HTML5 по умолчанию. Передача `0` скрывает элементы управления браузером для воспроизведения, громкости и т. д.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?controls=0)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].controls(0) }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
```html
{{ page.media['hal9000.mp3'].controls(0) }}
```
[/ui-tab]
[/ui-tabs]

#### предварительная загрузка

Позволяет установить свойство `preload`, которое по умолчанию имеет значение `auto`. Допустимые параметры: `auto`, `metadata` и `none`.

!! <q cite="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio#attr-preload">Если не установлен, его значение по умолчанию определяется браузером (т.е. каждый браузер может иметь собственное значение по умолчанию). В спецификации рекомендуется установить значение <code>метаданные</code>.</q>

!! Атрибут `preload` игнорируется, если присутствует `autoplay`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?preload=metadata)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].preload('metadata') }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

#### Атрибут autoplay

Позволяет установить, будет ли аудио воспроизводиться автоматически при загрузке страницы. Если не задано, по умолчанию используется `false`.

!! Если в заданном элементе `audio` присутствуют и `autoplay`, и `preload`, `preload` игнорируется.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?autoplay=1)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].autoplay(1) }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]


#### controlsList

Позволяет установить свойство `controlsList`, которое принимает одно или несколько из трех возможных значений: `nodownload`, `nofullscreen` и `noremoteplayback`.

!! Если вы устанавливаете более одного параметра в markdown, разделяйте их тире (`-`). Они будут заменены пробелами в выходном HTML.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?controlsList=nodownload-nofullscreen-noremoteplayback)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].controlsList('nodownload nofullscreen noremoteplayback') }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

#### muted

Позволяет установить, будет ли звук отключен при загрузке. Если не задано, по умолчанию используется `false`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?muted=1)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].muted(1) }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

#### loop

Позволяет установить, будет ли звук зацикливаться после завершения воспроизведения. Если не задано, по умолчанию используется `false`.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Hal 9000: I'm Sorry Dave](hal9000.mp3?loop=1)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['hal9000.mp3'].loop(1) }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

## Действия с файлами

На данный момент Grav не предоставляет никаких настраиваемых действий с файлами, и мы не планируем их добавлять. Если вы что-то придумаете, свяжитесь с нами.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
[View Text File](acronyms.txt)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
<a href="{{ page.media['acronyms.txt'].url() }}">View Text File</a>
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

[View Text File](acronyms.txt)

### Комбинации

Как видите: Grav предоставляет несколько мощных функций для работы с изображениями, которые упрощают работу с изображениями! Настоящая сила приходит, когда вы комбинируете несколько эффектов и производите очень сложные динамические манипуляции с изображениями. Например, это абсолютно верно:

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Sample Image](sample-image.jpg?negate&lightbox&cropZoom=200,200)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['sample-image.jpg'].negate.lightbox.cropZoom(200,200) }}
```
{% endverbatim %}
[/ui-tab]
[/ui-tabs]

##### Результат:

![Sample Image](sample-image.jpg?negate&lightbox&cropZoom=200,200)

#### Сброс нескольких вызовов на одно и то же изображение

Когда вы обращаетесь к одному и тому же изображению несколько раз на одной странице, действия, которые вы предоставили для изображения, не сбрасываются по умолчанию. Поэтому, если вы измените размер изображения и выведите HTML-код, а затем на той же странице просто выведите URL-адрес изображения, вы также получите URL-адрес измененного изображения. Вы, вероятно, ожидали URL-адрес исходного изображения.

Чтобы бороться с этим, вы можете сбросить действия с изображениями, передав `false` методу `url()`:

{% verbatim %}
```twig
{% for item in page.header.gallery %}
    {% set image = page.media[item.src].cropZoom(800, 600).quality(70) %}
    <a href="{{ image.url(false) }}">
      <img src="{{ image.url }}" alt="{{ item.alt }}" title="{{ item.title }}" />
    </a>
{% endfor %}
```
{% endverbatim %}

### Адаптивные изображения

#### Дисплеи с более высокой плотностью

Grav имеет встроенную поддержку отзывчивых изображений для дисплеев с более высокой плотностью (например, экранов **Retina**). Grav выполняет это с помощью атрибута `srcset` в [HTML элементе изображения](https://html.spec.whatwg.org/multipage/embedded-content.html#the-picture-element). Хорошая статья для чтения, если вы хотите лучше понять это, - [это сообщение в блоге Эрика Портиса](http://ericportis.com/posts/2014/srcset-sizes/).

!! Grav по умолчанию устанавливает аргумент `sizes`, упомянутый в сообщениях выше, на полную ширину области просмотра. Используйте действие `sizes`, показанное ниже, чтобы выбрать себя.

Чтобы начать использовать адаптивные изображения, все, что вам нужно сделать, это добавить на свои страницы изображения с более высокой плотностью, добавив суффикс к имени файла. Если вы предоставляете только изображения с более высокой плотностью, Grav автоматически сгенерирует для вас версии более низкого качества. Именование работает следующим образом: `[image-name]@[density-ratio]x.[Image-extension]`, поэтому, например, добавление `sample-image@3x.jpg` на вашу страницу приведет к тому, что Grav создаст `2x` и версию `1x` (обычного размера) по умолчанию.

! Эти файлы, созданные Grav, будут храниться в папке кэша `images/`, а не в папке вашей страницы.

Предположим, у вас есть файл с именем `retina@2x.jpg`, вы на самом деле укажете его в своих ссылках как `retina.jpg`, и тогда Grav не найдет это изображение и начнет искать размеры изображения. Он найдет `retina@2x.jpg`, а затем поймет, что ему нужно создать вариант `@1x` и отобразить соответствующий вывод `srcset`:

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Retina Image](retina.jpg?sizes=80vw)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['retina.jpg'].sizes('80vw').html() }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
{% set code_sample %}
![Retina Image](retina.jpg?sizes=80vw)
{% endset %}
```html
{{ code_sample|e }}
```
[/ui-tab]
[/ui-tabs]

##### Результат:

![Retina Image](retina.jpg?sizes=80vw)

!!!! В зависимости от вашего дисплея и реализации вашего браузера и поддержки `srcset`, вы можете никогда не увидеть разницы. Мы включили разметку HTML в третью вкладку, чтобы вы могли видеть, что происходит за экранами.

##### Размеры с медиа-запросами

Grav также поддерживает медиа-запросы внутри атрибута `sizes`, что позволяет использовать разную ширину в зависимости от размера экрана устройства. В отличие от первого метода, вам не нужно создавать несколько изображений; они будут созданы автоматически. Резервное изображение - это текущее изображение, поэтому браузер без поддержки `srcset` будет отображать исходное изображение.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Retina Image](retina.jpg?sizes=%28max-width%3A26em%29+100vw%2C+50vw)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['retina.jpg'].sizes('(max-width:26em) 100vw, 50vw').html('Retina Image') }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
{% set code_sample %}
![Retina Image](retina.jpg?sizes=%28max-width%3A26em%29+100vw%2C+50vw)
{% endset %}
```html
{{ code_sample|e }}
```
[/ui-tab]
[/ui-tabs]

##### Результат:

![Retina Image](retina.jpg?sizes=%28max-width%3A26em%29+100vw%2C+50vw)

!!!! В зависимости от вашего дисплея и реализации вашего браузера и поддержки `srcset`, вы можете никогда не увидеть разницы. Мы включили разметку HTML в четвертую вкладку, чтобы вы могли видеть, что происходит за экранами.

##### Размеры с медиа-запросами с использованием производных

Если вы хотите настроить размеры автоматически созданных файлов, вы можете использовать метод `derivatives()` (как показано ниже). Первый параметр-это ширина наименьшего из сгенерированных изображений. Второй - это максимальная ширина генерируемых изображений. Третий и единственный необязательный параметр определяет интервалы, с которыми будут создаваться фотографии (по умолчанию-200). Например, если вы установите первый параметр равным `320`, а третий - `100`, Grav будет генерировать изображение для 320, 420, 520, 620 и так далее, пока не достигнет установленного максимума.

В нашем примере мы установили максимум в `1600`. В результате будет достигнуто увеличение на 300 с `320` до `1520`, так как `1620` будет выше порога.

!! На данный момент он не работает внутри markdown, только в ваших файлах ```twig```.

[ui-tabs]
[ui-tab title="Markdown"]
```markdown
![Retina Image](retina.jpg?derivatives=320,1600,300&sizes=%28max-width%3A26em%29+100vw%2C+50vw)
```
[/ui-tab]
[ui-tab title="Twig"]
{% verbatim %}
```twig
{{ page.media['retina.jpg'].derivatives(320,1600,300).sizes('(max-width:26em) 100vw, 50vw').html() }}
```
{% endverbatim %}
[/ui-tab]
[ui-tab title="HTML Code"]
{% set code_sample %}
![Retina Image](retina.jpg?derivatives=320,1600,300&sizes=%28max-width%3A26em%29+100vw%2C+50vw)
{% endset %}
```html
{{ code_sample|e }}
```
[/ui-tab]
[/ui-tabs]

##### Результат:

![Retina Image](retina.jpg?derivatives=320,1600,300&sizes=%28max-width%3A26em%29+100vw%2C+50vw)

!!!! В зависимости от вашего дисплея и реализации и поддержки `srcset` вашего браузера, вы, возможно, никогда не увидите разницы. Мы включили HTML-разметку в четвертую вкладку, чтобы вы могли видеть, что происходит за экранами.


#### Ручное определение размера

Вместо того чтобы позволить Grav генерировать размеры в четных шагах между заданными границами, вы можете вручную определить, какие размеры Grav должен генерировать:

```markdown
![Retina Image](retina.jpg?derivatives=[360,720,1200])
```

Это сгенерирует уменьшенные версии изображения `retina.jpg` в трех ширинах: 360, 720 и 1200px.

## Метафайлы

Каждое средство, на которое вы ссылаетесь в Grav, например `image1.jpg`, `sample-trailer.mov`, или даже `archive.zip` имеет возможность устанавливать переменные или даже переопределять их с помощью **мета-файла**. Эти файлы принимают формат `<filename>.meta.yaml`. Например, для изображения с именем файла `image1.jpg` можно создать метафайл `image1.jpg.meta.yaml`.

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

Допустим, вы хотели просто вытащить значение `alt_text`, указанное для файла изображения `sample-image.jpg`. Затем вы создаете файл под названием `sample-image.jpg.meta.yaml` и помещаете его в папку с файлом изображения, на который есть ссылка. Затем вставьте данные, использованные в примере выше, и сохраните этот YAML-файл. В файле уценки для страницы эти данные можно отобразить, используя следующую строку:

{% verbatim %}
```yaml
{{ page.media['sample-image.jpg'].meta.alt_text }}
```
{% endverbatim %}

При этом вместо изображения будет вытаскиваться пример фразы `My Alt Text`. Это всего лишь основной пример. Вы можете использовать этот метод для ряда вещей, включая создание галереи с несколькими уникальными точками данных, на которые вы хотите ссылаться для каждого изображения. Ваши изображения, по сути, имеют набор уникальных для них данных, на которые можно легко ссылаться и вытягивать по мере необходимости.

## Параметры видео

Встроенные опции управления видео - это еще одна возможность, запеченная в Grav. Эти опции, добавленные в строку с именем файла, дают вам возможность определить настройки `autoplay`, `controls` и `loop`.

Вот пример:

```markdown
![video.mov](video.mov?loop=1&controls=0&autoplay=1&muted)
```

Опции следующие:

[div class="table-keycol"]
| Атрибут   | Объяснение                                                                               |
| :-----      | :-----                                                                                    |
| autoplay    | Включает (`1`) или отключает (`0`) автоматическое воспроизведение видео при загрузке страницы.                       |
| controls    | Включает (`1`) или отключает (`0`) управление мультимедиа для встроенного видео.                    |
| loop        | Включает (`1`) или отключает (`0`) автоматическое петлевание для видео, проигрывая его по окончании. |
| muted       | Отключите видео и, как правило, разрешите его воспроизведение в автоматическом режиме.                                            |
[/div]


