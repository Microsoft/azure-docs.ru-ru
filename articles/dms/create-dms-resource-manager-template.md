---
title: Создание экземпляра DMS (шаблон Azure Resource Manager)
description: Узнайте, как создать экземпляр Database Migration Service с помощью шаблона Azure Resource Manager (шаблона ARM).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: dms
ms.openlocfilehash: a9c68bca4d50af734a0a2cd8a91c7e46d9b56ff1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94963221"
---
# <a name="quickstart-create-instance-of-azure-database-migration-service-using-arm-template"></a>Краткое руководство. Создание экземпляра Azure Database Migration Service с помощью шаблона ARM

Используйте этот шаблон Azure Resource Manager (шаблон ARM) для развертывания экземпляра Azure Database Migration Service. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Для использования шаблона Azure Database Migration Service ARM требуется следующее: 

- Последняя версия [Azure CLI](/cli/azure/install-azure-cli) и (или) [PowerShell](/powershell/scripting/install/installing-powershell). 
- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azure-database-migration-simple-deploy/).

:::code language="json" source="~/quickstart-templates/101-azure-database-migration-simple-deploy/azuredeploy.json":::

В шаблоне определены три ресурса Azure: 

- [Microsoft.Network/virtualNetworks.](/azure/templates/microsoft.network/virtualnetworks) Создает виртуальную сеть. 
- [Microsoft.Network/virtualNetworks/subnets.](/azure/templates/microsoft.network/virtualnetworks/subnets) Создает подсеть. 
- [Microsoft.DataMigration/services.](/azure/templates/microsoft.datamigration/services) Развертывает экземпляр Azure Database Migration Service. 

Дополнительные шаблоны Azure Database Migration Service доступны в [коллекции шаблонов быстрого запуска](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datamigration&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает экземпляр Azure Database Migration Service. 

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

2. Введите или выберите следующие значения.

    * **Подписка**: Выберите подписку Azure.
    * **Группа ресурсов.** Выберите в раскрывающемся списке существующую группу ресурсов или щелкните **Создать новую**, чтобы создать новую. 
    * **Регион**. Расположение, в котором будут развертываться ресурсы.
    * **Имя службы.** Имя новой службы миграции.
    * **Расположение.** Расположение группы ресурсов. Оставьте значение по умолчанию (`[resourceGroup().location]`).
    * **Имя виртуальной сети.** Имя новой виртуальной сети.
    * **Имя подсети.** Имя новой подсети, связанной с виртуальной сетью.



3. Выберите **Review + create** (Просмотреть и создать). После успешного развертывания экземпляра Azure Database Migration Service вы получите уведомление. 


Для развертывания шаблона используется портал Azure. В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Для проверки развернутых ресурсов можно использовать Azure CLI. 


```azurecli-interactive
echo "Enter the resource group where your SQL Server VM exists:" &&
read resourcegroupName &&
az resource list --resource-group $resourcegroupName 
```


## <a name="clean-up-resources"></a>Очистка ресурсов

Если группа ресурсов вам не нужна, удалите ее с помощью Azure CLI или Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона см. в следующей статье:

> [!div class="nextstepaction"]
> [Руководство. Создание и развертывание первого шаблона ARM](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

Другие способы развертывания Azure Database Migration Service описаны в следующих статьях: 
- [Портал Azure](quickstart-create-data-migration-service-portal.md)

Дополнительные сведения см. в [обзорной статье о Azure Database Migration Service](dms-overview.md)