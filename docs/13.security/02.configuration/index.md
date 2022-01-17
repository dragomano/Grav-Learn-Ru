# Рекомендуемая конфигурация

> Как и в случае с любым другим приложением, важно проверить параметры конфигурации, чтобы обезопасить и оптимизировать свой сайт.

## Сайт в продакшене

Важно обеспечить безопасность производственного сайта путем усиления конфигурации. Для этого мы рекомендуем вам установить основную конфигурацию в `user/config/`, содержащую настройки по умолчанию, которые вы хотите использовать в производственной среде, и переопределить эти настройки в среде сайта разработки, например, `user/env/localhost` или `user/env/site.local`. Вы также можете переопределить настройки производственного сайта через среды, например, если вы используете многосайтовую установку на нескольких доменах.

### Системная конфигурация (`user/config/system.yaml`)

```yaml
force_ssl: true       # Use HTTPS only (redirect from HTTP -> HTTPS)

cache:
  enabled: true       # Greatly speeds up the site
  check:
    method: hash      # Optimization, disables file modification checks for pages

twig:
  cache: true         # Greatly speeds up the site
  debug: false        # We do not want to display debug messages
  auto_reload: false  # Optimization, disables file modification checks for twig files
  autoescape: true    # Protects from many XSS attacks, but requires twig updates if used in older sites/themes/plugins

errors:
  display: 0          # Display only a simple error
  log: true           # Log errors for later inspection

debugger:
  enabled: false      # Never keep debugger enabled in a live site.
  censored: true      # In case if you happen to enable debugger, avoid displaying sensitive information

session:
  enabled: true       # NOTE: Disable sessions if you do not use user login and/or forms.
  secure: true        # Use this as your site should be using HTTPS only
  httponly: true      # Protects session cookies against client side scripts and XSS
  samesite: Strict    # Prevent all cross-site scripting attacks
  split: true         # Separate admin session from the site session for added security

strict_mode:          # Test your site before changing these. Removes backward compatibility and improves site security.
  yaml_compat: false
  twig_compat: false
  blueprint_compat: false
```

## Сайт в разработке

Для сервера разработки есть несколько настроек, которые мы должны изменить, чтобы было удобнее обновлять сайт.

### Системная конфигурация (`user/env/localhost/config/system.yaml`)

> **СОВЕТ:** Замените localhost на имя вашего локального сервера.

```yaml
force_ssl: false      # If the development site doesn't use SSL

cache:
  enabled: true       # Still keep cache enabled
  check:
    method: file      # Allow updating pages without clearing cache

twig:
  cache: true         # Still keep cache enabled
  debug: true         # We want to display debug messages
  auto_reload: true   # We may be editing twig files

errors:
  display: 1          # Display full backtrace if there are errors

debugger:
  enabled: true       # Debugger is handy to have
  censored: false     # We may want to see secure content in debugger

session:
  secure: false       # If the development site doesn't use SSL
  httponly: false     # If the development site doesn't use SSL

strict_mode:          # These settings help you to keep your site updated to use the latest standards
  yaml_compat: false
  twig_compat: false
  blueprint_compat: false
```
