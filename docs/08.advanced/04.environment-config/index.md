# Конфигурация окружения

> Как настроить среду (окружение) при разработке, постановке и производстве Grav CMS.

Grav теперь имеет возможность расширять [мощные возможности конфигурации](/01.basics/05.grav-configuration/index) для различных сред, чтобы поддерживать различные конфигурации для сценариев **разработки**, **постановки** и **производства**.

> Среды до Grav 1.6 хранились в папке `user/`. Grav 1.7 перемещает среду в `user/env/`, чтобы упростить обслуживание среды. Настоятельно рекомендуется переместить все среды в это новое место на существующих сайтах.

### Автоматическая настройка окружения

Это означает, что вы можете внести столько изменений конфигурации в среду, сколько необходимо. Хорошим примером этого является [Панель отладки](/08.advanced/03.debugging/index). По умолчанию новая панель отладки отключена в основном файле `system/config/system.yaml`, а также в файле переопределения пользователя:

```bash
user/config/system.yaml
```

Если вы хотите включить её, можно сделать это через файл `user/config/system.yaml`, однако лучшим решением может быть _enabled_ для вашей среды разработки при доступе через **localhost**, но _disabled_ на вашем **рабочем** сервере.

Этого можно легко добиться, переопределив этот параметр в файле:

```bash
user/env/localhost/config/system.yaml
```

где `localhost` - это имя хоста среды (это хост, который вы вводите в браузере, например, `http://localhost/your-site`), а ваш файл конфигурации содержит:

```yaml
debugger:
  enabled: true
```

Точно так же вы можете включить конвейерную обработку ресурсов CSS, Link, JS и JS-модулей (объединение + минификация) только для вашего рабочего сайта
(`user/env/www.mysite.com/config/system.yaml`):

```yaml
assets:
  css_pipeline: true
  js_pipeline: true
  js_module_pipeline: true
```

Если к вашему производственному серверу можно получить доступ через `http://www.mysite.com`, вы также можете указать конфигурацию, специфичную для этого производственного сайта, с помощью файла, расположенного по адресу `user/env/www.mysite.com/config/system.yaml`.

Конечно, вы не ограничены изменениями в `system.yaml`, вы можете фактически переопределить **любую** настройку Grav в` site.yaml` или даже в любой [конфигурации плагина](/04.plugins/01.plugin-basics/index)!

> Если вы используете [Планировщик](/08.advanced/06.scheduler/index), помните об этом, используя среду `localhost` и, следовательно, её конфигурацию.

#### Переопределения плагинов

Переопределение YAML-файла конфигурации плагина — это просто тот же процесс, что и переопределение обычного файла. Если стандартный файл конфигурации находится в:

```bash
user/config/plugins/email.yaml
```

Затем вы можете переопределить это с помощью параметра, который отменяет только определенные параметры, которые вы хотите использовать для локального тестирования:

```bash
user/env/localhost/config/plugins/email.yaml
```

С конфигурацией:

```yaml
mailer:
  engine: smtp
  smtp:
    server: smtp.mailtrap.io
    port: 2525
    encryption: none
    user: '9a320798e65135'
    password: 'a13e6e27bc7205'
```

#### Переопределения тем

Вы можете переопределить темы примерно таким же образом:

```bash
user/config/themes/antimatter.yaml
```

Может быть переопределено для любой среды, скажем, на каком-то производственном сайте (`http://www.mysite.com`):

```bash
user/env/www.mysite.com/config/themes/antimatter.yaml
```

### Конфигурация среды на основе сервера

Начиная с Grav 1.7, можно настроить среду, используя конфигурацию сервера. В этом сценарии использования вы устанавливаете переменные среды с сервера или из сценария, который запускается до Grav, чтобы выбрать среду, которая будет использоваться.

Самый простой способ установить среду - использовать `GRAV_ENVIRONMENT`. Значение `GRAV_ENVIRONMENT` должно быть действительным именем сервера с доменом или без него.

В следующем примере для локального хоста выбирается **среда разработки** (`development`):

