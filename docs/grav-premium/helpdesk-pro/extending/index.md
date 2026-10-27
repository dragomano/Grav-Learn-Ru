---
title: "Расширение"
description: "Для разработчиков: события Grav, доменные события, сервисы и реестры, на которых дополнения строят Helpdesk Pro."
tags:
  - "Helpdesk Pro"
  - "Разработка"
  - "PHP"
---

# Расширение {: #extending }

Дополнения расширяют Helpdesk Pro через события Grav, доменные события и несколько реестров. Эта страница для PHP-разработчиков, создающих плагин поверх Helpdesk Pro. Собственные пользовательские поля, организации, SLA и оценки Helpdesk Pro построены на этих же точках расширения, поэтому они служат рабочими примерами в `classes/` плагина.

Таблицы дополнений называются `helpdesk_<addon>_*` и лежат в том же файле SQLite. Никогда не пишите в таблицы `helpdesk_*` напрямую: журнал активности, события и проверки доступа этого не увидят.

## События с первого взгляда {: #events-at-a-glance }

| Событие Grav | Когда срабатывает | Для чего использовать |
|---|---|---|
| `onHelpdeskRegisterMigrations` | Раз за запрос, до сборки контейнера | Добавить папку ваших миграций |
| `onHelpdeskRegisterJobs` | При сборке реестра обработчиков заданий | Зарегистрировать обработчики заданий |
| `onHelpdeskRegisterSinks` | При сборке цепочки приёмников событий | Получать каждое доменное событие в процессе |
| `onHelpdeskWorkerTick` | Раз за проход фонового процесса | Запланировать повторяющуюся работу |
| `onHelpdeskEvent` | Раз за доменное событие, после фиксации транзакции | Реагировать на любое изменение |
| `onHelpdeskIntake` | До создания заявки | Обогатить или отклонить новую заявку |
| `onHelpdeskTicketSerialize` | Для каждой заявки, которую возвращает API | Добавить данные в `extra` |
| `onHelpdeskMailBuild` | Для каждого письма до его отрисовки | Изменить переменные или заголовки письма |
| `onHelpdeskRegisterPersonData` | При объединении, удалении или сводке человека | Добраться до собственных столбцов человека |
| `onHelpdeskRulesRegister` | При первой необходимости правил в запросе | Добавить триггеры, условия и действия |
| `onHelpdeskNotificationChannels` | При первой необходимости типов каналов | Добавить тип канала |
| `onHelpdeskReports` | При каждой сборке отчёта | Добавить карточки отчёта |

## События запуска {: #start-up-events }

### `onHelpdeskRegisterMigrations`

Срабатывает с `['paths' => []]`. Добавьте папку ваших миграций. Папки обрабатываются после собственных папок плагина в порядке добавления; отсутствующая папка записывается в журнал и пропускается. Файлы миграций следуют формату grav-db-kit: `NNNN_snake_name.php`, возвращающий анонимный класс, реализующий `Migration` набора, с идемпотентными шагами. Ссылайтесь на классы набора через префиксованные имена Helpdesk Pro (`Grav\Plugin\HelpdeskPro\Vendor\TrilbyMedia\GravDbKit\…`). `bin/plugin helpdesk-pro migrate` применяет их.

### `onHelpdeskRegisterJobs`

Срабатывает с `['registry' => JobHandlerRegistry]`. Регистрируйте обработчики через `$event['registry']->register('myaddon.sync', $handler)`. Типы совпадают с `^[a-z0-9][a-z0-9._-]*$`; собственные типы Helpdesk Pro регистрируются первыми, и их нельзя перехватить. Обработчик реализует `JobHandler` набора (`handle(array $payload): void`), бросает исключение, чтобы потерпеть неудачу и повториться, и должен быть идемпотентным.

### `onHelpdeskRegisterSinks`

Срабатывает с `['sinks' => []]`. Добавьте объекты, реализующие `EventSink` набора (`emit(string $event, array $payload): void`), чтобы получать каждое доменное событие после фиксации его транзакции. Приёмник, бросивший исключение, записывается в журнал, а остальные продолжают работать.

