---
title: "Пример: контактная форма"
description: "Плагин Form для Grav — самый простой способ использовать формы на сайте. Посмотрим, как создать простую контактную форму."
tags:
  - "Формы"
  - "Twig"
---

# Пример: контактная форма

## Простая контактная форма

**Плагин Form для Grav** — самый простой способ использовать формы на сайте. Посмотрим, как создать простую контактную форму.

### Живой пример

В скелете Sora Article есть готовая страница формы, которую можно посмотреть, читая этот урок:

[Живая страница](http://demo.getgrav.org/soraarticle-skeleton/contact)

[Markdown-файл страницы](https://raw.githubusercontent.com/getgrav/grav-skeleton-soraarticle-blog/develop/pages/03.contact/form.md)

### Настройка страницы

Форму можно поместить на любую страницу сайта. Нужно лишь переименовать markdown-файл страницы в `form.md` или добавить заголовок [template](../../../content/headers/index.md#template) во frontmatter страницы, чтобы использовался шаблон `form`.

> [!CAUTION] Внимание
> Шаблон страницы или шаблон родительской страницы должен реализовывать тег `{% block content %}`, чтобы **плагин Form для Grav** мог вывести поля ввода на странице.

Поля формы и инструкции обработки определяются во frontmatter YAML страницы, поэтому просто откройте markdown-файл страницы в любимом редакторе и вставьте в него следующий код:

```yaml linenums="1"
---
title: Contact Form

form:
    name: contact

    fields:
        name:
          label: Name
          placeholder: Enter your name
          autocomplete: on
          type: text
          validate:
            required: true

        email:
          label: Email
          placeholder: Enter your email address
          type: email
          validate:
            required: true

        message:
          label: Message
          placeholder: Enter your message
          type: textarea
          validate:
            required: true

        g-recaptcha-response:
          label: Captcha
          type: captcha
          captcha_not_validated: 'Captcha not valid!'

    buttons:
        submit:
          type: submit
          value: Submit
        reset:
          type: reset
          value: Reset

    process:
        captcha: true
        save:
            fileprefix: contact-
            dateformat: Ymd-His-u
            extension: txt
            body: "{% include 'forms/data.save.txt.twig' %}"
        email:
            subject: "[Site Contact Form] {{ form.value.name|e }}"
            body: "{% include 'forms/data.html.twig' %}"
        message: Thank you for getting in touch!
---

# Contact form

Some sample page content
```

> [!NOTE] Примечание
> Убедитесь, что в плагине Email заданы адреса «Email from» и «Email to» с вашим адресом.

> [!CAUTION] Внимание
> В этом примере используется Google reCAPTCHA через [поле captcha](https://learn.getgrav.org/forms/forms/fields-available#captcha-field); чтобы всё работало, настройте `site_key` и `secret_key` в плагине form. Если не хотите использовать Google reCaptcha, просто удалите поле `g-recaptcha-response` и обработку `captcha: true`.

Теперь в папке контактной страницы создайте подпапку `thankyou/` и новый файл `formdata.md`. Вставьте в файл следующий код:

```yaml linenums="1"
---
title: Email sent
cache_enable: false
process:
    twig: true
---

## Email sent!
```

Вот и всё!

### Живая демонстрация

[Живая страница](http://demo.getgrav.org/soraarticle-skeleton/contact)

[Markdown-файл страницы](https://raw.githubusercontent.com/getgrav/grav-skeleton-soraarticle-blog/develop/pages/03.contact/form.md)

> [!NOTE] Примечание
> Формы на модульных страницах работают иначе. Подробнее об этом рекомендуется прочитать в разделе [использование форм на модульных страницах](https://learn.getgrav.org/forms/forms/how-to-forms-in-modular-pages).

Когда пользователь отправляет форму, плагин отправляет вам письмо (согласно настройке `form` плагина Email для Grav) и сохраняет введённые данные в папку data/.

> [!WARNING] Предупреждение
> Полные сведения о настройке электронной почты см. в [документации плагина Email](https://github.com/getgrav/grav-plugin-email/blob/develop/README.md).

Можно включить плагин **Grav Data Manager**, чтобы увидеть эти данные в **плагине Admin**.

> [!NOTE] Примечание
> В будущем мы хотим, чтобы Grav умел динамически генерировать формы из плагина Admin.
