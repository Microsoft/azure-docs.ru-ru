---
title: Удаление назначений ролей Azure в Azure RBAC
description: Узнайте, как удалить доступ к ресурсам Azure для пользователей, групп, субъектов-служб или управляемых удостоверений с помощью портал Azure, Azure PowerShell, Azure CLI или REST API.
services: active-directory
author: rolyon
manager: daveba
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 02/15/2021
ms.author: rolyon
ms.openlocfilehash: 7a3e4853d6dffa7eb2c5cf80846f6f1bd6beba03
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100561510"
---
# <a name="remove-azure-role-assignments"></a>Удаление назначений ролей Azure

[Управление доступом на основе ролей Azure (Azure RBAC)](../../articles/role-based-access-control/overview.md) — это система авторизации, используемая для управления доступом к ресурсам в Azure. Чтобы удалить доступ из ресурса Azure, необходимо удалить назначение роли. В этой статье описывается удаление назначений ролей с помощью портал Azure, Azure PowerShell, Azure CLI и REST API.

## <a name="prerequisites"></a>Предварительные условия

Чтобы удалить назначения ролей, необходимо иметь следующее:

- `Microsoft.Authorization/roleAssignments/delete` разрешения, такие как [администратор доступа пользователей](../../articles/role-based-access-control/built-in-roles.md#user-access-administrator) или [владелец](../../articles/role-based-access-control/built-in-roles.md#owner)

## <a name="azure-portal"></a>Портал Azure

1. Откройте **Управление доступом (IAM)** для области, например группы управления, подписки, группы ресурсов или отдельного ресурса, чтобы заблокировать доступ.

1. Щелкните вкладку **Назначения ролей**, чтобы просмотреть все назначения ролей в этой области.

1. В списке назначений ролей добавьте флажок рядом с именем субъекта безопасности, для которого нужно удалить назначение ролей.

   ![Выбрано назначение ролей для удаления](./media/role-assignments-remove/rg-role-assignments-select.png)

1. Щелкните **Удалить**.

   ![Сообщение об удалении назначения роли](./media/role-assignments-remove/remove-role-assignment.png)

1. В появившемся сообщении об отзыве назначения роли щелкните **Да**.

    Если появится сообщение о том, что наследование назначений ролей не может быть удалено, вы пытаетесь удалить назначение роли в дочерней области. Следует открыть элемент управления доступом (IAM) в области, в которой была назначена роль, и повторить попытку. Чтобы быстро открыть элемент управления доступом (IAM) в нужной области, просмотрите столбец **область** и щелкните ссылку рядом с **(наследуется)**.

   ![Удаление сообщения о назначении роли для наследуемых назначений ролей](./media/role-assignments-remove/remove-role-assignment-inherited.png)

## <a name="azure-powershell"></a>Azure PowerShell

В Azure PowerShell вы удаляете назначение роли с помощью [Remove-азролеассигнмент](/powershell/module/az.resources/remove-azroleassignment).

В следующем примере удаляется назначение роли [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) из пользователя *патлонг \@ contoso.com* в группе ресурсов *Pharma-Sales* .

```azurepowershell
PS C:\> Remove-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales
```

Удаляет роль [читателя](built-in-roles.md#reader) из группы группы *Анна Mack* с идентификатором 22222222-2222-2222-2222-222222222222 в области действия подписки.

```azurepowershell
PS C:\> Remove-AzRoleAssignment -ObjectId 22222222-2222-2222-2222-222222222222 `
-RoleDefinitionName "Reader" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000"
```

Удаляет роль [читателя выставления счетов](built-in-roles.md#billing-reader) из пользователя *(Alain \@ example.com* в области группы управления.

```azurepowershell
PS C:\> Remove-AzRoleAssignment -SignInName alain@example.com `
-RoleDefinitionName "Billing Reader" `
-Scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

При появление сообщения об ошибке "предоставленные сведения не сопоставляются с назначением ролей" убедитесь, что также указаны `-Scope` `-ResourceGroupName` Параметры или. Дополнительные сведения см. в разделе [Устранение неполадок в Azure RBAC](troubleshooting.md#role-assignments-with-identity-not-found).

## <a name="azure-cli"></a>Azure CLI

В Azure CLI вы удаляете назначение роли с помощью команды [AZ Role назначение Delete](/cli/azure/role/assignment#az_role_assignment_delete).

В следующем примере удаляется назначение роли [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) из пользователя *патлонг \@ contoso.com* в группе ресурсов *Pharma-Sales* .

```azurecli
az role assignment delete --assignee "patlong@contoso.com" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

Удаляет роль [читателя](built-in-roles.md#reader) из группы группы *Анна Mack* с идентификатором 22222222-2222-2222-2222-222222222222 в области действия подписки.

```azurecli
az role assignment delete --assignee "22222222-2222-2222-2222-222222222222" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

Удаляет роль [читателя выставления счетов](built-in-roles.md#billing-reader) из пользователя *(Alain \@ example.com* в области группы управления.

```azurecli
az role assignment delete --assignee "alain@example.com" \
--role "Billing Reader" \
--scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

## <a name="rest-api"></a>REST API

В REST API вы удаляете назначение ролей с помощью [назначений ролей — Delete](/rest/api/authorization/roleassignments/delete).

1. Получите идентификатор (GUID) для назначения роли. Его можно получить при первом создании назначения роли, а также при перечислении назначений ролей.

1. Можете начать со следующего запроса:

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}?api-version=2015-07-01
    ```

1. Внутри URI замените *{scope}* областью для удаления назначения роли.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1` | Группа ресурсов |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1/providers/microsoft.web/sites/mysite1` | Ресурс |

1. Замените *{ролеассигнментид}* идентификатором GUID назначения роли.

Следующий запрос удаляет указанное назначение роли в области действия подписки:

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId1}/providers/microsoft.authorization/roleassignments/{roleAssignmentId1}?api-version=2015-07-01
```

Ниже приведен пример выходных данных:

```json
{
    "properties": {
        "roleDefinitionId": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/a795c7a0-d4a2-40c1-ae25-d81f01202912",
        "principalId": "{objectId1}",
        "scope": "/subscriptions/{subscriptionId1}",
        "createdOn": "2020-05-06T23:55:24.5379478Z",
        "updatedOn": "2020-05-06T23:55:24.5379478Z",
        "createdBy": "{createdByObjectId1}",
        "updatedBy": "{updatedByObjectId1}"
    },
    "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId1}",
    "type": "Microsoft.Authorization/roleAssignments",
    "name": "{roleAssignmentId1}"
}
```

## <a name="arm-template"></a>Шаблон ARM

Невозможно удалить назначение роли с помощью шаблона Azure Resource Manager (шаблон ARM). Чтобы удалить назначение ролей, необходимо использовать другие средства, такие как портал Azure, Azure PowerShell, Azure CLI или REST API.

## <a name="next-steps"></a>Дальнейшие действия

- [Вывод списка назначений ролей Azure с помощью портала Azure](role-assignments-list-portal.md).
- [Вывод списка назначений ролей Azure с помощью Azure PowerShell](role-assignments-list-powershell.md)
- [Устранение неполадок в Azure RBAC](troubleshooting.md)
