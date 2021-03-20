---
title: 'Azure ExpressRoute: Настройка S2S VPN через пиринг Майкрософт'
description: Настройка подключения IPsec/IKE к Azure через пиринговый канал Майкрософт ExpressRoute с использованием VPN-шлюза типа "сеть — сеть".
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 366f27a0e2a22e9aa10dda20e105bf644255bdd4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89393145"
---
# <a name="configure-a-site-to-site-vpn-over-expressroute-microsoft-peering"></a>Настройка VPN типа "сеть — сеть" через пиринговый канал Майкрософт ExpressRoute

Эта статья поможет вам настроить безопасное зашифрованное подключение между локальной сетью и виртуальными сетями Azure через закрытое подключение ExpressRoute. Вы можете использовать пиринг Майкрософт для создания VPN-туннеля типа "сеть — сеть" IPsec/IKE между выбранными локальными сетями и виртуальными сетями Azure. Настройка безопасного туннеля через ExpressRoute обеспечивает конфиденциальный обмен данными, защиту от повтора, подлинность и целостность данных.

>[!NOTE]
>При настройке VPN типа "сеть — сеть" через пиринговый канал Майкрософт вы платите за VPN-шлюз и исходящий трафик VPN. Дополнительные сведения см. в разделе [цены на VPN-шлюз](https://azure.microsoft.com/pricing/details/vpn-gateway).
>
>

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="architecture"></a><a name="architecture"></a>Архитектура


  ![Общие сведения о подключениях](./media/site-to-site-vpn-over-microsoft-peering/IPsecER_Overview.png)


Чтобы обеспечить высокую доступность и избыточность, вы можете настроить несколько туннелей через две пары MSEE-PE каналов ExpressRoute и включить балансировку нагрузки между туннелями.

  ![Варианты высокой доступности](./media/site-to-site-vpn-over-microsoft-peering/HighAvailability.png)

VPN-туннели через пиринговый канал Майкрософт можно завершить либо с помощью VPN-шлюза, либо с помощью соответствующего сетевого виртуального модуля (NVA), доступного в Azure Marketplace. Вы можете менять маршруты статически или динамически через зашифрованные туннели, не отображая обмен маршрутов в базовом пиринговом канале Майкрософт. В примерах в этой статье BGP (отличный от сеанса BGP, используемого для создания пиринга Майкрософт) используется для динамического обмена префиксами по зашифрованным туннелям.

>[!IMPORTANT]
>На стороне локальной среды пиринг Майкрософт обычно завершается в сети периметра, а частный пиринг завершается в основной сети. Две зоны будут разделены с помощью брандмауэров. Если вы настроите пиринг Майкрософт исключительно для обеспечения безопасного туннелирования через ExpressRoute, не забудьте фильтровать только публичные IP-адреса, которые объявляются через пиринг Майкрософт.
>
>

## <a name="workflow"></a><a name="workflow"></a>Рабочий процесс

1. Настройте пиринг Майкрософт для своего канала ExpressRoute.
2. Объявите выбранные региональные префиксы Azure в своей локальной сети через пиринг Майкрософт.
3. Настройте VPN-шлюз и установите IPsec-туннели.
4. Настройте локальное VPN-устройство.
5. Создайте подключение типа "сеть — сеть" IPsec/IKE.
6. (Необязательно.) Настройте брандмауэры или фильтрацию на локальном VPN-устройстве.
7. Протестируйте и проверьте обмен данными IPsec через канал ExpressRoute.

## <a name="1-configure-microsoft-peering"></a><a name="peering"></a>1. Настройка пиринга Майкрософт

Чтобы настроить VPN-подключение типа "сеть — сеть" через ExpressRoute, вы должны использовать пиринг Майкрософт ExpressRoute.

* Чтобы настроить новый канал ExpressRoute, ознакомьтесь с [предварительными требованиями ExpressRoute](expressroute-prerequisites.md), а затем [создайте и измените канал ExpressRoute](expressroute-howto-circuit-arm.md).

* Если у вас уже есть канал ExpressRoute, но не настроен пиринг Майкрософт, настройте его, как описано в разделе [Пиринг Майкрософт](expressroute-howto-routing-arm.md#msft).

После настройки канала и пиринга Майкрософт вы можете просмотреть его на странице **Обзор** на портале Azure.

![канал](./media/site-to-site-vpn-over-microsoft-peering/ExpressRouteCkt.png)

## <a name="2-configure-route-filters"></a><a name="routefilter"></a>2. Настройка фильтров маршрутов

Фильтр маршрутов позволяет вам идентифицировать службы, которые можно использовать через пиринг Майкрософт по каналу ExpressRoute. По сути, это список разрешений для всех значений сообщества BGP. 

![фильтр маршрутов](./media/site-to-site-vpn-over-microsoft-peering/route-filter.png)

В этом примере развертывание выполнено только в регионе *Azure Западная часть США 2*. Добавляется правило фильтра маршрута, чтобы разрешить объявление только региональных префиксов Azure Западная часть США 2, которое имеет значение сообщества BGP *12076:51026*. Укажите региональные префиксы, которые хотите разрешить, выбрав **правило управления**.

В пределах фильтра маршрутов вам также необходимо выбрать каналы ExpressRoute, для которых применяется фильтр. Вы можете выбрать каналы ExpressRoute, щелкнув **Добавить цепь**. На предыдущем рисунке фильтр маршрутов связан с примером канала ExpressRoute.

### <a name="21-configure-the-route-filter"></a><a name="configfilter"></a>2.1 Настройка фильтра маршрутов

Создайте фильтр маршрутов. Инструкции см. в статье [Configure route filters for Microsoft peering: Azure portal](how-to-routefilter-portal.md) (Настройка фильтров маршрутов для пиринга Майкрософт с помощью портала Azure).

### <a name="22-verify-bgp-routes"></a><a name="verifybgp"></a>2.2 Проверка маршрутов BGP

Создав пиринг Майкрософт через канал ExpressRoute и связав с ним фильтр маршрутов, вы можете проверить маршруты BGP, полученные от маршрутизаторов MSEE на устройствах PE, которые устанавливают пиринг с этими маршрутизаторами. Команда проверки может измениться в зависимости от операционной системы ваших устройств PE.

#### <a name="cisco-examples"></a>Примеры Cisco

В этом примере используется команда Cisco IOS-XE. В этом примере для изоляции пирингового трафика используется экземпляр виртуальной маршрутизации и перенаправления (VRF).

```
show ip bgp vpnv4 vrf 10 summary
```

В следующих частичных выходных данных показано, что префиксы 68 были получены из соседнего узла \* . 243.229.34 с ASN 12076 (MSEE):

```
...

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
X.243.229.34    4        12076   17671   17650    25228    0    0 1w4d           68
```

Чтобы просмотреть список префиксов, полученных от соседнего узла, используйте следующий пример:

```
sh ip bgp vpnv4 vrf 10 neighbors X.243.229.34 received-routes
```

Чтобы подтвердить получение правильного набора префиксов, вы можете использовать перекрестную проверку. В следующих выходных данных команды Azure PowerShell показаны префиксы, объявляемые с помощью пиринга Майкрософт для каждой службы и региона Azure:

```azurepowershell-interactive
Get-AzBgpServiceCommunity
```

## <a name="3-configure-the-vpn-gateway-and-ipsec-tunnels"></a><a name="vpngateway"></a>3. Настройка VPN-шлюза и туннелей IPsec

В этом разделе между VPN-шлюзом Azure и локальным VPN-устройством создаются VPN-туннели IPsec. В примерах используются VPN-устройства маршрутизатора облачной службы Cisco (CSR1000).

На следующей схеме показаны VPN-туннели IPsec, установленные между локальным VPN-устройством 1 и парой экземпляров VPN-шлюза Azure. Два VPN-туннеля IPsec, установленные между локальным VPN-устройством 2 и парой экземпляров VPN-шлюза Azure, и данные конфигурации не показаны на схеме. Однако наличие дополнительных VPN-туннелей обеспечивает высокую доступность.

  ![VPN-туннели](./media/site-to-site-vpn-over-microsoft-peering/EstablishTunnels.png)

Через пару туннелей IPsec устанавливается сеанс eBGP для обмена маршрутами частной сети. На следующей схеме показан сеанс eBGP, устанавливаемый через пару туннелей IPsec:

  ![сеансы eBGP через пару туннелей](./media/site-to-site-vpn-over-microsoft-peering/TunnelBGP.png)

На следующей схеме показан абстрактный обзор примера сети:

  ![пример сети](./media/site-to-site-vpn-over-microsoft-peering/OverviewRef.png)

### <a name="about-the-azure-resource-manager-template-examples"></a>Сведения о примерах шаблона Azure Resource Manager

В примерах VPN-шлюз и завершение туннелирования IPsec настраиваются с помощью шаблона Azure Resource Manager. Если вы впервые используете шаблоны Resource Manager или еще не ознакомились с основами работы с ними, см. статью [Understand the structure and syntax of Azure Resource Manager templates](../azure-resource-manager/templates/template-syntax.md) (Описание структуры и синтаксиса шаблонов Azure Resource Manager). Шаблон в этом разделе создает среду Azure (виртуальную сеть) "с нуля". Однако если у вас есть виртуальная сеть, вы можете ссылаться на нее в шаблоне. Если вы не знакомы с конфигурациями VPN-шлюза "сеть — сеть" IPsec/IKE, см. статью [Создание виртуальной сети с VPN-подключением типа "сеть — сеть" с помощью PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md).

>[!NOTE]
>Чтобы создать эту конфигурацию, вам не нужно использовать шаблоны Azure Resource Manager. Ее можно создать с помощью портала Azure или PowerShell.
>
>

### <a name="31-declare-the-variables"></a><a name="variables3"></a>3.1 Объявление переменных

В этом примере объявление переменных соответствует примеру сети. При объявлении переменных измените этот раздел в соответствии со своей средой.

* Переменная **localAddressPrefix** представляет собой массив значений локальных IP-адресов для завершения туннелей IPsec.
* **gatewaySku** определяет пропускную способность VPN. Дополнительные сведения о gatewaySku и vpnType см. в разделе [SKU шлюзов](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku). Дополнительные сведения см. на странице [цен на VPN-шлюз](https://azure.microsoft.com/pricing/details/vpn-gateway).
* Параметр **vpnType** должен иметь значение **RouteBased**.

```json
"variables": {
  "virtualNetworkName": "SecureVNet",       // Name of the Azure VNet
  "azureVNetAddressPrefix": "10.2.0.0/24",  // Address space assigned to the VNet
  "subnetName": "Tenant",                   // subnet name in which tenants exists
  "subnetPrefix": "10.2.0.0/25",            // address space of the tenant subnet
  "gatewaySubnetPrefix": "10.2.0.224/27",   // address space of the gateway subnet
  "localGatewayName": "localGW1",           // name of remote gateway (on-premises)
  "localGatewayIpAddress": "X.243.229.110", // public IP address of the on-premises VPN device
  "localAddressPrefix": [
    "172.16.0.1/32",                        // termination of IPsec tunnel-1 on-premises 
    "172.16.0.2/32"                         // termination of IPsec tunnel-2 on-premises 
  ],
  "gatewayPublicIPName1": "vpnGwVIP1",    // Public address name of the first VPN gateway instance
  "gatewayPublicIPName2": "vpnGwVIP2",    // Public address name of the second VPN gateway instance 
  "gatewayName": "vpnGw",                 // Name of the Azure VPN gateway
  "gatewaySku": "VpnGw1",                 // Azure VPN gateway SKU
  "vpnType": "RouteBased",                // type of VPN gateway
  "sharedKey": "string",                  // shared secret needs to match with on-premises configuration
  "asnVpnGateway": 65000,                 // BGP Autonomous System number assigned to the VPN Gateway 
  "asnRemote": 65010,                     // BGP Autonmous Syste number assigned to the on-premises device
  "bgpPeeringAddress": "172.16.0.3",      // IP address of the remote BGP peer on-premises
  "connectionName": "vpn2local1",
  "vnetID": "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
  "gatewaySubnetRef": "[concat(variables('vnetID'),'/subnets/','GatewaySubnet')]",
  "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables('subnetName'))]",
  "api-version": "2017-06-01"
},
```

### <a name="32-create-virtual-network-vnet"></a><a name="vnet"></a>3.2 Создание виртуальной сети

Если вы связываете VPN-туннели с существующей виртуальной сетью, этот шаг можно пропустить.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/virtualNetworks",
  "name": "[variables('virtualNetworkName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[variables('azureVNetAddressPrefix')]"
      ]
    },
    "subnets": [
      {
        "name": "[variables('subnetName')]",
        "properties": {
          "addressPrefix": "[variables('subnetPrefix')]"
        }
      },
      {
        "name": "GatewaySubnet",
        "properties": {
          "addressPrefix": "[variables('gatewaySubnetPrefix')]"
        }
      }
    ]
  },
  "comments": "Create a Virtual Network with Subnet1 and Gatewaysubnet"
},
```

### <a name="33-assign-public-ip-addresses-to-vpn-gateway-instances"></a><a name="ip"></a>3.3 Назначение общедоступных IP-адресов экземплярам VPN-шлюза
 
Назначьте общедоступный IP-адрес для каждого экземпляра VPN-шлюза.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName1')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the first instance of the VPN gateway"
  },
  {
    "apiVersion": "[variables('api-version')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName2')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the second instance of the VPN gateway"
  },
```

