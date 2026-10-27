---
title: "Editor Pro"
description: "Полное руководство по использованию и расширению Editor Pro для Grav."
tags:
  - "Grav Premium"
  - "Editor Pro"
  - "Grav 1.7"
---

# Editor Pro

> [!IMPORTANT] Важно
> Продукты Premium требуют бесплатного плагина [License Manager](../license-manager/index.md). Установите его и добавьте лицензию продукта перед установкой этого продукта.

## Установка

Сначала убедитесь, что у вас установлена последняя версия **Grav 1.7** (`-f` принудительно обновляет индекс GPM).

```
$ bin/gpm selfupgrade -f
```

Для полной функциональности Editor Pro требуется Grav 1.7.0 или более поздняя версия.

### Через GPM (рекомендуется)

```
$ bin/gpm install editor-pro
```

### Через плагин Admin

1. Перейдите в раздел **Плагины** на боковой панели админки
2. Нажмите кнопку **Добавить** (значок плюса)
3. Найдите «Editor Pro»
4. Нажмите **Установить**
5. Нажмите **Включить**, чтобы активировать

### Установка вручную

1. Скачайте плагин из своей учётной записи Grav Premium
2. Распакуйте zip-архив в `/user/plugins/`
3. Переименуйте папку в `editor-pro`
4. Очистите кэш: `bin/grav cache`

## Конфигурация

Перед настройкой плагина скопируйте `user/plugins/editor-pro/editor-pro.yaml` в `user/config/plugins/editor-pro.yaml` и редактируйте только эту копию.

Ниже приведена конфигурация по умолчанию и описание доступных параметров:

```yaml
enabled: true
default_for_all: false
toolbar: 'undo,redo,|,removeformat,|,heading,bold,italic,underline,strikethrough,|,link,image,|,blockquote,bulletList,orderedList,|,codeBlock,table,|,htmlBlock,shortcodeBlock'
extra_typography:
  enabled: true
  custom: []
```

### Параметры конфигурации

* `enabled` — включить или отключить плагин глобально
* `default_for_all` — назначить Editor Pro редактором по умолчанию для всех пользователей
* `toolbar` — настроить, какие инструменты отображаются на панели инструментов
* `extra_typography.enabled` — включить автоматические типографские преобразования
* `extra_typography.custom` — задать пользовательские замены текста

## Настройка редактора по умолчанию

Editor Pro можно настроить как редактор по умолчанию на нескольких уровнях:

### Глобальное значение по умолчанию (все пользователи)

`![По умолчанию для всех](global-default.webp?class=image-shadow)`

Включите параметр «Default for All Users» в конфигурации плагина, чтобы Editor Pro стал редактором по умолчанию для всех.

### Выбор отдельным пользователем

`![Выбор редактора в профиле](per-user-default.webp?class=image-shadow)`

Пользователи могут выбрать предпочитаемый редактор в настройках профиля:

1. Нажмите на свой аватар в админке
2. Выберите «Preferences»
3. Выберите «Editor Pro» в выпадающем списке «Редактор»
4. Сохраните настройки

### Настройка для отдельной схемы

Укажите Editor Pro для конкретных типов страниц в схемах:

```yaml
form:
  fields:
    header.content:
      type: editor-pro
      label: Content
      validate:
        required: true
```

## Сочетания клавиш

### Сочетания клавиш для разных платформ

