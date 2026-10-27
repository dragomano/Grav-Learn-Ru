---
title: "Разработка Grav"
description: "Эта страница для тех, кто хочет работать над самим Grav или создавать для него что-то. В ней рассказано, как развернуть сайт для разработки, запустить набор тестов, создать заготовку плагина или темы и вернуть изменения обратно. Она относится к…"
tags:
  - "Разработка"
---

# Разработка Grav

Эта страница для тех, кто хочет работать над самим Grav или создавать для него что-то. В ней рассказано, как развернуть сайт для разработки, запустить набор тестов, создать заготовку плагина или темы и вернуть изменения обратно. Она относится к ядру, плагинам, темам, Admin Next и плагину API.

Если вы поддерживаете плагин или тему из Grav 1.x и хотите узнать, что изменилось, начните с [Руководства по обновлению для разработчиков](../../migration/developer-upgrade-guide/index.md) и раздела [Совместимость плагинов](../../plugins/plugin-compatibility/index.md).

## Части Grav 2.0 {: #the-parts-of-grav-2-0 }

Grav 2.0 распределён по нескольким репозиториям. Знание того, какой из них за что отвечает, экономит время, когда вы берётесь что-то исправить.

| Часть | Репозиторий | Что это |
|------|------------|------------|
| Ядро | [getgrav/grav](https://github.com/getgrav/grav) | Папка `system`, CLI `bin/grav` и базовая установка. |
| Плагин API | [getgrav/grav-plugin-api](https://github.com/getgrav/grav-plugin-api) | REST API по адресу `/api/v1`. Через него с Grav общаются Admin Next, сервер MCP и любые внешние инструменты. |
| Admin Next | [getgrav/grav-plugin-admin2](https://github.com/getgrav/grav-plugin-admin2) и [getgrav/grav-admin-next](https://github.com/getgrav/grav-admin-next) | Панель администрирования. Репозиторий плагина — это PHP-обёртка и собранное приложение. Репозиторий `grav-admin-next` — исходники на SvelteKit. |
| Сервер MCP | [getgrav/grav-mcp](https://github.com/getgrav/grav-mcp) | Приложение на Node.js, позволяющее ИИ-клиентам управлять сайтом Grav через API. См. [Сервер MCP](../mcp-server/index.md). |
| Плагины и темы | По одному репозиторию на каждый | Например, `grav-plugin-error` или `grav-theme-quark2`. |

Классический плагин админки не входит в Grav 2.0. Не создавайте новый интерфейс админки на его основе.

## Требования

- **PHP 8.3 или новее.** Ядро протестировано вплоть до PHP 8.5.
- **Composer** — для установки зависимостей и запуска тестов.
- **Git** и учётная запись GitHub, если вы планируете отправлять pull-запросы.
- **Node.js 18 или новее** — только если вы работаете с фронтендом Admin Next, сервером MCP или темой со сборочным шагом. Для работы с чистым PHP и Twig Node не нужен.

## Настройка сайта для разработки

Обычная установка копирует файлы в корень веб-сервера. Для разработки нужно, чтобы корень веб-сервера указывал на ваши клоны Git, тогда изменение в клоне сразу появляется на сайте. Grav поддерживает это с помощью символических ссылок.

### 1. Клонируйте репозитории в одну папку

Поместите все репозитории, с которыми планируете работать, в одну родительскую папку. Инструменты CLI ищут их там.

```bash
mkdir -p ~/Projects/grav
cd ~/Projects/grav
git clone https://github.com/getgrav/grav.git
git clone https://github.com/getgrav/grav-plugin-api.git
git clone https://github.com/getgrav/grav-plugin-admin2.git
git clone https://github.com/getgrav/grav-plugin-error.git
git clone https://github.com/getgrav/grav-plugin-problems.git
git clone https://github.com/getgrav/grav-plugin-shortcode-core.git
git clone https://github.com/trilbymedia/grav-plugin-github-markdown-alerts.git
git clone https://github.com/getgrav/grav-theme-quark2.git
```

> [!TIP] Совет
> Если вы собираетесь присылать изменения, сначала сделайте форк репозитория на GitHub и клонируйте свой форк. В примере выше для простоты используются адреса исходных репозиториев.

Перечисленные здесь плагины и тема — те, которые ожидает файл `.dependencies` ядра. В ту же папку можно добавить любой другой репозиторий плагина или темы.

### 2. Укажите Grav, где находятся клоны

Создайте файл `~/.grav/config` с одной строкой, указывающей на родительскую папку:

```yaml
github_repos: /Users/your_name/Projects/grav/
```

И `bin/grav new-project`, и `bin/grav install` читают этот файл, когда их просят создать символические ссылки.

### 3. Создайте сайт с символическими ссылками

Запустите `new-project` из клона ядра с флагом `-s`. Назначение — папка, которую обслуживает ваш веб-сервер.

```bash
cd ~/Projects/grav/grav
composer install
bin/grav new-project -s ~/www/grav2
```

Эта команда создаёт папки сайта, делает символические ссылки на `system`, `bin`, `index.php` и `composer.json` обратно в клон ядра, а затем связывает каждую зависимость из `.dependencies` в `user/plugins` и `user/themes`. Откройте новый сайт в браузере — и у вас рабочая установка Grav, где каждое изменение происходит в репозитории Git.

### 4. Добавьте другие плагины или темы как символические ссылки

Чтобы связать другой плагин или тему из той же родительской папки с существующим сайтом, используйте `install` с флагом `-p` или `-t`:

```bash
cd ~/www/grav2
bin/grav install -p api
bin/grav install -p admin2
bin/grav install -t my-theme
```

Имя — это часть после `grav-plugin-` или `grav-theme-`. Запуск `bin/grav install -s` без имени заново связывает всё из `.dependencies`.

> [!NOTE] Примечание
> Плагинам, подключённым символической ссылкой, всё равно нужны собственные зависимости Composer. Если плагин поставляется с `composer.json`, запустите `composer install` внутри его клона.

## Запуск тестов ядра

Ядро использует [Codeception](https://codeception.com) для модульных тестов и [PHPStan](https://phpstan.org) для статического анализа. Установите зависимости для разработки один раз, затем запускайте набор тестов из клона ядра:

```bash
cd ~/Projects/grav/grav
composer install
composer test
```

Чтобы запустить один тестовый класс или метод, вызовите Codeception напрямую:

```bash
vendor/bin/codecept run unit Grav/Common/InflectorTest
vendor/bin/codecept run unit Grav/Common/InflectorTest:testPluralize
```

У статического анализа есть собственные скрипты:

```bash
composer phpstan             # system/src на уровне 2
composer phpstan-framework   # Framework, Events и Installer на уровне 6
composer phpstan-plugins     # установленные плагины на уровне 1
```

Полный набор тестов должен проходить до слияния pull-запроса. Если вы добавляете поведение, добавьте тест в `tests/unit` рядом с существующими.

## Разработка плагинов

Большая часть разработки Grav — это разработка плагинов. Плагин — это папка в `user/plugins` с PHP-классом, который подписывается на [события](../../plugins/event-hooks/index.md). В 2.0 эта модель не изменилась.

### Создание заготовки нового плагина

Установите плагин [DevTools](https://github.com/getgrav/grav-plugin-devtools) и запустите мастер:

```bash
bin/gpm install devtools
bin/plugin devtools new-plugin
```

Мастер спрашивает, на какую версию Grav ориентироваться, и записывает соответствующий блок `compatibility`, зависимость от Grav и требование к PHP. Шаблон `api` даёт плагин, который уже регистрирует маршрут API и контроллер. Любой параметр можно передать в командной строке:

```bash
bin/plugin devtools new-plugin --grav=2.0 --template=api --name="My Plugin" --desc="What it does" --dev="Me" --email="me@example.com"
```

### Что нового для плагинов в 2.0 {: #what-is-new-for-plugins-in-2-0 }

- **Флаги совместимости.** Каждый плагин должен объявить в `blueprints.yaml`, какие версии Grav он поддерживает. Их читают GPM и инструмент миграции. См. [Совместимость плагинов](../../plugins/plugin-compatibility/index.md).
- **Эндпоинты API.** Плагин может зарегистрировать собственные REST-маршруты событием `onApiRegisterRoutes`. См. [Интеграция с API](../../plugins/plugin-api-integration/index.md).
- **Интерфейс Admin Next.** Пользовательские поля, страницы плагинов, панели и элементы меню — это веб-компоненты, опирающиеся на ваши эндпоинты API. Стандартным страницам конфигурации из схем ничего дополнительно не нужно. См. [Руководство разработчика API](../../api/developer-guide/index.md).
- **Переводы.** Admin Next читает языковые файлы плагинов иначе, чем классическая админка. См. [Переводы админки](../../plugins/admin-translations/index.md).
- **Расширения Markdown.** Плагины могут добавлять синтаксис Markdown через новый парсер. См. [Расширения Markdown](../../plugins/markdown-extensions/index.md).
- **Песочница Twig.** Twig в контенте страниц выполняется в песочнице. Если ваш плагин добавляет функции или фильтры Twig, предназначенные для контента, их нужно внести в белый список. См. [Руководство по обновлению для разработчиков](../../migration/developer-upgrade-guide/index.md).

Когда плагин готов для других людей, следуйте разделу [Публикация в GPM](../../plugins/gpm-submission/index.md). Эта страница описывает обязательные файлы, формат журнала изменений, процесс выпуска и то, как перенять заброшенный плагин.

## Разработка тем

Тема — это папка в `user/themes` с шаблонами Twig, ресурсами и файлом `blueprints.yaml`. Подробности приведены в главе [Темы](../../themes/index.md). DevTools может создать заготовку:

```bash
bin/plugin devtools new-theme
```

Шаблоны тем включают варианты с Tailwind CSS и daisyUI со сборочным шагом Node, а также простой вариант без всего этого. Шаблон `copy` берёт за основу уже установленную у вас тему.

У тем тоже есть блок `compatibility`. Добавьте его перед публикацией, а контрольный список выпуска темы см. в разделе [Публикация в GPM](../../themes/gpm-submission/index.md).

## Работа над Admin Next

Admin Next — это два репозитория. `grav-plugin-admin2` — то, что устанавливается на сайт. В нём PHP-обёртка и собранная копия приложения в `app/`. `grav-admin-next` — исходники на SvelteKit.

Чтобы изменить интерфейс, клонируйте оба репозитория рядом и собирайте из папки плагина:

```bash
cd ~/Projects/grav
git clone https://github.com/getgrav/grav-admin-next.git
cd grav-plugin-admin2
./bin/build.sh
```

Скрипт запускает `npm run build` в проекте SvelteKit и копирует результат в `app/`. Во время работы `npm run dev:plugin` внутри `grav-admin-next` пересобирает в плагин при каждом изменении.

Приложение собирается для фиксированного базового пути. Если маршрут админки у вас не по умолчанию, передайте путь при сборке:

```bash
ADMIN2_BASE=/my-site/admin2 ./bin/build.sh
```

У Admin Next нет прямого доступа к Grav. Всё, что он делает, — это вызовы API, поэтому если в админке нужны новые данные, сначала добавьте эндпоинт в плагин API или в свой плагин. Именно так плагины и расширяют админку: см. [Руководство разработчика API](../../api/developer-guide/index.md).

## Работа над плагином API

Плагин API — обычный плагин Grav. Подключите его к сайту для разработки символической ссылкой с помощью `bin/grav install -p api`, создайте ключ API в Admin Next и проверяйте через `curl`:

```bash
curl -s https://grav2.test/api/v1/system/info \
  -H "X-API-Key: $GRAV_API_KEY" | jq
```

Передавайте аутентификацию в заголовке `X-API-Key`, а не `Authorization`. Некоторые конфигурации PHP-FPM отбрасывают заголовок `Authorization` до того, как его увидит PHP.

В главе [API](../../api/index.md) описаны все эндпоинты. Если вы добавляете или меняете эндпоинт, обновите его страницу в папке `api-docs/` плагина, чтобы справочник оставался точным.

## Участие в разработке ядра

Grav использует модель [GitFlow](https://nvie.com/posts/a-successful-git-branching-model/). Текущая работа ведётся в ветке `develop`. Фичи попадают в ветки `feature/`, которые сливаются в `develop`. При выпуске `develop` сливается в `master` и получает тег с обычным номером версии, например `2.1.6`.

Чтобы прислать изменение:

1. Сделайте форк репозитория и создайте ветку от `develop`.
2. Внесите изменение и добавьте или обновите тесты.
3. Запустите `composer test` и `composer phpstan`. Оба должны пройти.
4. Откройте pull-запрос к `develop`. Опишите, что делает изменение и зачем.

Сообщайте об ошибках в репозитории, которому принадлежит код. Ошибки ядра — в `getgrav/grav`. Ошибки админки — в `grav-plugin-admin2`, ошибки API — в `grav-plugin-api`, а ошибки плагина или темы — в собственный репозиторий этого плагина или темы. Если не уверены, быстрее всего спросить в [Discord](https://chat.getgrav.org).

## Разработка с помощью ИИ

Команда Grav публикует навыки для Claude Code, в которых заложены шаблоны интеграции с API и Admin Next. Они полезны для переноса плагина 1.x или создания заготовки нового. См. [Разработка с помощью ИИ](../../migration/ai-assisted-development/index.md).

## Что читать дальше

В блоге Grav есть серия для разработчиков, где те же темы разобраны с более длинными примерами:

- [Grav 2.0 for Plugin Developers: Overview](https://getgrav.org/blog/grav-2-for-plugin-developers)
- [Compatibility Flags](https://getgrav.org/blog/grav-2-dev-compatibility-flags)
- [API Integration for Plugins](https://getgrav.org/blog/grav-2-dev-api-integration)
- [Admin Next: Custom Form Fields](https://getgrav.org/blog/grav-2-dev-admin-next-custom-fields)
- [Admin Next: Custom Pages](https://getgrav.org/blog/grav-2-dev-admin-next-custom-pages)
- [Admin Next: Panels and Menu Bar](https://getgrav.org/blog/grav-2-dev-admin-next-panels-menubar)
- [Markdown Extensions](https://getgrav.org/blog/grav-2-dev-markdown-extensions)

Бесплатные плагины [git-sync](https://github.com/trilbymedia/grav-plugin-git-sync) и [license-manager](https://github.com/getgrav/grav-plugin-license-manager) — законченные рабочие примеры интеграции с Admin Next, и их стоит прочитать целиком.
