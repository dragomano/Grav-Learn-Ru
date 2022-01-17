# Разрешения

> Конфигурация пользовательских разрешений в Grav CMS.

Разрешения пользователя и группы для управления информацией, связанной с учётной записью:


| Параметр                              | Значение                          | Описание                                                          |
| :-----                                | :-----                            | :-----                                                            |
| **Настройка**                     | *admin.configuration*             | Предоставляет пользователю доступ к области **Настройка**. |
| &nbsp; &nbsp; **Настройка учётных записей** | *admin.configuration.accounts*    | Предоставляет пользователю доступ к области **Настройка учётных записей** внутри раздела **Аккаунты**.  |
| **Аккаунты**                          | *admin.accounts*                  | Предоставляет пользователю доступ к области **Аккаунты**.    |
| &nbsp; &nbsp; **Создание**              | *admin.accounts.create*           | Предоставляет пользователю доступ к **Созданию** аккаунтов и групп.      |
| &nbsp; &nbsp; **Просмотр**                | *admin.accounts.read*             | Предоставляет пользователю доступ к **Просмотру** аккаунтов и групп.          |
| &nbsp; &nbsp; **Обновление**              | *admin.accounts.upodate*          | Предоставляет пользователю доступ к **Обновлению** аккаунтов и групп.        |
| &nbsp; &nbsp; **Удаление**              | *admin.accounts.delete*           | Предоставляет пользователю доступ к **Удаление** аккаунтов и групп.        |
| &nbsp; &nbsp; **Список**                | *admin.accounts.list*             | Предоставляет пользователю доступ к области **Аккаунты**.             |


Возможные значения разрешений:


| Параметр                              | Значение | Описание                                                                                                   |
| :-----                                | :-----   | :-----                                                                                                     |
| **Разрешено**                         | `true`   | **Разрешает** действие, если не установлен **Запрет**.                                                     |
| **Запрещено**                         | `false`  | **Запрещает** действие. При наличии разрешения и запрета на одно и то же действие в приоритете **Запрет**. |
| **Не задано**                         | `null`   | Никакого эффекта, но действует как **Запрет**, если никакие другие правила не применяются.                 |


Разрешения, установленные специально для учётной записи пользователя, имеют приоритет над разрешениями группы. Если разрешение не было задано в учётной записи, проверка доступа будет выполнена для всех групп пользователей, к которым принадлежит пользователь. Если какая-либо из групп пользователей **отказала** в действии, пользователь не имеет разрешения на это действие. В противном случае, если какая-либо из групп пользователей **разрешила** действие, разрешение будет предоставлено. Если разрешение не было установлено ни в одной из групп пользователей, то разрешение **суперпользователь** действует как универсальное **разрешено**, в противном случае будет применено **отказано**.