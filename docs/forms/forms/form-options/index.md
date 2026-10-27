---
title: "Справочник: параметры форм"
description: "Для форм нет обязательных параметров. Однако, как описано в обзоре «Формы на сайте», настоятельно рекомендуется хотя бы задать имя формы…"
tags:
  - "Формы"
  - "Фронтенд"
---

# Справочник: параметры форм

### Имя

Для форм нет обязательных параметров. Однако, как описано в обзоре [«Формы на сайте»](../index.md), настоятельно рекомендуется хотя бы задать имя формы:

```yaml linenums="1"
form:
    name: my-form
```

Оно **должно быть уникальным** для вашего сайта Grav. Имя формы служит уникальным идентификатором формы в системе. На форму можно ссылаться по этому имени с любой другой страницы.

### Метод

Этот параметр позволяет управлять тем, отправляется ли форма через `POST` или `GET`. По умолчанию — `POST`. Учтите также: если в форме есть поле `file`, к методу также добавляется `enctype="multipart/form-data"`:

```yaml linenums="1"
form:
    method: GET
```

### Действие

По умолчанию действием будет маршрут текущей страницы. Обычно это логично, потому что форма должна обрабатываться той же страницей, на которой находится. Иногда действие нужно переопределить, чтобы указать другое расширение файла (например, `.json`) или даже конкретный якорь страницы:

```yaml linenums="1"
form:
    action: '/contact-us#contact-form'
```

Можно даже обрабатывать форму на другой странице, если результаты нужно обрабатывать именно там. Это также можно использовать, чтобы изменить шаблон ответа по сравнению с шаблоном исходной формы:

```yaml linenums="1"
form:
    action: /contact-us/ajax-process
```

Где есть файл страницы `form-messages.html.twig`, возвращающий только данные сообщения. В качестве альтернативы можно использовать подход ниже…

### Шаблон

Обычно шаблон Twig страницы, отображающий форму, вполне способен обработать сообщения об успехе и ошибках или результаты встроенной валидации. Однако иногда полезно вернуть ответ формы с помощью другого шаблона Twig. Хороший пример — обработка формы через Ajax. Скорее всего, вам нужен только HTML сообщений об успехе/ошибке, возвращаемый шаблоном, чтобы JavaScript мог вставить его обратно на страницу:

```yaml linenums="1"
form:
    template: form-messages
```

### ID

Возможность задать CSS-`id` на уровне формы. Если он не указан, используется имя формы.

```yaml linenums="1"
form:
    id: my-form-id
```

### Классы

На форме также можно задать явные классы. Значений по умолчанию здесь нет.

```yaml linenums="1"
form:
    classes: 'form-style form-surround'
```

### Атрибуты

Добавление собственных атрибутов к элементу формы. В примере ниже замените `key` на ваш атрибут, а `value` — на его значение.

```yaml linenums="1"
form:
    attributes:
        key: value
```

### Встроенные ошибки

Установка Inline Errors в markdown-файле или определении формы включает показ ошибок рядом с полями — важный инструмент диагностики.

```yaml linenums="1"
form:
    inline_errors: true
```

### Клиентская валидация

Отключение клиентской валидации позволяет увидеть встроенные ошибки и подробную серверную валидацию, выходящую за пределы клиентской валидации HTML5. Клиентскую валидацию можно отключить через form.yaml или в определении формы.

```yaml linenums="1"
form:
    client_side_validation: false
```

### Проверки XSS {: #xss-checks }

По умолчанию Grav 1.7 и более поздние версии включают различные проверки XSS во всех формах. Настройки по умолчанию можно найти в разделе [Конфигурация безопасности](../../../basics/grav-configuration/index.md#security). Однако их можно переопределить для каждой формы или поля, например отключить проверки XSS во всей форме так:

```yaml linenums="1"
form:
    xss_check: false
```

> [!CAUTION] Внимание
> **ПРЕДУПРЕЖДЕНИЕ.** Не рекомендуется отключать все проверки XSS — лучше переопределять конкретные правила для отдельных полей. Все примеры здесь также работают внутри поля формы.

Отдельные правила можно включать или отключать, переопределяя основную конфигурацию. Правила, которые не переопределены, берутся из конфигурации безопасности:

```yaml linenums="1"
form:
    xss_check:
        enabled_rules:
            on_events: false
            invalid_protocols: false
            moz_binding: false
            html_inline_styles: false
            dangerous_tags: false
```

Ещё лучше — можно разрешить конкретные протоколы и теги:

```yaml linenums="1"
form:
    xss_check:
        safe_protocols:
            - javascript
        safe_tags:
            - iframe
```

### Поддержание сессии

Можно сделать так, чтобы формы не переставали отправляться при истечении сессии, включив параметр `keep_alive`. При этом перед истечением сессии к Grav выполняется AJAX-запрос, чтобы поддерживать её «свежей»:

```yaml linenums="1"
form:
    keep_alive: true
```

### Наборы полей

Можно создать теги `<fieldset></fieldset>` для полей формы с помощью обозначения `fieldset:` в форме.

```yaml linenums="1"
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
```

Приведённая выше форма выводится так:

```html linenums="1"
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
```

В примере выше поля появляются внутри набора полей `my-fieldset`. Вы также заметите, что теги `<legend></legend>` поддерживаются через параметр `legend:`.
