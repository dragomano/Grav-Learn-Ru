---
title: Проблемы с прокси
metadata:
    description: 'Описание ошибки, связанной с выполнением команд GPM через прокси, в Grav CMS.'
taxonomy:
    category: docs
---

Выполнение команд GPM через прокси может привести к ошибке.

cURL позволяет вам установить прокси в качестве переменной окружения (`http_proxy` и `https_proxy`) без изменений, необходимых в Grav.

См. [https://stackoverflow.com/questions/7559103/how-to-setup-curl-to-permanently-use-a-proxy](https://stackoverflow.com/questions/7559103/how-to-setup-curl-to-permanently-use-a-proxy)

Но сначала, если в вашей среде включен `fopen`, вам нужно отключить его, отключив `allow_url_fopen` через php.ini.

Это потому, что если доступен `fopen`, Grav автоматически использует его поверх `curl`.