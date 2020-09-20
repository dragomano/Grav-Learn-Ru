---
title: Конфигурация окружения
taxonomy:
    category: docs
---

Grav теперь имеет возможность расширять [мощные возможности конфигурации](../../basics/grav-configuration) для различных сред, чтобы поддерживать различные конфигурации для сценариев **разработки**, **постановки** и **производства**.

### Автоматическая настройка окружения

Это означает, что вы можете внести столько изменений конфигурации в среду, сколько необходимо. Хорошим примером этого является [Панель отладки](../debugging). По умолчанию новая панель отладки отключена в основном файле `system/config/system.yaml`, а также в файле переопределения пользователя:

[prism classes="language-bash command-line"]
user/config/system.yaml
[/prism]

Если вы хотите включить его, вы можете легко включить его в своем файле `user/config/system.yaml`, однако лучшим решением может быть _enabled_ для вашей среды разработки при доступе через **localhost**, но _disabled_ на вашем **рабочем** сервере.

Этого можно легко добиться, переопределив этот параметр в файле:
[prism classes="language-bash command-line"]
user/localhost/config/system.yaml
[/prism]

где `localhost` - это имя хоста среды (это хост, который вы вводите в браузере, например, `http://localhost/your-site`), а ваш файл конфигурации содержит:

[prism classes="language-yaml line-numbers"]
debugger:
  enabled: true
[/prism]

Точно так же вы можете включить конвейерную обработку ресурсов CSS и JS (объединение + минификация) только для вашего рабочего сайта (`user/www.mysite.com/config/system.yaml`):

[prism classes="language-yaml line-numbers"]
assets:
  css_pipeline: true
  js_pipeline: true
[/prism]

Если к вашему производственному серверу можно получить доступ через `http://www.mysite.com`, вы также можете указать конфигурацию, специфичную для этого производственного сайта, с помощью файла, расположенного по адресу `user/www.mysite.com/config/system.yaml`.

Конечно, вы не ограничены изменениями в `system.yaml`, вы можете фактически переопределить **любую** настройку Grav в` site.yaml` или даже в любой [конфигурации плагина](../../plugins/plugin-basics)!

#### Переопределения плагинов

Переопределение YAML-файла конфигурации плагина - это просто тот же процесс, что и переопределение обычного файла. Если стандартный файл конфигурации находится в:

[prism classes="language-bash command-line"]
user/config/plugins/email.yaml
[/prism]

Затем вы можете переопределить это с помощью параметра, который отменяет только определенные параметры, которые вы хотите использовать для локального тестирования:

[prism classes="language-bash command-line"]
user/localhost/config/plugins/email.yaml
[/prism]

С конфигурацией:

[prism classes="language-yaml line-numbers"]
mailer:
  engine: smtp
  smtp:
    server: smtp.mailtrap.io
    port: 2525
    encryption: none
    user: '9a320798e65135'
    password: 'a13e6e27bc7205'
[/prism]

#### Переопределения тем

Вы можете переопределить темы примерно таким же образом:

[prism classes="language-bash command-line"]
user/config/themes/antimatter/antimatter.yaml
[/prism]

Может быть переопределено для любой среды, скажем, на каком-то производственном сайте (`http://www.mysite.com`):

[prism classes="language-bash command-line"]
user/www.mysite.com/config/themes/antimatter/antimatter.yaml
[/prism]
