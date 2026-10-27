---
title: "События"
description: "API вызывает события до и после каждой операции записи и использует события, чтобы плагины могли расширять интерфейс Admin2 (боковые панели, панели меню, виджеты, контекстные панели, панели настроек, компоненты пользовательских полей, пользовательские отчёты)…"
tags:
  - "REST API"
  - "Справочник API"
  - "События"
---

# События API

API вызывает события до и после каждой операции записи и использует события, чтобы плагины могли расширять интерфейс Admin2 (боковые панели, панели меню, виджеты, контекстные панели, панели настроек, компоненты пользовательских полей, пользовательские отчёты). Плагины реагируют на эти события, чтобы проверять и изменять данные, отменять операции или собирать интерфейс.

## События страниц

| Событие | Когда | Данные |
|-------|------|------|
| `onApiBeforePageCreate` | До создания страницы (изменяемое) | `route`, `header`, `content`, `template`, `lang` |
| `onApiPageCreated` | После создания страницы (также после копирования или пакетного копирования) | `page`, `route`, `lang`; при копировании вместо `lang` отправляются `page`, `route`, `source_route` и `method` (`copy` или `batch`) |
| `onApiBeforePageUpdate` | До обновления страницы (изменяемое) | `page`, `data` (тело запроса); для пакетного обновления `method` равно `batch` |
| `onApiPageUpdated` | После обновления страницы | `page`; `previous_template`, когда шаблон изменился; при пакетном обновлении отправляются `page`, `route` и `method` (`batch`) |
| `onApiBeforePageDelete` | До удаления страницы | `page`, плюс `lang`, когда удаляется один перевод, или `method` (`batch`) при пакетном удалении |
| `onApiPageDeleted` | После удаления страницы | `route`, плюс `lang`, когда удаляется один перевод, или `method` (`batch`) при пакетном удалении |
| `onApiPageMoved` | После перемещения страницы (также по одному разу на каждую перемещённую страницу после реорганизации) | `page`, `old_route`, `new_route`; `method` равно `reorganize`, когда событие приходит из `POST /pages/reorganize` |
| `onApiBeforePageTranslate` | До создания перевода (изменяемое) | `page`, `lang`, `header`, `content` |
| `onApiPageTranslated` | После создания перевода | `page`, `route`, `lang` |
| `onApiBeforePageAdoptLanguage` | До принятия базового файла как файла конкретного языка | `page`, `route`, `lang`, `from_file`, `to_file` |
| `onApiPageLanguageAdopted` | После переименования базового файла в имя файла конкретного языка | `page`, `route`, `lang` |
| `onApiBeforePageSync` | До перезаписи перевода из другого языка (изменяемое) | `page`, `source_lang`, `target_lang`, `header`, `content` |
| `onApiPageSynced` | После перезаписи перевода | `page`, `route`, `source_lang`, `target_lang` |
| `onApiBeforePagesReorder` | До изменения порядка дочерних страниц | `parent`, `order` |
| `onApiPagesReordered` | После изменения порядка дочерних страниц | `parent`, `order` |
| `onApiBeforePagesReorganize` | До массового перемещения или изменения порядка | `operations` (разрешённый список) |
| `onApiPagesReorganized` | После массового перемещения или изменения порядка | `operations` |

## События медиафайлов

| Событие | Когда | Данные |
|-------|------|------|
| `onApiBeforeMediaUpload` | До загрузки каждого файла | `page`, `filename`, `type`, `size`; для медиафайлов сайта добавляется `path` |
| `onApiMediaUploaded` | После завершения загрузки | `page`, `filenames`; для медиафайлов сайта добавляется `path` |
| `onApiBeforeMediaDelete` | До удаления медиафайла | `page`, `filename`; для медиафайлов сайта добавляется `path` |
| `onApiMediaDeleted` | После удаления медиафайла | `page`, `filename`; для медиафайлов сайта добавляется `path` |
| `onApiMediaMetadataUpdated` | После сохранения метаданных медиафайла | `page`, `filename` для медиафайлов страницы; `path`, `filename` для медиафайлов сайта |
| `onApiMediaMetadataDeleted` | После удаления метаданных медиафайла | `page`, `filename` для медиафайлов страницы; `path`, `filename` для медиафайлов сайта |

