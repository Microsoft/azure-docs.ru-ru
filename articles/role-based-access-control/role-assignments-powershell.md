---
title: Назначение ролей Azure с помощью Azure PowerShell — Azure RBAC
description: Узнайте, как предоставить доступ к ресурсам Azure для пользователей, групп, субъектов-служб или управляемых удостоверений с помощью Azure PowerShell и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: daveba
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 02/15/2021
ms.author: rolyon
ms.openlocfilehash: 00f663b90f34f3b557329692f844bbbc1bf3207d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100556798"
---
# <a name="assign-azure-roles-using-azure-powershell"></a>Назначение ролей Azure с помощью Azure PowerShell

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control/definition-grant.md)] В этой статье описывается назначение ролей с помощью Azure PowerShell.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Для назначения ролей необходимо иметь следующее:

- `Microsoft.Authorization/roleAssignments/write` разрешения, такие как [администратор доступа пользователей](built-in-roles.md#user-access-administrator) или [владелец](built-in-roles.md#owner)
- [PowerShell в Azure Cloud Shell](../cloud-shell/overview.md) или [Azure PowerShell](/powershell/azure/install-az-ps)
- Учетная запись, используемая для выполнения команды PowerShell, должна иметь `Directory.Read.All` разрешение Microsoft Graph.

## <a name="steps-to-assign-an-azure-role"></a>Действия по назначению роли Azure

Назначение роли состоит из трех элементов: субъекта безопасности, определения роли и области.

### <a name="step-1-determine-who-needs-access"></a>Шаг 1. Определение пользователей, которым требуется доступ

Роль можно назначить пользователю, группе, субъекту-службе или управляемому удостоверению. Чтобы назначить роль, может потребоваться указать уникальный идентификатор объекта. Идентификатор имеет следующий формат: `11111111-1111-1111-1111-111111111111`. Идентификатор можно получить с помощью портал Azure или Azure PowerShell.

**Пользователь**

Для пользователя Azure AD получите имя участника-пользователя, например *патлонг \@ contoso.com* или идентификатор объекта пользователя. Чтобы получить идентификатор объекта, можно использовать [Get-азадусер](/powershell/module/az.resources/get-azaduser).

```azurepowershell
Get-AzADUser -StartsWith <userName>
(Get-AzADUser -DisplayName <userName>).id
```

**Группа**

Для группы Azure AD требуется идентификатор объекта группы. Чтобы получить идентификатор объекта, можно использовать [Get-азадграуп](/powershell/module/az.resources/get-azadgroup).

```azurepowershell
Get-AzADGroup -SearchString <groupName>
(Get-AzADGroup -DisplayName <groupName>).id
```

**Субъект-служба**

Для субъекта-службы Azure AD (удостоверения, используемого приложением) необходим идентификатор объекта субъекта-службы. Чтобы получить идентификатор объекта, можно использовать [Get-азадсервицепринЦипал](/powershell/module/az.resources/get-azadserviceprincipal). Для субъекта-службы используйте идентификатор объекта, а **не** идентификатор приложения.

```azurepowershell
Get-AzADServicePrincipal -SearchString <principalName>
(Get-AzADServicePrincipal -DisplayName <principalName>).id
```

**Управляемое удостоверение**

Для назначенного системой или управляемого пользователем удостоверения требуется идентификатор объекта. Чтобы получить идентификатор объекта, можно использовать [Get-азадсервицепринЦипал](/powershell/module/az.resources/get-azadserviceprincipal).

```azurepowershell
Get-AzADServicePrincipal -SearchString <principalName>
(Get-AzADServicePrincipal -DisplayName <principalName>).id
```
    
### <a name="step-2-select-the-appropriate-role"></a>Шаг 2. Выбор соответствующей роли

Разрешения объединяются в роли. Вы можете выбрать из списка несколько [встроенных ролей Azure](built-in-roles.md) или использовать собственные пользовательские роли. Рекомендуется предоставлять доступ с минимальными необходимыми привилегиями, поэтому не следует назначать более широкие роли.

Чтобы вывести список ролей и получить уникальный идентификатор роли, можно использовать [Get-азроледефинитион](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition | FT Name, IsCustom, Id
```

Вот как можно вывести сведения о конкретной роли.

```azurepowershell
Get-AzRoleDefinition <roleName>
```

Дополнительные сведения см. в разделе [список определений ролей Azure](role-definitions-list.md#azure-powershell).
 
### <a name="step-3-identify-the-needed-scope"></a>Шаг 3. Определение требуемой области

Azure предоставляет четыре уровня области: ресурс, [Группа ресурсов](../azure-resource-manager/management/overview.md#resource-groups), подписка и [Группа управления](../governance/management-groups/overview.md). Рекомендуется предоставлять доступ с минимальными необходимыми привилегиями, поэтому не следует назначать роль в более широкой области. Дополнительные сведения об областях см. в статье [Общие сведения об областях для Azure RBAC](scope-overview.md).

**Область ресурса**

Для области ресурсов требуется идентификатор ресурса. Идентификатор ресурса можно найти, просмотрев свойства ресурса в портал Azure. Идентификатор ресурса имеет следующий формат.

```
/subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/<providerName>/<resourceType>/<resourceSubType>/<resourceName>
```

**Область группы ресурсов**

Для области группы ресурсов требуется имя группы ресурсов. Имя можно найти на странице " **группы ресурсов** " в портал Azure или можно использовать [Get-азресаурцеграуп](/powershell/module/az.resources/get-azresourcegroup).

```azurepowershell
Get-AzResourceGroup
```

**Область действия подписки** 

Для области подписки необходим идентификатор подписки. Идентификатор можно найти на странице " **подписки** " в портал Azure или можно использовать [Get-азсубскриптион](/powershell/module/az.accounts/get-azsubscription).

```azurepowershell
Get-AzSubscription
```

**Область группы управления** 

Для области группы управления требуется имя группы управления. Имя можно найти на странице " **группы управления** " в портал Azure или можно использовать [Get-азманажементграуп](/powershell/module/az.resources/get-azmanagementgroup).

```azurepowershell
Get-AzManagementGroup
```
    
### <a name="step-4-assign-role"></a>Шаг 4. Назначение роли

Чтобы назначить роль, используйте команду [New-азролеассигнмент](/powershell/module/az.resources/new-azroleassignment) . В зависимости от области команда обычно имеет один из следующих форматов.

**Область ресурса**

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/<providerName>/<resourceType>/<resourceSubType>/<resourceName>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionId <roleId> `
-ResourceName <resourceName> `
-ResourceType <resourceType> `
-ResourceGroupName <resourceGroupName>
```

**Область группы ресурсов**

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-ResourceGroupName <resourceGroupName>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-ResourceGroupName <resourceGroupName>
```

**Область действия подписки** 

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>
```

**Область группы управления** 

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-Scope /providers/Microsoft.Management/managementGroups/<groupName>
``` 

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /providers/Microsoft.Management/managementGroups/<groupName>
``` 
    
## <a name="assign-role-examples"></a>Примеры назначения ролей

#### <a name="assign-a-role-for-all-blob-containers-in-a-storage-account-resource-scope"></a>Назначение роли для всех контейнеров больших двоичных объектов в области ресурсов учетной записи хранения

Назначает роль [участника данных большого двоичного объекта хранилища](built-in-roles.md#storage-blob-data-contributor) субъекту-службе с идентификатором объекта *55555555-5555-5555-5555-555555555555* в области действия ресурса для учетной записи хранения с именем *storage12345*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 55555555-5555-5555-5555-555555555555 `
-RoleDefinitionName "Storage Blob Data Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/providers/Microsoft.Authorization/roleAssignments/cccccccc-cccc-cccc-cccc-cccccccccccc
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345
DisplayName        : example-identity
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 55555555-5555-5555-5555-555555555555
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-specific-blob-container-resource-scope"></a>Назначение роли для определенной области ресурсов контейнера больших двоичных объектов

Назначает роль [участника данных большого двоичного объекта хранилища](built-in-roles.md#storage-blob-data-contributor) субъекту-службе с идентификатором объекта *55555555-5555-5555-5555-555555555555* в области действия ресурса для контейнера больших двоичных объектов с именем *BLOB-Container-01*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 55555555-5555-5555-5555-555555555555 `
-RoleDefinitionName "Storage Blob Data Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01/providers/Microsoft.Authorization/roleAssignm
                     ents/dddddddd-dddd-dddd-dddd-dddddddddddd
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01
DisplayName        : example-identity
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 55555555-5555-5555-5555-555555555555
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-group-in-a-specific-virtual-network-resource-scope"></a>Назначение роли для группы в определенной области ресурсов виртуальной сети

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) группе *"Администраторы продаж Pharma"* с идентификатором аааааааа-АААА-АААА-АААА-аааааааааааа в области ресурсов для виртуальной сети с именем *Pharma-Sales-Project-Network*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceName pharma-sales-project-network `
-ResourceType Microsoft.Network/virtualNetworks `
-ResourceGroupName MyVirtualNetworkResourceGroup

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network/providers/Microsoft.Authorizat
                     ion/roleAssignments/bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network
DisplayName        : Pharma Sales Admins
SignInName         :
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
ObjectType         : Group
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-resource-group-scope"></a>Назначение роли для пользователя в области группы ресурсов

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) пользователю *патлонг \@ contoso.com* в области группы ресурсов *Pharma-Sales* .

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/pr
                     oviders/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

Кроме того, можно указать полное имя группы ресурсов с помощью `-Scope` параметра:

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-using-the-unique-role-id-at-a-resource-group-scope"></a>Назначение роли для пользователя с помощью уникального идентификатора роли в области группы ресурсов

Существует несколько случаев, когда имя роли может измениться, например:

- Вы используете собственную пользовательскую роль и решили изменить ее имя.
- Вы используете роль предварительного просмотра с именем **(Предварительная версия)** . При освобождении роли роль переименовывается.

Даже если роль переименована, идентификатор роли не изменяется. Если вы используете скрипты или автоматизацию для создания назначений ролей, рекомендуется использовать уникальный идентификатор роли вместо имени роли. Таким образом, если роль переименована, скорее всего, сценарии будут работать.

В следующем примере роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) назначается пользователю *патлонг \@ contoso.com* в области группы ресурсов *Pharma-Sales* .

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 44444444-4444-4444-4444-444444444444 `
-RoleDefinitionId 9980e02c-c2be-4d73-94e8-173b1dc7cf3c `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

