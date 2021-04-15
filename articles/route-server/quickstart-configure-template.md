---
title: Краткое руководство. Создание Azure Route Server с помощью шаблона Azure Resource Manager (ARM)
description: В этом кратком руководстве показано, как создать Azure Route Server с помощью шаблона Azure Resource Manager (шаблона ARM).
services: route-server
author: duongau
ms.service: route-server
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 04/05/2021
ms.author: duau
ms.openlocfilehash: 6f56b9fb1f6a1f5a1fe0811617fb20412c52fd72
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450840"
---
# <a name="quickstart-create-an-azure-route-server-using-an-arm-template"></a>Краткое руководство. Создание Azure Route Server с помощью шаблона ARM

В этом кратком руководстве описано, как использовать шаблон Azure Resource Manager (шаблон ARM) для развертывания сервера Azure Route Server в новой или существующей виртуальной сети.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-route-server%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-route-server).

В этом кратком руководстве вы развернете сервер маршрутов Azure Route Server в новой или существующей виртуальной сети. Для размещения сервера маршрутов будет создана выделенная подсеть с именем `RouteServerSubnet`. На сервере маршрутизации также будет настроен одноранговый ASN и одноранговый IP-адрес для установки пиринга BGP.

:::code language="json" source="~/quickstart-templates/101-route-server/azuredeploy.json" range="001-145" highlight="105-142":::

В шаблоне определено несколько ресурсов Azure:

* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks);
* [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/microsoft.network/virtualNetworks/subnets) (две подсети, одна с именем `routeserversubnet`)
* [**Microsoft.Network/virtualHubs**](/azure.templates/microsoft.network/virtualhubs) (развертывание Route Server)
* [**Microsoft.Network/virtualHubs/ipConfigurations**](/azure.templates/microsoft.network/virtualhubs/ipConfigurations)
* [**Microsoft.Network/virtualHubs/bgpConnections**](/azure.templates/microsoft.network/virtualhubs/bgpConnections) (одноранговый узел ASN и конфигурация IP-адреса узла)


Дополнительные сведения о шаблонах, связанных с ExpressRoute, см. в разделе [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите **Попробовать** в следующем блоке кода, чтобы открыть Azure Cloud Shell, и следуйте отображающимся инструкциям, чтобы войти в Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-route-server/azuredeploy.json"

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

    :::image type="content" source="./media/quickstart-configure-template/powershell-output.png" alt-text="Результаты развертывания шаблона Resource Manager Route Server в PowerShell":::

Для развертывания шаблона используется Azure PowerShell. В дополнение к Azure PowerShell можно также использовать портал Azure, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

1. Войдите на [портал Azure](https://portal.azure.com).

1. В области слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. Имя группы ресурсов по умолчанию — это имя проекта с добавлением **rg**.

1. Группа ресурсов должна содержать только виртуальную сеть.

     :::image type="content" source="./media/quickstart-configure-template/resource-group.png" alt-text="Группа ресурсов для развертывания Route Server с виртуальной сетью.":::

1. Перейдите к https://aka.ms/routeserver.

1. Выберите сервер маршрутизации с именем **routeserver**, чтобы убедиться, что развертывание прошло успешно.

    :::image type="content" source="./media/quickstart-configure-template/deployment.png" alt-text="Экран &quot;Снимок экрана: страница обзора сервера маршрутизации&quot;.":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью Route Server, удалите группу ресурсов. При этом будет удалена служба Route Server и все связанные с ней ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:

* Route Server
* Виртуальная сеть
* Подсеть

Создав сервер маршрутизации Azure, вы можете продолжить изучение способов взаимодействия между сервером маршрутизации Azure с ExpressRoute и VPN-шлюзами: 

> [!div class="nextstepaction"]
> [Поддержка Azure ExpressRoute и VPN-шлюза Azure](expressroute-vpn-support.md)
