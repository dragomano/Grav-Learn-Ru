---
description: Как управлять группами и разрешениями в Grav CMS.
---

# Группы и разрешения

!!! warning ""

	См. [ЧаВо по админке](/admin-panel/faq/#dobavlenie-i-upravlenie-polzovateliami), чтобы узнать, как управлять пользователями

## Определение групп

По умолчанию Grav не предоставляет никаких групп. Вам нужно их определить.

Группы определены в файле `user/config/groups.yaml`. Если этот файл ещё не существует, создайте его.

Вот пример определения группы пользователей:

```yaml
registered:
  icon: users
  readableName: 'Registered Users'
  description: 'The group of registered users'
  access:
    site:
      login: true
paid:
  readableName: 'Paid Members'
  description: 'The group of paid members'
  icon: money
  access:
    site:
      login: true
      paid: true
administrators:
  groupname: administrators
  readableName: Administrators
  description: 'The group of administrators'
  icon: child
  access:
    admin:
      login: true
    site:
      login: true
```

Здесь мы определяем 3 группы.

## Включение пользователя в группу

Каждого пользователя можно назначить в группу.

Просто добавьте

```yaml
groups:
  - paid
```

в файл yaml пользователя в папке `user/accounts`.

Вы можете добавить несколько групп:

```yaml
groups:
  - administrators
  - another-group
```

Вы также можете редактировать информацию о группе пользователя с помощью подключаемого модуля администратора.

## Разрешения

Пользователи, назначенные группе, наследуют разрешения группы. Например, вы можете определить группу с разрешением `site.paid`, добавив:

```yaml
access:
  site:
    paid: true
```

к определению группы в `user/config/groups.yaml`.

Когда пользователь попадает в эту группу, он наследует разрешение `site.paid: true`.

Когда пользователь принадлежит к нескольким группам, достаточно, чтобы группа предоставила разрешение, и оно будет добавлено к разрешениям пользователя.

### Тонкая настройка разрешений на уровне пользователя

Вы, как обычно, можете точно настроить разрешения на уровне пользователя. С помощью групп вы можете определить глобальное разрешение и запретить его на уровне пользователя, добавив

```yaml
access:
  site:
    paid: false
```

в файл `user.yaml`.

!!! warning ""

	См. [ЧаВо по админке](/admin-panel/faq/#upravlenie-dostupom), чтобы узнать больше о доступных уровнях доступа