### <a name="34-specify-the-on-premises-vpn-tunnel-termination-local-network-gateway"></a><a name="termination"></a>3.4 Указание завершения локального VPN-туннеля (шлюз локальной сети)

Локальные VPN-устройства называются **шлюзами локальной сети**. В следующем фрагменте кода json указаны сведения об удаленном пиринговом узле BGP:

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/localNetworkGateways",
  "name": "[variables('localGatewayName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "localNetworkAddressSpace": {
      "addressPrefixes": "[variables('localAddressPrefix')]"
    },
    "gatewayIpAddress": "[variables('localGatewayIpAddress')]",
    "bgpSettings": {
      "asn": "[variables('asnRemote')]",
      "bgpPeeringAddress": "[variables('bgpPeeringAddress')]",
      "peerWeight": 0
    }
  },
  "comments": "Local Network Gateway (referred to your on-premises location) with IP address of remote tunnel peering and IP address of remote BGP peer"
},
```

### <a name="35-create-the-vpn-gateway"></a><a name="creategw"></a>3.5 Создание VPN-шлюза

В этом разделе шаблона настраивается VPN-шлюз с необходимыми параметрами для конфигурации "активный — активный". Не забывайте о следующих требованиях:

* Создайте VPN-шлюз с параметром **"RouteBased"** VpnType. Это обязательный параметр, если вы хотите включить маршрутизацию BGP между VPN-шлюзом и локальной сетью VPN.
* Чтобы установить VPN-туннели между двумя экземплярами VPN-шлюза и данным локальным устройством в режиме "активный — активный", параметр **"activeActive"** должен иметь значение **true** в шаблоне Resource Manager. Чтобы получить представление о высокой доступности VPN-шлюзов, см. статью [Настройка высокодоступных подключений: распределенных и между виртуальными сетями](../vpn-gateway/vpn-gateway-highlyavailable.md).
* Чтобы настроить сеансы eBGP между VPN-туннелями, необходимо указать два различных номера ASN с обеих сторон. Мы рекомендуем указывать закрытые номера ASN. Дополнительные сведения см. в статье [Обзор использования BGP с VPN-шлюзами Azure](../vpn-gateway/vpn-gateway-bgp-overview.md).

```json
{
"apiVersion": "[variables('api-version')]",
"type": "Microsoft.Network/virtualNetworkGateways",
"name": "[variables('gatewayName')]",
"location": "[resourceGroup().location]",
"dependsOn": [
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName1'))]",
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName2'))]",
  "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
],
"properties": {
  "ipConfigurations": [
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName1'))]"
        }
      },
      "name": "vnetGtwConfig1"
    },
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName2'))]"
        }
      },
          "name": "vnetGtwConfig2"
        }
      ],
      "sku": {
        "name": "[variables('gatewaySku')]",
        "tier": "[variables('gatewaySku')]"
      },
      "gatewayType": "Vpn",
      "vpnType": "[variables('vpnType')]",
      "enableBgp": true,
      "activeActive": true,
      "bgpSettings": {
        "asn": "[variables('asnVpnGateway')]"
      }
    },
    "comments": "VPN Gateway in active-active configuration with BGP support"
  },
