---
title: Краткое руководство. Создание группы управления с помощью JavaScript
description: В этом кратком руководстве показано, как с помощью JavaScript создать группу управления для организации ресурсов в иерархию ресурсов.
ms.date: 02/05/2021
ms.topic: quickstart
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 553133f30bd16543444356e8cda3e70458247b5c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107536011"
---
# <a name="quickstart-create-a-management-group-with-javascript"></a>Краткое руководство. Создание группы управления с помощью JavaScript

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Перед началом работы убедитесь, что установлена по меньшей мере версия [Node.js](https://nodejs.org/) 12.

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Настройка приложения

Чтобы обеспечить управление группами управления с помощью JavaScript, необходимо настроить среду. Эта настройка подойдет для всех сред, где можно использовать JavaScript, в том числе [Bash в Windows 10](/windows/wsl/install-win10).

1. Настройте новый проект Node.js, выполнив следующую команду.

   ```bash
   npm init -y
   ```

1. Добавьте ссылку на модуль yargs.

   ```bash
   npm install yargs
   ```

1. Добавьте ссылку на модуль Azure Resource Graph.

   ```bash
   npm install @azure/arm-managementgroups
   ```

1. Добавьте ссылку на библиотеку аутентификации Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Убедитесь, что в файле _package.json_ указана версия `@azure/arm-managementgroups` **1.1.0** или более поздняя, а также версия `@azure/ms-rest-nodeauth` **3.0.5** или более поздняя.

## <a name="create-the-management-group"></a>Создание группы управления

1. Создайте файл с именем _index.js_ и введите следующий код.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const managementGroups = require("@azure/arm-managementgroups");

   if (argv.groupID && argv.displayName) {
       const createMG = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new managementGroups.ManagementGroupsAPI(credentials);
          const result = await client.managementGroups.createOrUpdate(
             groupId: argv.groupID,
             {
                 displayName: argv.displayName
             }
          );
          console.log(result);
       };

       createMG();
   }
   ```

1. Введите следующую команду в окне терминала.

   ```bash
   node index.js --groupID "<NEW_MG_GROUP_ID>" --displayName "<NEW_MG_FRIENDLY_NAME>"
   ```

   Обязательно замените каждый заполнитель маркера `<>` своим _идентификатором группы безопасности_ и _понятным именем группы управления_ соответственно.

   Когда скрипт попытается выполнить аутентификацию, в окне терминала отобразится примерно такое сообщение:

   > Чтобы войти, откройте в браузере страницу https://microsoft.com/devicelogin. Введите код FGB56WJUGK для аутентификации.

   Когда вы пройдете аутентификацию в браузере, скрипт продолжит выполнение.

Результат создания группы управления выводится в консоль.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные библиотеки из приложения, выполните следующую команду.

```bash
npm uninstall @azure/arm-managementgroups @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)
