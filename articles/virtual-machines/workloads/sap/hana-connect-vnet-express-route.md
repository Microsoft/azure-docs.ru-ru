---
title: Подключение, настроенное из виртуальной сети для SAP HANA в Azure (крупные экземпляры) | Документация Майкрософт
description: Подключение, настроенное из виртуальной сети для использования SAP HANA в Azure (крупные экземпляры).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/25/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 218862b2df959fc0a39baa3c2934fc05f252af32
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101672977"
---
# <a name="connect-a-virtual-network-to-hana-large-instances"></a>Подключение виртуальной сети к крупным экземплярам HANA

После создания виртуальной сети Azure, вы можете подключить сеть к SAP HANA в крупных экземплярах Azure. Создайте шлюз Azure ExpressRoute в виртуальной сети. Этот шлюз позволяет связать виртуальную сеть с каналом ExpressRoute, который подключается к клиенту клиента в отметке крупных экземпляров HANA.

> [!NOTE] 
> Этот шаг может занять до 30 минут, так как при этом создается шлюз в обозначенной подписке Azure, который затем подключается к указанной виртуальной сети Azure.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

Если шлюз уже существует, проверьте, является ли он шлюзом ExpressRoute. Если это не шлюз ExpressRoute, удалите шлюз и создайте его повторно в качестве шлюза ExpressRoute. Если шлюз ExpressRoute уже установлен, обратитесь к разделу данной статьи "Связывание виртуальных сетей". 

