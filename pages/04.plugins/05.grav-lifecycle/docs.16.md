---
title: Жизненный цикл Grav
metadata:
    description: 'Знакомимся с цепочкой запуска различных процессов в Grav CMS.'
body_classes: lifecycle
taxonomy:
    category: docs
markdown:
    extra: true
---

Часто бывает полезно знать, как работает Grav, чтобы полностью понять, как лучше всего расширить Grav с помощью плагинов. Это жизненный цикл Grav:

<div class="level level-1" markdown=1>
### index.php
1. Проверка версии PHP, чтобы убедиться, что используется хотя бы версия **7.1.3**
1. Инициализация загрузчика классов
1. Получение экземпляров Grav
    <div class="level level-2" markdown=1>
    ### Grav.php
    1. Экземпляра не существует, поэтому вызываем `load()`
    1. Добавляем `loader`
    1. Добавляем и инициализируем `debugger`
    1. Добавляем `grav` (устарело)
    1. Регистрируем сервисы
        1. Accounts Service Provider
            1. Добавляем `accounts`
            1. Добавляем `users` (устарело)
        1. Assets Service Provider
            1. Добавляем `assets`
        1. Backups Service Provider
            1. Добавляем `backups`
        1. Config Service Provider
            1. Добавляем `setup`
            1. Добавляем `blueprints`
            1. Добавляем `config`
            1. Добавляем `languages`
            1. Добавляем `language`
        1. Error Service Provider
            1. Добавляем `error`
        1. Filesystem Service Provider
            1. Добавляем `filesystem`
        1. Inflector Service Provider
            1. Добавляем `inflector`
        1. Logger Service Provider
            1. Добавляем `log`
        1. Output Service Provider
            1. Добавляем `output`
        1. Pages Service Provider
            1. Добавляем `pages`
            1. Добавляем `page`
        1. Request Service Provider
            1. Добавляем `request`
        1. Scheduler Service Provider
            1. Добавляем `scheduler`
        1. Session Service Provider
            1. Добавляем `session`
            1. Добавляем `messages`
        1. Streams Service Provider
            1. Добавляем `locator`
            1. Добавляем `streams`
        1. Task Service Provider
            1. Добавляем `task`
            1. Добавляем `action`
        1. Other Service Providers
            1. Добавляем `browser`
            1. Добавляем `cache`
            1. Добавляем `events`
            1. Добавляем `exif`
            1. Добавляем `plugins`
            1. Добавляем `taxonomy`
            1. Добавляем `themes`
            1. Добавляем `twig`
            1. Добавляем `uri`
    </div>
