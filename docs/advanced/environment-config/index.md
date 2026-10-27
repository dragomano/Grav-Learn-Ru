---
title: "Конфигурация окружения"
description: "Grav умеет расширять мощные возможности конфигурации для разных окружений, чтобы поддерживать разные настройки для разработки, тестирования и эксплуатации."
tags:
  - "Grav 1.7"
  - "Конфигурация"
---

# Конфигурация окружения

Grav умеет расширять [мощные возможности конфигурации](../../basics/grav-configuration/index.md) для разных окружений, чтобы поддерживать разные настройки для сценариев **разработки**, **тестирования** и **эксплуатации**.

> [!CAUTION] Внимание
> До Grav 1.6 окружения хранились в папке `user/`. Grav 1.7 переносит окружения в `user/env/`, чтобы ими было проще управлять. Настоятельно рекомендуется перенести все окружения в это новое расположение на существующих сайтах.

### Автоматическая конфигурация окружения

Это означает, что вы можете задавать столько изменений конфигурации для каждого окружения, сколько нужно — мало или много. Хороший пример — [панель отладки](../debugging/index.md). По умолчанию новая панель отладки выключена в основном файле `system/config/system.yaml`, а также в файле переопределений пользователя:

```bash
user/config/system.yaml
```

Если вы хотели бы её включить, это легко сделать в файле `user/config/system.yaml`, но лучшее решение — _включить_ её для окружения разработки при доступе через **localhost** и _выключить_ на **продакшен**-сервере.

Это легко сделать, задав переопределение этой настройки в файле:

```bash
user/env/localhost/config/system.yaml
```

