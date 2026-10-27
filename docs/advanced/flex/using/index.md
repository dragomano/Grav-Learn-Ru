---
title: "Работа с Flex-объектами"
description: "Flex Objects рассчитан на простоту использования. Показывать коллекции и группы на страницах чаще всего можно прямо из шаблонов Twig."
tags:
  - "Flex-объекты"
  - "Twig"
---

# Работа с Flex-объектами

**Flex Objects** рассчитан на простоту использования. Показывать коллекции и группы на страницах чаще всего можно прямо из шаблонов Twig.

> [!WARNING] Предупреждение
> **Совет:** чтобы включить и отобразить директорию Flex, прочитайте раздел **[Включение директории](../administration/introduction/index.md)**.

## Использование типа страницы `flex-objects`

Вывод нескольких директорий на странице `directories/flex-objects.md`:

```text
title: Directories
flex:
  layout: default
  list:
  - contacts
  - services
---

# Directories
```

Либо можно передать отдельные параметры для каждой директории:

```text
title: Directories
flex:
  layout: default
  directories:
    contacts:
      collection:
        title: '{{ directory.title }}'
        layout: default
        object:
          layout: list-default
      object:
        title: 'Contact: {{ object.first_name }} {{ object.last_name }}'
        layout: default
    services:
---

# Directories
```

Вывод одной директории на странице `contacts/flex-objects.md`:

```text
title: Contacts
flex:
  directory: contacts
  collection:
    title: '{{ directory.title }}'
    layout: default
    object:
      layout: list-default
  object:
    title: 'Contact: {{ object.first_name }} {{ object.last_name }}'
    layout: default
---

# Contacts
```

Вывод одного объекта на странице `my-contact/flex-objects.md`:

```text
title: Contact
flex:
  directory: contacts
  id: ki2ts4cbivggmtlj
  object:
    title: 'Contact: {{ object.first_name }} {{ object.last_name }}'
    layout: default
---

# Contacts
```

По умолчанию тип страницы `flex-objects` принимает два параметра URL: **directory** и **id**. Они используются для навигации по директориям. Примеры URL выглядят так:

```text
https://www.domain.com/directories/directory:contacts/id:ki2ts4cbivggmtlj

https://www.domain.com/contacts/id:ki2ts4cbivggmtlj
```

> [!WARNING] Предупреждение
> **Совет:** в блок `flex` можно передать собственные параметры и использовать их в файлах шаблонов коллекции и объекта.

## Рендеринг коллекций и объектов

И коллекции, и объекты поддерживают рендеринг своего вывода в HTML. Вывод можно настроить двумя параметрами: layout и context. Layout позволяет задать собственный вид — например, список карточек и более подробный вывод для деталей. Context позволяет передавать в файлы шаблонов переменные.

```twig
{% render collection layout: 'custom' with { context_variable: true } %}

{% render object layout: 'custom' with { context_variable: true } %}
```

Подробнее см. разделы [Рендеринг коллекции](collection/index.md#render) и [Рендеринг объекта](object/index.md#render).

## Основы шаблонизации

Шаблоны Flex лежат в папке `templates/flex`:

```text
templates/
  flex/
    contacts/
      collection/
        default.html.twig
      object/
        default.html.twig
```

У каждого типа есть две папки: одна для рендеринга коллекции, другая для рендеринга объекта. Файлы внутри — это макеты, названные по имени файла. В нашем примере у нас есть макет `default` и для коллекции, и для объекта.

### Шаблон коллекции

Шаблон коллекции `flex/contacts/collection/default.html.twig` отвечает за рендеринг всех объектов коллекции. Отрендеренный вывод по умолчанию кэшируется. Ключ кэша определяется коллекцией и контекстом, переданным в метод `render()`.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** если контекст содержит нескалярные значения, кэширование отключается. Старайтесь держать контекст как можно проще!

Вот пример из типа Contacts:
```twig
<div id="flex-objects">
  <div class="text-center">
    <input class="form-input search" type="text" placeholder="Search by name, email, etc" />
    <button class="button button-primary sort asc" data-sort="name">
      Sort by Name
    </button>
  </div>

  <ul class="list">
    {% for object in collection.filterBy({ published: true }) %}
      <li>
        {% render object layout: layout with { options: options } %}
      </li>
    {% endfor %}
  </ul>
</div>

<script>
    var options = {
        valueNames: [ 'name', 'email', 'website', 'entry-extra' ]
    };
    var flexList = new List('flex-objects', options);
</script>
```

> [!WARNING] Предупреждение
> **Совет:** если отрендеренный HTML содержит динамический контент, кэш рендеринга можно отключить из шаблона Twig с помощью `{% do block.disableCache() %}`.

### Шаблон объекта

Шаблон объекта `flex/contacts/object/default.html.twig` отвечает за рендеринг одного объекта. Отрендеренный вывод по умолчанию кэшируется. Ключ кэша определяется объектом и контекстом, переданным в метод `render()`.

> [!CAUTION] Внимание
> **ВНИМАНИЕ:** если контекст содержит нескалярные значения, кэширование отключается. Старайтесь держать контекст как можно проще!

Вот пример из типа Contacts:
```twig
<div class="entry-details">
    {% if object.website %}
        <a href="{{ object.website|e }}"><span class="name">{{ object.last_name|e }}, {{ object.first_name|e }}</span></a>
    {% else %}
        <span class="name">{{ object.last_name|e }}, {{ object.first_name|e }}</span>
    {% endif %}
    {% if object.email %}
        <p><a href="mailto:{{ object.email|e }}" class="email">{{ object.email|e }}</a></p>
    {% endif %}
</div>
<div class="entry-extra">
    {% for tag in object.tags %}
        <span>{{ tag|e }}</span>
    {% endfor %}
</div>
```

> [!WARNING] Предупреждение
> **Совет:** если отрендеренный HTML содержит динамический контент, кэш рендеринга можно отключить из шаблона Twig с помощью `{% do block.disableCache() %}`.

### Собственные макеты

С помощью собственных макетов можно создать бесконечное множество разных видов и коллекций, и объектов.

Создать собственный макет можно, просто добавив новый файл рядом с `default.html.twig`. Имя файла без расширения совпадает с именем макета.

> [!WARNING] Предупреждение
> **Совет:** в макетах коллекций рекомендуется вызывать `{% render object layout: 'xxx' %}` вместо вывода переменных объекта напрямую в шаблоне коллекции.