### `onHelpdeskWorkerTick`

Срабатывает раз за проход `bin/plugin helpdesk-pro work`, после собственной уборки Helpdesk Pro, с `['queue' => JobQueue, 'now' => int]`. Место для планирования повторяющейся работы дополнения.

## Доменные события {: #domain-events }

Каждое изменение, которое делает Helpdesk Pro, объявляется доменным событием после сохранения. Индексация поиска, уведомления, обновления в реальном времени и дополнения слушают один и тот же поток.

### `onHelpdeskEvent`

Событие Grav, срабатывающее раз за доменное событие после фиксации транзакции базы данных, с `['name' => 'ticket.created', 'payload' => [...]]`:

```php
public static function getSubscribedEvents(): array
{
    return ['onHelpdeskEvent' => ['onHelpdeskEvent', 0]];
}

public function onHelpdeskEvent(Event $event): void
{
    if ($event['name'] === 'ticket.status_changed' && $event['payload']['to_category'] === 'resolved') {
        // ...
    }
}
```

Слушатель, бросивший исключение, записывается в журнал и никогда не отменяет изменение.

### Полезные нагрузки {: #payloads }

Каждая полезная нагрузка несёт:

| Поле | Значение |
|---|---|
| `ticket_id` | Заявка (null для событий не об одной заявке) |
| `project_id` | Проект заявки после изменения |
| `actor_id` | Человек, сделавший изменение; системный человек Helpdesk (id 1) для автоматизации; null для гостя |
| `visibility` | `public` или `internal`. Событие `internal` никогда не должно доходить до клиента, и дополнения обязаны это соблюдать. |
| `at` | Когда, в секундах epoch |

События, описывающие строку активности, также несут её `activity_id`. Полезные нагрузки — плоские id и значения, никогда тела сообщений: получайте содержимое через сервисы, которые проверяют доступ. Новые поля могут добавляться; поля никогда не переименовываются в пределах основной версии.

| Событие | Дополнительные поля | Видимость |
|---|---|---|
| `ticket.created` | `requester_id`, `assignee_id`, `created_by_id`, `channel`, `triage_state`, `kind`, `priority`, `first_message_id`, `mentioned_person_ids`, `activity_id`, `extra` | public |
| `ticket.updated` | `changes`: поле → `[before, after]` для `subject`, `priority`, `kind`, `visibility_scope`, `labels` | internal |
| `ticket.status_changed` | `from_status_id`, `to_status_id`, `from_category`, `to_category`, `activity_id` | public при входе в `resolved` или `closed` и выходе из них; иначе internal |
| `ticket.assigned` | `from_assignee_id`, `to_assignee_id`, `activity_id` | internal |
| `ticket.moved` | `from_project_id`, `to_project_id`, `activity_id` | internal |
| `ticket.triaged` | `decision` (`accept`, `duplicate`, `decline`), `duplicate_of_id`, `notify`, `reason`, `followed_original`, `reason_public`, `requester_id`, `decision_id`, `activity_id` | public для отказа и дубликата, internal для принятия |
| `ticket.merged` | `into_ticket_id`, `into_project_id`, `moved_messages`, `widened`, `activity_id` | public |
| `ticket.deleted` | `permanent`, `activity_id` | internal |
| `ticket.restored` | `activity_id` | internal |
| `ticket.rated` | `rating_id`, `round`, `rating`, `previous_rating`, `staff_id`, `has_comment`, `comment_changed`, `source` (`email` или `portal`), `activity_id`, когда оценка изменилась | internal |
| `ticket.fields_changed` | `changes` (ключ → `[before, after]`), `field_keys`, `activity_id`. Публикуется раз для видимых клиенту полей и раз для внутренних. | public для публичных и readonly полей; иначе internal |
| `ticket.sla_warning` | `target`, `due_at`, `policy_id`, `assignee_id`, `late`, `retroactive`, `via` (`sla`) | internal |
| `ticket.sla_breached` | То же; `late` равно true для нарушения, найденного после того, как заявку ответили или решили | internal |
| `ticket.person_merged` | `from_person_id`, `into_person_id`, `activity_id`; раз на заявку | internal |
| `ticket.person_erased` | `person_id`, `mode`, `activity_id`; раз на заявку | internal |
| `ticket.organization_changed` | `from_organization_id`, `to_organization_id`, `activity_id` | internal |
| `message.created` | `message_id`, `author_id`, `author_kind`, `source`, `mentioned_person_ids`, `status_after_category` | Собственная видимость сообщения |
| `message.updated` | `message_id`, `mentioned_person_ids` (только новые упоминания), `activity_id` | Собственная видимость сообщения |
| `message.deleted` | `message_id`, `activity_id` | Собственная видимость сообщения |
| `participant.added` | `person_id`, `role` (`requester`, `cc`, `watcher`), `reason` | internal |
| `participant.removed` | `person_id`, `role`, `reason` | internal |
| `attachment.added` | `attachment_id`, `message_id` | Собственная видимость сообщения |
| `organization.created` | `organization_id`, `name`, `domains`, `share_tickets` | internal |
| `organization.updated` | `organization_id`, `name`, `changes`, `domains`, `share_tickets` | internal |
| `organization.deleted` | `organization_id`, `name` | internal |
| `organization.member_added` | `organization_id`, `person_id`, `via` (`staff` или `domain`), `from_organization_id` | internal |
| `organization.member_removed` | `organization_id`, `person_id`, `via` | internal |
| `kb.feedback` | `page_key`, `helpful`, `session_hash` (без заявки) | internal |
| `inbound.rejected` | `inbound_id`, `reason` (без заявки) | internal |

