# Жизненный цикл Grav

> Знакомимся с цепочкой запуска различных процессов в Grav CMS.

Часто бывает полезно знать, как работает Grav, чтобы полностью понять, как лучше всего расширить Grav с помощью плагинов. Это жизненный цикл Grav:

<div class="level level-1">
<h3 id="index-php">index.php</h3>
<ol>
<li>Проверка версии PHP, чтобы убедиться, что используется хотя бы версия <strong>7.3.6</strong></li>
<li>Инициализация загрузчика классов</li>
<li>Получение экземпляров Grav
<div class="level level-2">
<h3 id="grav-php">Grav.php</h3>
<ol>
<li>Экземпляра не существует, поэтому вызываем <code>load()</code></li>
<li>Добавляем <code>loader</code></li>
<li>Добавляем и инициализируем <code>debugger</code></li>
<li>Добавляем <code>grav</code> (устарело)</li>
<li>Регистрируем сервисы по умолчанию</li>
<li>Register Service Providers
<ol>
<li>Accounts Service Provider
<ol>
<li>Добавляем <code>permissions</code> (1.7)</li>
<li>Добавляем <code>accounts</code> (1.6)</li>
<li>Добавляем <code>user_groups</code> (1.7)</li>
<li>Добавляем <code>users</code> <em>(устарело)</em></li>
</ol></li>
<li>Assets Service Provider
<ol>
<li>Добавляем <code>assets</code></li>
</ol></li>
<li>Backups Service Provider
<ol>
<li>Добавляем <code>backups</code> (1.6)</li>
</ol></li>
<li>Config Service Provider
<ol>
<li>Добавляем <code>setup</code></li>
<li>Добавляем <code>blueprints</code></li>
<li>Добавляем <code>config</code></li>
<li>Добавляем <code>languages</code></li>
<li>Добавляем <code>language</code></li>
</ol></li>
<li>Error Service Provider
<ol>
<li>Добавляем <code>error</code></li>
</ol></li>
<li>Filesystem Service Provider
<ol>
<li>Добавляем <code>filesystem</code></li>
</ol></li>
<li>Flex Service Provider
<ol>
<li>Добавляем <code>flex</code> (1.7)</li>
</ol></li>
<li>Inflector Service Provider
<ol>
<li>Добавляем <code>inflector</code></li>
</ol></li>
<li>Logger Service Provider
<ol>
<li>Добавляем <code>log</code></li>
</ol></li>
<li>Output Service Provider
<ol>
<li>Добавляем <code>output</code></li>
</ol></li>
<li>Pages Service Provider
<ol>
<li>Добавляем <code>pages</code></li>
<li>Добавляем <code>page</code></li>
</ol></li>
<li>Request Service Provider
<ol>
<li>Добавляем <code>request</code> (1.7)</li>
</ol></li>
<li>Scheduler Service Provider
<ol>
<li>Добавляем <code>scheduler</code> (1.6)</li>
</ol></li>
<li>Session Service Provider
<ol>
<li>Добавляем <code>session</code></li>
<li>Добавляем <code>messages</code></li>
</ol></li>
<li>Streams Service Provider
<ol>
<li>Добавляем <code>locator</code></li>
<li>Добавляем <code>streams</code></li>
</ol></li>
<li>Task Service Provider
<ol>
<li>Добавляем <code>task</code></li>
<li>Добавляем <code>action</code></li>
</ol></li>
<li>Simple Service Providers
<ol>
<li>Добавляем <code>browser</code></li>
<li>Добавляем <code>cache</code></li>
<li>Добавляем <code>events</code></li>
<li>Добавляем <code>exif</code></li>
<li>Добавляем <code>plugins</code></li>
<li>Добавляем <code>taxonomy</code></li>
<li>Добавляем <code>themes</code></li>
<li>Добавляем <code>twig</code></li>
<li>Добавляем <code>uri</code></li>
</ol></li>
</ol></li>
</ol>
</div></li>
<li>вызываем <code>Grav::process()</code>
<div class="level level-2">
<h3 id="grav-php-1">Grav.php</h3>
<ol>
<li>Запускаем обработчик Initialize
<ol>
<li>Configuration
<ol>
<li>Инициализируем <code>$grav['config']</code></li>
<li>Инициализируем <code>$grav['plugins']</code></li>
</ol></li>
<li>Logger
<ol>
<li>Инициализируем <code>$grav['log']</code></li>
</ol></li>
<li>Errors
<ol>
<li>Инициализируем <code>$grav['errors']</code></li>
<li>Регистрируем обработчики ошибок PHP</li>
</ol></li>
<li>Debugger
<ol>
<li>Инициализируем <code>$grav['debugger']</code></li>
</ol></li>
<li>Обрабатываем запросы отладчика</li>
<li>Начинаем буферизацию вывода</li>
<li>Localization
<ol>
<li>Устанавливаем локаль и часовой пояс</li>
</ol></li>
<li>Plugins
<ol>
<li>Инициализируем <code>$grav['plugins']</code></li>
</ol></li>
<li>Pages
<ol>
<li>Инициализируем <code>$grav['pages']</code></li>
</ol></li>
<li>Uri
<ol>
<li>Инициализируем <code>$grav['uri']</code></li>
<li>Добавляем <code>$grav['base_url_absolute']</code></li>
<li>Добавляем <code>$grav['base_url_relative']</code></li>
<li>Добавляем <code>$grav['base_url']</code></li>
</ol></li>
<li>Handle redirect
<ol>
<li>Делаем перенаправление, если <code>system.pages.redirect_trailing_slash</code> имеет значение <code>true</code> и в URL есть завершающий слэш</li>
</ol></li>
<li>Accounts
<ol>
<li>Инициализируем <code>$grav['accounts']</code></li>
</ol></li>
<li>Session
<ol>
<li>Инициализируем <code>$grav['session']</code>, если <code>system.session.initialize</code> имеет значение <code>true</code></li>
</ol></li>
</ol></li>
<li>Запускаем обработчик Plugins
<ol>
<li>Запускаем событие <strong>onPluginsInitialized</strong></li>
</ol></li>
<li>Запускаем обработчик Themes
<ol>
<li>Инициализируем <code>$grav['themes']</code></li>
<li>Запускаем событие <strong>onThemeInitialized</strong></li>
</ol></li>
<li>Запускаем обработчик Request
<ol>
<li>Инициализируем <code>$grav['request']</code></li>
<li>Запускаем событие <strong>onRequestHandlerInit</strong> с [request, handler]</li>
<li>Если ответ устанавливается внутри события, останавливаем дальнейшую обработку и выводим ответ</li>
</ol></li>
<li>Запускаем обработчик Tasks
<ol>
<li>Если запрос имеет атрибут <em>controller.class</em>, либо <em>task</em>, либо <em>action</em>:
<ol>
<li>Запускаем контроллер</li>
<li>Если <code>NotFoundException</code>: продолжаем (проверяем задачу и действие)</li>
<li>Если код ответа 418: продолжаем (игнорируем задачу и действие)</li>
<li>Иначе: останавливаем дальнейшую обработку и выводим ответ</li>
</ol></li>
<li>Если <em>task</em>:
<ol>
<li>Запускаем событие <strong>onTask</strong></li>
<li>Запускаем событие <strong>onTask.[TASK]</strong></li>
</ol></li>
<li>Если <em>action</em>:
<ol>
<li>Запускаем событие <strong>onAction</strong></li>
<li>Запускаем событие <strong>onAction.[ACTION]</strong></li>
</ol></li>
</ol></li>
<li>Запускаем обработчик Backups
<ol>
<li>Инициализируем <code>$grav['backups']</code></li>
<li>Запускаем событие <strong>onBackupsInitialized</strong></li>
</ol></li>
<li>Запускаем обработчик Scheduler
<ol>
<li>Инициализируем <code>$grav['scheduler']</code></li>
<li>Запускаем событие <strong>onSchedulerInitialized</strong></li>
</ol></li>
<li>Запускаем обработчик Assets
<ol>
<li>Инициализируем <code>$grav['assets']</code></li>
<li>Запускаем событие <strong>onAssetsInitialized</strong></li>
</ol></li>
<li>Запускаем обработчик Twig
<ol>
<li>Инициализируем <code>$grav['twig']</code>
<div class="level level-3">
<h3 id="twig-php">Twig.php</h3>
<ol>
<li>Задаем пути к шаблонам Twig на основе конфигурации</li>
<li>Обрабатываем языковые шаблоны, если они доступны</li>
<li>Запускаем событие <strong>onTwigTemplatePaths</strong></li>
<li>Запускаем событие <strong>onTwigLoader</strong></li>
<li>Загружаем конфигурацию Twig и цепочку загрузчика</li>
<li>Запускаем событие <strong>onTwigInitialized</strong></li>
<li>Загружаем расширения Twig</li>
<li>Запускаем событие <strong>onTwigExtensions</strong></li>
<li>Устанавливаем стандартные переменные Twig (config, uri, taxonomy, assets, browser и т. д.)</li>
</ol>
</div></li>
</ol></li>
<li>Запускаем обработчик Pages
<ol>
<li>Инициализируем <code>$grav['pages']</code>
<div class="level level-3">
<h3 id="pages-php">Pages.php</h3>
<ol>
<li>Вызываем <code>buildPages()</code></li>
<li>(логика несколько отличается для Flex-страниц, но идея та же)</li>
<li>Проверяем наличие кэша</li>
<li>Если <strong>кэш в порядке</strong>, загружаем данные страниц из него</li>
<li>Если <strong>кэш отсутствует</strong>, вызываем <code>recurse()</code></li>
<li>Запускаем событие <strong>onBuildPagesInitialized</strong> в <code>recurse()</code></li>
<li>Если файл <code>.md</code> найден:
<div class="level level-4">
<h3 id="page-php">Page.php</h3>
<ol>
<li>Вызываем <code>init()</code> для загрузки сведений о файле</li>
<li>Устанавливаем <code>filePath</code>, <code>modified</code>, <code>id</code></li>
<li>Вызываем <code>header()</code> для инициализации переменных заголовка</li>
<li>Вызываем <code>slug()</code> для установки слага URL</li>
<li>Вызываем <code>visible()</code> для установки видимого состояния</li>
<li>Устанавливаем статус <code>modularTwig()</code> в зависимости от того, начинается ли имя папки с <code>_</code> или нет</li>
</ol>
</div></li>
<li>Запускаем событие <strong>onPageProcessed</strong></li>
<li>Если в <code>папке</code> найдены потомки <code>recurse()</code></li>
<li>Запускаем событие <strong>onFolderProcessed</strong></li>
<li>Вызываем <code>buildRoutes()</code></li>
<li>Инициализируем <code>taxonomy</code> для всех страниц</li>
<li>Строим таблицу маршрутов (<code>route</code>) для быстрого поиска</li>
</ol>
</div></li>
<li>Запускаем событие <strong>onPagesInitialized</strong> с [pages]</li>
<li>Запускаем событие <strong>onPageInitialized</strong> с [page]</li>
<li>Если страница не маршрутизируется:
<ol>
<li>Запускаем событие <strong>onPageNotFount</strong> с [page]</li>
</ol></li>
<li>Если <em>task</em>:
<ol>
<li>Запускаем событие <strong>onPageTask</strong> с [task, page]</li>
<li>Запускаем событие <strong>onPageTask.[TASK]</strong> с [task, page]</li>
</ol></li>
<li>Если <em>action</em>:
<ol>
<li>Запускаем событие <strong>onPageAction</strong> с [action, page]</li>
<li>Запускаем событие <strong>onPageAction.[ACTION]</strong> с [action, page]</li>
</ol></li>
</ol></li>
<li>Запускаем обработчик Debugger Assets
<ol>
<li>Только для панели отладки: Добавляем отладчик CSS/JS в активы</li>
</ol></li>
<li>Запускаем обработчик Render
<ol>
<li>Инициализируем <code>$grav['output']</code></li>
<li>Если <code>output</code> является объектом класса <code>ResponseInterface</code>:
<ol>
<li>Останавливаем дальнейшую обработку и выводим ответ</li>
</ol></li>
<li>Иначе:
<ol>
<li>Визуализируем страницу с помощью метода Twig <code>processSite()</code>
<div class="level level-3">
<h3 id="twig-php-1">Twig.php</h3>
<ol>
<li>Запускаем событие <strong>onTwigSiteVariables</strong></li>
<li>Получаем вывод страницы</li>
<li>Запускаем событие <strong>onTwigPageVariables</strong>, в том числе для каждой модульной подстраницы</li>
<li>Если страница не найдена или не маршрутизируется, сначала запускаем событие <strong>onPageFallBackUrl</strong> чтобы посмотреть, есть ли у нас запасной вариант для медиа-актива, а затем событие <strong>onPageNotFound</strong>, если нет</li>
<li>Устанавливаем все переменные Twig в объекте Twig</li>
<li>Устанавливаем имя шаблона на основе информации о файле/заголовке/расширении</li>
<li>Вызываем метод <code>render()</code></li>
<li>Возвращаем результирующий HTML</li>
</ol>
</div></li>
<li>Запускаем событие <strong>onOutputGenerated</strong></li>
<li>Выводим данные в буфер вывода</li>
<li>Запускаем событие <strong>onOutputRendered</strong></li>
<li>Строим объект <em>Response</em></li>
<li>Останавливаем дальнейшую обработку и выдаем ответ</li>
</ol></li>
</ol></li>
<li>Выводим заголовок и тело HTTP</li>
<li>Подключаем отладчик рендеринга (если включен)</li>
<li>Выключаем
<ol>
<li>Закрываем сессию</li>
<li>Закрываем соединение с клиентом</li>
<li>Запускаем событие <strong>onShutdown</strong></li>
</ol></li>
</ol>
</div></li>
</ol>
</div>

