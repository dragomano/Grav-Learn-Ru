---
title: "Lightbox Gallery"
description: "Как использовать Lightbox Gallery на вашем сайте."
tags:
  - "Grav Premium"
  - "Lightbox Gallery"
  - "Медиафайлы"
  - "Grav 1.7"
---

# Lightbox Gallery

> [!IMPORTANT] Важно
> Продукты Premium требуют бесплатного плагина [License Manager](../license-manager/index.md). Установите его и добавьте лицензию продукта перед установкой этого продукта.

## Установка

Сначала убедитесь, что у вас установлена последняя версия **Grav 1.7** (`-f` принудительно обновляет индекс GPM).

```
$ bin/gpm selfupgrade -f
```

Lightbox Gallery требует плагин **shortcode-core** как зависимость, хотя синтаксис шорткодов необязателен. Они доступны через GPM, и поскольку у плагина есть зависимости, вам достаточно продолжить установку плагина Lightbox Gallery и подтвердить установку остальных, когда появится запрос:

```
$ bin/gpm install lightbox-gallery
```

Плагин также можно установить через раздел **Плагины** в плагине Admin.

## Конфигурация

Проще всего настроить плагин через админку: перейдите в раздел **Плагины** и нажмите на запись плагина **Lightbox Gallery**. Доступные параметры:

#### Общая конфигурация

`![Общая конфигурация](general.png?class=image-shadow)`

* **Статус плагина** → включает или отключает весь плагин.

* **Всегда активен** → если отключено, вы можете решать, когда включать Lightbox Gallery для каждой страницы. Чтобы включить на странице, задайте `lightbox: active: true` в метаданных страницы.

* **Всегда подключать ресурсы** → если отключено, вы будете сами подключать CSS- и JS-файлы из шаблонов Twig. По умолчанию это делает класс плагина.

#### Внешний вид и эффекты

`![Внешний вид и эффекты](appearance-effects.png?class=image-shadow)`

* **Селектор** → имя CSS-селектора, который Lightbox Gallery ищет на странице. Может быть списком через запятую и поддерживает любой допустимый CSS-селектор. По умолчанию — `[rel="lightbox"], .glightbox`.

* **Скин** → имя скина; добавляет класс к лайтбоксу, чтобы вы могли оформить его через CSS. По умолчанию доступен только один скин — `clean`.

* **Эффект открытия** → тип эффекта при открытии Lightbox Gallery. Может быть **Zoom** (по умолчанию), **Fade** или **None**.

* **Эффект закрытия** → тип эффекта при закрытии Lightbox Gallery. Может быть **Zoom** (по умолчанию), **Fade** или **None**.

* **Эффект смены слайдов** → тип эффекта при переключении между слайдами. Может быть **Slide** (по умолчанию), **Fade**, **Zoom** или **None**.

* **Текст «Подробнее»** → текст, который отображается на мобильных устройствах для длинных описаний.

* **Длина до «Подробнее»** → количество символов описания, отображаемых до добавления ссылки **Текст «Подробнее»** (только для мобильных). Если задать 0, будет показано всё описание.

#### Действия

`![Действия](actions.png?class=image-shadow)`

* **Кнопка закрытия** → показывает или скрывает кнопку закрытия.

* **Сенсорная навигация** → включает или отключает сенсорную навигацию (свайпы).

* **Следование оси при сенсорном управлении** → изображение следует за осью при перетаскивании на мобильном устройстве.

* **Навигация с клавиатуры** → включает или отключает навигацию с клавиатуры.

* **Закрытие по клику вне лайтбокса** → закрывает Lightbox Gallery при нажатии за пределами активного слайда.

* **Начальный слайд** → начинает Lightbox Gallery с указанного индекса. Учтите, что нумерация начинается с нуля (слайд 1 — это `0`, слайд 2 — `1` и т. д.).

#### Оформление

`![Оформление](styling.png?class=image-shadow)`

* **Ширина** → ширина по умолчанию для встроенных элементов и iframe; можно задать отдельный размер для каждого слайда. Можно использовать любые допустимые единицы (90%, 100vw для полной ширины).

* **Высота** → высота по умолчанию для встроенных элементов и iframe; можно задать отдельный размер для каждого слайда. Можно использовать любые допустимые единицы (90%, 100vh для полной высоты).

  !!! warning
      Для встроенных элементов можно задать высоту `auto`.

* **Ширина видео** → ширина по умолчанию для видео. Ширина может быть в любых допустимых единицах (например, 500px, 90%, 100vw для видео на всю ширину).
  ! Видео адаптивны, поэтому задавать высоту не требуется.

