---
title: "Пример: конфигурация плагина"
description: "В предыдущем примере мы увидели, как определить схему плагина и/или темы."
tags:
  - "Формы"
  - "Схемы"
  - "Плагины"
---

# Пример: конфигурация плагина

В [предыдущем примере](../example-plugin-blueprint/index.md) мы увидели, как определить схему плагина и/или темы.

Теперь посмотрим, как предложить параметры конфигурации для плагина или темы, которые будет показывать плагин Admin.

Если вы хотите, чтобы у плагина (или темы) были параметры, настраиваемые прямо из интерфейса админки, нужно заполнить файл blueprints.yaml формами.

Например, вот файл **archives.yaml** плагина **Archives**:

```yaml linenums="1"
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
```

Это настройки плагина по умолчанию. Без плагина Admin для настройки этих параметров пользователю нужно скопировать этот файл в папку `/user/config/plugins/` и редактировать значения там.

Предоставив правильно оформленный файл **blueprints.yaml**, вы можете позволить пользователю менять настройки из интерфейса админки. При сохранении настройки автоматически записываются в `/user/config/plugins/archives.yaml` (или в config/themes, если это тема). Структура начинается так:

```yaml linenums="1"
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
```

Здесь начинается нужная нам часть. Каждому полю в файле **archives.yaml** нужен соответствующий элемент формы, например:

**Переключатель**

```yaml linenums="1"
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
```

**Список**

```yaml linenums="1"
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
```

**Текст**

```yaml linenums="1"
limit:
  type: text
  size: x-small
  label: Count Limit
  validate:
    type: number
    min: 1
```

Корневой элемент (в этих примерах `enabled`, `date_display_format`, `limit`) — это имя параметра. Дополнительные компоненты каждого поля определяют, как это поле отображается. Например, его тип (`type`), размер (`size`), отображаемая метка (`label`) и необязательная полезная подсказка, появляющаяся при наведении (`help`). `default` и `placeholder` позволяют задать значения по умолчанию и улучшить то, как поле выглядит для пользователя.

Остальные поля могут меняться в зависимости от типа поля. Например, типу поля `select` требуется список `options`.

К вложенным параметрам можно обращаться через точечную нотацию (например, `order.dir`).

```yaml linenums="1"
order.dir:
  type: toggle
  label: Order Direction
  highlight: asc
  default: desc
  options:
    asc: Ascending
    desc: Descending
```

Плагин Admin определяет множество других типов полей, которые можно использовать, в `plugins/admin/themes/grav/templates/forms/fields`.

Важно отметить: если `form.validation` установлен в `strict`, как в примере с плагином **Archives**, нужно добавить схемы форм для _всех_ параметров, иначе при сохранении появится ошибка.
Если же вы хотите разрешить настраивать в админке лишь несколько полей, а не все, установите `form.validation` в `loose`.