Каждый раз, когда у страницы вызывается метод **content()**, происходит следующий жизненный цикл:

<div class="level level-1">
<h3 id="page-php-1">Page.php</h3>
<ol>
<li>Если содержимое <strong>НЕ</strong> кэшировано:
<ol>
<li>Запускаем событие <strong>onPageContentRaw</strong></li>
<li>Обрабатываем страницу в соответствии с настройками Markdown и Twig. Запускаем событие <strong>onMarkdownInitialized</strong></li>
<li>Запускаем событие <strong>onPageContentProcessed</strong></li>
</ol></li>
<li>Запускаем событие <strong>onPageContent</strong></li>
</ol>
</div>

<style>
    .level { position: relative; }
    .level h3 { font-size: 18px; margin: 0; padding: 2px 10px 4px; left: 0; z-index: 1000; color: #fff; }
    .level strong { color: #000; }
    .level .level { margin-top: 0.5rem; margin-left: -1rem; margin-right: 1rem; margin-bottom: 0.5rem; padding-bottom: 1rem; }
    .level ol li { padding-top: 0.1rem; margin-top: 0.2rem; }
    .level > ol { margin-top: 2rem; }
    .level.level-1 { background: #f6fefc; border: 4px solid #1ABC9C; color: #1ABC9C; margin-bottom: 1rem; }
    .level.level-1 h3 { background: #1ABC9C; }
    .level.level-2 { background: #f7fdf9; border: 4px solid #2ECC71; color: #2ECC71; }
    .level.level-2 h3 { background: #2ECC71; }
    .level.level-3 { background: #f3f9fd; border: 4px solid #3498DB; color: #3498DB; }
    .level.level-3 h3 { background: #3498DB; }
    .level.level-4 { background: #e4eaf0; border: 4px solid #34495E; color: #34495E; }
    .level.level-4 h3 { background: #34495E; }
</style>