---
description: Знакомимся с цепочкой запуска различных процессов в Grav CMS.
---

# Жизненный цикл Grav

Часто бывает полезно знать, как работает Grav, чтобы полностью понять, как лучше всего расширить Grav с помощью плагинов. Это жизненный цикл Grav:

!!! info "index.php"

    ``` markdown
    1. Проверка версии PHP, чтобы убедиться, что используется хотя бы версия **7.3.6**
    2. Инициализация загрузчика классов
    3. Получение экземпляров Grav
    ```

    !!! question "Grav.php"
        ``` markdown
        1. Экземпляра не существует, поэтому вызываем `load()`
        2. Добавляем `loader`
        3. Добавляем и инициализируем `debugger`
        4. Добавляем `grav` (устарело)
        5. Регистрируем сервисы по умолчанию
        6. Регистрируем Service Providers
            * Accounts Service Provider
                1. Добавляем `permissions` (1.7)
                2. Добавляем `accounts` (1.6)
                3. Добавляем `user_groups` (1.7)
                4. Добавляем `users` _(устарело)_
            * Assets Service Provider
                1. Добавляем `assets`
            * Backups Service Provider
                1. Добавляем `backups` (1.6)
            * Config Service Provider
                1. Добавляем `setup`
                2. Добавляем `blueprints`
                3. Добавляем `config`
                4. Добавляем `languages`
                5. Добавляем `language`
            * Error Service Provider
                1. Добавляем `error`
            * Filesystem Service Provider
                1. Добавляем `filesystem`
            * Flex Service Provider
                1. Добавляем `flex` (1.7)
            * Inflector Service Provider
                1. Добавляем `inflector`
            * Logger Service Provider
                1. Добавляем `log`
            * Output Service Provider
                1. Добавляем `output`
            * Pages Service Provider
                1. Добавляем `pages`
                2. Добавляем `page`
            * Request Service Provider
                1. Добавляем `request` (1.7)
            * Scheduler Service Provider
                1. Добавляем `scheduler` (1.6)
            * Session Service Provider
                1. Добавляем `session`
                2. Добавляем `messages`
            * Streams Service Provider
                1. Добавляем `locator`
                2. Добавляем `streams`
            * Task Service Provider
                1. Добавляем `task`
                2. Добавляем `action`
            * Simple Service Providers
                1. Добавляем `browser`
                2. Добавляем `cache`
                3. Добавляем `events`
                4. Добавляем `exif`
                5. Добавляем `plugins`
                6. Добавляем `taxonomy`
                7. Добавляем `themes`
                8. Добавляем `twig`
                9. Добавляем `uri`
        ```

    ``` markdown
    4. Вызов `Grav::process()`
    ```

    !!! question "Grav.php"
        ``` markdown
        1. Запускаем обработчик Initialize
            * Configuration
                1. Инициализируем `$grav['config']`
                2. Инициализируем `$grav['plugins']`
            * Logger
                1. Инициализируем `$grav['log']`
            * Errors
                1. Инициализируем `$grav['errors']`
                2. Регистрируем обработчики ошибок PHP
            * Debugger
                1. Инициализируем `$grav['debugger']`
            * Обрабатываем запросы отладчика
            * Начинаем буферизацию вывода
            * Localization
                1. Устанавливаем локаль и часовой пояс
            * Plugins
                1. Инициализируем `$grav['plugins']`
            * Pages
                1. Инициализируем `$grav['pages']`
            * Uri
                1. Инициализируем `$grav['uri']`
                2. Добавляем `$grav['base_url_absolute']`
                3. Добавляем `$grav['base_url_relative']`
                4. Добавляем `$grav['base_url']`
            * Handle redirect
                1. Делаем перенаправление, если `system.pages.redirect_trailing_slash` имеет значение `true` и в URL есть завершающий слэш
            * Accounts
                1. Инициализируем `$grav['accounts']`
            * Session
                1. Инициализируем `$grav['session']`, если `system.session.initialize` имеет значение `true`
        2. Запускаем обработчик Plugins
            * Запускаем событие **onPluginsInitialized**
        3. Запускаем обработчик Request
            * Инициализируем `$grav['request']`
            * Запускаем событие **onRequestHandlerInit** с [request, handler]
            * Если ответ устанавливается внутри события, останавливаем дальнейшую обработку и выводим ответ
        4. Запускаем обработчик Tasks
            * Если запрос имеет атрибут _controller.class_, либо _task_, либо _action_:
                * Запускаем контроллер
                * Если `NotFoundException`: продолжаем (проверяем задачу и действие)
                * Если код ответа 418: продолжаем (игнорируем задачу и действие)
                * Иначе: останавливаем дальнейшую обработку и выводим ответ
            * Если _task_:
                * Запускаем событие **onTask**
                * Запускаем событие **onTask.[TASK]**
            * Если _action_
                * Запускаем событие **onAction**
                * Запускаем событие **onAction.[ACTION]**
        5. Запускаем обработчик Backups
            * Инициализируем `$grav['backups']`
            * Запускаем событие **onBackupsInitialized**
        6. Запускаем обработчик Scheduler
            * Инициализируем `$grav['scheduler']`
            * Запускаем событие **onSchedulerInitialized**
        7. Запускаем обработчик Assets
            * Инициализируем `$grav['assets']`
            * Запускаем событие **onAssetsInitialized**
        8. Запускаем обработчик Twig
            * Инициализируем `$grav['twig']`
        ```

        !!! summary "Twig.php"
            ``` markdown
            * Задаем пути к шаблонам Twig на основе конфигурации
            * Обрабатываем языковые шаблоны, если они доступны
            * Запускаем событие **onTwigTemplatePaths**
            * Запускаем событие **onTwigLoader**
            * Загружаем конфигурацию Twig и цепочку загрузчика
            * Запускаем событие **onTwigInitialized**
            * Загружаем расширения Twig
            * Запускаем событие **onTwigExtensions**
            * Устанавливаем стандартные переменные Twig (config, uri, taxonomy, assets, browser и т. д.)
            ```

        ``` markdown
        9. Запускаем обработчик Pages
            * Инициализируем `$grav['pages']`
        ```

        !!! summary "Pages.php"
            ``` markdown
            * Вызываем `buildPages()`
            * (логика несколько отличается для Flex-страниц, но идея та же)
            * Проверяем наличие кэша
            * Если **кэш в порядке**, загружаем данные страниц из него
            * Если **кэш отсутствует**, вызываем `recurse()`
            * Запускаем событие **onBuildPagesInitialized** в `recurse()`
            * Если файл `.md` найден:
            ```

            !!! quote "Page.php"
                ``` markdown
                * Вызываем `init()` для загрузки сведений о файле
                * Устанавливаем `filePath`, `modified`, `id`
                * Вызываем `header()` для инициализации переменных заголовка
                * Вызываем `slug()` для установки слага URL
                * Вызываем `visible()` для установки видимого состояния
                * Устанавливаем статус `modularTwig()` в зависимости от того, начинается ли имя папки с `_` или нет
                ```

            ``` markdown
            * Запускаем событие **onPageProcessed**
            * Если в `папке` найдены потомки `recurse()`
            * Запускаем событие **onFolderProcessed**
            * Вызываем `buildRoutes()`
            * Инициализируем `taxonomy` для всех страниц
            * Строим таблицу маршрутов (`route`) для быстрого поиска
            ```

        ``` markdown
            * Запускаем событие **onPagesInitialized** с [pages]
            * Запускаем событие **onPageInitialized** с [page]
            * Если страница не маршрутизируется:
                * Запускаем событие **onPageNotFount** с [page]
            * Если _task_:
                * Запускаем событие **onPageTask** с [task, page]
                * Запускаем событие **onPageTask.[TASK]** с [task, page]
            * Если _action_:
                * Запускаем событие **onPageAction** с [action, page]
                * Запускаем событие **onPageAction.[ACTION]** с [action, page]
        10. Запускаем обработчик Debugger Assets
            * Только для панели отладки: Добавляем отладчик CSS/JS в активы
        11. Запускаем обработчик Render
            * Инициализируем `$grav['output']`
            * Если `output` является объектом класса `ResponseInterface`:
                * Останавливаем дальнейшую обработку и выводим ответ
            * Иначе:
                * Визуализируем страницу с помощью метода Twig `processSite()`
        ```

        !!! summary "Twig.php"
            ``` markdown
            * Запускаем событие **onTwigSiteVariables**
            * Получаем вывод страницы
            * Запускаем событие **onTwigPageVariables**, в том числе для каждой модульной подстраницы
            * Если страница не найдена или не маршрутизируется, сначала запускаем событие **onPageFallBackUrl** чтобы посмотреть, есть ли у нас запасной вариант для медиа-актива, а затем событие **onPageNotFound**, если нет
            * Устанавливаем все переменные Twig в объекте Twig
            * Устанавливаем имя шаблона на основе информации о файле/заголовке/расширении
            * Вызываем метод `render()`
            * Возвращаем результирующий HTML
            ```

        ``` markdown
                * Запускаем событие **onOutputGenerated**
                * Выводим данные в буфер вывода
                * Запускаем событие **onOutputRendered**
                * Строим объект _Response_
                * Останавливаем дальнейшую обработку и выдаем ответ
        12. Выводим заголовок и тело HTTP
        13. Подключаем отладчик рендеринга (если включен)
        14. Выключаем
            * Закрываем сессию
            * Закрываем соединение с клиентом
            * Запускаем событие **onShutdown**
        ```

Каждый раз, когда у страницы вызывается метод **content()**, происходит следующий жизненный цикл:

!!! example "Page.php"

    ``` markdown title="Если содержимое НЕ кэшировано"
    1. Запускаем событие **onPageContentRaw**
    2. Обрабатываем страницу в соответствии с настройками Markdown и Twig. Запускаем событие **onMarkdownInitialized**
    3. Запускаем событие **onPageContentProcessed**
    ```
    ``` markdown
    Запускаем событие **onPageContent**
    ```
