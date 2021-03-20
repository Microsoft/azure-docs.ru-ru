---
title: 'VPN-шлюз Azure: Подключение шлюзов к нескольким локальным VPN-устройствам на основе политик'
description: Настройка VPN-шлюза Azure на основе маршрутов для нескольких VPN-устройств на основе политики с помощью Azure Resource Manager и PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 2a85204fef026940394a19934bef1c631a8e2d21
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89418888"
---
# <a name="connect-azure-vpn-gateways-to-multiple-on-premises-policy-based-vpn-devices-using-powershell"></a>Подключение VPN-шлюзов Azure к нескольким локальным VPN-устройствам на основе политики с помощью PowerShell

Эта статья поможет вам настроить VPN-шлюз Azure на основе маршрутов для подключения к нескольким локальным VPN-устройствам на основе политики, которые используют настраиваемые политики IPsec/IKE для VPN-подключений типа "сеть — сеть".

## <a name="about-policy-based-and-route-based-vpn-gateways"></a><a name="about"></a>VPN-шлюзы на основе политики и маршрутов

На основе политик *и* VPN-устройств, основанных на маршрутах, различаются настройки селекторов трафика IPSec для соединения.

* **На основе политик** VPN-устройства используют сочетание префиксов из обеих сетей для определения способа шифрования и расшифровки трафика через туннели IPsec. Обычно это включено для устройств брандмауэра, выполняющих фильтрацию пакетов. Шифрование и расшифровка туннеля IPsec добавляются в фильтрацию пакетов и подсистему обработки.
* **На основе маршрутов** VPN-устройства используют селекторы трафика "любой к любому" (с подстановочными знаками) и позволяют маршрутным и перенаправляемым таблицам направить трафик в разные туннели IPsec. Обычно это включено для платформ маршрутизатора, где каждый IPsec-туннель смоделирован как сетевой интерфейс или VTI (виртуальный интерфейс туннеля).

Следующие схемы выделяют две модели:

### <a name="policy-based-vpn-example"></a>Пример VPN-устройства на основе политики
![на основе политик](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedmultisite.png)

### <a name="route-based-vpn-example"></a>Пример VPN-устройства на основе маршрута
![на основе маршрута](./media/vpn-gateway-connect-multiple-policybased-rm-ps/routebasedmultisite.png)

### <a name="azure-support-for-policy-based-vpn"></a>Поддержка Azure VPN-устройств на основе политики
В настоящее время Azure поддерживает оба режима VPN-шлюзов: VPN-шлюзы на основе маршрута и VPN-шлюзы на основе политики. Они разработаны для различных внутренних платформ. Дополнительные сведения см. в следующих спецификациях:

| Category | VPN-шлюзы на основе политики | VPN-шлюзы на основе маршрута | VPN-шлюзы на основе маршрута |
| -------- | ----------------------- | ---------------------- | ---------------------- |---                                                 |
| **SKU шлюза Azure**    | Базовый                       | Базовый                            | VpnGw1, VpnGw2, VpnGw3, VpnGw4, VpnGw5  |
| **Версия IKE**          | IKEv1                       | IKEv2                            | IKEv1 и IKEv2                         |
| **Максимальное число подключений S2S** | **1**                       | 10                               | 30                     |
|                          |                             |                                  |                                                    |

С настраиваемой политикой IPsec/IKE теперь можно настроить VPN-шлюзы Azure на основе маршрута, чтобы использовать селекторы трафика на основе префикса с параметром "**PolicyBasedTrafficSelectors**" для подключения к локальным VPN-устройствам на основе политики. Эта возможность позволит подключиться из виртуальной сети Azure и VPN-шлюза к нескольким локальным устройствам VPN и брандмауэра на основе политики, не ограничиваясь единым подключением из текущих VPN-шлюзов Azure на основе политики.

