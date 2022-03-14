---
description: Как изменить URL-адрес сайта на Grav CMS.
---

# Изменение URL-адреса сайта

Установив `custom_base_url` в system.yaml (или пользовательский базовый URL-адрес в системных настройках, в админке), мы можем держать Grav в папке, но запускать из корня домена.

## Сценарий 1, запуск в корневой папке домена

Grav установлен в `http://localhost:8080/grav-develop`, но вы хотите, чтобы он отвечал на `http://localhost: 8080`

В system.yaml установите

```yaml
custom_base_url: 'http://localhost:8080'
```

и установите путь сеанса к новому пути к сайту Grav:

```yaml
session:
  path: /
```

И в корне домена установите перенаправление, например с .htaccess:

```text
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/grav-develop/
RewriteRule ^(.*)$ /grav-develop/$1
```

где `grav-develop` — это подпапка, в которой находится Grav.

## Сценарий 2, запустить в другой подпапке

Grav установлен в `http://localhost:8080/grav-develop`, но вы хотите, чтобы он отвечал на `http://localhost:8080/xxxxx`.

В system.yaml установите

```yaml
custom_base_url: 'http://localhost:8080/xxxxx'
```

и установите путь сеанса к новому пути к сайту Grav:

```yaml
session:
  path: /xxxxx
```

И в новой корневой папке /xxxxx установите перенаправление, например с .htaccess:

```text
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/grav-develop/
RewriteRule ^(.*)$ /grav-develop/$1
```

где `grav-develop` — это дочерняя подпапка, в которой находится Grav.
