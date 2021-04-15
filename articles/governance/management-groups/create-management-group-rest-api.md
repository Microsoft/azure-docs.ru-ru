---
title: Краткое руководство. Создание группы управления с помощью REST API
description: В этом кратком руководстве показано, как с помощью REST API создать группу управления для организации ресурсов в иерархию ресурсов.
ms.date: 02/05/2021
ms.topic: quickstart
ms.openlocfilehash: 26a1c0a8025a00155bcf0498e6dfa89a8a73d983
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259138"
---
# <a name="quickstart-create-a-management-group-with-rest-api"></a>Краткое руководство. Создание группы управления с помощью REST API

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Установите [ARMClient](https://github.com/projectkudu/ARMClient), если его у вас еще нет. Это средство, которое отправляет HTTP-запросы к REST API на основе Azure Resource Manager. Вы можете использовать функцию "Попробовать" в документации по REST или средствах, например PowerShell [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) или [Postman](https://www.postman.com).

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-in-rest-api"></a>Создание с помощью REST API

Чтобы создать новую группу управления в REST API, используйте конечную точку [Группы управления — Создание или обновление](/rest/api/managementgroups/managementgroups/createorupdate). В нашем примере группа управления **GroupId** — это _Contoso_.

- Универсальный код ресурса (URI) REST API

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Без текста запроса

**GroupId** — это уникальный создаваемый идентификатор. Этот идентификатор используется в других командах для обращения к этой группе. Его нельзя изменить позже.

Если требуется, чтобы группа управления отображала другое имя на портале Azure, добавьте свойство **properties.displayName** в текст запроса. Например, чтобы создать группу управления с идентификатором **groupId** для _Contoso_ и отображаемым именем _Contoso Group_, используйте следующие конечную точку и текст запроса:

- Универсальный код ресурса (URI) REST API

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Текст запроса

  ```json
  {
    "properties": {
      "displayName": "Contoso Group"
    }
  }
  ```

В приведенных выше примерах новая группа управления создается в корневой группе управления. Чтобы указать другую группу управления в качестве родительской, используйте свойство **properties.parent.id**.

- Универсальный код ресурса (URI) REST API

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Текст запроса

  ```json
  {
    "properties": {
      "displayName": "Contoso Group",
      "parent": {
        "id": "/providers/Microsoft.Management/managementGroups/HoldingGroup"
      }
    }
  }
  ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданную ранее группу управления, используйте конечную точку [Группы управления — Удалить](/rest/api/managementgroups/managementgroups/delete):

- Универсальный код ресурса (URI) REST API

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Без текста запроса

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)
