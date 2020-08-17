---
title: Заголовки / Frontmatter
page-toc:
  active: true
taxonomy:
    category: docs
---

Заголовки (альтернативно известные как frontmatter) в верхней части файла страницы совершенно необязательны, они вообще не нужны для отображения страницы в Grav. В Grav существует 3 основных типа страниц (**Обычная**, **Листинг** и **Модульная**), и каждая из них имеет соответствующие заголовки.

! Заголовки также известны как **Frontmatter страницы** и обычно называются таковыми, чтобы не путать их с HTTP-заголовками.

## Основные заголовки страниц

Есть несколько основных доступных опций заголовка.

### Использование кэша

```yaml
cache_enable: false
```

По умолчанию Grav будет кэшировать содержимое файла подкачки, чтобы гарантировать, что всё будет работать как можно быстрее. Существуют расширенные сценарии, в которых вы не хотите, чтобы страница кэшировалась.

Примером этого может служить использование динамических переменных Twig в вашем контенте. В `cache_enable` переменная позволяет такое поведение, чтобы быть переопределен.  Мы рассмотрим переменные контента Twig в следующей главе. Допустимыми значениями являются `true` или `false`.

### Дата

```yaml
date: 01/01/2014 3:14pm
```

Переменная `date` позволяет специально установить дату, связанную с этой страницей. Это часто используется для указания того, когда была создана запись,и может использоваться для отображения или сортировки по порядку. Если не задано, то по умолчанию используется последнее **время изменения** страницы.

! Даты в форматах `m/d/y` или `d-m-y` можно определить, посмотрев на разделитель между различными компонентами: если разделитель представляет собой косую черту (`/`), то предполагается **американский** `m/d/y`; тогда как если разделитель представляет собой тире (`-`) или точку (`.`), то предполагается **европейский** формат `d-m-y'.

### Меню

```yaml
menu: My Page
```

Переменная `menu` позволяет задать текст, который будет использоваться в навигации. Существует несколько уровней резервных копий для меню, поэтому, если переменная не установлена, Grav попытается использовать переменную `title`.

### Статус публикации

```yaml
published: true
```

По умолчанию любая страница считается **опубликованной**, если не задать вручную `published: false` или если переменная `publish_date` содержит дату из будущего, или `unpublish_date` содержит дату из прошлого. Допустимые значения: `true` или `false`.

### Слаг

```yaml
slug: my-page-slug
```

Переменная `slug` позволяет вам специально задать часть URL-адреса страницы. Например: `http://yoursite.com/my-page-slug-это` будет URL-адрес, если вы поставите `slug` выше.  Если `slug` не установлен на странице, Grav возвращается к использованию имени папки (без каких-либо числовых префиксов).

