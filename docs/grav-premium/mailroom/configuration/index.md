---
title: "Конфигурация"
description: "Каждая настройка Mailroom по вкладкам, с её названием, значением по умолчанию и описанием."
tags:
  - "Mailroom"
  - "Почта"
  - "Конфигурация"
---

# Конфигурация {: #configuration }

Эта страница перечисляет каждую настройку Mailroom, сгруппированную по вкладке, на которой она находится. Настройки лежат в `user/config/plugins/mailroom.yaml` и редактируются на вкладке **Settings** страницы Mailroom в Admin Next (кнопка **Settings** над страницей и кнопка **Configure** плагина открывают её обе). Значения по умолчанию поставляются в собственном `mailroom.yaml` плагина.

Некоторые вещи вообще не настройки: списки, теги, сегменты, шаблоны, кампании, автоматизации и исключения создаются на собственных экранах Mailroom и хранятся в его базе данных.

## Карта вкладок {: #tab-map }

| Вкладка | Что управляет |
|---|---|
| Sending | Может ли почта уходить, как быстро, от кого она приходит и три вида сообщений |
| Email Design | Логотип, цвета, текст подвала и почтовый адрес макета письма |
| Tracking | Открытия, переходы, сколько хранится детализация и разметка ссылок |
| Signup | Значение двойного подтверждения по умолчанию, предложение согласия, файлы импорта |
| Public Pages | Адрес сайта, базовый маршрут, шаблон страницы, страница подтверждения, ограничения частоты и доверенные прокси |
| Segments | Как часто пересчитываются сегменты |
| Automations | Работа автоматизаций и сколько людей двигается за запуск |
| Health | Переопределения проверок доставляемости и предупреждение о проценте возвратов |
| Providers | Карточки webhook провайдеров |
| Advanced | База данных и обработчик |

`enabled` — переключатель состояния плагина над вкладками.

## Sending {: #sending }

См. [Отправку и провайдеров](../sending-and-providers/index.md). Вкладка открывается карточкой **Can this site send mail?** и имеет карточку **The queue worker** под скоростью отправки.

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `sending.per_minute` | Messages Per Minute | `60` | Наибольшее число сообщений, которое сайт отправляет за минуту, по всем кампаниям. |
| `sending.batch` | Recipients Per Run | `50` | Сколько людей один запуск задания отправки берёт на себя, прежде чем вернуть управление обработчику. |
| `sending.abort_after_failures` | Stop After Consecutive Failures | `25` | Отклонённых отправок подряд до того, как кампания остановит себя. Один успех сбрасывает счётчик. |
| `from_name` | From Name | пусто | Имя, от которого отправляются кампании. Пусто использует имя плагина Email. Кампания может использовать другое, заданное в её редакторе, как и список через API. |
| `from_email` | From Address | пусто | Адрес, с которого отправляются кампании. Пусто использует адрес From плагина Email. |
| `messages.confirmation` | Confirmation Emails | Включено | Отправлять письма двойного подтверждения подписки. Выключено — никто новый не сможет подтвердить. |
| `messages.campaign` | Campaigns | Включено | Отправлять кампании. |
| `messages.flow_email` | Automation Emails | Включено | Отправлять письма, которые отправляют автоматизации. |

## Email Design {: #email-design }