| Действие    | Windows/Linux         | Mac                 |
|-----------|-----------------------|---------------------|
| Сохранить      | `Ctrl` + `S`          | `⌘` + `S`           |
| Копировать      | `Ctrl` + `C`          | `⌘` + `C`           |
| Вставить     | `Ctrl` + `V`          | `⌘` + `V`           |
| Вырезать       | `Ctrl` + `X`          | `⌘` + `X`           |
| Отменить      | `Ctrl` + `Z`          | `⌘` + `Z`           |
| Повторить      | `Ctrl` + `Y`          | `⌘` + `Y`           |
|           | `Ctrl` + `Shift` + `Z`| `⌘` + `Shift` + `Z` |
| Полужирный      | `Ctrl` + `B`          | `⌘` + `B`           |
| Курсив    | `Ctrl` + `I`          | `⌘` + `I`           |
| Подчёркивание | `Ctrl` + `U`          | `⌘` + `U`           |
| Зачёркивание    | `Ctrl` + `Shift` + `X`| `⌘` + `Shift` + `X` |
| Ссылка      | `Ctrl` + `K`          | `⌘` + `K`           |

### Сочетания клавиш для заголовков

| Действие    | Windows/Linux         | Mac                 |
|-----------|-----------------------|---------------------|
| Заголовок 1 | `Ctrl` + `Alt` + `1`  | `⌘` + `Option` + `1`   |
| Заголовок 2 | `Ctrl` + `Alt` + `2`  | `⌘` + `Option` + `2`   |
| Заголовок 3 | `Ctrl` + `Alt` + `3`  | `⌘` + `Option` + `3`   |
| Заголовок 4 | `Ctrl` + `Alt` + `4`  | `⌘` + `Option` + `4`   |
| Заголовок 5 | `Ctrl` + `Alt` + `5`  | `⌘` + `Option` + `5`   |
| Заголовок 6 | `Ctrl` + `Alt` + `6`  | `⌘` + `Option` + `6`   |

### Сочетания клавиш для списков и блоков

| Действие         | Windows/Linux           | Mac                   |
|----------------|-------------------------|-----------------------|
| Маркированный список    | `Ctrl` + `Shift` + `8`  | `⌘` + `Shift` + `8`   |
| Нумерованный список  | `Ctrl` + `Shift` + `7`  | `⌘` + `Shift` + `7`   |
| Цитата     | `Ctrl` + `Shift` + `B`  | `⌘` + `Shift` + `B`   |
| Блок кода     | `Ctrl` + `Alt` + `C`    | `⌘` + `Option` + `C`     |
| Вставка шорткода| `Ctrl` + `Shift` + `S` | `⌘` + `Shift` + `S`   |

### Сочетания клавиш навигации

| Действие                      | Клавиатура                |
|-----------------------------|-------------------------|
| Переход к следующему блоку          | `Tab` (когда применимо) |
| Переход к предыдущему блоку      | `Shift` + `Tab`         |
| Выход из блока (в абзац)   | `Enter` (дважды)         |
| Навигация между блоками      | `↑` / `→` / `↓` / `←`  |
| Выделить всё                  | `Ctrl/⌘` + `A`          |
| Поиск                        | `Ctrl/⌘` + `F`          |

### Навигация по таблице

| Действие                           | Клавиатура            |
|----------------------------------|---------------------|
| Следующая ячейка                        | `Tab`               |
| Предыдущая ячейка                    | `Shift` + `Tab`     |
| Новая строка (в последней ячейке)     | `Tab`               |
| Навигация по ячейкам                   | `↑` / `→` / `↓` / `←` |
| Выход из таблицы                       | `Ctrl/⌘` + `Enter`  |

## Настройка панели инструментов

`![Настройка панели инструментов](toolbar-config.webp?class=image-shadow)`

Панель инструментов можно настроить так, чтобы на ней отображались только нужные инструменты. Доступные элементы панели инструментов:

### Форматирование текста
* `bold` — полужирный текст
* `italic` — курсив
* `underline` — подчёркнутый текст
* `strikethrough` — зачёркнутый текст
* `code` — встроенный код
* `subscript` — текст нижним индексом
* `superscript` — текст верхним индексом
* `removeformat` — очистка форматирования

### Структура
* `heading` — выпадающий список заголовков (H1–H6)
* `paragraph` — формат абзаца
* `blockquote` — цитата
* `horizontalRule` — горизонтальная линия