- Используйте [портал Azure](https://portal.azure.com/) или PowerShell, чтобы создать VPN-шлюз ExpressRoute, подключенный к виртуальной сети.
  - Если вы используете портал Azure, добавьте новый **шлюз виртуальной сети**, а затем выберите **ExpressRoute** в качестве типа шлюза.
  - Если вы используете PowerShell, сначала загрузите и используйте последнюю версию [Пакета SDK для Azure PowerShell](https://azure.microsoft.com/downloads/). 
 
Следующие команды создают шлюз ExpressRoute. Тексты, начинающиеся с, _$_ представляют собой определяемые пользователем переменные, которые должны быть обновлены с учетом конкретной информации.

```powershell
# These Values should already exist, update to match your environment
$myAzureRegion = "eastus"
$myGroupName = "SAP-East-Coast"
$myVNetName = "VNet01"

# These values are used to create the gateway, update for how you wish the GW components to be named
$myGWName = "VNet01GW"
$myGWConfig = "VNet01GWConfig"
$myGWPIPName = "VNet01GWPIP"
$myGWSku = "HighPerformance" # Supported values for HANA large instances are: HighPerformance or UltraPerformance

# These Commands create the Public IP and ExpressRoute Gateway
$vnet = Get-AzVirtualNetwork -Name $myVNetName -ResourceGroupName $myGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
New-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName `
-Location $myAzureRegion -AllocationMethod Dynamic
$gwpip = Get-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name $myGWConfig -SubnetId $subnet.Id `
-PublicIpAddressId $gwpip.Id

New-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName -Location $myAzureRegion `
-IpConfigurations $gwipconfig -GatewayType ExpressRoute `
-GatewaySku $myGWSku -VpnType PolicyBased -EnableBgp $true
```

В этом примере использовался номер SKU шлюза HighPerformance. Существуют следующие варианты: HighPerformance или UltraPerformance. Это единственные номера SKU шлюза, которые поддерживает SAP HANA в Azure (крупные экземпляры).

> [!IMPORTANT]
> Для крупных экземпляров HANA с номерами SKU класса 2 вы должны использовать SKU "UltraPerformance Gateway".

## <a name="link-virtual-networks"></a>Связывание виртуальных сетей

Теперь, когда для виртуальной сети Azure создан шлюз ExpressRoute, Используйте сведения об авторизации, предоставляемые корпорацией Майкрософт, для подключения шлюза ExpressRoute к каналу SAP HANA (крупные экземпляры) ExpressRoute. Вы можете подключиться с помощью портала Azure или PowerShell. Ниже приведены инструкции PowerShell. 

Выполните следующие команды для каждого шлюза ExpressRoute, используя разные AuthGUID для каждого подключения. Первые две записи, приведенные в сценарии ниже, получены из сведений, предоставленных корпорацией Майкрософт. Кроме того, каждый AuthGUID связан с конкретной виртуальной сетью и шлюзом. Если вы хотите добавить другую виртуальную сеть Azure, необходимо получить еще одну AuthID для канала ExpressRoute, который подключает крупные экземпляры HANA к Azure от Майкрософт. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

> [!NOTE]
> Последний параметр в команде New-Азвиртуалнетворкгатевайконнектион, **експрессраутегатевайбипасс** — это новый параметр, включающий быстрый путь ExpressRoute. Функция, которая сокращает задержку в сети между единицами крупных экземпляров HANA и виртуальными машинами Azure. Функциональные возможности добавлены в 2019 мая. Дополнительные сведения см. в статье [SAP HANA (крупные экземпляры) сетевая архитектура](./hana-network-architecture.md). Перед выполнением команд убедитесь, что вы используете последнюю версию командлетов PowerShell.

Возможно, вам потребуется сделать это несколько раз, чтобы подключить шлюз к нескольким каналам ExpressRoute, связанным с вашей подпиской. Например, скорее всего, вам потребуется подключить тот же шлюз виртуальной сети к каналу ExpressRoute, с помощью которого виртуальная сеть подключается к локальной сети.

## <a name="applying-expressroute-fast-path-to-existing-hana-large-instance-expressroute-circuits"></a>Применение быстрого пути ExpressRoute к существующим каналам ExpressRoute крупного экземпляра HANA
В документации мы объяснили, как подключить новый канал ExpressRoute, созданный с помощью развертывания крупных экземпляров HANA, в шлюз Azure ExpressRoute одной из виртуальных сетей Azure. Но у многих клиентов уже есть настроенные каналы ExpressRoute, и виртуальные сети уже подключены к крупным экземплярам HANA. Так как новый быстрый путь ExpressRoute сокращает задержку в сети, рекомендуется применить изменения для использования этой функции. Команды для подключения нового канала Експрисрауте и изменения существующего канала ExpressRoute одинаковы. В результате необходимо выполнить эту последовательность команд PowerShell, чтобы изменить существующий канал для использования. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

Важно добавить последний параметр, как показано выше, чтобы включить функцию быстрого пути ExpressRoute.


## <a name="expressroute-global-reach"></a>Global Reach ExpressRoute
Если вы хотите включить Global Reach для одного или обоих из двух сценариев:

 - Репликация системы HANA без дополнительных прокси-серверов или брандмауэров
 - Копирование резервных копий между единицами крупных экземпляров HANA в двух разных регионах для выполнения системных копий или обновлений системы

необходимо учитывать следующее:

- Необходимо указать диапазон адресного пространства для адресного пространства/29. Этот диапазон адресов может не перекрывать любые другие диапазоны адресного пространства, которые использовались для подключения крупных экземпляров HANA к Azure и могут не перекрываться с любым диапазоном IP-адресов, который вы использовали в Azure или локальной среде.
- Существует ограничение на ASN (номер автономной системы), которое можно использовать для объявления локальных маршрутов для крупных экземпляров HANA. В локальной среде не должны объявляться маршруты с частными ASN в диапазоне 65000 – 65020 или 65515. 
- В случае подключения локального прямого доступа к крупным экземплярам HANA необходимо рассчитать плату за канал, который подключает вас к Azure. Для цен ознакомьтесь с ценами на [Global REACH надстройку](https://azure.microsoft.com/pricing/details/expressroute/).

Чтобы получить один или оба сценария, применяемые к развертыванию, откройте сообщение поддержки в Azure, как описано в разделе [Открытие запроса на поддержку для крупных экземпляров Hana](./hana-li-portal.md#open-a-support-request-for-hana-large-instances) .

Необходимые данные и ключевые слова, которые необходимо использовать для маршрутизации и выполнения запроса в Майкрософт, выглядят следующим образом:

- Служба: SAP HANA крупные экземпляры
- Тип проблемы: Конфигурация и настройка
- Подтип проблемы: Моя проблема не указана выше
- Тема "изменить мою сеть — добавить Global Reach"
- Сведения: "Добавление Global Reach к крупному экземпляру HANA в клиенте крупных экземпляров HANA или" Добавление Global Reach в локальную среду в клиенте крупных экземпляров HANA.
- Дополнительные сведения о крупном экземпляре HANA для клиента крупного экземпляра HANA: необходимо определить **два региона Azure** , где находятся два клиента для подключения, **и** необходимо отправить **диапазон IP-адресов/29** .
- Дополнительные сведения о клиенте крупного экземпляра HANA в локальной среде: необходимо определить **регион Azure** , в котором развернут клиент крупных экземпляров Hana, к которому вы хотите подключиться напрямую. Кроме того, необходимо указать идентификатор **GUID проверки подлинности** и **ИД однорангового узла канала** , полученные при установлении канала ExpressRoute между локальной средой и Azure. Кроме того, необходимо присвоить имя **ASN**. Последний доставке — это **диапазон IP-адресов** для ExpressRoute Global REACH.

> [!NOTE]
> Если вы хотите, чтобы оба варианта обрабатывались, необходимо предоставить два разных или 29 диапазонов IP-адресов, которые не перекрываются с любым другим диапазоном IP-адресов, использованным до сих пор. 




## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные требования к сети для крупных экземпляров HANA](hana-additional-network-requirements.md)
