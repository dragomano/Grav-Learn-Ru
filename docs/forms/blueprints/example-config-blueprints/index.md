---
title: "Пример: схемы конфигурации"
description: "Часто в site.yaml добавляют некоторые параметры конфигурации, чтобы они отображались в содержимом сайта."
tags:
  - "Формы"
  - "Схемы"
  - "Конфигурация"
---

# Пример: схемы конфигурации

Часто в site.yaml добавляют некоторые параметры конфигурации, чтобы они отображались в содержимом сайта.

Чтобы сделать эти параметры настраиваемыми через админку, добавьте поля в `user/blueprints/config/site.yaml`. Например:

```yaml linenums="1"
extends@:
    '@parent'

form:
    fields:
        content:

            fields:
                myfield:
                    type: text
                    label: My Field
```

Это добавит поле ввода «My Field», прикрепив его к разделу Content конфигурации Site.

Можно добавлять и целые новые разделы, например:

```yaml linenums="1"
extends@:
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
