# Разрешения

> Проблема, связанная с отсутствием прав на запись на файлы и папки, при использовании Grav CMS.

В зависимости от вашей среды хостинга разрешения могут быть или не быть проблемой, которую вам нужно решить. Важно понимать, что существует потенциальная проблема, если пользователь, которого вы используете для редактирования файлов в файловой системе, отличается от пользователя, под которым работает PHP (обычно это веб-сервер), или, по крайней мере, два пользователя у вас нет доступа на **чтение/запись** к этим файлам.

Сначала выясните, с каким пользователем работает Apache или Nginx, выполнив следующую команду
Для Apache:

    ps aux | grep -v root | grep apache | cut -d\  -f1 | sort | uniq

Для Nginx:

    ps aux | grep -v root | grep nginx | cut -d\  -f1 | sort | uniq

И узнайте, какому пользователю принадлежит файл в вашем каталоге grav, запустив

    ls -l


Поскольку Grav является файловой CMS, ей необходимо писать в файловую систему для создания файлов кэша и журналов. Есть три основных сценария:

1. ##### PHP/веб-сервер работает с тем же пользователем, который редактирует файлы (рекомендуется)
   Этот подход используется в большинстве **виртуальных хостингов**, а также хорошо подходит для локальной разработки. В сообщении в блоге, которое мы написали относительно [MacOS Yosemite, Apache и PHP](https://getgrav.org/blog/mac-os-x-apache-setup-multiple-php-versions), описывается, как настроить Apache для работы как ваша личная учетная запись пользователя. Этот подход не считается достаточно безопасным для использования на выделенном веб-хосте, поэтому следует использовать второй или третий вариант.

2. ##### PHP/веб-сервер работает с разными учетными записями, но с той же группой
   Используя общую группу между вашим пользователем и учетной записью PHP/веб-сервера с разрешениями `775` и `664`, вы гарантируете, что даже если у вас есть две разные учетные записи, обе будут иметь доступ на **чтение/запись** к файлам. Вам также, вероятно, следует установить в корне параметр `umask 0002`, чтобы новые файлы создавались с соответствующими разрешениями.

3. ##### Разные учетные записи, исправлять разрешения вручную
   Последний подход состоит в том, чтобы иметь совершенно разные учетные записи и просто обновлять права собственности и разрешения файлов после редактирования, чтобы гарантировать, что пользователь PHP/веб-сервера может **читать/писать** соответствующим образом.

Для этого можно использовать простой **сценарий оболочки с исправлением разрешений**:

```bash
#!/bin/sh
chown -R joeblow:staff .
find . -type f -exec chmod 664 {} \;
find ./bin -type f -exec chmod 775 {} \;
find . -type d -exec chmod 775 {} \;
find . -type d -exec chmod +s {} \;
```

Вы можете использовать этот файл и при необходимости отредактировать его для соответствующего пользователя и группы, которые подходят для вашей установки. В основном этот сценарий делает следующее:

1. Изменяет текущий каталог, все файлы и подпапку на владение `joeblow` и `staff`
2. Находит все файлы из текущего каталога ниже и устанавливает права доступа `664`, так что они будут `RW` для пользователя и группы и `R` для других.
3. Находит все папки из текущего каталога ниже и устанавливает права доступа `775`, так что они являются `RWX` для пользователя и группы и `RX` для других.
4. Устанавливает **право собственности** для всех каталогов, чтобы гарантировать сохранение изменений пользователей и групп.

### Разрешения папки кэша изображений

Если файлы изображений в папке кэша записаны с неправильными разрешениями, попробуйте установить их в файле `user/config/system.yaml`,

```yaml
images:
  cache_perms: '0775'
```

если свойство `images` уже присутствует, просто добавьте в его конец `cache_perms: 0775`.

Если это по-прежнему не работает, создайте файл `setup.php` в корневой папке Grav (тот, который содержит `index.php`), и добавьте туда

```php
<?php
umask(0002);
```

Если у вас уже есть файл `setup.php`, просто добавьте эту строку в начало. Этот файл обычно используется для настройки нескольких сайтов, но, будучи вызванным при каждом вызове Grav, вы также можете использовать его для других целей.

### Совместный хостинг с сайтом WordPress

В общем, Grav можно установить в папку корневого уровня существующего сайта WordPress, и две CMS будут прекрасно сосуществовать. (Не забудьте установить Base Rewrite в htaccess папки Grav.) Если вы сталкиваетесь с ошибками разрешений с файлами кэша при доступе к администратору и/или просмотру страниц Grav, проверьте, установлен ли WP-Engine для этого сайта WordPress. Если это так, вам нужно будет связаться с их службой поддержки, чтобы создать исключение для папки Grav из их агрессивной службы распределенного кэша.

### Совет для SELinux

Если приведенные выше предложения по-прежнему не работают, запустите

`chcon -Rv system_u:object_r:httpd_sys_rw_content_t:s0 ./` в корневую папку Grav.

Ссылки:
- [https://unix.stackexchange.com/questions/337704/selinux-is-preventing-nginx-from-writing-via-php-fpm](https://unix.stackexchange.com/questions/337704/selinux-is-preventing-nginx-from-writing-via-php-fpm)
- [https://github.com/getgrav/grav/issues/912#issuecomment-227627196](https://github.com/getgrav/grav/issues/912#issuecomment-227627196)
- [https://stopdisablingselinux.com](https://stopdisablingselinux.com/)
- [https://stackoverflow.com/questions/28786047/failed-to-open-stream-on-file-put-contents-in-php-on-centos-7](https://stackoverflow.com/questions/28786047/failed-to-open-stream-on-file-put-contents-in-php-on-centos-7)
- [https://www.serverlab.ca/tutorials/linux/web-servers-linux/configuring-selinux-policies-for-apache-web-servers/](https://www.serverlab.ca/tutorials/linux/web-servers-linux/configuring-selinux-policies-for-apache-web-servers/)