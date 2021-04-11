---
title: Краткое руководство. Настройка высокого уровня доступности с помощью Azure Front Door — Azure PowerShell
description: В этом кратком руководстве показано, как использовать Azure Front Door для создания высокодоступного и высокопроизводительного глобального веб-приложения с помощью Azure PowerShell.
services: front-door
documentationcenter: na
author: duongau
manager: KumudD
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/21/2020
ms.author: duau
ms.openlocfilehash: a3ecb8cacd8fa47709432e26243bd754511658d2
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106057921"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-powershell"></a>Краткое руководство. Создание Front Door для глобального веб-приложения высокой доступности с помощью Azure PowerShell

Начните работу с Azure Front Door, используя Azure PowerShell для создания высокодоступного и высокопроизводительного глобального веб-приложения.

Front Door направляет веб-трафик к конкретным ресурсам в серверном пуле. Определив интерфейсный домен, добавьте ресурсы в серверный пул и создайте правило маршрутизации. В этой статье используется простая конфигурация одного серверного пула с двумя ресурсами веб-приложения и одним правилом маршрутизации с использованием согласования путей по умолчанию "/*".

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-resource-group"></a>Создать группу ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupFD -Location centralus
```

## <a name="create-two-instances-of-a-web-app"></a>Создание двух экземпляров веб-приложения

Для этого краткого руководства потребуются два экземпляра веб-приложения, которые выполняются в разных регионах Azure. Оба экземпляра этого веб-приложения работают в режиме активный — активный, то есть любой из них будет принимать трафик. Эта конфигурация отличается от конфигурации активный — резервный, когда один из экземпляров используется только для отработки отказа.

Если у вас нет веб-приложения, создайте два его экземпляра, используя показанный ниже скрипт.

```azurepowershell-interactive
# Create first web app in Central US region.
$webapp1 = New-AzWebApp `
-Name "WebAppContoso-$(Get-Random)" `
-Location centralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanCentralUS

# Create second web app in South Central US region.
$webapp2 = New-AzWebApp `
-Name "WebAppContoso-$(Get-Random)" `
-Location southcentralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanSouthCentralUS
```

## <a name="create-a-front-door"></a>Создание профиля Front Door

В этом разделе описано, как создать и настроить следующие компоненты Front Door:
    
* Интерфейсный объект содержит домен Front Door по умолчанию.
* Серверный пул — это набор эквивалентных серверных компонентов, для которых Front Door выполняет распределение клиентских запросов.
* Правило маршрутизации сопоставляет интерфейсный узел и шаблон пути соответствующего URL-адреса с указанным серверным пулом.

### <a name="create-a-frontend-object"></a>Создание интерфейсного объекта

Интерфейсный объект настраивает имя узла для Front Door. По умолчанию имя узла будет иметь суффикс * *.azurefd.net*.

```azurepowershell-interactive
# Create a unique name
$fdname = "contoso-frontend-$(Get-Random)"

#Create the frontend object
$FrontendEndObject = New-AzFrontDoorFrontendEndpointObject `
-Name "frontendEndpoint1" `
-HostName $fdname".azurefd.net"
```

### <a name="create-the-backend-pool"></a>Создание внутреннего пула

Серверный пул состоит из двух веб-приложений, созданных в начале этого краткого руководства. В параметрах проверки работоспособности и балансировки нагрузки, определенных на этом шаге, используются значения по умолчанию.

```azurepowershell-interactive
# Create backend objects that points to the hostname of the web apps
$backendObject1 = New-AzFrontDoorBackendObject `
-Address $webapp1.DefaultHostName
$backendObject2 = New-AzFrontDoorBackendObject `
-Address $webapp2.DefaultHostName

# Create a health probe object
$HealthProbeObject = New-AzFrontDoorHealthProbeSettingObject `
-Name "HealthProbeSetting"

# Create the load balancing setting object
$LoadBalancingSettingObject = New-AzFrontDoorLoadBalancingSettingObject `
-Name "Loadbalancingsetting" `
-SampleSize "4" `
-SuccessfulSamplesRequired "2" `
-AdditionalLatencyInMilliseconds "0"

# Create a backend pool using the backend objects, health probe, and load balancing settings
$BackendPoolObject = New-AzFrontDoorBackendPoolObject `
-Name "myBackendPool" `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-Backend $backendObject1,$backendObject2 `
-HealthProbeSettingsName "HealthProbeSetting" `
-LoadBalancingSettingsName "Loadbalancingsetting"
```

### <a name="create-a-routing-rule"></a>Создание правила маршрутизации

Правило маршрутизации сопоставляет серверный пул с интерфейсным доменом и задает для пути по умолчанию значение "/*".

```azurepowershell-interactive
# Create a routing rule mapping the frontend host to the backend pool
$RoutingRuleObject = New-AzFrontDoorRoutingRuleObject `
-Name LocationRule `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-FrontendEndpointName "frontendEndpoint1" `
-BackendPoolName "myBackendPool" `
-PatternToMatch "/*"
```
### <a name="create-the-front-door"></a>Создание Front Door

Теперь, когда вы создали необходимые объекты, создайте Front Door:

```azurepowershell-interactive
# Creates the Front Door
New-AzFrontDoor `
-Name $fdname `
-ResourceGroupName myResourceGroupFD `
-RoutingRule $RoutingRuleObject `
-BackendPool $BackendPoolObject `
-FrontendEndpoint $FrontendEndObject `
-LoadBalancingSetting $LoadBalancingSettingObject `
-HealthProbeSetting $HealthProbeObject
```

После успешного развертывания его можно протестировать, выполнив действия, описанные в следующем разделе.

## <a name="test-the-front-door"></a>Тестирование Front Door

Выполните следующие команды, чтобы получить имя узла для Front Door.

```azurepowershell-interactive
# Gets Front Door in resource group and output the hostname of the frontend domain.
$fd = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD
$fd.FrontendEndpoints[0].Hostname
```

Откройте веб-браузер и введите имя узла, полученное из команд. Front Door направляет запрос к одному из внутренних ресурсов. 

:::image type="content" source="./media/quickstart-create-front-door-powershell/front-door-test-page.png" alt-text="Тестовая страница Front Door":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью Front Door, удалите группу ресурсов. Удалив ее, вы также удалите Front Door и все связанные с ним ресурсы. 

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupFD
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:
* Front Door
* два веб-приложения.

Сведения о добавлении личного домена в Front Door см. в следующих учебниках.

> [!div class="nextstepaction"]
> [Добавить личный домен](front-door-custom-domain.md)
