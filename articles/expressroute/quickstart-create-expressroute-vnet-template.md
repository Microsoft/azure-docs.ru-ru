---
title: Краткое руководство. Создание канала ExpressRoute с помощью шаблона Azure Resource Manager (шаблона ARM)
description: В этом кратком руководстве показано, как создать канал ExpressRoute с помощью шаблона Azure Resource Manager (шаблона ARM).
services: expressroute
author: duongau
ms.author: duau
manager: kumud
ms.date: 10/12/2020
ms.topic: quickstart
ms.service: expressroute
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 3dc1d5f5ec3dfb004468deb2bec80927c7ec189d
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529888"
---
# <a name="quickstart-create-an-expressroute-circuit-with-private-peering-using-an-arm-template"></a>Краткое руководство. Создание канала ExpressRoute с частным пирингом с помощью шаблона ARM

В этом кратком руководстве описано, как использовать шаблон Azure Resource Manager (шаблон ARM) для создания канала ExpressRoute с частным пирингом.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-expressroute-private-peering-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-expressroute-private-peering-vnet).

В этом кратком руководстве описывается создание канала ExpressRoute с *Equinix* в качестве поставщика услуг. Канал будет использовать *SKU уровня "Премиум"* с пропускной способностью *50 Мбит/с*, а также расположением пиринга *Вашингтон, О.К.* . Частный пиринг будет включен с первичной и вторичной подсетью *192.168.10.16/30* и *192.168.10.20/30* соответственно. Вместе со шлюзом *HighPerformance ExpressRoute* также будет создана виртуальная сеть.

:::code language="json" source="~/quickstart-templates/101-expressroute-private-peering-vnet/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

* [**Microsoft.Network/expressRouteCircuits**](/azure/templates/microsoft.network/expressRouteCircuits)
* [**Microsoft.Network/expressRouteCircuits/peerings**](/azure/templates/microsoft.network/expressRouteCircuits/peerings) (используется для включения частного пиринга в канале)
* [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networkSecurityGroups) (группа безопасности сети применяется к подсетям в виртуальной сети)
* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks);
* [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicIPAddresses) (общедоступный IP-адрес используется шлюзом ExpressRoute)
* [**Microsoft.Network/virtualNetworkGateways**](/azure/templates/microsoft.network/virtualNetworkGateways) (для связи виртуальной сети с каналом используется шлюз ExpressRoute)

Дополнительные сведения о шаблонах, связанных с ExpressRoute, см. в разделе [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите **Попробовать** в следующем блоке кода, чтобы открыть Azure Cloud Shell, и следуйте отображающимся инструкциям, чтобы войти в Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-private-peering-vnet/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Подождите, пока появится запрос из консоли.

1. Выберите **Копировать** из предыдущего блока кода, чтобы скопировать сценарий PowerShell.

1. Щелкните правой кнопкой в области консоли оболочки и выберите **Вставить**.

1. Введите значения.

    Имя группы ресурсов — это имя проекта с добавлением **rg**.

    Развертывание шаблона занимает около 20 минут. По завершении выходные данные должны быть следующего вида:

    :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-powershell-output.png" alt-text="Результаты развертывания шаблона Resource Manager ExpressRoute в PowerShell":::

Для развертывания шаблона используется Azure PowerShell. В дополнение к Azure PowerShell можно также использовать портал Azure, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

1. Войдите на [портал Azure](https://portal.azure.com).

1. В области слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. Имя группы ресурсов по умолчанию — это имя проекта с добавлением **rg**.

1. Группа ресурсов должна содержать следующие ресурсы:

     :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-resource-group.png" alt-text="Группа ресурсов развертывания ExpressRoute":::

1. Выберите канал ExpressRoute **er-ck01**, чтобы убедиться, что канал имеет состояние **Включен**, поставщик — состояние **Не подготовлено**, а частный пиринг — состояние **Подготовлен**.

    :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-circuit.png" alt-text="Канал развертывания ExpressRoute":::

> [!NOTE]
> Прежде чем связать виртуальную сеть с каналом, необходимо будет вызвать поставщик для завершения процесса подготовки.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные вместе с каналом ExpressRoute, удалите эту группу ресурсов. Это приведет к удалению канала ExpressRoute и всех связанных с ним ресурсов.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:

* Канал ExpressRoute
* Виртуальная сеть
* VPN-шлюз
* Общедоступный IP-адрес
* группы безопасности сети.

Чтобы узнать, как связать виртуальную сеть с каналом, см. руководства по ExpressRoute.

> [!div class="nextstepaction"]
> [Учебники по ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
