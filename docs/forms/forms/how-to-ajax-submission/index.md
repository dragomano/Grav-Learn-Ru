---
title: "Как настроить отправку через Ajax"
description: "Механизм обработки форм по умолчанию основан на стандартной отправке формы в стиле HTML, при которой содержимое формы отправляется на сервер через POST или GET (по умолчанию POST). После того как форма…"
tags:
  - "Формы"
  - "Фронтенд"
---

# Как настроить отправку через Ajax

## Отправка форм через XHR/Ajax {: #submitting-forms-via-xhr-ajax }

Механизм обработки форм по умолчанию основан на стандартной отправке формы в стиле HTML, при которой содержимое HTML-формы отправляется на сервер через `POST` или `GET` (по умолчанию `POST`). После того как форма [провалидирована](../fields-available/index.md) и [обработана](../reference-form-actions/index.md), результаты возвращаются форме (или на [страницу перенаправления](../reference-form-actions/index.md#redirect)), где показываются сообщения, и при необходимости форму можно отредактировать и отправить снова.

Это требует перезагрузки страницы, что иногда нежелательно. В таких случаях предпочтительна отправка формы через JavaScript с использованием Ajax или XHR. К счастью, возможностей форм Grav достаточно для этой задачи.

## Автоматический подход (плагин Form >= `v7.3.0`) {: #automatic-approach-form-plugin-v7-3-0 }

С выпуском плагина Form версии `7.3.0` появилась возможность отправлять формы через XHR, обрабатывая их на месте без полной перезагрузки страницы, с помощью быстрой настройки.

Чтобы включить это, добавьте `xhr_submit: true` в схему формы. Например:

```yaml
title: My XHR Form
form:
    name: my-xhr-form
    xhr_submit: true
```

Не требуется указывать `action:`, `template:` и даже `id:`. Плагин просто «заработает» даже с несколькими ajax-формами на одной странице. Используется новый шаблон `form-xhr.html.twig`, который предоставляет плагин, а также немного нативного JS-кода для выполнения запроса.

> [!CAUTION] Внимание
> Этот подход отправляет всю форму через XHR-запрос и заменяет весь HTML формы из ответа. Он задуман как простой; при необходимости можно создать собственные более продвинутые решения.

> [!NOTE] Примечание
> JavaScript-код для XHR-запроса находится в `form/layouts/xhr.html.twig`. При необходимости можно скопировать его в папку `templates` темы (сохранив структуру путей) и изменить.

> [!WARNING] Предупреждение
> **Если ваша тема ускоряет формы с помощью HTMX, отключите это для формы.** Когда на `<body>` или любом предке установлен `hx-boost="true"`, HTMX перехватывает отправку и выполняет собственный запрос, а затем подменяет содержимое страницы ответом. Это конкурирует с XHR плагина: форма отправляется дважды, и страница заметно перерисовывается — выглядит так, будто `xhr_submit` вообще не работает.
>
> Чаще всего это проявляется, когда в форме есть поле captcha: обработчики captcha вызывают `form.requestSubmit()` после решения токена, и HTMX воспринимает это как отправку формы с boost.
>
> Добавьте `hx-boost="false"` к форме или к любому элементу, который её оборачивает. HTMX наследует атрибут, поэтому одного обёрточного элемента достаточно для формы и всего внутри неё. Чтобы понять, какая библиотека отправляет запрос, перехватите `XMLHttpRequest.prototype.open` и выведите `new Error().stack` — запрос самого плагина идёт из `xhr-submitter.js` и возвращает только фрагмент формы, а boost-запрос идёт из `htmx.org` и возвращает целую страницу.

## Ручной подход (нужен для плагина Form < `v7.3.0`) {: #manual-approach-required-for-form-plugin-v7-3-0 }

### Создание формы

Можно создать любую стандартную форму; в этом примере мы сделаем форму как можно проще, чтобы сосредоточиться на части с Ajax. Сначала создадим форму на странице `forms/ajax-test/` и файл страницы формы `form.md`:

```yaml linenums="1"
---
title: Ajax Test-Form
form:
    name: ajax-test-form
    action: '/forms/ajax-test'
    template: form-messages
    refresh_prevention: true

    fields:
        name:
            label: Your Name
            type: text

    buttons:
        submit:
            type: submit
            value: Submit

    process:
        message: 'Thank you for your submission!'
---
```

Как видите, это очень простая форма, которая просто спрашивает имя и содержит кнопку отправки. Выделяется только часть `template: form-messages`. Как описано в разделе [«Формы на сайте»](../index.md), можно указать собственный шаблон Twig для вывода результата обработки формы. Это отличный способ обработать форму, а затем просто вернуть сообщения через Ajax и вставить их на страницу. Плагин forms уже предоставляет шаблон `form-messages.html.twig`, который именно это и делает.

> [!CAUTION] Внимание
> ПРИМЕЧАНИЕ. Мы используем жёстко заданный `action: '/forms/ajax-test'`, чтобы у ajax был постоянный URL, а не действие, указывающее на маршрут текущей страницы. Это решает проблему, когда Ajax-запрос неправильно обрабатывает перенаправления. Иначе могут возникать проблемы на главной странице. Это не обязательно должна быть текущая страница формы — нужен просто постоянный доступный маршрут.

`![](simple-form.png?classes=shadow)`

### Содержимое страницы

На этой же странице нужно разместить немного HTML и JavaScript:

```twig title="Vanilla JS" linenums="1"
<div id="form-result"></div>

<script>
document.addEventListener('DOMContentLoaded', function() {
    const form = document.querySelector('#ajax-test-form');
    form.addEventListener('submit', function(event) {
        event.preventDefault();

        const result = document.querySelector('#form-result');
        const action = form.getAttribute('action');
        const method = form.getAttribute('method');

        fetch(action, {
            method: method,
            body: new FormData(form)
        })
        .then(function(response) {
            if (response.ok) {
                return response.text();
            } else {
                return response.json();
            }
        })
        .then(function(output) {
            if (result) {
                result.innerHTML = output;
            }
        })
        .catch(function(error) {
            if (result) {
                result.innerHTML = 'Error: ' + error;
            }

            throw new Error(error);
        });
    });
});
</script>
```

```twig title="jQuery" linenums="1"
<div id="form-result"></div>

<script>
$(document).ready(function(){

    var form = $('#ajax-test-form');
    form.submit(function(e) {
        // предотвращаем отправку формы
        e.preventDefault();

        // отправляем форму через Ajax
        $.ajax({
            url: form.attr('action'),
            type: form.attr('method'),
            dataType: 'html',
            data: form.serialize(),
            success: function(result) {
                // вставляем результат в HTML
                $('#form-result').html(result);
            }
        });
    });
});
</script>
```

Сначала мы определяем div-плейсхолдер с ID `#form-result`, куда будем вставлять результаты формы.

Здесь для простоты используется синтаксис jQuery, но, разумеется, можно использовать любой JavaScript, выполняющий аналогичную функцию. Сначала мы останавливаем стандартную отправку формы и делаем Ajax-вызов на действие формы с сериализованными данными. Затем результат этого вызова помещается в созданный ранее div.

`![](submitted-form.png?classes=shadow)`
