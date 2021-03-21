---
title: Вывод списка назначений ролей Azure с помощью Azure PowerShell — Azure RBAC
description: Узнайте, как определить, к каким ресурсам пользователи, группы, субъекты-службы или управляемые удостоверения имеют доступ с помощью Azure PowerShell и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/28/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: bf5445f6ca04e56aab466e97967a58c3e4b735a4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100556936"
---
# <a name="list-azure-role-assignments-using-azure-powershell"></a>Вывод списка назначений ролей Azure с помощью Azure PowerShell

[!INCLUDE [Azure RBAC definition list access](../../includes/role-based-access-control/definition-list.md)] В этой статье описывается, как вывести список назначений ролей с помощью Azure PowerShell.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

> [!NOTE]
> Если в вашей организации есть функции управления с использованием внешнего источника для поставщика услуг, использующего [Управление делегированными ресурсами Azure](../lighthouse/concepts/azure-delegated-resource-management.md), назначения ролей, предоставленные этим поставщиком услуг, не будут показаны здесь.

## <a name="prerequisites"></a>Предварительные условия

- [PowerShell в Azure Cloud Shell](../cloud-shell/overview.md) или [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="list-role-assignments-for-the-current-subscription"></a>Вывод списка назначений ролей для текущей подписки

Самый простой способ получить список всех назначений ролей в текущей подписке (включая наследуемые назначения ролей из корневых и групп управления) — использовать [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment) без параметров.

```azurepowershell
Get-AzRoleAssignment
```

```Example
PS C:\> Get-AzRoleAssignment

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
DisplayName        : Alain
SignInName         : alain@example.com
RoleDefinitionName : Storage Blob Data Reader
RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Marketing
SignInName         :
RoleDefinitionName : Contributor
RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
ObjectId           : 22222222-2222-2222-2222-222222222222
ObjectType         : Group
CanDelegate        : False

...
```

## <a name="list-role-assignments-for-a-subscription"></a>Получение списка назначений ролей для подписки

Чтобы получить список всех назначений ролей в области действия подписки, используйте [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment). Чтобы получить идентификатор подписки, его можно найти в колонке **подписки** в портал Azure или воспользоваться [Get-азсубскриптион](/powershell/module/Az.Accounts/Get-AzSubscription).

```azurepowershell
Get-AzRoleAssignment -Scope /subscriptions/<subscription_id>
```

```Example
PS C:\> Get-AzRoleAssignment -Scope /subscriptions/00000000-0000-0000-0000-000000000000
```

## <a name="list-role-assignments-for-a-user"></a>Список назначений ролей для пользователя

Чтобы получить список всех ролей, назначенных соответствующему пользователю, используется команда [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -SignInName <email_or_userprincipalname>
```

```Example
PS C:\> Get-AzRoleAssignment -SignInName isabella@example.com | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
```

Чтобы получить список всех ролей, назначенных указанному пользователю и группам, к которым он принадлежит, используется команда [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -SignInName <email_or_userprincipalname> -ExpandPrincipalGroups
```

```Example
Get-AzRoleAssignment -SignInName isabella@example.com -ExpandPrincipalGroups | FL DisplayName, RoleDefinitionName, Scope
```

## <a name="list-role-assignments-for-a-resource-group"></a>Вывод списка назначений ролей для группы ресурсов

Чтобы получить список всех назначений ролей в области группы ресурсов, используйте [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -ResourceGroupName <resource_group_name>
```

```Example
PS C:\> Get-AzRoleAssignment -ResourceGroupName pharma-sales | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Alain Charon
RoleDefinitionName : Backup Operator
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales

DisplayName        : Alain Charon
RoleDefinitionName : Virtual Machine Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
```

## <a name="list-role-assignments-for-a-management-group"></a>Список назначений ролей для группы управления

Чтобы получить список всех назначений ролей в области группы управления, используйте [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment). Чтобы получить идентификатор группы управления, его можно найти в колонке " **группы управления** " в портал Azure или можно использовать [Get-азманажементграуп](/powershell/module/az.resources/get-azmanagementgroup).

```azurepowershell
Get-AzRoleAssignment -Scope /providers/Microsoft.Management/managementGroups/<group_id>
```

```Example
PS C:\> Get-AzRoleAssignment -Scope /providers/Microsoft.Management/managementGroups/marketing-group
```

## <a name="list-role-assignments-for-a-resource"></a>Вывод списка назначений ролей для ресурса

Чтобы вывести список назначений ролей для определенного ресурса, используйте командлет [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment) и `-Scope` параметр. Область будет отличаться в зависимости от ресурса. Чтобы получить область, можно запустить `Get-AzRoleAssignment` без параметров, чтобы вывести список всех назначений ролей, а затем найти область, которую нужно вывести в список.

```azurepowershell
Get-AzRoleAssignment -Scope "/subscriptions/<subscription_id>/resourcegroups/<resource_group_name>/providers/<provider_name>/<resource_type>/<resource>
```

В следующем примере показано, как получить список назначений ролей для учетной записи хранения. Обратите внимание, что эта команда также выводит список назначений ролей в более высоких областях, таких как группы ресурсов и подписки, которые применяются к этой учетной записи хранения.

```Example
PS C:\> Get-AzRoleAssignment -Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/storage-test-rg/providers/Microsoft.Storage/storageAccounts/storagetest0122"
```

Если вы хотите просто перечислить назначения ролей, назначенные непосредственно в ресурсе, можно использовать команду [Where-Object](/powershell/module/microsoft.powershell.core/where-object) для фильтрации списка.

```Example
PS C:\> Get-AzRoleAssignment | Where-Object {$_.Scope -eq "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/storage-test-rg/providers/Microsoft.Storage/storageAccounts/storagetest0122"}
```

## <a name="list-role-assignments-for-classic-service-administrator-and-co-administrators"></a>Вывод списка назначений ролей классического администратора и соадминистраторов службы

Чтобы получить список назначений ролей для администратора и соадминистраторов классической подписки, используйте командлет [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -IncludeClassicAdministrators
```

## <a name="list-role-assignments-for-a-managed-identity"></a>Вывод списка назначений ролей для управляемого удостоверения

1. Возвращает идентификатор объекта назначенного системой или управляемого пользователем удостоверения. 

    Чтобы получить идентификатор объекта для управляемого пользователем удостоверения, можно использовать [Get-азадсервицепринЦипал](/powershell/module/az.resources/get-azadserviceprincipal).

    ```azurepowershell
    Get-AzADServicePrincipal -DisplayNameBeginsWith "<name> or <vmname>"
    ```

1. Чтобы получить список назначений ролей, используйте [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment).

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId <objectid>
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение ролей Azure с помощью Azure PowerShell](role-assignments-powershell.md)