```

### <a name="36-establish-the-ipsec-tunnels"></a><a name="ipsectunnel"></a>3.6 Установка туннелей IPsec

Последнее действие сценария создает туннели IPsec между VPN-шлюзом Azure и локальным VPN-устройством.

```json
{
  "apiVersion": "[variables('api-version')]",
  "name": "[variables('connectionName')]",
  "type": "Microsoft.Network/connections",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Network/virtualNetworkGateways/', variables('gatewayName'))]",
    "[concat('Microsoft.Network/localNetworkGateways/', variables('localGatewayName'))]"
  ],
  "properties": {
    "virtualNetworkGateway1": {
      "id": "[resourceId('Microsoft.Network/virtualNetworkGateways', variables('gatewayName'))]"
    },
    "localNetworkGateway2": {
      "id": "[resourceId('Microsoft.Network/localNetworkGateways', variables('localGatewayName'))]"
    },
    "connectionType": "IPsec",
    "routingWeight": 0,
    "sharedKey": "[variables('sharedKey')]",
    "enableBGP": "true"
  },
  "comments": "Create a Connection type site-to-site (IPsec) between the Azure VPN Gateway and the VPN device on-premises"
  }
```

## <a name="4-configure-the-on-premises-vpn-device"></a><a name="device"></a>4. Настройка локального VPN-устройства

VPN-шлюз Azure совместим со многими VPN-устройствами от разных поставщиков. Сведения о конфигурации и устройства, сертифицированные для работы с VPN-шлюзом, см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз](../vpn-gateway/vpn-gateway-about-vpn-devices.md).

Чтобы настроить локальное VPN-устройство, вам потребуется следующее:

* Общий ключ. Это тот же общий ключ, который указывается при создании VPN-подключения "сеть — сеть". В примерах используются простые общие ключи. Для практического использования рекомендуется создавать более сложные ключи.
* Общедоступный IP-адрес вашего VPN-шлюза. Общедоступный IP-адрес можно просмотреть с помощью портала Azure, PowerShell или CLI. Чтобы найти общедоступный IP-адрес VPN-шлюза с помощью портала Azure, перейдите к разделу "Шлюзы виртуальной сети" и щелкните имя шлюза.

Обычно пиринговые узлы eBGP подключены напрямую (часто через глобальную сеть). Однако при настройке eBGP через VPN-туннели IPsec с помощью пиринга Майкрософт ExpressRoute между узлами eBGP есть несколько доменов маршрутизации. Используйте команду **​bgp-multihop**, чтобы установить соседнюю связь eBGP между двумя пиринговыми узлами, подключенными не напрямую. Целое число после команды ebgp-multihop указывает значение срока жизни в пакетах BGP. Команда **maximum-paths eibgp 2** обеспечивает балансировку нагрузки трафика между двумя путями BGP.

### <a name="cisco-csr1000-example"></a><a name="cisco1"></a>Пример Cisco CSR1000

В следующем примере показана конфигурация Cisco CSR1000 на виртуальной машине Hyper-V в качестве локального VPN-устройства:

```
!
crypto ikev2 proposal az-PROPOSAL
 encryption aes-cbc-256 aes-cbc-128 3des
 integrity sha1
 group 2
