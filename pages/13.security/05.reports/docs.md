---
title: Отчёты
published: true
taxonomy:
    category: docs
content:
    items: '@self.children'
    order:
        by: date
        dir: desc
    limit: 10
    pagination: true
process:
    markdown: true
    twig: true
twig_first: true
---

Это публичный репозиторий отчётов о безопасности Grav. Ниже приведены самые последние отчёты, которые были отправлены и обработаны.

{% for p in page.collection %}
  <a href="{{ p.url }}">
    <h4>{{ p.title }}</h4>
  </a>
{% endfor %}
