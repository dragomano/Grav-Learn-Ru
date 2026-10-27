---
title: "Серверная часть"
description: "Защита установки Grav на стороне сервера заключается в разумных настройках сервера и PHP. Это руководство не описывает настройки сервера, на котором работает Grav, и не идеальные условия, а скорее содержит советы и рекомендации по защите Grav или ссылки на материалы о защите сервера."
tags:
  - "Безопасность"
  - "PHP"
---

# Серверная часть

Защита установки Grav на стороне сервера заключается в разумных настройках сервера и PHP. Это руководство не описывает настройки сервера, на котором работает Grav, и не идеальные условия, а скорее содержит советы и рекомендации по защите Grav или ссылки на материалы о защите сервера. **Это актуально для боевого сервера, а не для локальной разработки, и не рекомендуется новичкам!**

## Grav и конфигурация по умолчанию

Для Grav всегда используйте актуальную конфигурацию для конкретного сервера. Такие конфигурации есть в [репозитории GitHub](https://github.com/getgrav/grav/tree/develop/webserver-configs). Кроме того, периодически обновляйте установку Grav, так как новые исправления безопасности появляются в новых версиях; подробности — в [CHANGELOG](https://github.com/getgrav/grav/blob/develop/CHANGELOG.md).

## Конфигурация PHP

Прежде чем вмешиваться в конфигурацию PHP, учтите, что большинство хостинг-провайдеров, у которых вы арендуете место, скорее всего, уже задали разумные и безопасные настройки по умолчанию. Кроме того, в большинстве случаев они не разрешают редактировать её самостоятельно. Прежде чем отключать или менять какие-либо настройки, ознакомьтесь с [требованиями Grav, включая расширения PHP](https://github.com/getgrav/grav/blob/develop/composer.json), и с тем, как изменения на них повлияют.

Обычно конфигурация PHP меняется через `php.ini`. Расположение этого файла можно узнать из командной строки командой `php --ini`, а если прямого доступа к командам нет, создайте в публичной корневой папке веб-сервера файл `phpinfo.php` с содержимым `<?php phpinfo(); ?>` и откройте его в браузере. Путь будет указан в разделе «Loaded Configuration File». Найдя файл, удалите `phpinfo.php`.

Несколько общих рекомендаций:

- **Всегда поддерживайте PHP в актуальной версии**: используйте [поддерживаемую версию](https://php.net/supported-versions.php) PHP, желательно ту, что активно и стабильно развивается. Например, в PHP 5.6 и PHP 7.0 исправления безопасности будут выходить только до декабря 2018 года, тогда как PHP 7.1 продолжает активно развиваться вместе с PHP 7.2.
- Подумайте об отключении публичного показа ошибок и версии PHP: [статья на PHP.earth](https://docs.php.earth/security/intro/#php-configuration).
- Используйте отдельного пользователя с ограниченными правами для выполнения PHP для Grav: [права доступа в документации](https://learn.getgrav.org/troubleshooting/permissions).
- Используйте Suhosin для [расширенной защиты PHP](https://suhosin.org/stories/feature-list.html).

## Конфигурация веб-сервера

Распространённое ПО веб-серверов, или HTTP-серверов, включает Nginx и Apache, а также более современные альтернативы, такие как LiteSpeed или CaddyServer. Упомянутые [конфигурации веб-серверов](https://github.com/getgrav/grav/tree/develop/webserver-configs) содержат необходимые настройки по умолчанию для Grav, но веб-сервер можно дополнительно защитить через его конфигурацию. Полезные материалы:

- [How To Secure Nginx](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-on-ubuntu-14-04) на DigitalOcean и [Nginx WebServer Best Security Practices](https://www.cyberciti.biz/tips/linux-unix-bsd-nginx-webserver-security.html) на nixCraft.
- [Apache Web Server Hardening & Security Guide](https://geekflare.com/apache-web-server-hardening-security/) на Geek Flare и [Apache Web Server Security and Hardening Tips](https://www.tecmint.com/apache-security-tips/) на Tecmint.
- [Ways of improving security in Litespeed](https://bobcares.com/blog/ways-of-improving-security-in-litespeed/) на Bobcares.

## Конфигурация сервера

**Всегда поддерживайте операционную систему (ОС) в актуальном состоянии**. ОС уязвимы для эксплойтов и вторжений, даже больше, чем PHP, и обновлять их следует как можно чаще. Также **всегда поддерживайте в актуальном состоянии другое ПО**: ваша установка — это не только ОС, PHP и Grav. Другие программные пакеты тоже являются факторами риска, и их следует часто обновлять.

Чтобы защитить соединение пользователей с сайтом, включите и требуйте [HTTPS с SSL-сертификатом](https://docs.php.earth/security/ssl/). Это гарантирует, что всё общение между сервером и браузером остаётся приватным и зашифрованным. Бесплатные сертификаты и услуги доступны, например, через [Let's Encrypt](https://letsencrypt.org/about/) или [CloudFlare](https://www.cloudflare.com/ssl/).

Если сервер работает на Linux, включите [Security Enhanced Linux](https://selinuxproject.org/page/Main_Page). SELinux обычно включён по умолчанию, и [овчинка стоит выделки](http://www.computerworld.com/article/2717423/security/why-selinux-is-more-work--but-well-worth-the-trouble.html). Ещё несколько рекомендаций для системных администраторов есть на [nixCraft](https://www.cyberciti.biz/tips/php-security-best-practices-tutorial.html).