!
crypto ikev2 policy az-POLICY
 proposal az-PROPOSAL
!
crypto ikev2 keyring key-peer1
 peer azvpn1
  address 52.175.253.112
  pre-shared-key secret*1234
 !
!
crypto ikev2 keyring key-peer2
 peer azvpn2
  address 52.175.250.191
  pre-shared-key secret*1234
 !
!
!
crypto ikev2 profile az-PROFILE1
 match address local interface GigabitEthernet1
 match identity remote address 52.175.253.112 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer1
!
crypto ikev2 profile az-PROFILE2
 match address local interface GigabitEthernet1
 match identity remote address 52.175.250.191 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer2
!
crypto ikev2 dpd 10 2 on-demand
!
!
crypto ipsec transform-set az-IPSEC-PROPOSAL-SET esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile az-VTI1
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE1
!
crypto ipsec profile az-VTI2
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE2
!
!
interface Loopback0
 ip address 172.16.0.3 255.255.255.255
!
interface Tunnel0
 ip address 172.16.0.1 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.253.112
 tunnel protection ipsec profile az-VTI1
!
interface Tunnel1
 ip address 172.16.0.2 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.250.191
 tunnel protection ipsec profile az-VTI2
!
interface GigabitEthernet1
 description External interface
 ip address x.243.229.110 255.255.255.252
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet2
 ip address 10.0.0.1 255.255.255.0
 negotiation auto
 no mop enabled
 no mop sysid
