---
description: Как добавить свои поля в интерфейс настроек админки.
---

# Пример: чертежи конфигурации

Обычно в `site.yaml` добавляются некоторые параметры конфигурации, которые будут отображаться в содержимом сайта.

Чтобы сделать эти параметры настраиваемыми через панель админки, добавьте несколько полей в `user/blueprints/config/site.yaml`. Например:


```yaml
@extends:
    '@parent'

form:
    fields:
        content:

            fields:
                myfield:
                    type: text
                    label: My Field
```

Добавит тип ввода «Мое поле», добавив его в раздел «Контент» конфигурации сайта.

Вы также можете добавлять целые новые разделы, например:

```yaml
@extends:
    '@parent'

form:
    fields:
        anothersection:
            type: section
            title: Another Section
            underline: true

            fields:
                myfield:
                    type: text
                    label: A label
                    size: large
```