* **Положение описания** → глобальное положение описания слайдов. Можно также указать другое положение для каждого слайда. Доступные положения: **Bottom** (по умолчанию), **Top**, **Left**, **Right**.

#### Поведение

`![Поведение](behaviors.png?class=image-shadow)`

* **Зацикливание** → включает или отключает бесконечную прокрутку. По достижении последнего слайда показ начинается заново.

* **Масштабирование** → включает или отключает поведение масштабирования изображений.

* **Перетаскивание** → включает или отключает поведение перетаскивания мышью для перехода между слайдами. Поддерживаются только изображения и встроенный контент.

* **Порог перетаскивания по X** → когда **Draggable** включено, определяет количество пикселей, которое пользователь должен перетащить, прежде чем перейти к предыдущему или следующему слайду.

* **Порог перетаскивания по Y** → когда **Draggable** включено, определяет количество пикселей, которое пользователь должен перетащить по вертикали, прежде чем Lightbox Gallery закроется.
  ! Установите `0`, чтобы отключить вертикальное перетаскивание.

* **Автопереход при перетаскивании** → включает или отключает автоматическую смену слайдов на предыдущий и следующий либо закрытие Lightbox Gallery при достижении **Drag Tolerance X** или **Drag Tolerance Y**. Если отключено, плагин будет ждать, пока кнопка мыши не будет отпущена.

* **Предзагрузка** → если включено, предыдущий и следующий слайды загружаются заранее.

* **Автовоспроизведение видео** → включает или отключает автовоспроизведение видео, когда слайд с ними становится активным.

* **Автофокус видео** → если включено, видео получают фокус, чтобы работали сочетания клавиш плеера.
  !!!! Включение этого параметра отключает работу сочетаний клавиш для стрелок **назад** и **вперёд**. Вы по-прежнему сможете нажимать на стрелки, чтобы менять слайд.

## Использование
#### Стандартный режим лайтбокса

