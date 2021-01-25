---
title: 'Ссылка: параметры формы'
metadata:
    description: 'Список доступных параметров форм в Grav CMS.'
page-toc:
  active: true
taxonomy:
    category: docs
---


### Имя

Для форм нет обязательных параметров. Однако, как указано в обзоре [Фронтенд-формы](/forms), настоятельно рекомендуется иметь хотя бы имя формы:

[prism classes="language-yaml line-numbers"]
form:
    name: my-form
[/prism]

Это имя **должно быть уникальным** для вашего сайта. Это происходит потому, что имя формы служит уникальным идентификатором для этой формы через систему. На форму можно ссылаться с помощью этого имени с любой другой страницы.

### Метод

Эта опция позволяет вам контролировать, должна ли форма быть отправлена ​​через POST или GET. По умолчанию - POST. Также обратите внимание, что если у вас есть поле `file` в вашей форме, к методу также будет добавлено `enctype="multipart/form-data"`:

[prism classes="language-yaml line-numbers"]
form:
    method: GET
[/prism]


### Action

Действие по умолчанию будет маршрутом текущей страницы. В большинстве случаев это имеет смысл, поскольку форма должна обрабатываться на той же странице, на которой находится форма. Бывают случаи, когда вы хотите переопределить действие, чтобы указать другое расширение файла (возможно, `.json`) или даже указать привязку конкретной страницы:

[prism classes="language-yaml line-numbers"]
form:
    action: '/contact-us#contact-form'
[/prism]

Вы даже можете обработать форму на другой странице, если именно на этой странице вы хотите обрабатывать результаты. Это также можно использовать в качестве метода для изменения шаблона ответа от шаблона, использованного в исходной форме:

[prism classes="language-yaml line-numbers"]
form:
    action: /contact-us/ajax-process
[/prism]

Если у вас есть файл подкачки с именем `form-messages.html.twig`, который возвращает только данные сообщения. В качестве альтернативы вы можете использовать подход ниже...

### Шаблон

Обычно шаблон страницы Twig, который отображает форму, отлично подходит для обработки любых сообщений об успехе/неудаче или ответов на встроенную проверку. Однако иногда бывает полезно отправить ответ формы, используя другой шаблон Twig. Хороший пример этого - когда вы хотите обработать форму через Ajax. Вероятно, вы хотите, чтобы шаблон возвращал только HTML для сообщений об успешном/неудачном завершении, чтобы их можно было вернуть на страницу с помощью JavaScript:

[prism classes="language-yaml line-numbers"]
form:
    template: form-messages
[/prism]

### ID

Возможность установить поле `id` на уровне формы. Если не указано, используется имя формы.

[prism classes="language-yaml line-numbers"]
form:
    id: my-form-id
[/prism]

### Классы

Вы также можете указать явные классы в форме. Здесь нет значений по умолчанию.

[prism classes="language-yaml line-numbers"]
form:
    classes: 'form-style form-surround'
[/prism]

### Встроенные ошибки

Установка встроенных ошибок в файле или определении Markdown формы позволяет отображать встроенные ошибки, что является важным инструментом устранения неполадок.

[prism classes="language-yaml line-numbers"]
form:
    inline_errors: true
[/prism]

### Проверка на стороне клиента

Отключение проверки на стороне клиента позволит вам увидеть встроенные ошибки и подробную проверку на стороне сервера, которые выходят за рамки проверки на стороне клиента HTML5. Вы можете отключить проверку на стороне клиента через form.yaml или в определении формы.

[prism classes="language-yaml line-numbers"]
form:
    client_side_validation: false
[/prism]

### Проверка XSS

По умолчанию Grav 1.7 и более поздние версии включают различные проверки XSS во всех формах. Настройки по умолчанию можно найти в разделе [Настройка > Безопасность](/basics/grav-configuration#security). Однако вы можете переопределить эти настройки для каждой формы или для каждого поля, например, вы можете отключить проверки XSS во всей форме:

[prism classes="language-yaml line-numbers"]
form:
    xss_check: false
[/prism]

!! **ПРЕДУПРЕЖДЕНИЕ** Не рекомендуется отключать все проверки XSS, но рекомендуется переопределить определенные правила для каждого поля. Все приведенные здесь примеры также будут работать внутри поля формы.

Вы можете включить или отключить отдельные правила, переопределив основную конфигурацию. Правила, которые не отменяются, вернутся к использованию конфигурации безопасности:

[prism classes="language-yaml line-numbers"]
form:
    xss_check:
        enabled_rules:
            on_events: false
            invalid_protocols: false
            moz_binding: false
            html_inline_styles: false
            dangerous_tags: false
[/prism]

Более того, вы также можете разрешить определенные протоколы и теги:

[prism classes="language-yaml line-numbers"]
form:
    xss_check:
        safe_protocols:
            - javascript
        safe_tags:
            - iframe
[/prism]

### Keep Alive

Вы можете гарантировать, что ваши формы не будут отправлены по истечении вашего сеанса, включив опцию `keep_alive` в форме. Включив это, запрос AJAX будет отправлен в Grav до истечения вашего сеанса, чтобы он оставался «свежим»:

[prism classes="language-yaml line-numbers"]
form:
    keep_alive: true
[/prism]

### Наборы полей

Вы можете установить теги `<fieldset></fieldset>` для полей в вашей форме, используя обозначение `fieldset:` в форме.

[prism classes="language-yaml line-numbers"]
form:
    name: Example Form
    fields:
        example:
            type: fieldset
            id: my-fieldset
            legend: 'Test Fieldset'
            fields:
                first_field: { type: text, label: 'First Field' }
                second_field: { type: text, label: 'Second Field' }
[/prism]

Приведенная выше форма выводит следующее:

[prism classes="language-html line-numbers"]
<form action="/grav/example/forms" class="" id="my-example-form" method="post" name="Example Form">
  <fieldset id="my-fieldset">
    <legend>Test Fieldset</legend>
    <div class="form-group">
      <div class="form-label-wrapper">
        <label class="form-label">First Field</label>
      </div>
      <div class="form-data" data-grav-default="null" data-grav-disabled="true" data-grav-field="text">
        <div class="form-input-wrapper">
          <input class="form-input" name="data[first_field]" type="text" value="">
        </div>
      </div>
    </div>
    <div class="form-group">
      <div class="form-label-wrapper">
        <label class="form-label">Second Field</label>
      </div>
      <div class="form-data" data-grav-default="null" data-grav-disabled="true" data-grav-field="text">
        <div class="form-input-wrapper">
          <input class="form-input" name="data[second_field]" type="text" value="">
        </div>
      </div>
    </div>
  </fieldset>
</form>
[/prism]

В приведенном выше примере поля появляются в наборе полей my-fieldset. Вы также заметите, что теги `<legend></legend>` поддерживаются параметром `legend:`.
