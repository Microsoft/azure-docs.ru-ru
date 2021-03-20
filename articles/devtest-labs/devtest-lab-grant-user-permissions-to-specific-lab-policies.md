---
title: Предоставление пользователю разрешений для определенных политик лаборатории | Документация Майкрософт
description: Узнайте, как предоставить пользователю разрешения для определенных политик лаборатории в DevTest Labs исходя из его потребностей.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 976862476d25e4e9a4933d8a5319eec9d77ca39b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92328476"
---
# <a name="grant-user-permissions-to-specific-lab-policies"></a>Предоставление пользователю разрешений для определенных политик лаборатории
## <a name="overview"></a>Обзор
В этой статье рассказывается, как с помощью PowerShell предоставить пользователям разрешения для определенной политики лаборатории. Разрешения могут предоставляться в зависимости от потребностей каждого пользователя. Например, определенному пользователю можно разрешить изменить параметры политики виртуальной машины, но не политики затрат.

## <a name="policies-as-resources"></a>Политики как ресурсы
Как описано в статье [Управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) , Azure RBAC обеспечивает точное управление доступом к ресурсам для Azure. С помощью Azure RBAC вы можете разделить обязанности в группе DevOps и предоставить доступ только пользователям, которые им необходимы для выполнения своих задач.

В DevTest Labs политика — это тип ресурса, который позволяет действию Azure RBAC **Microsoft. DevTestLab/Labs/policySets/Policies/**. Каждая политика лаборатории является ресурсом в типе ресурса политики и может быть назначена роли Azure в качестве области.

Например, чтобы предоставить пользователям разрешение на чтение и запись для политики **разрешенных размеров виртуальных машин** , создайте пользовательскую роль, которая работает с **Microsoft. DevTestLab/Labs/policySets/** policys/Action, а затем назначьте соответствующие пользователи этой пользовательской роли в области **Microsoft. DevTestLab/Labs/policySets/Policies/AllowedVmSizesInLab**.

Дополнительные сведения о пользовательских ролях в Azure RBAC см. в статье [пользовательские роли Azure](../role-based-access-control/custom-roles.md).

## <a name="creating-a-lab-custom-role-using-powershell"></a>Создание пользовательской роли лаборатории с помощью PowerShell
Чтобы начать работу, необходимо [установить Azure PowerShell](/powershell/azure/install-az-ps). 

Настроив командлеты Azure PowerShell, вы сможете выполнять следующие задачи:

* Получать список всех операций и действий по тому или иному поставщику ресурсов.
* Получать список действий по определенной роли:
* Создание пользовательской роли

Примеры выполнения этих задач демонстрирует следующий сценарий PowerShell:

```azurepowershell
# List all the operations/actions for a resource provider.
Get-AzProviderOperation -OperationSearchString "Microsoft.DevTestLab/*"

# List actions in a particular role.
(Get-AzRoleDefinition "DevTest Labs User").Actions

# Create custom role.
$policyRoleDef = (Get-AzRoleDefinition "DevTest Labs User")
$policyRoleDef.Id = $null
$policyRoleDef.Name = "Policy Contributor"
$policyRoleDef.IsCustom = $true
$policyRoleDef.AssignableScopes.Clear()
$policyRoleDef.AssignableScopes.Add("/subscriptions/<SubscriptionID> ")
$policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/policySets/policies/*")
$policyRoleDef = (New-AzRoleDefinition -Role $policyRoleDef)
```

## <a name="assigning-permissions-to-a-user-for-a-specific-policy-using-custom-roles"></a>Предоставление пользователю разрешений в отношении определенной политики с помощью пользовательских ролей
Определив пользовательские роли, можно назначить их пользователям. Чтобы назначить пользовательскую роль, необходимо получить **ObjectId** соответствующего пользователя. Для этого используйте командлет **Get-азадусер** .

В следующем примере **ObjectId** пользователя *SomeUser* имеет значение 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3.

```azurepowershell
PS C:\>Get-AzADUser -SearchString "SomeUser"

DisplayName                    Type                           ObjectId
-----------                    ----                           --------
someuser@hotmail.com                                          05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3
```

После получения идентификатора **ObjectID** для пользователя и имени пользовательской роли можно назначить эту роль пользователю с помощью командлета **New-азролеассигнмент** :

```azurepowershell
PS C:\>New-AzRoleAssignment -ObjectId 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3 -RoleDefinitionName "Policy Contributor" -Scope /subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.DevTestLab/labs/<LabName>/policySets/default/policies/AllowedVmSizesInLab
```

В предыдущем примере использовалась политика **AllowedVmSizesInLab** . Также можно использовать любую из следующих политик:

* MaxVmsAllowedPerUser
* MaxVmsAllowedPerLab
* AllowedVmSizesInLab
* LabVmsShutdown

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Дальнейшие действия
После того как пользователю будут предоставлены разрешения для определенных политик лаборатории, можно выполнить следующие действия.

* [Безопасный доступ к лаборатории](devtest-lab-add-devtest-user.md)
* [Настройка политик лаборатории](devtest-lab-set-lab-policy.md)
* [Создание лабораторного шаблона](devtest-lab-create-template.md)
* [Создание пользовательских артефактов для виртуальных машин](devtest-lab-artifact-author.md)
* [Добавление виртуальной машины в лабораторию](devtest-lab-add-vm.md)
