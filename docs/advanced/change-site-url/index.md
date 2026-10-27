---
title: "Смена URL сайта"
description: "Задав custom_base_url в system.yaml (или Custom Base URL в системных настройках админки), можно держать Grav в папке, но работать в корне домена."
tags:
  - "Конфигурация"
---

# Смена URL сайта

Задав `custom_base_url` в system.yaml (или Custom Base URL в системных настройках админки), можно держать Grav в папке, но работать в корне домена.

## Сценарий 1: работа в корневой папке домена

Grav установлен в `http://localhost:8080/grav-develop`, но вы хотите, чтобы он отвечал на `http://localhost:8080`

В system.yaml задайте

```yaml
custom_base_url: 'http://localhost:8080'
```

и установите путь сессии в путь нового сайта Grav,

```yaml linenums="1"
session:
  path: /
```

А в корне домена настройте перенаправление, например через .htaccess:

```text linenums="1"
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/grav-develop/
RewriteRule ^(.*)$ /grav-develop/$1
```

где `grav-develop` — подпапка, в которой находится Grav.

## Сценарий 2: работа в другой подпапке

Grav установлен в `http://localhost:8080/grav-develop`, но вы хотите, чтобы он отвечал на `http://localhost:8080/xxxxx`

В system.yaml задайте

```yaml
custom_base_url: 'http://localhost:8080/xxxxx'
```

и установите путь сессии в путь нового сайта Grav,

```yaml linenums="1"
session:
  path: /xxxxx
```

А в новой корневой папке /xxxxx настройте перенаправление, например через .htaccess:

```text linenums="1"
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/grav-develop/
RewriteRule ^(.*)$ /grav-develop/$1
```

где `grav-develop` — соседняя подпапка, в которой находится Grav.