!
router bgp 65010
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 network 10.0.0.0 mask 255.255.255.0
 network 10.1.10.0 mask 255.255.255.128
 neighbor 10.2.0.228 remote-as 65000
 neighbor 10.2.0.228 ebgp-multihop 5
 neighbor 10.2.0.228 update-source Loopback0
 neighbor 10.2.0.228 soft-reconfiguration inbound
 neighbor 10.2.0.228 filter-list 10 out
 neighbor 10.2.0.229 remote-as 65000    
 neighbor 10.2.0.229 ebgp-multihop 5
 neighbor 10.2.0.229 update-source Loopback0
 neighbor 10.2.0.229 soft-reconfiguration inbound
 maximum-paths eibgp 2
!
ip route 0.0.0.0 0.0.0.0 10.1.10.1
ip route 10.2.0.228 255.255.255.255 Tunnel0
ip route 10.2.0.229 255.255.255.255 Tunnel1
!
```

## <a name="5-configure-vpn-device-filtering-and-firewalls-optional"></a><a name="firewalls"></a>5. Настройка фильтрации и брандмауэров VPN-устройств (необязательно)

Настройте брандмауэр и фильтрацию в соответствии с требованиями.

## <a name="6-test-and-validate-the-ipsec-tunnel"></a><a name="testipsec"></a>6. Тестирование и проверка туннеля IPsec

Статус туннелей IPsec можно проверить в VPN-шлюзе Azure с помощью команд PowerShell:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object  ConnectionStatus,EgressBytesTransferred,IngressBytesTransferred | fl
```

