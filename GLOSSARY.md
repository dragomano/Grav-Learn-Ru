# Глоссарий перевода документации Grav

Единый словарь терминов для перевода документации Grav (`docs/`, 274 страницы, Grav 2.0 + Grav Premium)
на русский язык. Цель — чтобы один и тот же английский термин везде переводился одинаково.

---

## 1. Как пользоваться словарём

1. Перед переводом страницы находим в словаре все термины, которые в ней встречаются.
2. Если термина нет — не выдумываем на ходу, а добавляем в раздел «Новые термины» (в конце файла)
   и согласуем. Только потом используем.
3. Если термин в словаре есть, но в конкретном контексте он читается плохо — не отступаем молча,
   а помечаем страницу и выносим на обсуждение.
4. Приоритет: сначала правила из разделов 2–3, потом таблицы терминов.

---

## 2. Общие правила

### 2.1. Принципы

* Переводим смысл, а не слова: русский текст должен читаться как написанный по-русски, а не как подстрочник.
* Обращение к читателю — на «вы» со строчной буквы. Прямая речь преобладает над безличными конструкциями.
* Единообразие важнее изящества: если термин закреплён в словаре, используем его даже там, где есть
  более красивый синоним.
* UI-элементы (кнопки, пункты меню, вкладки, заголовки экранов) переводим по словарю, чтобы перевод
  совпадал с интерфейсом. Названия продуктов и плагинов — не переводим (см. 2.2).
* Аббревиатуры, знакомые русскому читателю (API, CLI, CSS, SLA, SPF, SMTP), оставляем латиницей.
* Не добавляем пояснений и примеров, которых нет в оригинале (исключение — примечания переводчика). Не удаляем ничего.
* Ссылки, пути, имена файлов, ключи YAML, классы, методы, названия полей и значений — не трогаем.

### 2.2. Что не переводим

| Что | Примеры |
| --- | --- |
| Название продукта и самой платформы | Grav, Grav Premium, Grav CMS |
| Компоненты Grav | Admin Next, Admin2, Admin Classic, Grav API, API plugin, License Manager |
| Продукты Grav Premium | Mailroom, Helpdesk Pro, Forum Pro, Algolia Pro, AI Pro, AI Translate, Comments Pro, Downloads Pro, Editor Pro, Image Optimize, Lightbox Gallery, Polls, Revisions Pro, SEO Magic, Site Toolbox, Typhoon, YetiSearch Pro, Cloudflare Manager, Helios, `email-office365` |
| Названия плагинов и тем (в коде) | `form`, `email`, `login`, `flex-objects`, `sync`, `sync-mercure`, `codesh`, `antimatter` |
| Технологии | Twig, Markdown, YAML, JSON, CSS, Sass/SCSS, Less, PHP, SQLite, MySQL, PostgreSQL, Mercure, SvelteKit, Yjs |
| Имена файлов и пути | `user/config/system.yaml`, `user/pages`, `blueprints.yaml`, `index.md` |
| Ключи, классы, методы, события | `visible`, `routable`, `cache_enable`, `FlexRegisterEvent`, `getCollection()` |
| Значения ключей | `true`, `false`, `null`, `default`, `anonymize`, `delete_content`, `#/tickets` |
| Сокращения в скобках | GPM, GPM-пакеты, MCP, REST, CSV, IMAP, DKIM, DMARC, SPF, CDN, CSRF, XSS, GDPR, CVE, SPA |
| Версии | Grav 1.7, Grav 2.0, `>= 2.1.0` |

> Названия продуктов склонять не нужно и нельзя присоединять к ним русские окончания:
> «в Admin Next», «для Mailroom», «интерфейс Helpdesk Pro».
> Не «Грав», не «Мейлрум», не «Форум Про».

### 2.3. Оформление

* `code` — всё, что в оригинале в `code`: ключи, пути, команды, имена файлов, имена шаблонов,
  названия плагинов, значения параметров, куски кода. Внутри `code` ничего не переводим (возможны обоснованные исключения).