Первое сообщение заявки — часть `ticket.created`; отдельного `message.created` для него нет. Когда одно действие меняет несколько вещей, каждая получает своё событие по порядку: ответ сотрудника, который также задаёт статус, публикует `message.created`, а затем `ticket.status_changed`.

- Каждое событие одного массового редактирования несёт `bulk_id` и `bulk_count`.
- Каждое событие, вызванное действиями правила, несёт `via: rules` и `rule_id` (0 для автоматического закрытия). Правила никогда не реагируют на них, а дополнение может использовать их, чтобы отличить изменение правила от изменения человека.

## Изменение заявок из дополнения {: #change-tickets-from-an-add-on }

Контейнер — `Grav\Plugin\HelpdeskPro\HelpdeskPro::instance()`. Каждая запись принимает `Actor`, выполняется в одной транзакции, записывает свою активность и публикует события после фиксации; чтение идёт через политику доступа.

- `actorResolver()->resolveUser($gravUser)` превращает учётную запись Grav в `Actor`. `Actor::system('myaddon')` действует как системный человек Helpdesk с вашей меткой в активности. `Actor::system('myaddon')->withDetail(['rule' => 'Nightly cleanup'])` сохраняет деталь рядом.
- `tickets()`: `get`, `list` (с `TicketFilter`), `create`, `apply($ticketId, TicketChange, $actor)` для любого изменения поля, `delete`, `restore`.
- `messages()`: `reply`, `note`, `thread`, `timeline`, `edit`, `delete`.
- `intake()`: новые заявки из портала, гостевой формы и почты.
- `accessPolicy()`: `canReadTicket`, `canReadMessage`, `canWriteTicket`, `readableTicketsSql` для собственных запросов, `audienceFor` для рассылки.
- `people()`, `projects()`, `statuses()`, `labels()`, `participants()`, `fields()`, `organizations()` для остального.

```php
use Grav\Plugin\HelpdeskPro\Domain\TicketChange;
use Grav\Plugin\HelpdeskPro\HelpdeskPro;

HelpdeskPro::instance()->tickets()->apply($ticketId, TicketChange::fromArray(['status_id' => 5]), $actor);
```

