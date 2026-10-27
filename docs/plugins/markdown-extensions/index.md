---
title: "Расширения Markdown"
description: "Grav рендерит Markdown через форк Parsedown и Parsedown Extra, обёрнутый собственным слоем расширений Grav. Плагин может научить этот парсер новому блочному и встроенному синтаксису — алертам, блокам в стиле шорткодов, пользовательским…"
tags:
  - "Плагины"
  - "Markdown"
  - "PHP"
---

# Расширение Markdown

Grav рендерит Markdown через форк [Parsedown](https://parsedown.org/) и Parsedown Extra, обёрнутый собственным слоем расширений Grav. Плагин может научить этот парсер **новому блочному и встроенному синтаксису** — алертам, блокам в стиле шорткодов, пользовательским встроенным маркерам и так далее.

Grav 2.0 добавляет для этого формальный **Markdown Extension API**: небольшой реестр, набор интерфейсов обработчиков и fluent-построитель `Element`. Старый подход (назначение замыканий парсеру и вызов `addBlockType()`/`addInlineType()`) по-прежнему работает без изменений, поэтому существующие плагины продолжают работать — но новым плагинам следует предпочитать API, описанный здесь.

## Обзор

Каждый раз, когда Grav создаёт парсер Markdown для страницы, он вызывает событие `onMarkdownInitialized`. Ваш плагин слушает это событие, оборачивает парсер в `MarkdownExtensionRegistry` и добавляет одно или несколько **расширений**. Каждое расширение регистрирует **обработчики** для символа-триггера («маркера»), и каждый обработчик возвращает разобранный вывод в виде массива элемента — проще всего созданного построителем `Element`.

Компоненты, все в пространстве `Grav\Common\Markdown`:

| Класс / интерфейс | Роль |
| --- | --- |
| `Extension\MarkdownExtensionInterface` | Расширение: `getName()`, `isEnabled()`, `register()`. |
| `Extension\AbstractMarkdownExtension` | Базовый класс — принимает конфигурацию в конструкторе и по умолчанию устанавливает `isEnabled()` в `true`. |
| `Extension\MarkdownExtensionRegistry` | Оборачивает парсер; добавляет расширение через `add()` или напрямую через `registerBlock()` / `registerInline()`. |
| `Extension\BlockHandlerInterface` | Обработчик блока — `block()`. |
| `Extension\BlockContinuableInterface` | Необязательный — `blockContinue()` для многострочных блоков. |
| `Extension\BlockCompletableInterface` | Необязательный — `blockComplete()` для завершения блока. |
| `Extension\InlineHandlerInterface` | Обработчик встроенного синтаксиса — `inline()`. |
| `Element` | Fluent-построитель, компилируемый в массив элемента Parsedown. |
| `BlockResult` | Необязательный помощник, объединяющий элемент и состояние блока. |

## Событие `onMarkdownInitialized`

Это событие срабатывает один раз для каждого парсера, передавая экземпляр парсера и текущую страницу:

```php
public static function getSubscribedEvents(): array
{
    return [
        'onMarkdownInitialized' => ['onMarkdownInitialized', 0],
    ];
}

public function onMarkdownInitialized(\RocketTheme\Toolbox\Event\Event $event): void
{
    $markdown = $event['markdown'];   // экземпляр Parsedown
    $page     = $event['page'];       // отрисовываемый PageInterface

    $registry = new \Grav\Common\Markdown\Extension\MarkdownExtensionRegistry($markdown, $page);
    $registry->add(new \Grav\Plugin\MyMarkdown\CiteExtension($this->config->get('plugins.my-plugin')));
}
```

> [!NOTE] Примечание
> Поскольку событие передаёт `page`, расширение может менять поведение для каждой страницы — например, читать флаг из метаданных страницы перед тем, как решить, регистрироваться ли.

## Создание расширения

### 1. Класс расширения

Расширьте `AbstractMarkdownExtension` и реализуйте один или несколько интерфейсов обработчиков. `getName()` возвращает короткий идентификатор; `isEnabled()` позволяет полностью отключить расширение (по умолчанию `true`); `register()` — место, где вы прикрепляете обработчики к реестру.

```php
use Grav\Common\Markdown\Element;
use Grav\Common\Markdown\Extension\AbstractMarkdownExtension;
use Grav\Common\Markdown\Extension\InlineHandlerInterface;
use Grav\Common\Markdown\Extension\MarkdownExtensionRegistry;

class CiteExtension extends AbstractMarkdownExtension implements InlineHandlerInterface
{
    public function getName(): string
    {
        return 'cite';
    }

    public function register(MarkdownExtensionRegistry $r): void
    {
        // тег (логическое имя в StudlyCase), маркер (символ-триггер), обработчик
        $r->registerInline('Cite', '@', $this);
    }

    public function inline(array $excerpt): ?array
    {
        // Совпадение @@text@@ и вывод <cite>text</cite>
        if (preg_match('/^@@(?=\S)(.+?)@@/', $excerpt['text'], $m)) {
            return [
                'extent'  => strlen($m[0]),                          // потреблённые символы
                'element' => Element::create('cite')->setInlineText($m[1])->toArray(),
            ];
        }

        return null; // отклонить — пусть попробуют другие обработчики
    }
}
```

Это единственное расширение превращает `a @@b c@@ d` в `<p>a <cite>b c</cite> d</p>`.

### 2. Регистрация обработчиков блоков или встроенного синтаксиса

Реестр сопоставляет ваш обработчик с диспетчеризацией Parsedown по **маркеру** — первому символу, который может запустить ваш синтаксис.

| Метод | Назначение | Опции |
| --- | --- | --- |
| `registerInline($tag, $marker, $handler, $options)` | Встроенный синтаксис (выполняется внутри строки). | `index` |
| `registerBlock($tag, $marker, $handler, $options)` | Блочный синтаксис (целая строка/область). | `continuable`, `completable`, `index` |

- **`tag`** — логическое имя в StudlyCase (`'Cite'`, `'Note'`). Оно должно быть уникальным только для парсера.
- **`marker`** — символ-триггер (`'@'`, `'>'`). Для блока пустая строка `''` регистрирует *немаркированный* блок, который проверяется на каждой строке.
- **`index`** — позиция обработчика в списке для этого маркера (меньшие значения выполняются первыми). Используйте `['index' => 0]`, чтобы получить приоритет.
- **`continuable` / `completable`** — установите их (или просто реализуйте `BlockContinuableInterface` / `BlockCompletableInterface`) для многострочных блоков.

Методы обработчиков получают рабочие массивы Parsedown и возвращают массивы элементов:

| Метод | Получает | Возвращает |
| --- | --- | --- |
| `inline(array $excerpt)` | `text` (начиная с маркера) и `context` (вся строка) | `['extent' => int, 'element' => array]` или `null`, чтобы отклонить |
| `block(array $line, ?array $block)` | `line` (`body`, `indent`, `text`) и открытый блок, если он есть | массив блока `['element' => [...], ...state]` или `null` |
| `blockContinue(array $line, array $block)` | следующая строка и текущий блок | изменённый блок, чтобы оставить его открытым, или `null`, чтобы закрыть |
| `blockComplete(array $block)` | завершённый блок | окончательный блок |

### 3. Построение элементов с помощью `Element`

Parsedown представляет вывод в виде вложенных массивов. Построитель `Element` создаёт такие массивы, избавляя вас от необходимости помнить точные ключи, и выбирает правильный «обработчик» для типа содержимого:

```php
use Grav\Common\Markdown\Element;

// <span class="badge">New</span>  (текст разбирается как встроенный markdown)
Element::span()->addClass('badge')->setInlineText('New')->toArray();

// <div class="note"> ...дочерние строки блока разбираются как markdown... </div>
Element::div()->addClass('note')->setRawLines(['line one', 'line two'])->toArray();

// произвольный элемент с атрибутами
Element::create('figure')->attr('id', 'fig-1')->setChildren([$child])->toArray();
```

| Метод | Обработка содержимого |
| --- | --- |
| `setInlineText(string)` | Разбирается как **встроенный** markdown (обработчик `line` Parsedown). |
| `setRawLines(array)` | Каждая строка разбирается как **блок** (обработчик `lines` Parsedown). |
| `setChildren(array)` | Дочерние `Element`/массивы выводятся по порядку (обработчик `elements`). |
| `setListItems(array)` | Каждая строка становится элементом списка (обработчик `li`). |
| `setText(string)` | Экранированный буквальный текст — без markdown и HTML. |
| `setRawHtml(string, $allowInSafeMode=false)` | Буквальный HTML (используйте осторожно). |
| `attr(name, value)` / `attributes([...])` / `addClass(...)` | Задать атрибуты/классы. |
| `setNonNestables(array)` | Встроенные типы, которые не должны вкладываться в этот элемент. |
| `toArray()` | Скомпилировать в окончательный массив элемента Parsedown. |

### 4. Подключение к плагину

Соберите всё в обычном плагине Grav. Плагин подписывается на `onMarkdownInitialized`, создаёт реестр и добавляет расширение:

```php
<?php
namespace Grav\Plugin;

use Composer\Autoload\ClassLoader;
use Grav\Common\Plugin;
use Grav\Common\Markdown\Extension\MarkdownExtensionRegistry;
use Grav\Plugin\MyMarkdown\CiteExtension;
use RocketTheme\Toolbox\Event\Event;

class MyMarkdownPlugin extends Plugin
{
    public static function getSubscribedEvents(): array
    {
        return [
            'onPluginsInitialized'  => ['onPluginsInitialized', 0],
            'onMarkdownInitialized' => ['onMarkdownInitialized', 0],
        ];
    }

    public function onPluginsInitialized(): void
    {
        /** @var ClassLoader $loader */
        $loader = $this->grav['loader'];
        $loader->addPsr4('Grav\\Plugin\\MyMarkdown\\', __DIR__ . '/classes', true);
    }

    public function onMarkdownInitialized(Event $event): void
    {
        $registry = new MarkdownExtensionRegistry($event['markdown'], $event['page']);
        $registry->add(new CiteExtension($this->config->get('plugins.my-markdown')));
    }
}
```

Это всё, что нужно — теперь `@@…@@` отображается везде, где Grav рендерит Markdown, и его можно отключить в конфигурации самого плагина, вернув `false` из `isEnabled()` расширения.

## Пример: многострочный блок

*Продолжаемый* блок читает больше одной строки. Зарегистрируйте его как продолжаемый, затем реализуйте `blockContinue()`, чтобы продолжать принимать строки, пока блок не закончится. Здесь строка `@note` открывает `<div class="note">`, содержимым которого становятся следующие строки:

```php
use Grav\Common\Markdown\BlockResult;
use Grav\Common\Markdown\Element;
use Grav\Common\Markdown\Extension\AbstractMarkdownExtension;
use Grav\Common\Markdown\Extension\BlockHandlerInterface;
use Grav\Common\Markdown\Extension\BlockContinuableInterface;
use Grav\Common\Markdown\Extension\MarkdownExtensionRegistry;

class NoteExtension extends AbstractMarkdownExtension implements BlockHandlerInterface, BlockContinuableInterface
{
    public function getName(): string
    {
        return 'note';
    }

    public function register(MarkdownExtensionRegistry $r): void
    {
        $r->registerBlock('Note', '@', $this, ['index' => 0]);
    }

    public function block(array $line, ?array $block = null): ?array
    {
        if (preg_match('/^@note\s*$/', $line['text'])) {
            // BlockResult объединяет элемент с дополнительным состоянием блока ('note' здесь)
            return BlockResult::fromElement(Element::div()->addClass('note')->setRawLines([]))
                ->set('note', true)
                ->toArray();
        }

        return null;
    }

    public function blockContinue(array $line, array $block): ?array
    {
        if (isset($block['interrupted']) || empty($block['note'])) {
            return null; // пустая строка (или чужой блок) закрывает note
        }

        $block['element']['text'][] = $line['body']; // добавить исходную строку
        return $block;
    }
}
```

Ввод:

```markdown
@note
hello **world**
second line
```

отрендерит `<div class="note">`, содержащий две строки содержимого, причём `**world**` разбирается как встроенный markdown внутри.

## Устаревший подход (всё ещё поддерживается)

До появления API плагины расширяли парсер, назначая замыкание методу с динамическим именем и вручную регистрируя маркер. Это **по-прежнему работает** и полностью обратно совместимо — замыкания имеют приоритет над новой маршрутизацией обработчиков, поэтому существующие плагины никогда не перекрываются:

```php
$markdown = $event['markdown'];

$markdown->blockLegacy = function ($line) {
    if (preg_match('/^%legacy\s*$/', $line['text'])) {
        return [
            'legacy'  => true,
            'element' => ['name' => 'div', 'handler' => 'lines', 'attributes' => ['class' => 'legacy'], 'text' => []],
        ];
    }
    return null;
};
$markdown->blockLegacyContinue = function ($line, array $block) {
    if (isset($block['interrupted']) || empty($block['legacy'])) {
        return null;
    }
    $block['element']['text'][] = $line['body'];
    return $block;
};

// addBlockType($marker, $tag, $continuable, $completable, $index)
$markdown->addBlockType('%', 'Legacy', true, false, 0);
```

Новый API делает то же самое безопаснее (типизированные объекты обработчиков вместо магических имён методов и построитель `Element` вместо вручную собранных массивов), но вы можете свободно смешивать оба подхода в одном плагине.

> [!NOTE] Примечание
> Встроенные фичи GitHub Flavored Markdown в Grav — [списки задач, marks, автолинки и улучшенные таблицы](../../content/markdown/index.md) — являются частью ядра и *не* регистрируются через этот API. Этот API предназначен для синтаксиса, который добавляет ваш плагин.

## Справочник API

Все классы находятся в пространстве имён `Grav\Common\Markdown` (интерфейсы обработчиков — в `Grav\Common\Markdown\Extension`).

| Член | Сигнатура |
| --- | --- |
| `MarkdownExtensionInterface::getName()` | `: string` |
| `MarkdownExtensionInterface::isEnabled()` | `: bool` |
| `MarkdownExtensionInterface::register()` | `(MarkdownExtensionRegistry $registry): void` |
| `MarkdownExtensionRegistry::__construct()` | `($markdown, ?PageInterface $page = null)` |
| `MarkdownExtensionRegistry::add()` | `(MarkdownExtensionInterface $extension): void` |
| `MarkdownExtensionRegistry::registerBlock()` | `(string $tag, string $marker, object $handler, array $options = []): void` |
| `MarkdownExtensionRegistry::registerInline()` | `(string $tag, string $marker, object $handler, array $options = []): void` |
| `BlockHandlerInterface::block()` | `(array $line, ?array $block = null): ?array` |
| `BlockContinuableInterface::blockContinue()` | `(array $line, array $block): ?array` |
| `BlockCompletableInterface::blockComplete()` | `(array $block): array` |
| `InlineHandlerInterface::inline()` | `(array $excerpt): ?array` |
