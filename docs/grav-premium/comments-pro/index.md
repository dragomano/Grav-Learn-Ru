---
title: "Comments Pro"
description: "Узнайте, как внедрить Comments Pro и управлять им."
tags:
  - "Grav Premium"
  - "Comments Pro"
  - "Grav 1.7"
---

# Comments Pro

> [!IMPORTANT] Важно
> Продукты Premium требуют бесплатного плагина [License Manager](../license-manager/index.md). Установите его и добавьте лицензию продукта перед установкой этого продукта.

## Установка

Сначала убедитесь, что у вас последняя версия **Grav 1.7** (`-f` принудительно обновляет индекс GPM).

```
$ bin/gpm selfupgrade -f
```

У плагина Comments Pro нет зависимостей, поэтому установка сводится к выполнению команды:

```
$ bin/gpm install comments-pro
```

Вы также можете установить плагин через раздел **Плагины** в плагине админки.

---

## Базовая интеграция

### В шаблонах вашей темы

Чтобы добавить комментарии на любую страницу, включите раздел комментариев в свой шаблон:

```twig
{# Способ 1: использовать вспомогательную функцию #}
{{ comments_section() }}

{# Способ 2: подключить шаблон напрямую #}
{% include 'partials/comments-section.html.twig' %}
```

### Включение комментариев на отдельных страницах

Добавьте это в метаданные страницы:

```yaml
---
title: My Blog Post
comments: true
---
```

## Расширенное использование

### Своя форма комментариев

Вы можете настроить форму комментариев, переопределив шаблон:

```twig
{# В вашей теме: templates/partials/comment-form-custom.html.twig #}
{% include 'partials/_comment-form.html.twig' with {
    parent_id: null,
    show_email: true,
    placeholder_text: "Share your thoughts..."
} %}
```

### Получение количества комментариев

Выводите количество комментариев в любом месте:

```twig
This post has {{ comment_count() }} comments.
```

### Функции админки

#### Модерация комментариев

- Откройте `/admin` и перейдите в Comments Pro
- Одобряйте, отклоняйте комментарии или помечайте их как спам
- Доступны массовые действия

#### Эндпоинты API

Плагин предоставляет эндпоинты REST API:

- `POST /_comments/` — добавить новый комментарий
- `GET /_comments/` — получить все комментарии
- `PATCH /_comments/{id}/status` — обновить статус комментария
- `DELETE /_comments/{id}` — удалить комментарий
- `GET /_comments/{id}/replies` — получить ответы на комментарий

## Параметры конфигурации

### Базовые настройки

```yaml
enabled: true
built_in_css: true
built_in_js: true
api_route: '/_comments'
```

### Модерация

```yaml
moderation:
  enabled: true
  default_status: 'pending'  # или 'published'
  auto_approve_registered: false
  auto_approve_previous: false
```

### Защита от спама

```yaml
spam:
  enabled: true
  honeypot_field: 'website_url'
  min_time_seconds: 3
  link_limit: 2
  spam_threshold: 50
  scores:
    honeypot_filled: 100
    time_too_fast: 20
    per_extra_link: 10
    keyword_match: 15
    suspicious_patterns: 10
    email_in_text: 20
  blacklist:
    - 'spam keyword'
    - 'another bad word'
```

### Своё оформление

Отключите встроенный CSS и добавьте свой:

```yaml
built_in_css: false
```

