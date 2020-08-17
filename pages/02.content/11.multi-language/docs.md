---
title: Многоязычность
page-toc:
  active: true
taxonomy:
    category: docs
---

Поддержка нескольких языков в Grav является прямым результатом [обсуждения в сообществе](https://github.com/getgrav/grav/issues/170) по этой теме. Теперь мы разберем их и предоставим примеры того, как вы можете настроить свой сайт Grav на нескольких языках.

## Один язык, отличный от английского

Если вы используете только один язык, включите переводы и добавьте код своего языка в файл `user/config/system.yaml`:

[prism classes="language-yaml line-numbers"]
languages:
  translations: true
  supported:
    - fr
[/prism]

или в конфигурации системы в админке:

![Настройки перевода](translations-settings.png)

Это гарантирует, что Grav использует правильные языковые строки во внешнем интерфейсе.
Кроме того, если тема поддерживает это, она добавит код вашего языка в тег HTML.

## Основы многоязычия

Поскольку вы уже должны быть знакомы с тем, как Grav использует файлы разметки в папках для определения архитектурной структуры, а также для установки важных опций страницы и ее содержимого, мы не будем углубляться в эту механику напрямую. Тем не менее, имейте в виду, что по умолчанию Grav ищет **один файл** `.md` в папке, представляющей страницу. При включенной многоязычной поддержке Grav будет искать соответствующий языковой файл, например, `default.en.md` или `default.fr.md`.

### Конфигурация языка

Для того, чтобы Grav сделала это, вы должны сначала установить некоторую базовую языковую конфигурацию в вашем файле `user/config/system.yaml`.

[prism classes="language-yaml line-numbers"]
languages:
  supported:
    - en
    - fr
[/prism]

Предоставляя блок `языков` со списком `поддерживаемых` языков, Вы эффективно включили поддержку нескольких языков в Grav.

В этом примере видно, что были описаны два поддерживаемых языка (`en` и `fr`). Это позволит вам поддерживать **английский** и **французский** языки.

Если язык не запрашивается явно (через URL или код), Grav будет использовать указанный порядок языков для выбора правильного языка. Итак, в приведенном выше примере **языком по умолчанию** является `en` или английский. Если бы у вас сначала был `fr`, французский язык был бы языком по умолчанию.

!! Конечно, Вы можете предоставить столько языков, сколько захотите, и Вы даже можете использовать локальные коды типа `en-GB`, `en-US` и `fr-FR`.  Если вы используете это именование на основе локали, вам придется заменить все короткие языковые коды на локальные версии.

### Многоязычные страницы

По умолчанию в Grav каждая страница представлена одним файлом, например, `default.md`. Когда вы включите поддержку нескольких языков, Grav будет искать файл с соответствующим названием.  Например, поскольку английский язык является нашим языком по умолчанию, он сначала будет искать `default.en.md`.

Если этот файл не найден, он попробует следующий язык и будет искать `default.fr.md`. Если этот файл не будет найден, он вернется к настройкам по умолчанию Grav и будет искать `default.md` для предоставления информации для страницы.

Если бы у нас были самые базовые Grav-сайты, с одним файлом `01.home/default.md`, мы могли бы начать с переименования `default.md` в `default.en.md`, и его содержимое могло бы выглядеть так:

[prism classes="language-markdown line-numbers"]
---
title: Homepage
---

This is my Grav-powered homepage!
[/prism]

Затем вы можете создать новую страницу, расположенную в той же папке `01.home/` под названием `default.fr.md` с содержимым:

[prism classes="language-markdown line-numbers"]
---
title: Page d'accueil
---

Ceci est ma page d'accueil générée par Grav !
[/prism]

Теперь Вы определили две страницы для Вашей текущей домашней страницы на нескольких языках.

### Активный язык через URL-адрес

Поскольку английский язык является языком по умолчанию, если бы вы указали ваш браузер без указания языка, вы бы получили содержимое, как описано в файле `default.en.md`, но вы также могли бы явно запросить английский, указав вашему браузеру на

[prism classes="language-text line-numbers"]
http://yoursite.com/en
[/prism]

To access the French version, you would of course, use

[prism classes="language-text line-numbers"]
http://yoursite.com/fr
[/prism]

### Активный язык через браузер

Большинство браузеров позволяют настроить, на каких языках вы предпочитаете просматривать содержимое. Grav имеет возможность прочитать эти `http_accept_language` значения и сравнить их с текущими поддерживаемыми языками сайта, и если определенный язык не был обнаружен, показать вам содержимое на предпочтительном для вас языке.

Для этого вы должны включить опцию в вашем файле `user/system.yaml` в секции `languages:`:

[prism classes="language-yaml line-numbers"]
languages:
  http_accept_language: false
[/prism]

### Установка локали для активного языка

Настройка булевых параметров установит метод PHP `setlocale()`, который контролирует такие вещи, как денежные значения, даты, сравнение строк, классификацию символов и другие специфические для локали настройки в соответствии с настройками активного языка. По умолчанию это значение установлено в `false`, а затем оно будет использовать системную локаль, если вы установите это значение в `true`, то оно переопределит локаль с текущим активным языком.

[prism classes="language-yaml line-numbers"]
languages:
   override_locale: false
[/prism]

### Префикс языка по умолчанию

По умолчанию код языка по умолчанию префиксован во всех URL. Например, если у вас есть поддержка английского и французского языков (`en` и `fr`), и по умолчанию английский. Маршрут страницы может выглядеть как `/en/my-page` на английском языке и `/fr/ma-page` на французском. Однако часто предпочтительнее иметь язык по умолчанию без префикса, так что вы можете просто установить этот параметр в `false` и страница на английском языке будет выглядеть как `/my-page`.

[prism classes="language-yaml line-numbers"]
languages:
    include_default_lang: false
[/prism]

### Мультиязыковая маршрутизация

Grav обычно использует имена папок для создания URL маршрута для определенной страницы. Это позволяет легко понять и реализовать архитектуру сайта в виде вложенного набора папок. Однако при создании мультиязычного сайта вы можете использовать URL-адрес, который имеет больший смысл в данном конкретном языке.

Если бы у нас была следующая структура папок:

[prism classes="language-yaml line-numbers"]
- 01.Animals
  - 01.Mammals
    - 01.Bats
    - 02.Bears
    - 03.Foxes
    - 04.Cats
  - 02.Reptiles
  - 03.Birds
  - 04.Insets
  - 05.Aquatic
[/prism]

Это приведет к появлению таких URL, как `http://yoursite.com/animals/mammals/bears`. Это отлично подходит для английского сайта, но если вы хотите иметь французскую версию, вы предпочитаете, чтобы они были переведены соответствующим образом. Самый простой способ достичь этого - добавить пользовательский [слаг](.../headers#slug) для каждого файла страницы `fr.md`. Например, страница с млекопитающими может выглядеть как-то так:

[prism classes="language-markdown line-numbers"]
---
title: Mammifères
slug: mammiferes
---

Les mammifères (classe des Mammalia) forment un taxon inclus dans les vertébrés, traditionnellement une classe, définie dès la classification de Linné. Ce taxon est considéré comme monophylétique...
[/prism]

Это в сочетании с соответствующими **переопределяющими слагами** в других файлах должно привести к URL `http://yoursite.com/animaux/mammiferes/ours`, который гораздо больше похож на французский!

Другим вариантом является использование новой поддержки [маршрутов на уровне страниц](.../headers#routes) и предоставление полного псевдонима маршрута для страницы.

### Домашняя страница на английском языке

Если вы переопределите маршрут/слаг для главной страницы, Grav не сможет найти главную страницу, как это определено вашей опцией `home.alias` в вашем `system.yaml`. Он будет искать `/homepage`, а у вашей французской домашней страницы может быть маршрут `/page-d-accueil`.

Для поддержки мультиязычных домашних страниц в Grav появилась новая опция, которая может быть использована вместо `home.alias` и это простое `home.aliases`, и это может выглядеть примерно так:

[prism classes="language-yaml line-numbers"]
home:
  aliases:
    en: /homepage
    fr: /page-d-accueil
[/prism]

Таким образом, Grav знает, как добраться до главной страницы, если активным языком является английский или французский.

### Языковые шаблоны Twig

По умолчанию Grav использует имя файла Markdown для определения шаблона Twig, который будет использоваться для рендеринга. Это работает с мультиязычностью так же, как и с мультиязычностью.  Например, `default.fr.md` будет искать Twig-файл под названием `default.html.twig` в соответствующих путях к Twig-шаблону текущей темы и любых плагинов, регистрирующих пути к Twig-шаблону.  При использовании нескольких языков Grav также добавляет текущий активный язык в структуру пути. Это означает, что если вам нужен специальный языковой Twig-файл, вы можете просто поместить его в папку с языками корневого уровня.  Например, если ваша текущая тема использует шаблон, расположенный в `templates/default.html.twig`, вы можете создать папку `templates/fr/` и поместить туда ваш французский специфический Twig-файл: `шаблоны/fr/default.html.twig`.

Другая опция, которая требует ручной настройки, это переопределение настройки `template:` в заголовках страниц. Например:

[prism classes="language-yaml line-numbers"]
template: default.fr
[/prism]

При этом будет искаться шаблон, расположенный в `templates/default.fr.html.twig`.

Это дает вам два варианта для предоставления языковых переопределений Twig.

!! Если не предоставлен шаблон Twig для конкретного языка, то будет использоваться шаблон по умолчанию.



### Перевод через Twig

Самый простой способ использовать эти строки перевода в ваших Twig-шаблонах - это использовать Twig-фильтр `|t`. Вы также можете использовать функцию `t()` Twig, но, честно говоря, фильтр чище и делает то же самое:

[prism classes="language-twig line-numbers"]
<h1 id="site-name">{{ "SITE_NAME"|t }}</h1>
<section id="header">
    <h2>{{ "HEADER.MAIN_TEXT"|t }}</h2>
    <h3>{{ "HEADER.SUB_TEXT"|t }}</h3>
</section>
[/prism]

При использовании функции Twig `t()` решение аналогично:

[prism classes="language-twig line-numbers"]
<h1 id="site-name">{{ t("SITE_NAME") }}</h1>
<section id="header">
    <h2>{{ t("HEADER.MAIN_TEXT") }}</h2>
    <h3>{{ t("HEADER.SUB_TEXT") }}</h3>
</section>
[/prism]

Еще один новый фильтр/функция Twig позволяет выполнять переводы из массива.  Это особенно полезно, если у вас есть список значений, таких как месяцы года или дни недели.  Например, скажем, у вас есть этот перевод:

[prism classes="language-yaml line-numbers"]
en:
  GRAV:
    MONTHS_OF_THE_YEAR: [January, February, March, April, May, June, July, August, September, October, November, December]
[/prism]

Вы можете получить подходящий перевод для месяца поста следующим образом:

[prism classes="language-twig line-numbers"]
{{ 'GRAV.MONTHS_OF_THE_YEAR'|ta(post.date|date('n') - 1) }}
[/prism]

Вы также можете использовать это в качестве функции Twig с `ta()`.

### Переводы с переменными

Вы также можете использовать переменные в ваших Twig-переводах, используя синтаксис [PHP-функции sprintf](https://php.net/sprintf):

[prism classes="language-yaml line-numbers"]
SIMPLE_TEXT: There are %d monkeys in the %s
[/prism]

А затем вы можете заполнить эти переменные с помощью Twig:

[prism classes="language-twig line-numbers"]
{{ "SIMPLE_TEXT"|t(12, "London Zoo") }}
[/prism]

в результате получается перевод:

[prism classes="language-text line-numbers"]
There are 12 monkeys in the London Zoo
[/prism]

### Комплексные переводы

Иногда требуется выполнять сложные переводы с заменой на определенные языки. Вы можете использовать всю мощь метода `translate()` с помощью фильтра/функции `tl`. Например:

[prism classes="language-twig line-numbers"]
{{ ["SIMPLE_TEXT", 12, 'London Zoo']|tl(['fr']) }}
[/prism]

Переведет строку `SIMPLE_TEXT` и заменит плейсхолдеры на `12` и `Лондонский зоопарк` соответственно. Также есть массив, переданный с переводами языков, чтобы попробовать в порядке первого поиска-первого использования. Результат будет выведен на французский язык:


[prism classes="language-text line-numbers"]
Il y a 12 singes dans le Zoo de Londres
[/prism]

### Переводы PHP

Также как и Twig-фильтр и функции, вы можете использовать тот же подход в вашем Grav-плагине:

[prism classes="language-php line-numbers"]
$translation = $this->grav['language']->translate(['HEADER.MAIN_TEXT']);
[/prism]

Вы также можете указать язык:

[prism classes="language-php line-numbers"]
$translation = $this->grav['language']->translate(['HEADER.MAIN_TEXT'], 'fr');
[/prism]

Для перевода определенного элемента в использовании массива:

[prism classes="language-php line-numbers"]
$translation = $this->grav['language']->translateArray('GRAV.MONTHS_OF_THE_YEAR', 3);
[/prism]

### Перевод плагинов и тем

Вы также можете предоставить свои собственные переводы в плагинах и темах. Это делается путем создания файла `languages.yaml` в корне вашего плагина или темы (например. `/user/plugins/error/languages.yaml`, или `user/themes/antimatter/languages.yaml`), и должны содержать все поддерживаемые языки, префиксные по языку или локальному коду:

[prism classes="language-yaml line-numbers"]
en:
  PLUGIN_ERROR:
    TITLE: Error Plugin
    DESCRIPTION: The error plugin provides a simple mechanism for handling error pages within Grav.
fr:
  PLUGIN_ERROR:
    TITLE: Plugin d'Erreur
    DESCRIPTION: Le plugin d'erreur fournit un mécanisme simple de manipulation des pages d'erreur au sein de Grav.
[/prism]

! Условием для плагинов является использование PLUGIN_PLUGINNAME.* в качестве префикса для всех строк языка, чтобы избежать конфликтов имен. Темы с меньшей вероятностью приведут к конфликтам языковых строк, но хорошей идеей является префикс строк, добавляемых в темы с помощью THEME_THEMENAME.*.

### Переопределения перевода

Если вы хотите переопределить определенный перевод, просто поместите измененную пару ключ/значение в соответствующий языковой файл в вашей папке `user/languages/`. Например, файл под названием `user/languages/en.yaml` может содержать:

[prism classes="language-yaml line-numbers"]
PLUGIN_ERROR:
  TITLE: My Error Plugin
[/prism]

Это гарантирует, что вы всегда сможете переопределить строку перевода, не путаясь с самими плагинами или темами, а также позволит избежать перезаписи пользовательского перевода при их обновлении.

## Расширенные настройки

### Языковая работа с окружающей средой

Вы можете воспользоваться [конфигурацией окружения Grav](../../advanced/environment-config) для автоматического перенаправления пользователей на правильную версию вашего сайта на основе URL-адреса.  Например, если у вас есть URL типа `http://french.mysite.com`, который является псевдонимом для вашего стандартного `http://www.mysite.com`, вы можете настроить конфигурацию среды:

`/user/french.mysite.com/config/system.yaml`

[prism classes="language-yaml line-numbers"]
languages:
  supported:
    - fr
    - en
[/prism]

При этом используется **инвертированный языковой порядок**, так что по умолчанию язык теперь `fr`, так что французский язык будет отображаться по умолчанию.

### Языковые маршруты псевдонимов

Поскольку каждая страница может иметь свой собственный маршрут, будет сложно переключаться между различными языковыми версиями одной и той же страницы. Однако на объекте Page()**Page.rawRoute()** появился новый метод, который получит один и тот же необработанный маршрут для любого перевода одной страницы на другой язык. Все, что вам нужно будет сделать, это поместить код лэнга вперед, чтобы получить правильный маршрут к определенной языковой версии страницы.

Например, скажем, вы находитесь на странице на английском языке с пользовательским маршрутом:

[prism classes="language-text line-numbers"]
/my-custom-english-page
[/prism]

Французская страница имеет пользовательский маршрут:

[prism classes="language-text line-numbers"]
/ma-page-francaise-personnalisee
[/prism]

Вы можете получить необработанную страницу на английском языке, и это может быть:

[prism classes="language-text line-numbers"]
/blog/custom/my-page
[/prism]

Затем просто добавьте нужный язык, и это будет ваш новый URL:

[prism classes="language-text line-numbers"]
/fr/blog/custom/my-page
[/prism]

Это позволит получить ту же самую страницу, что и `/ma-pagee-francaise-personisee`.

## Поддержка перевода

Grav предоставляет простой, но мощный механизм для предоставления переводов в Twig, а также через PHP для использования в темах и плагинах. Это включено по умолчанию, и будет использовать язык `en`, если не определен ни один язык. Для ручного включения или выключения переводов в вашем `system.yaml` есть настройка:

[prism classes="language-yaml line-numbers"]
languages:
  translations: true
[/prism]

В переводах используется тот же список языков, который определен параметром `languages: supported:` в вашем `system.yaml`.

Система перевода работает аналогично конфигурации Grav, и есть несколько мест и способов предоставления переводов.

Первое место, где Grav ищет файлы перевода, находится в папке `system/languages`. Ожидается, что файлы будут созданы в формате: `en.yaml`, `fr.yaml` и т. д. Каждый файл yaml должен содержать массив или вложенные массивы пар ключ/значение:

[prism classes="language-yaml line-numbers"]
SITE_NAME: My Blog Site
HEADER:
    MAIN_TEXT: Welcome to my new blog site
    SUB_TEXT: Check back daily for the latest news
[/prism]

Для удобства идентификации Grav предпочитает использовать строки на языке с заглавными буквами, поскольку это помогает определить непереведенные строки, а также делает их более понятными при использовании в шаблонах Twig.

Grav имеет возможность вернуться назад через поддерживаемые языки, чтобы найти перевод, если один для активного языка не найден. Это включено по умолчанию, но может быть отключено с помощью опции `translations_fallback`:

[prism classes="language-yaml line-numbers"]
languages:
  translations_fallback: true
[/prism]

!!! Помогите Grav охватить более широкое сообщество пользователей, предоставляя переводы на **вашем языке**. Мы используем [платформу для переводов Crowdin](https://crowdin.com/) для облегчения переводов [ядра Grav](https://crowdin.com/project/grav-core) и [плагина админки Grav](https://crowdin.com/project/grav-admin). [Зарегистрируйтесь](https://crowdin.com/join) и начните переводить сегодня же!

### Сессионный активный язык

Если вы хотите запомнить активный язык независимо от URL, вы можете активировать **сессионное** хранение активного языка. Чтобы это произошло, вы должны убедиться, что включена соответствующая опция в [system.yaml](../../basics/grav-configuration). Затем необходимо включить настройку языка:

[prism classes="language-yaml line-numbers"]
languages:
  session_store_active: true
[/prism]

В этом случае активный язык будет сохранен в сеансе.

### Переключатель языков

Вы можете загрузить простой плагин **переключения языков** через плагин админки или через GPM с помощью:

[prism classes="language-bash command-line"]
bin/gpm install langswitcher
[/prism]

[Документация по конфигурации и использованию находится на GitHub](https://github.com/getgrav/grav-plugin-langswitcher).


### Настройка с языковыми доменами

Настройте свой сайт с помощью [языковой обработки на основе окружения](#environment-based-language-handling), чтобы назначить языки по умолчанию (первый язык) доменам.


Убедитесь, что вариант

[prism classes="language-yaml line-numbers"]
pages.redirect_default_route: true
[/prism]

в вашем `system.yaml` установлено значение `true`.

Добавьте следующее в ваш **.htaccess** файл и примите языковые слизняки и доменные имена к вашим потребностям:

[prism classes="language-htaccess line-numbers"]
# http://www.cheat-sheets.org/saved-copy/mod_rewrite_cheat_sheet.pdf
# http://www.workingwith.me.uk/articles/scripting/mod_rewrite

# handle top level e.g. http://grav-site.com/de
RewriteRule ^en/?$ "http://grav-site.com" [R=302,L]
RewriteRule ^de/?$ "http://grav-site.de" [R=302,L]

# handle sub pages, exclude admin path
RewriteCond %{REQUEST_URI} !(admin) [NC]
RewriteRule ^en/(.*)$ "http://grav-site.com/$1" [R=302,L]
RewriteCond %{REQUEST_URI} !(admin) [NC]
RewriteRule ^de/(.*)$ "http://grav-site.de/$1" [R=302,L]
[/prism]

Если вы знаете, как упростить правила переписывания, пожалуйста, отредактируйте эту страницу на GitHub, нажав на ссылку **Редактировать** в верхней части страницы.

### Языковая логика в шаблонах Twig

Часто возникает необходимость доступа к состоянию и логике языка из шаблонов Twig.  Например, если Вам необходимо получить доступ к определённому файлу изображения, который отличается для определённого языка и имеет различные имена (`myimage.en.jpg` и `myimage.fr.jpg`).

Для отображения правильной версии изображения необходимо знать текущий активный язык.  Это возможно в Grav путем доступа к объекту `Language` через объект `Grav` и вызова соответствующего метода. В приведенном выше примере это может быть достигнуто с помощью следующего кода Twig:

[prism classes="language-twig line-numbers"]
{{ page.media.images['myimage.'~grav.language.getActive~'.jpg'].html }}
[/prism]

Вызов `getActive` в Twig эффективно вызывает `Language->getActive()` для возврата текущего активного кода языка.  Несколько полезных Языковые методы включают в себя:

* `getLanguages()` - Возвращает массив всех поддерживаемых языков
* `getLanguage()` - Возвращает текущий активный, в противном случае возвращает язык по умолчанию
* `getActive()` - Возвращает текущий активный язык
* `getDefault()` - Возвращает язык по умолчанию (первый)

Полный список доступных методов можно посмотреть в файле `\Grav\Common\Language\Language.php`.