> [!IMPORTANT]
> 1. Чтобы включить возможность такого подключения, ваши локальные устройства VPN на основе политики должны поддерживать **IKEv2** для подключения к VPN-шлюзам Azure на основе маршрута. Проверьте характеристики VPN-устройства.
> 2. Локальные сети, выполняющие подключение через VPN-устройства на основе политики с помощью этого механизма, могут подключаться только к виртуальной сети Azure. **Подключение к другим локальным или виртуальным сетям с помощью одного VPN-шлюза Azure невозможно**.
> 3. Параметр конфигурации является частью настраиваемой политики подключения IPsec/IKE. При включении параметра селектора трафика на основе политики вы должны указать полную политику (алгоритмы шифрования и целостности IPsec/IKE, уровни стойкости ключей и время существования SA).

На следующей схеме показано, почему транзитная маршрутизация через VPN-шлюз Azure не работает с включенным параметром на основе политики.

![передача на основе политики](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedtransit.png)

Как показано на схеме, у VPN-шлюза Azure есть селекторы трафика из виртуальной сети для каждого локального префикса в сети, а не префиксов кросс-подключения. Например, локальные сайты 2, 3 и 4 могут все взаимодействовать с сетью VNet1, но не могут подключиться через VPN-шлюз Azure друг к другу. На схеме показаны селекторы трафика с кросс-подключением, недоступные для VPN-шлюза Azure в этой конфигурации.

## <a name="workflow"></a><a name="workflow"></a>Рабочий процесс

Инструкции в этой статье основаны на том же примере, который описан в статье о [настройке политики IPsec/IKE для подключений типа "сеть — сеть" или "виртуальная сеть — виртуальная сеть"](vpn-gateway-ipsecikepolicy-rm-powershell.md) для установки подключения VPN типа "сеть — сеть". Это показано на схеме ниже:

![s2s-policy](./media/vpn-gateway-connect-multiple-policybased-rm-ps/s2spolicypb.png)

Этапы подключения:
1. Создайте виртуальную сеть, VPN-шлюз и шлюз локальной сети для подключения между организациями.
2. Создайте политику IPsec/IKE.
3. Примените политику при создании подключения S2S или VNet-to-VNet, а также **включите селекторы трафика на основе политики** для подключения.
4. Если подключение уже создано, можно применить или обновить политику к существующему соединению.

## <a name="before-you-begin"></a>Перед началом

* Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

