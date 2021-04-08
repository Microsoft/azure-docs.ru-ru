---
title: Руководство по Предоставление доступа группам к ресурсам Azure с помощью Azure PowerShell — Azure RBAC
description: Узнайте, как предоставить группам доступ к ресурсам Azure с помощью Azure PowerShell и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/02/2019
ms.author: rolyon
ms.openlocfilehash: e25db6213c99afaa46732831534956b01adba0b4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100555229"
---
# <a name="tutorial-grant-a-group-access-to-azure-resources-using-azure-powershell"></a>Руководство по Предоставление доступа группам к ресурсам Azure с помощью Azure PowerShell

[Управление доступом на основе ролей Azure (Azure RBAC)](overview.md) — это способ управления доступом к ресурсам в Azure. В этом руководстве описано, как предоставлять групповой доступ для просмотра любого содержимого в рамках подписки и обеспечивать полное управление в группе ресурсов с помощью Azure PowerShell.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Предоставление доступа для группы в разных областях
> * Вывод списка доступа
> * Запрет доступа

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Для выполнения этого руководства потребуется следующее:

- разрешения на создание групп в Azure Active Directory (или наличие существующей группы);
- [Azure Cloud Shell](../cloud-shell/quickstart-powershell.md);

## <a name="role-assignments"></a>Назначения ролей

При использовании Azure RBAC для предоставления доступа нужно создать назначение ролей. Назначение ролей состоит из трех элементов: субъект безопасности, определение роли и область действия. Ниже приведены два варианта назначения роли, которые мы выполним в ходе работы с этим руководством.

| Субъект безопасности | Определение роли | Область |
| --- | --- | --- |
| Группа<br>(Группа из руководства по RBAC) | [Читатель](built-in-roles.md#reader) | Подписка |
| Группа<br>(Группа из руководства по RBAC)| [Участник](built-in-roles.md#contributor) | Группа ресурсов<br>(группа-ресурсов-из-руководства-по-rbac) |

   ![Назначения ролей для группы](./media/tutorial-role-assignments-group-powershell/rbac-role-assignments.png)

## <a name="create-a-group"></a>Создание группы

Чтобы назначить роль, нам нужен пользователь, группа или субъект-служба. Если у вас еще нет группы, создайте ее.

- В Azure Cloud Shell создайте группу, используя команду [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup).

   ```azurepowershell
   New-AzureADGroup -DisplayName "RBAC Tutorial Group" `
     -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
   ```

   ```Example
   ObjectId                             DisplayName         Description
   --------                             -----------         -----------
   11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
   ```

Если у вас нет разрешений на создание групп, ознакомьтесь со статьей [Руководство Предоставление доступа пользователям к ресурсам Azure с помощью Azure PowerShell](tutorial-role-assignments-user-powershell.md).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Используйте группу ресурсов, чтобы показать, как назначить роль в области действия группы ресурсов.

1. Получите список расположений в регионе с помощью команды [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

   ```azurepowershell
   Get-AzLocation | select Location
   ```

1. Выберите расположение недалеко от вас и присвойте его переменной.

   ```azurepowershell
   $location = "westus"
   ```

1. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

   ```azurepowershell
   New-AzResourceGroup -Name "rbac-tutorial-resource-group" -Location $location
   ```

   ```Example
   ResourceGroupName : rbac-tutorial-resource-group
   Location          : westus
   ProvisioningState : Succeeded
   Tags              :
   ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
   ```

## <a name="grant-access"></a>Предоставление доступа

Чтобы предоставить доступ для группы, используйте команду [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) для назначения роли. Необходимо указать субъект безопасности, определение роли и область действия.

1. Получите идентификатор объекта группы с помощью команды [Get AzureADGroup](/powershell/module/azuread/new-azureadgroup).

    ```azurepowershell
    Get-AzureADGroup -SearchString "RBAC Tutorial Group"
    ```

    ```Example
    ObjectId                             DisplayName         Description
    --------                             -----------         -----------
    11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
    ```

1. Сохраните идентификатор объекта группы в переменной.

    ```azurepowershell
    $groupId = "11111111-1111-1111-1111-111111111111"
    ```

1. Получите идентификатор подписки с помощью команды [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

    ```azurepowershell
    Get-AzSubscription
    ```

    ```Example
    Name     : Pay-As-You-Go
    Id       : 00000000-0000-0000-0000-000000000000
    TenantId : 22222222-2222-2222-2222-222222222222
    State    : Enabled
    ```

1. Сохраните область действия подписки в переменной.

    ```azurepowershell
    $subScope = "/subscriptions/00000000-0000-0000-0000-000000000000"
    ```

1. Назначьте группе роль [Читатель](built-in-roles.md#reader) в области действия подписки.

    ```azurepowershell
    New-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/44444444-4444-4444-4444-444444444444
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

1. Назначьте роль [Участник](built-in-roles.md#contributor) в области действия группы ресурсов.

    ```azurepowershell
    New-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

## <a name="list-access"></a>Вывод списка доступа

1. Чтобы проверить доступ для подписки, используйте команду [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment), которая отобразит список назначений ролей.

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId $groupId -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    В выходных данных вы увидите, что роль "Читатель" была назначена группе руководства по RBAC в пределах области действия подписки.

1. Чтобы проверить доступ для группы ресурсов, используйте команду [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment), которая отобразит список назначений ролей.

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId $groupId -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    В выходных данных вы увидите, что обе роли — и "Читатель", и "Участник" — были назначены группе руководства по RBAC. Роль "Участник" находится в области действия группы ресурсов из руководства по RBAC, а роль "Читатель" наследуется в области действия подписки.

## <a name="optional-list-access-using-the-azure-portal"></a>(Необязательно.) Вывод списка доступа с помощью портала Azure

1. Чтобы увидеть, как выглядят назначения ролей на портале Azure, просмотрите колонку **Управление доступом (IAM)** для подписки.

    ![Назначение ролей для группы в области действия подписки](./media/tutorial-role-assignments-group-powershell/role-assignments-subscription.png)

1. Просмотрите колонку **Управление доступом (IAM)** для группы ресурсов.

    ![Назначения ролей для группы в области действия группы ресурсов](./media/tutorial-role-assignments-group-powershell/role-assignments-resource-group.png)

## <a name="remove-access"></a>Запрет доступа

Чтобы лишить прав доступа пользователей, группы и приложения, используйте команду [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) для удаления назначений ролей.

1. Используйте следующую команду, чтобы удалить назначение роли участника для группы в области действия группы ресурсов.

    ```azurepowershell
    Remove-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

1. Используйте следующую команду, чтобы удалить назначение роли читателя для группы в области действия подписки.

    ```azurepowershell
    Remove-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов и саму группу.

1. Удалите группу ресурсов с помощью команды [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

    ```azurepowershell
    Remove-AzResourceGroup -Name "rbac-tutorial-resource-group"
    ```

    ```Example
    Confirm
    Are you sure you want to remove resource group 'rbac-tutorial-resource-group'
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    ```
    
1. В запросе на подтверждение выберите **Да**. Удаление займет несколько секунд.

1. Удалите группу с помощью команды [Remove-AzureADGroup](/powershell/module/azuread/remove-azureadgroup).

    ```azurepowershell
    Remove-AzureADGroup -ObjectId $groupId
    ```
    
    Если при попытке удалить группу появится ошибка, можно также удалить группу на портале.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Назначение ролей Azure с помощью Azure PowerShell](role-assignments-powershell.md)