#### <a name="assign-a-role-for-an-application-at-a-resource-group-scope"></a>Назначение роли для приложения в области группы ресурсов

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) приложению с идентификатором объекта субъекта-службы 77777777-7777-7777-7777-777777777777 в области группы ресурсов *Pharma — Sales* .

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 77777777-7777-7777-7777-777777777777 `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/66666666-6666-6666-6666-666666666666
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : MyApp1
SignInName         :
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 77777777-7777-7777-7777-777777777777
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-subscription-scope"></a>Назначение роли для пользователя в области действия подписки

Назначает роль [читателя](built-in-roles.md#reader) пользователю *аннм \@ example.com* в области действия подписки.

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName annm@example.com `
-RoleDefinitionName "Reader" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/66666666-6666-6666-6666-666666666666
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
DisplayName        : Ann M
SignInName         : annm@example.com
RoleDefinitionName : Reader
RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
ObjectId           : 77777777-7777-7777-7777-777777777777
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-management-group-scope"></a>Назначение роли для пользователя в области группы управления

Назначает роль [читателя выставления счетов](built-in-roles.md#billing-reader) пользователю *(Alain \@ example.com* в области группы управления.

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName alain@example.com `
-RoleDefinitionName "Billing Reader" `
-Scope "/providers/Microsoft.Management/managementGroups/marketing-group"

RoleAssignmentId   : /providers/Microsoft.Management/managementGroups/marketing-group/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
Scope              : /providers/Microsoft.Management/managementGroups/marketing-group
DisplayName        : Alain Charon
SignInName         : alain@example.com
RoleDefinitionName : Billing Reader
RoleDefinitionId   : fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

## <a name="next-steps"></a>Дальнейшие действия

- [Вывод списка назначений ролей Azure с помощью Azure PowerShell](role-assignments-list-powershell.md)
- [Руководство по Предоставление доступа группам к ресурсам Azure с помощью Azure PowerShell](tutorial-role-assignments-group-powershell.md)
- [Управление ресурсами с помощью Azure PowerShell](../azure-resource-manager/management/manage-resources-powershell.md)
