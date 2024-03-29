---
description: Описание ошибки 403 Forbidden в Grav CMS. Причины возникновения.
---

# Доступ запрещён — 403

![](forbidden-403.png)

Существует неясная, но постоянная проблема с **Apache** на **Windows** , вызывающая ошибку «403 forbidden».

В принципе, Apache не допускает `:` в URL-адресе, который работает на других системах, из-за проблемы безопасности, основанной на том факте, что пути windows могут иметь двоеточия в них: `C:\some\path`.

Мы решили эту проблему, предоставив настраиваемый параметр для разделителя параметров, который по умолчанию равен `:`

Просто отредактируйте свой `user/config/system.yaml` и добавьте это вверху:

```yaml
param_sep: ';'
```

Это настроит Grav на использование точки с запятой, а не двоеточия. Например, вместо `http://yoursite.com/blog/tag:something` теперь будет `http://yoursite.com/blog/tag;something`.

## Ошибка 403 в админке

Если у вас установлен `mod_security`, у нас были сообщения о правиле [350147](https://wiki.atomicorp.com/wiki/index.php/WAF_350147), вызывающем ложное срабатывание. Добавьте это правило в белый список или попросите поддержку у своего хостинг-провайдера.

```yaml
ModSecurity: [file "/etc/httpd/conf/modsecurity.d/rules/tortix/modsec/50_plesk_basic_asl_rules.conf"] [line "308"] [id "350147"] [rev "143"] [msg "Protected by Atomicorp.com Basic Non-Realtime WAF Rules: Potentially Untrusted Web Content Detected"] [data ""] [severity "CRITICAL"] Access denied with code 403 (phase 2). Match of "rx ((?:submit(?:\\+| )?(request)?(?:\\+| )?>+|<<(?:\\+| )remove|(?:sign ?in|log ?(?:in|out)|next|modifier|envoyer|add|continue|weiter|account|results|select)?(?:\\+| )?>+)$|^< ?\\??(?: |\\+)?xml|^<samlp|^>> ?$)" against "ARGS:notifications" required. [hostname "mydomain"] [uri "/grav/admin/notifications.json/task:processNotifications"] [unique_id "WXoYHcpkEKz0qCI66845gQAAAAo"], referer: http://mydomain/grav/admin/tools
```