* **Жирный** — термины, на которые автор обращает внимание, и названия UI-элементов, если так в оригинале.
  Разметку (`**`, `` ` ``, `[]()`, `![alt]()`) сохраняем на тех же местах.
* Блоки кода: код, вывод команд, примеры YAML/Twig/PHP — без изменений. Пояснительные комментарии
  внутри примеров (`# ...`, `// ...`) переводим, если они не являются частью синтаксиса.
* Ссылки: URL не меняем. Подписи ссылок переводим. **Якоря в ссылках сохраняем** — см. 2.4.
* Alt-текст изображений переводим; сами файлы изображений не трогаем.
* Заголовки в frontmatter (`title`, `description`) переводим. Ключи frontmatter — нет.

### 2.4. Якоря заголовков (важно!)

В документации более 1100 внутренних ссылок с якорями вида `../headers/index.md#visible`.
Когда мы переводим заголовок `## Visible`, MkDocs строит новый якорь `#видимая`, и все ссылки на
`#visible` ломаются.

**Правило:** переводя заголовок, на который есть ссылка с якорем, сохраняем исходный якорь явно:

```markdown
## Видимость {: #visible }
```

Синтаксис `{: #anchor }` требует включённого `attr_list` — он включён в `zensical.toml`.
Так уже сделано в части страниц (`## Menubar Items {: #menubar-items }`).

Порядок работы со страницей:

1. Находим в файле все заголовки с якорями `{: #... }` — их якоря уже закреплены, менять их нельзя.
2. Ищем ссылки на эту страницу с якорями (по всему `docs/`): `Select-String -Path docs -Recurse -Pattern 'headers/index.md#'`.
3. Для каждого такого якоря добавляем `{: #исходный-якорь }` к переведённому заголовку.
4. Проверяем сборкой.

### 2.5. Callout-блоки (врезки)

В документации 564 блока `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!IMPORTANT]`, `> [!CAUTION]`.
Тема **не переводит** их заголовки автоматически (в собранном сайте остаётся `Note`, `Tip`, `Warning`),
поэтому заголовок задаём сами — прямо в строке маркера:

```markdown
> [!NOTE] Примечание
> Текст заметки.
```

Закреплённые заголовки:

| Маркер | Заголовок перевода |
| --- | --- |
| `[!NOTE]` | Примечание |
| `[!TIP]` | Совет |
| `[!IMPORTANT]` | Важно |
| `[!WARNING]` | Предупреждение |
| `[!CAUTION]` | Внимание |

Маркер `[!NOTE]` и т. п. не меняем — это синтаксис.

### 2.6. Смешанный текст и формат

* Кавычки — русские «ёлочки»; внутри них — „лапки“. Прямые кавычки не используем.
* Тире — длинное (—) с пробелами; дефис — только внутри слов.
* «и т. д.», «и т. п.», «например», «то есть» — вместо `etc.`, `e.g.`, `i.e.`.
* Английские буквы не смешиваем с русскими окончаниями: не «плагины Grav-а». Пишем «плагины Grav»,
  «для Grav», «настройки Grav».
* Числа, единицы измерения (`300px`, `604800` секунд) и коды ответов (`404`, `200`) — как в оригинале.

---

## 3. Глоссарий

### 3.1. Базовые понятия Grav

| Английский | Русский | Примечание |
| --- | --- | --- |
| Grav | Grav | Не «Грав» |
| core | ядро | «ядро Grav»; `core` в коде — как есть |
| flat-file CMS | файловая CMS | Первое упоминание: «файловая CMS (flat-file CMS)» |
| file-based | на основе файлов | |
| package | пакет | |
| skeleton | скелет | Демонстрационный сайт-заготовка. Имена конкретных пакетов — как есть: `Blog Skeleton` |
| GPM (Grav Package Manager) | GPM (менеджер пакетов Grav) | Аббревиатуру оставляем |
| hosting / host | хостинг / хостинг-провайдер | |
| web server | веб-сервер | |
| requirements | требования | |
| prerequisites | предварительные требования | |
| installation | установка | |
| setup / setting up | настройка | |
| environment | окружение | Не «среда» |
| multisite | мультисайт | |
| stream | поток | «поток `user://`» |
| content | контент | Содержимое страницы (body) — «содержимое» |
| asset | ресурс | CSS, JS, изображения |
| asset pipeline | сборка ресурсов | |
| cache | кэш | Не «кеш» |
| caching | кэширование | |
| debug | отладка | |
| hosting setup | конфигурация хостинга | |
| documentation | документация | |

### 3.2. Страницы и контент

| Английский | Русский | Примечание |
| --- | --- | --- |
| page | страница | |
| Page (объект) | объект `Page` | В коде — как есть |
| regular page | обычная страница | |
| listing page | страница-список | |
| modular page | модульная страница | |
| module | модуль | |
| child page | дочерняя страница | |
| parent page | родительская страница | |
| sibling | соседняя страница | |
| one-page layout | одностраничный макет | |
| page file | файл страницы | |
| page folder | папка страницы | |
| folder | папка | Не «каталог»: «каталог» — только Flex Directory |
| slug | слаг | |
| route | маршрут | |
| routing | маршрутизация | |
| routable | маршрутизируемая | `routable` как ключ — как есть |
| visible | видимая | `visible` как ключ — как есть |
| invisible | невидимая | |
| published | опубликованная | |
| draft | черновик | |
| headers (страницы) | заголовки (страницы) | Не «шапка» |
| frontmatter | метаданные страницы | блок метаданных |
| metadata | метаданные | |
| meta tags | метатеги | |
| taxonomy | таксономия | |
| category | категория | |
| tag | тег | Для метки интерфейса — «метка» (см. 3.11) |
| collection | коллекция | |
| page collection | коллекция страниц | |
| ordering | порядок | «порядок страниц» |
| summary | аннотация | |
| summary separator / delimiter | разделитель аннотации | |
| excerpt | отрывок | |
| pagination | пагинация | |
| navigation | навигация | |
| menu | меню | |
| breadcrumbs | хлебные крошки | |
| table of contents | оглавление | |
| anchor | якорь | |
| permalink | постоянная ссылка | |
| canonical URL | канонический URL | |
| alias | псевдоним | «псевдоним маршрута» |
| redirect | перенаправление | |
| external URL | внешний URL | |
| content type | тип контента | |
| shortcode | шорткод | |
| multi-language | многоязычность | |
| language (страницы) | язык | |
| default language | язык по умолчанию | |
| translation | перевод | |
| media | медиафайлы | «медиа страницы» — медиафайлы страницы |
| image | изображение | |
| thumbnail | миниатюра | |
| lightbox | лайтбокс | Название плагина — как есть |
| gallery | галерея | |

### 3.3. Конфигурация, YAML, параметры

| Английский | Русский | Примечание |
| --- | --- | --- |
| configuration | конфигурация | Раздел админки — «Конфигурация» |
| config file | файл конфигурации | |
| system configuration | системная конфигурация | |
| site configuration | конфигурация сайта | |
| setting | параметр | «Настройки» — для раздела/экрана |
| option | параметр | «опция» — только для выбора из вариантов |
| property | свойство | |
| value | значение | |
| key | ключ | |
| default | значение по умолчанию | |
| override | переопределение | «переопределить» |
| fallback | резервное значение | «с запасным вариантом» |
| inherit / inheritance | наследование | |
| merge (конфигурации) | слияние | |
| validation | валидация | |
| validate | проверять | |
| environment config | конфигурация окружения | |
| blueprint | схема | |
| blueprint file | файл схемы | |

### 3.4. Twig, темы, шаблоны

| Английский | Русский | Примечание |
| --- | --- | --- |
| theme | тема | |
| template | шаблон | |
| templating engine | шаблонизатор | |
| Twig template | шаблон Twig | |
| partial | фрагмент шаблона | |
| block | блок | |
| layout | макет | |
| variable | переменная | |
| filter (Twig) | фильтр | |
| function (Twig) | функция | |
| tag (Twig) | тег | |
| macro (Twig) | макрос | |
| escaping | экранирование | |
| render | рендеринг | «рендерить», «отрендерить» |
| compile | компиляция | |
| minify | минификация | |
| source map | карта исходников | |
| stylesheet | таблица стилей | CSS-файл |
| theme variables | переменные темы | |

### 3.5. Плагины и разработка

| Английский | Русский | Примечание |
| --- | --- | --- |
| plugin | плагин | |
| event | событие | |
| event hook | хук события | |
| hook | хук | |
| listener | слушатель | |
| subscriber (события) | подписчик | |
| priority | приоритет | |
| lifecycle | жизненный цикл | |
| Dependency Injection Container | контейнер внедрения зависимостей | |
| service | сервис | |
| namespace | пространство имён | |
| class | класс | |
| method | метод | |
| property | свойство | |
| interface | интерфейс | |
| trait | трейт | |
| callback | колбек | обратный вызов |
| closure | замыкание | |
| deprecated | устаревший | |
| backward compatibility | обратная совместимость | |
| breaking change | несовместимое изменение | |
| bootstrap | загрузка | |
| middleware | мидлвар | |
| router | маршрутизатор | |
| controller | контроллер | |
| add-on | дополнение | |
| feature | фича | Не «функция» — «функция» оставлена для Twig/PHP |
| capability | возможность | |
| compatibility | совместимость | |
| scaffolding | каркас | «каркас плагина» |
| stub | заготовка | |
| dependency | зависимость | |
| repository | репозиторий | |
| commit | коммит | |
| branch | ветка | |
| pull request | пулреквест | |
| issue | задача | GitHub issue — `issue` в ссылках не трогаем |
| changelog | список изменений | |
| release | релиз / выпуск | «примечания к выпуску» |
| version | версия | |

### 3.6. Формы и схемы

| Английский | Русский | Примечание |
| --- | --- | --- |
| form | форма | |
| field | поле | |
| field type | тип поля | |
| custom field | настраиваемое поле | |
| form action | действие формы | |
| form definition | определение формы | |
| validation rule | правило валидации | |
| required | обязательное | |
| default value | значение по умолчанию | |
| captcha | капча | |
| submission | отправка | |
| submit | отправить | |
| honeypot | ловушка | `honeypot` (если речь о примере в коде схемы) |
| file upload | загрузка файла | |

### 3.7. Flex-объекты

| Английский | Русский | Примечание |
| --- | --- | --- |
| Flex | Flex | Не переводим |
| Flex Type | тип Flex | |
| Flex Directory | директория Flex | Директория — чтобы не путать с папкой |
| Flex Object | Flex-объект | |
| Flex Collection | коллекция Flex | |
| Flex Index | индекс Flex | |
| Flex Storage | хранилище Flex | |
| Flex Form | форма Flex | |
| object | объект | |
| record | запись | |
| directory | директория | |
| collection | коллекция | |
| index | индекс | |
| storage | хранилище | |
| formatter | форматтер | |
| key | ключ | |
| checksum | контрольная сумма | |
| hydrate | наполнять данными | |
| query | запрос | |
| filter | фильтр | |
| sort | сортировка | |
| search | поиск | |
| permission | разрешение | «права доступа» |

### 3.8. Админ-панель (Admin Next)

| Английский | Русский | Примечание |
| --- | --- | --- |
| admin panel | админка | |
| shell | оболочка | |
| sidebar | боковая панель | |
| topbar | верхняя панель | |
| dashboard | панель управления | |
| widget | виджет | |
| preset | набор | `Default` / `Minimal` / `Compact` → «По умолчанию» / «Минимум» / «Компактный» |
| toolbar | панель инструментов | |
| content area | область содержимого | |
| page editor | редактор страниц | |
| Normal / Expert | Обычный / Эксперт | |
| right rail | правая панель | |
| page info | информация о странице | |
| translations | переводы | |
| page media | медиафайлы страницы | |
| page navigator | навигатор по страницам | |
| view | представление | `Tree` / `List` / `Columns` → «Дерево» / «Список» / «Колонки» |
| add page | добавить страницу | |
| add folder | добавить папку | |
| add module | добавить модуль | |
| environment selector | переключатель окружений | |
| presence | присутствие | |
| avatar | аватар | |
| collaborative editing | совместное редактирование | |
| sync | синхронизация | Плагин `sync` — как есть |
| notification | уведомление | |
| bell | значок уведомлений | |
| toggle | переключатель | |
| dropdown | выпадающий список | |
| checkbox | флажок | |
| tab | вкладка | |
| panel | панель | |
| icon | иконка | |
| badge | значок | |
| pill | метка | «метка „Черновик“» |
| chip | чип | Фильтр-чип; кнопка-метка; переключатель |
| shortcut | ярлык | Экран под элементом боковой панели, который пользователь закрепляет звёздочкой. Не «ярлык Windows» |
| star (действие) | поставить звёздочку / снять звёздочку | Закрепление ярлыка в боковой панели; `star` в коде — как есть |
| tooltip | всплывающая подсказка | |
| toast | всплывающее уведомление | |
| super-admin | супер-администратор | |
| account | учётная запись | |
| user | пользователь | |
| group | группа | |
| permission | разрешение | «права доступа» |
| access | доступ | |
| read-only | только для чтения | Режим без прав на запись: демо-учётная запись, форум в режиме только для чтения, поле формы, страница |
| login / sign in | вход | Не «логин» |
| logout | выход | |
| sign up / register | регистрация | |
| invitation | приглашение | Приглашение пользователя на сайт или в группу: Admin Next, `api/endpoints/users` |
| password | пароль | |
| two-factor authentication | двухфакторная аутентификация | |
| filter input | поле фильтра | |
| search | поиск | |
| draft | черновик | |
| publish | публикация | «опубликовать» |
| unpublish | снятие с публикации | |
| preview | предпросмотр | |
| save | сохранить | |
| delete | удалить | |
| edit | изменить | |
| create | создать | |
| copy | копировать | |
| cancel | отмена | |
| back / next | назад / далее | |
| refresh | обновить | |
| update | обновление | |
| upgrade | обновление | Смена мажорной версии |
| install | установка | «установить» |
| enable / disable | включить / отключить | |
| enabled / disabled | включено / отключено | |
| marked as read | отмечено как прочитанное | |
| unread | непрочитанное | |

### 3.9. CLI (командная строка)

| Английский | Русский | Примечание |
| --- | --- | --- |
| command line | командная строка | |
| CLI | CLI | |
| terminal | терминал | |
| shell | оболочка | |
| command | команда | |
| argument | аргумент | |
| option / flag | параметр / флаг | |
| verbose | подробный вывод | |
| exit code | код завершения | |
| prompt | приглашение | |
| output | вывод | |
| execute / run | выполнить / запустить | |
| interactive | интерактивный | |
| cron | cron | Не переводим |
| crontab | crontab | |
| scheduler | планировщик | |
| job | задание | |
| task | задача | |
| queue | очередь | |
| worker | воркер | обработчик очереди |
| run (сущ.) | прогон | в зависимости от контекста |
| due | к выполнению | |

### 3.10. REST API, MCP, интеграции

| Английский | Русский | Примечание |
| --- | --- | --- |
| endpoint | эндпоинт | |
| API reference | справочник API | |
| request | запрос | |
| response | ответ | |
| payload | данные запроса/ответа | в зависимости от контекста |
| body | тело | «тело запроса», «тело ответа» |
| header (HTTP) | заголовок | |
| status code | код состояния | |
| authentication | аутентификация | |
| authorization | авторизация | |
| token | токен | |
| Bearer token | Bearer-токен | |
| API key | ключ API | |
| credential | учётные данные | |
| webhook | вебхук | |
| rate limit | ограничение частоты запросов | |
| throttle | ограничение скорости | |
| pagination | пагинация | |
| offset / limit | смещение / лимит | |
| cursor | курсор | |
| schema | схема | |
| integration | интеграция | |
| audit | аудит | |
| audit log | журнал аудита | |
| permission | разрешение | Права: `api.access` — как есть |
| scope | область действия | |
| demo mode | демо-режим | Режим публичного демо-сайта: общая учётная запись и автоматический откат к начальному состоянию |
| demo account | демо-учётная запись | Учётная запись с правом «Демо-режим» (`access.api.demo`) |
| baseline | базовый снимок | Сохранённое начальное состояние папок, к которому возвращается сброс. Не путать с `backup` → «резервная копия» |
| safety copy | страховочная копия | Копия папок, сделанная перед сбросом. Не «резервная копия» |
| reset | сброс | Возврат папок к базовому снимку; также «сброс пароля», «сброс ключа» |
| grant | разрешение | Синоним `permission`: что разрешено демо-учётной записи |
| visitor | посетитель | Кто заходит на публичный демо-сайт. Не «гость» (`guest` → «гость») |

### 3.11. Производительность, кэш, инфраструктура

| Английский | Русский | Примечание |
| --- | --- | --- |
| performance | производительность | |
| optimization | оптимизация | |
| cache | кэш | |
| clear cache | очистка кэша | |
| cache busting | принудительный сброс кэша | |
| warm up cache | прогрев кэша | |
| stale | устаревший | |
| TTL | время жизни (TTL) | |
| compression | сжатие | |
| memory | память | |
| timeout | тайм-аут | |
| log | журнал | Для файлов — «журнал (лог)» |
| stack trace | трассировка стека | |
| error | ошибка | |
| warning | предупреждение | |
| failure | сбой | |
| exception | исключение | |
| handler | обработчик | |
| debugging | отладка | |
| debugger | отладчик | |
| health check | проверка состояния | |
| backup | резервная копия | |
| restore | восстановление | |
| migration | миграция | |
| symlink | символическая ссылка | |
| permissions (файлы) | права доступа | |
| ownership | владелец | |
| umask | umask | |

### 3.12. Безопасность и приватность

| Английский | Русский | Примечание |
| --- | --- | --- |
| security | безопасность | |
| vulnerability | уязвимость | |
| exploit | эксплойт | |
| mitigation | меры защиты | |
| advisory | уведомление о безопасности | |
| disclosure | раскрытие | |
| hardening | усиление защиты | |
| trusted host | доверенный хост | |
| security token | токен безопасности | |
| sanitize | очистка | |
| escape | экранирование | |
| encryption | шифрование | |
| hash | хеш | |
| privacy | конфиденциальность | |
| consent | согласие | |
| personal data | персональные данные | |
| data subject | субъект данных | |
| subject access request | запрос субъекта данных | |
| retention | хранение | «срок хранения» |
| retention window | срок хранения | |
| erasure | удаление данных | GDPR: «право на забвение» |
| erase | удалить данные | |
| anonymize | анонимизировать | |
| anonymization | анонимизация | |
| legal hold | юридическое удержание | |
| suppression | исключение | «список исключений» |
| blocked | заблокированный | |
| tombstone | надгробие | |
| queue / job (данные) | очередь / задание | |

### 3.13. Grav Premium: общее

| Английский | Русский | Примечание |
| --- | --- | --- |
| license | лицензия | |
| license key | лицензионный ключ | |
| trial | пробный период | |
| purchase | покупка | |
| product | продукт | |
| settings | настройки | |
| report | отчёт | |
| metrics | метрики | Числовые показатели в сводке и на экране «Панель управления» |
| automation | автоматизация | |
| notification channel | канал уведомлений | |
| digest | дайджест | |
| deliverability | доставляемость | |
| bounce | возврат | |
| hard bounce / soft bounce | жёсткий / мягкий возврат | |
| complaint | жалоба | |
| unsubscribe | отписка | |
| double opt-in | двойное подтверждение подписки | |
| provider | провайдер | |
| transport | транспорт | |

### 3.14. Mailroom

| Английский | Русский | Примечание |
| --- | --- | --- |
| newsletter | рассылка | Как продукт — «рассылка новостей» |
| subscriber | подписчик | |
| list | список | «список рассылки» |
| signup form | форма подписки | |
| signup box | блок подписки | |
| campaign | кампания | |
| subject test | тест темы письма | |
| test send | тестовая отправка | |
| scheduling | планирование | |
| segment | сегмент | |
| tag | тег | |
| template | шаблон | |
| branding | брендирование | |
| automation | автоматизация | |
| open | открытие | |
| click | переход | |
| open rate | процент открытий | |
| click rate | процент переходов | |
| tracking | отслеживание | |
| preference center | центр предпочтений | |
| import / export | импорт / экспорт | |
| column mapping | сопоставление столбцов | |
| dry run | пробный запуск | |
| consent record | запись согласия | |
| consent history | история согласий | |
| suppression hash | хеш исключения | |

### 3.15. Helpdesk Pro

| Английский | Русский | Примечание |
| --- | --- | --- |
| helpdesk | служба поддержки | |
| ticket | тикет | |
| request | запрос | В Helpdesk может быть «заявка», если речь о клиентском обращении |
| reply | ответ | Публичное сообщение |
| note | заметка | Внутренняя заметка |
| activity | активность | |
| project | проект | |
| person | человек | |
| requester | заявитель | |
| cc | копия | «добавлен в копию» |
| watcher | наблюдатель | |
| agent | агент | |
| staff | сотрудники | |
| client | клиент | |
| guest | гость | |
| status | статус | |
| status category | категория статуса | |
| desk | рабочая область | |
| triage | триаж | первичная обработка и сортировка входящих тикетов |
| channel | канал | |
| visibility scope | область видимости | |
| custom field | настраиваемое поле | |
| board | доска | |
| column | колонка | |
| filter chip | фильтр-чип | |
| saved reply | сохранённый ответ | |
| saved view | сохранённый вид | |
| macro | макрос | |
| bulk edit | массовое изменение | |
| assignee | исполнитель | |
| assignment | назначение | |
| priority | приоритет | |
| kind | вид | `support` / `bug` / `feature` — как есть |
| label | метка | |
| merge (тикетов) | объединение | |
| duplicate | дубликат | |
| decline | отклонение | |
| close | закрытие | |
| reopen | повторное открытие | |
| resolve | решение | «отметить решённым» |
| internal note | внутренняя заметка | |
| mention | упоминание | |
| notification channel | канал уведомлений | |
| knowledge base | база знаний | |
| help center | справочный центр | |
| portal | портал | «клиентский портал» |
| sign-in link | ссылка для входа | |
| SLA | SLA | Соглашение об уровне обслуживания |
| business hours | рабочие часы | |
| breach | нарушение | «нарушение SLA» |
| due soon | скоро срок | |
| auto-close | авто-закрытие | |
| rule | правило | |
| condition | условие | |
| action | действие | |
| organization | организация | |
| rating | оценка | |
| customer rating | оценка клиента | |
| attachments | вложения | |
| attachment | вложение | |
| outbound email | исходящая почта | |
| inbound email | входящая почта | |
| inbound log | журнал входящей почты | |
| mailbox | почтовый ящик | |
| team inbox | общий ящик | |
| escalation | эскалация | |
| spam score | оценка спама | |

### 3.16. Forum Pro

| Английский | Русский | Примечание |
| --- | --- | --- |
| forum | форум | |
| discussion | обсуждение | |
| section | раздел | |
| category | категория | |
| topic | тема | |
| thread | ветка | |
| threaded replies | вложенные ответы | |
| post | сообщение | Не «пост» |
| reply | ответ | |
| quote | цитата | |
| mention | упоминание | |
| attachment | вложение | |
| reaction | реакция | |
| poll | опрос | |
| tag | тег | |
| accepted answer | принятый ответ | |
| bookmark | закладка | |
| revision history | история версий | |
| member | участник | |
| member directory | список участников | |
| registration | регистрация | |
| email verification | подтверждение почты | |
| password reset | сброс пароля | |
| profile | профиль | |
| avatar | аватар | |
| rank | ранг | |
| badge | значок | |
| group | группа | |
| title (участника) | титул | |
| moderation | модерация | |
| moderator | модератор | |
| moderator role | роль модератора | |
| premoderation | премодерация | |
| review queue | очередь проверки | |
| member report | жалоба участника | |
| ban | блокировка | бан |
| appeal | апелляция | |
| content disposition | судьба контента | |
| audit log | журнал аудита | |
| spam defense | защита от спама | |
| heuristic | эвристика | |
| classifier | классификатор | |
| provisional account | предварительная учётная запись | |
| private message | личное сообщение | |
| watch level | уровень отслеживания | |
| live updates | обновления в реальном времени | |
| presence | присутствие | |
| typing indicator | индикатор набора текста | |
| importer | инструмент импорта | |
| idempotent | идемпотентный | |
| cutover | переключение | |
| comment | комментарий | |
| commentable article | статья с комментариями | |

### 3.17. Поиск

| Английский | Русский | Примечание |
| --- | --- | --- |
| search | поиск | |
| full-text search | полнотекстовый поиск | |
| semantic search | семантический поиск | |
| fuzzy search | нечёткий поиск | |
| index | индекс | |
| reindex | переиндексация | |
| rebuild (индекса) | перестроение | |
| relevance | релевантность | |
| facet | фасет | |
| autocomplete | автодополнение | |
| synonym | синоним | |
| stop word | стоп-слово | |
| stemming | стемминг | |
| highlight | подсветка | |
| snippet | фрагмент | |
| engine | движок | |

### 3.18. ИИ

| Английский | Русский | Примечание |
| --- | --- | --- |
| AI | ИИ | В названиях продуктов — `AI Pro`, `AI Translate` |
| prompt | промпт | |
| token (LLM) | токен | |
| embedding | эмбеддинг | |
| provider | провайдер | |
| model | модель | |
| assistant | ассистент | |

### 3.19. Миграция и развёртывание

| Английский | Русский | Примечание |
| --- | --- | --- |
| wizard | мастер | «мастер миграции» |
| handoff | передача управления | |
| stage (сущ.) | подготовленная копия; подготовленная установка | `stage directory` — папка подготовки; `staged site` — подготовленный сайт |
| stage (глагол) | подготовить | |
| staging | подготовка | |
| promote | переключение; переключить | |
| rollback | откат | |
| live site | действующий сайт | |
| live install | действующая установка | |
| live webroot | рабочий webroot | |
| production site | рабочая версия сайта | Не путать с `live site` — «действующий сайт» |
| compatibility flag | флаг совместимости | `compatibility:` в `blueprints.yaml` |
| allowlist / denylist | список разрешённых / список запрещённых | |
| sandbox | песочница | `Twig content sandbox` — песочница Twig в контенте |
| vendor stack | стек vendor | Symfony, Twig, Monolog и другие зависимости |

---

## 4. Типовые фразы и заголовки

### 4.1. Заголовки страниц и разделов

| Английский | Русский |
| --- | --- |
| In this section | В этом разделе |
| Overview | Обзор |
| Getting Started | Начало работы |
| Before you begin | Перед началом |
| Requirements | Требования |
| Prerequisites | Предварительные требования |
| Installation | Установка |
| Manual Installation | Установка вручную |
| Configuration | Конфигурация |
| Settings | Настройки |
| Configuration Options | Параметры конфигурации |
| Usage | Использование |
| Example / Examples | Пример / Примеры |
| How It Works | Как это работает |
| Best Practices | Рекомендации |
| Troubleshooting | Решение проблем |
| Common Problems | Типичные проблемы |
| Problem: | Проблема: |
| Goal: | Цель: |
| Solution: | Решение: |
| Command | Команда |
| Results | Результат |
| Next steps | Что дальше |
| Where to go next | Что дальше |
| Related | См. также |
| Resources | Ресурсы |
| Credits | Благодарности |
| Table of Contents | Оглавление |
| What's in the box | Что внутри |
| API and MCP | API и MCP |
| API Reference | Справочник API |
| Advanced | Для продвинутых |
| Accessibility | Доступность |
| Privacy | Конфиденциальность |
| Permissions | Права доступа |
| Authentication | Аутентификация |
| Storage | Хранилище |
| Templates | Шаблоны |
| Template Variables | Переменные шаблона |
| Directories | Директории |
| Events | События |
| Filters | Фильтры |
| Tags | Теги |
| Options | Параметры |
| Controls | Элементы управления |
| In the desk | В рабочей области |
| Set up cron | Настройка cron |
| Legal holds | Юридические удержания |
| Live Updates | Обновления в реальном времени |
| Reports | Отчёты |

### 4.2. Ходовые обороты

| Английский | Русский |
| --- | --- |
| Note | Примечание |
| Tip | Совет |
| Important | Важно |
| Warning | Предупреждение |
| Caution | Внимание |
| See also | См. также |
| Learn more | Подробнее |
| For more information | Подробнее |
| ... is not compatible with ... | ... несовместим(а) с ... |
| requires | требует |
| needs | нужен / требуется |
| by default | по умолчанию |
| for example | например |
| that is | то есть |
| and so on | и т. д. |
| make sure | убедитесь |
| keep in mind | учитывайте |
| the same | то же самое |
| on your site | на вашем сайте |
| out of the box | из коробки |
| start with | начните с |
| this page covers | эта страница посвящена |
| you can | вы можете |
| click | нажмите |
| open | откройте |
| choose / pick | выберите |
| leave empty | оставьте пустым |
| set to | установите в |
| once | после того как / как только |

---

## 5. Названия разделов сайта и теги

### 5.1. Разделы навигации (`zensical.toml`)

| Сейчас | Предлагается |
| --- | --- |
| Home | Главная |
| Basics | Основы |
| Content | Контент |
| Migration | Миграция |
| Themes | Темы |
| Plugins | Плагины |
| Admin Panel (`admin-panel`) | Админка (если речь о разделе управления сайтом), Admin Panel (если речь о плагине) |
| Forms | Формы |
| Blueprints | Схемы |
| Frontend Forms | Формы на сайте |
| REST API | REST API |
| REST API Reference | Справочник REST API |
| CLI Console | Консоль CLI |
| Advanced | Для продвинутых |
| Grav Development | Разработка Grav |
| Flex Objects | Flex-объекты |
| Using Flex Objects | Работа с Flex-объектами |
| Creating a Custom Type | Создание своего типа |
| Managing Flex in Admin Next | Управление Flex в Admin Next |
| Cookbook | Рецепты |
| Tutorials | Уроки |
| Grav Premium | Grav Premium |
| Forum Pro | Forum Pro |
| Helpdesk Pro | Helpdesk Pro |
| Mailroom | Mailroom |
| Algolia Pro | Algolia Pro |
| Troubleshooting | Решение проблем |
| Security | Безопасность |
| Reports | Отчёты |
| Hints and Tips | Полезные советы |
| Blogging | Блогинг |
| Tags | Теги |
| Migrating to Grav | Переход на Grav |
| Twig Tags, Filters & Functions | Теги, фильтры и функции Twig |

### 5.2. Теги в блоке метаданных (frontmatter)

Теги видны на странице тегов и в карточках, поэтому их тоже переводим — по единому списку.
Названия продуктов оставляем.

| Сейчас | Предлагается |
| --- | --- |
| Basics | Основы |
| Content | Контент |
| Pages | Страницы |
| Media | Медиафайлы |
| Markdown | Markdown |
| Collections | Коллекции |
| Taxonomy | Таксономия |
| Routing | Маршрутизация |
| Multilingual | Многоязычность |
| Shortcodes | Шорткоды |
| Plugins | Плагины |
| Themes | Темы |
| Twig | Twig |
| Forms | Формы |
| Blueprints | Схемы |
| Configuration | Конфигурация |
| YAML | YAML |
| PHP | PHP |
| CSS | CSS |
| Development | Разработка |
| Frontend | Фронтенд |
| CLI | CLI |
| GPM | GPM |
| Events | События |
| API Reference | Справочник API |
| REST API | REST API |
| Authentication | Аутентификация |
| Webhooks | Вебхуки |
| MCP | MCP |
| Security | Безопасность |
| Permissions | Права доступа |
| Privacy | Конфиденциальность |
| Users | Пользователи |
| Migration | Миграция |
| Upgrade | Обновление |
| Installation | Установка |
| Getting Started | Начало работы |
| Troubleshooting | Решение проблем |
| Scheduler | Планировщик |
| Performance | Производительность |
| Debugging | Отладка |
| Backups | Резервные копии |
| Search | Поиск |
| Notifications | Уведомления |
| Reports | Отчёты |
| Automation | Автоматизация |
| Email | Электронная почта |
| Blogging | Блогинг |
| Community | Сообщество |
| Support | Поддержка |
| AI | ИИ |
| SEO | SEO |
| Multisite | Мультисайт |
| HTTP Errors | Ошибки HTTP |
| Advanced | Для продвинутых |
| Cookbook | Рецепты |
| Documentation | Документация |
| Hints and Tips | Советы |
| Grav | Grav |
| Grav 1.7 | Grav 1.7 |
| Grav Premium | Grav Premium |
| Admin Next | Admin Next |
| Flex Objects | Flex-объекты |
| WordPress | WordPress |
| Drupal | Drupal |
| Cloudflare | Cloudflare |
| Office 365 | Office 365 |
| Algolia Pro, AI Pro, AI Translate, Comments Pro, Downloads Pro, Editor Pro, Helios, Image Optimize, License Manager, Lightbox Gallery, Polls, Revisions Pro, SEO Magic, Site Toolbox, Typhoon, YetiSearch Pro | без изменений |

---

## 6. Новые термины

Сюда добавляем термины, которых ещё нет в словаре, вместе с контекстом. После согласования
переносим их в нужный раздел.

| Английский | Предлагаемый перевод | Контекст / страница | Статус |
| --- | --- | --- | --- |
| | | | |