Для медиафайлов сайта (эндпоинты `/media`) страницы нет, поэтому `page` равно `null`, а `path` содержит путь к файлу или папке относительно `user://media`. Слушателю, который читает `page` без проверки на null, нужно это учитывать.

## События конфигурации

| Событие | Когда | Данные |
|-------|------|------|
| `onApiConfigUpdated` | После сохранения конфигурации | `scope`, `data` |

## События аутентификации и пользователей

| Событие | Когда | Данные |
|-------|------|------|
| `onApiUserLogin` | После успешного входа по паролю, через 2FA или SSO | `user`, `method` (`password`/`2fa`/`sso`), `ip`, `request`; для SSO добавляется `provider` |
| `onApiUserLoginFailure` | После любой неудачной попытки входа | `username`, `reason` (`password`/`2fa`/`disabled`/`no_api_access`), `ip` |
| `onApiUserLogout` | После `/auth/revoke` с действительным токеном | `user`, `ip`, `request` |
| `onApiPasswordReset` | После успешного `/auth/reset-password` | `user`, `ip` |
| `onApiUserCreated` | После создания пользователя (через `/users`, `/auth/setup` или принятое приглашение) | `user` |
| `onApiUserUpdated` | После обновления пользователя | `user` |
| `onApiBeforeUserDelete` | До удаления пользователя | `user` |
| `onApiUserDeleted` | После удаления пользователя | `username` |
| `onApiUser2faEnabled` | После включения 2FA | `user` |
| `onApiUser2faDisabled` | После выключения 2FA | `user`, `forced_by_admin` (bool) |
| `onApiSetupComplete` | После того как первичная настройка создала первого суперадминистратора | `user` |
| `onApiInvitationAccepted` | После принятия приглашения и создания учётной записи по нему | `user`, `invitation` |
| `onApiGroupCreated` | После создания группы пользователей | `groupname`, `group` |
| `onApiGroupUpdated` | После обновления группы пользователей | `groupname`, `group` |
| `onApiGroupDeleted` | После удаления группы пользователей | `groupname` |

### События единого входа

Они позволяют плагину входа добавить провайдера на экран входа Admin2. API владеет маршрутами и обработкой сессии; плагин общается только с провайдером.

| Событие | Эндпоинт | Данные |
|-------|----------|------|
| `onApiLoginProviders` | `GET /auth/sso/providers` | `providers` (изменяемый список) |
| `onApiLoginStart` | `GET /auth/sso/{provider}/start` | `provider`, `request`, `callback_url`, `return_to`, `redirect` (устанавливается в URL авторизации провайдера), `error` |
| `onApiLoginCallback` | `GET /auth/sso/{provider}/callback` | `provider`, `request`, `callback_url`, `user` (устанавливается в пользователя Grav, выполнившего вход), `error` |

После того как `onApiLoginCallback` вернёт пользователя, API выполняет те же проверки доступа к API, блокировки учётной записи и 2FA, что и при входе по паролю, а затем вызывает `onApiUserLogin` с `method`, равным `sso`.

## События GPM

| Событие | Когда | Данные |
|-------|------|------|
| `onApiBeforePackageInstall` | До установки | `package`, `type` (`plugin`/`theme`) |
| `onApiPackageInstalled` | После установки | `package`, `type`, `dependencies` |
| `onApiBeforePackageRemove` | До удаления | `package`, `type` |
| `onApiPackageRemoved` | После удаления | `package`, `type` |
| `onApiBeforePackageUpdate` | До обновления одного пакета | `package`, `type` |
| `onApiPackageUpdated` | После обновления пакета | `package`, `type`, `dependencies` |
| `onApiBeforeGravUpgrade` | До обновления ядра Grav | `current_version`, `available_version` |
| `onApiGravUpgraded` | После обновления ядра Grav | `previous_version`, `new_version` |

