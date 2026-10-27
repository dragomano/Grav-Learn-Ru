---
title: "Пример: схема плагина"
description: "Схема плагина даёт Grav сведения о том, что представляет собой плагин, где его исходный код, поддержка и сведения об авторе, зависимости и поля формы, используемые для управления плагином в админке Grav."
tags:
  - "Формы"
  - "Схемы"
  - "Плагины"
---

# Пример: схема плагина

Схема плагина даёт Grav сведения о том, что представляет собой плагин, где его исходный код, поддержка и сведения об авторе, зависимости и поля формы, используемые для управления плагином в админке Grav.

Вот пример схемы плагина:

```yaml linenums="1"
name: Assets
slug: assets
type: plugin
version: 1.0.4
description: "This plugin provides a convenient way to add CSS and JS assets directly from your pages."
icon: list-alt
author:
  name: Team Grav
  email: devs@getgrav.org
  url: https://getgrav.org
homepage: https://github.com/getgrav/grav-plugin-assets
demo: https://learn.getgrav.org
keywords: assets, javascript, css, inline
bugs: https://github.com/getgrav/grav-plugin-assets/issues
license: MIT

dependencies:
  - { name: afterburner2 }
  - { name: github }
  - { name: email, version: '~2.0' }
```

Есть разные свойства, с помощью которых можно задать идентичность ресурса. Некоторые из них **обязательны**, другие — _необязательны_.

| свойство         | описание |
| :-----           | :----- |
| __name*__        | Имя ресурса. Не добавляйте к нему Plugin или Theme — в этом нет необходимости. |
| __slug*__        | Уникальный идентификатор ресурса; он также используется для определения имени папки, в которой хранится ресурс, например `user/plugins/__slug__`. |
| __type*__        | Тип ресурса: `plugin` или `theme`. |
| __version*__     | Версия ресурса. Это значение должно меняться при каждом выпуске, инкрементально. Также следует придерживаться стандарта [semver](http://semver.org/). |
| __description*__ | Описание ресурса. Не превышайте **200** символов. Описание должно быть коротким и по существу. При необходимости можно использовать синтаксис markdown. Также рекомендуется заключать описание в кавычки. |
| __icon*__        | Иконка, которая будет использоваться на [getgrav.org](https://getgrav.org). Сейчас используется библиотека иконок [Font Awesome](https://fontawesome.com/icons), поэтому при разработке нового плагина или темы вы должны убедиться, что выбранная иконка ещё не занята. Иначе нам придётся заменить её за вас. |
| _screenshot_     | _(необязательно)_ Скриншот учитывается только для _тем_ и полностью игнорируется для _плагинов_. Для _тем_ это имя файла скриншота, поставляемого с темой (по умолчанию: `screenshot.jpg`). Если в корне темы есть изображение _screenshot.jpg_, этот параметр можно не указывать: наш репозиторий подхватит его автоматически. |
| __author.name*__ | Полное имя разработчика. |
| _author.email_   | _(необязательно)_ Email разработчика. |
| _author.url_     | _(необязательно)_ Домашняя страница разработчика. |
| _homepage_       | _(необязательно)_ Если у ресурса есть отдельная домашняя страница, укажите её здесь. |
| _docs_           | _(необязательно)_ Если для ресурса написана документация, её можно указать здесь. |
| _demo_           | _(необязательно)_ Если у ресурса есть работающее демо, укажите ссылку здесь. |
| _guide_          | _(необязательно)_ Если у ресурса есть уроки или руководства, укажите ссылку здесь. |
| _keywords_       | _(необязательно)_ Хотя практической пользы от ключевых слов пока нет, их можно перечислить здесь через запятую. |
| _bugs_           | _(необязательно)_ URL, по которому можно сообщить об ошибках; обычно это ссылка на [GitHub issues](https://guides.github.com/features/issues/). |
| _license_        | _(необязательно)_ Тип лицензии ресурса (MIT, GPL и т. д.). Рекомендуется всегда прилагать к ресурсу файл `LICENSE`. |
| _dependencies_   | _(необязательно)_ Список зависимостей, необходимых плагину или теме. По умолчанию они устанавливаются через GPM, но если указан необязательный URL репозитория GIT, установка прямо из репозитория тоже будет доступна. Если используется массив, можно явно указать имя и версию с помощью [версий пакетов в стиле Composer](https://getcomposer.org/doc/articles/versions.md). |
| _gpm_            | _(необязательно)_ Получать ли обновления из GPM. Установите `false`, чтобы отключить обновления GPM для ресурсов, распространяемых не через GPM. |

> [!CAUTION] Внимание
> Учтите, что в зависимостях должно использоваться имя, заданное в свойстве `slug` файла `blueprints.yaml` плагина или темы.

Вот пример части с идентификацией из схемы [плагина GitHub](https://github.com/getgrav/grav-plugin-github):

```yaml linenums="1"
name: GitHub
slug: github
type: plugin
version: 1.0.1
description: "This plugin wraps the [GitHub v3 API](https://developer.github.com/v3/) and uses the [php-github-api](https://github.com/KnpLabs/php-github-api/) library to add a nice GitHub touch to your Grav pages."
icon: github
author:
  name: Team Grav
  email: devs@getgrav.org
  url: https://getgrav.org
homepage: https://github.com/getgrav/grav-plugin-github
keywords: github, plugin, api
bugs: https://github.com/getgrav/grav-plugin-github/issues
license: MIT
```

Схемы тем работают почти так же, как схемы плагинов.
