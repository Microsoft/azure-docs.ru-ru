---
title: Руководство. Управление виртуальными машинами с помощью PowerShell
description: Из этого руководства вы узнаете, как управлять виртуальными машинами Azure с применением Azure RBAC, политик, блокировок и тегов с помощью Azure PowerShell.
author: tfitzmac
ms.service: virtual-machines
ms.workload: infrastructure
ms.topic: tutorial
ms.date: 12/05/2018
ms.author: tomfitz
ms.custom: mvc
ms.openlocfilehash: f0107a28a655097f983e0711c2ccab19f47f7cae
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102554315"
---
# <a name="tutorial-learn-about-windows-virtual-machine-management-with-azure-powershell"></a>Руководство. Управление виртуальными машинами Windows с помощью Azure PowerShell

[!INCLUDE [Resource Manager governance introduction](../../../includes/resource-manager-governance-intro.md)]

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="understand-scope"></a>Общие сведения об области

[!INCLUDE [Resource Manager governance scope](../../../includes/resource-manager-governance-scope.md)]

В этом руководстве к группе ресурсов применяются все параметры управления, поэтому вы сможете легко отменить их по завершении работы.

Создайте группу ресурсов.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

Сейчас группа ресурсов пуста.

## <a name="azure-role-based-access-control"></a>Управление доступом на основе ролей в Azure

Вам нужно, чтобы у пользователей вашей организации был необходимый уровень доступа к этим ресурсам. Вы не хотите предоставлять пользователям неограниченный доступ, но при этом требуется обеспечить им возможность работать. [Управление доступом Azure на основе ролей (Azure RBAC)](../../role-based-access-control/overview.md) позволяет предоставлять пользователям разрешения на выполнение определенных действий в той или иной области.

Для создания и удаления назначений ролей пользователи должны иметь доступ `Microsoft.Authorization/roleAssignments/*`. Такой доступ предоставляется с помощью ролей владельца или администратора доступа пользователей.

Для управления решениями виртуальной машины существует три роли для конкретных ресурсов, предоставляющие необходимый тип доступа:

