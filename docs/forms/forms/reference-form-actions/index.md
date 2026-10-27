---
title: "Справочник: действия форм"
description: "Некоторые примеры действий форм мы видели в примере простой формы выше. Разберём действия, которые можно использовать."
tags:
  - "Формы"
---

# Справочник: действия форм

## Действия форм

Некоторые примеры действий форм мы видели в примере простой формы выше. Разберём действия, которые можно использовать.

### Email

Отправляет письмо с указанными параметрами.

Пример:

```yaml linenums="1"
process:
    - email:
        from: "{{ config.plugins.email.from }}"
        to: "{{ config.plugins.email.to }}"
        subject: "Contact by {{ form.value.name|e }}"
        body: "{% include 'forms/data.html.twig' %}"
```

Отправляет письмо с адреса, указанного в конфигурации плагина Email, на тот же адрес (это контактная форма, мы отправляем письмо сами себе).
Если не нужны другие значения, `from` и `to` можно свободно опустить: по умолчанию они уже настроены на эти значения.

> [!CAUTION] Внимание
> Если в файле вроде site.yaml хранится «полный» адрес email (например, `My Name <myemail@example.com>`) и вы ссылаетесь на него через Twig, обязательно добавьте фильтр `|raw` (например, `{{ config.site.emails.sales|raw }}`. Иначе вы получите ошибку «An email must have a 'To', 'Cc', or 'Bcc' header».

Письмо имеет заданные тему и тело. В данном случае тело формируется файлом `forms/data.html.twig`, который находится в активном шаблоне (он есть в Antimatter и других основных темах, но не гарантируется, что он есть в каждой теме).

В Antimatter он такой:

```twig linenums="1"
{% for field in form.fields %}
    <div><strong>{{ field.label }}</strong>: {{ string(form.value(field.name)|e) }}</div>
{% endfor %}
```

Коротко говоря, он просто проходит по значениям и выводит их в теле письма.

> [!CAUTION] Внимание
> Дополнительные [важные параметры email для форм](https://github.com/getgrav/grav-plugin-email#emails-sent-with-forms), включая [многочастные тела сообщений](https://github.com/getgrav/grav-plugin-email#multi-part-mime-messages) (полезно для антиспам-оценки), `reply_to` и [вложения](https://github.com/getgrav/grav-plugin-email#sending-attachments), см. в документации плагина email.

##### Динамический атрибут email

Если нужно, например, задать поле `email.from` из ввода формы, можно получить его содержимое и использовать так:

`from: "{{ form.value.email|e }}"`

В этом случае мы берём поле «email» из формы и используем его для атрибута «from». Так владелец сайта получит письмо и сможет напрямую ответить на адрес, введённый в форме.

### Перенаправление {: #redirect }

Перенаправляет пользователя на другую страницу. Действие выполняется немедленно, поэтому при его использовании его, вероятно, нужно ставить в конец списка действий.

```yaml linenums="1"
process:
    - redirect: '/forms/landing-page'
```

Некоторые или все части поля `redirect` также можно задать из ввода формы или скрытого поля формы. Получить его содержимое и использовать можно так:

`redirect: "/path to/location/{{ form.value.hiddenfield }}"`

В этом случае мы берём поле «hiddenfield» из формы и используем его для последней части адреса перенаправления. Это может быть полезно при создании форм, которые, например, после завершения перенаправляют на скачивание.

### Сообщение

Задаёт сообщение, которое показывается после отправки формы.

```yaml linenums="1"
process:
    - message: Thank you for your feedback!
```

По умолчанию сообщение выводится в начале элемента `form`.

Однако представление можно дополнительно изменить через `display` или `redirect`.

#### Сообщение валидации

Действие message можно использовать для срабатывания при неудачной валидации. Например:

```yaml linenums="1"
username:
   type: text
   label: Username
   validate:
     required: true
     message: My custom message when validation fails!
```

Это позволит написать собственное сообщение, которое пользователи увидят при неудачной валидации.

### Отображение

После отправки формы её представление обновляется, встраивая подстраницу. Например, если форма находится в `/form`, подстраницу `/form/thankyou` можно встроить таким кодом:

```yaml linenums="1"
process:
    - display: thankyou
```

Если нужно встроить абсолютный путь страницы, например `site.com/thankyou`, добавьте перед ним `/`, например: `display: /thankyou`.

Плагин Form предоставляет шаблон `formdata`, подходящий для целевой страницы обработки: он выводит результат отправки формы. В примере выше можно создать страницу `pages/form/thankyou/formdata.md`.

Antimatter и совместимые темы предоставляют шаблон Twig `formdata.html.twig`, который выглядит так:

```twig linenums="1"
{% extends 'partials/base.html.twig' %}

{% block content %}

    {{ content|raw }}

    <div class="alert">{{ form.message|e }}</div>
    <p>Here is the summary of what you wrote to us:</p>

    {% include "forms/data.html.twig" %}

{% endblock %}
```

Если страница `thankyou/formdata.md` такая:

```yaml linenums="1"
---
title: Email sent
cache_enable: false
process:
    twig: true
---

## Email sent!
```

Выводом будет страница с заголовком «Email sent!», затем сообщение-подтверждение и данные формы, введённые на предыдущей странице.

В качестве целевой можно использовать страницу любого типа. Просто создайте свою и задайте соответствующий тип целевой страницы.

### Сохранение

Сохраняет данные формы в файл. Файл сохраняется в папку `user/data`, в подпапку с именем параметра `form.name`. Для успешного выполнения этого действия у формы **должно** быть имя, а подпапка должна быть создана с соответствующими правами до того, как в неё можно будет сохранять данные, поскольку новая папка не создаётся, если её нет. Например:

> [!CAUTION] Внимание
> `fileprefix` и `body` могут содержать разметку Twig.

```yaml linenums="1"
process:
    - save:
        fileprefix: feedback-
        dateformat: Ymd-His-u
        extension: txt
        body: "{% include 'forms/data.save.txt.twig' %}"
        operation: create
```

Сохранённые отправки могут содержать приватные данные. Убедитесь, что веб-сервер блокирует их URL, в том числе когда nginx отдаёт статические файлы перед Apache. Если хостинг открывает `.txt`, но блокирует `.dat`, вариант `extension: dat` — временное решение; сначала проверьте его на безобидных данных. Расширения из `security.uploads_dangerous_extensions` отклоняются, поэтому смена на `yaml` или `json` может привести к ошибке сохранения. О защите на стороне сервера см. [Доступ к пользовательской папке](../../../security/user-folder-exposure/index.md).

Если совсем не указывать `body`, по умолчанию используется `forms/data.save.txt.twig`. Этот шаблон предоставляет плагин Form, и тема может переопределить его своим `templates/forms/data.save.txt.twig`. Используйте его, а не `forms/data.txt.twig`, для сохраняемых файлов: `forms/data.txt.twig` экранирует HTML, поскольку его также включают в тела HTML-писем, и в сохранённом файле было бы `Tom &amp; Jerry` вместо `Tom & Jerry`.

> [!WARNING] Предупреждение
> `operation` может быть `create` (по умолчанию) — создавать новый файл при каждой отправке формы, или `add` — дописывать в один файл.

> [!WARNING] Предупреждение
> Обратите внимание, что операция `add` теперь требует статического имени файла: его нужно определить, см. пример ниже.

```yaml linenums="1"
process:
    - save:
        filename: feedback.txt
        body: "{% include 'forms/data.save.txt.twig' %}"
        operation: add
```

### Капча

Чтобы также проверять капчу на сервере, добавьте действие процесса captcha.

```yaml linenums="1"
process:
    - captcha:
        recaptcha_secret: ENTER_YOUR_CAPTCHA_SECRET_KEY
```

> [!CAUTION] Внимание
> `recaptcha_secret` необязателен; если вы указали его в конфигурации плагина Form, будет использовано значение оттуда.

### IP-адрес пользователя

Выводит IP-адрес пользователя в результатах. Разместите его выше процессов email / save в 'form.md', чтобы он использовался процессами вывода.

```yaml linenums="1"
process:
    - ip:
        label: User IP Address
```

### Отметка времени

Добавляет отметку времени отправки формы в результаты. Разместите её выше процессов email / save в 'form.md', чтобы она использовалась процессами вывода.

```yaml linenums="1"
process:
    - timestamp:
        label: Submission Timestamp
```

### Сброс формы после отправки

По умолчанию форма не очищается после отправки. Поэтому если нет действия `display` и пользователь возвращается на страницу формы, она всё ещё заполнена введёнными данными. Чтобы этого избежать, добавьте действие `reset`:

```yaml
process:
    - reset: true
```

### Запоминание значений полей

С помощью действия `remember` можно разрешить пользователям «вспоминать» _некоторые_ значения полей с момента последней отправки формы. Это особенно полезно для форм, которые отправляются многократно, например анонимной отправки, требующей сведений об отправителе.

> [!WARNING] Предупреждение
> [HTML5](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete) и [плагин Form для Grav](../fields-available/index.md#common-field-attributes) уже предоставляют это в ограниченном виде через браузер, так что воспользуйтесь этим. Однако автозаполнение может работать ненадёжно для некоторых пользователей и полей.

> [!WARNING] Предупреждение
> Действие `remember` **использует cookie** для хранения последнего значения, поэтому оно будет работать только на том же устройстве и в том же браузере, где браузеру разрешено принимать cookie вашего сайта.

Чтобы использовать это действие, просто перечислите имена полей, которые нужно запоминать.

Например, хороший вариант использования — онлайн-форма направления к врачу. Обычно её заполняют с одного компьютера, часть значений меняется редко и надоедает заполнять её повторно.

```yaml
process:
    - remember:
        - referrer-name
        - referrer-address
        - referrer-specialty
        - preferred-practitioner
```

## Собственные действия {: #custom-actions }

Можно «подключиться» к обработке формы и выполнять любые операции. Своя обработка, добавление данных в онлайн-веб-приложение, даже сохранение в базу данных.

Для этого в поле process формы добавьте имя своего действия обработки, например 'yourAction'.

```yaml
process:
    yourAction: true
```

Затем создайте простой плагин.

В его основном PHP-файле подпишитесь на событие `onFormProcessed`:

```php linenums="1"
namespace Grav\Plugin;
use Grav\Common\Plugin;
use RocketTheme\Toolbox\Event\Event;

class EmailPlugin extends Plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onFormProcessed' => ['onFormProcessed', 0]
        ];
    }
}
```

Затем предоставьте обработчик для действия saveToDatabase:

```php linenums="1"
public function onFormProcessed(Event $event)
{
    $form = $event['form'];
    $action = $event['action'];
    $params = $event['params'];

    switch ($action) {
        case 'yourAction':
            //делаем что нужно
    }
}
```

Если обработка может завершиться ошибкой и нужно остановить следующие действия формы, выполняемые по очереди, можно остановить обработку, вызвав `stopPropagation` у объекта $event:

```php linenums="1"
$event->stopPropagation();
return;
```

Пример кода с обработкой формы есть в репозиториях плагина Form и плагина Email.

#### Пример собственной обработки формы

Плагин Form умеет отправлять письма, сохранять файлы, задавать сообщения о статусе, и это очень удобно.
Иногда же нужен полный контроль. Именно так, например, работает плагин Login.

Он определяет frontmatter страницы `login.md`:

```yaml linenums="1"
title: Login
template: form

form:
    name: login

    fields:
        - name: username
          type: text
          placeholder: Username
          autofocus: true

        - name: password
          type: password
          placeholder: Password
```

Плагин Forms правильно генерирует и показывает форму. Обратите внимание: `process` не определён.

Кнопки формы `buttons` тоже отсутствуют, поскольку они добавляются вручную в `templates/login.html.twig`. Там же определяются `action` и `task` формы.

В этом случае `task` — `login.login`, а `action` установлен в URL страницы.

Когда пользователь нажимает «Login» в форме, Grav вызывает событие `onTask.login.login`.

`user/plugins/login/login.php` подключается к `onTask.login.login` в своём файле `classes/controller.php`, где и происходит аутентификация.
