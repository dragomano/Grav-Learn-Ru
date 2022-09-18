---
description: Используемые хуки событий в Grav CMS.
---

# Хуки событий

В предыдущей [главе](../plugin-tutorial), вы, возможно, заметили, что наша логика плагина была заключена в двух методах. Каждый из этих методов `onPluginsInitialized` и `onPageInitialized` соответствуют **хукам событий**, которые доступны на протяжении всего жизненного цикла Grav.

Чтобы полностью использовать мощь плагинов Grav, вам нужно знать, какие хуки событий доступны, в каком порядке вызываются и что доступно во время этих вызовов. **Хуки событий** имеют прямое отношение к общему [жизненному циклу Grav](../grav-lifecycle).

## Порядок событий

Большинство событий в Grav происходят в определенном порядке, и важно понимать этот порядок, если вы создаете плагины:

1. [onFatalException](#onfatalexception) _(нет порядка, может произойти в любое время)_
1. `PluginsLoadedEvent` class (1.7)
1. `PluginsLoadedEvent` class (1.7)
1. [onPluginsInitialized](#onpluginsinitialized)
1. `FlexRegisterEvent` class (1.7)
1. onThemeInitialized
1. onRequestHandlerInit (1.6)
1. onTask (1.6)
  1. onTask.{task}
1. onAction (1.6)
  1. onAction.{action} (1.6)
1. onBackupsInitialized
1. onSchedulerInitialized (1.6)
1. [onAssetsInitialized](#onassetsinitialized)
1. [onTwigTemplatePaths](#ontwigtemplatepaths)
1. [onTwigLoader](#ontwigloader)
1. [onTwigInitialized](#ontwiginitialized)
1. [onTwigExtensions](#ontwigextensions)
1. [onBuildPagesInitialized](#onbuildpagesinitialized) _(один раз при повторной обработке страниц)_
  1. [onPageProcessed](#onpageprocessed) _(каждая страница, ещё не кэшированная)_
  1. onFormPageHeaderProcessed (1.6) _(каждая страница, ещё не кэшированная)_
  1. [onFolderProcessed](#onfolderprocessed) _(для каждой найденной папки)_
1. [onPagesInitialized](#onpagesinitialized)
1. [onPageInitialized](#onpageinitialized)
  1. [onPageContentRaw](#onpagecontentraw) _(каждая страница, ещё не кэшированная)_
  1. [onMarkdownInitialized](#onmarkdowninitialized)
  1. [onPageContentProcessed](#onpagecontentprocessed) _(каждая страница, ещё не кэшированная)_
  1. onPageContent _(вызывается при первом вызове Page::content() даже при кэшировании.)_
1. [onPageNotFound](#onpagenotfound)
1. onPageAction (1.6)
  1. onPageAction.{action} (1.6)
1. onPageTask (1.6)
  1. onPageTask.{task} (1.6)
1. [onTwigPageVariables](#ontwigpagevariables) _(каждая страница, ещё не кэшированная)_
1. onHttpPostFilter (1.5.2)
1. [onTwigSiteVariables](#ontwigsitevariables)
1. [onCollectionProcessed](#oncollectionprocessed) _(по запросу коллекции)_
1. [onOutputGenerated](#onoutputgenerated)
1. [onPageHeaders](#onPageHeaders)
1. [onOutputRendered](#onoutputrendered)
1. [onShutdown](#onshutdown)

Разные события:

1. [onBlueprintCreated](#onblueprintcreated)
1. onTwigTemplateVariables
1. onTwigStringVariables
1. [onBeforeDownload](#onbeforedownload)
1. [onPageFallBackUrl](#onpagefallbackurl)
1. [onMediaLocate](#onmedialocate)
1. [onGetPageBlueprints](#ongetpageblueprints)
1. [onGetPageTemplates](#ongetpagetemplates)
1. onFlexObjectRender (1.6)
1. onFlexCollectionRender (1.6)
1. onBeforeCacheClear
1. onImageMediumSaved (ImageFile)
1. onAfterCacheClear (1.7)
1. onHttpPostFilter (1.7)
1. `PermissionsRegisterEvent` class (1.7)

## Хуки событий ядра Grav

Есть несколько основных обработчиков событий Grav, которые запускаются во время обработки страницы:

### onFatalException

Это событие может быть запущено в любое время, если PHP выдает фатальное исключение. В настоящее время это используется плагином `problems` для обработки списка возможных причин, по которым Grav выдает фатальное исключение.

### onPluginsInitialized

Это первое доступное событие плагина. На данный момент были инициированы следующие объекты:

* Uri
* Config
* Debugger
* Cache
* Plugins

!!! danger ""

	Плагин не будет загружен вообще, если для этого конкретного плагина установлена ​​опция конфигурации `enabled: false`.

### onAssetsInitialized

Событие означает, что менеджер активов инициализирован и готов к добавлению активов и управлению ими.

### onPagesInitialized

Это событие означает, что все страницы в папке Grav `user/pages` были загружены как объекты и доступны в **объекте Pages**.

### onPageNotFound

Это событие, которое может быть запущено, если ожидаемая страница не найдена. В настоящее время это используется плагином `error` для отображения красивой страницы с ошибкой 404.

### onPageInitialized

Текущая страница по запросу URL загружена в **объект Page**.

### onOutputGenerated

Вывод был обработан **механизмом создания шаблонов Twig** и теперь представляет собой всего лишь строку HTML.

### onPageHeaders

Позволяет манипулировать объектом заголовков страницы.

### onOutputRendered

Вывод был полностью обработан и отправлен на дисплей.

### onShutdown

Новое и очень мощное событие, которое позволяет выполнять действия после того, как Grav завершил обработку и соединение с клиентом было закрыто. Это особенно полезно для выполнения действий, которые не требуют взаимодействия с пользователем и потенциально могут повлиять на производительность. Возможные варианты использования включают отслеживание пользователей и обработку заданий.

### onBeforeDownload

Это новое событие передает объект события, содержащий `file`. Это событие можно использовать для ведения журнала или предоставления/запрета доступа для загрузки указанного файла.

### onGetPageTemplates

Это событие позволяет плагинам предоставлять свои собственные шаблоны в дополнение к шаблонам, собранным из структуры каталогов и ядра темы. Это особенно полезно, если вы хотите, чтобы плагин предоставлял собственный шаблон.

**Пример**

```twig
/**
 * Add page template types.
 */
public function onGetPageTemplates(Event $event)
{
    /** @var Types $types */
    $types = $event->types;
    $types->register('downloads');
}
```

Это позволяет плагину зарегистрировать шаблон (который он может предоставить), чтобы он отображался в раскрывающемся списке типов шаблонов страниц (например, при редактировании страницы). В приведенном выше примере добавлен тип шаблона `downloads`, поскольку в каталоге `downloads` есть файл `downloads.html.twig`.

![](ongetpagetemplates.png)

### onGetPageBlueprints

Это событие, такое как `onGetPageTemplates`, позволяет плагину предоставлять свои собственные ресурсы в дополнение к основным и специфическим для темы. В данном случае это чертежи.

**Пример**

```php
<?php

$scanBlueprintsAndTemplates = function () use ($grav) {
    // Scan blueprints
    $event = new Event();
    $event->types = self::$types;
    $grav->fireEvent('onGetPageBlueprints', $event);

    self::$types->scanBlueprints('theme://blueprints/');

    // Scan templates
    $event = new Event();
    $event->types = self::$types;
    $grav->fireEvent('onGetPageTemplates', $event);

    self::$types->scanTemplates('theme://templates/');
};
```

В этом примере мы используем хуки `onGetPageTemplates` и `onGetPageBlueprints`, чтобы сделать эти предоставляемые плагином ресурсы (шаблоны и чертежи) доступными Grav для наследования и других целей.

## Хуки событий Twig

Twig имеет собственный набор обработчиков событий.

### onTwigTemplatePaths

Базовые местоположения для путей к шаблонам были установлены на **объекте Twig**. Если вам нужно добавить другие места, где Twig будет искать пути к шаблонам, это событие следует использовать.

**Пример**

```php
<?php

/**
 * Add template directory to twig lookup path.
 */
 public function onTwigTemplatePaths()
 {
     $this->grav['twig']->twig_paths[] = __DIR__ . '/templates';
 }
```

### onTwigInitialized

На этом этапе шаблонизатор Twig инициализирован.

### onTwigExtensions

Основные расширения Twig были загружены, но если вам нужно добавить собственное расширение Twig, вы можете сделать это с помощью этого обработчика событий.

### onTwigPageVariables

Где Twig обрабатывает страницу напрямую, то есть когда вы устанавливаете `process: twig: true` в заголовках YAML страницы. Здесь вы должны добавить в Twig любые переменные, которые должны быть доступны Twig во время этого процесса.

### onTwigSiteVariables

Где Twig обрабатывает полную иерархию шаблонов сайта. Здесь вы должны добавить в Twig любые переменные, которые должны быть доступны Twig во время этого процесса.

## Хуки событий коллекции

### onCollectionProcessed

Если вам нужно манипулировать коллекцией после того, как она была обработана, самое время это сделать.

## Хуки событий страницы

### onBuildPagesInitialized

Это событие запускается один раз, когда страницы будут повторно обработаны. Обычно это происходит, если срок действия кэша истек или его необходимо обновить. Это полезное событие для плагинов, которым необходимо управлять контентом и кэшировать результаты.

### onBlueprintCreated

Это используется для обработки и обработки форм.

### onPageContentRaw

После того, как страница найдена, заголовок обрабатывается, но содержимое **не** обрабатывается. Это запускается для **каждой страницы** в системе Grav. Производительность не является проблемой, потому что это событие не будет запускаться на кэшированной странице, только когда кэш очищен или происходит событие очистки кэша.

### onPageProcessed

После того, как страница проанализирована и обработана. Это запускается для **каждой страницы** в системе Grav. Производительность не является проблемой, потому что это событие не будет запускаться на кэшированной странице, только когда кэш очищен или происходит событие очистки кэша.

### onMarkdownInitialized

Вызывается при инициализации Markdown. Позволяет переопределить реализацию обработки Parsedown по умолчанию. См. [Пример использования в PR](https://github.com/getgrav/grav/pull/747#issuecomment-206821370).

### onPageContentProcessed

Это событие запускается после того, как метод страницы `content()` обработал содержимое страницы. Это особенно полезно, если вы хотите выполнять действия с постобработанным содержимым, но при этом убедитесь, что результаты кэшируются. Производительность не является проблемой, потому что это событие не будет запускаться на кэшированной странице, только когда кэш очищен или происходит событие очистки кэша.

### onFolderProcessed

После того, как папка проанализирована и обработана. Он запускается для **каждой папки** в системе Grav. Производительность не является проблемой, потому что это событие не будет запускаться на кэшированной странице, только когда кэш очищен или происходит событие очистки кэша.

### onPageFallBackUrl

Если маршрут не распознается как страница, Grav пытается получить доступ к медиаресурсу страницы. Событие запускается, как только начинается процедура, поэтому плагины могут подключаться и предоставлять дополнительные функции.

### onMediaLocate

Adds support for custom media locations for excerpts.

### onTwigLoader

Добавляет поддержку использования пространств имен вместе с двумя новыми методами в классе Twig: `Twig::addPath($path, $namespace)` и `Twig::prependPath($path, $namespace)`.
