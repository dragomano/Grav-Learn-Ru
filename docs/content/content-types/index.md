---
title: "Типы контента"
description: "Узнайте о типах контента в Grav."
tags:
  - "Контент"
  - "Страницы"
---

# Типы контента

## Тип контента по умолчанию

Как и в большинстве веб-платформ, тип контента Grav по умолчанию — **HTML**. Это значит, что когда пользователь запрашивает в браузере маршрут, например `/blog/new-macbook-pros-soon`, Grav, поскольку расширение файла отсутствует, считает, что вы запрашиваете HTML-страницу.

---

Если страница задана файлом `blog-item.md`, Grav ищет Twig-шаблон с именем `blog-item.html.twig`, чтобы отрендерить страницу.

Если пользователь явно запросит тип через `/blog/new-macbook-pros-soon.html`, Grav всё равно будет искать тот же файл `blog-item.html.twig`.

## Другие типы контента

Однако Grav — гибкая платформа и может отдавать любой тип контента, какой пожелаете (`xml`, `rss`, `json`, `pdf` и т. д.), нужно лишь предусмотреть способ правильно его отрендерить.

Если запросить маршрут с расширением `.xml`, например `/blog.xml`, Grav вместо обычного шаблона `blog.html.twig` будет искать шаблон с именем `blog.xml.twig`. Вам нужно убедиться, что этот шаблон выводит правильную структуру XML.

### Пример с файлами JSON

Очень распространённый способ доступа к файлам — через расширение `.json`. Это позволяет запрашивать данные в виде JSON-файлов, которые легко обрабатываются JavaScript.

Допустим, вы хотите получить **метаданные страницы** и **контент** конкретной страницы в формате JSON, а страница задана файлом `item.md`. Всё, что нужно, — создать Twig-шаблон `item.json.twig`. Его можно положить в папку `templates/` вашей темы или, если вы используете плагин для загрузки собственных шаблонов, добавить туда.

Содержимое файла `item.json.twig` может выглядеть примерно так:

```twig linenums="1"
{% set payload = {frontmatter: page.header, content: page.content}  %}
{{ payload|json_encode|raw }}
```

Этот Twig-файл всего лишь создаёт массив, где заголовок страницы используется как **метаданные страницы**, а содержимое — как **контент**, затем кодирует его фильтром Twig `json_encode`.

Когда пользователь запрашивает URL `/blog/new-macbook-pros-soon.json`, используется новый Twig-файл, а вывод отправляется в таком формате:

```json linenums="1"
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

Это корректный JSON, который легко разобрать и обработать в JavaScript. Проще простого!

## Собственные типы контента

Чтобы отправить данные с правильным типом контента, Grav нужно знать MIME-тип, который ожидает браузер для отображения этого типа контента. Grav знает большинство стандартных типов контента, определённых в файле `system/config/media.yaml`. Если вы хотите обрабатывать тип контента, которого там нет, достаточно добавить запись в этот файл.

Например, чтобы отдавать события календаря iCal, нужно добавить этот медиатип в `media.yaml`:

```yaml linenums="1"
ics:
  type: iCal
  thumb: media/thumb.png
  mime: text/calendar
```

Это определяет расширение файла `.ics` как файл `iCal` с MIME-типом `text/calendar`. Затем нужно лишь создать подходящий шаблон `.ics.twig`, чтобы отрендерить любой запрашиваемый файл этого типа.
