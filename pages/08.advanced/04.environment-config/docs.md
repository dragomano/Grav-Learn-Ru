---
title: Конфигурация окружения
taxonomy:
    category: docs
---

Grav теперь имеет возможность расширять [мощные возможности конфигурации](../../basics/grav-configuration) для различных сред, чтобы поддерживать различные конфигурации для сценариев **разработки**, **постановки** и **производства**.

[version=17]
!! Среды до Grav 1.6 хранились в папке `user/`. Grav 1.7 перемещает среду в `user/env/`, чтобы упростить обслуживание среды. Настоятельно рекомендуется переместить все среды в это новое место на существующих сайтах.
[/version]

### Автоматическая настройка окружения

Это означает, что вы можете внести столько изменений конфигурации в среду, сколько необходимо. Хорошим примером этого является [Панель отладки](../debugging). По умолчанию новая панель отладки отключена в основном файле `system/config/system.yaml`, а также в файле переопределения пользователя:

[prism classes="language-bash command-line"]
user/config/system.yaml
[/prism]

Если вы хотите включить её, можно сделать это через файл `user/config/system.yaml`, однако лучшим решением может быть _enabled_ для вашей среды разработки при доступе через **localhost**, но _disabled_ на вашем **рабочем** сервере.

Этого можно легко добиться, переопределив этот параметр в файле:

[version=16]
[prism classes="language-bash command-line"]
user/localhost/config/system.yaml
[/prism]
[/version]
[version=17]
[prism classes="language-bash command-line"]
user/env/localhost/config/system.yaml
[/prism]
[/version]

где `localhost` - это имя хоста среды (это хост, который вы вводите в браузере, например, `http://localhost/your-site`), а ваш файл конфигурации содержит:

[prism classes="language-yaml line-numbers"]
debugger:
  enabled: true
[/prism]

Точно так же вы можете включить конвейерную обработку ресурсов CSS и JS (объединение + минификация) только для вашего рабочего сайта
([version=16]`user/www.mysite.com/config/system.yaml`[/version][version=17]`user/env/www.mysite.com/config/system.yaml`[/version]):

[prism classes="language-yaml line-numbers"]
assets:
  css_pipeline: true
  js_pipeline: true
[/prism]

Если к вашему производственному серверу можно получить доступ через `http://www.mysite.com`, вы также можете указать конфигурацию, специфичную для этого производственного сайта, с помощью файла, расположенного по адресу
[version=16]`user/www.mysite.com/config/system.yaml`[/version][version=17]`user/env/www.mysite.com/config/system.yaml`[/version].

Конечно, вы не ограничены изменениями в `system.yaml`, вы можете фактически переопределить **любую** настройку Grav в` site.yaml` или даже в любой [конфигурации плагина](../../plugins/plugin-basics)!

!! Если вы используете [Планировщик](/advanced/scheduler), помните об этом, используя среду `localhost` и, следовательно, её конфигурацию.

#### Переопределения плагинов

Переопределение YAML-файла конфигурации плагина — это просто тот же процесс, что и переопределение обычного файла. Если стандартный файл конфигурации находится в:

[prism classes="language-bash command-line"]
user/config/plugins/email.yaml
[/prism]

Затем вы можете переопределить это с помощью параметра, который отменяет только определенные параметры, которые вы хотите использовать для локального тестирования:

[version=16]
[prism classes="language-bash command-line"]
user/localhost/config/plugins/email.yaml
[/prism]
[/version]
[version=17]
[prism classes="language-bash command-line"]
user/env/localhost/config/plugins/email.yaml
[/prism]
[/version]

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

[version=16]
[prism classes="language-bash command-line"]
user/www.mysite.com/config/themes/antimatter/antimatter.yaml
[/prism]
[/version]
[version=17]
[prism classes="language-bash command-line"]
user/env/www.mysite.com/config/themes/antimatter/antimatter.yaml
[/prism]

### Конфигурация среды на основе сервера

Начиная с Grav 1.7, можно настроить среду, используя конфигурацию сервера. В этом сценарии использования вы устанавливаете переменные среды с сервера или из сценария, который запускается до Grav, чтобы выбрать среду, которая будет использоваться.

Самый простой способ установить среду - использовать `GRAV_ENVIRONMENT`. Значение `GRAV_ENVIRONMENT` должно быть действительным именем сервера с доменом или без него.

В следующем примере для локального хоста выбирается **среда разработки** (`development`):

[ui-tabs]
[ui-tab title="Apache 2"]
[prism classes="language-apacheconf line-numbers"]
<VirtualHost 127.0.0.1:80>
    ...

    SetEnv GRAV_ENVIRONMENT development
