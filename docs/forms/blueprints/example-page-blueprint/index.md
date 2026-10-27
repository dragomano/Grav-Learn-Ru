---
title: "Пример: схемы страниц"
description: "Схемы страниц расширяют страницу по умолчанию и позволяют добавлять параметры. По сути, настраиваемые страницы оживают благодаря схемам страниц. Со схемой страницы можно на 100% настроить форму редактирования…"
tags:
  - "Формы"
  - "Схемы"
  - "Страницы"
---

# Пример: схемы страниц

**Схемы страниц** расширяют страницу по умолчанию и позволяют добавлять параметры. По сути, настраиваемые страницы оживают благодаря схемам страниц. Со схемой страницы можно на 100% настроить форму редактирования страницы в том виде, в каком она отображается в админке.

### Первый пример

Если вы хотите использовать стандартную форму страницы и просто добавить пару выпадающих списков, можно расширить страницу по умолчанию.

Будет использована стандартная форма страницы, а текстовое поле добавится на вкладку **Advanced** в раздел **Overrides**:

```yaml linenums="1"
title: Gallery
extends@:
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

А так добавляется новая вкладка **Gallery** с несколькими полями.

```yaml linenums="1"
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

Типы полей, которые можно добавлять, перечислены в разделе [Доступные поля формы для использования в админке](../fields-available/index.md).

### Как называть поля

Важно, чтобы поля использовали структуру `header.*`, тогда содержимое поля сохраняется в **Page Header**.

### Создание полностью настраиваемой формы страницы

Можно не расширять стандартную форму, а создать полностью уникальную форму страницы.

Например:

```yaml linenums="1"
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

> [!CAUTION] Внимание
> **ПРЕДУПРЕЖДЕНИЕ.** Поле `route` изменилось в Grav 1.7. Обновите существующие схемы, чтобы использовать новый `type: parents`.

### Примечание для экспертного режима

При редактировании страниц в режиме **Expert** схема **Blueprint** не читается, и форма страницы одинакова для всех страниц. Это потому, что в экспертном режиме вы редактируете поля страницы напрямую в поле **Frontmatter**, и настраиваемое представление не нужно.

### Куда помещать схемы страниц

Чтобы плагин Admin подхватил схемы и показал новые типы страниц, схемы нужно поместить в правильное место.

#### В папке пользовательских схем

Поместите их в `user/blueprints/pages/`. Это хорошее место, если вы просто хотите, чтобы схемы присутствовали на сайте.

#### В теме

Поместите их в `user/themes/YOURTHEME/blueprints/`. Это лучше всего, если вы также собираетесь распространять тему: тема будет предоставлять схемы страниц, и ими будет проще пользоваться.

#### В папке data

Если вы используете тему на основе Gantry5, лучшее место — `user/data/gantry5/themes/YOURTHEME/blueprints/`, иначе ваши файлы могут быть потеряны при обновлении темы.

#### В плагине

Поместите их в `user/plugins/YOURPLUGIN/blueprints/`. Это место подойдёт, если вы определяете и добавляете настраиваемые страницы в плагине.

Затем подпишитесь на событие `onGetPageBlueprints` и добавьте их в Grav. Следующий пример добавляет схемы из папки `blueprints/`.

```php linenums="1"
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
