---
title: Краткое руководство. Создание профиля для обеспечения высокого уровня доступности приложений с помощью Azure PowerShell — Диспетчер трафика Azure
description: В этом кратком руководстве описано, как создать профиль диспетчера трафика для сборки высокодоступного веб-приложения.
services: traffic-manager
author: duongau
mnager: twooley
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/01/2020
ms.author: duau
ms.openlocfilehash: ed3f6c96f6c02d5dad686947ee7f61e8852b559f
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106063837"
---
# <a name="quickstart-create-a-traffic-manager-profile-for-a-highly-available-web-application-using-azure-powershell"></a>Краткое руководство. Создание профиля диспетчера трафика для высокодоступного веб-приложения с помощью Azure PowerShell

В этом кратком руководстве показано, как создать профиль диспетчера трафика, который обеспечивает высокий уровень доступности веб-приложения.

В этом кратком руководстве вы создадите два экземпляра веб-приложения. Каждый из них выполняется в разном регионе Azure. Будет создан профиль диспетчера трафика, который основывается на [приоритете конечной точки](traffic-manager-routing-methods.md#priority-traffic-routing-method). Профиль будет направлять пользовательский трафик к первичному сайту, который запускает веб-приложение. Диспетчер трафика постоянно отслеживает веб-приложение. Если основной сайт недоступен, он предоставляет автоматический переход на резервный сайт.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

```azurepowershell-interactive

# Variables
$Location1="WestUS"

# Create a Resource Group
New-AzResourceGroup -Name MyResourceGroup -Location $Location1
```

## <a name="create-a-traffic-manager-profile"></a>Создание профиля диспетчера трафика

С помощью командлета [New-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile) создайте профиль диспетчера трафика, который направляет пользовательский трафик на основании приоритета конечной точки.

```azurepowershell-interactive

# Generates a random value
$Random=(New-Guid).ToString().Substring(0,8)
$mytrafficmanagerprofile="mytrafficmanagerprofile$Random"

New-AzTrafficManagerProfile `
-Name $mytrafficmanagerprofile `
-ResourceGroupName MyResourceGroup `
-TrafficRoutingMethod Priority `
-MonitorPath '/' `
-MonitorProtocol "HTTP" `
-RelativeDnsName $mytrafficmanagerprofile `
-Ttl 30 `
-MonitorPort 80
```

## <a name="create-web-apps"></a>Создание веб-приложений

Для этого краткого руководства необходимо развернуть два экземпляра веб-приложения в разных регионах Azure (*западная часть США* и *восточная часть США*). Каждый будет использоваться в качестве основных конечных точек и конечных точек отработки отказа для диспетчера трафика.

### <a name="create-web-app-service-plans"></a>Создание планов службы веб-приложений
С помощью командлета [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) создайте план службы веб-приложений для двух экземпляров веб-приложения, которые будут развернуты в двух разных регионах Azure.

```azurepowershell-interactive

# Variables
$App1Name="AppServiceTM1$Random"
$App2Name="AppServiceTM2$Random"
$Location1="WestUS"
$Location2="EastUS"

# Create an App service plan
New-AzAppservicePlan -Name "$App1Name-Plan" -ResourceGroupName MyResourceGroup -Location $Location1 -Tier Standard
New-AzAppservicePlan -Name "$App2Name-Plan" -ResourceGroupName MyResourceGroup -Location $Location2 -Tier Standard

```
### <a name="create-a-web-app-in-the-app-service-plan"></a>Создание веб-приложения в плане службы приложений
Создайте два экземпляра веб-приложения в плане службы приложений, используя командлет [New-AzWebApp](/powershell/module/az.websites/new-azwebapp), в таких регионах Azure, как *западная часть США* и *восточная часть США*.

```azurepowershell-interactive
$App1ResourceId=(New-AzWebApp -Name $App1Name -ResourceGroupName MyResourceGroup -Location $Location1 -AppServicePlan "$App1Name-Plan").Id
$App2ResourceId=(New-AzWebApp -Name $App2Name -ResourceGroupName MyResourceGroup -Location $Location2 -AppServicePlan "$App2Name-Plan").Id

```

## <a name="add-traffic-manager-endpoints"></a>Добавление конечных точек диспетчера трафика
Чтобы добавить два веб-приложения в качестве конечных точек в профиль диспетчера трафика, используйте командлет [New-AzTrafficManagerEndpoint](/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) и сделайте следующее.
- Добавьте веб-приложение в регион Azure *западная часть США* как основную конечную точку для маршрутизации всего пользовательского трафика. 
- Добавьте веб-приложение в регион Azure *восточная часть США* как конечную точку отработки отказа. Если основная конечная точка недоступна, трафик автоматически направляется на конечную точку отработки отказа.

```azurepowershell-interactive
New-AzTrafficManagerEndpoint -Name "$App1Name-$Location1" `
-ResourceGroupName MyResourceGroup `
-ProfileName "$mytrafficmanagerprofile" `
-Type AzureEndpoints `
-TargetResourceId $App1ResourceId `
-EndpointStatus "Enabled"

New-AzTrafficManagerEndpoint -Name "$App2Name-$Location2" `
-ResourceGroupName MyResourceGroup `
-ProfileName "$mytrafficmanagerprofile" `
-Type AzureEndpoints `
-TargetResourceId $App2ResourceId `
-EndpointStatus "Enabled"
```

## <a name="test-traffic-manager-profile"></a>Тестирование профиля диспетчера трафика

В этом разделе произойдет проверка доменного имени профиля диспетчера трафика. Основная конечная точка должна быть недоступной. В результате вы увидите, что веб-приложение по-прежнему доступно. Причиной этого является отправление трафика к конечной точке отработки отказа диспетчером трафика.

### <a name="determine-the-dns-name"></a>Определение DNS-имени

Определите DNS-имя профиля диспетчера трафика с помощью командлета [Get-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/get-aztrafficmanagerprofile).

```azurepowershell-interactive
Get-AzTrafficManagerProfile -Name $mytrafficmanagerprofile `
-ResourceGroupName MyResourceGroup
```

Скопируйте значение **RelativeDnsName**. DNS-именем в профиле диспетчера трафика будет *http://<* relativednsname *>.trafficmanager.net*. 

### <a name="view-traffic-manager-in-action"></a>Просмотр диспетчера трафика в действии
1. Введите DNS-имя своего профиля диспетчера трафика (*http://<* relativednsname *>.trafficmanager.net*) в веб-браузере, чтобы просмотреть веб-сайт по умолчанию для веб-приложения.

    > [!NOTE]
    > В этом кратком сценарии все запросы направляются к основной конечной точке, которой присваивается **Приоритет 1**.
2. Чтобы просмотреть отработку отказа диспетчера трафика в действии, отключите свой основной сайт с помощью командлета [Disable-AzTrafficManagerEndpoint](/powershell/module/az.trafficmanager/disable-aztrafficmanagerendpoint).

   ```azurepowershell-interactive
    Disable-AzTrafficManagerEndpoint -Name $App1Name-$Location1 `
    -Type AzureEndpoints `
    -ProfileName $mytrafficmanagerprofile `
    -ResourceGroupName MyResourceGroup `
    -Force
   ```
3. Скопируйте DNS-имя своего профиля диспетчера трафика (*http://<* relativednsname *>.trafficmanager.net*), чтобы просмотреть веб-сайт в новом сеансе веб-браузера.
4. Убедитесь, что веб-приложение по-прежнему доступно.

## <a name="clean-up-resources"></a>Очистка ресурсов

Закончив работу, удалите группы ресурсов, веб-приложения и все связанные ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали профиль диспетчера трафика, который обеспечивает высокий уровень доступности веб-приложения. Дополнительные сведения о маршрутизации трафика см. в руководствах по диспетчеру трафика.

> [!div class="nextstepaction"]
> [Руководства по диспетчеру трафика](tutorial-traffic-manager-improve-website-response.md)