Выходные данные примера:

```azurepowershell
ConnectionStatus        : Connected
EgressBytesTransferred  : 17734660
IngressBytesTransferred : 10538211
```

Чтобы проверить состояние туннелей в экземплярах VPN-шлюза Azure независимо друг от друга, используйте следующий пример:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object -ExpandProperty TunnelConnectionStatus
```

Выходные данные примера:

```azurepowershell
Tunnel                           : vpn2local1_52.175.250.191
ConnectionStatus                 : Connected
IngressBytesTransferred          : 4877438
EgressBytesTransferred           : 8754071
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:30

Tunnel                           : vpn2local1_52.175.253.112
ConnectionStatus                 : Connected
IngressBytesTransferred          : 5660773
EgressBytesTransferred           : 8980589
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:13
```

Вы также можете проверить состояние туннеля на своем локальном VPN-устройстве.

Пример Cisco CSR1000

```
show crypto session detail
show crypto ikev2 sa
show crypto ikev2 session detail
show crypto ipsec sa
```

Выходные данные примера:

```
csr1#show crypto session detail

Crypto session current status

Code: C - IKE Configuration mode, D - Dead Peer Detection
K - Keepalives, N - NAT-traversal, T - cTCP encapsulation
X - IKE Extended Authentication, F - IKE Fragmentation
R - IKE Auto Reconnect

Interface: Tunnel1
Profile: az-PROFILE2
Uptime: 00:52:46
Session status: UP-ACTIVE
Peer: 52.175.250.191 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.250.191
      Desc: (none)
  Session ID: 3
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.250.191/4500 Active
          Capabilities:DN connid:3 lifetime:23:07:14
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 279 drop 0 life (KB/Sec) 4607976/433
        Outbound: #pkts enc'ed 164 drop 0 life (KB/Sec) 4607992/433

Interface: Tunnel0
Profile: az-PROFILE1
Uptime: 00:52:43
Session status: UP-ACTIVE
Peer: 52.175.253.112 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.253.112
      Desc: (none)
  Session ID: 2
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.253.112/4500 Active
          Capabilities:DN connid:2 lifetime:23:07:17
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 668 drop 0 life (KB/Sec) 4607926/437
        Outbound: #pkts enc'ed 477 drop 0 life (KB/Sec) 4607953/437
```

Протокол линии передачи данных в виртуальном интерфейсе туннеля (VTI) не включается, пока не завершится этап IKE 2. Следующая команда проверяет сопоставление безопасности:

```
csr1#show crypto ikev2 sa

IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         10.1.10.50/4500       52.175.253.112/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3277 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status
3         10.1.10.50/4500       52.175.250.191/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3280 sec

IPv6 Crypto IKEv2  SA

csr1#show crypto ipsec sa | inc encaps|decaps
    #pkts encaps: 177, #pkts encrypt: 177, #pkts digest: 177
    #pkts decaps: 296, #pkts decrypt: 296, #pkts verify: 296
    #pkts encaps: 554, #pkts encrypt: 554, #pkts digest: 554
    #pkts decaps: 746, #pkts decrypt: 746, #pkts verify: 746
```

### <a name="verify-end-to-end-connectivity-between-the-inside-network-on-premises-and-the-azure-vnet"></a><a name="verifye2e"></a>Проверка сквозной связи между внутренней локальной сетью и виртуальной сетью Azure

Если туннели IPsec работают и статические маршруты настроены правильно, вы сможете проверить связь с IP-адресом удаленного узла BGP:

```
csr1#ping 10.2.0.228
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.228, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms

#ping 10.2.0.229
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.229, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/6 ms
```

### <a name="verify-the-bgp-sessions-over-ipsec"></a><a name="verifybgp"></a>Проверка сеансов BGP через IPsec

Проверьте состояние узла BGP в VPN-шлюзе Azure:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayBGPPeerStatus -VirtualNetworkGatewayName vpnGtw -ResourceGroupName SEA-C1-VPN-ER | ft
```

Выходные данные примера:

```azurepowershell
  Asn ConnectedDuration LocalAddress MessagesReceived MessagesSent Neighbor    RoutesReceived State    
  --- ----------------- ------------ ---------------- ------------ --------    -------------- -----    
65010 00:57:19.9003584  10.2.0.228               68           72   172.16.0.10              2 Connected
65000                   10.2.0.228                0            0   10.2.0.228               0 Unknown  
65000 07:13:51.0109601  10.2.0.228              507          500   10.2.0.229               6 Connected
```

Чтобы проверить список префиксов сети, полученных через локальный VPN-концентратор eBGP, выполните фильтрацию по атрибуту "Origin":

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG  | Where-Object Origin -eq "EBgp" |ft
```

В выходных данных примера ASN 65010 — это номер автономной системы BGP в локальной сети VPN.

```azurepowershell
AsPath LocalAddress Network      NextHop     Origin SourcePeer  Weight
------ ------------ -------      -------     ------ ----------  ------
65010  10.2.0.228   10.1.10.0/25 172.16.0.10 EBgp   172.16.0.10  32768
65010  10.2.0.228   10.0.0.0/24  172.16.0.10 EBgp   172.16.0.10  32768
```

Чтобы просмотреть список объявленных маршрутов:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG -Peer 10.2.0.228 | ft
```

Выходные данные примера:

```azurepowershell
AsPath LocalAddress Network        NextHop    Origin SourcePeer Weight
------ ------------ -------        -------    ------ ---------- ------
       10.2.0.229   10.2.0.0/24    10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.10/32 10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.5/32  10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.1/32  10.2.0.229 Igp                  0
65010  10.2.0.229   10.1.10.0/25   10.2.0.229 Igp                  0
65010  10.2.0.229   10.0.0.0/24    10.2.0.229 Igp                  0
```

Пример для CSR1000 Cisco в локальной среде.

```
csr1#show ip bgp neighbors 10.2.0.228 routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.2.0.0/24      10.2.0.228                             0 65000 i
 r>   172.16.0.1/32    10.2.0.228                             0 65000 i
 r>   172.16.0.2/32    10.2.0.228                             0 65000 i
 r>   172.16.0.3/32   10.2.0.228                             0 65000 i

Total number of prefixes 4
```

Список сетей, объявленных из локального Cisco CSR1000 для VPN-шлюза Azure, можно указать с помощью следующей команды:

```
csr1#show ip bgp neighbors 10.2.0.228 advertised-routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.0.0.0/24      0.0.0.0                  0         32768 i
 *>   10.1.10.0/25     0.0.0.0                  0         32768 i

Total number of prefixes 2
```

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка решения "Монитор производительности сети" для ExpressRoute](how-to-npm.md)

* [Добавление подключения типа "сеть — сеть" в виртуальную сеть с существующим VPN-шлюзом](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