### Списки
* `bulletList` — маркированный список
* `orderedList` — нумерованный список

### Контент
* `link` — вставка и изменение ссылки
* `image` — вставка изображения
* `table` — вставка таблицы
* `codeBlock` — блок кода с подсветкой синтаксиса

### Специальные блоки
* `htmlBlock` — сохранение блока HTML
* `shortcodeBlock` — вставка шорткода
* `twigBlock` — сохранение шаблона Twig
* `githubAlert` — оповещения в стиле GitHub

### История
* `undo` — отменить последнее действие
* `redo` — повторить последнее действие

### Разделители
* `|` — визуальный разделитель на панели инструментов

Пример конфигурации панели инструментов:
```yaml
toolbar: 'undo,redo,|,heading,bold,italic,|,link,image,|,bulletList,orderedList'
```

## Типы контента

### Расширенное форматирование текста

Editor Pro поддерживает всестороннее форматирование текста:

* **Полужирный** — сильное выделение (`**text**` или `__text__`)
* **Курсив** — выделение (`*text*` или `_text_`)
* **Подчёркивание** — подчёркнутый текст (`<u>text</u>`)
* **Зачёркивание** — зачёркнутый текст (`~~text~~`)
* **Нижний индекс** — текст нижним индексом (`<sub>text</sub>`)
* **Верхний индекс** — текст верхним индексом (`<sup>text</sup>`)
* **Код** — встроенный код (`` `code` ``)

### Заголовки

Поддерживаются шесть уровней заголовков (H1–H6). Используйте выпадающий список на панели инструментов или сочетания клавиш:

```markdown
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

### Списки {: #lists-1 }

#### Маркированные списки
```markdown
* First item
* Second item
  * Nested item
  * Another nested item
* Third item
```

#### Нумерованные списки
```markdown
1. First item
2. Second item
   1. Nested item
   2. Another nested item
