---
description: Настройка сервера Apache. Конфигурация сайта Grav CMS.
---

# Apache

*Apache* — самый распространённый HTTP-сервер и сервер по умолчанию на большинстве хостингов с общим доступом. Grav изначально рассчитан на работу с Apache: каждый релиз включает файл `.htaccess` в корневом каталоге сайта, который отвечает за перенаправление URL и блокирует прямой доступ к закрытым каталогам. Поэтому в большинстве случаев никакой дополнительной настройки не требуется — достаточно убедиться, что Apache имеет право читать этот файл.

## Требования

Для работы Grav с Apache необходимы:

* `apache2` с включённым модулем `mod_rewrite` (`sudo a2enmod rewrite`)
* `php`, доступный для Apache либо через `mod_php`, либо (рекомендуется) через `php-fpm` с использованием `mod_proxy_fcgi`

## Включение поддержки `.htaccess`

Apache читает поставляемый с Grav файл `.htaccess` только в том случае, если для корневого каталога сайта параметр `AllowOverride` имеет значение `All`. Это самая распространённая причина, по которой правила перенаправления и безопасности Grav не применяются. В конфигурации виртуального хоста (или в соответствующем блоке `<Directory>`) укажите:

```apache
<Directory /var/www/grav>
    AllowOverride All
    Require all granted
</Directory>
```

Затем перезагрузите Apache:

```bash
sudo systemctl reload apache2   # или: sudo apachectl graceful
```

Если вы не можете включить `AllowOverride` (например, на хостинге с жёсткими ограничениями), вместо этого скопируйте содержимое поставляемого файла `.htaccess` в конфигурацию вашего виртуального хоста, поместив его в соответствующий блок `<Directory>`.

## Установка в подкаталог и `RewriteBase`

Если Grav установлен в корне домена, правила по умолчанию работают без изменений. Если же Grav установлен в подкаталоге и при открытии внутренних страниц возникают ошибки 500 или 404, раскомментируйте директиву `RewriteBase` в начале файла `.htaccess` и укажите путь к вашему подкаталогу:

```apache
RewriteBase /your_sub_folder
```

## Работа за балансировщиком нагрузки или прокси

В средах с хостингом или балансировкой нагрузки, где SSL завершается на промежуточном узле, раскомментируйте блок `X-Forwarded-Proto` в файле `.htaccess`, чтобы Grav распознавал соединение как защищённое:

```apache
RewriteCond %{HTTP:X-Forwarded-Proto} https
RewriteRule .* - [E=HTTPS:on]
```

## Встроенные правила безопасности

Поставляемый файл `.htaccess` является основным источником этих правил, поэтому вам не потребуется поддерживать собственную их копию. Для справки: блок Security запрещает прямой доступ к закрытым каталогам Grav и к файлам сценариев в каталогах `system` и `user`, при этом сохраняя возможность напрямую обслуживать общедоступные медиафайлы, загруженные в `user/data` (например, изображения, добавленные через Flex-объекты):

```apache
## Begin - Security
# Block all direct access for these folders
RewriteRule ^(\.git|cache|bin|logs|backup|webserver-configs|tests)/(.*) error [F,NC]
# Block all direct access to these sensitive user folders, whatever the file type
RewriteRule ^(user)/(accounts|config|env)/(.*) error [F,NC]
# Block user/data too, but allow public asset uploads (e.g. Flex Object images)
# to be served directly. SVG stays blocked as a stored-XSS vector; .css/.js are
# served per project policy despite the same risk on this user-writable folder.
RewriteCond %{REQUEST_URI} !\.(jpe?g|png|gif|webp|avif|bmp|ico|mp4|webm|ogg|ogv|mov|mp3|wav|m4a|flac|pdf|woff2|woff|ttf|otf|eot|css|js)$ [NC]
RewriteRule ^(user)/data/(.*) error [F,NC]
# Block access to specific file types for these system folders
RewriteRule ^(system|vendor)/(.*)\.(txt|xml|md|html|htm|shtml|shtm|json|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ error [F,NC]
# Block access to specific file types for these user folders
RewriteRule ^(user)/(.*)\.(txt|md|json|yaml|yml|php|php2|php3|php4|php5|phar|phtml|pl|py|cgi|twig|sh|bat)$ error [F,NC]
# Block all direct access to .md files:
RewriteRule \.md$ error [F,NC]
# Block all direct access to files and folders beginning with a dot
RewriteRule (^|/)\.(?!well-known) - [F]
# Block access to specific files in the root folder
RewriteRule ^(LICENSE\.txt|composer\.lock|composer\.json|\.htaccess)$ error [F,NC]
## End - Security
```

!!! note
    Если вы заменили поставляемый файл `.htaccess` или существенно изменили его, сравните его с актуальной версией из [репозитория Grav](https://github.com/getgrav/grav/blob/develop/.htaccess) и убедитесь, что блок Security присутствует и соответствует текущей версии. На странице [Раскрытие папки `user`](/security/user-folder-exposure) объясняется предупреждение, которое отображается в панели администратора, когда эти правила не применяются.

## Проверка работы

Запрос к закрытому файлу, например `https://www.example.com/user/config/system.yaml`, должен возвращать ответ **403 Forbidden**, а не содержимое файла. При этом запрос к медиафайлу, загруженному в каталог `user/data` (например, файлу `.jpg`), должен по-прежнему успешно обрабатываться.

!!! tip
    LiteSpeed читает `.htaccess` и полностью совместим с правилами Apache, используемыми Grav, поэтому применяется та же конфигурация. Достаточно убедиться, что для виртуального хоста включены правила перенаправления.