См. [Шаблоны и брендирование](../templates-and-branding/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `branding.logo` | Logo | пусто | URL изображения или путь на этом сайте. Пусто печатает название сайта. |
| `branding.logo_width` | Logo Width (pixels) | `160` | Какой ширины рисуется логотип (от 16 до 600). |
| `branding.accent` | Accent Color | пусто | Ссылки, кнопка подтверждения и линия под шапкой, а также кнопки и ссылки на публичных страницах и в блоке подписки. Пусто — бирюзовый в письме и акцент вашей темы на страницах, где он у неё есть. |
| `branding.background` | Background Color | `#f4f5f7` | На чём лежит карточка. |
| `branding.text` | Text Color | `#222222` | Цвет сообщения. |
| `branding.footer_text` | Footer Text | пусто | Строка или две под каждым сообщением, брендированным или простым. Обычный текст. |
| `branding.postal_address` | Postal Address | пусто | Печатается внизу каждого сообщения, брендированного или простого, и в его текстовой части. |

## Tracking {: #tracking }

См. [Отчёты и отслеживание](../reports-and-tracking/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `tracking.opens` | Track Opens | Включено | Добавлять пиксель открытия в кампании. Новая кампания начинает с этого и может сказать иначе. |
| `tracking.clicks` | Track Clicks | Включено | Отправлять ссылки кампании через редирект перехода. Новая кампания начинает с этого и может сказать иначе. |
| `tracking.retain_days` | Keep Detail For (days) | `365` | Сколько хранятся отдельные переходы и события провайдера. Итоги остаются. |
| `utm.enabled` | Tag Campaign Links | Включено | Добавлять параметры UTM к ссылкам обратно на ваш сайт. |
| `utm.source` | Source | `newsletter` | `utm_source`. |
| `utm.medium` | Medium | `email` | `utm_medium`. |
| `utm.campaign` | Campaign | пусто | `utm_campaign`. Пусто использует имя каждой кампании. |
| `utm.content` | Content | пусто | `utm_content`. |

## Signup {: #signup }

См. [Двойное подтверждение подписки и согласие](../double-opt-in-and-consent/index.md) и [Импорт и экспорт](../import-and-export/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `double_opt_in` | Ask People to Confirm | Включено | Значение по умолчанию для нового списка. У каждого списка своё. |
| `consent_text` | Consent Sentence | "I agree to receive email from this site. I can leave with one click at any time." | Формулировка рядом с флажком. Её хеш хранится у каждого согласившегося или хеш собственного предложения блока подписки, когда он его показывает. Пусто не просит согласия. |
| `import.keep_days` | Keep Uploaded Files For (days) | `30` | Сколько загруженный список контактов остаётся на диске, если его импорт так и не запускался. `0` хранит их. |

## Public Pages {: #public-pages }

См. [Списки и формы подписки](../lists-and-signup-forms/index.md#the-public-pages) и [API подписки](../subscribe-api/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `site_url` | Site URL | пусто | Адрес, с которого начинается каждая ссылка в письме, например `https://www.example.com`. Пусто использует Custom Base URL Grav и заполняется при первом визите администратора, когда не задано ни то, ни другое. |
| `route` | Route Base | `/newsletter` | Где отвечают публичные страницы. Задайте до отправки первой кампании. |
| `pages.base_template` | Page Template | пусто | Шаблон темы, который расширяют публичные страницы. Пусто использует `partials/base.html.twig`. |
| `pages.builtin_css` | Mailroom Stylesheet | Включено | Небольшой файл стилей Mailroom на каждой странице сайта для публичных страниц и блока подписки где угодно. Каждое правило в нём ограничено собственными классами Mailroom. |
| `confirm.on_get` | Confirm on Opening the Link | Выключено | Подтверждать при открытии ссылки. Оставьте выключенным. |
| `confirm.auto_submit` | Press the Button for Them | Включено | Страница подтверждения нажимает свою кнопку скриптом. |
| `rate_limits.subscribe` | Signups Per Hour | `10` | Подписок с одного адреса посетителя в час. `0` выключает ограничение. |
| `rate_limits.token` | Link Pages Per Hour | `60` | Страницы подтверждения, отписки, предпочтений и веб-просмотра на адрес посетителя. |
| `rate_limits.tracking` | Opens and Clicks Per Hour | `600` | Пиксель и редирект перехода на адрес посетителя. Держите высоким. |
| `rate_limits.webhook` | Webhook Requests Per Hour | `6000` | Запросы webhook провайдера с одного адреса. Держите высоким. |
| `security.trusted_proxies` | Trusted Proxies | `[]` | Адреса или диапазоны CIDR, чьему `X-Forwarded-For` верят. |

## Segments {: #segments }

См. [Теги и сегменты](../tags-and-segments/index.md#counts).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `segments.count_every_minutes` | Recount Segments Every | `15` | Минут между пересчётами. `0` пересчитывает при каждом запуске обработчика. |

## Automations {: #automations }

См. [Автоматизации](../automations/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `automations.enabled` | Run Automations | Включено | Выключено удерживает все автоматизации сразу. Никого не выводят. |
| `automations.per_tick` | People Moved Per Run | `200` | Сколько людей один запуск автоматизаций двигает на шаг (от 1 до 1000). |

## Health {: #health }

См. [Доставляемость](../deliverability/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `deliverability.sending_domain` | Sending Domain | пусто | Домен, на котором живут ваши записи DNS, когда он не тот, что в адресе From. |
| `deliverability.dkim_selector` | DKIM Selector | пусто | Селектор DKIM, когда транспорт не может его назвать. Два через запятую во время ротации. |
| `deliverability.return_path` | Custom Return Path | пусто | Ваш собственный хост return-path, когда проверка SPF продолжает падать на домене, который, как вы знаете, настроен. |
| `alerts.bounce_rate_percent` | Bounce Rate That Raises an Alert | `5` | Баннер на панели управления, когда больше этого процента почты за последние тридцать дней вернулось. |

## Providers {: #providers }

На вкладке только карточки провайдеров (см. [Отправку и провайдеров](../sending-and-providers/index.md#provider-webhooks)). Их кнопки пишут под `providers.<name>` в `mailroom.yaml`:

| Ключ | Что это |
|---|---|
| `providers.<name>.secret` | Случайный секрет пути в URL webhook этого провайдера, записывается **Generate a secret**. Никогда не вводится вручную. |
| `providers.<name>.webhook_id`, `webhook_events`, `set_up_at`, `secret_at`, `webhook_dead_at` | Что зарегистрировал **Set up in** и когда, чтобы карточка могла это сказать. |
| `providers.smtp2go.auth_header` | Только YAML. Полное значение заголовка `Authorization`, когда вы добавляете его в webhook в панели SMTP2GO. |

Ключи подписи провайдеров живут в настройках каждого транспортного плагина, не здесь.

## Advanced {: #advanced }

См. [Установку](../installation/index.md#the-database) и [Задания и cron](../jobs-and-cron/index.md).

| Ключ | Название | По умолчанию | Что делает |
|---|---|---|---|
| `database.type` | Database | `sqlite` | `sqlite`, `mysql` (MySQL или MariaDB) или `pgsql` (PostgreSQL). Смена направляет на другую, пустую базу данных. |
| `database.auto_migrate` | Update the Tables | `sqlite` | `sqlite` (само на SQLite, иначе из командной строки), `auto` (само в любой базе данных) или `manual` (только из командной строки). |
| `database.sqlite.path` | SQLite File | пусто | Пусто оставляет его в `user-data://mailroom/mailroom.sqlite`. Потоковый или абсолютный путь. |
| `database.mysql.host`, `.port`, `.unix_socket`, `.dbname`, `.username`, `.password` | Host, Port, Socket, Database Name, Username, Password | `localhost`, `3306` | MySQL или MariaDB. Заданный сокет используется вместо хоста и порта. |
| `database.pgsql.host`, `.port`, `.dbname`, `.username`, `.password`, `.sslmode` | Host, Port, Database Name, Username, Password, SSL Mode | `localhost`, `5432` | PostgreSQL. SSL Mode — значение сервера по умолчанию, `disable`, `allow`, `prefer`, `require`, `verify-ca` или `verify-full`. |
| `worker.scheduler` | Run From Grav's Scheduler | Включено | Запускать обработчик из планировщика Grav каждую минуту. |
| `worker.inline_drain` | Send Right After the Request | Включено | Выполнять задания, поставленные запросом, после его ответа. |
| `worker.inline_budget_seconds` | Seconds After a Request | `3` | Сколько может длиться эта разгрузка (от 1 до 30). |
| `worker.job_timeout_seconds` | Longest One Job May Run (seconds) | `120` | Задание дольше этого останавливается и пробуется снова. `0` выключает ограничение. |
| `worker.retain_days` | Keep Finished Jobs For (days) | `30` | Завершённые и неудачные задания убираются через столько дней. |
| `mail_log.retain_days` | Keep the Message Log For (days) | `180` | Строки журнала сообщений убираются через столько дней. `0` хранит их. |

## Связанные разделы {: #related }

- [Установка](../installation/index.md)
- [CLI](../cli/index.md)
- [Устранение неполадок](../troubleshooting/index.md)