* [Участник виртуальной машины](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [Участник сети](../../role-based-access-control/built-in-roles.md#network-contributor)
* [Участник учетной записи хранения](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

Чтобы не назначать роли для отдельных пользователей, зачастую бывает проще создать группу Azure Active Directory для пользователей, которым нужно выполнять подобные действия. А затем назначить этой группе соответствующую роль. В этой статье описано, как использовать существующую группу, чтобы управлять виртуальной машиной, или портал, чтобы [создать группу Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Создав новую группу или найдя существующую, воспользуйтесь командой [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment), чтобы назначить группе Azure Active Directory роль участника виртуальных машин для группы ресурсов.  

```azurepowershell-interactive
$adgroup = Get-AzADGroup -DisplayName <your-group-name>

New-AzRoleAssignment -ObjectId $adgroup.id `
  -ResourceGroupName myResourceGroup `
  -RoleDefinitionName "Virtual Machine Contributor"
```

Если возникнет ошибка с сообщением о том, что **участник \<guid> не существует в каталоге**, это значит, что новая группа еще не используется в Azure Active Directory. Попробуйте выполнить команду снова.

Как правило, нужно повторить эту процедуру для *участника сети* и *участника учетной записи хранения*, чтобы назначить пользователей для управления развернутыми ресурсами. В текущей статье эти действия можно пропустить.

## <a name="azure-policy"></a>Политика Azure

Служба [Политика Azure](../../governance/policy/overview.md) позволяет следить за тем, чтобы все ресурсы в подписке соответствовали корпоративным стандартам. Ваша подписка уже имеет несколько определений политики. Чтобы просмотреть доступные определения политик, воспользуйтесь командой [Get-AzPolicyDefinition](/powershell/module/az.resources/get-azpolicydefinition):

```azurepowershell-interactive
(Get-AzPolicyDefinition).Properties | Format-Table displayName, policyType
```

Отобразится список имеющихся определений политики. Политика может относиться к типу **Встроенная** или **Настраиваемая**. Просмотрите определения, которые описывают необходимое вам условие. В этой статье вы назначаете политики, которые:

* ограничивают расположения для всех ресурсов;
* ограничивают номера SKU для виртуальных машин;
* выполняют аудит виртуальных машин, не использующих управляемые диски.

В следующем примере вы получите три определения политик на основе отображаемого имени. Назначьте эти определения группе ресурсов с помощью команды [New-AzPolicyAssignment](/powershell/module/az.resources/new-azpolicyassignment). Для некоторых политик нужно указать допустимые значения, задав значения параметров.

```azurepowershell-interactive
# Values to use for parameters
$locations ="eastus", "eastus2"
$skus = "Standard_DS1_v2", "Standard_E2s_v2"

# Get the resource group
$rg = Get-AzResourceGroup -Name myResourceGroup

# Get policy definitions for allowed locations, allowed SKUs, and auditing VMs that don't use managed disks
$locationDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed locations"}
$skuDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed virtual machine size SKUs"}
$auditDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Audit VMs that do not use managed disks"}

# Assign policy for allowed locations
New-AzPolicyAssignment -Name "Set permitted locations" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $locationDefinition `
  -listOfAllowedLocations $locations

# Assign policy for allowed SKUs
New-AzPolicyAssignment -Name "Set permitted VM SKUs" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $skuDefinition `
  -listOfAllowedSKUs $skus

# Assign policy for auditing unmanaged disks
New-AzPolicyAssignment -Name "Audit unmanaged disks" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $auditDefinition
```

## <a name="deploy-the-virtual-machine"></a>Развертывание виртуальной машины

Роли и политики назначены — теперь все готово для развертывания вашего решения. Размер по умолчанию — Standard_DS1_v2. Он соответствует одному из ваших разрешенных номеров SKU. При выполнении этого шага будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины.

```azurepowershell-interactive
New-AzVm -ResourceGroupName "myResourceGroup" `
     -Name "myVM" `
     -Location "East US" `
     -VirtualNetworkName "myVnet" `
     -SubnetName "mySubnet" `
     -SecurityGroupName "myNetworkSecurityGroup" `
     -PublicIpAddressName "myPublicIpAddress" `
     -OpenPorts 80,3389
```

После завершения развертывания к решению можно применить дополнительные параметры управления.

## <a name="lock-resources"></a>Блокировка ресурсов

[Блокировки ресурсов](../../azure-resource-manager/management/lock-resources.md) не позволяют пользователям в организации случайно удалить или изменить критически важные ресурсы. В отличие от управления доступом на основе ролей, блокировки ресурсов применяют ограничение для всех пользователей и ролей. Можно установить уровень блокировки *CanNotDelete* или *ReadOnly*.

Чтобы заблокировать виртуальную машину и группу безопасности сети, воспользуйтесь командой [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock):

```azurepowershell-interactive
# Add CanNotDelete lock to the VM
New-AzResourceLock -LockLevel CanNotDelete `
  -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup

# Add CanNotDelete lock to the network security group
New-AzResourceLock -LockLevel CanNotDelete `
  -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

Чтобы протестировать блокировки, попробуйте выполнить следующую команду:

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup
```

Появится ошибка с сообщением о том, что операцию удаления не удалось выполнить из-за блокировки. Группу ресурсов можно удалить, только если намеренно снять блокировки. Этот шаг описан в разделе [Очистка ресурсов](#clean-up-resources).

## <a name="tag-resources"></a>Добавление тегов к ресурсам

К ресурсам Azure можно применять [теги](../../azure-resource-manager/management/tag-resources.md), чтобы логически упорядочивать их по категориям. Каждый тег состоит из имени и значения. Например, имя Environment и значение Production можно применить ко всем ресурсам в рабочей среде.

[!INCLUDE [Resource Manager governance tags Powershell](../../../includes/resource-manager-governance-tags-powershell.md)]

Чтобы применить теги к виртуальной машине, воспользуйтесь командой [Set-AzResource](/powershell/module/az.resources/set-azresource):

```azurepowershell-interactive
# Get the virtual machine
$r = Get-AzResource -ResourceName myVM `
  -ResourceGroupName myResourceGroup `
  -ResourceType Microsoft.Compute/virtualMachines

# Apply tags to the virtual machine
Set-AzResource -Tag @{ Dept="IT"; Environment="Test"; Project="Documentation" } -ResourceId $r.ResourceId -Force
```

### <a name="find-resources-by-tag"></a>Поиск ресурсов по тегу

Чтобы найти ресурсы по имени и значению тега, воспользуйтесь командой [Get-AzResource](/powershell/module/az.resources/get-azresource):

```azurepowershell-interactive
(Get-AzResource -Tag @{ Environment="Test"}).Name
```

Возвращаемые значения можно использовать для выполнения задач управления, например, остановки всех виртуальных машин с определенным значением тега.

```azurepowershell-interactive
Get-AzResource -Tag @{ Environment="Test"} | Where-Object {$_.ResourceType -eq "Microsoft.Compute/virtualMachines"} | Stop-AzVM
```

### <a name="view-costs-by-tag-values"></a>Просмотр данных о затратах по значениям тега

[!INCLUDE [Resource Manager governance tags billing](../../../includes/resource-manager-governance-tags-billing.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалить заблокированные группы безопасности сети невозможно до тех пор, пока не будет снята блокировка. Чтобы снять блокировку, воспользуйтесь командой [Remove-AzResourceLock](/powershell/module/az.resources/remove-azresourcelock):

```azurepowershell-interactive
Remove-AzResourceLock -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup
Remove-AzResourceLock -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="manage-costs"></a>Управление затратами

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы создали пользовательский образ виртуальной машины. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * назначение пользователей для роли;
> * применение политик, принудительно внедряющих стандарты;
> * защита важных ресурсов с помощью блокировок;
> * добавление к ресурсам тегов для выставления счетов и управления.

Перейдите к следующему руководству, чтобы узнать, как выявлять изменения и управлять обновлениями пакетов на виртуальной машине Linux.

> [!div class="nextstepaction"]
> [Управление виртуальными машинами](tutorial-config-management.md)
