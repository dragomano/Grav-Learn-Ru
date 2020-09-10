---
title: 'Ссылка: действия формы'
page-toc:
  active: true
taxonomy:
    category: docs
---

## Действия формы

Мы видели некоторые примеры действий формы в приведенном выше примере простой формы. Давайте подробно рассмотрим действия, которые вы можете использовать.

### Имейл

Отправляет электронное письмо с указанными опциями.

Пример:

[prism classes="language-yaml line-numbers"]
process:
    - email:
        from: "{{ config.plugins.email.from }}"
        to: "{{ config.plugins.email.to }}"
        subject: "Contact by {{ form.value.name|e }}"
        body: "{% include 'forms/data.html.twig' %}"
[/prism]

Отправляет электронное письмо с адреса электронной почты, указанного в конфигурации плагина электронной почты, отправляет его на тот же адрес электронной почты (это контактная форма, мы отправляем её себе).
Если вы не хотите использовать другие значения, вы можете свободно опускать from и to, поскольку они уже настроены по умолчанию для использования этих значений.
Письмо имеет заданную тему и тело.
В этом случае тело создается файлом `forms/data.html.twig`, который находится в активном шаблоне (он есть в Antimatter и других основных темах, но не гарантируется, что каждая тема включает его).

Antimatter устанавливает его в

[prism classes="language-twig line-numbers"]
{% for field in form.fields %}
    <div><strong>{{ field.label }}</strong>: {{ string(form.value(field.name)|e) }}</div>
{% endfor %}
[/prism]

Короче говоря, он просто зацикливает значения и печатает их в теле письма.