3. Third item
```

### Ссылки

Ссылки поддерживают заголовки и могут быть внутренними или внешними:

```markdown
[Link text](https://example.com "Optional title")
[Internal link](/path/to/page)
[Reference link][ref]

[ref]: https://example.com "Reference title"
```

### Изображения

Изображения поддерживают alt-текст и заголовки:

```markdown
![Alt text](image.jpg "Optional title")
![](path/to/image.png)
```

Перетаскивайте изображения прямо в редактор для автоматической загрузки и вставки.

### Таблицы

Полноценная поддержка таблиц с визуальным редактированием:

```markdown
| Header 1 | Header 2 | Header 3 |
|----------|----------|----------|
| Cell 1   | Cell 2   | Cell 3   |
| Cell 4   | Cell 5   | Cell 6   |
```

Фичи работы с таблицами:
* Добавление и удаление строк и столбцов
* Объединение и разделение ячеек
* Назначение строк и столбцов заголовками
* Изменение ширины столбцов перетаскиванием
* Навигация с помощью Tab/Shift+Tab

### Блоки кода

Блоки кода с подсветкой синтаксиса и выбором языка:

````markdown
```javascript
function hello() {
    console.log("Hello, World!");
}
```
````

Поддерживаемые языки включают:
* JavaScript/TypeScript
* PHP
* Python
* HTML/CSS
* Markdown
* YAML/JSON
* И многие другие…

### Цитаты

Вложенные цитаты с корректным форматированием:

```markdown
> This is a blockquote
> 
> > This is nested
> > 
> > > Even deeper nesting
```

### Оповещения GitHub

Встроенная поддержка блоков оповещений в стиле GitHub:

```markdown
> [!NOTE]
> Useful information that users should know.

> [!TIP]
> Helpful advice for doing things better.

> [!IMPORTANT]
> Key information users need to know.

> [!WARNING]
> Urgent info that needs immediate attention.

> [!CAUTION]
> Advises about risks or negative outcomes.
```

## Визуальные блоки

Editor Pro сохраняет специальный контент в виде визуальных блоков, чтобы предотвратить его повреждение:

### Блоки необработанного кода (тёмно-серые)

`![Блоки необработанного кода](raw-code.webp?class=image-shadow)`

Необработанный код — это контент **HTML** или **Twig**; он сохраняется в виде синих блоков:

* Предотвращает случайное изменение
* Сохраняет исходное форматирование
* Редактирование на месте с подсветкой синтаксиса
* Поддерживает любой допустимый HTML или Twig, начинающийся с новой строки

Пример:
```twig
{% for item in page.collection %}
  <h2>{{ item.title }}</h2>
  {{ item.content|raw }}
{% endfor %}
```

Пример:
```html
<div class="custom-widget">
  <h3>Custom HTML</h3>
  <p>This HTML is preserved exactly as written.</p>
</div>
```

### Блоки шорткодов (зелёные)

`![Блоки шорткодов](shortcode.webp?class=image-shadow)`

Шорткоды отображаются в виде зелёных блоков с визуальными индикаторами:

* Показывает имя шорткода и параметры
* Нажмите, чтобы изменить через модальное окно
* Поддерживает вложенные шорткоды
* Предпросмотр в реальном времени, когда доступен

Пример:

```
[columns count="2"]  
[column]First column[/column]  
[column]Second column[/column]  
[/columns]  
```

## Типографские преобразования

`![Настройки типографики](transforms.webp?class=image-shadow)`

Editor Pro может автоматически преобразовывать текстовые шаблоны по мере ввода.

### Встроенные преобразования

#### Типографика
* `--` → — (длинное тире)
* `...` → … (многоточие)
* `(c)` → © (знак копирайта)
* `(r)` → ® (зарегистрированный знак)
* `(tm)` → ™ (торговая марка)

#### Кавычки
* `"text"` → "text" (умные кавычки)
* `'text'` → 'text' (умные одинарные кавычки)

#### Стрелки
* `->` → →
* `<-` → ←
* `<->` → ↔
* `=>` → ⇒

#### Дроби
* `1/2` → ½
* `1/3` → ⅓
* `1/4` → ¼
* `3/4` → ¾

#### Математические символы
* `+-` → ±
* `!=` → ≠
* `<=` → ≤
* `>=` → ≥

### Пользовательские преобразования

Добавьте собственные преобразования в конфигурации:

```yaml
extra_typography:
  enabled: true
  custom:
    ':)': '😊'
    ':(': '😢'
    ':D': '😃'
    '->': '→'
    '<3': '❤️'
```

### Преобразования для отдельных языков

Настройте преобразования для каждого языка:

```yaml
extra_typography:
  languages:
    fr:
      quotes: ['« ', ' »']
      custom:
        'oe': 'œ'
    de:
      quotes: ['„', '"']
```

## Интеграция шорткодов

При использовании с плагином shortcode-core Editor Pro предоставляет расширенную поддержку шорткодов.

### Модальное окно шорткодов

`![Модальное окно шорткодов](shortcode-modal.webp?class=image-shadow&width=600px)`

Нажмите `Ctrl/⌘ + Shift + S`, чтобы открыть модальное окно шорткодов:

1. **Поиск** шорткодов по имени или описанию
2. **Настройка** параметров через поля формы
3. **Предпросмотр** результата (если предпросмотр доступен)
4. **Вставка** шорткода в ваш контент

### Поддерживаемые типы полей

* **Текст** — однострочное поле ввода
* **Многострочный текст** — многострочное поле ввода
* **Выбор** — выбор из выпадающего списка
* **Флажок** — логический переключатель
* **Число** — числовой ввод
* **Цвет** — выбор цвета
* **Дата** — выбор даты
* **Файл** — выбор файла или медиафайла
* **Страница** — выбор страницы

### Вложенные шорткоды

Сложные шорткоды с иерархией «родитель — потомок»:

```
[tabs]
  [tab title="First"]
    First tab content
  [/tab]
  [tab title="Second"]
    Second tab content
  [/tab]
[/tabs]
```

### Предпросмотр в реальном времени

Если включено, шорткоды отображают предпросмотр в реальном времени в редакторе:

```yaml
plugins:
  shortcode-core:
    live_preview: true
    preview_delay: 500  # миллисекунды
```

## Расширение Editor Pro

### Интеграция плагинов

Зарегистрируйте свой плагин в Editor Pro:

```php
<?php
namespace Grav\Plugin;

use Grav\Common\Plugin;

class YourPlugin extends Plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'registerEditorProPlugin' => ['registerEditorProPlugin', 0]
        ];
    }

    public function registerEditorProPlugin($event)
    {
        $plugins = $event['plugins'];
        
        // Добавить CSS
        $plugins['css'][] = 'plugin://your-plugin/assets/editor-pro.css';
        
        // Добавить JavaScript
        $plugins['js'][] = 'plugin://your-plugin/assets/editor-pro.js';
        
        $event['plugins'] = $plugins;
        return $event;
    }
}
```

### JavaScript API

Обращайтесь к API Editor Pro в своём JavaScript:

```javascript
window.EditorPro.registerPlugin({
    name: 'your-plugin',
    
    init(editorPro) {
        const { editor, toolbar, schema } = editorPro;
        
        // Добавить пользовательскую кнопку
        toolbar.addButton({
            name: 'customButton',
            icon: '🎨',
            title: 'Custom Action',
            action: () => {
                // Ваше пользовательское действие
                editor.chain()
                    .focus()
                    .insertContent('Custom content')
                    .run();
            }
        });
        
        // Слушать события редактора
        editor.on('update', ({ editor }) => {
            console.log('Content updated');
        });
        
        // Доступ к состоянию редактора
        editor.on('selectionUpdate', ({ editor }) => {
            const { from, to } = editor.state.selection;
            console.log(`Selection: ${from} to ${to}`);
        });
    }
});
```

### Пользовательские узлы

Создавайте пользовательские узлы TipTap:

```javascript
import { Node } from '@tiptap/core';