[Слаги](http://en.wikipedia.org/wiki/Semantic_URL#Slug) обычно полностью строчные, с акцентированными символами, замененными буквами из английского алфавита, и пробелами, замененными тире или подчеркиванием. В то время как будущие версии Grav будут поддерживать пробелы в слагах, наличие пробелов или заглавных букв не рекомендуется.

Например: если заголовок сообщения в блоге - `Пример сообщения в блоге`, то рекомендуемый слаг будет `primer-soobshhenija-v-bloge`.

### Таксономия

```yaml
taxonomy:
    category: blog
    tag: [пример, демо, cms]
```

Очень полезная переменная заголовка `taxonomy` позволяет присваивать значения **taxonomy**, которые вы определили как допустимые типы в [конфигурации сайта](../../basics/grav-configuration#site-configuration).

Если таксономия не определена в этом файле, она будет проигнорирована. В этом примере страница определяется как находящаяся в категории `blog` и имеет теги: `пример`, `демо` и `cms`. Эти таксономии можно использовать для поиска этих страниц с других страниц, плагинов и даже тем. В главе [Таксономия](../taxonomy) эта концепция будет рассмотрена более подробно.

### Заголовок

Если у вас нет заголовков вообще, вы не будете иметь никакого контроля над заголовком страницы, как это показано в браузере и поисковых системах. По этой причине рекомендуется _как минимум_ поместить переменную `title` в начале файла страницы:

```yaml
title: Заголовок страницы
```

Если переменная `title` не задана, Grav имеет запасное решение и попытается использовать заглавную переменную `slug`.

## Дополнительные заголовки

Они по-прежнему важны, но используются реже. Они могут быть использованы для обеспечения расширенной функциональности внутри вашей страницы.

### Окончание URL-адреса

```yaml
append_url_extension: '.json'
```

Позволяет странице переопределить расширение по умолчанию и установить его программно. Он также установит соответствующие атрибуты заголовка для ответа.

### Управление кэшем

```yaml
cache_control: max-age=604800
```

Может быть пустым или иметь одно из [допустимых значений](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) переменной `cache-control`.

### Формат даты

```yaml
dateformat: 'Y-m-d H:i:s'
```

Переопределяет конфигурацию Grav по умолчанию для форматов дат и позволяет устанавливать её на уровне страницы. Вы можете использовать любой из [допустимых форматов дат](https://php.net/manual/en/datetime.formats.date.php).

### Отладчик

Когда вы включаете отладчик через конфигурационный файл `system.yaml`, отладчик будет отображаться на каждой странице. Есть случаи, когда это может быть нежелательно или может привести к конфликтам с выводом. Таким примером является, когда вы запрашиваете страницу, которая предназначена для возвращения отображаемый HTML-код для AJAX-вызова. При этом отладчик не должен быть введен в результирующие данные. Чтобы отключить отладчик на этой странице, вы можете использовать переменную `debugger`:

```yaml
debugger: false
```

### ETag

```yaml
etag: true
```

Включить или отключить на уровне страницы отображение переменной заголовка ETag с уникальным значением. `false` по умолчанию, если только он не переопределен в вашем `system.yaml`.

### Expires

```yaml
expires: 604800
```

Время истечения срока действия страницы в секундах (604800 секунд = 7 дней) (также возможно отключение кэша — `no cache`).

### Внешний URL

```yaml
external_url: https://www.mysite.com/foo/bar
```

Позволяет переопределить динамически сгенерированный URL-адрес тем, который вы явно предоставляете.


### Код ответа HTTP

```yaml
http_response_code: 404
```

Позволяет динамическую настройку кода ответа HTTP.

### Язык

```yaml
language: fr
```

Это позволяет переопределить язык для конкретной страницы.

### Дата изменения

```yaml
last_modified: true
```

Включить или отключить на уровне страницы отображение последней измененной переменной заголовка с измененной датой. `false` по умолчанию, если только он не переопределен в вашем `system.yaml`.

### Lightbox

```yaml
lightbox: true
```

Хотя, строго говоря, это не стандартный заголовок страницы, это обычный способ разрешить загрузку стандартного лайтбокса JavaScript и CSS для страницы. По умолчанию основная тема `antimatter` не загружает предварительные условия для включения возможностей лайтбокса изображений, обязательно установите плагин лайтбокса, такой как **Featherlight**, который доступен через GPM.

### Перенаправление после авторизации

```yaml
login_redirect_here: false
```

Заголовок `login_redirect_here` позволяет вам определить, находится ли кто-то на этой странице после входа в систему через [плагин Login](https://github.com/getgrav/grav-plugin-login). Если установить для этого заголовка значение `false`, кто-нибудь перешлет на предыдущую страницу после успешного входа.

Значение `true` здесь позволит человеку оставаться на текущей странице после успешного входа в систему. Это также настройка по умолчанию, которая применяется, если в frontmatter нет заголовка `login_redirect_here`.

Вы можете переопределить это поведение по умолчанию принудительно установив стандартное расположение указав явный параметр в конфигурации входа YAML:

```yaml
redirect_after_login: '/profile'
```

Это всегда приведет вас к маршруту `/profile` после успешного входа в систему.

### Markdown

```yaml
  markdown:
    extra: false
    auto_line_breaks: false
    auto_url_links: false
    escape_markup: false
    special_chars:
      '>': 'gt'
      '<': 'lt'
```

[div class="table-keycol"]
| Параметр | Описание |
| -------- | ----------- |
| **extra:** | Включить поддержку Markdown Extra (GFM по умолчанию) |
| **auto_line_breaks:** | Включить автоматические разрывы линий |
| **auto_url_links:** | Включить автоматические HTML-ссылки |
| **escape_markup:** | Экранирование тегов разметки в HTML-сущности |
| **special_chars:** | Список специальных символов для автоматического преобразования |
[/div]

Вы можете включить их глобально через файл конфигурации `user/config/system.yaml`, или вы можете переопределить этот глобальный параметр _через_ указанные опции заголовка `markdown`.

### Запрет кэширования Twig

```yaml
never_cache_twig: true
```

Включение этого позволит вам добавить логику обработки, которая может динамически изменяться при каждой загрузке страницы, а не кэшировать результаты и сохранять их для каждой загрузки страницы. Это можно включить/отключить по всему сайту в **system.yaml** или на определенной странице. Может быть установлено `true` или `false`.

Это тонкое изменение, но оно особенно полезно на модульных страницах, так как избавляет вас от необходимости постоянно отключать кэширование при работе с ним. Страница все еще кешируется, но не веточка. Веточка обрабатывается после извлечения кэшированного содержимого. Для модульных форм теперь он работает только с этим параметром, вместо того, чтобы отключать модульный кеш страниц.

!! В настоящее время это несовместимо с `twig_first: true`, потому что вся обработка происходит в одном вызове Twig.

### Обработка текста

```yaml
process:
	markdown: false
	twig: true
```

Обработка страницы - еще одна продвинутая возможность. По умолчанию Grav будет обрабатывать `markdown`, но **не будет** обрабатывать `twig` на странице. Этот выбор не обрабатывать Twig по умолчанию исключительно из соображений производительности, так как это не очень часто используемая функция. Переменная `process` позволяет вам переопределить это поведение.

Возможно, вы захотите отключить `markdown` на определенной странице, если вы хотите использовать 100% HTML на своей странице и вообще не запускать процесс уценки. Также он позволяет плагину полностью обрабатывать контент. Допустимые значения: `true` или `false`.

Существуют ситуации, когда вы хотите использовать шаблонную функциональность Twig в своем контенте, и это достигается установкой переменной `twig` в true.

### Приоритет обработки Twig

```yaml
twig_first: false
```

Если установлено значение `true`, обработка Twig будет происходить перед любой обработкой Markdown. Это может быть особенно полезно, если ваш Twig генерирует код Markdown, которая должна быть доступна для обработки компилятором Markdown. Стоит отметить, что если кэширование страниц `cache_enable: false` отключено, **то и** `twig_first: true` будет соответственно отключаться.

### Дата публикации

```yaml
publish_date: 01/23/2015 13:00
```

Необязательное поле, но можно указать дату для автоматического запуска публикации. Допустимые значения - это любые строковые значения даты, которые поддерживает [strtotime()](https://php.net/manual/ru/function.strtotime.php).

### Перенаправление

```yaml
redirect: '/some/custom/route'
```

или

```yaml
redirect: 'http://someexternalsite.com'
```

Вы можете перенаправить на другую внутреннюю или внешнюю страницу прямо из заголовка страницы. Конечно, это означает, что эта страница не будет отображаться, но страница все еще может быть в коллекции, меню и т. д., потому что она будет существовать как страница в Grav.

Вы также можете добавить код перенаправления в URL, используя квадратные скобки:

```yaml
redirect: '/some/custom/route[303]'
```

### Routes

```yaml
routes:
  default: '/my/example/page'
  canonical: '/canonical/url/alias'
  aliases:
    - '/some/other/route'
    - '/can-be-any-valid-slug'
```

Теперь вы можете указать **маршрут по умолчанию**, который переопределяет стандартную структуру маршрута, определенную структурой папок.

Вы также можете указать конкретный **канонический маршрут** , который может быть использован в темах для вывода канонической ссылки:

```html
<link rel="canonical" href="https://yoursite/dresses/green-dresses-are-awesome" />
```

Наконец, вы можете указать массив **псевдонимов маршрутов**, которые можно использовать в качестве альтернативных маршрутов для конкретной страницы.

### Маршрутизируемость

```yaml
routable: false
```

По умолчанию все страницы **маршрутизируемые**. Это означает, что они могут быть достигнуты, указав Ваш браузер на URL-адрес страницы. Однако вам может потребоваться создать страницу, созданную для хранения определенного контента, но она предназначена для прямого вызова плагина, другого контента или даже темы напрямую. Хорошим примером этого является страница `Ошибка 404`.

Grav автоматически ищет страницу с маршрутом `/error`, если не удается найти другую страницу. Поскольку это фактическая страница в Grav, вы можете полностью контролировать, как выглядит эта страница. Однако вы, вероятно, не хотите, чтобы люди обращались к этой странице непосредственно в своем браузере, поэтому для этой страницы обычно устанавливается переменная `routable`, равная false. Допустимые значения: `true` или `false`.

### SSL

```yaml
ssl: true
```

Теперь вы можете активировать определенную страницу с помощью SSL **on** или **off**. Это **работает только** с опцией `absolute_urls: true`, установленной в конфигурации `system.yaml`. Это связано с тем, что для переключения между страницами SSL и без SSL необходимо использовать полные URL-адреса с включенным протоколом и хостом.

### Тизер

```yaml
summary:
  enabled: true
  format: short | long
  size: int
```

Параметр ** summary ** определяет, что возвращает метод `page.summary()`. Это чаще всего используется в сценарии типа листинга блога, но может использоваться в любое время, когда вам нужен краткий обзор или краткое содержание содержимого страницы. Сценарии следующие:

[div class="table-keycol"]
| Параметр | Описание |
| -------- | ----------- |
| **enabled:** | Отключить сводку страницы (сводка возвращается так же, как содержимое страницы) |
| **format:** | <ul><li>`long` = Любой итоговый разделитель контента будет игнорироваться<li>`short` = Обнаружение и усечение содержимого до позиции суммарного разделителя</ul> |
[/div]

Атрибут `size` имеет разные значения, когда формат установлен в `short` и `long`:

[div class="table-keycol"]
| Короткий размер | Описание |
| -------- | ----------- |
| **size: 0** | Если разделитель сводки не найден, сводка равна содержанию страницы, в противном случае содержимое будет усечено до позиции разделителя. |
| **size:** `int` | Всегда обрезайте содержимое после **int** символов. Если был найден итоговый разделитель, то обрезать содержимое до позиции итогового разделителя |
[/div]

[div class="table-keycol"]
| Длинный размер | Описание |
| -------- | ----------- |
| **size: 0** | Тизер соответствует всему содержанию страницы |
| **size:** `int` | Содержимое будет усечено после **int** символов, независимо от позиции суммарного разделителя |
[/div]

### Шаблон

```yaml
template: custom
```

Как объяснено в [предыдущей главе](../content-pages), шаблон из темы, который используется для визуализации страницы, основан на имени файла файла `.md`.

Таким образом, файл с именем `default.md` будет использовать шаблон `default` в активной теме. Конечно, вы можете переопределить это поведение, просто установив переменную `template` в заголовке и выбрав другой шаблон.

В приведенном выше примере на странице будет использоваться шаблон `custom` из темы. Эта переменная существует, потому что вам может понадобиться изменить шаблон страницы программно из плагина.

### Формат шаблона

```yaml
template_format: xml
```

Традиционно, если вы хотите, чтобы страница выводила определенный формат (например, xml, json и т. д.), вам необходимо добавить формат в URL. Например, если ввести `http://example.com/sitemap.xml`, браузеру будет отображаться содержимое с использованием шаблона ветки `xml`, заканчивающегося на `.xml.twig`. Это все хорошо, потому что в Grav мы любим делать вещи просто.

Используя заголовок страницы `template_format`, мы можем указать браузеру, как отображать страницу без необходимости расширений в URL. Введя `template_format: xml` на нашей странице` sitemap`, мы можем заставить работать `http://example.com/sitemap` для нас, не добавляя `.xml` в конец.

Мы [использовали этот метод](https://github.com/getgrav/grav-plugin-sitemap/commit/00c23738bdbfe9683627bf0f99bda12eab9505d5#diff-190081f40350c0272970d9171f3437a2) в [плагине Sitemap](https://github.com/getgrav/grav-plugin-sitemap).

### Дата отмены публикации

```yaml
unpublish_date: 05/17/2015 00:32
```

Необязательное поле, но можно указать дату автоматического запуска отмены публикации. Допустимые значения - это любые строковые значения даты, которые поддерживает [strtotime()](https://php.net/manual/ru/function.strtotime.php).

### Видимость

```yaml
visible: false
```

По умолчанию страница **видима** в **навигации**, если окружающая папка имеет числовой префикс, то есть `/01.home` видна, а `/error` **не видна**. Это поведение можно переписать, установив переменную visible в заголовке. Допустимые значения: `true` или `false`.

## Пользовательские заголовки

Конечно, вы можете создавать свои собственные заголовки страниц, используя любой допустимый синтаксис YAML. Они будут привязаны к конкретной странице и будут доступны для любого плагина или темы. Хорошим примером этого может быть установка некоторой переменной, специфичной для плагина карты сайта, такой как:

[prism classes="language-yaml line-numbers"]
sitemap:
    changefreq: monthly
    priority: 1.03
[/prism]

Значимость этих заголовков в том, что Grav не использует их по умолчанию. Они считываются только плагином **sitemap**, чтобы определить, как часто эта конкретная страница изменяется и каков должен быть ее приоритет.

Любой заголовок страницы, такой как этот, должен быть задокументирован, и, как правило, будет определенное значение по умолчанию, которое будет использоваться, если страница не предоставляет его.

Другим примером может быть сохранение специфичных для страницы данных, которые затем могут быть использованы Twig в содержимом страницы.

Например, вы, возможно, захотите связать ссылку на автора для этой страницы. Если вы добавили эти настройки YAML в заголовок страницы:

[prism classes="language-yaml line-numbers"]
author:
    name: Сэнди Джонсон
    twitter: @sandyjohnson
    bio: Сэнди является независимым журналистом и автором нескольких публикаций на платформах CMS с открытым исходным кодом.
[/prism]

Затем вы можете получить к ним доступ из Twig:

[prism classes="language-twig line-numbers"]
<section id="author-details">
    <h2>{{ page.header.author.name }}</h2>
    <p>{{ page.header.author.bio }}</p>
    <span>Contact: <a href="https://twitter.com/{{ page.header.author.twitter }}"><i class="fa fa-twitter"></i></a></span>
</section>
[/prism]

## Мета-заголоки

Мета-заголовки позволяют вам установить [стандартный набор тегов HTML **<meta>**](https://www.w3schools.com/tags/tag_meta.asp) для каждой страницы, а также [OpenGraph](https://ogp.me/), [Facebook](https://developers.facebook.com/docs/sharing/best-practices) и [Twitter](https://dev.twitter.com/cards/overview)

#### Пример стандартных мета-заголовков

[prism classes="language-yaml line-numbers"]
metadata:
    refresh: 30
    generator: 'Grav'
    description: 'Здесь находится описание вашей страницы'
    keywords: 'HTML, CSS, XML, JavaScript'
    author: 'Вася Пупкин'
    robots: 'noindex, nofollow'
    my_key: 'my_value'
[/prism]

Это произведет HTML:

[prism classes="language-twig line-numbers"]
<meta name="generator" content="Grav" />
<meta name="description" content="Здесь находится описание вашей страницы" />
<meta http-equiv="refresh" content="30" />
<meta name="keywords" content="HTML, CSS, XML, JavaScript" />
<meta name="author" content="Вася Пупкин" />
<meta name="robots" content="noindex, nofollow" />
<meta name="my_key" content="my_value" />
[/prism]

Все метатеги HTML5 поддерживаются.

#### Пример мета-тегов OpenGraph

[prism classes="language-yaml line-numbers"]
metadata:
    'og:title': The Rock
    'og:type': video.movie
    'og:url': http://www.imdb.com/title/tt0117500/
    'og:image': http://ia.media-imdb.com/images/rock.jpg
[/prism]

Это произведет HTML:

[prism classes="language-html line-numbers"]
<meta name="og:title" property="og:title" content="The Rock" />
<meta name="og:type" property="og:type" content="video.movie" />
<meta name="og:url" property="og:url" content="http://www.imdb.com/title/tt0117500/" />
<meta name="og:image" property="og:image" content="http://ia.media-imdb.com/images/rock.jpg" />
[/prism]

Для получения полной информации обо всех метатегах OpenGraph, которые можно использовать, обратитесь к [официальной документации](https://ogp.me/).

#### Пример мета-тега Facebook

[prism classes="language-yaml line-numbers"]
metadata:
    'fb:app_id': your_facebook_app_id
[/prism]

Это произведет HTML:

[prism classes="language-html line-numbers"]
<meta name="fb:app_id" property="fb:app_id" content="your_facebook_app_id" />
[/prism]

Facebook в основном использует метатеги OpenGraph, но есть некоторые специфичные для Facebook теги, которые Grav поддерживает автоматически.

#### Пример мета-тегов Twitter

[prism classes="language-yaml line-numbers"]
metadata:
    'twitter:card' : summary
    'twitter:site' : @flickr
    'twitter:title' : Заголовок вашей страницы
    'twitter:description' : Описание вашей страницы может содержать сводную информацию
    'twitter:image' : https://farm6.staticflickr.com/5510/14338202952_93595258ff_z.jpg
[/prism]

Это произведет HTML:

[prism classes="language-twig line-numbers"]
<meta name="twitter:card" property="twitter:card" content="summary" />
<meta name="twitter:site" property="twitter:site" content="@flickr" />
<meta name="twitter:title" property="twitter:title" content="Заголовок вашей страницы" />
<meta name="twitter:description" property="twitter:description" content="Описание вашей страницы может содержать сводную информацию" />
<meta name="twitter:image" property="twitter:image" content="https://farm6.staticflickr.com/5510/14338202952_93595258ff_z.jpg" />
[/prism]

Чтобы получить полную информацию обо всех метатегах Twitter, которые можно использовать, обратитесь к [официальной документации](https://dev.twitter.com/cards/overview).

Это действительно обеспечивает большую гибкость и мощность.

## Заголовки коллекции

Коллекции выросли! Вся [информация о заголовках коллекции](../collections) теперь находится в [отдельной главе](../collections).

## Frontmatter.yaml

Расширенная функция, которая может пригодиться некоторым опытным пользователям, - это возможность использовать общие значения frontmatter через `frontmatter.yaml`, находящийся в папке страницы. Это особенно полезно при работе с многоязычными сайтами, где вы можете поделиться частью frontmatter между всеми языковыми версиями данной страницы.

Чтобы воспользоваться этим, просто создайте файл `frontmatter.yaml` рядом с файлом вашей страницы `.md` и добавьте любые допустимые значения frontmatter. Например:

[prism classes="language-yaml line-numbers"]
metadata:
    generator: 'Super Grav'
    description: Give your page a powerup with Grav!
[/prism]

! Если заголовок определен в обоих frontmatter.yaml и в заголовках frontmatter используются значения страницы frontmatter.yaml переопределяются.

!!!! Использование frontmatter.yaml — это функция на стороне файла,которая **не поддерживается** плагином администратора.