<!-- tabs:start -->
<!-- tab:Apache 2-->
```apacheconf
<VirtualHost 127.0.0.1:80>
    ...

    SetEnv GRAV_ENVIRONMENT development
</VirtualHost>
```
<!-- tabs:end -->
<!-- tab:NGINX php-fpm-->
```nginx
location / {
    ...

   fastcgi_param GRAV_ENVIRONMENT development;
}
```
<!-- tabs:end -->
<!-- tab:NGINX php-cgi-->
```nginx
location / {
   ...

    env[GRAV_ENVIRONMENT] = development
}
```
<!-- tabs:end -->
<!-- tab:Docker-->
```yaml
web:
  environment:
    - GRAV_ENVIRONMENT=development
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
// Set environment in setup.php or make sure it runs before Grav.
define('GRAV_ENVIRONMENT', 'development');
```
<!-- tabs:end -->

### Пользовательские пути к среде

Начиная с Grav 1.7, вы также можете изменить расположение локаций. Есть две возможности: либо вы настраиваете общее расположение для всех сред, либо определяете их одну за другой.

#### Пользовательское расположение для всех сред

Если по какой-то причине вас не устраивает расположение по умолчанию `user/env` для вашей среды, его можно изменить, используя переменную окружения `GRAV_ENVIRONMENTS_PATH`.

Значение `GRAV_ENVIRONMENTS_PATH` должно быть существующим путем в `GRAV_ROOT`. Не используйте завершающую косую черту.

В следующем примере все среды будут расположены в `user/sites/GRAV_ENVIRONMENT`, где `GRAV_ENVIRONMENT` либо автоматически определяется, либо устанавливается вручную в конфигурации сервера:

<!-- tabs:start -->
<!-- tab:Apache 2-->
```apacheconf
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENTS_PATH user://sites
</VirtualHost>
```
<!-- tabs:end -->
<!-- tab:NGINX php-fpm-->
```nginx
location / {
    ...

fastcgi_param GRAV_ENVIRONMENTS_PATH user://sites;
}
```
<!-- tabs:end -->
<!-- tab:NGINX php-cgi-->
```nginx
location / {
...

    env[GRAV_ENVIRONMENTS_PATH] = user://sites
}
```
<!-- tabs:end -->
<!-- tab:Docker-->
```yaml
web:
  environment:
    - GRAV_ENVIRONMENTS_PATH=user://sites
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
// Set environments path in setup.php or make sure that the following code runs before Grav.
define('GRAV_ENVIRONMENTS_PATH', 'user://sites');
```
<!-- tabs:end -->

#### Пользовательское местоположение для текущей среды

Иногда может быть полезно иметь настраиваемое местоположение для вашей среды

Значение `GRAV_ENVIRONMENT_PATH` должно быть существующим путем в `GRAV_ROOT`. Не используйте завершающую косую черту.

В следующем примере только текущая среда будет расположена в `user/development`:

<!-- tabs:start -->
<!-- tab:Apache 2-->
```apacheconf
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENT_PATH user://development
</VirtualHost>
```
<!-- tabs:end -->
<!-- tab:NGINX php-fpm-->
```nginx
location / {
    ...

fastcgi_param GRAV_ENVIRONMENT_PATH user://development;
}
```
<!-- tabs:end -->
<!-- tab:NGINX php-cgi-->
```nginx
location / {
...

    env[GRAV_ENVIRONMENT_PATH] = user://development
}
```
<!-- tabs:end -->
<!-- tab:Docker-->
```yaml
web:
  environment:
    - GRAV_ENVIRONMENT_PATH=user://development
```
<!-- tabs:end -->
<!-- tab:PHP-->
```php
// Set environment path in setup.php or make sure that the following code runs before Grav.
define('GRAV_ENVIRONMENT_PATH', 'user://development');
```
<!-- tabs:end -->

Обратите внимание, что `GRAV_ENVIRONMENT_PATH` отделен от `GRAV_ENVIRONMENT`, поэтому вы также можете захотеть установить имя среды, если вы не хотите, чтобы оно автоматически совпадало с текущим именем домена.
### Дальнейшая настройка

Среды можно настраивать гораздо больше, чем описано на этой странице.

Для получения дополнительной информации перейдите на следующую страницу: [Настройка мультисайта](/08.advanced/05.multisite-setup/index).


