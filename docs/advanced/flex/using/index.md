---
description: Как использовать объекты Flex в Grav CMS. Шаблоны коллекций и объектов.
---

# Использование объектов Flex

**Объекты Flex** предназначены для того, чтобы быть простыми в использовании. Отображение коллекций и групп на ваших страницах в основном можно реализовать с помощью шаблонов Twig.

!!! tip ""

	Чтобы включить и отобразить каталог Flex, пожалуйста, прочтите **[Включение каталога](/advanced/flex/administration/introduction)**

## Использование типа страницы `flex-objects`

Отображение нескольких каталогов на странице `directories/flex-objects.md`:

```yaml
title: Directories
flex:
  layout: default
  list:
  - contacts
  - services
---

# Directories
```

В качестве альтернативы вы можете передать отдельные параметры для каждого каталога:

```yaml
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

Отображение одного каталога на странице `contacts/flex-objects.md`:

```yaml
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

Отображение одного объекта на странице `my-contact/flex-objects.md`:

```yaml
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

По умолчанию тип страницы `flex-objects` принимает два параметра URL-адреса: **directory** и **id**. Они используются для навигации по каталогам. Пример url-адресов выглядит следующим образом:

```text
https://www.domain.com/directories/directory:contacts/id:ki2ts4cbivggmtlj

https://www.domain.com/contacts/id:ki2ts4cbivggmtlj
```

!!! tip ""

	Вы можете передать свои собственные параметры внутри `flex` и использовать их в своей коллекции и файлах шаблонов объектов.

## Отрисовка коллекций и объектов

И Коллекции, и Объекты поддерживают рендеринг своего вывода в HTML. Вывод можно настроить с помощью двух параметров: макета и контекста. Макет позволяет вам настраивать внешний вид, например, у вас может быть список карточек, а затем более подробный вывод для деталей. Контекст позволяет передавать переменные, которые будут использоваться в файлах шаблонов.

```twig
{% render collection layout: 'custom' with { context_variable: true } %}

{% render object layout: 'custom' with { context_variable: true } %}
```

Смотрите более подробную документацию: [Визуализация коллекции](collection/#render) и [визуализация объекта](object/#render).

## Основы создания шаблонов

Шаблоны Flex находятся в папке `templates/flex`:

```yaml
templates/
  flex/
    contacts/
      collection/
        default.html.twig
      object/
        default.html.twig
```

У каждого типа есть две папки: одна для коллекции рендеринга, а другая - для объекта рендеринга. Файлы внутри - это макеты, названные по имени файла. В нашем примере у нас есть макет `default` как для коллекции, так и для объекта.

### Шаблон коллекции

Шаблон коллекции `flex/contacts/collection/default.html.twig` отвечает за рендеринг всех объектов в коллекции. Визуализированный вывод по умолчанию кэшируется. Ключ кэша определяется коллекцией и контекстом, передаваемым методу `render()`.

!!! warning ""

	Если контекст содержит нескалярные значения, кэширование будет отключено. Постарайтесь делать контекст как можно более простым!

Вот пример для Flex-типа «Контакты»:
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

!!! tip ""

	Если визуализированный HTML-код имеет динамическое содержимое, кэш визуализации можно отключить в шаблоне Twig с помощью `{% do block.disableCache() %}`.

### Шаблон объекта

Шаблон объекта `flex/contacts/object/default.html.twig` отвечает за рендеринг одного объекта. Визуализированный вывод по умолчанию кэшируется. Ключ кеширования определяется объектом и контекстом, передаваемым методу `render()`.

!!! warning ""

	Если контекст содержит нескалярные значения, кэширование будет отключено. Постарайтесь делать контекст как можно более простым!

Вот пример для Flex-типа «Контакты»:
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

!!! tip ""

	Если визуализированный HTML-код имеет динамическое содержимое, кэш визуализации можно отключить в шаблоне Twig с помощью `{% do block.disableCache() %}`.

### Пользовательские макеты

Используя настраиваемые макеты, вы можете создавать бесконечное количество различных представлений как для своих коллекций, так и для объектов.

Вы можете создавать свои собственные макеты, просто добавляя новый файл рядом с `default.html.twig`. Базовое имя файла совпадает с именем вашего макета.

!!! tip ""

	В макетах коллекций рекомендуется вызывать `{% render object layout: 'xxx' %}` вместо вывода переменных объекта непосредственно в шаблон коллекции.