</VirtualHost>
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-fpm"]
[prism classes="language-nginx line-numbers"]
location / {
    ...

   fastcgi_param GRAV_ENVIRONMENT development;
}
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-cgi"]
[prism classes="language-nginx line-numbers"]
location / {
   ...

    env[GRAV_ENVIRONMENT] = development
}
[/prism]
[/ui-tab]
[ui-tab title="Docker"]
[prism classes="language-yaml line-numbers"]
web:
  environment:
    - GRAV_ENVIRONMENT=development
[/prism]
[/ui-tab]
[ui-tab title="PHP"]
[prism classes="language-php line-numbers"]
// Set environment in setup.php or make sure it runs before Grav.
define('GRAV_ENVIRONMENT', 'development');
[/prism]
[/ui-tab]
[/ui-tabs]

### Пользовательские пути к среде

Начиная с Grav 1.7, вы также можете изменить расположение локаций. Есть две возможности: либо вы настраиваете общее расположение для всех сред, либо определяете их одну за другой.

#### Пользовательское расположение для всех сред

Если по какой-то причине вас не устраивает расположение по умолчанию `user/env` для вашей среды, его можно изменить, используя переменную окружения `GRAV_ENVIRONMENTS_PATH`.

Значение `GRAV_ENVIRONMENTS_PATH` должно быть существующим путем в `GRAV_ROOT`. Не используйте завершающую косую черту.

В следующем примере все среды будут расположены в `user/sites/GRAV_ENVIRONMENT`, где `GRAV_ENVIRONMENT` либо автоматически определяется, либо устанавливается вручную в конфигурации сервера:

[ui-tabs]
[ui-tab title="Apache 2"]
[prism classes="language-apacheconf line-numbers"]
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENTS_PATH user://sites
</VirtualHost>
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-fpm"]
[prism classes="language-nginx line-numbers"]
location / {
    ...

fastcgi_param GRAV_ENVIRONMENTS_PATH user://sites;
}
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-cgi"]
[prism classes="language-nginx line-numbers"]
location / {
...

    env[GRAV_ENVIRONMENTS_PATH] = user://sites
}
[/prism]
[/ui-tab]
[ui-tab title="Docker"]
[prism classes="language-yaml line-numbers"]
web:
  environment:
    - GRAV_ENVIRONMENTS_PATH=user://sites
[/prism]
[/ui-tab]
[ui-tab title="PHP"]
[prism classes="language-php line-numbers"]
// Set environments path in setup.php or make sure that the following code runs before Grav.
define('GRAV_ENVIRONMENTS_PATH', 'user://sites');
[/prism]
[/ui-tab]
[/ui-tabs]

#### Пользовательское местоположение для текущей среды

Иногда может быть полезно иметь настраиваемое местоположение для вашей среды

Значение `GRAV_ENVIRONMENT_PATH` должно быть существующим путем в `GRAV_ROOT`. Не используйте завершающую косую черту.

В следующем примере только текущая среда будет расположена в `user/development`:

[ui-tabs]
[ui-tab title="Apache 2"]
[prism classes="language-apacheconf line-numbers"]
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENT_PATH user://development
</VirtualHost>
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-fpm"]
[prism classes="language-nginx line-numbers"]
location / {
    ...

fastcgi_param GRAV_ENVIRONMENT_PATH user://development;
}
[/prism]
[/ui-tab]
[ui-tab title="NGINX php-cgi"]
[prism classes="language-nginx line-numbers"]
location / {
...

    env[GRAV_ENVIRONMENT_PATH] = user://development
}
[/prism]
[/ui-tab]
[ui-tab title="Docker"]
[prism classes="language-yaml line-numbers"]
web:
  environment:
    - GRAV_ENVIRONMENT_PATH=user://development
[/prism]
[/ui-tab]
[ui-tab title="PHP"]
[prism classes="language-php line-numbers"]
// Set environment path in setup.php or make sure that the following code runs before Grav.
define('GRAV_ENVIRONMENT_PATH', 'user://development');
[/prism]
[/ui-tab]
[/ui-tabs]

Обратите внимание, что `GRAV_ENVIRONMENT_PATH` отделен от `GRAV_ENVIRONMENT`, поэтому вы также можете захотеть установить имя среды, если вы не хотите, чтобы оно автоматически совпадало с текущим именем домена.
### Дальнейшая настройка

Среды можно настраивать гораздо больше, чем описано на этой странице.

Для получения дополнительной информации перейдите на следующую страницу: [Настройка мультисайта](/advanced/multisite-setup).


