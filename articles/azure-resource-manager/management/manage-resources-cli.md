---
title: Управление ресурсами — Azure CLI
description: Используйте Azure CLI и Azure Resource Manager для управления ресурсами. Показывает, как развертывать и удалять ресурсы.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 077ebdb4dd33249923064a4f5ed20c5641a82e26
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97695883"
---
# <a name="manage-azure-resources-by-using-azure-cli"></a>Управление ресурсами Azure с помощью Azure CLI

Узнайте, как использовать Azure CLI с [Azure Resource Manager](overview.md) для управления ресурсами Azure. Сведения об управлении группами ресурсов см. в статье [Управление группами ресурсов Azure с помощью Azure CLI](manage-resource-groups-cli.md).

Другие статьи об управлении ресурсами:

- [Управление ресурсами Azure с помощью портал Azure](manage-resources-portal.md)
- [Управление ресурсами Azure с помощью Azure PowerShell](manage-resources-powershell.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>Развертывание ресурсов в существующей группе ресурсов

Ресурсы Azure можно развернуть непосредственно с помощью Azure CLI или развернуть шаблон диспетчер ресурсов для создания ресурсов Azure.

### <a name="deploy-a-resource"></a>Развертывание ресурса

Следующий скрипт создает учетную запись хранения.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account create --resource-group $resourceGroupName --name $storageAccountName --location $location --sku Standard_LRS --kind StorageV2 &&
az storage account show --resource-group $resourceGroupName --name $storageAccountName 
```

### <a name="deploy-a-template"></a>Развертывание шаблона

Следующий скрипт создает шаблон быстрого запуска для создания учетной записи хранения. Дополнительные сведения см. [в разделе Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell).

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az deployment group create --resource-group $resourceGroupName --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

Дополнительные сведения см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../templates/deploy-cli.md).

## <a name="deploy-a-resource-group-and-resources"></a>Развертывание группы ресурсов и ресурсов

Вы можете создать группу ресурсов и развернуть в нее ресурсы. См. дополнительные сведения в разделе [Создание группы ресурсов и развертывание ресурсов](../templates/deploy-to-subscription.md#resource-groups).

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>Развертывание ресурсов в нескольких подписках или группах ресурсов

Обычно развертывание всех ресурсов в шаблоне выполняется в отдельную группу ресурсов. Тем не менее возможны ситуации, когда необходимо развернуть набор ресурсов одновременно, но при этом разместить их в отдельных подписках или группах ресурсов. Дополнительные сведения см. [в статье Развертывание ресурсов Azure в нескольких подписках или группах ресурсов](../templates/deploy-to-resource-group.md).

## <a name="delete-resources"></a>Удаление ресурсов

В следующем сценарии показано, как удалить учетную запись хранения.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --resource-group $resourceGroupName --name $storageAccountName 
```

Дополнительные сведения о том, как Azure Resource Manager упорядочивают удаление ресурсов, см. в разделе [Azure Resource Manager удаление группы ресурсов](delete-resource-group.md).

## <a name="move-resources"></a>Перемещение ресурсов

В следующем сценарии показано, как удалить учетную запись хранения из одной группы ресурсов в другую.

```azurecli-interactive
echo "Enter the source Resource Group name:" &&
read srcResourceGroupName &&
echo "Enter the destination Resource Group name:" &&
read destResourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
storageAccount=$(az resource show --resource-group $srcResourceGroupName --name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --query id --output tsv) &&
az resource move --destination-group $destResourceGroupName --ids $storageAccount
```

Дополнительные сведения см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Блокировка ресурсов

Блокировка запрещает другим пользователям в организации случайно удалить или изменить критически важные ресурсы, такие как подписка Azure, Группа ресурсов или ресурс. 

Следующий скрипт блокирует учетную запись хранения, чтобы учетную запись нельзя было удалить.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock create --name LockSite --lock-type CanNotDelete --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts 
```

Следующий скрипт получает все блокировки для учетной записи хранения:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock list --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --parent ""
```

Следующий скрипт удаляет блокировку учетной записи хранения:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
lockId=$(az lock show --name LockSite --resource-group $resourceGroupName --resource-type Microsoft.Storage/storageAccounts --resource-name $storageAccountName --output tsv --query id)&&
az lock delete --ids $lockId
```

Дополнительные сведения см. [в разделе Блокировка ресурсов с помощью Azure Resource Manager](lock-resources.md).

## <a name="tag-resources"></a>Добавление тегов к ресурсам

Теги позволяют логически упорядочивать группы ресурсов и ресурсы. Дополнительные сведения см. [в статье Использование тегов для Организации ресурсов Azure](tag-resources.md#azure-cli).

## <a name="manage-access-to-resources"></a>Управление доступом к ресурсам

Управление [доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md) — это способ управления доступом к ресурсам в Azure. Дополнительные сведения см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure CLI](../../role-based-access-control/role-assignments-cli.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о Azure Resource Manager см. в разделе [Общие сведения о Azure Resource Manager](overview.md).
- Сведения о синтаксисе шаблона диспетчер ресурсов см. в разделе [Общие сведения о структуре и синтаксисе шаблонов Azure Resource Manager](../templates/template-syntax.md).
- Дополнительные сведения о разработке шаблонов см. в пошаговых [руководствах](../index.yml).
- Сведения о том, как просмотреть схемы шаблонов Azure Resource Manager, см. в разделе [Справочник по шаблонам](/azure/templates/).