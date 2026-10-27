---
title: "Создание шорткода"
description: "Практическое руководство «сначала Twig, потом шорткод», которое выносит повторяющуюся логику из контента страниц в небольшой безопасный шорткод."
tags:
  - "Контент"
  - "Шорткоды"
  - "Twig"
---

# Создание шорткода

Поскольку Grav 2.0 помещает [Twig в контенте страниц](../twig-in-content/index.md) в песочницу, рекомендуемый способ дать авторам повторяемые динамические фрагменты — **шорткод**, а не сырой Twig. Шорткод — это короткий читаемый тег вроде `[team folder="/team" /]`, который автор вставляет на любую страницу. Вся логика живёт в PHP вашего плагина и доверенных шаблонах, поэтому внутри песочницы контента ничего не выполняется.

---

На этой странице показано, как заменить фрагмент Twig в контенте эквивалентным шорткодом с помощью плагина [Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core). Сначала установите **Shortcode Core**:

```bash
bin/gpm install shortcode-core
```

## Проблема: Twig в контенте

Допустим, вы хотите, чтобы страница выводила список всех сотрудников из папки `/team`. Раньше можно было включить Twig в контенте и написать прямо в теле страницы:

```twig
{% set staff = page.collection({'items': {'@page.children': '/team'}}) %}
<div class="team">
{% for person in staff %}
  <div class="member">
    <h3>{{ person.title }}</h3>
    {{ person.summary|raw }}
  </div>
{% endfor %}
</div>
```

Это работает только при включённом Twig в контенте, раскрывает логику шаблона всем, кто редактирует страницу, и каждый автор, которому нужен список, должен правильно скопировать фрагмент. Именно от такого песочница и призвана отучать.

## Решение: шорткод `[team]`

С шорткодом автор пишет одну строку:

```
[team folder="/team" /]
```

Логика переходит в небольшой плагин. Вот он целиком.

### 1. Схема плагина

`user/plugins/team-shortcode/blueprints.yaml` объявляет плагин и его зависимость от Shortcode Core:

```yaml
name: Team Shortcode
slug: team-shortcode
type: plugin
version: 1.0.0
description: Adds a [team] shortcode that lists pages under a folder.
icon: users
author:
  name: Your Name
dependencies:
  - { name: grav, version: '>=2.0.0' }
  - { name: shortcode-core }
```

### 2. Класс плагина

`user/plugins/team-shortcode/team-shortcode.php` подписывается на событие `onShortcodeHandlers` и регистрирует все шорткоды из папки плагина `shortcodes/`. Он также добавляет папку плагина `templates/` в Twig, чтобы шорткод мог рендерить доверенный шаблон:

```php
<?php
namespace Grav\Plugin;

use Grav\Common\Plugin;

class TeamShortcodePlugin extends Plugin
{
    public static function getSubscribedEvents(): array
    {
        return [
            'onPluginsInitialized' => ['onPluginsInitialized', 0],
        ];
    }

    public function onPluginsInitialized(): void
    {
        if ($this->isAdmin()) {
            return;
        }

        $this->enable([
            'onShortcodeHandlers' => ['onShortcodeHandlers', 0],
            'onTwigTemplatePaths' => ['onTwigTemplatePaths', 0],
        ]);
    }

    public function onShortcodeHandlers(): void
    {
        $this->grav['shortcode']->registerAllShortcodes(__DIR__ . '/shortcodes');
    }

    public function onTwigTemplatePaths(): void
    {
        $this->grav['twig']->twig_paths[] = __DIR__ . '/templates';
    }
}
```

### 3. Шорткод

`user/plugins/team-shortcode/shortcodes/TeamShortcode.php` выполняет работу. Он читает параметр `folder`, строит коллекцию в PHP, а затем передаёт данные доверенному Twig-шаблону:

```php
<?php
namespace Grav\Plugin\Shortcodes;

use Thunder\Shortcode\Shortcode\ShortcodeInterface;

class TeamShortcode extends Shortcode
{
    public function init()
    {
        $this->shortcode->getHandlers()->add('team', function (ShortcodeInterface $sc) {
            $folder = $sc->getParameter('folder', '/');

            $root = $this->grav['pages']->find($folder);
            $members = $root ? $root->children()->published() : [];

            return $this->twig->processTemplate('shortcodes/team.html.twig', [
                'members' => $members,
            ]);
        });
    }
}
```

> [!NOTE] Примечание
> Имя класса (`TeamShortcode`) должно совпадать с именем файла. Шорткоды расширяют базовый класс `Shortcode` из Shortcode Core, который предоставляет `$this->grav`, `$this->twig` и менеджер `$this->shortcode`.

### 4. Шаблон

`user/plugins/team-shortcode/templates/shortcodes/team.html.twig` содержит разметку. Это **доверенный файл шаблона на диске**, поэтому он никогда не помещается в песочницу, и именно здесь должна находиться логика отображения:

```twig
<div class="team">
{% for person in members %}
  <div class="member">
    <h3>{{ person.title }}</h3>
    {{ person.summary|raw }}
  </div>
{% endfor %}
</div>
```

Вот и вся миграция. Тело страницы снова состоит из одной читаемой строки, Twig живёт в контролируемом вами файле, а Twig в контенте может оставаться отключённым.

## Устройство шорткода

Обработчик шорткода получает объект шорткода со всем необходимым:

* `$sc->getParameter('name', $default)` читает атрибут, например `folder="/team"` выше.
* `$sc->getContent()` возвращает текст, обёрнутый парным шорткодом, например `[red]wrapped text[/red]`.
* `$sc->getName()` возвращает имя шорткода.

Шорткоды бывают двух форм:

* **Самозакрывающиеся**, как `[team folder="/team" /]`, для тегов, которые генерируют собственный вывод.
* **Парные**, как `[red]...[/red]`, которые оборачивают контент. Минимальный парный пример:

```php
$this->shortcode->getHandlers()->add('red', function (ShortcodeInterface $sc) {
    return '<span style="color:red;">' . $sc->getContent() . '</span>';
});
```

По умолчанию обработчики выполняются **после** обработки Markdown, поэтому шорткод может оборачивать Markdown-контент. Если нужно выполнить **до** Markdown (например, чтобы защитить блок кода от разбора), регистрируйтесь через `getRawHandlers()` вместо `getHandlers()`.

## Дальнейшие шаги

* [README Shortcode Core](https://github.com/getgrav/grav-plugin-shortcode-core) описывает ресурсы, CLI-команду `bin/plugin shortcode-core display` и расширенные сырые обработчики.
* Плагин [Shortcode UI](https://github.com/getgrav/grav-plugin-shortcode-ui) — хороший пример шорткодов на Twig с более богатой разметкой.
* Если Twig всё ещё нужен прямо на странице, см. [Twig в контенте](../twig-in-content/index.md), как его включить и настроить песочницу.
