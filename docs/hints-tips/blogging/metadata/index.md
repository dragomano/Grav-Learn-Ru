---
title: "Метаданные записей блога"
description: "Если вы используете Grav как платформу для блогинга, вам нужно добавить метаданные, которые помогают заполнять описания и изображения, когда люди делятся вашей записью в социальных сетях, таких как Facebook, Twitter и т. д."
tags:
  - "Блогинг"
  - "SEO"
---

# Метаданные записей блога

Если вы используете Grav как платформу для блогинга, вам нужно добавить метаданные, которые помогают заполнять описания и изображения, когда люди делятся вашей записью в социальных сетях, таких как Facebook, Twitter и т. д.

Эту информацию нужно добавить в раздел [Заголовки](../../../content/headers/index.md) страницы Grav.

В документации есть ссылка на метаданные, которые нужно добавить в раздел заголовков, — [Метаданные страницы](../../../content/headers/index.md#meta-page-headers). Однако если вы перешли с платформы вроде WordPress, где для этого используется плагин, вы можете недооценивать важность метаданных.

В начале каждой записи блога нужно указать следующее:

```yaml linenums="1"
---
title: Blog Post Title
publish_date: Date the blog post will go live
date: Date the blog post was written
metadata:
    'og:title': Blog Post Title
    'og:type': article
    'og:description': Description of what your blog post is covering.  This will be visible when people share your post on social media.
    'og:url': The URL of the blog post
    'og:site_name': The name of the overall site the blog post belongs to. 
    'og:locale': The language your blog post is written in
    'og:image': The image you reference here will be visible when shared on social media. 
    'twitter:card' : The type of Twitter card that should be used. 
    'twitter:site' : Your Twitter handle
    'twitter:title' : Blog Post Title
    'twitter:description' : Description of what your blog post is covering.  This will be visible when people share your post on social media.
    'twitter:image' : The image you reference here will be visible when shared on social media. 
    'twitter:creator': The twitter handle of the blog post author. 
taxonomy:
    category: [Blog post category]
    tag: [Tag 1, Tag 2, Tag 3, Tag 4]
    author: Author's name
---
```

Подробнее о [Twitter Cards](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/guides/getting-started.html)

Подробнее о [протоколе Open Graph](https://ogp.me/)
