---
title: Хуки событий админки
taxonomy:
    category: docs
process:
    twig: true
---

Плагин админки имеет несколько обработчиков событий, которые можно использовать во время [жизненного цикла Grav](/plugins/grav-lifecycle). См. общую документацию плагина для использования обработчиков событий в главе [Плагины](/plugins).

## Доступные хуки событий админки
* [onAdminTaskExecute](../admin-events#onAdminTaskExecute)
* [onAdminCreatePageFrontmatter](../admin-events#onAdminCreatePageFrontmatter)
* [onAdminSave](../admin-events#onAdminSave)
* [onAdminAfterSave](../admin-events#onAdminAfterSave)
* [onAdminAfterSaveAs](../admin-events#onAdminAfterSaveAs)
* [onAdminAfterDelete](../admin-events#onAdminAfterDelete)
* [onAdminAfterAddMedia](../admin-events#onAdminAfterAddMedia)
* [onAdminAfterDelMedia](../admin-events#onAdminAfterDelMedia)


## Включение хука событий админки
Перехватчики событий админки вызываются [таким же образом](/plugins/plugin-tutorial#step-6-define-if-the-plugin-should-run), что и основные перехватчики событий.


* * *

<a name="onAdminTaskExecute"></a>
### onAdminTaskExecute

Плагин админки запускает различные задачи в зависимости от взаимодействия с пользователем. Задачи могут включать выход из системы, вход в систему, сохранение, 2faverify и т. д. После завершения задачи срабатывает этот обработчик событий.

<a name="onAdminCreatePageFrontmatter"></a>
### onAdminCreatePageFrontmatter

При создании новой страницы это событие запускается после того, как данные заголовка изначально настроены так, чтобы плагины могли программно манипулировать frontmatter.

<a name="onAdminSave"></a>
### onAdminSave

Используйте событие `onAdminSave()` для управления данными объекта страницы `$object` перед их сохранением в файловой системе.

<a name="onAdminAfterSave"></a>
### onAdminAfterSave

После сохранения страницы в панели админки это событие запускается.

<a name="onAdminAfterSaveAs"></a>
### onAdminAfterSaveAs

При создании папки через панель это событие срабатывает сразу после создания новой папки и выполнения стандартной очистки кэша.

<a name="onAdminAfterDelete"></a>
### onAdminAfterDelete

Срабатывает после удаления страницы или папки. Сразу после этого следует стандартная очистка кэша.

<a name="onAdminAfterAddMedia"></a>
### onAdminAfterAddMedia

Срабатывает после завершения задачи добавления мультимедиа, но до отображения подтверждающего сообщения.

<a name="onAdminAfterDelMedia"></a>
### onAdminAfterDelMedia

Срабатывает после завершения задачи удаления мультимедиа, но до отображения подтверждающего сообщения.
