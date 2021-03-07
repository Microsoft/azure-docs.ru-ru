---
title: 'Настройка параллельных подключений ExpressRoute и S2S VPN: Azure PowerShell'
description: Узнайте, как настроить параллельные подключения ExpressRoute и VPN-подключений типа "сеть — сеть" для модели развертывания с помощью Resource Manager, используя PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/06/2021
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: df88bd9a1d4901b348fbec47ea9e2946542a08e3
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102440094"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-using-powershell"></a>Настройка параллельных подключений "сеть — сеть" и ExpressRoute с помощью PowerShell
> [!div class="op_single_selector"]
> * [PowerShell — Resource Manager](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell — классическая модель](expressroute-howto-coexist-classic.md)
> 
> 

Сведения в этой статье помогут настроить параллельные соединения ExpressRoute и соединения VPN типа "сеть — сеть". Возможность настройки VPN типа "сеть-сеть" и ExpressRoute дает целый ряд преимуществ. Вы можете настроить VPN-подключение "сеть — сеть" как защищенный путь отработки отказа для ExressRoute или использовать эту сеть VPN для подключения к сайтам, не подключенным через ExpressRoute. В этой статье мы рассмотрим порядок действия в каждом из этих вариантов. Эта статья посвящена модели развертывания Resource Manager.

Настройка параллельных VPN-подключений типа "сеть — сеть" и ExpressRoute дает ряд преимуществ.

* VPN типа "сеть — сеть" можно настроить как безопасный путь отработки отказа для ExpressRoute. 
* Кроме того, VPN типа "сеть — сеть" можно использовать для подключения к сайтам, которые не подключены через ExpressRoute. 

В этой статье описан порядок действий для каждого из этих вариантов. Эта статья посвящена модели развертывания Resource Manager. Кроме того, в ней используется PowerShell. Эти сценарии можно также настроить с помощью портала Azure, несмотря на то, что документация еще недоступна. Вы можете начать настройку c любого шлюза. Как правило, не будет взиматься плата за простои при добавлении нового шлюза или подключении к шлюзу.

>[!NOTE]
>Если вы хотите создать VPN типа "сеть — сеть" через канал ExpressRoute, см. [эту статью](site-to-site-vpn-over-microsoft-peering.md).
>

## <a name="limits-and-limitations"></a>Квоты и ограничения
* **Поддерживается только VPN-шлюз на основе маршрутов.** Необходимо использовать [VPN-шлюз](../vpn-gateway/vpn-gateway-about-vpngateways.md)на основе маршрутов. Также вы можете использовать VPN-шлюз на основе маршрутов с VPN-подключением, настроенным для "селекторы трафика на основе политик", как описано в разделе [Подключение к нескольким VPN-устройствам на основе политик](../vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md).
* **Значение ASN VPN-шлюза Azure должно быть равно 65515.** VPN-шлюз Azure поддерживает протокол маршрутизации BGP. Для совместной работы ExpressRoute и Azure VPN необходимо, чтобы номер автономной системы VPN-шлюза Azure был равен значению по умолчанию 65515. Если ранее вы выбрали ASN, отличный от 65515, и измените параметр на 65515, необходимо сбросить VPN-шлюз, чтобы этот параметр вступил в силу.
* **Подсеть шлюза должна иметь длину/27 или более короткий префикс**(например,/26,/25), или при добавлении шлюза виртуальной сети ExpressRoute появится сообщение об ошибке.
* **Сосуществование в виртуальной сети с двумя стеками не поддерживается.** Если вы используете поддержку протокола IPv6 для ExpressRoute и шлюз ExpressRoute с двумя стеками, сосуществование с VPN-шлюзом будет невозможно.

## <a name="configuration-designs"></a>Схемы конфигурации
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>Настройка VPN типа "сеть-сеть" как пути отработки отказа для ExpressRoute
VPN-подключение типа "сеть-сеть" можно настроить как службу архивации для ExpressRoute. Это подключение применяется только к виртуальным сетям, связанным с путем частного пиринга Azure. Для служб, доступ к которым осуществляется через пиринг Microsoft Azure, решения для отработки отказа на основе VPN не существует. Канал ExpressRoute всегда является основной ссылкой. Данные передаются по пути VPN "сеть — сеть", только если канал ExpressRoute не справляется с этой задачей. Во избежание ассиметричной маршрутизации конфигурация локальной сети должна предпочитать канал ExpressRoute, а не VPN типа "сеть — сеть". Чтобы настроить предпочтение для канала ExpressRoute, задайте для маршрутов, полученных через ExpressRoute, более высокий локальный приоритет. 