* [!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="enable-policy-based-traffic-selectors"></a><a name="enablepolicybased"></a>Включить селекторы трафика на основе политики

В этом разделе показано, как включить селекторы трафика на основе политики для соединения. Убедитесь, что вы завершили [часть 3 статьи Настройка политики IPSec/IKE](vpn-gateway-ipsecikepolicy-rm-powershell.md). В действиях, описанных в этой статье, используются те же параметры.

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>Шаг 1. Создание виртуальной сети, VPN-шлюза и шлюза локальной сети

#### <a name="connect-to-your-subscription-and-declare-your-variables"></a>Подключитесь к подписке и объявите переменные.

1. Если вы используете PowerShell локально на компьютере, выполните вход с помощью командлета *Connect-азаккаунт* . Вместо этого используйте Azure Cloud Shell в браузере.

2. Объявите переменные. В этом упражнении мы используем следующие переменные:

   ```azurepowershell-interactive
   $Sub1          = "<YourSubscriptionName>"
   $RG1           = "TestPolicyRG1"
   $Location1     = "East US 2"
   $VNetName1     = "TestVNet1"
   $FESubName1    = "FrontEnd"
   $BESubName1    = "Backend"
   $GWSubName1    = "GatewaySubnet"
   $VNetPrefix11  = "10.11.0.0/16"
   $VNetPrefix12  = "10.12.0.0/16"
   $FESubPrefix1  = "10.11.0.0/24"
   $BESubPrefix1  = "10.12.0.0/24"
   $GWSubPrefix1  = "10.12.255.0/27"
   $DNS1          = "8.8.8.8"
   $GWName1       = "VNet1GW"
   $GW1IPName1    = "VNet1GWIP1"
   $GW1IPconf1    = "gw1ipconf1"
   $Connection16  = "VNet1toSite6"
   $LNGName6      = "Site6"
   $LNGPrefix61   = "10.61.0.0/16"
   $LNGPrefix62   = "10.62.0.0/16"
   $LNGIP6        = "131.107.72.22"
   ```

#### <a name="create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>Создание виртуальной сети, VPN-шлюза и шлюза локальной сети

1. Создайте группу ресурсов.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG1 -Location $Location1
   ```
2. Используйте приведенный ниже пример, чтобы создать виртуальную сеть TestVNet1 с тремя подсетями и VPN-шлюзом. При замене значений важно, чтобы вы назвали подсеть шлюза именем GatewaySubnet. Если вы используете другое имя, создание шлюза завершится сбоем.

    ```azurepowershell-interactive
    $fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
    $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
    $gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
    
    New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
    
    $gw1pip1    = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
    $vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
    $subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
    $gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
    
    New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
    
    New-AzLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
    ```

### <a name="step-2---create-an-s2s-vpn-connection-with-an-ipsecike-policy"></a>Шаг 2. Создание VPN-подключения S2S с помощью политики IPsec/IKE

1. Создайте политику IPsec/IKE.

   > [!IMPORTANT]
   > Вам необходимо создать политику IPsec/IKE для включения параметра "UsePolicyBasedTrafficSelectors" для подключения.

   В следующем примере создается политика IPsec/IKE с этими алгоритмами и параметрами:
    * IKEv2: AES256, SHA384, DHGroup24
    * IPsec: AES256, SHA256, PFS (Нет), срок действия SA (14 400 секунд и 102 400 000 КБ).

   ```azurepowershell-interactive
   $ipsecpolicy6 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
   ```
1. Создайте VPN-подключение S2S с селекторами трафика на основе политики и политикой IPsec/IKE и примените политику IPsec/IKE, созданную на предыдущем шаге. Обратите внимание на дополнительный параметр "-UsePolicyBasedTrafficSelectors $True", который позволяет использовать селекторы трафика на основе политик в соединении.

   ```azurepowershell-interactive
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
   $lng6 = Get-AzLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

   New-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -UsePolicyBasedTrafficSelectors $True -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
   ```
1. После выполнения этих действий подключение VPN типа "сеть — сеть" будет использовать для подключения определенную выше политику IPsec/IKE и включит селекторы трафика на основе политики. Вы можете повторить те же действия, чтобы добавить подключения для дополнительных локальных VPN-устройств на основе политики из одного VPN-шлюза Azure.

## <a name="to-update-policy-based-traffic-selectors"></a><a name="update"></a>Обновление селекторов трафика на основе политик
В этом разделе показано, как обновить параметр селекторов трафика на основе политики для существующего VPN-подключения S2S.

1. Получение ресурса подключения

   ```azurepowershell-interactive
   $RG1          = "TestPolicyRG1"
   $Connection16 = "VNet1toSite6"
   $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
   ```
1. Просмотрите параметр селекторов трафика на основе политики.
В следующей строке показано, используются ли для подключения селекторы трафика на основе политики:

   ```azurepowershell-interactive
   $connection6.UsePolicyBasedTrafficSelectors
   ```

   Если строка возвращает значение **True**, селекторы трафика на основе политики настроены для подключения. В противном случае возвращается значение **False**.
1. После получения ресурса подключения можно включить или отключить селекторы трафика на основе политики для соединения.

   - Включение

      В следующем примере включается параметр селекторов трафика на основе политики и не изменяется политика IPsec/IKE:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $True
      ```

   - Для отключения

      В следующем примере отключается параметр селекторов трафика на основе политики и не изменяется политика IPsec/IKE:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $False
      ```

## <a name="next-steps"></a>Дальнейшие действия
Установив подключение, можно добавить виртуальные машины в виртуальные сети. Инструкции см. в статье о [создании виртуальной машины](../virtual-machines/windows/quick-create-portal.md).

Дополнительные сведения о настраиваемых политиках IPsec/IKE см. в статье [Настройка политики IPsec/IKE для VPN-подключений типа "сеть — сеть" или "виртуальная сеть — виртуальная сеть"](vpn-gateway-ipsecikepolicy-rm-powershell.md).
