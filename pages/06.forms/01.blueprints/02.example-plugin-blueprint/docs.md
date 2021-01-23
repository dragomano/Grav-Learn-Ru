---
title: 'Пример: чертеж плагина'
metadata:
    description: 'Чертежи плагинов в Grav CMS содержат информацию об авторе и источнике, а также зависимости и поля формы, используемые при настройке.'
taxonomy:
    category: docs
---

Чертеж плагина дает Grav представление о том, что такое плагин, его источник, информация о поддержке и авторе, зависимости и поля формы, используемые для администрирования плагина в Grav Admin.

В качестве примера вот чертеж для плагина:

[prism classes="language-yaml line-numbers"]
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
[/prism]

Существуют различные свойства, которые вы можете использовать, чтобы придать своему ресурсу идентичность. Некоторые из них **обязательны**, другие - _опциональны_.

[div class="table"]
| Свойство         | Описание                                                                                                                                                                                                                                                                                                                                                                                                      |
| :-----           | :-----                                                                                                                                                                                                                                                                                                                                                                                                           |
| __name*__        | Это название ресурса. Избегайте добавления плагинов или тем, в этом нет необходимости.                                                                                                                                                                                                                                                                                                                    |
| __slug*__        | Это уникальный идентификатор ресурса, он также используется для определения имени папки, в которой хранится ресурс, например `user/plugins/__slug__` |
| __type*__        | Это тип ресурса, он должен быть либо `plugin`, либо `theme` |
| __version*__     | Версия ресурса. Это значение всегда должно меняться постепенно с каждым выпуском. Вы также должны следовать стандарту [semver](https://semver.org/).                                                                                                                                                                                                                                                   |
| __description*__ | Описание вашего ресурса. Пожалуйста, не превышайте **200** символов. Описание должно быть кратким и по существу. При необходимости вы можете использовать синтаксис Markdown. Также неплохо заключить описание в цитату marks.                                                                                                                                                                     |
| __icon*__        | Значок - это то, что будет использоваться на [getgrav.org](https://getgrav.org). На этом этапе мы используем библиотеку значков [FontAwesome](https://fortawesome.github.io/Font-Awesome/icons/), поэтому, если вы разрабатываете новый плагин или тему, ваша работа должна быть выбранный вами значок ещё не используется. В противном случае нам придется поменять его за вас.                                                                |
| _screenshot_     | _(не обязательно)_ Снимок экрана используется только _темами_ и полностью игнорируется _плагинами_. Для _тем_ это будет имя файла снимка экрана, который поставляется с темой (по умолчанию: `screenshot.jpg`). Если у вас есть изображение _screenshot.jpg_ в корне вашей темы, вы можете не использовать это свойство. Наш репозиторий автоматически подберет его.                                        |
| __author.name*__ | ФИО разработчика                                                                                                                                                                                                                                                                                                                                                                                          |
| _author.email_   | _(не обязательно)_ Электронная почта разработчика.                                                                                                                                                                                                                                                                                                                                                                                |
| _author.url_     | _(не обязательно)_ Домашняя страница разработчика.                                                                                                                                                                                                                                                                                                                                                                             |
| _homepage_       | _(не обязательно)_ Если у вас есть специальная домашняя страница для вашего ресурса, это место для него.                                                                                                                                                                                                                                                                                                                 |
| _docs_           | _(не обязательно)_ Если у вас есть документация для вашего ресурса, можно указать её здесь.                                                                                                                                                                                                                                                                                                                        |
| _demo_           | _(не обязательно)_ Если у вас есть демо-версия вашего ресурса, укажите её здесь.                                                                                                                                                                                                                                                                                                                                |
| _guide_          | _(не обязательно)_ Если у вас есть учебные пособия или руководства по использованию вашего ресурса, укажите их здесь.                                                                                                                                                                                                                                                                                                                             |
| _keywords_       | _(не обязательно)_ Хотя реального использования ключевых слов пока нет, можно перечислить их здесь, разделив запятыми.                                                                                                                                                                                                                                                                               |
| _bugs_           | _(не обязательно)_ URL-адрес, по которому можно сообщить об ошибках, обычно это ссылка на [GitHub issues](https://guides.github.com/features/issues/) ссылка.                                                                                                                                                                                                                                                                     |
| _license_        | _(не обязательно)_ Тип лицензии вашего ресурса (MIT, GPL и т. д.). Рекомендуется всегда предоставлять файл `LICENSE` вместе с вашим ресурсом.                                                                                                                                                                                                                                                                    |
| _dependencies_   | _(не обязательно)_ Список зависимостей, которые требуются плагину/теме. По умолчанию для их установки используется GPM, однако, если указан необязательный URL-адрес репозитория GIT, возможна также установка прямо из репозитория. Также, если вы используете массив, вы можете явно определить имя и версию, используя [версии пакетов в стиле Composer](https://getcomposer.org/doc/articles/versions.md) |
| _gpm_            | _(не обязательно)_ Следует ли получать обновления через GPM. Установите значение `false`, чтобы отключить обновления GPM для ресурсов, не относящихся к GPM. |
[/div]

Вот пример идентификационной части плагина [GitHub](https://github.com/getgrav/grav-plugin-github):

[prism classes="language-yaml line-numbers"]
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
[/prism]

Чертежи тем работают по тому же принципу.