## События схем

| Событие | Когда | Данные |
|-------|------|------|
| `onApiBlueprintResolved` | После сериализации схемы для Admin2 (изменяемое `fields`) | `context`, `fields`, `user`, плюс `template`, `plugin`, `theme` или `page_id` в зависимости от схемы |
| `onAdminPageTypes` / `onAdminModularPageTypes` | Пока `GET /blueprints/pages` строит список типов страниц | `types` (по ссылке) |

`context` сообщает слушателю, какую схему он видит: `page` (с `template`), `plugin` (с `plugin`), `theme` (с `theme`), `account`, `group` или `group_new` (с `template`), `config` (только для конфигурации учётных записей, с `template`, равным `accounts`) или `plugin-page` (с `plugin` и `page_id`). Проверяйте его перед изменением полей, чтобы слушатель, предназначенный для схем страниц, не трогал остальные.

## События интеграции с Admin2

Они вызываются, пока Admin2 собирает интерфейс. Плагины добавляют элементы, виджеты и панели в данные события, чтобы зарегистрировать свои интеграции. Текущий пользователь передаётся, чтобы слушатели могли ограничивать регистрацию по разрешениям, кроме `onApiGenerateReports` и `onApiLogFiles`, которые не несут `user`.

| Событие | Эндпоинт | Данные |
|-------|----------|------|
| `onApiSidebarItems` | `GET /sidebar/items` | `items`, `user` |
| `onApiMenubarItems` | `GET /menubar/items` | `items`, `user` |
| `onApiMenubarAction` | `POST /menubar/actions/{plugin}/{action}` | `plugin`, `action`, `body`, `user`, `result` (изменяемое) |
| `onApiFloatingWidgets` | `GET /floating-widgets` | `widgets`, `user` |
| `onApiContextPanels` | `GET /context-panels` | `panels`, `user` |
| `onApiAdminSettingsPanels` | `GET /settings/panels` | `panels`, `user` |
| `onApiPluginPageInfo` | `GET /gpm/plugins/{slug}/page` | `plugin`, `definition` (изменяемое), `user` |
| `onApiGenerateReports` | `GET /reports` | `reports` |
| `onApiDashboardNotifications` | `GET /dashboard/notifications` | `notifications` (изменяемое, сгруппированное по месту), `user`, `force` |
| `onApiDashboardWidgets` | `GET /dashboard/widgets` | `widgets`, `user` |
| `onApiMarkdownEditorButtons` | `GET /editor/toolbar-buttons` | `buttons`, `user` |
| `onApiLogFiles` | `GET /system/logs/files` | `files` (изменяемый список `{file, label}`) |
| `onApiUserListFilters` | `GET /users/filters` | `filters`, `defaultFilter`, `showAll`, `user` |
| `onApiUserListFilter` | `GET /users?filter={id}` | `filter`, `collection` (изменяемая), `query`, `user` |
| `onApiUserListColumns` | `GET /users/columns` | `columns`, `user` |
| `onApiUserListColumnData` | `GET /users` | `usernames`, `data` (изменяемое), `user` |
| `onApiUserListRowActions` | `GET /users/row-actions` | `actions`, `user` |
| `onApiUserListRowAction` | `POST /users/{username}/row-action` | `id`, `plugin`, `action`, `username`, `user`, `result` (изменяемое) |

### Вкладки фильтров списка пользователей

Эти два события позволяют плагину добавить в список пользователей вкладку, показывающую отфильтрованный набор учётных записей, — аналог вкладок страницы пользователей из классической админки в Admin Next.