export const CustomWidget = Node.create({
    name: 'customWidget',
    
    group: 'block',
    
    content: 'inline*',
    
    addAttributes() {
        return {
            type: {
                default: 'default',
            },
        };
    },
    
    parseHTML() {
        return [
            {
                tag: 'div[data-widget]',
            },
        ];
    },
    
    renderHTML({ HTMLAttributes }) {
        return ['div', { 'data-widget': '', ...HTMLAttributes }, 0];
    },
    
    addCommands() {
        return {
            setWidget: (attributes) => ({ commands }) => {
                return commands.setNode(this.name, attributes);
            },
        };
    },
});
```

### Пользовательские марки

Создавайте пользовательские текстовые марки:

```javascript
import { Mark } from '@tiptap/core';

export const Highlight = Mark.create({
    name: 'highlight',
    
    addAttributes() {
        return {
            color: {
                default: 'yellow',
            },
        };
    },
    
    parseHTML() {
        return [
            {
                tag: 'mark',
            },
        ];
    },
    
    renderHTML({ HTMLAttributes }) {
        return ['mark', HTMLAttributes, 0];
    },
    
    addCommands() {
        return {
            toggleHighlight: (attributes) => ({ commands }) => {
                return commands.toggleMark(this.name, attributes);
            },
        };
    },
});
```

## Расширенная конфигурация

### Конфигурация для отдельной страницы

Управляйте поведением Editor Pro для каждой страницы:

```yaml
---
title: My Page
editor_pro:
  enabled: true
  toolbar: 'bold,italic,link'
  preserve_html: false
  preserve_twig: true
  typography:
    enabled: false
