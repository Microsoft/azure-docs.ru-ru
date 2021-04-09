---
title: Создание виртуальной машины SQL Server с помощью шаблона ARM
description: Узнайте, как создать сервер SQL Server на виртуальной машине Azure с помощью шаблона Azure Resource Manager (шаблона ARM).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: virtual-machines-sql
ms.subservice: deployment
ms.openlocfilehash: a20cb27ac91a0b01ed9cc3a5ac4c5c57f90ceda1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97359648"
---
# <a name="quickstart-create-sql-server-vm-using-an-arm-template"></a>Краткое руководство. Создание виртуальной машины SQL Server с помощью шаблона ARM

Используйте этот шаблон Azure Resource Manager (шаблон ARM) для развертывания сервера SQL Server на виртуальной машине Azure. 

[!INCLUDE [About Azure Resource Manager](../../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать шаблон ARM для развертывания виртуальной машины SQL Server, требуется следующее:

- Последняя версия [Azure CLI](/cli/azure/install-azure-cli) и (или) [PowerShell](/powershell/scripting/install/installing-powershell). 
- Предварительно настроенная [группа ресурсов](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) с подготовленной [виртуальной сетью](../../../virtual-network/quick-create-portal.md) и [подсетью](../../../virtual-network/virtual-network-manage-subnet.md#add-a-subnet).
- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.


## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-sql-vm-new-storage/).

:::code language="json" source="~/quickstart-templates/101-sql-vm-new-storage/azuredeploy.json":::

В шаблоне определено пять ресурсов Azure: 

- [Microsoft.Network/publicIpAddresses.](/azure/templates/microsoft.network/publicipaddresses) Создает общедоступный IP-адрес. 
- [Microsoft.Network/networkSecurityGroups.](/azure/templates/microsoft.network/networksecuritygroups) Создает группу безопасности сети. 
- [Microsoft.Network/networkInterfaces:](/azure/templates/microsoft.network/networkinterfaces) настраивает сетевой интерфейс. 
- [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines): создает виртуальную машину в Azure. 
- [Microsoft.SqlVirtualMachine/SqlVirtualMachines](/azure/templates/microsoft.sqlvirtualmachine/sqlvirtualmachines): регистрирует виртуальную машину в расширении агента IaaS для SQL. 

Другие шаблоны для SQL Server на виртуальных машинах Azure можно найти в [коллекции шаблонов быстрого запуска](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sqlvirtualmachine&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает виртуальную машину с нужной версией SQL Server и регистрирует ее в расширении агента IaaS для SQL. 

   [![Развертывание в Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

2. Введите или выберите следующие значения.

    * **Подписка**: Выберите подписку Azure.
    * **Группа ресурсов.** Подготовленная группа ресурсов для виртуальной машины SQL Server. 
    * **Регион**. Выберите регион.  Например, **центральная часть США**.
    * **Имя виртуальной машины**. Введите имя виртуальной машины SQL Server. 
    * **Размер виртуальной машины.** Выберите нужный размер виртуальной машины из раскрывающегося списка.
    * **Имя существующей виртуальной сети.** Введите имя подготовленной виртуальной сети для виртуальной машины SQL Server. 
    * **Существующая группа ресурсов виртуальной сети.** Введите имя группы ресурсов, в которой была подготовлена виртуальная сеть. 
    * **Имя существующей подсети.** Имя подготовленной подсети. 
    * **Предложение образа.** Выберите образ SQL Server и Windows Server, соответствующий потребностям вашего бизнеса. 
    * **SKU SQL.** Выберите номер SKU SQL Server, соответствующий потребностям вашего бизнеса. 
    * **Имя администратора**. Имя пользователя администратора виртуальной машины. 
    * **Пароль администратора**. Пароль, используемый для учетной записи администратора виртуальной машины. 
    * **Тип рабочей нагрузки хранилища.**  Тип хранилища для рабочей нагрузки, соответствующий потребностям вашего бизнеса. 
    * **Количество дисков данных для SQL.**  Количество дисков, которые SQL Server использует для файлов данных.  
    * **Путь к данным.**  Путь к файлам данных SQL Server. 
    * **Количество дисков журналов для SQL.**  Количество дисков, которые SQL Server использует для файлов журналов. 
    * **Путь к журналам.**  Путь к файлам журналов SQL Server. 
    * **Расположение.**  Расположение всех ресурсов (оставьте значение по умолчанию для `[resourceGroup().location]`). 

3. Выберите **Review + create** (Просмотреть и создать). После успешного развертывания виртуальной машины SQL Server вы получите уведомление.

Для развертывания шаблона используется портал Azure. Помимо портала Azure, вы также можете использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../../../azure-resource-manager/templates/deploy-powershell.md).

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
> [Руководство. Создание и развертывание первого шаблона ARM](../../../azure-resource-manager/templates/template-tutorial-create-first-template.md)

Другие способы развертывания виртуальной машины SQL Server см. в следующих статьях: 
- [Портал Azure](create-sql-vm-portal.md)
- [PowerShell](create-sql-vm-powershell.md)

Дополнительные сведения см. в [обзоре SQL Server на виртуальных машинах Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).