Затем добавьте свои стили в CSS темы. Подробнее см. в [руководстве по темизации](#table-of-contents-1).

## Хранение данных

Комментарии хранятся в виде YAML-файлов в папке каждой страницы:
```
/user/pages/01.blog/my-post/comments.yaml
```

## Фичи безопасности

- Защита от спама с помощью ловушки (honeypot)
- Ограничение частоты запросов
- Фильтрация контента
- Защита от XSS
- Для модерации требуется аутентификация администратора

## Решение проблем

### Комментарии не отображаются

1. Проверьте, что плагин включён
2. Убедитесь, что HTMX загружается
3. Проверьте консоль браузера на ошибки JavaScript
4. Убедитесь, что у папок страниц правильные права доступа

### Ошибки API

1. Проверьте журналы ошибок сервера
2. Убедитесь, что установлен Bramus Router
3. Убедитесь, что конфигурация `.htaccess` верна

### Проблемы со спамом

1. Скорректируйте порог спама
2. Добавьте больше ключевых слов в чёрный список
3. Включите более строгую модерацию
4. Проверьте реализацию ловушки

## Производительность

Плагин оптимизирован для производительности:

- Атомарная запись файлов
- Минимум запросов к базе данных
- Эффективное построение веток комментариев
- Опциональная поддержка кэширования

Для сайтов с высокой посещаемостью рассмотрите:
- Включение кэширования страниц
- Использование внешнего хранилища комментариев
- Внедрение ограничения частоты запросов

---

## Руководство по конфигурации

В этом руководстве подробно описаны все параметры конфигурации, доступные в плагине Comments Pro.

## Оглавление

1. [Расположение файла конфигурации](#configuration-file-location)
2. [Общие настройки](#general-settings)
3. [Настройки модерации](#moderation-settings)
4. [Настройки спам-фильтра](#spam-filter-settings)
5. [Настройки аутентификации](#authentication-settings)
6. [Настройки безопасности](#security-settings)
7. [Настройки отображения](#display-settings)
8. [Настройки редактора Markdown](#markdown-editor-settings)
9. [Настройки обновлений в реальном времени](#real-time-updates-settings)
10. [Типичные сценарии конфигурации](#common-configuration-scenarios)
11. [Тонкая настройка производительности](#performance-tuning)

## Расположение файла конфигурации {: #configuration-file-location }

Конфигурацию плагина следует хранить в:

```
user/config/plugins/comments-pro.yaml
```

Никогда не редактируйте файл конфигурации по умолчанию:
```
user/plugins/comments-pro/comments-pro.yaml
```

## Общие настройки {: #general-settings }

### Базовые настройки плагина

```yaml
# Включить/отключить плагин целиком
enabled: true

# Подключать встроенные стили CSS
built_in_css: true

# Подключать встроенный JavaScript (HTMX и скрипты плагина)
built_in_js: true

# Префикс маршрута эндпоинта API
api_route: '/_comments'
```

**Пояснения к параметрам:**

- `enabled`: главный переключатель плагина
- `built_in_css`: установите `false`, если хотите использовать свои стили
- `built_in_js`: установите `false`, если HTMX уже загружается у вас в другом месте
- `api_route`: префикс URL для эндпоинтов API (измените при конфликтах)

## Настройки модерации {: #moderation-settings }

### Конфигурация модерации комментариев

```yaml
moderation:
  # Включить модерацию комментариев
  enabled: true
  
  # Статус по умолчанию для новых комментариев
  # Варианты: 'published', 'pending', 'spam'
  default_status: pending
  
  # Автоматически одобрять комментарии зарегистрированных пользователей
  auto_approve_registered: false
  
  # Автоматически одобрять, если предыдущий комментарий пользователя был одобрен
  auto_approve_previous: false
```

**Стратегии модерации:**

1. **Открытые комментарии** (без модерации):
   ```yaml
   moderation:
     enabled: false
   ```

2. **Полная модерация** (все комментарии проверяются):
   ```yaml
   moderation:
     enabled: true
     default_status: pending
   ```

3. **Доверять зарегистрированным пользователям**:
   ```yaml
   moderation:
     enabled: true
     default_status: pending
     auto_approve_registered: true
   ```

## Настройки спам-фильтра {: #spam-filter-settings }

### Конфигурация обнаружения спама

```yaml
spam:
  # Включить фильтрацию спама
  enabled: true
  
  # Имя поля-ловушки (должно быть скрыто через CSS)
  honeypot_field: 'website_url'
  
  # Минимальное число секунд до отправки формы
  min_time_seconds: 3
  
  # Порог баллов для пометки как спама
  spam_threshold: 50
  
  # Порог баллов для немедленного удаления
  delete_threshold: 75
  
  # Максимум ссылок до начисления спам-баллов
  link_limit: 2
  
  # Веса баллов для различных признаков спама
  scores:
    honeypot_filled: 100      # Мгновенный спам, если ловушка заполнена
    time_too_fast: 20         # Слишком быстрая отправка
    per_extra_link: 10        # Каждая ссылка сверх лимита
    keyword_match: 15         # Каждое ключевое слово из чёрного списка
    suspicious_patterns: 10   # Подозрительные текстовые шаблоны
    email_in_text: 20        # Адреса электронной почты в тексте комментария
  
  # Ключевые слова из чёрного списка (без учёта регистра)
  blacklist:
    - 'viagra'
    - 'casino'
    - 'cheap meds'
    - 'click here'
    - 'limited offer'
```

**Примеры начисления спам-баллов:**

- Заполнена ловушка: 100 баллов (мгновенный спам)
- Форма отправлена за 2 секунды + 3 лишние ссылки: 20 + 30 = 50 баллов (помечено как спам)
- Обычный комментарий с 1 ссылкой: 0 баллов (чисто)

## Настройки аутентификации {: #authentication-settings }

### Конфигурация аутентификации пользователей

```yaml
authentication:
  # Разрешить комментировать пользователям без входа
  allow_guests: true
  
  # Требовать вход для комментирования
  require_login: false
  
  # Показывать встроенную форму входа
  grav_login_integration: false
```

**Режимы аутентификации:**

1. **Открытое комментирование** (по умолчанию):
   ```yaml
   authentication:
     allow_guests: true
     require_login: false
   ```

2. **Только для участников**:
   ```yaml
   authentication:
     allow_guests: false
     require_login: true
     grav_login_integration: true
   ```

3. **Необязательный вход**:
   ```yaml
   authentication:
     allow_guests: true
     require_login: false
     grav_login_integration: true
   ```

## Настройки безопасности {: #security-settings }

### Конфигурация безопасности

```yaml
security:
  # Включить защиту от CSRF
  csrf_protection: true
  
  # Отслеживать IP-адреса (отключите для GDPR)
  track_ip_addresses: true
  
  # Конфигурация ограничения частоты запросов
  rate_limit:
    enabled: true
    max_requests: 10          # Максимум комментариев с одного IP
    time_window: 300          # В пределах 5 минут
    timeout_duration: 600     # Штраф на 10 минут
  
  # Валидация времени заполнения формы
  form_timing:
    enabled: true
    min_time: 3              # Минимум секунд
    max_time: 3600           # Максимум секунд (1 час)
  
  # Список заблокированных IP (поддерживает маски и CIDR)
  ip_blocklist:
    - '192.168.1.100'        # Конкретный IP
    - '10.0.0.*'             # Диапазон по маске
    - '172.16.0.0/12'        # Нотация CIDR
```

**Профили безопасности:**

1. **Максимальная безопасность**:
   ```yaml
   security:
     csrf_protection: true
     track_ip_addresses: true
     rate_limit:
       enabled: true
       max_requests: 5
       time_window: 300
       timeout_duration: 1800
   ```

2. **Соответствие GDPR**:
   ```yaml
   security:
     csrf_protection: true
     track_ip_addresses: false  # Без хранения IP
     rate_limit:
       enabled: true  # Всё ещё работает с хешированием IP
   ```

## Настройки отображения {: #display-settings }

### Конфигурация отображения комментариев

```yaml
display:
  # Режим просмотра: 'nested', 'tree' или 'flat'
  view_mode: nested
  
  # Максимальная визуальная глубина вложенности
  nested_max_depth: 5
  
  # Позиция формы: 'top' или 'bottom'
  comment_form_position: bottom
  
  # Разрешить ссылки Markdown в комментариях
  allow_links: true
  
  # Пагинация: 'none', 'manual' или 'infinite'
  pagination_type: manual
  
  # Комментариев на страницу
  pagination_items: 10
  
  # Максимальная длина комментария
  max_characters: 5000
  
  # Показывать аватары пользователей
  show_avatars: true
  
  # Размер аватара в пикселях
  avatar_size: 40
```

**Пояснения к режимам отображения:**

1. **Режим Nested**: простые ответы с отступами
   - Лучше всего для: простых обсуждений
   - Производительность: хорошая

2. **Режим Tree**: сворачиваемая вложенная структура
   - Лучше всего для: глубоких обсуждений
   - Производительность: средняя

3. **Режим Flat**: хронологический с индикаторами ответов
   - Лучше всего для: большого количества комментариев
   - Производительность: лучшая

## Настройки редактора Markdown {: #markdown-editor-settings }

### Конфигурация редактора

```yaml
markdown_editor:
  # Включить редактор Markdown с панелью инструментов
  enabled: true
  
  # Автоматически менять размер текстового поля при вводе
  auto_resize: true
  
  # Размеры текстового поля
  min_height: 100
  max_height: 400
```

**Настройка редактора:**

Для своих кнопок панели инструментов измените шаблон:
```twig
data-markdown-toolbar="bold,italic,link,heading,quote,code,unordered_list,ordered_list"
```

## Настройки обновлений в реальном времени {: #real-time-updates-settings }

### Конфигурация фич реального времени

```yaml
realtime:
  # Включить автоматический опрос обновлений
  enabled: true
  
  # Базовый интервал проверки (секунды)
  # Адаптивно: уменьшается вдвое при высокой активности, удваивается при низкой
  check_interval: 10
  
  # Показывать уведомления о новых комментариях
  notifications: true
  
  # Позиция уведомлений
  notification_position: top-right
  
  # Автоматически обновлять список комментариев
  auto_update: true
  
  # Показывать кнопку ручного обновления
  show_update_button: true
```

**Режимы реального времени:**

1. **Полное реальное время**:
   ```yaml
   realtime:
     enabled: true
     check_interval: 5
     auto_update: true
   ```

2. **Только уведомления**:
   ```yaml
   realtime:
     enabled: true
     notifications: true
     auto_update: false
     show_update_button: true
   ```

## Типичные сценарии конфигурации {: #common-configuration-scenarios }

### Комментарии в блоге

```yaml
# Идеально для записей блога со средней посещаемостью
enabled: true
moderation:
  enabled: true
  default_status: pending
  auto_approve_previous: true
spam:
  enabled: true
  spam_threshold: 40
display:
  view_mode: nested
  pagination_type: manual
  pagination_items: 20
realtime:
  enabled: false  # Для блогов не нужно
```

### Форумы сообщества

```yaml
# Обсуждения в сообществе с высокой вовлечённостью
enabled: true
moderation:
  enabled: true
  auto_approve_registered: true
authentication:
  require_login: true
display:
  view_mode: tree
  nested_max_depth: 10
  pagination_type: infinite
realtime:
  enabled: true
  check_interval: 5
```

### Корпоративный сайт

```yaml
# Строгая модерация, без спама
enabled: true
moderation:
  enabled: true
  default_status: pending
spam:
  enabled: true
  spam_threshold: 30
  delete_threshold: 50
security:
  rate_limit:
    max_requests: 3
    time_window: 600
authentication:
  allow_guests: false
  require_login: true
```

### Новостной сайт

```yaml
# Большой объём, обсуждения в реальном времени
enabled: true
moderation:
  enabled: true
  auto_approve_registered: true
display:
  view_mode: flat
  pagination_type: infinite
  pagination_items: 50
realtime:
  enabled: true
  check_interval: 10
  auto_update: true
```

## Тонкая настройка производительности {: #performance-tuning }

### Оптимизация при высокой посещаемости

```yaml
# Оптимизация под производительность
display:
  view_mode: flat           # Самое быстрое отображение
  pagination_type: manual   # Загрузка по требованию
  pagination_items: 25      # Балансировка нагрузки
  show_avatars: false       # Меньше запросов

realtime:
  enabled: false            # Снижение нагрузки на сервер
  # Или используйте более длинные интервалы
  check_interval: 30

# Агрессивное кэширование (если используется кэш Grav)
cache_lifetime: 3600        # 1 час
```

### Оптимизация при низкой посещаемости

```yaml
# Богатые фичи для вовлечённых пользователей
display:
  view_mode: tree
  pagination_type: none     # Показывать все
  show_avatars: true

markdown_editor:
  enabled: true
  auto_resize: true

realtime:
  enabled: true
  check_interval: 5         # Быстрые обновления
```

## Рекомендации по конфигурации

1. **Начните осторожно**: включите модерацию и корректируйте её в зависимости от уровня спама
2. **Следите за спам-баллами**: проверяйте админку, чтобы точно настроить пороги спама
3. **Проверьте ограничения частоты запросов**: убедитесь, что добросовестные пользователи не блокируются
4. **Учитывайте кэш**: очищайте кэш после изменения конфигурации
5. **Делайте резервные копии конфигурации**: храните версионные копии своей конфигурации

## Решение проблем с конфигурацией

### Комментарии не сохраняются

Проверьте:
- Права доступа к `user/data/`
- Конфликты `csrf_protection`
- Слишком строгое ограничение частоты запросов

### Спам проходит

Скорректируйте:
- Уменьшите `spam_threshold`
- Добавьте ключевые слова в `blacklist`
- Включите `form_timing`

### Проблемы с производительностью

Попробуйте:
- Переключитесь на режим просмотра `flat`
- Включите пагинацию
- Отключите обновления в реальном времени
- Уменьшите размер аватара или отключите его

### Проблемы с аутентификацией

Убедитесь:
- Плагин Grav Login установлен
- `require_login` соответствует вашим потребностям
- Конфигурация сессии верна

---

## Руководство по темизации и настройке

В этом руководстве рассказано, как настроить внешний вид и поведение плагина Comments Pro под вашу тему и требования.

## Оглавление {: #table-of-contents-1 }

1. [Система переопределения шаблонов](#template-override-system)
2. [Настройка CSS](#css-customization)
3. [Хуки JavaScript](#javascript-hooks)
4. [Переменные шаблона](#template-variables)
5. [Свои режимы просмотра](#custom-view-modes)
6. [Примеры интеграции с темами](#theme-integration-examples)
7. [Расширенная настройка](#advanced-customization)

## Система переопределения шаблонов {: #template-override-system }

### Иерархия переопределений

Плагин ищет переопределения шаблонов в таком порядке:

1. `user/themes/your-theme/templates/plugins/comments-pro/`
2. `user/plugins/comments-pro/templates/`

### Доступные шаблоны

```text
templates/
├── partials/
│   ├── comments-section.html.twig          # Основная обёртка
│   ├── _comments-base.html.twig             # Базовый шаблон с переменными
│   ├── _comments-header.html.twig           # Количество комментариев и элементы сортировки
│   ├── _comment-form.html.twig              # Форма отправки комментария
│   ├── _comment.html.twig                   # Отображение отдельного комментария
│   ├── _comments-list.html.twig             # Вложенное представление
│   ├── _comments-tree.html.twig             # Древовидное представление (сворачиваемое)
│   ├── _comments-flat.html.twig             # Плоское/хронологическое представление
│   ├── _login-form.html.twig                # Встроенная форма входа
│   └── _vote-count.html.twig                # Кнопки голосования и счётчик
└── admin/
    └── templates/                           # Шаблоны интерфейса админки
```

### Базовое переопределение шаблона

Создайте папку в вашей теме:

```bash
mkdir -p user/themes/your-theme/templates/plugins/comments-pro/partials/
```

Скопируйте и измените любой шаблон:

```bash
cp user/plugins/comments-pro/templates/partials/_comment.html.twig \
   user/themes/your-theme/templates/plugins/comments-pro/partials/
```

### Пример: свой шаблон комментария

```twig
{# user/themes/your-theme/templates/plugins/comments-pro/partials/_comment.html.twig #}
<article class="comment {{ comment.status }}" id="comment-{{ comment.id }}">
    <div class="comment-avatar">
        {% if config.plugins['comments-pro'].display.show_avatars %}
            <img src="{{ comment.email|avatar_url }}" 
                 alt="{{ comment.author }}" 
                 class="avatar">
        {% endif %}
    </div>
    
    <div class="comment-content">
        <header class="comment-meta">
            <h4 class="comment-author">{{ comment.author }}</h4>
            <time class="comment-date" 
                  datetime="{{ comment.date|date('c') }}">
                {{ comment.date|date('M j, Y \\a\\t g:i A') }}
            </time>
            
            {# Своё: добавить значок пользователя для зарегистрированных пользователей #}
            {% if comment.logged_in %}
                <span class="user-badge">Verified Member</span>
            {% endif %}
        </header>
        
        <div class="comment-text">
            {{ comment.text|comment_text_render }}
        </div>
        
        {# Своё: социальные кнопки #}
        <div class="comment-actions">
            {% include 'partials/_vote-count.html.twig' %}
            
            <button class="reply-button" 
                    onclick="toggleReplyForm('{{ comment.id }}')">
                Reply
            </button>
            
            <button class="share-comment" 
                    data-comment-id="{{ comment.id }}">
                Share
            </button>
        </div>
    </div>
    
    {# Вложенные ответы #}
    {% if comment.replies is not empty %}
        <div class="comment-replies">
            {% for reply in comment.replies %}
                {% include 'partials/_comment.html.twig' with {'comment': reply} %}
            {% endfor %}
        </div>
    {% endif %}
</article>
```

## Настройка CSS {: #css-customization }

### Пользовательские свойства CSS

Плагин использует пользовательские свойства CSS для удобной темизации:

```css
/* Переопределение в CSS вашей темы */
:root {
    /* Основные цвета */
    --comments-color-primary: #007bff;
    --comments-color-primary-hover: #0056b3;
    
    /* Цвета текста */
    --comments-color-text: #333;
    --comments-color-text-light: #666;
    --comments-color-text-muted: #999;
    
    /* Цвета фона */
    --comments-color-bg: #ffffff;
    --comments-color-bg-light: #f8f9fa;
    --comments-color-bg-alt: #e9ecef;
    
    /* Цвета рамок */
    --comments-color-border: #dee2e6;
    --comments-color-border-light: #e9ecef;
    
    /* Цвета статусов */
    --comments-color-success: #28a745;
    --comments-color-warning: #ffc107;
    --comments-color-error: #dc3545;
    --comments-color-info: #17a2b8;
    
    /* Отступы */
    --comments-spacing-xs: 0.25rem;
    --comments-spacing-sm: 0.5rem;
    --comments-spacing-md: 1rem;
    --comments-spacing-lg: 1.5rem;
    --comments-spacing-xl: 3rem;
    
    /* Типографика */
    --comments-font-family: inherit;
    --comments-font-size: 1rem;
    --comments-font-size-sm: 0.875rem;
    --comments-line-height: 1.5;
    
    /* Рамки */
    --comments-border-radius: 0.375rem;
    --comments-border-width: 1px;
    
    /* Тени */
    --comments-shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
    --comments-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
    --comments-shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}
```

### Отключение встроенного CSS

Чтобы использовать полностью свои стили:

```yaml
# В comments-pro.yaml
built_in_css: false
```

Затем создайте свой CSS:

```css
/* CSS-файл вашей темы */
.comments-section {
    /* Ваши стили */
}

.comment-form {
    /* Оформление формы */
}

.comment {
    /* Оформление отдельного комментария */
}
```

### Поддержка тёмного режима

```css
/* Автоматический тёмный режим */
@media (prefers-color-scheme: dark) {
    :root {
        --comments-color-text: #e9ecef;
        --comments-color-text-light: #adb5bd;
        --comments-color-bg: #212529;
        --comments-color-bg-light: #343a40;
        --comments-color-border: #495057;
    }
}

/* Ручное переключение тёмного режима */
[data-theme="dark"] {
    --comments-color-text: #e9ecef;
    --comments-color-text-light: #adb5bd;
    --comments-color-bg: #212529;
    --comments-color-bg-light: #343a40;
    --comments-color-border: #495057;
}
```

### Адаптивный дизайн

```css
/* Адаптивный дизайн mobile-first */
.comments-section {
    padding: var(--comments-spacing-md);
}

.comment {
    margin-bottom: var(--comments-spacing-lg);
}

.comment-form {
    display: grid;
    gap: var(--comments-spacing-md);
}

@media (min-width: 768px) {
    .comment-form {
        grid-template-columns: 1fr 1fr;
    }
    
    .comment-form .form-field-text {
        grid-column: 1 / -1;
    }
}

/* Отступы для вложенных комментариев */
.comment-replies {
    margin-left: var(--comments-spacing-lg);
    border-left: 2px solid var(--comments-color-border-light);
    padding-left: var(--comments-spacing-md);
}

@media (max-width: 767px) {
    .comment-replies {
        margin-left: var(--comments-spacing-md);
    }
}
```

## Хуки JavaScript {: #javascript-hooks }

### Доступные события

Плагин генерирует пользовательские события, которые вы можете слушать:

```javascript
// События комментариев
document.addEventListener('comments:loaded', function(event) {
    console.log('Comments loaded:', event.detail);
});

document.addEventListener('comments:added', function(event) {
    console.log('New comment added:', event.detail);
});

document.addEventListener('comments:updated', function(event) {
    console.log('Comments updated:', event.detail);
});

document.addEventListener('comments:error', function(event) {
    console.log('Comment error:', event.detail);
});

// События формы
document.addEventListener('commentForm:submit', function(event) {
    console.log('Form submitted:', event.detail);
});

document.addEventListener('commentForm:reset', function(event) {
    console.log('Form reset:', event.detail);
});

// События реального времени
document.addEventListener('comments:realtime:update', function(event) {
    console.log('Real-time update:', event.detail);
});
```

### Своя интеграция JavaScript

```javascript
// Инициализация своих фич комментариев
document.addEventListener('DOMContentLoaded', function() {
    initCommentFeatures();
});

function initCommentFeatures() {
    // Добавить время чтения к комментариям
    addReadingTime();
    
    // Настроить подсветку комментариев
    setupCommentHighlighting();
    
    // Инициализировать социальные кнопки
    initSocialSharing();
}

function addReadingTime() {
    document.querySelectorAll('.comment-text').forEach(function(element) {
        const text = element.textContent;
        const words = text.split(/\s+/).length;
        const readingTime = Math.ceil(words / 200); // 200 слов в минуту
        
        if (readingTime > 1) {
            const badge = document.createElement('span');
            badge.className = 'reading-time';
            badge.textContent = `${readingTime} min read`;
            
            const meta = element.parentNode.querySelector('.comment-meta');
            meta.appendChild(badge);
        }
    });
}

function setupCommentHighlighting() {
    // Подсветить комментарии, на которые ведёт ссылка
    const hash = window.location.hash;
    if (hash.startsWith('#comment-')) {
        const commentElement = document.querySelector(hash);
        if (commentElement) {
            commentElement.classList.add('highlighted');
            commentElement.scrollIntoView({ behavior: 'smooth' });
        }
    }
}

function initSocialSharing() {
    document.querySelectorAll('.share-comment').forEach(function(button) {
        button.addEventListener('click', function() {
            const commentId = this.dataset.commentId;
            const url = `${window.location.origin}${window.location.pathname}#comment-${commentId}`;
            
            if (navigator.share) {
                navigator.share({
                    title: 'Check out this comment',
                    url: url
                });
            } else {
                // Резервный вариант: скопировать в буфер обмена
                navigator.clipboard.writeText(url);
                showToast('Link copied to clipboard!');
            }
        });
    });
}
```

### Расширение возможностей плагина

```javascript
// Расширить ядро Comments
if (window.CommentsCore) {
    // Добавить свою валидацию
    CommentsCore.addValidator('profanity', function(text) {
        const profanityWords = ['badword1', 'badword2'];
        const hasProfanity = profanityWords.some(word => 
            text.toLowerCase().includes(word)
        );
        
        if (hasProfanity) {
            return 'Please keep comments family-friendly';
        }
        return null; // Корректно
    });
    
    // Добавить свой форматтер
    CommentsCore.addFormatter('mentions', function(text) {
        return text.replace(/@(\w+)/g, '<span class="mention">@$1</span>');
    });
}
```

## Переменные шаблона {: #template-variables }

### Доступные переменные в шаблонах

```twig
{# Глобальная конфигурация плагина #}
{{ config.plugins['comments-pro'] }}

{# Текущий объект страницы #}
{{ page }}

{# Данные комментария #}
{{ comment.id }}
{{ comment.author }}
{{ comment.email }}
{{ comment.text }}
{{ comment.html }}           {# Отрендеренный HTML #}
{{ comment.date }}
{{ comment.status }}
{{ comment.parent_id }}
{{ comment.score }}
{{ comment.votes.up }}
{{ comment.votes.down }}
{{ comment.replies }}        {# Массив ответов на комментарий #}
{{ comment.ip_address }}     {# Если отслеживание включено #}
{{ comment.logged_in }}      {# Логическое значение #}

{# Конфигурация представления #}
{{ view_mode }}              {# 'nested', 'tree', 'flat' #}
{{ form_position }}          {# 'top', 'bottom' #}
{{ api_base }}               {# URL эндпоинта API #}
{{ pagination_type }}        {# 'none', 'manual', 'infinite' #}
{{ pagination_items }}       {# Количество на страницу #}
{{ realtime_enabled }}       {# Логическое значение #}

{# Информация о пользователе #}
{{ grav.user }}              {# Текущий объект пользователя #}
{{ is_logged_in }}           {# Логическое значение #}
```

### Пользовательские переменные шаблона

Добавьте свои переменные в переопределённом шаблоне:

```twig
{# _comments-base.html.twig #}
{% set custom_config = {
    'show_user_badges': true,
    'enable_threading': comment_depth < 3,
    'social_sharing': true
} %}

{# Передать дочерним шаблонам #}
{% block content %}
    {# Содержимое шаблона с доступным custom_config #}
{% endblock %}
```

## Свои режимы просмотра {: #custom-view-modes }

### Создание своего режима просмотра

1. **Создайте шаблон:**

```twig
{# user/themes/your-theme/templates/plugins/comments-pro/partials/_comments-cards.html.twig #}
<div class="comments-cards-view">
    {% for comment in comments %}
        <div class="comment-card">
            <div class="card-header">
                <img src="{{ comment.email|avatar_url }}" alt="{{ comment.author }}">
                <div class="author-info">
                    <h4>{{ comment.author }}</h4>
                    <time>{{ comment.date|date('M j') }}</time>
                </div>
            </div>
            <div class="card-body">
                {{ comment.text|comment_text_render }}
            </div>
            {% if comment.replies %}
                <div class="card-replies">
                    {{ comment.replies|length }} replies
                    <button onclick="toggleReplies('{{ comment.id }}')">
                        Show/Hide
                    </button>
                </div>
            {% endif %}
        </div>
    {% endfor %}
</div>
```

2. **Измените контроллер, чтобы он использовал ваш шаблон:**

```php
// В своём плагине или функции темы
public function selectCommentsTemplate($viewMode) {
    switch ($viewMode) {
        case 'cards':
            return 'partials/_comments-cards.html.twig';
        case 'timeline':
            return 'partials/_comments-timeline.html.twig';
        default:
            return 'partials/_comments-list.html.twig';
    }
}
```

3. **Добавьте CSS для нового представления:**

```css
.comments-cards-view {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: var(--comments-spacing-lg);
}

.comment-card {
    background: var(--comments-color-bg);
    border: var(--comments-border-width) solid var(--comments-color-border);
    border-radius: var(--comments-border-radius);
    padding: var(--comments-spacing-md);
    box-shadow: var(--comments-shadow);
}

.card-header {
    display: flex;
    align-items: center;
    gap: var(--comments-spacing-sm);
    margin-bottom: var(--comments-spacing-md);
}
```

## Примеры интеграции с темами {: #theme-integration-examples }

### Интеграция с Bootstrap

```twig
{# Форма комментариев в стиле Bootstrap #}
<form class="comment-form" 
      hx-post="{{ api_base }}" 
      hx-target="#comments-list">
    
    <div class="row">
        <div class="col-md-6">
            <div class="mb-3">
                <label for="author" class="form-label">Name *</label>
                <input type="text" 
                       class="form-control" 
                       name="author" 
                       id="author" 
                       required>
            </div>
        </div>
        <div class="col-md-6">
            <div class="mb-3">
                <label for="email" class="form-label">Email</label>
                <input type="email" 
                       class="form-control" 
                       name="email" 
                       id="email">
            </div>
        </div>
    </div>
    
    <div class="mb-3">
        <label for="text" class="form-label">Comment *</label>
        <textarea class="form-control" 
                  name="text" 
                  id="text" 
                  rows="4" 
                  required></textarea>
    </div>
    
    {{ nonce_field('comments-pro')|raw }}
    
    <button type="submit" class="btn btn-primary">
        <i class="fas fa-comment"></i> Post Comment
    </button>
</form>
```

### Интеграция с Tailwind CSS

```twig
{# Комментарий в стиле Tailwind #}
<article class="bg-white rounded-lg shadow-md p-6 mb-6">
    <div class="flex items-start space-x-4">
        <img class="w-10 h-10 rounded-full" 
             src="{{ comment.email|avatar_url }}" 
             alt="{{ comment.author }}">
        
        <div class="flex-1 min-w-0">
            <div class="flex items-center space-x-2">
                <h3 class="text-sm font-semibold text-gray-900">
                    {{ comment.author }}
                </h3>
                <time class="text-sm text-gray-500">
                    {{ comment.date|date('M j, Y') }}
                </time>
            </div>
            
            <div class="mt-2 text-sm text-gray-700">
                {{ comment.text|comment_text_render }}
            </div>
            
            <div class="mt-4 flex items-center space-x-4">
                <button class="text-sm text-blue-600 hover:text-blue-800">
                    Reply
                </button>
                <div class="flex items-center space-x-1">
                    <button class="text-gray-400 hover:text-blue-600">
                        👍 {{ comment.votes.up }}
                    </button>
                </div>
            </div>
        </div>
    </div>
</article>
```

### Интеграция с Foundation

```css
/* Стили, специфичные для Foundation */
.comments-section {
    @include xy-grid-container;
}

.comment-form {
    @include xy-grid;
    
    .form-field-author,
    .form-field-email {
        @include xy-cell(6);
        
        @include breakpoint(small only) {
            @include xy-cell(12);
        }
    }
    
    .form-field-text {
        @include xy-cell(12);
    }
}

.comment {
    @include callout;
    margin-bottom: 1rem;
    
    &.highlighted {
        @include callout($background: $warning-color);
    }
}
```

## Расширенная настройка {: #advanced-customization }

### Своя фильтрация комментариев

```javascript
// Добавить фильтрацию комментариев на стороне клиента
class CommentFilter {
    constructor() {
        this.filters = new Set();
        this.initializeControls();
    }
    
    initializeControls() {
        const filterContainer = document.createElement('div');
        filterContainer.className = 'comment-filters';
        filterContainer.innerHTML = `
            <label>
                <input type="checkbox" value="show-votes"> 
                Show highly voted only
            </label>
            <label>
                <input type="checkbox" value="recent"> 
                Recent comments only
            </label>
        `;
        
        filterContainer.addEventListener('change', (e) => {
            if (e.target.checked) {
                this.filters.add(e.target.value);
            } else {
                this.filters.delete(e.target.value);
            }
            this.applyFilters();
        });
        
        document.querySelector('.comments-header').appendChild(filterContainer);
    }
    
    applyFilters() {
        const comments = document.querySelectorAll('.comment');
        
        comments.forEach(comment => {
            let show = true;
            
            if (this.filters.has('show-votes')) {
                const votes = parseInt(comment.querySelector('.vote-count')?.textContent || '0');
                show = show && votes >= 5;
            }
            
            if (this.filters.has('recent')) {
                const date = new Date(comment.querySelector('time').dateTime);
                const dayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);
                show = show && date > dayAgo;
            }
            
            comment.style.display = show ? 'block' : 'none';
        });
    }
}

// Инициализировать фильтрацию
new CommentFilter();
```

### Реакции на комментарии

```twig
{# Добавить кнопки реакций в шаблон комментария #}
<div class="comment-reactions">
    {% set reactions = ['👍', '❤️', '😂', '😮', '😢', '😠'] %}
    {% for reaction in reactions %}
        <button class="reaction-button" 
                data-reaction="{{ reaction }}"
                data-comment-id="{{ comment.id }}">
            {{ reaction }} 
            <span class="reaction-count">{{ comment.reactions[reaction]|default(0) }}</span>
        </button>
    {% endfor %}
</div>

<script>
document.addEventListener('click', function(e) {
    if (e.target.matches('.reaction-button')) {
        const reaction = e.target.dataset.reaction;
        const commentId = e.target.dataset.commentId;
        
        fetch(`/_comments${window.location.pathname}?action=react`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                comment_id: commentId,
                reaction: reaction,
                nonce: document.querySelector('[name="nonce"]').value
            })
        })
        .then(response => response.json())
        .then(data => {
            if (data.success) {
                e.target.querySelector('.reaction-count').textContent = data.count;
            }
        });
    }
});
</script>
```

### Интеграция с конструктором страниц

```twig
{# Для конструкторов страниц вроде Gantry #}
{% if config.plugins['comments-pro'].enabled and page.header.comments %}
    <div class="g-content">
        <div class="g-container">
            {{ comments_section() }}
        </div>
    </div>
{% endif %}
```

Это руководство по темизации подробно охватывает все возможности настройки, доступные в плагине Comments Pro, — от простых правок CSS до продвинутой интеграции JavaScript и собственных режимов просмотра.
