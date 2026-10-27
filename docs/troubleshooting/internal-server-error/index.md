---
title: "500 Internal Server Error"
description: "Эта ошибка может быть вызвана неверной конфигурацией сервера, проблемами с .htaccess или mod_security."
tags:
  - "Решение проблем"
  - "Ошибки HTTP"
  - "PHP"
---

# 500 Internal Server Error

> The server encountered an internal error or misconfiguration and was unable to complete your request.
>
> Please contact the server administrator at webmaster@localhost to inform them of the time this error occurred, and the actions you performed just before this error.
>
> More information about this error may be available in the server error log.
> <cite>Apache/2.4.7 Server at localhost Port 80</cite>

Эта ошибка может быть вызвана следующим:

- неверная конфигурация сервера (httpd.conf)
- проблемы с .htaccess
- mod_security и подобные

### Проверка работы PHP

Первое, что нужно сделать, — убедиться, что PHP на сервере работает правильно и Grav не является прямой причиной проблемы. Для проверки создайте в корне папки Grav временный файл `info.php` (после проверки удалите его ради безопасности!). В файле должен быть такой PHP-код:

```php linenums="1"
<?php phpinfo();
```

Затем откройте в браузере `http://my-site.example/your_grav_directory/info.php`. Вы должны увидеть страницу отчёта со всей информацией о конфигурации PHP, включая версию и загруженные расширения.

### Проверка прав доступа

Ошибку 500 могут вызывать неверные права доступа. См. [руководство по правам доступа](../permissions/index.md).

### Проблема с Register Globals

У некоторых пользователей, недавно обновившихся до PHP 5.5 с версии 5.4 или 5.3, в файле `php.ini` могут остаться устаревшие настройки. Одна из них, способная вызвать **500 Internal Server Error**, — параметр `register_globals`. Просто удалите или закомментируйте строку:

```apache linenums="1"
register_global = On
```

Затем перезапустите сервер Apache.

### ThreadStackSize в Windows

Если сервер работает под Windows, ошибка 500 Internal Server Error может возникать из-за того, что **ThreadStackSize** слишком мал. Просто добавьте этот код в конец файла `httpd.conf`:

```apache linenums="1"
<IfModule mpm_winnt_module>
  ThreadStackSize 8388608
</IfModule>
```

Затем перезапустите сервер Apache.

### Options -Indexes

Grav использует параметр `-Indexes`, чтобы запретить листинг папок. Некоторые хостинги не любят, когда Apache `.htaccess` управляет настройкой `Options`.

Мы видели сообщения, что пользователям в такой ситуации помогает простое комментирование этой строки в файле `.htaccess` Grav:

```apache linenums="1"
# Запрещаем просмотр файлов
#Options -Indexes
```

### Проблемы с RewriteBase

Были сообщения о 500 Internal Server Error без указания RewriteBase на хостинге 1&1 (но это может касаться и других). Попробуйте изменить

```apache linenums="1"
# RewriteBase /
```

на

```apache linenums="1"
RewriteBase /
```

(Источник: [http://ahcox.com/webdev/1and1-internal-server-error-grav/](http://ahcox.com/webdev/1and1-internal-server-error-grav/))

### Навигация в админке

При навигации по админке Grav в левом верхнем углу появляется сообщение **Internal Server Error**. Это из-за неверных прав доступа к папке /cache.

 ![Внутренняя ошибка сервера](http://i.imgur.com/vyPfoZ7.png)

Если эта ошибка появляется, скорее всего, вы не задали правильные права доступа к папке /cache: её нужно сделать рекурсивно доступной для записи, а не просто доступной для записи. Команда ниже, выполненная из папки Grav, должна решить проблему.

```bash
sudo chmod 755 cache/ -R
```