где `localhost` — имя хоста окружения (то, что вы вводите в браузере, например http://localhost/your-site), а ваш файл конфигурации содержит:

```yaml linenums="1"
debugger:
  enabled: true
```

Аналогично, возможно, вы захотите включить объединение и минификацию ресурсов CSS, Link, JS и JS Module (Asset Pipelining) только для продакшен-сайта
(`user/env/www.example.com/config/system.yaml`):

```yaml linenums="1"
assets:
  css_pipeline: true
  js_pipeline: true
  js_module_pipeline: true
```

Если ваш продакшен-сервер был доступен по `http://www.example.com`, вы также можете задать конфигурацию, специфичную для этого продакшен-сайта, в файле
`user/env/www.example.com/config/system.yaml`.

Разумеется, вы не ограничены изменениями в `system.yaml`: на самом деле можно задавать переопределения для **любой** настройки Grav в `site.yaml` или даже в любой [конфигурации плагина](../../plugins/plugin-basics/index.md)!

> [!CAUTION] Внимание
> Если вы используете [планировщик](../scheduler/index.md) Grav, учтите, что CLI работает в окружении `cli`, если не передать `--env=<host>` или не задать `GRAV_ENVIRONMENT`, поэтому переопределения для конкретного окружения по умолчанию не загружаются из cron.

#### Переопределения плагинов

Переопределить YAML-файл конфигурации плагина — тот же процесс, что и переопределить обычный файл. Если стандартный файл конфигурации находится в:

```bash
user/config/plugins/email.yaml
```

то его можно переопределить настройкой, которая меняет только нужные вам параметры для локального тестирования:

```bash
user/env/localhost/config/plugins/email.yaml
```

с конфигурацией:

```yaml linenums="1"
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

Темы можно переопределять почти так же:

```bash
user/config/themes/antimatter.yaml
```

можно переопределить для любого окружения, скажем для продакшен-сайта (`http://www.example.com`):

```bash
user/env/www.example.com/config/themes/antimatter.yaml
```

### Конфигурация окружения на основе сервера {: #server-based-environment-configuration }

Начиная с Grav 1.7 окружение можно задать через конфигурацию сервера. В этом сценарии вы задаёте переменные окружения на сервере или в скрипте, который выполняется до Grav, чтобы выбрать используемое окружение.

Простейший способ задать окружение — использовать `GRAV_ENVIRONMENT`. Значение `GRAV_ENVIRONMENT` должно быть допустимым именем сервера с доменом или без.

Следующий пример выбирает окружение **development** для localhost:

```apache title="Apache 2" linenums="1"
<VirtualHost 127.0.0.1:80>
    ...

    SetEnv GRAV_ENVIRONMENT development
</VirtualHost>
```

```nginx title="NGINX php-fpm" linenums="1"
location / {
    ...

   fastcgi_param GRAV_ENVIRONMENT development;
}
```

```nginx title="NGINX php-cgi" linenums="1"
location / {
   ...

    env[GRAV_ENVIRONMENT] = development
}
```

```yaml title="Docker" linenums="1"
web:
  environment:
    - GRAV_ENVIRONMENT=development
```

```php title="PHP" linenums="1"
// Задайте окружение в setup.php или убедитесь, что код выполняется до Grav.
define('GRAV_ENVIRONMENT', 'development');
```

### Пользовательские пути окружений

Начиная с Grav 1.7 можно также изменить расположение окружений. Есть два варианта: задать общее расположение для всех окружений или определить их по одному.

#### Общее расположение для всех окружений

Если по какой-то причине вас не устраивает расположение `user/env` по умолчанию для окружений, его можно изменить с помощью переменной окружения `GRAV_ENVIRONMENTS_PATH`.

Значение `GRAV_ENVIRONMENTS_PATH` должно быть существующим путём внутри `GRAV_ROOT`. Не используйте завершающую косую черту.

> [!WARNING] Предупреждение
> Правила сервера Grav защищают только папки `config`, `accounts` и `data` внутри `user/env/<name>/`. Если вы переносите окружения в другое место, например в `user/sites`, добавьте те же правила для этого пути в конфигурацию сервера, иначе эти папки можно будет скачать напрямую. См. [Доступность папки user](../../security/user-folder-exposure/index.md).

В следующем примере все окружения будут находиться в `user/sites/GRAV_ENVIRONMENT`, где `GRAV_ENVIRONMENT` определяется автоматически или задаётся вручную в конфигурации сервера:

```apache title="Apache 2" linenums="1"
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENTS_PATH user://sites
</VirtualHost>
```

```nginx title="NGINX php-fpm" linenums="1"
location / {
    ...

fastcgi_param GRAV_ENVIRONMENTS_PATH user://sites;
}
```

```nginx title="NGINX php-cgi" linenums="1"
location / {
...

    env[GRAV_ENVIRONMENTS_PATH] = user://sites
}
```

```yaml title="Docker" linenums="1"
web:
  environment:
    - GRAV_ENVIRONMENTS_PATH=user://sites
```

```php title="PHP" linenums="1"
// Задайте путь окружений в setup.php или убедитесь, что следующий код выполняется до Grav.
define('GRAV_ENVIRONMENTS_PATH', 'user://sites');
```

#### Пользовательское расположение текущего окружения

Иногда бывает полезно задать отдельное расположение для вашего окружения

Значение `GRAV_ENVIRONMENT_PATH` должно быть существующим путём внутри `GRAV_ROOT`. Не используйте завершающую косую черту.

В следующем примере только текущее окружение будет находиться в `user/development`:

```apache title="Apache 2" linenums="1"
<VirtualHost 127.0.0.1:80>
...

    SetEnv GRAV_ENVIRONMENT_PATH user://development
</VirtualHost>
```

```nginx title="NGINX php-fpm" linenums="1"
location / {
    ...

fastcgi_param GRAV_ENVIRONMENT_PATH user://development;
}
```

```nginx title="NGINX php-cgi" linenums="1"
location / {
...

    env[GRAV_ENVIRONMENT_PATH] = user://development
}
```

```yaml title="Docker" linenums="1"
web:
  environment:
    - GRAV_ENVIRONMENT_PATH=user://development
```

```php title="PHP" linenums="1"
// Задайте путь окружения в setup.php или убедитесь, что следующий код выполняется до Grav.
define('GRAV_ENVIRONMENT_PATH', 'user://development');
```

Учтите, что `GRAV_ENVIRONMENT_PATH` не связан с `GRAV_ENVIRONMENT`, поэтому, возможно, вы захотите также задать имя окружения, если не хотите, чтобы оно автоматически подставлялось по текущему доменному имени.

### Дальнейшая настройка

Окружения можно настраивать гораздо глубже, чем описано на этой странице.

Подробнее см. на следующей странице: [Настройка мультисайта](../multisite-setup/index.md).
