---
title: "Шаблон отчёта"
published: false
date: 17-06-2017
---

Краткие сведения, включая идентификатор, полное название проекта, дату, уровень риска. Например:

- ID: CORE-2017-001
- Project: Grav Core
- Date: 17-06-2017
- Risk-level: Less Critical

Сводка. Например. «REST-API публично доступен без аутентификации».

Заголовок "PROJECT_FULL: LEVEL, TARGET, ID", например: `Grav Core: Less Critical, Access Bypass, CORE-2017-001`, где ID "ID: PROJECT_SHORT-YEAR-INT(3)", например: `CORE-2017-001`. ID должен быть увеличен после предыдущего отчёта в рамках проекта, года. Дата должна быть в формате [Grav-parsable format](/content/headers#date).

### Рекомендация

Что должны делать пользователи, чтобы решить проблему или предотвратить уязвимость. Например. «Обновление Grav Core до версии 1.2.2», «Измените настройки PHP на `safe_mode = On` »,« Убедитесь, что конфигурация сервера предотвращает доступ через HTTP/1.1».

### Описание

Что раскрывает уязвимость, почему она небезопасна и как её использовать. Например: «Открытые запросы к api могут изменять страницу с запросом POST или PATCH без аутентификации».

### Затронутые версии

Затронуты определенные версии Grav Core, включая обратные SemVer-диапазоны, такие как `1.1.x`, и текущие SemVer-диапазоны, такие как `1.2.x`, а также определенные версии, такие как `1.2.2`. Например. "Ядро `v1.1.x`, обнаруженное в `v1.1.5`".

### Решение

Как уязвимость была исправлена ​​в коде, с конкретными ссылками на файл в теге и какая конкретная версия впоследствии решила её, например, `1.2.2`. Например: «Исправлено в [Grav/Common/Page/Page](https://github.com/getgrav/grav/blob/1.1.4/system/src/Grav/Common/Page/Page.php#L94-L97), строки 94-97 в версии 1.1.4».

### Атрибуция

Кто сообщил о проблеме, кто решил проблему, кто координировал усилия по её решению. Например:

Сообщил [John Doe](https://en.wikipedia.org/wiki/John_Doe), исправили [Foo](https://en.wikipedia.org/wiki/Placeholder_name#Computing), [Bar](https://en.wikipedia.org/wiki/Placeholder_name#Computing), [Alice](https://en.wikipedia.org/wiki/Placeholder_name#Computing) и [Bob](https://en.wikipedia.org/wiki/Placeholder_name#Computing). Согласовано [основной группой](https://getgrav.org/about).