Для значений пользовательских полей `fields()->serialize($actor, $ticket)` даёт значения, которые актор может видеть, а `fields()->setValues($actor, $ticketId, ['plan' => 'gold'])` меняет их с теми же проверками, активностью и событиями, что и рабочая область.

## Новые заявки: `onHelpdeskIntake` {: #new-tickets-onhelpdeskintake }

Срабатывает до создания заявки из каждого канала с `['draft' => TicketDraft, 'actor' => Actor, 'channel' => 'web']`. Слушатель может изменить черновик на месте (добавить метки, задать приоритет, заполнить `extra`) или отклонить заявку, бросив `Grav\Plugin\HelpdeskPro\Exception\DomainValidationException` с сообщением, которое увидит человек. Всё, что слушатель меняет, проверяется снова перед записью заявки. То, что слушатели добавляют в `extra`, идёт в событии `ticket.created`.

## Полезные нагрузки заявки: `onHelpdeskTicketSerialize` {: #ticket-payloads-onhelpdeskticketserialize }

Срабатывает для каждой заявки, которую возвращает API, с `ticket` (сырая строка), `extra` (массив для заполнения), `actor` (вызывающий) и `staff` (получает ли вызывающий вид сотрудника). Слушатель сам решает, что может видеть клиент:

```php
public function onHelpdeskTicketSerialize(Event $event): void
{
    $extra = $event['extra'];
    $extra['sla_due'] = $this->slaDue((int)$event['ticket']['id']);
    $event['extra'] = $extra;
}
```

## Почта {: #email }

`onHelpdeskMailBuild` срабатывает для каждого письма до его отрисовки с `template`, `vars`, `headers`, `audience` (`client` или `staff`), `ticket_id` (или null для дайджеста) и `person_id` (получатель или null). Слушатель может изменить `vars` или `headers` и вернуть их обратно; `Message-ID` изменить нельзя.

Проверка, которая не пускает заметки в письма клиентам, выполняется после слушателей, поэтому дополнение не может протолкнуть содержимое заметки в клиентскую почту. `vars.brand` заполняется до запуска слушателей, поэтому слушатель может подставить другой логотип или цвет для письма одного проекта.

