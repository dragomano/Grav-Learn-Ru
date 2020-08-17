---
title: Основное руководство
taxonomy:
    category: docs
---

Предполагая, что вы успешно [установили Grav](../installation) в соответствии с инструкциями, изложенными в предыдущей главе, мы можем продолжить и немного поиграть с этой CMS, чтобы вы почувствовали себя комфортней.

Поскольку Grav не требует базы данных, с ней довольно легко работать, не беспокоясь о возникновении проблем между вашей установкой Grav и любым другим важным источником данных. Если что-то пойдет не так, вы можете очень легко восстановиться.

## Основы содержания

Во-первых, давайте познакомимся с тем, где Grav хранит контент. Подробнее об этом мы узнаем в [следующей главе](../folder-structure), но на данный момент вы должны знать, что весь наш пользовательский контент хранится в папке `user/pages/` вашей установки Grav.

В настоящее время в папке страниц есть две папки: первая называется `01.home`, а вторая — `02.typography`. Числа в названиях папок являются необязательными, но предоставляют несколько полезных вещей.

Во-первых, это позволяет вам четко определить порядок ваших страниц. Например, `01` будет предшествовать `02`, но `00` будет предшествовать `01`.

Другая вещь, которую делает числовая часть имени папки, это явно сообщает Grav, что эта страница должна быть видна в меню. Важно отметить, что числовая часть вплоть до `.` будет удалена из URL.

## Конфигурация домашней страницы

В файле `user/config/system.yaml` есть опция, которая задает расположение __домашней_страницы__, на которую указывает Grav, когда вы ссылаетесь на корень вашего сайта: `https://yoursite.com`.

Если вы изучите этот файл конфигурации в вашей установке, вы увидите, что он уже указывает на псевдоним `/home`. Мы можем оставить это так в этом примере.

## Редактирование страниц

Страницы **Grav** составляются с помощью разметки **Markdown**. Markdown — синтаксис форматирования простого текста, который компьютер может легко проанализировать и преобразовать в HTML.Он использует основные текстовые символы для обозначения представления (например. **жирный**, _курсив_, заголовки, списки и т. д.), облегчая написание без необходимости изучать всю структуру HTML. Преимущества Markdown включают в себя более низкий уровень ошибок, удобочитаемость, простоту изучения и использования и т. д.

На досуге вы можете прочитать [подробное описание доступного синтаксиса] (../../content/markdown) с примерами, ну а пока следуйте дальше.

Откройте домашнюю страницу в текстовом редакторе. Файл, управляющий домашней страницей, находится в папке `user/pages/01.home/` и по умолчанию называется `default.md`. Все содержимое, которое вы создадите, будет сохранено в папке `user/pages/`.

При редактировании страницы в текстовом редакторе содержимое будет выглядеть примерно так:

[div class="no-margin-bottom"]
[prism classes="language-yaml line-numbers"]
---
title: Главная
body_classes: title-center title-h1h2
---
[/prism]
[/div]
[div class="no-margin-top"]
[prism classes="language-markdown line-numbers" ln-start="5"]
# Поздоровайтесь с Grav!
## установка прошла успешно...

Поздравляем! Вы успешно установили **основной пакет Grav**, включающий в себя **страницу-пример** и шаблон **Quark**.

