---
title: "Теги Twig"
description: "Grav также предоставляет различные пользовательские теги Twig, расширяющие и без того широкие возможности шаблонизации Twig несколькими новыми тегами, которые мы сочли полезными."
tags:
  - "Темы"
  - "Grav 1.7"
  - "Twig"
---

# Теги Twig

Grav также предоставляет различные пользовательские теги Twig, расширяющие и без того широкие возможности шаблонизации Twig несколькими новыми тегами, которые мы сочли полезными.

### markdown

Тег Markdown предоставляет мощный новый способ встраивать Markdown в шаблон Twig. Можно использовать переменную и рендерить её фильтром `|markdown`, но синтаксис `{% markdown %}` делает создание блоков текста Markdown ещё проще.

```twig title="Twig" linenums="1"
{% markdown %}
This is **bold** and this _underlined_

1. This is a bullet list
2. This is another item in that same list
{% endmarkdown %}
```

```html title="Вывод" linenums="1"
<p>This is <strong>bold</strong> and this <em>underlined</em></p>
<ol>
<li>This is a bullet list</li>
<li>This is another item in that same list</li>
</ol>
```

### script

Тег Script — по сути удобный тег, который делает Twig более читаемым по сравнению с обычным подходом `{% do assets...%}`. Это просто альтернативный способ записи.

#### Файл скрипта

```twig title="Twig"
{% script 'theme://js/something.js' at 'bottom' priority: 20 with { defer: true, async: true } %}
```

```html title="Вывод"
<script src="/user/themes/mytheme/js/something.js" defer async></script>
```

Grav 1.7.28 также добавляет поддержку модулей:

```twig title="Twig"
{% script module 'theme://js/module.mjs' %}
```

```html title="Вывод"
<script type="module" src="/user/themes/mytheme/js/module.mjs"></script>
```

#### Встроенный скрипт

```twig title="Twig"
{% script at 'bottom' priority: 20 %}
    alert('Warning!');
{% endscript %}
```

```html title="Вывод"
<script>
    alert('Warning!');
</script>
```

### style

#### CSS-файл

```twig title="Twig"
{% style 'theme://css/foo.css' priority: 20 %}
```

```html title="Вывод"
<link rel="stylesheet" href="/user/themes/mytheme/css/foo.css">
```

#### Встроенный CSS

```twig title="Twig"
{% style priority: 20 with { media: 'screen' } %}
    a { color: red; }
{% endstyle %}
```

```html title="Вывод"
<style media="screen">
    a { color: red; }
</style>
```

### link

```twig title="Twig"
{% link icon 'theme://images/favicon.png' priority: 20 with { type: 'image/png' } %}
{% link modulepreload 'plugin://grav-plugin/build/js/vendor.js' %}
```

```html title="Вывод"
<link rel="icon" href="/user/themes/mytheme/images/favicon.png" type="image/png">
<link rel="modulepreload" href="/user/plugins/grav-plugin/build/js/vendor.js">
```

### switch

В большинстве языков программирования `switch` — обычный способ сделать набор `if else` чище и читаемее. К тому же он может быть немного быстрее. Мы просто предоставляем простой способ создавать такие конструкции, которых не хватало в базовом Twig.

```twig title="Twig" linenums="1"
{% switch type %}
  {% case 'foo' %}
     {{ my_data.foo }}
  {% case 'bar' %}
     {{ my_data.bar }}
  {% default %}
     {{ my_data.default }}
{% endswitch %}
```

```text title="Вывод"
(выводит значение my_data.foo, my_data.bar или my_data.default в зависимости от type)
```

### deferred

С традиционными блоками после рендеринга блока с ним нельзя ничего сделать. Возьмём пример `{% block scripts %}` с записями для подключения JavaScript. Если у вас есть дочерний шаблон Twig, расширяющий базовый шаблон, где определён этот блок, можно расширить блок и добавить свои JavaScript-записи. Однако фрагменты шаблона Twig, включённые с этой страницы, не могут добраться до блока или взаимодействовать с ним.

Атрибут deferred у блока, работающий благодаря [Deferred Extension](https://github.com/rybakit/twig-deferred-extension), означает, что блок можно определить в любом шаблоне Twig, но его рендеринг откладывается, поэтому он выводится после всего остального. Это значит, что ссылки JavaScript можно добавлять вызовом `{% do assets.addJs() %}` из любого места страницы, и благодаря отложенному рендерингу вывод будет содержать все ресурсы, известные Grav, независимо от того, когда вы их добавили.

```twig title="Twig"
{% block myblock deferred %}
    This will be rendered after everything else.
{% endblock %}
```

```text title="Вывод"
This will be rendered after everything else.
```

Также можно объединить содержимое родительского блока с отложенным блоком с помощью `{{ parent() }}`. Это особенно полезно темам при добавлении дополнительных CSS- или JavaScript-файлов.

```twig title="Twig"
{% block stylesheets %}
    <!-- Дополнительная CSS-библиотека -->
    {% do assets.addCss('theme://libraries/leaflet/dist/leaflet.css') %}
    {{ parent() }}
{% endblock %}
```

```html title="Вывод"
<link rel="stylesheet" href="/user/themes/mytheme/libraries/leaflet/dist/leaflet.css">
<!-- содержимое родительских stylesheets -->
```

### throw

В некоторых ситуациях нужно вручную выбросить исключение, поэтому у нас есть и такой тег.

```twig title="Twig"
{% throw 404 'Not Found' %}
```

```text title="Вывод"
(выбрасывает исключение HTTP 404 Not Found)
```

### try & catch

Также полезно иметь в шаблонах Twig более мощную обработку ошибок в стиле PHP, поэтому у нас есть новый тег `try/catch`.

```twig title="Twig"
{% try %}
   <li>{{ user.get('name') }}</li>
{% catch %}
   User Error: {{ e.message }}
{% endcatch %}
```

```text title="Вывод"
<li>John Doe</li>
(или при ошибке: User Error: User not found)
```

### render

Flex-объекты постепенно проникают во всё больше элементов Grav. Это самосознающие объекты со связанной структурой шаблонов Twig, поэтому они знают, как рендерить себя. Чтобы использовать их, мы реализовали новый тег `render`, принимающий необязательный макет, который определяет, с каким из макетов шаблонов рендерить объект.

```twig title="Twig"
{% render collection layout: 'list' %}
{% render object layout: 'default' with { variable: 'value' } %}
```

```html title="Вывод"
<!-- рендерит коллекцию шаблоном list.html.twig -->
<!-- рендерит объект шаблоном default.html.twig с variable -->
```

### cache

Иногда нужно кэшировать части страницы, рендеринг которых занимает много времени. Это можно сделать тегом `cache`.

```twig title="Twig"
{% cache 600 %}
  {{ some_complex_work() }}
{% endcache %}
```

```text title="Вывод"
(кэшированный вывод some_complex_work() на 600 секунд)
```

В примере `600` — необязательное время жизни в секундах. Если параметр не передан, используется время жизни кэша по умолчанию.
