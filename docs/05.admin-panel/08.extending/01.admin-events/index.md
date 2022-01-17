# Хуки событий админки

> Список доступных хуков админки Grav CMS.

Плагин админки имеет несколько обработчиков событий, которые можно использовать во время [жизненного цикла Grav](/04.plugins/05.grav-lifecycle/index). См. общую документацию плагина для использования обработчиков событий в главе [Плагины](/04.plugins/index).

## Доступные хуки событий админки
* [onAdminTaskExecute](#onAdminTaskExecute)
* [onAdminCreatePageFrontmatter](#onAdminCreatePageFrontmatter)
* [onAdminSave](#onAdminSave)
* [onAdminAfterSave](#onAdminAfterSave)
* [onAdminAfterSaveAs](#onAdminAfterSaveAs)
* [onAdminAfterDelete](#onAdminAfterDelete)
* [onAdminAfterAddMedia](#onAdminAfterAddMedia)
* [onAdminAfterDelMedia](#onAdminAfterDelMedia)


## Включение хука событий админки
Перехватчики событий админки вызываются [таким же образом](/04.plugins/03.plugin-tutorial/index?id=Шаг-7-Определите-следует-ли-запускать-плагин), как и основные перехватчики событий.


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