---
title: Краткое руководство. Создание Диспетчера трафика с помощью шаблона Azure Resource Manager (шаблон ARM)
description: В этой статье краткого руководства описано, как создать профиль Диспетчера трафика Azure с помощью шаблона Azure Resource Manager (шаблон ARM).
services: traffic-manager
author: duongau
ms.author: duau
ms.date: 09/01/2020
ms.topic: quickstart
ms.service: traffic-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 03342d33731ed29b60908044cd29d529aaa5677b
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531245"
---
# <a name="quickstart-create-a-traffic-manager-profile-using-an-arm-template"></a>Краткое руководство. Создание профиля Диспетчера трафика, используя шаблон ARM

В этом кратком руководстве описано, как использовать шаблон Azure Resource Manager (шаблон ARM) для создания профиля Диспетчера трафика с внешними конечными точками, используя метод маршрутизации производительности.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-traffic-manager-external-endpoint%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-traffic-manager-external-endpoint).

:::code language="json" source="~/quickstart-templates/101-traffic-manager-external-endpoint/azuredeploy.json":::

В этом шаблоне определяется один ресурс Azure.

* [**Microsoft.Network/trafficManagerProfiles**](/azure/templates/microsoft.network/trafficmanagerprofiles)

Чтобы найти дополнительные связанные шаблоны Диспетчера трафика Azure, перейдите на страницу [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите **Попробовать** в следующем блоке кода, чтобы открыть Azure Cloud Shell, и следуйте отображающимся инструкциям, чтобы войти в Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-traffic-manager-external-endpoint/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Подождите, пока появится запрос из консоли.

1. Выберите **Копировать** из предыдущего блока кода, чтобы скопировать сценарий PowerShell.

1. Щелкните правой кнопкой в области консоли оболочки и выберите **Вставить**.

1. Введите значения.

    При развертывании шаблона создается профиль с двумя внешними конечными точками. **Endpoint1** использует целевую конечную точку `www.microsoft.com` с расположением в регионе **Северная Европа**. **Endpoint2** использует целевую конечную точку `docs.microsoft.com` с расположением в регионе **Центрально-южная часть США**.

    Имя группы ресурсов — это имя проекта с добавлением **rg**.

    > [!NOTE]
    > Чтобы шаблон можно было успешно развернуть, имя **uniqueDNSname** должно быть глобально уникальным. Если развертывание завершается неудачно, начните с шага 1.

    Развертывание шаблона занимает несколько минут. По завершении выходные данные должны быть следующего вида:

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-arm-powershell-output.png" alt-text="Результат развертывания шаблона Resource Manager для Диспетчера трафика Azure в PowerShell":::

Для развертывания шаблона используется Azure PowerShell. В дополнение к Azure PowerShell можно также использовать портал Azure, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

1. Определите DNS-имя профиля диспетчера трафика с помощью командлета [Get-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/get-aztrafficmanagerprofile).

    ```azurepowershell-interactive
    Get-AzTrafficManagerProfile -Name ExternalEndpointExample -ResourceGroupName $resourceGroupName | Select RelativeDnsName
    ```

    Скопируйте значение **RelativeDnsName**. DNS-имя вашего профиля Диспетчера трафика: `<relativednsname>.trafficmanager.net`.

1. Из локальной среды PowerShell выполните следующую команду, заменив переменную **{relativeDNSname}** на `<relativednsname>.trafficmanager.net`.

    ```powershell
    Resolve-DnsName -Name {relativeDNSname} | Select-Object NameHost | Select -First 1
    ```

    Вы должны получить имя NameHost `www.microsoft.com` или `docs.microsoft.com` в зависимости от ближайшего региона.

1. Чтобы проверить, можно ли разрешить доступ к другой конечной точке, отключите конечную точку для целевого объекта, полученного на последнем шаге. Замените **{endpointName}** на **endpoint1** или **endpoint2**, чтобы отключить целевой объект для `www.microsoft.com` или `docs.microsoft.com` соответственно.

    ```azurepowershell-interactive
    Disable-AzTrafficManagerEndpoint -Name {endpointName} -Type ExternalEndpoints -ProfileName ExternalEndpointExample -ResourceGroupName $resourceGroupName -Force
    ```

1. Снова выполните команду из шага 2 в локальной среде PowerShell. На этот раз вы должны получить другое имя NameHost для другой конечной точки.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если профиль Диспетчера трафика больше не нужен, удалите группу ресурсов. При этом будет удален профиль Диспетчера трафика и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве был создан профиль диспетчера трафика.

Дополнительные сведения о маршрутизации трафика см. в руководствах по диспетчеру трафика.

> [!div class="nextstepaction"]
> [Руководства по диспетчеру трафика](tutorial-traffic-manager-improve-website-response.md)