!! Дополнительные важные параметры электронной почты для формы см. в [документации к модулю электронной почты](https://github.com/getgrav/grav-plugin-email#emails-sent-with-forms) включая [составные тела сообщения](https://github.com/getgrav/grav-plugin-email#multi-part-mime-messages) (хорошо для оценки защиты от спама), `reply_to` и [вложения](https://github.com/getgrav/grav-plugin-email#sending-attachments).

##### Атрибут динамической электронной почты

Если вы хотите, например, установить поле `email.from` из ввода формы, вы можете получить его содержимое и использовать его следующим образом:

`from: "{{ form.value.email }}"`

В этом случае мы получаем поле «электронная почта» из формы и используем его в качестве атрибута «от». Таким образом, владелец сайта получит электронное письмо и сможет напрямую ответить на электронное письмо, указанное в форме.

### Перенаправление

Перенаправляет пользователя на другую страницу. Действие выполняется немедленно, поэтому, если вы его используете, вам, вероятно, нужно поместить его в конец списка действий.

[prism classes="language-yaml line-numbers"]
process:
    - redirect: '/forms/landing-page'
[/prism]

Вы также можете установить некоторые или все поля `redirect` из поля ввода формы или скрытого поля формы. Вы можете получить его содержимое и использовать его следующим образом:

`redirect: "/path to/location/{{ form.value.hiddenfield }}"`

В этом случае мы получаем поле "hiddenfield" из формы и используем его для последней части местоположения перенаправления. Это может быть полезно при создании форм, которые, например, перенаправляют на загрузку после завершения.

### Сообщение

Устанавливает сообщение, которое будет показано на следующей странице. Работает, если вы также установите действие `display`, которое перенаправляет пользователя на другую страницу. Обратите внимание: вы можете использовать Twig в сообщении, если хотите.

[prism classes="language-yaml line-numbers"]
process:
    - message: Thank you for your feedback!
    - display: thankyou
[/prism]

#### Сообщение о проверке

Вы можете использовать сообщение о непройденной валидации. Например:

[prism classes="language-yaml line-numbers"]
username:
   type: text
   label: Username
  validate:
     required: true
     message: My custom message when validation fails!
[/prism]

Это позволит вам написать собственное сообщение, которое пользователи увидят в случае неудачной проверки.

### Display

После отправки формы пользователь может быть перенаправлен на другую страницу. Эта страница будет подстраницей формы, поэтому, например, если ваша форма находится в `/form`, вы можете перенаправить пользователей на `/form/thankyou` с помощью следующего кода:

[prism classes="language-yaml line-numbers"]
process:
    - display: thankyou
[/prism]

Плагин Form предоставляет шаблон formdata, который подходит для целевой страницы процесса, поскольку он выводит результат отправки формы. В приведенном выше примере вы можете создать страницу `pages/form/thankyou/formdata.md`.

Если вы перенаправляете на подстраницу, `display: thankyou` работает отлично. Если вы перенаправляете на абсолютный путь к странице, например `site.com/thankyou`, добавьте перед ним `/`, например: `display: /thankyou`.

Антивещество и совместимые темы предоставляют шаблон Twig `formdata.html.twig`, который выглядит следующим образом:

[prism classes="language-twig line-numbers"]
{% extends 'partials/base.html.twig' %}

{% block content %}

    {{ content }}

    <div class="alert">{{ form.message }}</div>
    <p>Here is the summary of what you wrote to us:</p>

    {% include "forms/data.html.twig" %}

{% endblock %}
[/prism]

Если страница `thankyou/formdata.md`

[prism classes="language-yaml line-numbers"]
---
title: Электронное письмо отправлено!
cache_enable: false
process:
    twig: true
---

## Электронное письмо отправлено!
[/prism]

Результатом будет страница с надписью «Электронное письмо отправлено!». заголовок, за которым следует подтверждающее сообщение и данные формы, введенные на предыдущей странице.

Вы можете использовать любой тип страницы в качестве целевой страницы. Просто создайте свою и установите соответствующий тип целевой страницы.

### Сохранение

Сохраняет данные формы в файл. Файл сохраняется в папке `user/data` во вложенной папке, названной параметром `form.name`. Форма **должна** иметь имя для успешного выполнения этого действия, а подпапка должна быть создана с соответствующими разрешениями, прежде чем данные могут быть сохранены в ней, поскольку новый каталог не будет создан, если он не существует. Например:

!! `fileprefix` и` body` могут содержать разметку Twig.

[prism classes="language-yaml line-numbers"]
process:
    - save:
        fileprefix: feedback-
        dateformat: Ymd-His-u
        extension: txt
        body: "{% include 'forms/data.txt.twig' %}"
        operation: create
[/prism]

Тело берется из файла темы `templates/forms/data.html.twig`, предоставленного Antimatter и обновленными темами.

! «операция» может быть либо `create` (по умолчанию) для создания нового файла для отправки формы, либо `add` для добавления в отдельный файл.

! обратите внимание, что для операции `add` теперь требуется статическое имя файла: для определения см. пример ниже.

[prism classes="language-yaml line-numbers"]
process:
    - save:
        filename: feedback.txt
        body: "{% include 'forms/data.txt.twig' %}"
        operation: add
[/prism]

### Капча

Чтобы также проверить правильность ввода капчи на стороне сервера, добавьте действие процесса ввода капчи.

[prism classes="language-yaml line-numbers"]
    process:
        - captcha:
            recaptcha_secret: ENTER_YOUR_CAPTCHA_SECRET_KEY
[/prism]

!! `recaptcha_secret` не является обязательным и будет использовать значения конфигурации плагина формы, если вы предоставили их там.

### IP-адрес пользователя

Отображение IP-адреса пользователя на выходе. Поместите его над процессами электронной почты/сохранения в `form.md`, чтобы убедиться, что он используется процессами вывода.

[prism classes="language-yaml line-numbers"]
process:
    - ip:
        label: User IP Address
[/prism]

### Сброс формы после отправки

По умолчанию форма не очищается после отправки. Таким образом, если у вас нет действия `display` и пользователь отправляется обратно на страницу формы, она все равно заполнена введенными данными. Если вы хотите избежать этого, добавьте действие `reset`:

[prism classes="language-yaml"]
process:
    - reset: true
[/prism]

## Настраиваемые действия

Вы можете «зацепиться» за обработку формы и выполнять любые операции. Выполняйте индивидуальную обработку, добавляйте данные для онлайн-веб-приложения и даже сохраняйте в базе данных.

Для этого в поле процесса формы добавьте собственное имя обработки, например `yourAction`.

Затем создайте простой плагин.

В своем основном файле PHP зарегистрируйтесь для события `onFormProcessed`

[prism classes="language-php line-numbers"]
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
[/prism]

Затем укажите обработчик для действия `saveToDatabase`:

[prism classes="language-php line-numbers"]
    public function onFormProcessed(Event $event)
    {
        $form = $event['form'];
        $action = $event['action'];
        $params = $event['params'];

        switch ($action) {
            case 'yourAction':
                //do what you want
        }
    }
[/prism]

Если ваша обработка может пойти не так, и вы хотите остановить следующие действия формы, которые выполняются последовательно, вы можете остановить обработку, вызвав `stopPropagation` для объекта $event:

[prism classes="language-php line-numbers"]
$event->stopPropagation();
return;
[/prism]

Пример кода с обработкой формы доступен в подключаемом модуле формы и в репозиториях подключаемого модуля электронной почты.

#### Пример обработки нестандартной формы

Плагин Form предлагает возможность отправлять электронные письма, сохранять файлы, устанавливать статусные сообщения, и это действительно удобно.
Однако иногда вам нужен полный контроль. Это, например, то, что делает плагин входа в систему (**Login**).

Он определяет внешний вид страницы `login.md`:

[prism classes="language-yaml line-numbers"]
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
[/prism]

Плагин Forms правильно генерирует и отображает форму. Обратите внимание, что нет определения "процесса".

Также отсутствуют формы `buttons`, так как они вручную добавляются в `templates/login.html.twig`. Здесь также определяются формы «действие» и «задача».

В этом случае `задача` - это `login.login`, а `действие` устанавливается на адрес страницы.

Когда пользователь нажимает "Войти" в форме, Grav вызывает событие `onTask.login.login`.

`user/plugins/login/login.php` подключается к `onTask.login.login` к своему файлу `classes/controller.php`, и там происходит аутентификация.
