---
title: 'Переезд с Drupal 7 на Grav'
metadata:
    description: 'Как мигрировать с Drupal 7 на Grav.'
taxonomy:
    category:
        - docs
page-toc:
    active: true
---

## Требования

* PHP v7.1 или выше для зависимостей Composer.
* Drush
* Рабочий сайт Drupal 7, с которого будет экспортироваться контент.
* Доступ на чтение/запись к `public://` а также к папке пользовательских модулей на сайте Drupal (обычно `sites/all/modules/contrib`).

## Установка

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/I6UVFUqZMOU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1. Скачайте плагин [grav_export](https://www.drupal.org/project/grav_export/) и переместите в папку `sites/all/modules/contrib`.
2. Запустите `composer install` в папке `grav_export` для установки зависимостей.
3. Включите модуль grav_export командой `drush en grav_export -y` или через плагин админки.
4. Запустите `drush grav_export_all`, или его псевдоним `drush gravea`, для экспорта всех элементов. Смотрите другие варианты ниже.
5. Экспортированные файлы находятся по адресу `[DRUPAL_ROOT]/sites/default/files/grav_export/EXPORT`
6. Плагин [admin-addon-user-manager](https://github.com/david-szabo97/grav-plugin-admin-addon-user-manager) рекомендуется для просмотра и управления пользователями.
7. Выполните следующие действия для импорта данных в Grav.

===

## Экспорт пользователей Drupal

### Команда

`drush grav_export_users` или `drush graveu` будет генерировать файлы учетных записей пользователей Grav.

### Результаты

* Учетные записи пользователей в папке экспорт в разделе `EXPORT/accounts/`.
  * Имена пользователей будут дополнены минимум 3 символами, максимум 16.
  * Если имя пользователя усечено или дополнено, то оно также будет иметь uid Drupal, чтобы избежать коллизий.
  * Пароли в каждой учетной записи генерируются случайным образом и не имеют никакой связи с соответствующей учетной записью Drupal. Пароль автоматически переключается на hashed_password, как только учетная запись аутентифицируется в первый раз.

### Импорт пользователей в Grav

Скопируйте папку `EXPORT/accounts` в директорию `user` (например, файлы username.yaml должны быть размещены в директорию `user/accounts`).

## Экспорт ролей пользователей из Drupal

### Команда

`drush grav_export_roles` или `drush graver` создаст файл Grav groups.yaml.

### Результаты

Роли пользователей Drupal экспортируются как группы Grav в файл `groups.yaml` по адресу `config/groups.yaml`. Некоторые примечания об экспорте ролей:

* Каждая роль Drupal преобразуется в группу Grav `drupal_<ROLE_WITH_UNDERSCORES>` (например, `authenticated user` становится `drupal_authenticated_user`).
* Группа `drupal_administrator` получает доступ `admin.super` вместе с доступом `admin.login`.
* Группа `drupal_authenticated_user` получает доступ к `admin.login`.
* Все учетные записи получают группу `drupal_authenticated_user`.
* Пользователи Drupal с ролями администратора получают группу `drupal_administrator`.

### Импорт ролей пользователей

Скопируйте папку `EXPORT/config` в `users/config`.

## Экспорт типов контента из Drupal

### Команда

`drush grav_export_content_types` или `drush gravect` генерирует чертежи Grav и файлы html.twig.

### Результаты

Для каждого определенного типа поля drush gravect будет пытаться создать совместимый чертеж и соответствующий файл html.twig для каждого типа содержимого Drupal. Файлы будут экспортированы в `EXPORT/themes/drupal_export/blueprints` и `EXPORT/themes/drupal_export/templates` соответственно.

### Известные ограничения

1. Поле "number_integer"

Количество элементов во многих полях Grav поддерживает только одно значение. Экспортируется только первая запись Drupal.

2. Поле "addressfield"

У Grav нет понятия адресного поля. Данные поля Drupal экспортируются как форма типа `array`.

### Импорт типов контента в Grav

Скопируйте папки `EXPORT/themes/drupal_export/blueprints` и `EXPORT/themes/drupal_export/templates` в активную тему в Grav. Плагин админки теперь должен предоставлять дополнительные параметры для каждого типа контента и связанных полей.

Примечание. Хотя содержимое поля добавляется в заголовки страниц Grav, отображение этих полей **не** экспортируется из Drupal. Файл html.twig необходимо изменить, чтобы отобразить любые дополнительные поля (помимо основного содержимого).

### Экспорт узлов из Drupal

### Команда

`drush grav_export_nodes` или `drush graven` создаст пользователей и группы Grav.

### Результаты

* Каждый узел экспортируется в свою собственную структуру папок в `EXPORT/pages` на основе псевдонима url-адреса Drupal и типа содержимого (например, `pages/content/my_first_page/page.yaml` или `pages/content/cool_article/article.yaml` ).
* Данные поля Drupal хранятся в заголовке страницы.
* Файлы хранятся в `EXPORT/data/files/`, следуя модели хранения, подобной Drupal.
* В выводе drush могут выводиться дополнительные термины таксономии. Скопируйте их в файл Grav `user/config/site.yaml` под ключом таксономии.

### Импорт узлов в страницы Grav

Скопируйте папки `EXPORT/data` и `EXPORT/pages` в каталог `user` в Grav.
