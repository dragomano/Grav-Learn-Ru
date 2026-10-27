---
title: "Конфигурация темы"
description: "В Grav можно легко получить доступ к конфигурации темы и информации из схемы из файлов Twig и PHP."
tags:
  - "Темы"
  - "Grav 1.7"
  - "Схемы"
  - "Twig"
  - "Конфигурация"
---

# Конфигурация темы

В Grav можно легко получить доступ к конфигурации темы и информации из схемы из файлов Twig и PHP.

## Доступ к информации из схемы темы

Информацию из файла `blueprints.yaml` активной темы можно получить из объекта `theme`. Используем следующий файл `blueprints.yaml` как пример:

```yaml linenums="1"
name: Antimatter
slug: antimatter
type: theme
version: 1.7.0
description: "Antimatter is the default theme included with **Grav**"
icon: empire
author:
  name: Team Grav
  email: devs@getgrav.org
  url: https://getgrav.org
homepage: https://github.com/getgrav/grav-theme-antimatter
demo: http://demo.getgrav.org/blog-skeleton
keywords: antimatter, theme, core, modern, fast, responsive, html5, css3
bugs: https://github.com/getgrav/grav-theme-antimatter/issues
license: MIT
```

Любой из этих пунктов можно получить через `grav.theme`, используя стандартный **синтаксис с точками**:

```twig linenums="1"
Author Email: {{ grav.theme.author.email }}
Theme License: {{ grav.theme.license }}
```

Те же значения можно получить из плагина Grav с помощью синтаксиса PHP:

```php linenums="1"
$theme_author_email = $this->grav['theme']['author']['email'];
$theme_license = $this->grav['theme']['license'];
```

## Доступ к конфигурации темы

У тем тоже есть файлы конфигурации. Файл конфигурации темы называется `<themename>.yaml`. Файл по умолчанию находится в корневой папке темы (`user/themes/<themename>`).

**Настоятельно** рекомендуется не менять YAML-файл темы по умолчанию, а переопределять настройки в папке `user/config/themes`. Так исходные настройки темы останутся нетронутыми, и вы сможете быстро получить доступ к изменениям и/или откатить их при необходимости.

Для примера рассмотрим тему Antimatter. По умолчанию в корневой папке темы есть файл `antimatter.yaml`. Его содержимое выглядит так:

```yaml linenums="1"
enabled: true
color: blue
```

Это простой файл, но он даёт представление о том, что можно делать с настройками темы. Давайте переопределим эти настройки и добавим новую.

Создайте файл `user/config/themes/antimatter.yaml` со следующим содержимым:

> *Замечу, что `enabled` здесь не повторяется. Если файлы конфигурации объединяются, а не просто заменяются, это следует явно указать.*

```yaml linenums="1"
color: red
info: Grav is awesome!
```

Затем в шаблонах темы эти переменные можно получить через объект `grav.theme.config`:

```
<h1 style="color:{{ grav.theme.config.color|e }}">{{ grav.theme.config.info|e }}</h1>
```

Это должно отрендериться так:

<h1 style="color:red">Grav is awesome!</h1>

В PHP текущую конфигурацию темы можно получить так:

```php linenums="1"
$color = $this->grav['theme']->config()['color'];
$info = $this->grav['theme']->config()['info'];
```

Просто! В конфигурации тем вас ограничивает только небо. Вы можете использовать её как угодно! :)

### Альтернативная запись

Следующие псевдонимы тоже работают:

```twig linenums="1"
Theme Color Option: {{ config.theme.color_option|e }}
   or
Theme Color Option: {{ theme_var(color_option)|e }}
   or
Theme Color Option: {{ grav.themes.antimatter.color_option|e }} [AVOID!]
```

**Хотя `grav.themes.<themename>` поддерживается, этого следует избегать, потому что это делает невозможным правильное наследование темы.**