---
```

### Параметры поля схемы

Расширенная конфигурация поля в схемах:

```yaml
form:
  fields:
    content:
      type: editor-pro
      label: Content
      toolbar: 'heading,bold,italic,link,image'
      height: 500
      autofocus: true
      placeholder: 'Start writing...'
      typography:
        enabled: true
        arrows: true
        quotes: true
      preserve:
        html: true
        shortcodes: true
        twig: false
```

### Правила сохранения контента

Настройте, какие типы контента сохранять:

```yaml
plugins:
  editor-pro:
    preserve:
      html:
        enabled: true
        tags: ['script', 'style', 'iframe']
      shortcodes:
        enabled: true
        list: ['notice', 'columns', 'tabs']
      twig:
        enabled: true
        patterns: ['{{', '{%', '{#']
```

## Оптимизация производительности

### Большие документы

Для документов объёмом более 50 000 слов:

```yaml
plugins:
  editor-pro:
    performance:
      lazy_blocks: true
      debounce_save: 1000
      max_history: 50
```

### Управление памятью

Оптимизируйте использование памяти:

```yaml
plugins:
  editor-pro:
    performance:
      clear_history_on_save: true
      compress_history: true
      max_document_size: 5000000  # 5MB
```

## Решение проблем

### Редактор не появляется

1. **Проверьте, что плагин включён**
   ```bash
   bin/gpm status editor-pro
   ```

2. **Очистите кэш**
   ```bash
   bin/grav cache
   ```

3. **Проверьте настройку пользователя**
   - Проверьте выбор редактора в профиле пользователя
   - Убедитесь, что выбран «Editor Pro»

4. **Проверьте консоль браузера**
   - Откройте инструменты разработчика (F12)
   - Найдите ошибки JavaScript
   - Сообщите об обнаруженных ошибках

### Контент не сохраняется

1. **Проверьте права доступа**
   ```bash
   ls -la user/pages/
   ```

2. **Проверьте обработку формы**
   - Проверьте наличие ошибок валидации формы
   - Убедитесь, что обязательные поля заполнены

3. **Протестируйте с простым контентом**
   - Попробуйте сохранить только простой текст
   - Постепенно добавляйте сложный контент

### Шорткоды не работают

1. **Проверьте, что shortcode-core установлен**
   ```bash
   bin/gpm status shortcode-core
   ```

2. **Проверьте регистрацию шорткодов**
   - Убедитесь, что шорткоды зарегистрированы правильно
   - Проверьте, что слушатели событий настроены

3. **Включите режим отладки**
   ```yaml
   plugins:
     editor-pro:
       debug: true
   ```

### Проблемы с производительностью

1. **Проверьте размер документа**
   - Большим документам могут потребоваться настройки оптимизации
   - Подумайте о разделении очень длинного контента

2. **Отключите предпросмотр в реальном времени**
   ```yaml
   plugins:
     shortcode-core:
       live_preview: false
   ```

3. **Сократите число элементов панели инструментов**
   - Включайте только необходимые инструменты
   - Сведите к минимуму пользовательские расширения

### Совместимость с браузерами

1. **Проверьте версию браузера**
   - Убедитесь, что браузер соответствует минимальным требованиям
   - При необходимости обновите до последней версии

2. **Отключите расширения браузера**
   - Блокировщики рекламы могут мешать
   - Попробуйте режим инкогнито или приватный режим

3. **Проверьте в другом браузере**
   - Попробуйте Chrome, если используете Firefox
   - Убедитесь, что проблема связана с конкретным браузером

## Руководство по миграции

### С NextGen Editor

1. **Установите Editor Pro**
   ```bash
   bin/gpm install editor-pro
   ```

2. **Обновите настройки пользователей**
   - Измените выбор редактора на «Editor Pro»
   - Миграция контента не требуется

3. **Обновите пользовательские интеграции**
   - Измените имена событий с `registerNextGenEditorPlugin` на `registerEditorProPlugin`
   - Обновите вызовы JavaScript API

4. **Тщательно протестируйте**
   - Проверьте все типы страниц
   - Убедитесь в работоспособности шорткодов
   - Проверьте пользовательские расширения

### С редактора Markdown

1. **Контент полностью совместим**
   - Преобразование не требуется
   - Структура Markdown сохраняется

2. **Включайте постепенно**
   - Начните с отдельных типов страниц
   - Постепенно внедряйте у пользователей

3. **Обучите пользователей**
   - Предоставьте справочник по сочетаниям клавиш
   - Объясните концепцию визуальных блоков

## Справочник API

### События PHP

#### registerEditorProPlugin

Регистрация ресурсов и конфигурации:

```php
public function registerEditorProPlugin($event)
{
    $plugins = $event['plugins'];
    $config = $event['config'];
    
    // Добавить ресурсы
    $plugins['js'][] = 'plugin://your-plugin/editor.js';
    $plugins['css'][] = 'plugin://your-plugin/editor.css';
    
    // Добавить конфигурацию
    $config['custom_setting'] = true;
    
    $event['plugins'] = $plugins;
    $event['config'] = $config;
    return $event;
}
```

#### onEditorProSave

Обработка контента перед сохранением:

```php
public function onEditorProSave($event)
{
    $content = $event['content'];
    $page = $event['page'];
    
    // Обработать контент
    $content = $this->processContent($content);
    
    $event['content'] = $content;
    return $event;
}
```

### JavaScript API {: #javascript-api-1 }

#### Экземпляр редактора

Доступ к редактору TipTap:

```javascript
const editor = window.EditorPro.getEditor(fieldId);

