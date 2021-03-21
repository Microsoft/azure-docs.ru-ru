---
title: Управление ресурсами — Azure PowerShell
description: Используйте Azure PowerShell и Azure Resource Manager для управления ресурсами. Показывает, как развертывать и удалять ресурсы.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: ce3b1d9f8d01e722e4ec5078d92ef4148072da25
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96185748"
---
# <a name="manage-azure-resources-by-using-azure-powershell"></a>Управление ресурсами Azure с помощью Azure PowerShell

Узнайте, как использовать Azure PowerShell с [Azure Resource Manager](overview.md) для управления ресурсами Azure. Сведения об управлении группами ресурсов см. в статье [Управление группами ресурсов Azure с помощью Azure PowerShell](manage-resource-groups-powershell.md).

Другие статьи об управлении ресурсами:

- [Управление ресурсами Azure с помощью портал Azure](manage-resources-portal.md)
- [Управление ресурсами Azure с помощью Azure CLI](manage-resources-cli.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>Развертывание ресурсов в существующей группе ресурсов

Ресурсы Azure можно развернуть непосредственно с помощью Azure PowerShell или развернуть шаблон диспетчер ресурсов для создания ресурсов Azure.

### <a name="deploy-a-resource"></a>Развертывание ресурса

Следующий скрипт создает учетную запись хранения.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

# Create the storage account.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
  -Name $storageAccountName `
  -Location $location `
  -SkuName "Standard_LRS"

# Retrieve the context.
$ctx = $storageAccount.Context
```

### <a name="deploy-a-template"></a>Развертывание шаблона

Следующий скрипт создает шаблон быстрого запуска для создания учетной записи хранения. Дополнительные сведения см. в разделе [Краткое руководство. Создание шаблонов Azure Resource Manager с помощью Visual Studio Code](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell).

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -Location $location
```

Дополнительные сведения см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../templates/deploy-powershell.md).

## <a name="deploy-a-resource-group-and-resources"></a>Развертывание группы ресурсов и ресурсов

Вы можете создать группу ресурсов и развернуть в нее ресурсы. См. дополнительные сведения в разделе [Создание группы ресурсов и развертывание ресурсов](../templates/deploy-to-subscription.md#resource-groups).

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>Развертывание ресурсов в нескольких подписках или группах ресурсов

Обычно развертывание всех ресурсов в шаблоне выполняется в отдельную группу ресурсов. Тем не менее возможны ситуации, когда необходимо развернуть набор ресурсов одновременно, но при этом разместить их в отдельных подписках или группах ресурсов. Дополнительные сведения см. [в статье Развертывание ресурсов Azure в нескольких подписках или группах ресурсов](../templates/deploy-to-resource-group.md).

## <a name="delete-resources"></a>Удаление ресурсов

В следующем сценарии показано, как удалить учетную запись хранения.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Remove-AzStorageAccount -ResourceGroupName $resourceGroupName -AccountName $storageAccountName
```

Дополнительные сведения о том, как Azure Resource Manager упорядочивают удаление ресурсов, см. в разделе [Azure Resource Manager удаление группы ресурсов](delete-resource-group.md).

## <a name="move-resources"></a>Перемещение ресурсов

В следующем сценарии показано, как удалить учетную запись хранения из одной группы ресурсов в другую.

```azurepowershell-interactive
$srcResourceGroupName = Read-Host -Prompt "Enter the source Resource Group name"
$destResourceGroupName = Read-Host -Prompt "Enter the destination Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$storageAccount = Get-AzResource -ResourceGroupName $srcResourceGroupName -ResourceName $storageAccountName
Move-AzResource -DestinationResourceGroupName $destResourceGroupName -ResourceId $storageAccount.ResourceId
```

Дополнительные сведения см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Блокировка ресурсов

Блокировка запрещает другим пользователям в организации случайно удалить или изменить критически важные ресурсы, такие как подписка Azure, Группа ресурсов или ресурс. 

Следующий скрипт блокирует учетную запись хранения, чтобы учетную запись нельзя было удалить.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

New-AzResourceLock -LockName LockStorage -LockLevel CanNotDelete -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts 
```

Следующий скрипт получает все блокировки для учетной записи хранения:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts
```

Следующий скрипт удаляет блокировку учетной записи хранения:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$lockId = (Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts).LockId
Remove-AzResourceLock -LockId $lockId
```

Дополнительные сведения см. [в разделе Блокировка ресурсов с помощью Azure Resource Manager](lock-resources.md).

## <a name="tag-resources"></a>Добавление тегов к ресурсам

Теги позволяют логически упорядочивать группы ресурсов и ресурсы. Дополнительные сведения см. [в статье Использование тегов для Организации ресурсов Azure](tag-resources.md#powershell).

## <a name="manage-access-to-resources"></a>Управление доступом к ресурсам

Управление [доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md) — это способ управления доступом к ресурсам в Azure. Дополнительные сведения см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о Azure Resource Manager см. в разделе [Общие сведения о Azure Resource Manager](overview.md).
- Сведения о синтаксисе шаблона диспетчер ресурсов см. в разделе [Общие сведения о структуре и синтаксисе шаблонов Azure Resource Manager](../templates/template-syntax.md).
- Дополнительные сведения о разработке шаблонов см. в пошаговых [руководствах](../index.yml).
- Сведения о том, как просмотреть схемы шаблонов Azure Resource Manager, см. в разделе [Справочник по шаблонам](/azure/templates/).