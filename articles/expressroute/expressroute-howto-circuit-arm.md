---
title: Краткое руководство. Создание и изменение канала с помощью ExpressRoute — Azure PowerShell
description: В этом кратком руководстве показано, как создать, подготовить, проверить, обновить, удалить и отозвать канал ExpressRoute.
services: expressroute
author: duongau
ms.author: duau
ms.date: 10/12/2020
ms.topic: quickstart
ms.service: expressroute
ms.custom:
- mode-api
ms.openlocfilehash: 50307e40526f5cce4d1349180b29fac00fb70943
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534950"
---
# <a name="quickstart-create-and-modify-an-expressroute-circuit-using-azure-powershell"></a>Краткое руководство. Создание и изменение канала ExpressRoute с помощью Azure PowerShell

В этом кратком руководстве показано, как создать канал ExpressRoute, используя командлеты PowerShell и модель развертывания Azure Resource Manager. Вы также сможете проверять состояние каналов, обновлять, удалять или отзывать их.

## <a name="prerequisites"></a>Предварительные требования

* Изучите [предварительные требования](expressroute-prerequisites.md) и [рабочие процессы](expressroute-workflows.md), прежде чем приступить к настройке.
* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Локальная установка Azure PowerShell или Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Создание и предоставление канала ExpressRoute
### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.