`onApiUserListFilters` собирает вкладки для строки навигации. Добавьте описание вкладки в массив `filters`; основной эндпоинт ставит первым встроенную вкладку `all` («All Users») и отбрасывает любую вкладку, на которую вызывающая сторона не авторизована. Событие также несёт две политики уровня страницы — `defaultFilter` (вкладка, на которой клиент открывается, когда в URL нет `filter`) и `showAll` (установите `false`, чтобы скрыть вкладку «All Users», когда показ всех учётных записей не является полезным стартовым представлением):

```php
public function onApiUserListFilters(Event $event): void
{
    // У события нет offsetGet по ссылке — добавляйте через чтение-изменение-запись,
    // тем же приёмом, что в onApiSidebarItems / onApiMenubarItems.
    $filters = $event['filters'];
    $filters[] = [
        'id'        => 'active',           // выбирается через ?filter=active; 'all' зарезервирован
        'plugin'    => 'my-plugin',
        'label'     => 'Active',           // обычный текст, не ключ перевода
        'icon'      => 'fa-bolt',          // необязательно
        'priority'  => 10,                 // необязательно, большее значение сортируется раньше
        'authorize' => 'api.users.read',   // необязательно, строка или массив «любое из»
    ];
    $event['filters'] = $filters;

    // Необязательная политика уровня страницы.
    $event['defaultFilter'] = 'active';    // открываться здесь, когда в URL нет ?filter
    $event['showAll'] = false;             // скрыть встроенную вкладку "All Users"
}
```

Поле `icon` принимает общий формат `IconSpec`: сокращение Font Awesome (`fa-bolt`, `fa-regular:clock`, `fa-brands:github`), любые CSS-классы иконок, которые Admin2 уже загружает (`class:ti ti-user`), или безопасный структурированный объект SVG (`['type' => 'svg', 'path' => '…']`). Необработанная разметка SVG не принимается — иконки SVG собираются заново из разрешённых тегов и атрибутов.

`defaultFilter` учитывается, только когда он соответствует вкладке, которую вызывающая сторона действительно видит, иначе используется первая вкладка в строке. `showAll => false` убирает «All Users», только когда после авторизации остаётся хотя бы одна собственная вкладка плагина, поэтому строка никогда не окажется пустой.

`onApiUserListFilter` выполняется, пока `GET /users?filter={id}` строит список, — после поиска и сортировки, но **до** фильтрации по разрешениям и группам и постраничной разбивки. Плагин, владеющий активным id `filter`, сужает коллекцию и присваивает её обратно; основной эндпоинт по-прежнему применяет область доступа вызывающей стороны и разбивает на страницы, поэтому вкладка может только сузить уже видимое, но никогда не расширить:

```php
public function onApiUserListFilter(Event $event): void
{
    if ($event['filter'] !== 'active') {
        return;
    }
    // filter() работает с обоими бэкендами учётных записей; см. ниже.
    $event['collection'] = $event['collection']->filter(
        fn ($user) => $user->get('state', 'enabled') === 'enabled'
    );
}
```

Вкладки фильтров работают с обоими бэкендами учётных записей, и событие несёт одинаковые ключи для каждого. На бэкенде учётных записей Flex (по умолчанию в Grav) `collection` — это `FlexCollectionInterface`, и слушатель должен присвоить обратно коллекцию Flex, иначе его результат игнорируется. В файловом хранилище учётных записей `collection` — это Grav `ArrayCollection` пользователей с ключами по имени пользователя, поддерживающая `filter()`, `matching()` и перебор, но не имеющая методов Flex вроде `filterBy()`; слушатель может присвоить обратно любой перебираемый набор пользователей, и сохранятся только учётные записи, уже присутствующие в списке. `filter()` с замыканием работает с обоими. Чтобы использовать метод только для Flex, сначала проверьте тип:

```php
use Grav\Framework\Flex\Interfaces\FlexCollectionInterface;

$collection = $event['collection'];
$event['collection'] = $collection instanceof FlexCollectionInterface
    ? $collection->filterBy(['state' => 'enabled'])
    : $collection->filter(fn ($user) => $user->get('state', 'enabled') === 'enabled');
```

### Столбцы списка пользователей

Эти два события позволяют плагину добавить собственные столбцы в список пользователей — ссылку на WhatsApp, дату окончания подписки, число входов — не трогая разметку таблицы. Контракт намеренно узкий: плагин *объявляет* столбец и *поставляет скалярные значения*, но рендерингом, макетом, обновлениями и состоянием списка владеет Admin2. У плагина нет способа внедрить разметку или рендерер.

`onApiUserListColumns` собирает дополнительные столбцы, доступные через `GET /users/columns` (по аналогии с `/users/filters`, под разрешением `api.users.read`). Добавьте описание столбца в массив `columns`:

```php
public function onApiUserListColumns(Event $event): void
{
    // У события нет offsetGet по ссылке — добавляйте через чтение-изменение-запись,
    // тем же приёмом, что в onApiUserListFilters / onApiSidebarItems.
    $columns = $event['columns'];
    $columns[] = [
        'id'        => 'my-plugin-valid-till', // обязательно, уникально
        'plugin'    => 'my-plugin',
        'label'     => 'Valid until',          // обычный текст, не ключ перевода
        'field'     => 'subscription.valid_till', // ключ в карте данных каждого пользователя
        'formatter' => 'datetime',             // один из встроенных форматтеров ниже
        'sortable'  => false,                  // необязательно; на клиенте, только текущая страница
        'priority'  => 50,                     // необязательно, большее значение сортируется раньше
        'authorize' => 'api.users.read',       // необязательно, строка или массив «любое из»
    ];
    $event['columns'] = $columns;
}
```

`formatter` проверяется по фиксированному белому списку — `text`, `link`, `date`, `datetime`, `boolean`, `number`, `badge` — и неизвестное значение откатывается к `text`. Сервер лишь называет форматтер; рендерит его клиент, поэтому по сети не передаются ни функция рендеринга, ни HTML. Значение `link` проверяется на клиенте как href (только относительный или `http(s)`; всё остальное выводится как инертный текст). Проверка `authorize` повторно выполняется на сервере, а поле удаляется из ответа.

`onApiUserListColumnData` поставляет значения для каждого пользователя. Он вызывается **один раз на ответ списка**, и его список `usernames` содержит только учётные записи, уже выбранные после поиска, фильтрации, ограничения по разрешениям и постраничной разбивки, поэтому плагин получает данные для текущей страницы, а не для всех учётных записей. Верните карту `username => [ field => value ]`:

```php
public function onApiUserListColumnData(Event $event): void
{
    $data = $event['data'];
    foreach ($event['usernames'] as $username) {   // только текущая страница
        $data[$username] = [
            'subscription.valid_till' => $this->validTill($username),
        ];
    }
    $event['data'] = $data;
}
```

Эти значения объединяются с каждым сериализованным пользователем в карту `extra` в `GET /users`, с ключами по `field` столбца, поэтому нет второго эндпоинта для вызова и нет соединения на клиенте. После объединения выживают только скаляры (и null) — массивы, объекты и слишком большие значения отбрасываются, — а событие изолировано, поэтому слушатель, выбросивший исключение, приводит к отсутствующим значениям, а не к поломке списка. Столбцы работают и с бэкендом учётных записей Flex, и с файловым хранилищем учётных записей.

### Действия в строке списка пользователей

