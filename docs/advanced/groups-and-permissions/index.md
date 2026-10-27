---
title: "Группы и разрешения"
description: "По умолчанию Grav не предоставляет никаких групп. Их нужно определить."
tags:
  - "Права доступа"
  - "Пользователи"
---

# Группы и разрешения

> [!CAUTION] Внимание
> См. [Grav Admin FAQ](https://learn.getgrav.org/admin-panel/faq#adding-and-managing-users), чтобы узнать, как управлять пользователями

## Определение групп

По умолчанию Grav не предоставляет никаких групп. Их нужно определить.

Группы определяются в файле user/config/groups.yaml. Если этого файла ещё нет, создайте его.

Вот пример определения групп пользователей:

```yaml linenums="1"
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

## Назначение пользователя группе

Каждого пользователя можно назначить группе.

Просто добавьте

```yaml linenums="1"
groups:
  - paid
```

в yaml-файл пользователя в `user/accounts`.

Можно добавить несколько групп:

```yaml linenums="1"
groups:
  - administrators
  - another-group
```

Информацию о группах пользователя можно также редактировать через плагин Admin.

## Разрешения

Пользователи, назначенные группе, наследуют её разрешения. Например, можно определить группу с разрешением `site.paid`, добавив:

```yaml linenums="1"
access:
  site:
    paid: true
```

в определение группы в `user/config/groups.yaml`.

Когда пользователь назначен этой группе, он унаследует разрешение site.paid: true.

Когда пользователь принадлежит нескольким группам, достаточно, чтобы разрешение предоставляла хотя бы одна группа, — оно будет добавлено к разрешениям пользователя.

### Тонкая настройка разрешений на уровне пользователя

Разрешения можно тонко настраивать и на уровне пользователя, как обычно. С группами можно определить глобальное разрешение и запретить его на уровне пользователя, добавив

```yaml linenums="1"
access:
  site:
    paid: false
```

в yaml-файл пользователя.

> [!CAUTION] Внимание
> См. [Grav Admin FAQs](https://learn.getgrav.org/16/admin-panel/faq#managing-acl), чтобы узнать больше о доступных разрешениях
