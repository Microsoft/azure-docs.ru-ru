---
title: Краткое руководство. Создание внутреннего балансировщика нагрузки с помощью шаблона
description: В этом кратком руководстве показано, как создать внутреннюю подсистему балансировки нагрузки Azure с помощью шаблона Azure Resource Manager (шаблона ARM).
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.author: allensu
ms.date: 09/14/2020
ms.openlocfilehash: 00126dde55ffe584be611ddf268bb759e127d7a1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788751"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-by-using-an-arm-template"></a>Краткое руководство. Создание внутренней подсистемы балансировки нагрузки для распределения нагрузки между виртуальными машинами с помощью шаблона ARM

В этом руководстве объясняется, как создать внутреннюю подсистему балансировки нагрузки Azure с использованием шаблона Azure Resource Manager (ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-2-vms-internal-load-balancer%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят со страницы [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/201-2-vms-internal-load-balancer).

:::code language="json" source="~/quickstart-templates/201-2-vms-internal-load-balancer/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Storage/storageAccounts.**](/azure/templates/microsoft.storage/storageaccounts) Учетная запись хранения виртуальной машины для диагностики загрузки.
- [**Microsoft.Compute/availabilitySets.**](/azure/templates/microsoft.compute/availabilitySets) Группа доступности для виртуальных машин.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks). Виртуальная сеть для подсистемы балансировки нагрузки и виртуальных машин.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkInterfaces). Сетевые интерфейсы для виртуальных машин.
- [**Microsoft.Network/loadBalancers.**](/azure/templates/microsoft.network/loadBalancers) Внутренний балансировщик нагрузки.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualMachines): виртуальные машины.

Чтобы найти дополнительные связанные шаблоны Azure Load Balancer, перейдите на страницу [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

**Azure CLI**

```azurecli-interactive
read -p "Enter the location (i.e. westcentralus): " location
resourceGroupName="myResourceGroupLB"
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json"

az group create \
--name $resourceGroupName \
--location $location

az deployment group create \
--resource-group $resourceGroupName \
--template-uri  $templateUri
```

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Войдите на [портал Azure](https://portal.azure.com).

1. В области слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. Имя группы ресурсов по умолчанию — **myResourceGroupLB**.

1. Убедитесь, что в группе ресурсов созданы следующие ресурсы:

:::image type="content" source="media/quickstart-load-balancer-standard-internal-template/verify-creation.png" alt-text="Проверка создания ресурсов на портале Azure." border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
  az group delete \
    --name myResourceGroupLB
```

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона см. в следующей статье:

> [!div class="nextstepaction"]
> Создание и развертывание первого шаблона ARM[