1. вызываем `Grav::process()`
    <div class="level level-2" markdown="1">
    ### Grav.php
    1. Запускаем обработчик Configuration
        1. Инициализируем `$grav['config']`
        1. Загружаем плагины с их настройками `$grav['plugins']`
    1. Запускаем обработчик Logger
        1. Инициализируем `$grav['log']`
    1. Запускаем обработчик Errors
        1. Инициализируем `$grav['errors']`
        1. Регистрируем обработчики ошибок PHP
    1. Запускаем обработчик Debugger
        1. Инициализируем `$grav['debugger']`
    1. Запускаем обработчик Initialize
        1. Начинаем буферизацию вывода
        1. Инициализируем the timezone
        1. Инициализируем `$grav['session']`, если `system.session.initialize` имеет значение `true`
        1. Инициализируем `$grav['uri']`
            1. Добавляем `$grav['base_url_absolute']`
            1. Добавляем `$grav['base_url_relative']`
            1. Добавляем `$grav['base_url']`
        1. Делаем перенаправление, если `system.pages.redirect_trailing_slash` имеет значение `true` и в URL есть завершающий слэш
    1. Запускаем обработчик Plugins
        1. Инициализируем `$grav['accounts']`
        1. Инициализируем `$grav['plugins']`
        1. Запускаем событие **onPluginsInitialized**
    1. Запускаем обработчик Themes
        1. Инициализируем `$grav['themes']`
        1. Запускаем событие **onThemeInitialized**
    1. Запускаем обработчик Request
        1. Инициализируем `$grav['request']`
        1. Запускаем событие **onRequestHandlerInit** с [request, handler]
        1. Если ответ устанавливается внутри события, останавливаем дальнейшую обработку и выводим ответ
    1. Запускаем обработчик Tasks
        1. Если запрос имеет атрибут _controller.class_, либо _task_, либо _action_:
            1. Запускаем контроллер
            1. Если `NotFoundException`: продолжаем (проверяем задачу и действие)
            1. Если код ответа 418: продолжаем (игнорируем задачу и действие)
            1. Иначе: останавливаем дальнейшую обработку и выводим ответ
        1. Если _task_:
            1. Запускаем событие **onTask** event
            1. Запускаем событие **onTask.[TASK]**
        1. Если _action_:
            1. Запускаем событие **onAction**
            1. Запускаем событие **onAction.[ACTION]**
    1. Запускаем обработчик Backups
        1. Инициализируем `$grav['backups']`
        1. Запускаем событие **onBackupsInitialized**
    1. Запускаем обработчик Scheduler
        1. Инициализируем `$grav['scheduler']`
        1. Запускаем событие **onSchedulerInitialized**
    1. Запускаем обработчик Assets
        1. Инициализируем `$grav['assets']`
        1. Запускаем событие **onAssetsInitialized**
    1. Запускаем обработчик Twig
        1. Инициализируем `$grav['twig']`
            <div class="level level-3" markdown="1">
            ### Twig.php
            1. Задаем пути к шаблонам Twig на основе конфигурации
            1. Обрабатываем языковые шаблоны, если они доступны
            1. Запускаем событие **onTwigTemplatePaths**
            1. Запускаем событие **onTwigLoader**
            1. Загружаем конфигурацию Twig и цепочку загрузчика
            1. Запускаем событие **onTwigInitialized**
            1. Загружаем расширения Twig
            1. Запускаем событие **onTwigExtensions**
            1. Устанавливаем стандартные переменные Twig (config, uri, taxonomy, assets, browser и т. д.)
            </div>
    1. Запускаем обработчик Pages
        1. Инициализируем `$grav['pages']`
            <div class="level level-3" markdown="1">
            ### Pages.php
            1. Вызываем `buildPages()`
            1. Проверяем наличие кэша
            1. Если **кэш в порядке**, загружаем данные страниц из него
            1. Если **кэш отсутствует**, вызываем `recurse()`
            1. Запускаем событие **onBuildPagesInitialized** в `recurse()`
            1. Если файл `.md` найден:
                <div class="level level-4" markdown="1">
                ### Page.php
                1. Вызываем `init()` для загрузки сведений о файле
                1. Устанавливаем `filePath`, `modified`, `id`
                1. Вызываем `header()` для инициализации переменных заголовка
                1. Вызываем `slug()` для установки слага URL
                1. Вызываем `visible()` для установки видимого состояния
                1. Устанавливаем статус `modularTwig()` в зависимости от того, начинается ли имя папки с `_` или нет
                </div>
            1. Запускаем событие **onPageProcessed**
            1. Если в `папке` найдены потомки `recurse()`
            1. Запускаем событие **onFolderProcessed**
            1. Вызываем `buildRoutes()`
            1. Инициализируем `taxonomy` для всех страниц
            1. Строим таблицу маршрутов (`route`) для быстрого поиска
            </div>
        1. Запускаем событие **onPagesInitialized** с [pages]
        1. Запускаем событие **onPageInitialized** с [page]
        1. Если страница не маршрутизируется:
            1. Запускаем событие **onPageNotFount** с [page]
        1. Если _task_:
            1. Запускаем событие **onPageTask** с [task, page]
            1. Запускаем событие **onPageTask.[TASK]** с [task, page]
        1. Если _action_:
            1. Запускаем событие **onPageAction** с [action, page]
            1. Запускаем событие **onPageAction.[ACTION]** с [action, page]
    1. Запускаем обработчик Debugger Assets
        1. Добавляем отладчик CSS/JS в активы
    1. Запускаем обработчик Render
        1. Инициализируем `$grav['output']`
        1. Если `output` является объектом класса `ResponseInterface`:
            1. Останавливаем дальнейшую обработку и выводим ответ
        1. Иначе:
            1. Визуализируем страницу с помощью метода Twig `processSite()`
                <div class="level level-3" markdown="1">
                ### Twig.php
                1. Запускаем событие **onTwigSiteVariables**
                1. Получаем вывод страницы
                1. Запускаем событие **onTwigPageVariables**, в том числе для каждой модульной подстраницы
                1. Если страница не найдена или не маршрутизируется, сначала запускаем событие **onPageFallBackUrl** чтобы посмотреть, есть ли у нас запасной вариант для медиа-актива, а затем событие **onPageNotFound**, если нет
                1. Устанавливаем все переменные Twig в объекте Twig
                1. Устанавливаем имя шаблона на основе информации о файле/заголовке/расширении
                1. Вызываем метод `render()`
                1. Возвращаем результирующий HTML
                </div>
            1. Запускаем событие **onOutputGenerated**
            1. Выводим данные в буфер вывода
            1. Запускаем событие **onOutputRendered**
            1. Строим объект _Response_
            1. Останавливаем дальнейшую обработку и выдаем ответ
    1. Выводим заголовок и тело HTTP
    1. Подключаем отладчик рендеринга (если включен)
    1. Выключаем
        1. Закрываем сессию
        1. Закрываем соединение с клиентом
        1. Запускаем событие **onShutdown**
    </div>
</div>
Каждый раз, когда у страницы вызывается метод `content()`, происходит следующий жизненный цикл:

<div class="level level-1" markdown="1">
### Page.php
1. Если содержимое **НЕ** кэшировано:
    1. Запускаем событие **onPageContentRaw**
    1. Обрабатываем страницу в соответствии с настройками Markdown и Twig. Запускаем событие **onMarkdownInitialized**
    1. Запускаем событие **onPageContentProcessed**
1. Запускаем событие **onPageContent**
</div>
