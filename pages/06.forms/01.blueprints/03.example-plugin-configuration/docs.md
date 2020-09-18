---
title: 'Пример: конфигурация плагина'
taxonomy:
    category: docs
---

Мы видели в [предыдущем примере](../example-plugin-blueprint), как определить план для плагина и/или темы.

Теперь давайте посмотрим, как предложить параметры конфигурации для плагина или темы, которые будут отображаться в плагине админки.

Если вы хотите, чтобы у вашего плагина (или темы) были параметры, настраиваемые напрямую из интерфейса администратора, вам необходимо заполнить файл blueprints.yaml формами.

Например, вот файл **archives.yaml** плагина **Archives**:

[prism classes="language-yaml line-numbers"]
enabled: true
built_in_css: true
date_display_format: 'F Y'
show_count: true
limit: 12
order:
    by: date
    dir: desc
filter_combinator: and
filters:
    category: blog
[/prism]

Это настройки плагина по умолчанию. Без подключаемого модуля администратора для настройки этих параметров пользователю необходимо скопировать этот файл в папку `/user/config/plugins/` и их туда.

Предоставляя правильно отформатированный файл **blueprints.yaml**, вы можете разрешить пользователю изменять настройки из интерфейса администратора. Когда настройки сохраняются, они автоматически записываются в `/user/config/plugins/archives.yaml` (или в `config/themes`, если это тема). Структура начинается следующим образом:

[prism classes="language-yaml line-numbers"]
name: Archives
version: 1.3.0
description: The **Archives** plugin creates links for pages grouped by month/year
icon: university
author:
  name: Team Grav
  email: devs@getgrav.org
  url: https://getgrav.org
homepage: https://github.com/getgrav/grav-plugin-archives
demo: http://demo.getgrav.org/blog-skeleton
keywords: archives, plugin, blog, month, year, date, navigation, history
bugs: https://github.com/getgrav/grav-plugin-archives/issues
license: MIT

form:
  validation: strict
  fields:
[/prism]

А вот и та часть, которая нам нужна. Каждому полю в файле **archives.yaml** нужен соответствующий элемент формы, например:

**Переключатель**

[prism classes="language-yaml line-numbers"]
enabled:
  type: toggle
  label: Plugin status
  highlight: 1
  default: 1
  options:
      1: Enabled
      0: Disabled
  validate:
       type: bool
[/prism]

**Список выбора**

[prism classes="language-yaml line-numbers"]
date_display_format:
  type: select
  size: medium
  classes: fancy
  label: Date Format
  default: 'jS M Y'
  options:
    'F jS Y': "January 1st 2014"
    'l jS of F': "Monday 1st of January"
    'D, m M Y': "Mon, 01 Jan 2014"
    'd-m-y': "01-01-14"
    'jS M Y': "10th Feb 2014"
[/prism]

**Текстовое поле**

[prism classes="language-yaml line-numbers"]
limit:
  type: text
  size: x-small
  label: Count Limit
  validate:
    type: number
    min: 1
[/prism]

Корневой элемент (в этих примерах `enabled`, `date_display_format`, `limit`) - это имя параметра. Дополнительные компоненты каждого поля определяют, как это поле отображается. Например, его тип (`type`), его размер (`size`), показанная метка (`label`) и дополнительная полезная подсказка, которая появляется при наведении курсора (`help`). `default` и `placeholder` позволяют вам создавать некоторые значения по умолчанию и улучшать отображение полей для пользователя.

Остальные поля могут меняться в зависимости от типа поля. Например, для типа поля `select` требуется и список `options`.

Вложенные параметры доступны через точечную нотацию (например, `order.dir`)

[prism classes="language-yaml line-numbers"]
order.dir:
  type: toggle
  label: Order Direction
  highlight: asc
  default: desc
  options:
    asc: Ascending
    desc: Descending
[/prism]

Плагин `Admin` определяет множество других типов полей, которые можно использовать в `plugins/admin/themes/grav/templates/forms/fields`.

Важно отметить, что когда для параметра `form.validation` установлено значение `strict`, как в примере плагина **Archives**, вам нужно добавить схемы форм для _все_ параметры, иначе при сохранении появится сообщение об ошибке.
Если вы вместо этого хотите разрешить настраивать несколько полей в интерфейсе администратора, а не все из них, установите для параметра `form.validation` значение `loose`.