Эти два события позволяют плагину добавить в список пользователей кнопку действия для пользователя — «Impersonate», «Send reset link», «Sync subscription», — которая выполняет серверную операцию над одной учётной записью. Они заменяют старый обходной путь с внедрением кнопки в отрендеренную таблицу через `MutationObserver`: Admin2 владеет таблицей, идентификацией строки и размещением кнопки, а плагин получает имя целевого пользователя на сервере. Модель повторяет [элементы и действия панели меню](../developer-guide/index.md#menubar-items-actions), разделяясь на событие объявления и событие выполнения.

`onApiUserListRowActions` собирает кнопки, доступные через `GET /users/row-actions` (под разрешением `api.users.read`). Добавьте описание действия в массив `actions`:

```php
public function onApiUserListRowActions(Event $event): void
{
    // У события нет offsetGet по ссылке — добавляйте через чтение-изменение-запись.
    $actions = $event['actions'];
    $actions[] = [
        'id'        => 'impersonate-user',   // обязательно, уникально — ключ выполнения
        'plugin'    => 'impersonate',
        'label'     => 'Impersonate',        // обычный текст, не ключ перевода
        'icon'      => 'fa-user-secret',     // необязательный класс Font Awesome
        'action'    => 'start',              // необязательный глагол, передаваемый обратно обработчику
        'confirm'   => 'Impersonate this user?', // необязательное подтверждение на клиенте
        'priority'  => 80,                   // необязательно, большее значение сортируется раньше
        'authorize' => ['admin.impersonate', 'admin.super'], // необязательно, строка или массив «любое из»
    ];
    $event['actions'] = $actions;
}
```

Как и столбцы, описание не содержит форматтера: `authorize` повторно проверяется на сервере и удаляется из ответа, список ограничен по длине, а по сети не передаются ни HTML, ни функция рендеринга. **`authorize` управляет только тем, какие кнопки отображаются, — это не граница безопасности.**

`onApiUserListRowAction` выполняет действие. Admin2 вызывает его через `POST /users/{username}/row-action` с телом `{ "id": "<action-id>" }`; API повторно проверяет `authorize` объявленного действия для вызывающей стороны (неизвестный или неавторизованный id даёт неотличимый `404`), а затем вызывает событие с разрешённым именем целевого пользователя. Проверяйте `$event['plugin']` — каждый слушатель получает каждый вызов, — повторно авторизуйте для конкретной цели и верните результат:

```php
public function onApiUserListRowAction(Event $event): void
{
    if (($event['plugin'] ?? '') !== 'impersonate') {
        return;
    }

    $username = (string) $event['username'];
    // Независимо перепроверьте разрешение для ЭТОЙ цели — фильтр объявления
    // был только UX. Выбросьте ForbiddenException, чтобы вернуть настоящий 403.
    $url = $this->createImpersonationUrl($username);

    $event['result'] = [
        'status'  => 'success',                 // 'success' | 'error'
        'message' => "Impersonating {$username}", // показывается во всплывающем уведомлении
        'url'     => $url,                        // необязательно — открывается в новой вкладке
    ];
}
```

Результат нормализуется до `{ status, message, url }`. **`url` проверяется на сервере, прежде чем достигнет клиента**: выживает только путь от корня (не протокол-относительная форма `//host`) или абсолютный URL того же источника — схема `javascript:`/`data:` или URL другого источника отбрасывается, поэтому поток «вернуть URL и перейти туда» не может стать открытым перенаправлением. Admin2 показывает `message` как всплывающее уведомление и открывает выживший `url` в новой вкладке с `noopener`. Обработчик, выбросивший исключение, приводит к всплывающему уведомлению об ошибке, а не к поломке списка пользователей; выброшенный `ForbiddenException` распространяется как `403`. Действия в строке требуют `api.users.read`, а вызывающая сторона, не являющаяся суперпользователем, не может выполнить его для учётной записи суперпользователя (`403`). Как и вкладки фильтров и столбцы, они работают с обоими бэкендами учётных записей.

### Метки и перевод

Метки элементов для боковой панели, панели меню, вкладок фильтров, столбцов и действий в строке могут быть либо **обычным отображаемым текстом** (`'Active'`), либо **ключом перевода** (`'PLUGIN_MYPLUGIN.ACTIVE'`). Когда метка выглядит как ключ, API разрешает её на сервере на **язык Admin Next вошедшего пользователя** и возвращает переведённую строку, поэтому плагин должен регистрировать исходный ключ и **не должен** сам вызывать `Language::translate()`. Перевод в плагине использует активный *контентный* язык сайта, что даёт смешанный язык в админке (например, русскую метку внутри английского интерфейса), когда эти два языка различаются. Регистрируйте ключ; пусть API его локализует.

## Сопоставление событий с вебхуками

Диспетчер исходящих вебхуков слушает часть перечисленных выше событий и преобразует их в события вебхуков со следующими именами:

| Событие API | Событие вебхука |
|-----------|---------------|
| `onApiPageCreated` | `page.created` |
| `onApiPageUpdated` | `page.updated` |
| `onApiPageDeleted` | `page.deleted` |
| `onApiPageMoved` | `page.moved` |
| `onApiPageTranslated` | `page.translated` |
| `onApiPagesReordered` | `pages.reordered` |
| `onApiMediaUploaded` | `media.uploaded` |
| `onApiMediaDeleted` | `media.deleted` |
| `onApiUserCreated` | `user.created` |
| `onApiUserUpdated` | `user.updated` |
| `onApiUserDeleted` | `user.deleted` |
| `onApiConfigUpdated` | `config.updated` |
| `onApiPackageInstalled` | `gpm.installed` |
| `onApiPackageRemoved` | `gpm.removed` |
| `onApiGravUpgraded` | `grav.upgraded` |

Создание подписок на вебхуки и управление ими см. в разделе [Эндпоинты вебхуков](../endpoints/webhooks/index.md). Вебхуки срабатывают *после* события API, поэтому слушатели, изменяющие состояние через события `onApi*`, увидят свои изменения в исходящем теле.

## События, совместимые с админкой (`onAdmin*`)

API также вызывает те же события, что и классический плагин Admin в Grav. Это гарантирует, что сторонние плагины, подписанные на события админки (SEO Magic, Auto Date, Mega Frontmatter и т. д.), работают корректно независимо от того, приходят ли изменения из классического интерфейса админки или из API/Admin2.

Оба семейства событий срабатывают для каждой операции — сначала события админки, затем события API.

| Событие | Когда | Данные |
|-------|------|------|
| `onAdminCreatePageFrontmatter` | Создание страницы (до сохранения) | `header`, `data` |
| `onAdminSave` | До сохранения (страница, пользователь, конфигурация) | `object` (по ссылке), `page` |
| `onAdminAfterSave` | После сохранения | `object`, `page` |
| `onAdminAfterDelete` | После удаления страницы | `object`, `page` |
| `onAdminAfterSaveAs` | После перемещения или переименования страницы | `path` |
| `onAdminAfterAddMedia` | После загрузки медиафайла | `object`, `page` |
| `onAdminAfterDelMedia` | После удаления медиафайла | `object`, `page`, `media`, `filename` |

### Пример порядка событий

Для операции создания страницы события срабатывают в таком порядке:

1. `onApiBeforePageCreate` — событие API «до»
2. `onAdminCreatePageFrontmatter` — внедрение метаданных страницы админкой
3. `onAdminSave` — сохранение в админке «до» (плагины могут изменить страницу)
4. `onAdminAfterSave` — сохранение в админке «после» (индексация, уведомления)
5. `onApiPageCreated` — событие API «после» (запускает вебхуки)

### Подписка из плагина

**Не** ставьте подписку на эти совместимые с админкой события за проверку `isAdmin()` в `onPluginsInitialized()`. Когда сохранение приходит через API (именно так Admin2 сохраняет всё), запрос не обрабатывается классическим плагином Admin, поэтому `$grav['admin']` регистрируется позже, во время диспетчеризации запроса, через `AdminProxy` плагина API. На момент `onPluginsInitialized()` он ещё не установлен, поэтому `isAdmin()` по-прежнему возвращает `false`, и любые обработчики, зарегистрированные внутри блока `isAdmin()`, никогда не подписываются. В результате плагин, чья логика сохранения и удаления молча никогда не выполняется под Admin2.

> [!NOTE] Примечание
> Распространённый приём эпохи 1.7 — регистрировать `onAdminAfterSave` только когда `isAdmin()` равно true. Для записей через API/Admin2 это больше не работает. Эти события в любом случае срабатывают только во время операций записи админки/API, поэтому подписываться на них безусловно безопасно.

Регистрируйте их напрямую в `getSubscribedEvents()` или безусловно в `onPluginsInitialized()`:

```php
public static function getSubscribedEvents(): array
{
    return [
        'onPluginsInitialized'    => ['onPluginsInitialized', 0],
        // Регистрируются всегда — они срабатывают только при записи из админки/API:
        'onAdminAfterSave'        => ['onObjectSave', 0],
        'onAdminAfterDelete'      => ['onObjectDelete', 0],
        'onAdminAfterSaveAs'      => ['onObjectMove', 0],
        'onFlexObjectAfterSave'   => ['onObjectSave', 0],
        'onFlexObjectAfterDelete' => ['onObjectDelete', 0],
    ];
}
```

Учтите, что объекты Flex (и Flex Pages) вызывают `onFlexObjectAfterSave` / `onFlexObjectAfterDelete` — варианты с `*After*`, — а не имена `onFlexObjectSave` / `onFlexObjectDelete` без приставки. Подписывайтесь на вариант, соответствующий нужным вам данным.

## Регистрация маршрутов

Плагины расширяют API пользовательскими эндпоинтами через `onApiRegisterRoutes`. Два связанных события позволяют плагину открыть маршруты для публики и добавить инструменты MCP:

| Событие | Когда | Данные |
|-------|------|------|
| `onApiRegisterRoutes` | Во время инициализации маршрутизатора | `routes` (`ApiRouteCollector`) |
| `onApiCollectPublicRoutes` | Один раз на запрос, до аутентификации | `api_base`, `prefixes` (изменяемое), `exact` (изменяемое) |
| `onApiMcpTools` | Пока `GET /mcp/tools` собирает определения инструментов MCP | `tools` (объект `McpToolCollector`; вызывайте `->add($plugin, [...])`) |

`onApiCollectPublicRoutes` позволяет плагину пометить собственные маршруты как публичные, чтобы они пропускали аутентификацию. Добавьте полные пути (включая `api_base`) в `prefixes` для совпадений по префиксу или в `exact` для точных совпадений. Запись можно ограничить одним методом, записав её как `"GET /api/v1/my-plugin/feed"`, что оставляет записи по тому же пути аутентифицированными.

Подробности см. в руководстве [Интеграция плагинов с API](../../plugins/plugin-api-integration/index.md).

## Системные события

| Событие | Когда | Данные |
|-------|------|------|
| `onApiLogCleared` | После очистки файла журнала через `DELETE /system/logs` | `file`, `bytes`, `user` |
| `onApiDemoBaselineCaptured` | После создания базового снимка демо-контента | `status` |
| `onApiDemoReset` | После сброса демо-контента к базовому снимку | `status` |

## Использование событий в плагине

```php
public static function getSubscribedEvents(): array
{
    return [
        'onApiPageCreated' => ['onPageChanged', 0],
        'onApiPageUpdated' => ['onPageChanged', 0],
        'onApiPageDeleted' => ['onPageChanged', 0],
    ];
}

public function onPageChanged(Event $event): void
{
    // Перестроить поисковый индекс, очистить кэш CDN и т. п.
    $page = $event['page'] ?? null;
    $route = $event['route'] ?? $page?->route();

    $this->rebuildIndex($route);
}
```
