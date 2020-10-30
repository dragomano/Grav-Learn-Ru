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

Резюме. Например: "REST-API is publicly exposed without authentication required."

Заголовок "PROJECT_FULL: LEVEL, TARGET, ID", например: `Grav Core: Less Critical, Access Bypass, CORE-2017-001`, где ID "ID: PROJECT_SHORT-YEAR-INT(3)", например: `CORE-2017-001`. ID должен быть увеличен после предыдущего отчета в рамках проекта, года. Дата должна быть в формате [Grav-parsable](https://learn.getgrav.org/content/headers#date).

### Рекомендация

Что должны делать пользователи, чтобы решить проблему или предотвратить уязвимость. Например: "Update to v1.2.2 of Grav Core", "Change PHP settings to `safe_mode = On`", "Ensure Server-configuration prevents access via HTTP/1.1".

### Описание

Что раскрывает уязвимость, почему она небезопасна и как её использовать. Например: "Public requests to /api could modify a page with a `POST` or `PATCH` request, without being authenticated."

### Затронутые версии

Затронуты определенные версии Grav Core, включая обратные SemVer-диапазоны, такие как `1.1.x`, и текущие SemVer-диапазоны, такие как `1.2.x`, а также определенные версии, такие как `1.2.2`. Например: "Core `v1.1.x`, discovered in `v1.1.5`."

### Решение

Как уязвимость была исправлена ​​в коде, с конкретными ссылками на файл в теге и какая конкретная версия впоследствии решила её, например, `1.2.2`. Например: "Fixed in [Grav/Common/Page/Page](https://github.com/getgrav/grav/blob/1.1.4/system/src/Grav/Common/Page/Page.php#L94-L97), lines 94-97 at v1.1.4."

### Атрибуция

Кто сообщил о проблеме, кто решил проблему, кто координировал усилия по её решению. Например:

Сообщил [John Doe](https://en.wikipedia.org/wiki/John_Doe), исправил [Foo](https://en.wikipedia.org/wiki/Placeholder_name#Computing), [Bar](https://en.wikipedia.org/wiki/Placeholder_name#Computing), [Alice](https://en.wikipedia.org/wiki/Placeholder_name#Computing), and [Bob](https://en.wikipedia.org/wiki/Placeholder_name#Computing). Координировала [команда ядра](https://getgrav.org/about).