// Получить/установить контент
editor.getHTML();
editor.getJSON();
editor.getText();
editor.setContent(content);

// Команды
editor.chain().focus().bold().run();
editor.chain().focus().insertContent('text').run();

// Состояние
editor.isActive('bold');
editor.can().bold();
```

#### API панели инструментов

Управление элементами панели инструментов:

```javascript
const toolbar = window.EditorPro.getToolbar(fieldId);

// Добавить кнопку
toolbar.addButton({
    name: 'custom',
    icon: '🎨',
    title: 'Custom',
    action: callback
});

// Удалить кнопку
toolbar.removeButton('custom');

// Обновить кнопку
toolbar.updateButton('custom', {
    disabled: true
});
```

#### API схемы

Доступ к схеме документа и её изменение:

```javascript
const schema = window.EditorPro.getSchema(fieldId);

// Зарегистрировать узел
schema.registerNode('customNode', nodeSpec);

// Зарегистрировать марку
schema.registerMark('customMark', markSpec);

// Получить узел/марку
schema.getNode('paragraph');
schema.getMark('bold');
```

## Поддержка

### Как получить помощь

1. **Документация** — это подробное руководство
2. **Задачи GitHub** — [сообщить об ошибках](https://github.com/getgrav/grav-premium-issues/issues)
3. **Discord** — [присоединиться к Grav Discord](https://discord.gg/5VhYVkR)
4. **Форум** — [форум Grav](https://discourse.getgrav.org/)

### Сообщение о проблемах

При сообщении о проблемах указывайте:

1. Версия Grav (`bin/grav version`)
2. Версия Editor Pro
3. Браузер и его версия
4. Шаги для воспроизведения
5. Сообщения об ошибках из консоли
6. Соответствующая конфигурация

### Запросы фич

Отправляйте запросы на новые фичи на GitHub с указанием:

1. Описание сценария использования
2. Ожидаемое поведение
3. Макеты или примеры, если применимо
4. Приоритет и срочность

## Благодарности

- **TipTap** — современный фреймворк редактора
- **ProseMirror** — базовый движок редактора
- **Grav Team** — платформа CMS
- **Trilby Media** — разработка плагина