!! Если вы видите ошибку **404** при нажатии кнопки `Типография` в меню, пожалуйста, обратитесь к [руководство по устранению неполадок](https://learn.getgrav.org/troubleshooting/page-not-found).
[/prism]
[/div]

Давайте убедимся в том, как легко писать на Markdown. Всё, что в [заголовках страницы](../../content/headers) находится между символами `---`, называется [YAML](../../advanced/yaml). Этот блок конфигурации, который находится в файле `.md`, обычно известен как **YAML-заголовок**.

[prism classes="language-bash line-numbers"]
title: Главная
body_classes: title-center title-h1h2
[/prism]

Этот блок задает HTML-тег заголовка страницы (текст, который вы видите на вкладке браузера). Вы также можете получить доступ к этому из ваших тем через атрибут `page.title`. Есть [несколько стандартных заголовков](../../content/headers), позволяющих настраивать различные параметры страницы. Другой пример: `menu: что-нибудь`, это позволяет переопределить текст, используемый для отображения имени страницы в меню. По умолчанию Grav будет использовать заголовок в качестве значения пункта меню.

[prism classes="language-markdown line-numbers"]
# Поздоровайтесь с Grav!
## установка прошла успешно...
[/prism]

Символ `#` или `хэши` в Markdown обозначает заголовок в тексте страницы. Одиночный символ `#` с последующим пробелом и текстом преобразуется в элемент `<h1>` в HTML. `##` или двойной хэш будет конвертирован в тег `<h2>`. Так продолжается вплоть до HTML-тега `<h6>`, который, конечно же, будет состоять из шести хэшей: `###### Заголовок H6`.

[prism classes="language-markdown line-numbers"]
Поздравляем! Вы успешно установили **основной пакет Grav**, включающий в себя **страницу-пример** и шаблон **Quark**.
[/prism]

Это простой абзац, который был бы завернут в обычные теги `<p>` при преобразовании в HTML. Маркеры `**` обозначают жирный текст или `<strong>`, ранее `<b>`, в HTML. Курсивный текст обозначается маркерами `_`.

[prism classes="language-markdown line-numbers"]
!! Если вы видите ошибку **404** при нажатии кнопки `Типография` в меню, пожалуйста, обратитесь к [руководство по устранению неполадок](https://learn.getgrav.org/troubleshooting/page-not-found).
[/prism]

В этом разделе используется пользовательская функция markdown, которая предоставляется включенным плагином `markdown-notices`. Он позволяет создавать простые уведомления, добавляя разное количество восклицательных знаков `!` перед текстом (от `!` до `!!!!`).

Этот обзор должен научить вас основам использования Markdown, но не стесняйтесь заглядывать и в более [подробное объяснение](../../content/markdown) при необходимости.

!! Всегда сохраняйте свои `.md`-файлы в кодировке `UTF8`. Это гарантирует, что они будут работать с конкретными языковыми специальными символами.

## Добавление новой страницы

Создать новую страницу в **Grav** очень просто. Просто следуйте этим простым шагам:

1. Перейдите к папке с вашими страницами: `user/pages/` и создайте новую папку. В этом примере мы будем использовать [явный порядок по умолчанию](https://learn.getgrav.org/content/content-pages) и назовем папку `03.mypage`.
2. Запустите текстовый редактор, создайте новый файл и вставьте следующий пример кода:

[div class="no-margin-bottom"]
[prism classes="language-yaml line-numbers"]
---
title: Моя новая страница
---
[/prism]
[/div]
[div class="no-margin-top"]
[prism classes="language-markdown line-numbers" ln-start="4"]
# Моя новая страница!

Это тело **моей новой страницы**, и я легко могу использовать здесь синтаксис _Markdown_.
[/prism]
[/div]

3. Сохраните этот файл в папке `user/pages/03.mypage/` как `default.md`. Это заставит **Grav** визуализировать страницу с использованием шаблона **default** (файл `user/themes/quark/templates/default.html.twig` в текущем шаблоне).
4. Это оно! Перезагрузите браузер, чтобы увидеть новый пункт в верхнем меню.

Страница автоматически отобразится в меню после пункта меню **Типография**. Если вы хотите изменить имя, которое отображается в меню, добавьте: `menu: Моя страница` между черточками в заголовке страницы.

**Поздравляем**, вы успешно создали новую страницу в Grav. С Grav вы можете сделать гораздо больше, поэтому, пожалуйста, продолжайте читать, чтобы узнать о более продвинутых возможностях и всесторонних функциях.

!! Если у вас есть какие-либо проблемы с доступом к этой новой странице, вам либо не хватает файла `.htaccess` (только для веб-сервера Apache), либо вам может потребоваться отредактировать команду `RewriteBase` в файле `.htaccess`. Пожалуйста, обратитесь к главе [Исправление проблем](../../troubleshooting) для получения дополнительной информации.