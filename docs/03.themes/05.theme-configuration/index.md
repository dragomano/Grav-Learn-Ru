# Конфигурация темы

> Как настроить используемую тему оформления в Grav CMS.

В Grav можно легко получить доступ к настройке тем и информации о чертежах из ваших файлов Twig и PHP.

## Доступ к информации о чертеже темы

Информацию из файла `blueprints.yaml` текущей активной темы можно получить из объекта `theme`. В качестве примера возьмем следующий файл `blueprints.yaml`:

```yaml
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

Вы можете получить доступ к любому из этих элементов через `theme`, используя стандартный **dot-синтаксис**:

```twig
Author Email: {{ theme.author.email }}
Theme License: {{ theme.license }}
```

Вы также можете получить эти же значения из плагина Grav с синтаксисом PHP:

```php
$theme_author_email = $this->grav['theme']['author']['email'];
$theme_license = $this->grav['theme']['license'];
```

## Доступ к конфигурации темы

У темы тоже есть файлы конфигурации. Файл конфигурации темы называется `<themename>.yaml`. Файл по умолчанию находится в корневой папке темы (`user/themes/<themename>`).

**Настоятельно** рекомендуется не изменять файл YAML по умолчанию для темы, а переопределить настройки в папке `user/config`. Это гарантирует, что исходные настройки темы останутся нетронутыми, что позволит вам быстро получить доступ к изменениям и/или вернуться к ним при необходимости.

Например, давайте рассмотрим тему антиматерии. По умолчанию в корневой папке темы есть файл под названием `antimatter.yaml`. Содержимое этого файла конфигурации выглядит следующим образом:

```yaml
enabled: true
color: blue
```

Это простой файл, но он дает вам представление о том, что вы можете делать с настройками конфигурации темы. Давайте переопределим эти настройки и добавим новые.

Итак, создайте файл в следующем месте: `user/config/themes/antimatter.yaml`. В этот файл поместите следующее содержимое:

> *Замечу, что `enabled` здесь не повторяется. Если файлы конфигурации объединены, а не просто заменены, это следует указать явно.*

```yaml
color: red
info: Grav is awesome!
```

Затем в шаблонах вашей темы вы можете получить доступ к этим переменным с помощью объекта `grav.theme.config`:

```
<h1 style="color:{{ grav.theme.config.color }}">{{ grav.theme.config.info }}</h1>
```

Это должно отображаться как:

<h1 style="color:red">Grav - это круто!</h1>

В PHP вы можете получить доступ к конфигурации текущей темы с помощью:

```php
$color = $this->grav['theme']->config()['color'];
$info = $this->grav['theme']->config()['info'];
```

Просто! Нет предела конфигурации ваших тем. Вы можете использовать их для чего угодно! :)

### Альтернативная нотация

Также работают следующие псевдонимы:

```twig
Вариант цвета темы: {{ config.theme.color_option }}
   или
Вариант цвета темы: {{ theme_var(color_option) }}
   или
Вариант цвета темы: {{ grav.themes.antimatter.color_option }} [AVOID!]
```

**Несмотря на то, что `grav.themes.<themename>` поддерживается, избегайте этого, потому что такой способ делает невозможным правильное наследование темы.**
