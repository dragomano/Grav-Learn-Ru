---
description: Доступные типы контента в Grav CMS. Пользовательские типы.
---

# Типы контента

## Тип контента по умолчанию

Как обычно на большинстве веб-платформ, тип контента Grav по умолчанию - **HTML**. Это означает, что когда пользователь запрашивает маршрут в своем браузере, например: `/blog/new-macbook-pros-soon`, Grav предполагает, что вы запрашиваете страницу HTML. Если ваша страница была определена страницей с именем файла `blog-item.md`, Grav, в свою очередь, ищет шаблон Twig под названием `blog-item.html.twig` для отображения страницы.

Если пользователь явно запросит тип через `/blog/new-macbook-pros-soon.html`, Grav все равно будет искать тот же файл` blog-item.html.twig`.

## Другие типы контента

Однако Grav является гибкой платформой и может фактически обслуживать любой тип контента, который вы можете пожелать (`xml`,` rss`, `json`, `pdf` и т. д.). Вам просто нужно предоставить способ его рендеринга.

Если вы запрашиваете маршрут с расширением `.xml`, например: `/blog.xml`, вместо использования обычного шаблона `blog.html.twig` для его визуализации,
Grav ищет шаблон под названием `blog.xml.twig`. Вы должны убедиться, что шаблон выводит соответствующую структуру XML.

### Пример с JSON-файлами

Конкретный распространенный способ доступа к файлам - расширение `.json`. Это позволяет запрашивать данные через файлы JSON, которые легко обрабатываются JavaScript.

Скажем, вы хотели бы сохранить **frontmatter** и **content** конкретной страницы в формате JSON, и эта страница должна быть определена в файле с именем `item.md`. Всё, что вам нужно сделать, это предоставить шаблон Twig с именем `item.json.twig`. Вы можете поместить это в папку `templates/` вашей темы, или, если вы используете плагин для загрузки пользовательских шаблонов, вы можете добавить его туда.

Содержимое этого файла `item.json.twig` может выглядеть примерно так:

```twig
{% set payload = {frontmatter: page.header, content: page.content}  %}
{{ payload|json_encode|raw }}
```

Всё, что делает этот файл Twig, это создает массив с заголовком страницы как **frontmatter** и **content**, затем использует фильтр Twig `json_encode` для его кодирования.

Когда пользователь запрашивает URL `/blog/new-macbook-pros-soon.json`, будет использоваться этот новый файл Twig и отправленный вывод будет иметь следующий формат:

```json
{
   "frontmatter":{
      "title":"New Macbook Pros Arriving Soon",
      "date": "14:23 08/01/2016",
      "taxonomy":{
         "category":[
            "blog"
         ],
         "tag":[
            "apple",
            "mbpr",
            "laptops"
         ]
      }
   },
   "content":"<p>this has an -&gt; arrow here and <strong>bold</strong> here</p>\n<blockquote>\n<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec ultricies tristique nulla et mattis. Phasellus id massa eget nisl congue blandit sit amet id ligula. Praesent et nulla eu augue tempus sagittis. Mauris faucibus nibh et nibh cursus in vestibulum sapien egestas. Curabitur ut lectus tortor. Sed ipsum eros, egestas ut eleifend non, elementum vitae eros.\n-- <cite> Ronald Wade</cite></p>\n</blockquote>\n<p>Mauris felis diam, pellentesque vel lacinia ac, dictum a nunc. Mauris mattis nunc sed mi sagittis et facilisis tortor volutpat. Etiam tincidunt urna mattis erat placerat placerat ac eu tellus.</p>\n<p>This is a new paragraph</p>\n<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec ultricies tristique nulla et mattis.</p>"
}
```

Это действительный JSON, который можно легко анализировать и обрабатывать с помощью JavaScript. Очень просто!

## Пользовательские типы

Чтобы отправлять данные с соответствующим типом контента, Grav должен знать тип MIME, который ожидает браузер, чтобы он отображал этот тип контента. Grav знает о большинстве стандартных типов контента, определённых в файле `system/config/media.yaml`. Если вы хотите обработать тип содержимого, который не предоставляется, вам просто нужно добавить запись в соответствующий файл.

Например, если вы хотите отображать события календаря iCal, вам нужно добавить этот тип мультимедиа в `user/config/media.yaml`:

```yaml
  ics:
    type: iCal
    thumb: media/thumb.png
    mime: text/calendar
```

Это определяет расширение файла `.ics` как файл` iCal` с типом mime: `text/calendar`. Затем все, что вам нужно сделать, это предоставить соответствующий шаблон `.ical.twig` для рендеринга любого файла этого типа, который вы запрашиваете.
