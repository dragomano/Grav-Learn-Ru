---
title: "Cloudflare Manager"
description: "Начните работу с Cloudflare Manager за несколько минут."
tags:
  - "Grav Premium"
  - "Cloudflare"
  - "Производительность"
  - "Grav 1.7"
---

# Cloudflare Manager

> [!IMPORTANT] Важно
> Для премиальных продуктов требуется бесплатный плагин [License Manager](../license-manager/index.md). Установите его и добавьте лицензию продукта перед установкой этого продукта.

## Установка

Сначала убедитесь, что у вас установлена последняя версия **Grav 1.7** (`-f` принудительно обновляет индекс GPM).

```
$ bin/gpm selfupgrade -f
```

У Cloudflare Manager нет зависимостей, поэтому установка сводится к выполнению команды:

```
$ bin/gpm install cloudflare
```

Вы также можете установить плагин через раздел **Plugins** в плагине admin.

## Конфигурация

У Cloudflare Manager совсем немного параметров конфигурации: нужны только `Zone ID` и `API Token`.

`![Конфигурация](configuration.png?class=image-shadow)`

* **Plugin status** → Включает или отключает весь плагин.

* **API Token** → Токен Cloudflare, имеющий право выполнять действия через API. Чтобы получить его, воспользуйтесь руководством [Cloudflare: создание API-токена](#cloudflare-create-api-token) ниже.

* **Zone ID** → Zone ID, связанный с доменом, которым вы хотите управлять. Чтобы узнать его, воспользуйтесь руководством [Cloudflare: получение Zone ID](#cloudflare-retrieve-zone-id) ниже.

## Cloudflare: создание API-токена {: #cloudflare-create-api-token }
Чтобы Cloudflare Manager мог взаимодействовать с Cloudflare, ему нужен API-токен. Его легко создать на сайте Cloudflare:

1. Сначала перейдите в [раздел API Tokens своего профиля](https://dash.cloudflare.com/profile/api-tokens?target=_blank&rel=noopener%20noreferrer) и войдите в систему
2. В разделе **API Tokens** нажмите кнопку `Create Token`
3. Прокрутите до раздела **Create Custom Token** и нажмите `Get Started`

Теперь заполним Custom Token:

1. **Token name**: Cloudflare Manager (acme.com)
2. **Permissions**: нам понадобятся 5 разрешений уровня Zone:

     | Тип | Эндпоинт             | Разрешения |
     |------|----------------------|-------------|
     | Zone | Zone Settings        | Edit        |
     | Zone | SSL and Certificates  | Edit        |
     | Zone | Cache Purge          | Purge       |
     | Zone | DNS                  | Edit        |
     | Zone | Analytics            | Read        |
3. **Zone Resources** (необязательно): этот параметр необязателен, но _настоятельно_ рекомендуется. Он позволяет ограничить взаимодействие API-токена только выбранным доменом и ничем больше.

     | Действие | Zone                 | Домен   |
     |--------|----------------------|----------|
     | Zone   | Specific Zone         | acme.com |
4. Нажмите **Continue to summary**
5. Проверьте сводку и нажмите **Create Token**
6. Теперь вы попадёте на страницу созданного токена. Нажмите кнопку _Copy_, чтобы скопировать ID в буфер обмена, и вставьте его в поле API Token в Cloudflare Manager.
   !!!! После создания токен восстановить нельзя. Если вы перезагрузите или покинете страницу, процесс придётся начать заново. Обязательно скопируйте сгенерированный ID в буфер обмена и сохраните его в Cloudflare Manager. Для дополнительной надёжности сохраните его ещё и в надёжном месте.

`![Настройки API-токена](api-token.png?class=image-shadow,w-80per)`

## Cloudflare: получение Zone ID {: #cloudflare-retrieve-zone-id }
`Zone ID` — это идентификатор, созданный Cloudflare и связанный с вашим доменным именем. Узнать его очень просто.

1. Сначала перейдите в [панель управления Cloudflare](https://dash.cloudflare.com/?target=_blank&rel=noopener%20noreferrer), войдите и выберите домен, которым хотите управлять (например, **acme.com**).
2. После выбора вы окажетесь на странице Overview с графиками аналитики; прокрутите вниз, и на правой боковой панели вы увидите **API** -> **Zone ID**
3. Нажмите ссылку _Click to copy_, чтобы скопировать ID в буфер обмена, и вставьте его в поле Zone ID в Cloudflare Manager.
