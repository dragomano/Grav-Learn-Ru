---
description: Список доступных хуков админки Grav CMS.
---

# Хуки событий админки

Плагин админки имеет несколько обработчиков событий, которые можно использовать во время [жизненного цикла Grav](/plugins/grav-lifecycle). См. общую документацию плагина для использования обработчиков событий в главе [Плагины](/plugins).

## Включение хука событий админки
Перехватчики событий админки вызываются [таким же образом](/plugins/plugin-tutorial/#shag-7-opredelenie-neobkhodimosti-zapuska-plagina), как и основные перехватчики событий.

## Доступные хуки событий админки

* * *

### onAdminTaskExecute

Плагин админки запускает различные задачи в зависимости от взаимодействия с пользователем. Задачи могут включать выход из системы, вход в систему, сохранение, 2faverify и т. д. После завершения задачи срабатывает этот обработчик событий.

### onAdminCreatePageFrontmatter

При создании новой страницы это событие запускается после того, как данные заголовка изначально настроены так, чтобы плагины могли программно манипулировать frontmatter.

### onAdminSave

Используйте событие `onAdminSave()` для управления данными объекта страницы `$object` перед их сохранением в файловой системе.

### onAdminAfterSave

После сохранения страницы в панели админки это событие запускается.

### onAdminAfterSaveAs

При создании папки через панель это событие срабатывает сразу после создания новой папки и выполнения стандартной очистки кэша.

### onAdminAfterDelete

Срабатывает после удаления страницы или папки. Сразу после этого следует стандартная очистка кэша.

### onAdminAfterAddMedia

Срабатывает после завершения задачи добавления мультимедиа, но до отображения подтверждающего сообщения.

### onAdminAfterDelMedia

Срабатывает после завершения задачи удаления мультимедиа, но до отображения подтверждающего сообщения.