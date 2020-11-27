---
title: Теги Twig
page-toc:
  active: true
  start: 3

process:
    twig: false
taxonomy:
    category: docs
---

Grav также предоставляет множество пользовательских тегов Twig, которые расширяют уже очень способные возможности шаблона Twig с помощью некоторых новых тегов, которые мы нашли полезными.

### Markdown

Тэг `markdown` предоставляет новый мощный способ для встраивания Markdown в шаблон Twig. Можно использовать переменную и выводить ее с помощью фильтра `|markdown`, но синтаксис `{% markdown %}` делает создание блоков текста еще более простым.

[prism classes="language-twig line-numbers"]
{% markdown %}
This is **bold** and this _underlined_

1. This is a bullet list
2. This is another item in that same list
{% endmarkdown %}
[/prism]

### Scripts

Scripts - это действительно удобный тег, который делает ваш Twig более читабельным по сравнению с обычным подходом `{% do assets...%}`.  Это чисто альтернативный способ написания вещей.

#### File

[prism classes="language-twig line-numbers"]
{% script 'theme://js/something.js' in 'bottom' priority: 20 with { defer: true, async: true } %}
[/prism]

#### Inline

[prism classes="language-twig line-numbers"]
{% script in 'bottom' priority: 20 %}
    alert('Warning!');
{% endscript %}
[/prism]

### CSS Styles

#### File

[prism classes="language-twig line-numbers"]
{% style 'theme://css/foo.css' priority: 20 %}
[/prism]

#### Inline

[prism classes="language-twig line-numbers"]
{% style priority: 20 with { media: 'screen' } %}
    a { color: red; }
{% endstyle %}
[/prism]

### Switch

В большинстве языков программирования использование `switch` утверждений является обычным способом сделать кучу `if else` утверждений более чистыми и читабельными. Также они могут оказаться немного быстрее. Мы просто предоставляем простой способ их создания, так как они отсутствовали в базовой функциональности Twig.

[prism classes="language-twig line-numbers"]
{% switch type %}
  {% case 'foo' %}
     {{ my_data.foo }}
  {% case 'bar' %}
     {{ my_data.bar }}
  {% default %}
     {{ my_data.default }}
{% endswitch %}
[/prism]

### Отложенные блоки

Большой новой особенностью Grav 1.6 является мощность отложенных блоков. С традиционными блоками, после того, как блок отрисован, им нельзя манипулировать. Возьмем пример `{% block scripts %}`, который может содержать некоторые записи для JavaScript включает в себя. Если у вас есть дочерний шаблон Twig, и вы расширяете базовый шаблон, где определяется этот блок, вы можете расширить блок, и добавить свои собственные записи JavaScript. Тем не менее, шаблоны частичных веток, которые включены в эту страницу, не могут достигать блока или взаимодействовать с ним.

Атрибут отложенного на блоке, который питается от [расширения Deferred](https://github.com/rybakit/twig-deferred-extension), означает, что вы можете определить этот блок в любом шаблоне Twig, но его отрисовка отложена, так что он отрисовывает после всего остального. Это означает, что вы можете добавить ссылки на JavaScript с помощью вызова `{% do assets.addJs() %}` из любой точки вашей страницы, и поскольку рендеринг отложен, вывод будет содержать все активы, о которых Grav знает, независимо от того, когда вы их добавили.

[prism classes="language-twig line-numbers"]
{% block myblock deferred %}
    This will be rendered after everything else.
{% endblock %}
[/prism]

[version=16]
### Throw an Exception

Есть определенные ситуации, когда нужно вручную бросить исключение, поэтому у нас тоже есть тэг для этого.

[prism classes="language-twig line-numbers"]
{% throw 404 'Not Found' %}
[/prism]

### Исключения Try / Catch

Также полезно иметь более мощную обработку ошибок в PHP-стиле в ваших Twig-шаблонах, так что у нас есть новый тег `try/catch`.

[prism classes="language-twig line-numbers"]
{% try %}
   <li>{{ user.get('name') }}</li>
{% catch %}
   User Error: {{ e.message }}
{% endcatch %}
[/prism]

### Объект Render (только Flex)

Гибкие объекты медленно проникают во все новые и новые элементы Grav. Это самонаблюдающие объекты, которые имеют соответствующую структуру шаблонов Twig, так что они знают, как рендерить себя. Для того, чтобы использовать их, мы реализовали новый тег `render`, который принимает опциональный макет, который в свою очередь управляет тем, с каким из макетов шаблонов должен быть визуализирован объект.

[prism classes="language-twig line-numbers"]
{% render object layout: 'default' with { variable: 'value' } %}
[/prism]
[/version]