>[!NOTE]
> Если пиринг Microsoft ExpressRoute включен, вы можете получить общедоступный IP-адрес VPN-шлюза Azure в подключении ExpressRoute. Чтобы настроить подключение VPN типа "сеть — сеть" в качестве резервной копии, необходимо настроить локальную сеть таким образом, чтобы VPN-подключение направлялось в Интернет.
>

> [!NOTE]
> Хотя канал ExpressRoute лучше использовать для VPN типа "сеть — сеть", когда оба маршрута одинаковы, Azure будет выбирать маршрут для назначения пакета по совпадению самого длинного префикса.
> 
> 

![Схема, на которой показано VPN-подключение типа "сеть — сеть" в качестве резервной копии ExpressRoute.](media/expressroute-howto-coexist-resource-manager/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>Настройка VPN типа "сеть-сеть" для подключения к сайтам, не подключенным через ExpressRoute
Сети можно настроить таким образом, чтобы одни из них подключались непосредственно к Azure по VPN типа "сеть-сеть", а другие — через ExpressRoute. 

![Существуют одновременно](media/expressroute-howto-coexist-resource-manager/scenario2.jpg)

> [!NOTE]
> Виртуальную сеть нельзя настроить как транзитный маршрутизатор.
> 
> 

## <a name="selecting-the-steps-to-use"></a>Выбор действий для использования
Существует два типа процедур. Выбор процедуры настройки зависит от того, существует ли уже виртуальная сеть, к которой необходимо подключиться, или требуется создать ее.

* У меня нет виртуальной сети, и мне нужно ее создать.
  
    Если у вас еще нет виртуальной сети, выполните эти шаги, чтобы создать ее с помощью модели развертывания Resource Manager, а также создать подключение ExpressRoute и VPN типа "сеть — сеть". Чтобы настроить виртуальную сеть, выполните шаги, описанные в разделе [Создание виртуальной сети и параллельных подключений](#new).
* У меня уже есть виртуальная сеть с моделью развертывания Resource Manager.
  
    Возможно, у вас уже имеется виртуальная сеть, а также подключения к VPN типа "сеть-сеть" или к ExpressRoute. В этом случае, если размер маски подсети шлюза равен или меньше /28 (например, /28, /29 и т. д.), существующий шлюз нужно удалить. В разделе [Настройка параллельных подключений для существующей виртуальной сети](#add) представлены пошаговые инструкции по удалению шлюза и созданию подключений ExpressRoute и VPN типа "сеть — сеть".
  
    Подключение между организациями прервется на время удаления и повторного создания шлюза. Но виртуальные машины и службы смогут по-прежнему осуществлять связь через подсистему балансировки нагрузки во время настройки шлюза, если они настроены соответствующим образом.

## <a name="before-you-begin"></a>Подготовка к работе

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [working with cloud shell](../../includes/expressroute-cloudshell-powershell-about.md)]


## <a name="to-create-a-new-virtual-network-and-coexisting-connections"></a><a name="new"></a>Создание новой виртуальной сети и параллельных подключений
В этой процедуре подробно рассматривается создание виртуальной сети, а также параллельных подключений ExpressRoute и "сеть — сеть". Командлеты, которые будут использоваться для этой конфигурации, могут немного отличаться от уже знакомых вам. Обязательно используйте командлеты, указанные в инструкциях.

1. Выполните вход и выберите подписку.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]
2. Задание переменных.

   ```azurepowershell-interactive
   $location = "Central US"
   $resgrp = New-AzResourceGroup -Name "ErVpnCoex" -Location $location
   $VNetASN = 65515
   ```
3. Создайте виртуальную сеть и подсеть шлюза. См. дополнительные сведения в статье о [создании виртуальной сети](../virtual-network/manage-virtual-network.md#create-a-virtual-network). См. дополнительные сведения в статье о [создании подсети](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet).
   
   > [!IMPORTANT]
   > Подсеть шлюза должна иметь префикс /27 или более короткий префикс (например, /26 или /25).
   > 
   > 
   
    Создайте виртуальную сеть.

   ```azurepowershell-interactive
   $vnet = New-AzVirtualNetwork -Name "CoexVnet" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AddressPrefix "10.200.0.0/16"
   ```
   
    Добавьте подсети.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name "App" -VirtualNetwork $vnet -AddressPrefix "10.200.1.0/24"
   Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
   ```
   
    Сохраните конфигурацию виртуальной сети.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
4. <a name="vpngw"></a>Далее создайте VPN-шлюз типа "сеть — сеть". Дополнительные сведения о настройке VPN-шлюза см. в статье [Создание виртуальной сети с подключением типа "сеть — сеть" с помощью PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md). GatewaySku поддерживается только в VPN-шлюзах *VpnGw1*, *VpnGw2*, *VpnGw3*, *Standard* и *HighPerformance*. Одновременное использование конфигураций VPN-шлюзов и ExpressRoute не поддерживается в SKU "Базовый". Параметр VpnType должен иметь значение *RouteBased*.

   ```azurepowershell-interactive
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   $gwIP = New-AzPublicIpAddress -Name "VPNGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
   $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "VPNGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
   New-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1"
   ```
   
    Шлюз VPN Azure поддерживает протокол маршрутизации BGP. Для этой виртуальной сети можно указать ASN (номер AS), добавив параметр -Asn в команду ниже. Без этого параметра номер AS получит значение по умолчанию — 65515.

   ```azurepowershell-interactive
   $azureVpn = New-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1" -Asn $VNetASN
   ```
   
    IP-адрес пиринга BGP и номер AS, используемый Azure для VPN-шлюза, можно найти в атрибутах $azureVpn.BgpSettings.BgpPeeringAddress и $azureVpn.BgpSettings.Asn. Дополнительные сведения см. в статье [Настройка BGP на VPN-шлюзах Azure с помощью Azure Resource Manager и PowerShell](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md).
5. Создайте сущность VPN-шлюза локального сайта. Эта команда не настраивает локальный VPN-шлюз. Она только позволяет указать параметры локального шлюза, такие как общедоступный IP-адрес и локальное адресное пространство, чтобы VPN-шлюз Azure мог подключиться к нему.
   
    Если локальное VPN-устройство поддерживает только статическую маршрутизацию, статические маршруты можно настроить следующим образом:

   ```azurepowershell-interactive
   $MyLocalNetworkAddress = @("10.100.0.0/16","10.101.0.0/16","10.102.0.0/16")
   $localVpn = New-AzLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress *<Public IP>* -AddressPrefix $MyLocalNetworkAddress
   ```
   
    Если локальное VPN-устройство поддерживает протокол BGP, чтобы включить динамическую маршрутизацию, необходимо знать используемый этим устройством IP-адрес пиринга BGP и номер AS.

   ```azurepowershell-interactive
   $localVPNPublicIP = "<Public IP>"
   $localBGPPeeringIP = "<Private IP for the BGP session>"
   $localBGPASN = "<ASN>"
   $localAddressPrefix = $localBGPPeeringIP + "/32"
   $localVpn = New-AzLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress $localVPNPublicIP -AddressPrefix $localAddressPrefix -BgpPeeringAddress $localBGPPeeringIP -Asn $localBGPASN
   ```
6. Настройте локальное VPN-устройство для подключения к новому VPN-шлюзу Azure. Дополнительные сведения о настройке VPN-устройства см. в статье [О VPN-устройствах для подключений VPN-шлюзов типа "сеть — сеть"](../vpn-gateway/vpn-gateway-about-vpn-devices.md).

7. Свяжите VPN-шлюз к сети типа "сеть-сеть" в Azure с локальным шлюзом.

   ```azurepowershell-interactive
   $azureVpn = Get-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
   New-AzVirtualNetworkGatewayConnection -Name "VPNConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $azureVpn -LocalNetworkGateway2 $localVpn -ConnectionType IPsec -SharedKey <yourkey>
   ```
 

8. При подключении к существующему каналу ExpressRoute пропустите шаги 8 и 9 и перейдите к шагу 10. Настройте каналы ExpressRoute. Дополнительные сведения о настройке каналов ExpressRoute см. в статье [о создании канала ExpressRoute](expressroute-howto-circuit-arm.md).


9. Настройте частный пиринг Azure через канал ExpressRoute. Дополнительные сведения о настройке открытого пиринга Azure через канал ExpressRoute см. в [этой статье](expressroute-howto-routing-arm.md)

10. <a name="gw"></a>Создайте шлюз ExpressRoute. Дополнительные сведения о настройке шлюза ExpressRoute см. в статье [Настройка шлюза виртуальной сети для ExpressRoute с помощью диспетчера ресурсов и PowerShell](expressroute-howto-add-gateway-resource-manager.md). Параметр GatewaySKU должен иметь значение *Standard*, *HighPerformance* или *UltraPerformance*.

    ```azurepowershell-interactive
    $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
    $gwIP = New-AzPublicIpAddress -Name "ERGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
    $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "ERGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
    $gw = New-AzVirtualNetworkGateway -Name "ERGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "ExpressRoute" -GatewaySku Standard
    ```
11. Свяжите шлюз ExpressRoute с каналом ExpressRoute. После выполнения этого действия подключение локальной сети к Azure через ExpressRoute будет установлено. Дополнительные сведения об операции связывания см. в статье [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md).

    ```azurepowershell-interactive
    $ckt = Get-AzExpressRouteCircuit -Name "YourCircuit" -ResourceGroupName "YourCircuitResourceGroup"
    New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $gw -PeerId $ckt.Id -ConnectionType ExpressRoute
    ```

## <a name="to-configure-coexisting-connections-for-an-already-existing-vnet"></a><a name="add"></a>Настройка параллельных подключений для существующей виртуальной сети
Если у вас есть виртуальная сеть только с одним шлюзом виртуальной сети (например, VPN-шлюз "сеть — сеть") и вы хотите добавить еще один шлюз другого типа (например, шлюз ExpressRoute), проверьте размер подсети шлюза. Если размер подсети шлюза — /27 или больше, можно пропустить описанные ниже шаги и перейти к предыдущему разделу, чтобы добавить шлюз ExpressRoute или VPN-шлюз "сеть — сеть". Если размер подсети шлюза — /28 или /29, необходимо сначала удалить шлюз виртуальной сети и увеличить размер подсети шлюза. В этом разделе показано, как это сделать.

Командлеты, которые будут использоваться для этой конфигурации, могут немного отличаться от уже знакомых вам. Обязательно используйте командлеты, указанные в инструкциях.

1. Удалите для сети типа "сеть — сеть" существующий VPN-шлюз или шлюз ExpressRoute.

   ```azurepowershell-interactive 
   Remove-AzVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
   ```
2. Удалите подсеть шлюза.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup> Remove-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
   ```
3. Добавьте подсеть шлюза с префиксом /27 или больше.
   
   > [!NOTE]
   > Если в виртуальной сети недостаточно свободных IP-адресов для увеличения размера подсети шлюза, необходимо добавить дополнительные пространства IP-адресов.
   > 
   > 

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
   Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
   ```
   
    Сохраните конфигурацию виртуальной сети.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
4. На этом этапе у вас есть виртуальная сеть без шлюзов. Чтобы создать новые шлюзы и настроить подключения, используйте следующие примеры.

   Задайте переменные.

    ```azurepowershell-interactive
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   $gwIP = New-AzPublicIpAddress -Name "ERGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
   $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "ERGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
   ```

   Создайте шлюз.

   ```azurepowershell-interactive
   $gw = New-AzVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup> -Location <yourlocation> -IpConfigurations $gwConfig -GatewayType "ExpressRoute" -GatewaySku Standard
   ```

   Создайте подключение.

   ```azurepowershell-interactive
   $ckt = Get-AzExpressRouteCircuit -Name "YourCircuit" -ResourceGroupName "YourCircuitResourceGroup"
   New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $gw -PeerId $ckt.Id -ConnectionType ExpressRoute
   ```

## <a name="to-add-point-to-site-configuration-to-the-vpn-gateway"></a>Добавление конфигурации "точка — сеть" к VPN-шлюзу

Чтобы добавить конфигурацию "точка — сеть" к VPN-шлюзу при настройке параллельных подключений, выполните следующее. Чтобы отправить корневой сертификат VPN, необходимо либо установить PowerShell локально на компьютер, либо использовать портал Azure.

1. Добавьте пул адресов VPN-клиента.

   ```azurepowershell-interactive
   $azureVpn = Get-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
   Set-AzVirtualNetworkGatewayVpnClientConfig -VirtualNetworkGateway $azureVpn -VpnClientAddressPool "10.251.251.0/24"
   ```
2. Отправьте корневой сертификат VPN в Azure для VPN-шлюза. В этом примере предполагается, что корневой сертификат хранится на локальном компьютере, где выполняются следующие командлеты PowerShell и вы используете PowerShell локально. Также можно передать сертификат с помощью портал Azure.

   ```powershell
   $p2sCertFullName = "RootErVpnCoexP2S.cer" 
   $p2sCertMatchName = "RootErVpnCoexP2S" 
   $p2sCertToUpload=get-childitem Cert:\CurrentUser\My | Where-Object {$_.Subject -match $p2sCertMatchName} 
   if ($p2sCertToUpload.count -eq 1){write-host "cert found"} else {write-host "cert not found" exit} 
   $p2sCertData = [System.Convert]::ToBase64String($p2sCertToUpload.RawData) 
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $p2sCertFullName -VirtualNetworkGatewayname $azureVpn.Name -ResourceGroupName $resgrp.ResourceGroupName -PublicCertData $p2sCertData
   ```

## <a name="to-enable-transit-routing-between-expressroute-and-azure-vpn"></a>Включение транзитной маршрутизации между ExpressRoute и VPN Azure
Если вы хотите разрешить подключение между одной из локальных сетей, подключенных к ExpressRoute, и другой локальной сетью, подключенной к VPN-подключению типа "сеть — сеть", необходимо настроить [сервер маршрутизации Azure](../route-server/expressroute-vpn-support.md).

Дополнительные сведения см. в статье [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью PowerShell](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об ExpressRoute см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).
