---
title: "Проблемы с прокси"
description: "Запуск команд GPM через прокси может привести к ошибке."
tags:
  - "Решение проблем"
  - "GPM"
---

# Проблемы с прокси

Запуск команд GPM через прокси может привести к ошибке.

cURL позволяет задать прокси через переменные окружения (`http_proxy` и `https_proxy`), без изменений в Grav.

См. [http://stackoverflow.com/questions/7559103/how-to-setup-curl-to-permanently-use-a-proxy](http://stackoverflow.com/questions/7559103/how-to-setup-curl-to-permanently-use-a-proxy)

Но сначала: если в вашем окружении включён `fopen`, его нужно отключить, задав `allow_url_fopen` в php.ini.

Это потому, что если `fopen` доступен, Grav автоматически использует его вместо `curl`.