[!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

### <a name="get-the-list-of-supported-providers-locations-and-bandwidths"></a>Получение списка поддерживаемых поставщиков, расположений и значений пропускной способности
Перед созданием канала ExpressRoute потребуется список поддерживаемых поставщиков услуг подключения, расположений и вариантов пропускной способности.

Командлет **Get-AzExpressRouteServiceProvider** в PowerShell возвращает эти сведения, которые будут использоваться в последующих действиях.

```azurepowershell-interactive
Get-AzExpressRouteServiceProvider
```

Проверьте, указан ли в списке поставщик услуг подключения. Запишите следующие сведения, которые потребуются позднее при создании канала:

* Имя
* PeeringLocations
* BandwidthsOffered

Теперь все готово к созданию канала ExpressRoute.

### <a name="create-an-expressroute-circuit"></a>Создание канала ExpressRoute
Перед созданием канала ExpressRoute необходимо создать группу ресурсов (если вы этого еще не сделали) с помощью следующей команды:

```azurepowershell-interactive
New-AzResourceGroup -Name "ExpressRouteResourceGroup" -Location "West US"
```

В приведенном ниже примере показано, как создать канал ExpressRoute со скоростью 200 Мбит/с каналом через Equinix в Кремниевой долине. Если вы используете другой поставщик и другие параметры, подставьте в запрос соответствующие данные. Для запроса нового ключа службы воспользуйтесь следующим примером:

```azurepowershell-interactive
New-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "West US" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Equinix" -PeeringLocation "Silicon Valley" -BandwidthInMbps 200
```

Убедитесь, что указаны правильный уровень SKU и семейство SKU:

* Уровень SKU определяет ценовую категорию канала ExpressRoute: [Локальный](expressroute-faqs.md#expressroute-local), "Стандартный" или [Премиум](expressroute-faqs.md#expressroute-premium). Вы можете указать *Локальный*, *"Стандартный" или *Премиум*. Изменить номер SKU *Стандартный / Премиум* на *Локальный* нельзя.
* Семейство SKU определяет тип выставления счетов. Выберите *MeteredData* для тарифного плана с оплатой за трафик или *UnlimitedData* для безлимитного тарифного плана. Тип выставления счетов можно изменить с *MeteredData* на *UnlimitedData*, но не с *UnlimitedData* на *MeteredData*. Для канала уровня *Локальный* всегда используется *UnlimitedData*.

> [!IMPORTANT]
> Выставление счетов за использование ExpressRoute начинается после получения ключа службы. Обязательно выполните эту операцию, как только поставщик услуг подключения будет готов предоставить канал.
>

Ответ будет содержать ключ службы. Подробное описание всех параметров можно получить, выполнив следующую команду.

```azurepowershell-interactive
get-help New-AzExpressRouteCircuit -detailed
```


### <a name="list-all-expressroute-circuits"></a>Получение списка всех каналов ExpressRoute
Чтобы получить список всех созданных вами каналов ExpressRoute, выполните команду **Get-AzExpressRouteCircuit**.

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

Ответ выглядит примерно так:

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState          : Enabled
ServiceProviderProvisioningState  : NotProvisioned
ServiceProviderNotes              :
ServiceProviderProperties         : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                        : **************************************
Peerings                          : []
```

Вы можете получить эти сведения в любое время с помощью командлета `Get-AzExpressRouteCircuit` . Если командлет вызывается без параметров, выводится список всех каналов. Ваш ключ службы показан в поле *ServiceKey*:

```azurepowershell-interactive
Get-AzExpressRouteCircuit
```

Ответ выглядит примерно так:

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

### <a name="send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>Отправка ключа службы поставщику услуг подключения для подготовки
Параметр *ServiceProviderProvisioningState* предоставляет сведения о текущем состоянии подготовки на стороне поставщика услуг. При этом отображается состояние на стороне Майкрософт. Дополнительные сведения о состояниях подготовки канала см. в разделе о [рабочих процессах](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Вновь созданный канал ExpressRoute будет имеет следующее состояние:

```azurepowershell
ServiceProviderProvisioningState : NotProvisioned
CircuitProvisioningState         : Enabled
```

Канал, включаемый поставщиком услуг подключения, переходит в следующее состояние:

```azurepowershell
ServiceProviderProvisioningState : Provisioning
Status                           : Enabled
```

Используемый канал ExpressRoute должен находиться в следующем состоянии:

```azurepowershell
ServiceProviderProvisioningState : Provisioned
CircuitProvisioningState         : Enabled
```

### <a name="periodically-check-the-status-and-the-state-of-the-circuit-key"></a>Периодическая проверка состояния и статуса ключа канала
Проверяя состояние ключа службы, вы можете узнать, когда поставщик подготовит ваш канал. После настройки канала значение параметра *ServiceProviderProvisioningState* изменится на *Provisioned*, как показано в примере ниже.

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

Ответ выглядит примерно так:

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

### <a name="create-your-routing-configuration"></a>Создание конфигурации маршрутизации
Пошаговые инструкции по созданию и изменению пиринга каналов см. в статье [Создание и изменение маршрутизации для канала ExpressRoute](expressroute-howto-routing-arm.md).

> [!IMPORTANT]
> Эти инструкции распространяются только на каналы от поставщиков, предоставляющих услуги подключения второго уровня. Если ваш поставщик услуг подключения предлагает услуги третьего уровня (обычно это IPVPN, например MPLS), то он возьмет на себя настройку маршрутизации и управление ею.
>

### <a name="link-a-virtual-network-to-an-expressroute-circuit"></a>Связывание виртуальной сети с каналом ExpressRoute
Теперь свяжите виртуальную сеть с каналом ExpressRoute. При работе с моделью развертывания Resource Manager пользуйтесь статьей [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md) .

## <a name="getting-the-status-of-an-expressroute-circuit"></a>Получение состояния канала ExpressRoute
Вы можете в любой момент получить эти сведения с помощью командлета **Get-AzExpressRouteCircuit**. Если командлет вызывается без параметров, выводится список всех каналов.

```azurepowershell-interactive
Get-AzExpressRouteCircuit
```

Ответ будет выглядеть примерно так:

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                        "ServiceProviderName": "Equinix",
                                        "PeeringLocation": "Silicon Valley",
                                        "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

Для получения сведений об определенном канале ExpressRoute передайте имя группы ресурсов и имя канала как параметр вызова.

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

Ответ выглядит примерно так:

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                        "Name": "Standard_MeteredData",
                                        "Tier": "Standard",
                                        "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                        "ServiceProviderName": "Equinix",
                                        "PeeringLocation": "Silicon Valley",
                                        "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

Подробное описание всех параметров можно получить, выполнив следующую команду.

```azurepowershell-interactive
get-help Get-AzExpressRouteCircuit -detailed
```

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Изменение канала ExpressRoute
Некоторые свойства канала ExpressRoute можно изменить, не повлияв на подключение.

Вы можете выполнять следующие задачи без простоя:

* включать и отключать надстройку ExpressRoute "Премиум" для канала ExpressRoute; Изменение SKU с уровня *Стандартный / Премиум* на *Локальный* не поддерживается.
* Увеличивать пропускную способность канала ExpressRoute, если в порту имеется доступная емкость. Снижение уровня пропускной способности канала не поддерживается.
* Перейдите с тарифного плана с оплатой за трафик на безлимитный тарифный план. Переход с безлимитного тарифного плана на тарифный план с оплатой за трафик не поддерживается.
* Параметр *Allow Classic Operations*(Разрешить классические операции) можно включать и отключать.

Дополнительные сведения об ограничениях см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).

### <a name="to-enable-the-expressroute-premium-add-on"></a>Включение надстройки ExpressRoute "Премиум"
Вы можете включить надстройку ExpressRoute "Премиум" для существующего канала с помощью следующего фрагмента кода PowerShell:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Premium"
$ckt.sku.Name = "Premium_MeteredData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Теперь для канала включены функции надстройки ExpressRoute "Премиум". С момента успешного выполнения команды мы начнем выставлять счета по тарифам надстройки уровня "Премиум".

### <a name="to-disable-the-expressroute-premium-add-on"></a>Отключение надстройки ExpressRoute "Премиум"
> [!IMPORTANT]
> Если использовать больше ресурсов, чем разрешено для канала "Стандартный", операция может завершиться ошибкой.
>

Обратите внимание на следующие сведения:

* Прежде чем переходить с канала "Премиум" на "Стандартный", убедитесь, что к каналу привязано менее 10 виртуальных сетей. Если этого не сделать, запрос на обновление завершится ошибкой и мы будем выставлять вам счета по тарифам ценовой категории "Премиум".
* Все связи с виртуальными сетями в других геополитических регионах необходимо разорвать. Если этого не сделать, запрос на обновление завершится ошибкой, и мы будем выставлять вам счета по тарифам уровня "Премиум".
* Для частного пиринга таблица маршрутов должна содержать менее 4000 маршрутов. Если таблица маршрутов содержит больше 4000 маршрутов, сеанс BGP будет прерван. Его можно будет снова включить, когда количество объявленных префиксов станет меньше 4000.

Вы можете отключить надстройку ExpressRoute "Премиум" для существующего канала с помощью следующего командлета PowerShell:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Standard"
$ckt.sku.Name = "Standard_MeteredData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>Обновление пропускной способности канала ExpressRoute
Параметры пропускной способности, поддерживаемые для вашего поставщика, приведены в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md). Можно выбрать любой размер, больший, чем размер существующего канала.

> [!IMPORTANT]
> Может потребоваться заново создать канал ExpressRoute, если существующий порт не обеспечивает достаточную емкость. Канал невозможно обновить, если в его расположении нет доступной дополнительной емкости.
>
> Уменьшить пропускную способность канала ExpressRoute без прерывания его работы нельзя. Для снижения пропускной способности нужно будет отозвать канал ExpressRoute и повторно подготовить новый канал ExpressRoute.
>

Выберите необходимый размер и используйте следующую команду для изменения размера канала:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Размер канала будет изменен на стороне Майкрософт. Свяжитесь с поставщиком услуг подключения и попросите обновить конфигурации с его стороны в соответствии с этим изменением. Когда вы сообщите поставщику услуг об изменении размера канала, мы начнем выставлять счета за обновленную пропускную способность.

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>Перевод SKU с измеряемых на неограниченные данные
Изменить SKU канала ExpressRoute можно, используя следующий фрагмент кода PowerShell:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Family = "UnlimitedData"
$ckt.sku.Name = "Premium_UnlimitedData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>Управление доступом к классической среде и среде диспетчера ресурсов
Инструкции см. в статье [Перемещение каналов ExpressRoute из классической модели развертывания в модель Resource Manager](expressroute-howto-move-arm.md).

## <a name="deprovisioning-an-expressroute-circuit"></a><a name="delete"></a>Отзыв канала ExpressRoute
Обратите внимание на следующие сведения:

* Связь между ExpressRoute и всеми виртуальными сетями необходимо разорвать. Если операция завершится ошибкой, проверьте, не привязаны ли к каналу какие-либо виртуальные сети.
* Если подготовка поставщика услуг канала ExpressRoute находится в состоянии **Идет подготовка** или **Подготовлено** то свяжитесь с поставщиком услуг, чтобы отозвать канал с его стороны. Мы будем резервировать ресурсы и выставлять вам счета до тех пор, пока поставщик услуг не завершит отзыв канала и не отправит нам соответствующее уведомление.
* Когда поставщик услуг отзовет канал (состояние подготовки поставщика услуг изменится на **Не подготовлено**), вы сможете удалить этот канал. Выставление счетов за этот канал будет приостановлено.

## <a name="clean-up-resources"></a>Очистка ресурсов

Для удаления канала ExpressRoute выполните следующую команду:

```azurepowershell-interactive
Remove-AzExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"
```

## <a name="next-steps"></a>Дальнейшие шаги

Когда вы вместе с поставщиком создадите и подготовите канал, перейдите к следующему шагу, чтобы настроить пиринг:

> [!div class="nextstepaction"]
> [Создание и изменение маршрутизации для канала ExpressRoute](expressroute-howto-routing-arm.md)
