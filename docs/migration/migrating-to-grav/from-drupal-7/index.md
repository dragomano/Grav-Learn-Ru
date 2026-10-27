---
title: "Переход с Drupal 7"
description: "Команды drush gravexportusers или drush graveu создадут файлы учётных записей Grav."
tags:
  - "Миграция"
  - "Drupal"
---

# Переход с Drupal 7 на Grav

## Требования

* PHP v7.1 или выше для зависимостей composer.
* Drush
* Работающий сайт Drupal 7, из которого будет экспортирован контент.
* Доступ на чтение и запись к `public://`, а также к папке пользовательских модулей сайта Drupal (обычно `sites/all/modules/contrib`).

## Установка

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/I6UVFUqZMOU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1. Скачайте плагин [grav_export](https://www.drupal.org/project/grav_export/) и переместите его в папку `sites/all/modules/contrib` вашего Drupal.
2. Выполните `composer install` в папке `grav_export`, чтобы установить зависимости.
3. Включите модуль grav_export через `drush en grav_export -y` или графический интерфейс администрирования.
4. Выполните `drush grav_export_all` или его псевдоним `drush gravea`, чтобы экспортировать все элементы. Другие варианты см. ниже.
5. Экспортированные файлы находятся в `[DRUPAL_ROOT]/sites/default/files/grav_export/EXPORT`
6. Для просмотра и управления пользователями рекомендуется плагин Grav [https://github.com/david-szabo97/grav-plugin-admin-addon-user-manager](https://github.com/david-szabo97/grav-plugin-admin-addon-user-manager).
7. Выполните шаги ниже, чтобы импортировать данные в Grav.

---

## Экспорт пользователей из Drupal

### Команда

`drush grav_export_users` или `drush graveu` создадут файлы учётных записей Grav.

### Результаты

* Учётные записи в папке экспорта `EXPORT/accounts/`.
  * Имена пользователей дополняются до минимума в 3 символа и максимума в 16.
  * Если имя пользователя усечено или дополнено, к нему также добавляется uid Drupal, чтобы избежать конфликтов.
  * Пароли в каждой учётной записи генерируются случайно и не связаны с соответствующей учётной записью Drupal. При первой аутентификации пароль автоматически заменяется на `hashed_password`.

### Импорт пользователей в Grav

Скопируйте папку `EXPORT/accounts` в вашу папку `user` (например, файлы `username.yaml` должны оказаться в `user/accounts`).

## Экспорт ролей пользователей из Drupal

### Команда {: #command-1 }

`drush grav_export_roles` или `drush graver` создадут файл `groups.yaml` Grav.

### Результаты {: #results-1 }

Роли пользователей Drupal экспортируются как группы Grav в файл `groups.yaml` по пути `config/groups.yaml`. Несколько замечаний об экспорте ролей:

* Каждая роль Drupal преобразуется в группу Grav `drupal_<ROLE_WITH_UNDERSCORES>` (например, `authenticated user` становится `drupal_authenticated_user`).
* Группа `drupal_administrator` получает доступ `admin.super` вместе с доступом `admin.login`.
* Группа `drupal_authenticated_user` получает доступ `admin.login`.
* Все учётные записи получают группу «drupal_authenticated_user».
* Пользователи Drupal с ролью администратора получают группу `drupal_administrator`.

### Импорт ролей пользователей

Скопируйте папку `EXPORT/config` в `users/config`.

## Экспорт типов контента из Drupal

### Команда {: #command-2 }

`drush grav_export_content_types` или `drush gravect` создадут схемы Grav и файлы `html.twig`.

### Результаты {: #results-2 }

Для каждого определённого типа поля `drush gravect` попытается создать совместимую схему и соответствующий файл `html.twig` для каждого типа контента Drupal. Файлы будут экспортированы в `EXPORT/themes/drupal_export/blueprints` и `EXPORT/themes/drupal_export/templates` соответственно.

### Известные ограничения

1. Поле «number_integer»

Кардинальность многих полей Grav поддерживает только одно значение. Экспортируется только первая запись Drupal.

2. Поле «addressfield»

В Grav нет понятия поля адреса. Данные поля Drupal экспортируются как тип формы `array`.

### Импорт типов контента в Grav

Скопируйте папки `EXPORT/themes/drupal_export/blueprints` и `EXPORT/themes/drupal_export/templates` в активную тему Grav. Плагин админки теперь должен предоставлять дополнительные параметры для каждого типа контента и связанных полей.

Примечание. Хотя содержимое полей добавляется в заголовки страниц Grav, отображение этих полей **не** экспортируется из Drupal. Файл `html.twig` нужно будет изменить, чтобы показывать дополнительные поля (помимо основного содержимого).

### Экспорт узлов из Drupal

### Команда {: #command-3 }

`drush grav_export_nodes` или `drush graven` создадут пользователей и группы Grav.

### Результаты {: #results-3 }

* Каждый узел экспортируется в собственную структуру папок в `EXPORT/pages` на основе URL-псевдонима и типа контента Drupal (то есть `pages/content/my_first_page/page.yaml` или `pages/content/cool_article/article.yaml`).
* Данные полей Drupal хранятся в заголовке страницы.
* Файлы хранятся в `EXPORT/data/files/` по модели хранения, похожей на Drupal.
* В выводе drush могут появляться дополнительные термины таксономии. Скопируйте их в файл `user/config/site.yaml` Grav под ключом таксономии.

### Импорт узлов в страницы Grav

Скопируйте папки `EXPORT/data` и `EXPORT/pages` в папку `user` в Grav.
