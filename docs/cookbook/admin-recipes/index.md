---
title: "Рецепты для админки"
description: "Задачи и решения по настройке интерфейса Admin2 — одностраничного приложения на SvelteKit, которое общается с вашим сайтом через REST API."
tags:
  - "Рецепты"
  - "Разработка"
  - "Admin Next"
---

# Рецепты для админки

На этой странице собраны задачи и решения по настройке интерфейса Admin2. Admin2 — одностраничное приложение на SvelteKit, которое общается с вашим сайтом через [REST API](../../api/index.md); плагины расширяют его, подписываясь на события `onApi*` и при необходимости поставляя небольшие веб-компоненты. Полный справочник, на который опираются эти рецепты, см. в [руководстве разработчика API](../../api/developer-guide/index.md).

> [!NOTE] Примечание
> Для этих рецептов нужен плагин API `1.0.0-rc.16` / Admin2 `2.0.0-rc.16` или новее.

## Добавление своей кнопки создания страницы

<h4 id="problem">Проблема:</h4>

Вам нужен простой способ создавать новую запись блога в нужной папке нажатием кнопки в админке, а не проходить каждый раз общий сценарий **Add Page**. В примере мы будем использовать запись блога.

<h4 id="solution">Решение:</h4>

В устаревшем интерфейсе admin-classic это делалось блоком `add_modals` в `user/config/plugins/admin.yaml`, который указывал на пользовательскую схему. **Admin2 не использует этот механизм.** Кнопки панели инструментов добавляет плагин через событие `onApiMenubarItems`, и кнопка может либо вести глубокой ссылкой на штатную форму новой страницы, либо открывать собственное модальное окно. Выберите то, что больше подходит.

##### Вариант A — глубокая ссылка на штатную форму новой страницы (проще всего) {: #option-a-deep-link-the-native-new-page-form-simple }

Элемент панели меню может нести клиентское намерение `route`, которое открывает в Admin2 собственный экран **Add Page** с заранее заполненными родительской страницей и шаблоном. Выбор шаблона при этом **блокируется**, поэтому кнопка всегда создаёт страницу нужного типа. Ни схемы, ни веб-компонента, ни обработчика сохранения — вы бесплатно получаете слаг, валидацию и многоязычность штатной формы.

В вашем плагине:

```php
public static function getSubscribedEvents(): array
{
    return [
        'onApiMenubarItems' => ['onApiMenubarItems', 0],
    ];
}

public function onApiMenubarItems(\RocketTheme\Toolbox\Event\Event $event): void
{
    $items = $event['items'] ?? [];
    $items[] = [
        'id'     => 'new-post',
        'plugin' => 'my-plugin',          // слаг вашего плагина
        'label'  => 'New Post',           // всплывающая подсказка кнопки
        'icon'   => 'fa-plus',            // иконка Font Awesome
        // Глубокая ссылка на форму Add Page. /pages/new принимает три необязательных параметра:
        //   parent   — предварительно выбирает родительскую страницу
        //   template — предварительно выбирает И блокирует выбор шаблона
        //   title    — заполняет заголовок (и подсказывает слаг)
        'route'  => '/pages/new?parent=/blog&template=post&title=New%20Post',
    ];
    $event['items'] = $items;
}
```

Этот единственный обработчик даёт вам кнопку **New Post** на верхней панели инструментов, которая открывает форму Add Page с родительской страницей `/blog`, заблокированным шаблоном `post` и заполненным заголовком.

##### Вариант B — собственное модальное окно

Когда нужны поля, которых нет в штатной форме, откройте модальное окно. Самый быстрый путь не требует веб-компонента: определите поля прямо в вызове, и Admin2 построит форму за вас. Из веб-компонента плагина (например, [плавающего виджета](../../api/developer-guide/index.md#floating-widgets) или [пользовательского поля](../../api/developer-guide/index.md#custom-admin-fields-via-web-components)):

```javascript
const values = await window.__GRAV_DIALOGS.form({
  title: 'New Post',
  fields: [
    { name: 'title',   label: 'Post Title', required: true },
    { name: 'summary', label: 'Summary', type: 'textarea' },
  ],
  submitLabel: 'Create',
});
if (values) {
  // Создаём страницу через REST API.
  await fetch(`${window.__GRAV_API_SERVER_URL}${window.__GRAV_API_PREFIX || '/api/v1'}/pages`, {
    method: 'POST',
    headers: { 'X-API-Token': window.__GRAV_API_TOKEN, 'Content-Type': 'application/json' },
    body: JSON.stringify({
      route: '/blog/' + values.title.toLowerCase().replace(/\s+/g, '-'),
      title: values.title,
      template: 'post',
      content: values.summary,
    }),
  });
}
```

Для более богатого интерфейса (несколько шагов, живые данные, свой макет) поставьте веб-компонент модального окна в `admin-next/modals/{id}.js` и подключите его прямо к кнопке панели меню:

```php
$items[] = [
    'id'     => 'new-post',
    'plugin' => 'my-plugin',
    'label'  => 'New Post',
    'icon'   => 'fa-plus',
    'modal'  => [
        'component' => 'new-post',   // → admin-next/modals/new-post.js
        'title'     => 'New Post',
        'size'      => 'lg',         // sm | md | lg | xl
    ],
];
```

Компонент возвращает результат, испуская событие `resolve` (или `cancel`/`close`, чтобы закрыть окно). Полный контракт см. в разделе [Модальные окна и наложения](../../api/developer-guide/index.md#modals-and-overlays).

##### Ограничение того, что создаётся

Как бы ни пришёл запрос на создание, вы можете принудительно задать шаблон и внедрить метаданные по умолчанию на стороне сервера событием `onApiBeforePageCreate`, чтобы запись всегда получала нужный тип и заголовки независимо от клиента:

```php
public function onApiBeforePageCreate(\RocketTheme\Toolbox\Event\Event $event): void
{
    if (str_starts_with((string) $event['route'], '/blog/')) {
        $event['template'] = 'post';
        $header = $event['header'] ?? [];
        $header['date'] = date('Y-m-d H:i');
        $event['header'] = $header;
    }
}
```

> [!NOTE] Примечание
> В отличие от рецепта с `add_modals` для admin-classic, который можно было добавить на любой сайт через `user/config/plugins/admin.yaml`, кнопка панели инструментов Admin2 регистрируется из плагина. Если у вас его ещё нет, достаточно крошечного плагина из одного файла с обработчиком `onApiMenubarItems` из примера выше.
