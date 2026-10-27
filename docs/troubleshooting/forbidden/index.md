---
title: "403 Forbidden"
description: "Есть малозаметная, но устойчивая проблема с Apache на Windows, вызывающая ошибку 403 forbidden."
tags:
  - "Решение проблем"
  - "Ошибки HTTP"
---

# 403 Forbidden

`![](forbidden-403.png?classes=border,shadow)`

Есть малозаметная, но устойчивая проблема с **Apache** на **Windows**, вызывающая ошибку 403 forbidden.

По сути, Apache не разрешает `:` в URL, который работает в других системах, из-за соображений безопасности: пути Windows могут содержать двоеточия — `C:\some\path`.

Мы решили это, добавив настраиваемый параметр разделителя параметров, по умолчанию равный `:`

Просто отредактируйте `user/config/system.yaml` и добавьте в начало:

```yaml
param_sep: ';'
```

Так Grav будет использовать точку с запятой вместо двоеточия для параметров: `http://my-site.example/blog/tag:something` теперь станет `http://my-site.example/blog/tag;something`.

## Проблема 403 в админке

Если у вас установлен `mod_security`, были сообщения о ложном срабатывании правила 350147 (http://wiki.atomicorp.com/wiki/index.php/WAF_350147). Добавьте это правило в белый список или попросите поддержку хостинг-провайдера сделать это.

```yaml linenums="1"
ModSecurity: [file "/etc/httpd/conf/modsecurity.d/rules/tortix/modsec/50_plesk_basic_asl_rules.conf"] [line "308"] [id "350147"] [rev "143"] [msg "Protected by Atomicorp.com Basic Non-Realtime WAF Rules: Potentially Untrusted Web Content Detected"] [data ""] [severity "CRITICAL"] Access denied with code 403 (phase 2). Match of "rx ((?:submit(?:\\+| )?(request)?(?:\\+| )?>+|<<(?:\\+| )remove|(?:sign ?in|log ?(?:in|out)|next|modifier|envoyer|add|continue|weiter|account|results|select)?(?:\\+| )?>+)$|^< ?\\??(?: |\\+)?xml|^<samlp|^>> ?$)" against "ARGS:notifications" required. [hostname "mydomain"] [uri "/grav/admin/notifications.json/task:processNotifications"] [unique_id "WXoYHcpkEKz0qCI66845gQAAAAo"], referer: http://mydomain/grav/admin/tools
```