Новый шаблон письма расширяет `emails/helpdesk/partials/layout.html.twig` (см. [Исходящую почту](../outbound-email/index.md#override-email-templates)).

## Входящая почта {: #inbound-email }

Дополнение, получающее почту другим способом, может провести её через тот же конвейер:

```php
use Grav\Plugin\Email\Providers\Inbound\InboundMessage;
use Grav\Plugin\HelpdeskPro\HelpdeskPro;

$stored = HelpdeskPro::instance()->inboundStore()->storeMessage(InboundMessage::fromMime($raw, 'myaddon'), 'myaddon');
$stored->inboundId; // строка журнала входящих
$stored->duplicate; // true, если это письмо уже приходило
```

Сначала проверьте `Grav\Plugin\HelpdeskPro\Inbound\InboundSupport::available()`: оно равно false с плагином Email старше 5.3.0. Слушайте `inbound.rejected`, чтобы узнавать об отклонённой почте.

Очистку ответов можно использовать отдельно. `(new ReplyStripper())->strip($textPart, $htmlPart, $providerStrippedText, $subject, $inlineImages)` возвращает Markdown для сохранения (`text`), было ли что-то отрезано (`stripped`), правило, которое отрезало (`cutAt`), `signature` и флаги `quoteOnly`, `keptFull` и `forwarded`. `HtmlToMarkdown::convert($html)` сам превращает HTML письма в Markdown.

## Поиск {: #search }

- `clientSearch()->search($actor, $text)` находит то, что может читать клиент или гость; `staffSearch()->search($actor, $text)` — то, что может агент, с заметками; `similarTickets()->for($ticketId, $actor)`. Каждая находка проверяется по политике доступа.
- `search()->markDirty($ticketId)` переиндексирует заявку в конце запроса для дополнения, которое меняет текст заявки вне основных сервисов.
- `useEmbeddingProvider($provider)` передаёт ваш собственный `EmbeddingProviderInterface` для семантического поиска.

## Обновления в реальном времени на ваших экранах {: #live-updates-in-your-own-screens }

Панель или представление рабочей области либо скрипт на странице портала может следовать тем же кадрам реального времени, что используют собственные экраны Helpdesk Pro. Слушайте событие `helpdesk:live` на `window`, чей `detail` — кадр (`{e, t, m, at}`, только id), и перезапрашивайте то, что показываете, через API или страницу. Рабочая область открывает одно живое соединение на вкладку; не открывайте другое. См. [Обновления в реальном времени](../live-updates/index.md#how-it-works).

Экран может участвовать в сочетаниях клавиш рабочей области: рабочая область сначала отправляет событие `hd-shortcut` (с `detail.action`, например `next`, `open` или `assign-me`) элементу экрана, и экран, который его обрабатывает, вызывает `preventDefault()`.

## Люди и удаление {: #people-and-erasure }

Объединение двух людей и удаление одного должны дойти до каждой таблицы со столбцом человека, включая вашу. Реализуйте `Grav\Plugin\HelpdeskPro\Service\People\PersonData` и зарегистрируйте его:

```php
public static function getSubscribedEvents(): array
{
    return ['onHelpdeskRegisterPersonData' => ['onHelpdeskRegisterPersonData', 0]];
}

public function onHelpdeskRegisterPersonData(Event $event): void
{
    $areas = $event['areas'];
    $areas[] = new MyAddonPersonData(HelpdeskPro::instance()->connection());
    $event['areas'] = $areas;
}
```

У реализации четыре метода:

- `name()`: короткое имя для отчётов (`myaddon`).
- `holdings(int $personId): array`: что вы храните о человеке, как метка → число строк, для сводки конфиденциальности.
- `merge(int $fromId, int $intoId, int $now): int`: направьте свои строки на сохранённого человека; удалите строку, которая столкнулась бы с уже имеющейся у него.
- `erase(ErasureRun $run): int`: удалите или очистите то, что храните, в `$run->mode` (`anonymize` или `delete_content`); `$run->email` — адрес, который у него был.

Оба выполняются внутри транзакции Helpdesk Pro: никогда не фиксируйте её, не отправляйте почту и не удаляйте файлы сами (посчитайте файл, который отпускаете, в `$run->releasedBlobs` и оставьте его `attachments.gc`), делайте каждый шаг идемпотентным и пропускайте отсутствующую таблицу. `Service\People\TablePersonData` содержит помощники для всего этого. Юридическое удержание проверяется до того, как ваш `erase()` вообще будет вызван.

## Правила: триггеры, условия и действия {: #rules-triggers-conditions-and-actions }

Правила читают триггеры, условия и действия из реестра. Helpdesk Pro заполняет его, затем публикует `onHelpdeskRulesRegister` с `['registry' => RuleRegistry]`. Редактор правил, каталог API и пробный запуск подхватывают то, что вы добавили. Первая регистрация ключа побеждает, поэтому добавляйте префикс (`billing.plan`).

```php
use Grav\Plugin\HelpdeskPro\Rules\{ActionType, ConditionField, RuleSubject, TriggerType};
use Grav\Plugin\HelpdeskPro\Exception\DomainValidationException;
use Grav\Plugin\HelpdeskPro\Security\Actor;

public function onHelpdeskRulesRegister(Event $event): void
{
    $registry = $event['registry'];

    // Условие: тарифный план клиента. Редактор рисует select с этими вариантами.
    $registry->addCondition(new ConditionField('billing.plan', 'Billing plan', ['in', 'not_in', 'is_empty'], 'select',
        fn (RuleSubject $s) => $s->remember('billing', fn () => $this->billing->forTicket($s->ticketId()))['plan'] ?? null,
        [['value' => 'gold', 'label' => 'Gold'], ['value' => 'free', 'label' => 'Free']],
        words: ['is_empty' => 'has none'],
        phrases: ['is_empty' => 'the customer has no plan']));

    // Триггер: одно из ваших доменных событий, отправляемое через приёмник событий.
    $registry->addTrigger(new TriggerType('billing.invoice_overdue', 'An invoice is overdue',
        fn (string $event, array $payload) => $event === 'billing.invoice_overdue'));

    // Действие: проверить значение при сохранении правила, сообщить, что оно сделает, и выполнить.
    $registry->addAction(new ActionType('webhook', 'Call a webhook', 'text',
        fn (mixed $value) => ['value' => filter_var($value, FILTER_VALIDATE_URL) ?: throw new DomainValidationException('Give a URL')],
        fn (array $action) => 'Call ' . $action['value'],
        fn (array $action, RuleSubject $s, Actor $actor) => $this->webhooks->post($action['value'], $s->ticketId()),
    ));
}
```

- **Условие** читает заявку через `RuleSubject`: `ticket`, `event`, `payload`, `labelIds()`, `requester()`, `message()` и `remember($name, $loader)`, чтобы загрузить ваши данные один раз для всех правил. Вид ввода (`text`, `tags`, `number`, `date`, `checkbox`, `select` или основной, например `labels`) говорит редактору, что рисовать. Необязательные именованные аргументы: `options` (может быть замыканием), `words`, `phrases`, `normalize` и `unavailable`.
- **Условия, которые появляются и исчезают вместе с вашими данными**, идут через `$registry->addConditionSource(fn () => [...ConditionField])`, который спрашивают при каждом поиске поля.
- **Триггер** проверяет имя и полезную нагрузку доменного события. Ваше событие доходит до правил, когда вы публикуете его через `HelpdeskPro::instance()->domainEvents()->record(...)` с `ticket_id`.
- **Действие** выполняется как `Actor::system('rules')` с `rule_id` правила и `rule` в `$actor->detail`. Меняйте заявки через `tickets()->apply()`; всё, что оно вызывает, несёт `via: rules` и никогда не запускает правила. Бросьте исключение, чтобы потерпеть неудачу: ошибка показывается на правиле, а следующее правило выполняется.

## Типы каналов уведомлений {: #notification-channel-types }

Добавьте тип канала (Microsoft Teams, Mattermost, сервис пейджера) через `onHelpdeskNotificationChannels`, который срабатывает с `['channels' => []]`. Он появляется в Automation → Channels, его поля рисуются в редакторе, сохраняются, маскируются и доставляются как основные:

```php
use Grav\Plugin\HelpdeskPro\Channels\ChannelMessage;

public function onHelpdeskNotificationChannels(Event $event): void
{
    $channels = $event['channels'];
    $channels[] = [
        'id' => 'teams',
        'label' => 'Microsoft Teams',
        'description' => 'Posts to a Teams channel through a workflow webhook.',
        'fields' => [
            ['name' => 'url', 'label' => 'Workflow URL', 'kind' => 'url', 'required' => true],
        ],
        'build' => static fn (ChannelMessage $m, array $settings): array => ['body' => [
            'type' => 'message',
            'attachments' => [[
                'contentType' => 'application/vnd.microsoft.card.adaptive',
                'content' => [
                    'type' => 'AdaptiveCard', 'version' => '1.4',
                    'body' => [['type' => 'TextBlock', 'text' => $m->title, 'weight' => 'Bolder']],
                    'actions' => [['type' => 'Action.OpenUrl', 'title' => 'Open', 'url' => $m->url()]],
                ],
            ]],
        ]],
    ];
    $event['channels'] = $channels;
}
```

| Ключ | Что это |
|---|---|
| `id` | Строчные буквы, цифры, `-` и `_`, от 2 до 32 символов. Первая регистрация побеждает. |
| `label`, `description` | Что показывает выбор типа. |
| `fields` | Что спрашивает редактор: `{name, label, kind, required, help, placeholder}`. `kind` — `url` (обрабатывается как секрет), `secret`, `text` или `addresses`. Значение `url` или `secret` никогда не возвращается API и может быть введено как `env:NAME`. |
| `build` | `fn (ChannelMessage $message, array $settings)`, возвращающая `Channels\HttpPayload` или `['body' => array|string, 'headers' => [...]]`. Тело-массив отправляется как JSON. |
| `endpoint` | Необязательно: `fn (array $settings): string`, куда делать `POST`; при отсутствии используется поле `url`. |
| `signs` | Необязательно: `true` добавляет `X-Grav-Signature`, когда у канала задано поле `secret`. |

`ChannelMessage` — всё, что может напечатать построитель: `title`, `ticket`, `fields()`, `details`, `excerpt`, `url()`, `ticketRef()`, `eventLabel()`, `event`, `siteName`, `deskUrl`, `occurredAt`, `test`, `deliveryId` и `toArray()`. У него нет поля для заметки, внутреннего пользовательского поля или журнала активности. Повторы, пауза, дедупликация и **Send a test message** работают для вашего типа без вашего кода. Если ваше дополнение позже отключат, его каналы помечаются **Broken**, и их можно только удалить.

## Карточки отчётов {: #report-cards }

Добавляйте карточки на экран Reports через `onHelpdeskReports`, который срабатывает с `reports` (список для добавления), `actor`, `period` (с `start`, `end`, `fromDate`, `toDate`, `timezone` и `days`), `scope` и `context` (`desk` или `tools` для страницы Reports в Admin Next, которая рисует только факты и таблицы):

```php
public function onHelpdeskReports(Event $event): void
{
    [$where, $params] = $event['scope']->where('t');
    $hours = (float)HelpdeskPro::instance()->connection()->fetchValue(
        "SELECT COALESCE(SUM(e.minutes), 0) / 60.0 FROM helpdesk_myaddon_time_entries e
         JOIN helpdesk_tickets t ON t.id = e.ticket_id
         WHERE {$where} AND e.created_at >= ? AND e.created_at < ?",
        [...$params, $event['period']->start, $event['period']->end]
    );
    $reports = $event['reports'];
    $reports[] = [
        'id' => 'myaddon.hours',
        'title' => 'Hours logged',
        'position' => 55,
        'facts' => [['label' => 'Hours', 'value' => $hours]],
    ];
    $event['reports'] = $reports;
}
```

`$event['scope']->where('t')` даёт фрагмент `WHERE` по псевдониму `helpdesk_tickets t`, уже содержащий фильтры отчёта и доступ смотрящего к проектам.

| Ключ | Что это |
|---|---|
| `id` | Обязателен. Строчные буквы, цифры, `.`, `_` и `-`, до 64 символов; добавьте префикс имени дополнения. Собственные id Helpdesk Pro заняты. |
| `title` | Обязателен. Обычный текст. |
| `facts` | Список `{label, value, format?, tone?, hint?}` |
| `columns`, `rows` | Таблица. Рабочая область предлагает её как CSV. |
| `series` | График: `{kind: line or bar, x: [labels], sets: [{label, values}], format?}`. Опускается на странице Reports в Admin Next. |
| `empty` | Почему нечего показывать: `{title, hint?, action?: {label, route}}` |
| `note` | Строка под карточкой |
| `wide` | `true`, чтобы занять всю строку |
| `position` | Порядок на экране, меньшее первым. Собственные карточки Helpdesk Pro используют 10–130; по умолчанию 100. |
| `provider` | Имя вашего дополнения (по умолчанию `addon`) |

У карточки ровно одно из `facts`, `columns` с `rows`, `series` или `empty`. `format` — `number`, `percent`, `duration` (секунды) или `text`; `tone` — `ok`, `warn`, `bad`, `info` или `muted`. Значения всегда показываются текстом, никогда HTML. Карточка, не следующая контракту, опускается, а её id перечисляется в `skipped` ответа API.

## Связанные разделы {: #related }

- [REST API](../rest-api/index.md)
- [Правила и автоматизация](../rules-and-automations/index.md)
- [Каналы уведомлений](../notification-channels/index.md)
- [Отчёты](../reports/index.md)
