---
title: Список запрещенных назначений Azure с помощью Azure PowerShell — Azure RBAC
description: Узнайте, как получить список пользователей, групп, субъектов-служб и управляемых удостоверений, которым запрещен доступ к определенным действиям ресурсов Azure в определенных областях с помощью Azure PowerShell и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/12/2019
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 6cf379b051ba42be2d7df7d288f07cdc2a0002a1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94657641"
---
# <a name="list-azure-deny-assignments-using-azure-powershell"></a>Список запрещенных назначений в Azure с помощью Azure PowerShell

[Запрещенные назначения Azure](deny-assignments.md) запрещают пользователям выполнять определенные действия с ресурсами Azure, даже если назначение ролей предоставляет им доступ. В этой статье описывается, как вывести список запрещенных назначений с помощью Azure PowerShell.

> [!NOTE]
> Вы не можете напрямую создавать собственные назначения Deny. Дополнительные сведения о создании назначений Deny см. в статье [запрещение назначений в Azure](deny-assignments.md).

## <a name="prerequisites"></a>Предварительные условия

Чтобы получить сведения о назначении Deny, необходимо следующее:

- `Microsoft.Authorization/denyAssignments/read` разрешение, которое входит в большинство [встроенных ролей Azure](built-in-roles.md)
- [PowerShell в Azure Cloud Shell](../cloud-shell/overview.md) или [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="list-deny-assignments"></a>Отображение списка запрещенных назначений

### <a name="list-all-deny-assignments"></a>Перечислить все назначения Deny

Чтобы получить список всех отклонения назначений для текущей подписки, используйте [Get-азденяссигнмент](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment
```

```Example
PS C:\> Get-AzDenyAssignment

Id                      : 22222222-2222-2222-2222-222222222222
DenyAssignmentName      : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          ObjectType:   ServicePrincipal
                          }
IsSystemProtected       : True

Id                      : 33333333-3333-3333-3333-333333333333
DenyAssignmentName      : Deny assignment '33333333-3333-3333-3333-333333333333' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks/providers/Microsoft.Storage/storageAccounts/storep6vkuxmu4m4pq
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          DisplayName:  assignment-locked-storageaccount-TestingBPLocks
                          ObjectType:   ServicePrincipal
                          ObjectId:     2311a0b7-657a-4ca2-af6f-d1c33f6d2fff
                          }
IsSystemProtected       : True
```

### <a name="list-deny-assignments-at-a-resource-group-scope"></a>Список запретов назначений в области группы ресурсов

Чтобы получить список всех запрещенных назначений в области группы ресурсов, используйте [Get-азденяссигнмент](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment -ResourceGroupName <resource_group_name>
```

```Example
PS C:\> Get-AzDenyAssignment -ResourceGroupName TestingBPLocks | FL DenyAssignmentName, Scope

DenyAssignmentName : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                     '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Scope              : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
Principals         : {
                     DisplayName:       All Principals
                     ObjectType:        SystemDefined
                     ObjectId:  00000000-0000-0000-0000-000000000000
                     }
```

### <a name="list-deny-assignments-at-a-subscription-scope"></a>Список запретов назначений в области действия подписки

Чтобы получить список всех запрещенных назначений в области действия подписки, используйте [Get-азденяссигнмент](/powershell/module/az.resources/get-azdenyassignment). Чтобы получить идентификатор подписки, его можно найти в колонке **подписки** в портал Azure или воспользоваться [Get-азсубскриптион](/powershell/module/Az.Accounts/Get-AzSubscription).

```azurepowershell
Get-AzDenyAssignment -Scope /subscriptions/<subscription_id>
```

```Example
PS C:\> Get-AzDenyAssignment -Scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о назначении Deny в Azure](deny-assignments.md)
- [Список запрещенных назначений в Azure с помощью портал Azure](deny-assignments-portal.md)
- [Список запрещенных назначений в Azure с помощью REST API](deny-assignments-rest.md)