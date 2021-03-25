---
title: 'Azure Resource Manager: Создание Управляемого экземпляра SQL Azure'
description: Узнайте, как создать Управляемый экземпляр SQL Azure с помощью шаблона Azure Resource Manager.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: subject-armqs
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 06/22/2020
ms.openlocfilehash: badeb850e8d0346347a994f053fb0f64fd01240a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91283349"
---
# <a name="quickstart-create-an-azure-sql-managed-instance-using-an-arm-template"></a>Краткое руководство. Создание Управляемого экземпляра SQL Azure с помощью шаблона ARM

В этом кратком руководстве показано, как развернуть шаблон Azure Resource Manager (шаблон ARM), чтобы создать Управляемый экземпляр SQL Azure и виртуальную сеть. [Управляемый экземпляр SQL Azure](sql-managed-instance-paas-overview.md) — это полностью управляемая и масштабируемая интеллектуальная облачная база данных с почти полным равенством функций с ядром СУБД SQL Server.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-sqlmi-new-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/).

:::code language="json" source="~/quickstart-templates/101-sqlmi-new-vnet/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.Network/networkSecurityGroups)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.Network/routeTables)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.Network/virtualNetworks);
- [**Microsoft.Sql/managedinstances**](/azure/templates/microsoft.sql/managedinstances)

На странице [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sql&pageNumber=1&sort=Popular) доступны также другие примеры шаблонов.

## <a name="deploy-the-template"></a>Развертывание шаблона

Выберите **Попробовать** из следующего блока кода PowerShell, чтобы открыть Azure Cloud Shell.

> [!IMPORTANT]
> Развертывание управляемого экземпляра — это длительная операция. Развертывание первого экземпляра в подсети обычно занимает гораздо больше времени, чем развертывание в подсети с существующими управляемыми экземплярами. Среднее время подготовки см. в информации об [операциях управления Управляемыми экземплярами SQL](sql-managed-instance-paas-overview.md#management-operations).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-sqlmi-new-vnet/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name that is used for generating resource names:" projectName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-sqlmi-new-vnet/azuredeploy.json" &&
resourceGroupName="${projectName}rg" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Перейдите на [портал Azure](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) и убедитесь, что управляемый экземпляр находится в выбранной группе ресурсов. Поскольку создание управляемого экземпляра может занять некоторое время, возможно, на странице **Обзор** группы ресурсов потребуется перейти по ссылке **Развертывания**.

- См. статью [Краткое руководство. Настройка виртуальной машины Azure для подключения к Управляемому экземпляру SQL](connect-vm-instance-configure.md).
- См. статью [Краткое руководство. Настройка подключения "точка — сеть" к Управляемому экземпляру SQL с локального клиентского компьютера](point-to-site-p2s-configure.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите выполнить [дальнейшие действия](#next-steps), сохраните управляемый экземпляр, но после выполнения инструкций во всех дополнительных учебниках удалите его вместе со связанными с ним ресурсами. После удаления управляемого экземпляра выполните инструкции из статьи [Удаление подсети после удаления управляемого экземпляра](virtual-cluster-delete.md).


Чтобы удалить группу ресурсов, сделайте следующее:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка виртуальной машины Azure для подключения к Управляемому экземпляру SQL Azure](connect-vm-instance-configure.md)