Обработка медиафайлов Grav поддерживает простой способ [создания лайтбокса](https://learn.getgrav.org/16/content/media?target=_blank#lightbox) с помощью чистого Markdown. Он не даёт много возможностей — это просто быстрый способ показать миниатюру и открыть лайтбокс при нажатии на неё.

``` twig
![Sample Image](blue-superleggera.jpg?lightbox=1024,768&resize=200,200)
```

#### Устаревший лайтбокс с шорткодом

Если нужна более гибкая настройка, используйте шорткод `[lightbox]` (требуется плагин **shorcode-core**). Он открывает все параметры во время выполнения и для конкретного экземпляра лайтбокса.

Поддерживаемые атрибуты шорткода:

* image → целевое изображение
* video → целевое видео
* class → классы, которые будут применены к ссылке лайтбокса (уже включает `.lightbox-gallery`)
* gallery → имя галереи
* title → заголовок изображения
* desc → текстовое описание или ссылка на класс HTML-элемента с тем же классом
* descPosition → положение описания (top, bottom, left, right)
* effect → эффект открытия/закрытия
* width → ширина миниатюры
* height → высота миниатюры
* zoomable → true или false
* draggable → true или false

``` bbcode
[lightbox image="white-vantage-v12.jpg" zoomable="false" draggable="false"]
![White V12 Vantage](white-vantage-v12.jpg?cropZoom=200,200)
[/lightbox]
```

#### Новый формат шорткода лайтбокса

Описанный выше подход устарел: в нём миниатюра задаётся ссылкой Markdown между тегами шорткода. Новый подход позволяет задать миниатюру атрибутом `thumb=""`, который даже поддерживает обработку медиафайлов, а между тегами шорткода можно разместить Markdown с описанием, отображаемым при открытии лайтбокса. Это новый и **предпочтительный** подход, поскольку он значительно упрощает использование нового шорткода `lightbox-gallery`.

Например:

```bbcode
[lightbox image="white-vantage-v12.jpg" thumb="white-vantage-v12.jpg?cropZoom=200,200"]
### White Vantage

This is a stunning photograph of a white Aston Martin V12 Vantage...
[/lightbox]
```

#### Видеолайтбокс с шорткодами

Lightbox Gallery умеет работать с локальными видеофайлами страницы и удалёнными URL, например YouTube. Вот пример пользовательской миниатюры, открывающей видео YouTube в лайтбоксе.

```bbcode
[lightbox video="https://www.youtube.com/watch?v=4czjS9h4Fpg"]
![Mars Lander](mars-thumb.jpg?cropZoom=330,200)
[/lightbox]
```

#### Галерея в лайтбоксе с шорткодами

Предпочтительный способ создать галерею — использовать новый шорткод-обёртку `lightbox-gallery`. Он упрощает синтаксис, автоматически управляет группировкой, позволяет совместно использовать параметры миниатюр и естественно поддерживает встроенные описания.

Вы можете передать `thumb_options` обёртке, чтобы автоматически изменять размер миниатюр для всех изображений в галерее. Также можно применить классы к обёртке (например, для сеточного макета). Учтите: если в именах классов нужны квадратные скобки (например, произвольные значения Tailwind), их необходимо экранировать: `class="... \[&>a>img\]:rounded-md"`.

``` bbcode
[lightbox-gallery thumb_options="cropZoom=200,200" class="grid grid-cols-4 gap-4"]
[lightbox image="red-dbs-1.jpg"]
### Aston Martin DBS 1
Morbi ac interdum velit. Ut sed purus in **erat feugiat mollis**. In porta ligula quis vulputate ullamcorper.
[/lightbox]
[lightbox image="red-dbs-2.jpg"]
### Aston Martin DBS 2
Aenean eu condimentum odio. Aliquam ac justo eget libero ullamcorper vehicula.
[/lightbox]
[lightbox image="red-dbs-3.jpg"]
### Aston Martin DBS 3
You can easily add rich descriptions to each item.
[/lightbox]
[/lightbox-gallery]
```

> [!IMPORTANT] Важно
> Примечание: в шорткодах `lightbox` выше атрибут `thumb=""` не указан, поскольку он автоматически создаётся из комбинации атрибута `image=""` и атрибута `thumb-options=""` родительского `lightbox-gallery`. Однако вы можете переопределить это поведение, добавив атрибут `thumb=""` в шорткод `lightbox`.

#### Шаблон Twig
Вы можете использовать предоставленный шаблон partials/lightbox-gallery.html.twig в своих шаблонах Twig. Этот шаблон принимает те же переменные Twig, что и шорткоды.

> [!NOTE] Примечание
> Шорткод `[div class="prose"] ... [/div]` обеспечивает автоматическое оформление CSS через класс `.prose` Twig.

#### Подключение собственных ресурсов
Если вы отключили автоматическое подключение ресурсов (**Всегда подключать ресурсы**), подключайте ресурсы через Twig на нужных страницах следующей командой.

``` twig
{% do lightbox_gallery.addAssets() %}
```

#### Отображение галереи миниатюр через Twig

`![Вид галереи](gallery-view.jpg?classes=shadow)`

Тема Typhoon уже предоставляет **модульный** шаблон Twig с аккуратным сеточным макетом для миниатюр. В нём используются классы Tailwind CSS для оформления. Если вы не используете Typhoon или просто хотите создать аккуратный сеточный макет в своей теме, вам потребуется создать шаблон Twig и немного стилей. В этом примере я создал простой фрагмент шаблона Twig: `partials/gallery.html.twig`, который включает базовую HTML-структуру и пример CSS, который вы, конечно, можете изменить по своим потребностям:

```twig
{% set styling %}
.lightbox-gallery {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.lightbox-gallery .lightbox-gallery__columns {
  display: flex;
  flex-wrap: wrap;
  margin: 0 -10px;
}

.lightbox-gallery .lightbox-gallery__column {
  width: 50%;
  padding: 0 10px;
  margin-bottom: 20px;
}

.lightbox-gallery img {
  display: block;
  border-radius: 5px;
  transition: all .2s ease-in-out;
}

.lightbox-gallery img:hover {
  filter: brightness(60%);
  transform: scale(1.05);
}

@media only screen and (min-width: 600px) {
  .lightbox-gallery .lightbox-gallery__column {
    width: calc(100% / 3);
  }
}

@media only screen and (min-width: 1000px) {
  .lightbox-gallery .lightbox-gallery__column {
    width: 25%;
  }
}

.lightbox-gallery .hidden {
  display: none;
}
{% endset %}

{% do assets.addInlineCss(styling) %}

{% set data = page.header.gallery %}
{% set thumb_width = data.thumb.width|default(600) %}
{% set thumb_height = data.thumb.height|default(450) %}

<div class="lightbox-gallery">
    <div class="lightbox-gallery__columns">
        {% set gallery = md5(page.url) %}
        {% for item in data.items %}
        <div class="lightbox-gallery__column">
            {% set item_image = page.media[item.image] %}
            {% if item.title %}
            {% set title = item.title %}
            {% endif %}
            {% if item.desc %}
            {% set desc = ".desc-" ~ md5(item.desc) %}
            {% endif %}
            {% set content = item_image.cropZoom(thumb_width,thumb_height).html(title, title) %}
            {% set image =  item_image.url %}
            {% include "partials/lightbox.html.twig" %}
        </div>
        {% endfor %}
    </div>
    <div class="hidden">
        {% for item in data.items %}
        {% if item.desc %}
            <div class="glightbox-desc desc-{{ md5(item.desc) }}">
            <p>{{ item.desc|markdown(false) }}</p>
            </div>
        {% endif %}
        {% endfor %}
    </div>
</div>
```

В этом примере предполагается, что галерея задана в настраиваемом заголовке страницы `gallery:`, и включает легко изменяемый размер миниатюр, а также описание, которое используется в модальном окне «просмотр галереи».

```yaml
---
title: 'Powerful Gallery'
subtitle: 'Exploring the globe...'
gallery:
    thumb:
        width: 400
        height: 300
    items:
        -
            title: 'Climbing Hilly Peaks'
            image: david-marcu-78A265wPiO4-unsplash.jpg
            desc: 'Etiam pretium elit vitae eros fringilla, vel gravida augue eleifend. Ut convallis, lacus non eleifend maximus, turpis odio mollis enim, ac malesuada quam leo ut augue. Quisque mollis urna ac ex varius, vel faucibus enim lacinia. Aenean a lorem consectetur, pulvinar sem nec, ultrices quam. Ut scelerisque, nibh vitae bibendum tincidunt, urna nisl viverra nulla, in mollis erat purus a mauris'
        -
            title: 'Golden Sand Dunes'
            image: sergey-pesterev-_VqyrvQi6do-unsplash.jpg
            desc: 'Nulla tempus, ex vel commodo ullamcorper, velit leo sodales odio, ut viverra quam odio ut lacus. Maecenas a dolor quis risus aliquam interdum. Duis id consectetur odio, at iaculis lacus. Fusce lacinia maximus tortor, ac lacinia odio rutrum sed. Fusce tempor quis lectus eu consequat. Quisque orci nisl, eleifend eu risus ut, porta pretium ex.'
        -
            title: 'Streets of Tokyo'
            image: sorasak-_UIN-pFfJ7c-unsplash.jpg
            desc: 'Nullam pellentesque justo magna. Nunc a faucibus mauris. Quisque a hendrerit sem, id scelerisque arcu. Aliquam laoreet faucibus elit, at volutpat metus finibus et. Aenean tempor justo nibh, quis dignissim sapien aliquet eu. Vivamus risus neque, convallis ac laoreet sit amet, condimentum id tellus. Quisque porttitor arcu vitae sem sollicitudin efficitur.'
        -
            title: 'A Day of Scuba'
            image: thomas-ashlock-RAjND0B3HDw-unsplash.jpg
            desc: 'Aliquam id convallis ante. Mauris nec purus lacinia, tempus mauris eget, blandit tortor. Ut ut dolor vitae lacus rutrum venenatis sed sed augue. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis egestas rhoncus commodo. Quisque id erat pharetra, consectetur nisl a, viverra nulla. Nunc quis euismod lorem. Aliquam erat volutpat. Fusce tincidunt mauris sed justo suscipit tempus.'
        -
            title: 'Travels with a VW Bus'
            image: simon-rae-kSJTEv9w5l4-unsplash.jpg
            desc: 'Maecenas maximus ipsum et dui pulvinar, vitae venenatis lectus sagittis. Praesent pellentesque felis in orci rhoncus pellentesque. Pellentesque mollis sed nibh sit amet vehicula. Integer vitae diam eget nisi feugiat rhoncus. Curabitur tristique nisl leo, et rutrum purus elementum et. Ut tincidunt mauris at nibh sagittis hendrerit. Vestibulum ut diam magna.'
        -
            title: 'Waterfall and Pool'
            image: stefan-stefancik-0wMmxNB6Xzc-unsplash.jpg
            desc: 'Pellentesque mattis enim non pharetra semper. Donec quis leo ut dolor ornare tempus. Mauris sagittis nisi ut mi malesuada, et commodo orci feugiat. Integer consectetur consequat lorem, vehicula faucibus mauris. Phasellus cursus mauris malesuada sapien rutrum, pharetra dictum tellus molestie. Integer vitae lectus sit amet mauris pulvinar rhoncus euismod ac dolor.'
```

## Благодарности

Этот плагин построен на замечательной библиотеке [GLightbox](https://biati-digital.github.io/glightbox?target=_blank) от `biati digital`
