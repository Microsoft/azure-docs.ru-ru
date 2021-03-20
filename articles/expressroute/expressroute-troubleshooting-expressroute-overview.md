---
title: 'Azure ExpressRoute: Проверка подключения. Руководство по устранению неполадок'
description: В этой статье содержатся инструкции по устранению неполадок и проверке сквозного подключения канала ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 10/31/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: af4ef156cccded6afe2db09628446a6ffe1ad53a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92204645"
---
# <a name="verifying-expressroute-connectivity"></a>Проверка подключения ExpressRoute
Изучив сведения в этой статье, вы узнаете, как проверить и устранить неполадки с подключением ExpressRoute. ExpressRoute расширяет локальную сеть в облако Майкрософт по частному подключению, которое обычно упрощается поставщиком услуг подключения. Подключение ExpressRoute обычно включает три отдельные зоны сети, как показано ниже.

-   клиентскую сеть;
-   сеть поставщика;
-   центр обработки данных Майкрософт.

> [!NOTE]
> В модели прямого подключения ExpressRoute (предлагается с пропускной способностью 10/100 Гбит/с) клиенты могут напрямую подключаться к порту маршрутизаторов Microsoft Enterprise (MSEE). Таким образом, в модели прямого подключения есть только зоны клиента и сети Microsoft.
>


Цель этого документа — помочь пользователю определить, где существует проблема с подключением. Таким образом, чтобы найти поддержку от соответствующей команды для решения проблемы. Если для устранения проблемы необходима помощь службы поддержки Майкрософт, отправьте запрос в [службу поддержки Майкрософт][Support].

> [!IMPORTANT]
> Данный документ предназначен для диагностики и устранения простых проблем. Он не заменит услуг службы поддержки Майкрософт. Если вам не удается устранить проблему, используя приведенные здесь рекомендации, то необходимо отправить запрос в [службу поддержки Майкрософт][Support].
>
>

## <a name="overview"></a>Обзор
На следующей схеме показано логическое подключение между клиентской сетью и сетью Майкрософт через ExpressRoute.
[![1]][1]

На предыдущей схеме цифры обозначают ключевые точки сети. Ссылки на эти точки сети можно указывать в этой статье в указанное число раз. В зависимости от модели подключения ExpressRoute — совместное размещение облачного Exchange, подключение типа "точка-точка" или "любой к любому" (IPVPN) — точки сети 3 и 4 могут быть коммутаторами (устройствам уровня 2) или маршрутизаторами (устройства уровня 3). В модели прямого подключения отсутствуют точки сети 3 и 4. Вместо этого CEs (2) напрямую подключаются к MSEE через темное волокно. Ниже приведены представленные на схеме ключевые точки сети.

1.  Клиентское вычислительное устройство (например, сервер или компьютер).
2.  Клиентские пограничные маршрутизаторы (CE). 
3.  Пограничные маршрутизаторы (коммутаторы) поставщика услуг связи, подключенные к клиентским пограничным маршрутизаторам (PE, подключенные к CE). В этом документе называются PE-CE.
4.  Пограничные маршрутизаторы (коммутаторы) поставщика услуг связи, подключенные к MSEE. В этом документе называются PE-MSEE.
5.  Маршрутизаторы MSEE ExpressRoute.
6.  Шлюз виртуальной сети
7.  Вычислительное устройство в виртуальной сети Azure

Если используется совместное размещение облачного Exchange, Ethernet "точка — точка" или модели прямого подключения, CEs (2) установите пиринг BGP с MSEE (5). 

Если используется модель подключения "любой к любому" (IPVPN), PE-MSEEs (4) установите пиринг BGP с MSEE (5). PE-MSEEs распространить маршруты, полученные от корпорации Майкрософт, обратно в сеть клиентов через сеть поставщика услуг IPVPN.

> [!NOTE]
>Для обеспечения высокой доступности Корпорация Майкрософт устанавливает полностью избыточное параллельное подключение между MSEE (5) и PE-MSEEs (4) парами. Кроме того, для связи между клиентом и PE-CEs парой рекомендуется полностью избыточный параллельный сетевой путь. Дополнительные сведения о высокой доступности см. в статье [Разработка для обеспечения высокой доступности с помощью ExpressRoute][HA] .
>
>

Ниже приведены логические шаги в разделе Устранение неполадок канала ExpressRoute:

* [Проверка подготовки и состояния канала](#verify-circuit-provisioning-and-state)
  
* [Проверка настройки пиринга](#validate-peering-configuration)
  
* [Проверка ARP](#validate-arp)
  
* [Проверка BGP и маршрутов MSEE](#validate-bgp-and-routes-on-the-msee)
  
* [Подтверждение потока трафика](#confirm-the-traffic-flow)


## <a name="verify-circuit-provisioning-and-state"></a>Проверка подготовки и состояния канала
Подготовка канала ExpressRoute создает избыточные подключения уровня 2 между CEs/PE-MSEE (2)/(4) и MSEE (5). Дополнительные сведения о создании, изменении, подготовке и проверке канала ExpressRoute см. в статье [Создание и изменение канала ExpressRoute][CreateCircuit].

>[!TIP]
>Ключ службы однозначно идентифицирует канал ExpressRoute. Если вам нужна помощь от корпорации Майкрософт или у партнера ExpressRoute, чтобы устранить неполадки ExpressRoute, предоставьте ключ службы для легкого обнаружения канала.
>
>

### <a name="verification-via-the-azure-portal"></a>Проверка на портале Azure
В портал Azure откройте колонку канал ExpressRoute. В разделе ![3][3] этой колонки перечислены основные компоненты ExpressRoute, как показано на следующем снимке экрана:

![4][4]    

Параметр *Состояние цепи* в разделе основных сведений об ExpressRoute указывает состояние канала на стороне Майкрософт. Параметр *Состояние поставщика* указывает, был ли канал *подготовлен* на стороне поставщика услуг. 

Для работы канала ExpressRoute параметр *Состояние цепи* должен иметь значение *Включено*, а параметр *Состояние поставщика* — значение *Подготовлено*.

> [!NOTE]
> Если после настройки канала ExpressRoute *состояние канала* не включено, обратитесь в [Служба поддержки Майкрософт][Support]. С другой стороны, если состояние *поставщика* в состоянии "в состоянии" не подготовлено, обратитесь к поставщику услуг.
>
>

### <a name="verification-via-powershell"></a>Проверка с помощью PowerShell
Чтобы получить список всех каналов ExpressRoute в группе ресурсов, используйте следующую команду:

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG"
```

>[!TIP]
>Если вы ищете имя группы ресурсов, вы можете получить его, перечисляя все группы ресурсов в подписке с помощью команды *Get-азресаурцеграуп* .
>


Чтобы выбрать конкретный канал ExpressRoute в группе ресурсов, используйте следующую команду:

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
```

Пример ответа:

```output
Name                             : Test-ER-Ckt
ResourceGroupName                : Test-ER-RG
Location                         : westus2
Id                               : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_UnlimitedData",
                                    "Tier": "Standard",
                                    "Family": "UnlimitedData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             : 
ServiceProviderProperties        : {
                                    "ServiceProviderName": "****",
                                    "PeeringLocation": "******",
                                    "BandwidthInMbps": 100
                                    }
ServiceKey                       : **************************************
Peerings                         : []
Authorizations                   : []
```

Чтобы подтвердить работоспособность канала ExpressRoute, обратите особое внимание на следующие поля:

```output
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
```

> [!NOTE]
> Если после настройки канала ExpressRoute *состояние канала* не включено, обратитесь в [Служба поддержки Майкрософт][Support]. С другой стороны, если состояние *поставщика* в состоянии "в состоянии" не подготовлено, обратитесь к поставщику услуг.
>
>

## <a name="validate-peering-configuration"></a>Проверка настройки пиринга
После того как поставщик услуг завершит подготовку канала ExpressRoute, можно создать несколько конфигураций маршрутизации на основе eBGP для канала ExpressRoute между CEs/MSEE-PEs (2)/(4) и MSEE (5). Каждый канал ExpressRoute может иметь: частный пиринг Azure (трафик к частным виртуальным сетям в Azure) и (или) пиринг Майкрософт (трафик к общедоступным конечным точкам PaaS и SaaS). Дополнительные сведения о создании и изменении настроек маршрутизации см. в статье [Создание и изменение маршрутизации для канала ExpressRoute][CreatePeering].

### <a name="verification-via-the-azure-portal"></a>Проверка на портале Azure

> [!NOTE]
> В модели подключения IPVPN поставщики услуг отвечают за настройку пиринга (службы уровня 3). В такой модели после настройки пиринга поставщиком услуг, а также в случае, если пиринг пуст на портале, попробуйте обновить конфигурацию канала с помощью кнопки Обновить на портале. Эта операция выберет текущую конфигурацию маршрутизации из канала. 
>

В портал Azure состояние пиринга канала ExpressRoute можно проверить в колонке канала ExpressRoute. В разделе, посвященном ![трем][3] колонкам, будут перечислены узлы ExpressRoute, как показано на следующем снимке экрана:

![5][5]

В предыдущем примере, как отмечалось в подготовности частного пиринга Azure, в то время как общедоступные пиринга Azure и пиринг Майкрософт не подготавливаются. В успешно подготовленном контексте пиринга также будут указаны первичные и вторичные подсети типа "точка — точка". Подсети/30 используются для IP-адреса интерфейса MSEE и CEs/PE-MSEE. Для пиринга, которые подготавливаются, в списке также указывается, кто последний раз изменил конфигурацию. 

> [!NOTE]
> Если включить пиринг не удается, проверьте, соответствуют ли назначенные первичные и вторичные подсети конфигурации на связанном CE/PE-MSEE. Также убедитесь, что в MSEE используются правильные *VlanId*, *AzureASN* и *PeerASN* , и если эти значения сопоставлены с теми, которые используются в связанной среде CE/PE-MSEE. Если выбрано хэширование MD5, общий ключ должен быть одинаковым для пары MSEE и PE-MSEE/CE. Ранее настроенный общий ключ не будет отображаться по соображениям безопасности. Если необходимо изменить любую из этих конфигураций на маршрутизаторе MSEE, см. статью [Создание и изменение маршрутизации для канала ExpressRoute][CreatePeering].  
>

> [!NOTE]
> В подсети/30, назначенной для интерфейса, Майкрософт выберет второй доступный IP-адрес подсети для интерфейса MSEE. Поэтому убедитесь, что первый доступный IP-адрес подсети назначен на одноранговом узле CE/PE-MSEE.
>


### <a name="verification-via-powershell"></a>Проверка с помощью PowerShell
Чтобы получить дополнительные сведения о конфигурации частного пиринга Azure, используйте следующие команды:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

Пример ответа в случае успешной настройки частного пиринга:

```output
Name                       : AzurePrivatePeering
Id                         : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt/peerings/AzurePrivatePeering
Etag                       : W/"################################"
PeeringType                : AzurePrivatePeering
AzureASN                   : 12076
PeerASN                    : 123##
PrimaryPeerAddressPrefix   : 172.16.0.0/30
SecondaryPeerAddressPrefix : 172.16.0.4/30
PrimaryAzurePort           : 
SecondaryAzurePort         : 
SharedKey                  : 
VlanId                     : 200
MicrosoftPeeringConfig     : null
ProvisioningState          : Succeeded
```

 Если контекст пиринга включен, в списке отобразятся первичные и вторичные префиксы адресов. Подсети/30 используются для IP-адреса интерфейса MSEE и CEs/PE-MSEE.

Чтобы получить дополнительные сведения о настройке общедоступного пиринга Azure, используйте следующие команды:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
```

Чтобы получить дополнительные сведения о настройке пиринга Майкрософт, используйте следующие команды:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

Если пиринг не настроен, отобразится сообщение об ошибке. Пример ответа, если указанный пиринг (в этом примере общедоступный пиринг Azure) не настроен в канале.

```azurepowershell
Get-AzExpressRouteCircuitPeeringConfig : Sequence contains no matching element
At line:1 char:1
    + Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzExpr...itPeeringConfig], InvalidOperationException
        + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.GetAzureExpressRouteCircuitPeeringConfigCommand
```

> [!NOTE]
> Если включить пиринг не удается, проверьте, соответствуют ли назначенные первичные и вторичные подсети конфигурации на связанном CE/PE-MSEE. Также убедитесь, что в MSEE используются правильные *VlanId*, *AzureASN* и *PeerASN* , и если эти значения сопоставлены с теми, которые используются в связанной среде CE/PE-MSEE. Если выбрано хэширование MD5, общий ключ должен быть одинаковым для пары MSEE и PE-MSEE/CE. Ранее настроенный общий ключ не будет отображаться по соображениям безопасности. Если необходимо изменить любую из этих конфигураций на маршрутизаторе MSEE, см. статью [Создание и изменение маршрутизации для канала ExpressRoute][CreatePeering].  
>
>

> [!NOTE]
> В подсети/30, назначенной для интерфейса, Майкрософт выберет второй доступный IP-адрес подсети для интерфейса MSEE. Поэтому убедитесь, что первый доступный IP-адрес подсети назначен на одноранговом узле CE/PE-MSEE.
>

## <a name="validate-arp"></a>Проверка ARP

Таблица ARP обеспечивает сопоставление IP-адреса и адреса MAC для конкретного пиринга. Таблица ARP для пиринга канала ExpressRoute содержит следующие сведения о каждом интерфейсе (первичном и вторичном).
* Сопоставление локального IP-адреса интерфейса маршрутизатора с MAC-адресом.
* Сопоставление IP-адреса интерфейса маршрутизатора ExpressRoute с MAC-адресом.
* Возраст таблиц сопоставления ARP поможет проверить конфигурацию уровня 2 и устранить проблемы с подключением базовых уровней 2.


Сведения о том, как просмотреть таблицу ARP пиринга ExpressRoute и как использовать информацию для устранения неполадок подключения уровня 2, см. [в статье получение ТАБЛИЦ ARP в документе модели развертывания Диспетчер ресурсов][ARP] .


## <a name="validate-bgp-and-routes-on-the-msee"></a>Проверка BGP и маршрутов MSEE

Чтобы получить таблицу маршрутизации из MSEE на *основном* пути для контекста *частной* маршрутизации, используйте следующую команду:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable -DevicePath Primary -ExpressRouteCircuitName ******* -PeeringType AzurePrivatePeering -ResourceGroupName ****
```

Пример ответа:

```output
Network : 10.1.0.0/16
NextHop : 10.17.17.141
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.1.0.0/16
NextHop : 10.17.17.140*
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.2.20.0/25
NextHop : 172.16.0.1
LocPrf  : 
Weight  : 0
Path    : 123##
```

> [!NOTE]
> Если состояние пиринга eBGP между MSEE и CE/PE-MSEE находится в активном состоянии или в режиме простоя, проверьте, соответствуют ли назначенные первичные и вторичные подсети конфигурации в связанной среде CE/PE-MSEE. Также убедитесь, что в MSEE используются правильные *VlanId*, *AzureAsn* и *PeerAsn* , и если эти значения сопоставлены с теми, которые используются в связанном PE-MSEE/CE. Если выбрано хэширование MD5, общий ключ должен быть одинаковым для пары MSEE и CE/PE-MSEE. Если необходимо изменить любую из этих конфигураций на маршрутизаторе MSEE, см. статью [Создание и изменение маршрутизации для канала ExpressRoute][CreatePeering].
>


> [!NOTE]
> Если некоторые целевые объекты недоступны через пиринг, проверьте таблицу маршрутов MSEE для соответствующего контекста пиринга. Если в таблице маршрутизации присутствует соответствующий префикс (может быть указан IP-адрес), проверьте наличие брандмауэров/NSG/ACL в пути, который блокирует трафик.
>


В следующем примере показан ответ команды для несуществующего пиринга:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="confirm-the-traffic-flow"></a>Подтверждение потока трафика
Для получения сводной статистики трафика (переданных и полученных байтов) по первичному и вторичному пути для контекста пиринга используйте следующую команду:

```azurepowershell
Get-AzExpressRouteCircuitStats -ResourceGroupName $RG -ExpressRouteCircuitName $CircuitName -PeeringType 'AzurePrivatePeering'
```

Пример результата выполнения команды:

```output
PrimaryBytesIn PrimaryBytesOut SecondaryBytesIn SecondaryBytesOut
-------------- --------------- ---------------- -----------------
     240780020       239863857        240565035         239628474
```

Пример результата выполнения команды при отсутствующем пиринге:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения доступны в следующих источниках:

- [Поддержка Microsoft][Support]
- [Создание и изменение канала ExpressRoute][CreateCircuit]
- [Создание и изменение маршрутизации для канала ExpressRoute][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-expressroute-overview/expressroute-logical-diagram.png "Логическое подключение ExpressRoute"
[2]: ./media/expressroute-troubleshooting-expressroute-overview/portal-all-resources.png "Значок "Все ресурсы""
[3]: ./media/expressroute-troubleshooting-expressroute-overview/portal-overview.png "Значок "Обзор""
[4]: ./media/expressroute-troubleshooting-expressroute-overview/portal-circuit-status.png "Снимок экран раздела с основными сведениями об ExpressRoute"
[5]: ./media/expressroute-troubleshooting-expressroute-overview/portal-private-peering.png "Снимок экран раздела с основными сведениями об ExpressRoute"

<!--Link References-->
[Support]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[CreateCircuit]: ./expressroute-howto-circuit-portal-resource-manager.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[ARP]: ./expressroute-troubleshooting-arp-resource-manager.md
[HA]: ./designing-for-high-availability-with-expressroute.md
[DR-Pvt]: ./designing-for-disaster-recovery-with-expressroute-privatepeering.md