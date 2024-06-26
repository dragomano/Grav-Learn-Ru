---
description: Инструкция по созданию простой контактной формы в Grav CMS.
---

# Пример: контактная форма

## Простая контактная форма

**Плагин Grav Form** — это самый простой способ использовать формы на вашем сайте. Давайте посмотрим, как мы можем создать простую контактную форму.

### Живой пример

У каркаса Sora Article есть страница с формой, которую можно просмотреть во время чтения этого руководства:

[Живая страница](http://demo.getgrav.org/soraarticle-skeleton/contact)

[Файл markdown](https://raw.githubusercontent.com/getgrav/grav-skeleton-soraarticle-blog/develop/pages/03.contact/form.md)

### Настройка страницы

Вы можете разместить форму на любой странице вашего сайта. Всё, что вам нужно сделать, это переименовать файл разметки страницы в `form.md` или добавить заголовок [шаблон](/content/headers#shablon) во внешний вид страницы, чтобы он использовал шаблон `form`.

!!! info ""

    Шаблон вашей страницы или родительский шаблон страницы должен содержать тег `{% block content %}`, чтобы **плагин Grav Form** отображал ваши данные на странице.

Поля формы и инструкции по обработке определены во внешнем интерфейсе YAML страницы, поэтому просто откройте файл разметки страницы в своем любимом редакторе и поместите в него следующий код:

```yaml
---
title: Контактная форма

form:
  name: contact

  fields:
    name:
      label: Имя
      placeholder: Укажите свое имя
      autocomplete: on
      type: text
      validate:
        required: true

    email:
      label: Имейл
      placeholder: Укажите свой имейл
      type: email
      validate:
        required: true

    message:
      label: Сообщение
      placeholder: Введите сообщение
      type: textarea
      validate:
        required: true

    g-recaptcha-response:
      label: Капча
      type: captcha
      recaptcha_not_validated: 'Капча не действительна!'

  buttons:
    submit:
      type: submit
      value: Отправить
    reset:
      type: reset
      value: Сброс

  process:
    captcha: true
    save:
      fileprefix: contact-
      dateformat: Ymd-His-u
      extension: txt
      body: "{% include 'forms/data.txt.twig' %}"
    email:
      subject: '[Контактная форма сайта] {{ form.value.name|e }}'
      body: "{% include 'forms/data.html.twig' %}"
    message: Спасибо, что связались с нами!
---
# Контактная форма

Пример содержимого страницы
```

!!! tip ""

    Убедитесь, что вы настроили адреса электронной почты в плагине Email.

!!! warning ""

    В этом примере используется Google reCAPTCHA через [поле Captcha](../fields-available/#pole-captcha), и вы должны настроить ваши `site_key` и `secret_key` в плагине Form, чтобы это работало. Если вы не хотите использовать Google reCaptcha, просто удалите поле `g-recaptcha-response` и строчку `captcha: true` из секции `process`.

Теперь внутри папки `contacts` создайте подпапку с именем `thankyou/`, а в ней — новый файл с именем `formdata.md`. И вставьте следующий код:

```yaml
---
title: Письмо отправлено
cache_enable: false
process:
  twig: true
---
## Письмо отправлено!
```

Вот и всё!

!!! success ""

    Формы на модульных страницах работают по-разному. Чтобы узнать об этом больше, рекомендуем прочитать [использование форм на модульных страницах](/forms/forms/how-to-forms-in-modular-pages)

Когда пользователи отправят форму, плагин отправит вам электронное письмо (как установлено в настройках `form` плагина Email) и сохранит введенные данные в папке `data/`.

!!! info ""

    Для получения полной информации о настройке электронной почты, пожалуйста, прочтите [документацию по плагину Email](https://github.com/getgrav/grav-plugin-email/blob/develop/README.md)

Вы можете активировать плагин **Grav Data Manager**, чтобы увидеть эти данные в **плагине админки**.

!!! info ""

    В будущем мы хотим, чтобы Grav могла динамически генерировать формы из плагина админки.
