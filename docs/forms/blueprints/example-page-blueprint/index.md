---
description: Изменение чертежей страниц для добавления произвольных полей.
---

# Пример: чертежи страниц

**Чертежи страниц** расширяются от страницы по умолчанию и дают вам возможность добавлять параметры. По сути, пользовательские страницы можно оживить, используя чертежи страниц. С помощью схемы страницы вы можете на 100% настроить форму редактирования для страницы, как она отображается в админке.

### Пример первый

Если вы хотите использовать форму страницы по умолчанию и просто добавить, например, пару полей выбора, вы можете расширить страницу со страницы по умолчанию.

Это будет использовать форму страницы по умолчанию и добавить текстовое поле на вкладку **Дополнительно** в разделе **Переопределения**:

```yaml
title: Gallery
'@extends':
    type: default
    context: blueprints://pages

form:
  fields:
    tabs:
      type: tabs
      active: 1

      fields:
        advanced:
          fields:
            overrides:
              fields:
                header.an_example_text_field:
                  type: text
                  label: Add a number
                  default: 5
                  validate:
                    required: true
                    type: int
```

Вместо этого будет добавлена ​​новая вкладка **Галерея** с некоторыми полями.

```yaml
title: Gallery
'@extends':
    type: default
    context: blueprints://pages

form:
  fields:
    tabs:
      type: tabs
      active: 1

      fields:
        gallery:
          type: tab
          title: Gallery

          fields:
            header.an_example_text_field:
              type: text
              label: Add a number
              default: 5
              validate:
                required: true
                type: int

            header.an_example_select_box:
              type: select
              label: Select one of the following
              default: one
              options:
                one: One
                two: Two
                three: Three
```

Типы полей, которые вы можете добавить, перечислены в главе [Доступные поля формы для использования в админке](../fields-available)

### Как именовать поля

Важно, чтобы поля использовали структуру `header.*`, Поэтому содержимое поля при сохранении сохраняется в **Заголовок страницы**.

### Создайте полностью настраиваемую форму страницы

Вы можете избежать расширения формы по умолчанию и создать полностью уникальную форму страницы.

Например:

```yaml
title: Gallery

form:
  fields:
    tabs:
      type: tabs
      active: 1

      fields:
        gallery:
          type: tab
          title: Gallery

          fields:
            header.an_example_text_field:
              type: text
              label: Add a number
              default: 5
              validate:
                required: true
                type: int

            header.an_example_select_box:
              type: select
              label: Select one of the following
              default: one
              options:
                one: One
                two: Two
                three: Three

            route:
              type: parents
              label: PLUGIN_ADMIN.PARENT
              classes: fancy

```

!!! warning ""

	Поле `route` изменилось в Grav 1.7. Пожалуйста, обновите существующие чертежи для использования нового `type: parents`.

### Примечание для экспертного режима

При редактировании страниц в режиме **Expert** **чертеж** не читается, и форма страницы одинакова для всех страниц. Это связано с тем, что в экспертном режиме вы редактируете поля страницы непосредственно в поле **Frontmatter**, и нет необходимости в настраиваемой презентации.

### Где разместить чертежи страницы

Чтобы плагин админки собирал чертежи и, таким образом, отображал новые типы страниц, вам необходимо разместить чертежи в правильном месте.

#### В пользовательской папке Blueprints

Поместите их в `user/blueprints/pages/`. Это хорошее место для их размещения, когда вы просто хотите, чтобы ваши чертежи присутствовали на вашем сайте.

#### В теме

Поместите их в `user/themes/YOURTHEME/blueprints/`. Это лучше всего, если вы также собираетесь распространять свою тему: тема будет содержать чертежи страниц, и её будет проще использовать.

#### В папке Data

Если вы используете тему на основе Gantry5, лучшее место - `user/data/gantry5/themes/YOURTHEME/blueprints/`, иначе ваши файлы могут быть потеряны во время обновления темы.

#### В плагине

Поместите их в `user/plugins/YOURPLUGIN/blueprints/`. Это место, куда их можно поместить, если вы определяете и добавляете пользовательские страницы в плагин.

Затем подпишитесь на событие `onGetPageBlueprints` и добавьте их в Grav. В следующем примере добавляются чертежи из папки `blueprints/`.

```php
<?php

public static function getSubscribedEvents()
{
  return [
    'onGetPageBlueprints' => ['onGetPageBlueprints', 0]

  ];
}

public function onGetPageBlueprints($event)
{
  $types = $event->types;
  $types->scanBlueprints('plugins://' . $this->name . '/blueprints');
}
```
