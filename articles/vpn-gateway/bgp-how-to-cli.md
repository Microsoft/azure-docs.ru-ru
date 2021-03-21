---
title: 'Настройка BGP на VPN-шлюзе Azure: CLI'
description: В этой статье описана поэтапная настройка протокола BGP для VPN-шлюза Azure с помощью Azure Resource Manager и интерфейса командной строки.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: a69ce0592b79be0868dd7c15ac054910eee75fc7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89393604"
---
# <a name="how-to-configure-bgp-on-an-azure-vpn-gateway-by-using-cli"></a>Настройка BGP на VPN-шлюзе Azure с помощью интерфейса командной строки

Сведения из этой статьи помогут вам с помощью модели развертывания Azure Resource Manager и Azure CLI включить BGP для VPN-подключения типа "сеть — сеть" (S2S), настроенного между локальными сетями, и подключения типа "виртуальная сеть — виртуальная сеть" (это подключение между виртуальными сетями).

## <a name="about-bgp"></a>О протоколе BGP

Стандартный протокол маршрутизации BGP обычно используется в Интернете для обмена данными о маршрутизации и доступности между двумя или несколькими сетями. Протокол BGP используется VPN-шлюзами Azure и локальными VPN-устройствами (так называемые узлы BGP) для обмена данными о маршрутах. Это позволяет информировать участников обмена о доступности и возможности передавать трафик для определенных сетей через шлюзы или маршрутизаторы. Кроме того, BGP позволяет передавать трафик транзитом через несколько сетей. Для этого шлюз BGP распространяет на все известные ему узлы BGP информацию о маршрутах, полученную от остальных узлов BGP.

Дополнительные сведения о преимуществах BGP, о технических требованиях и важных аспектах использования BGP см. в статье [Обзор использования BGP с VPN-шлюзами Azure](vpn-gateway-bgp-overview.md).

Эта статья поможет вам в решении следующих задач:

* [Включение BGP для VPN-шлюза](#enablebgp) (обязательно).

  Затем можно ознакомиться с одним из следующих разделов (или c обоими):

* [Установка подключения между организациями с помощью BGP](#crossprembgp)
* [Создание подключения между виртуальными сетями с использованием BGP](#v2vbgp).

Каждый из этих трех разделов является базовым блоком для использования BGP в вашей сети. Выполнив инструкции во всех трех разделах, приведенные в трех частях, вы создадите топологию, которая представлена на следующей схеме:

![Топология BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Вы можете объединять разделы для создания более сложных, многоскачковых и транзитных сетей в соответствии со своими задачами.

## <a name="enable-bgp-for-your-vpn-gateway"></a><a name ="enablebgp"></a>Включение BGP для VPN-шлюза

Выполните действия в этом разделе, прежде чем выполнять какие-либо шаги в остальных двух разделах конфигурации. Описанные ниже действия позволят настроить параметры BGP для VPN-шлюза Azure в соответствии со следующей схемой:

![Шлюз BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Перед началом

Установите последнюю версию команд интерфейса командной строки (версии 2.0 или более позднюю). См. дополнительные сведения об [установке Azure CLI 2.0](/cli/azure/install-azure-cli) и [начале работы с Azure CLI 2.0](/cli/azure/get-started-with-azure-cli).

### <a name="step-1-create-and-configure-testvnet1"></a>Шаг 1. Создание и настройка TestVNet1

#### <a name="1-connect-to-your-subscription"></a><a name="Login"></a>1. подключение к подписке

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

#### <a name="2-create-a-resource-group"></a>2. Создание группы ресурсов

В следующем примере создается группа ресурсов с именем TestRG1 в расположении eastus. Если у вас уже есть группа ресурсов в регионе, где вы хотите создать виртуальную сеть, вы можете воспользоваться ею.

```azurecli
az group create --name TestBGPRG1 --location eastus
```

#### <a name="3-create-testvnet1"></a>3. Создание TestVNet1

В примере ниже создается виртуальная сеть с именем TestVNet1 и три подсети: GatewaySubnet, FrontEnd и BackEnd. При замене значений важно, чтобы вы назвали подсеть шлюза именем GatewaySubnet. Если вы используете другое имя, создание шлюза завершится сбоем.

Первая команда создает адресное пространство переднего плана и подсеть FrontEnd. Вторая команда создает дополнительное адресное пространство для подсети BackEnd. Третья и четвертая команды создают подсеть BackEnd и GatewaySubnet.

```azurecli
az network vnet create -n TestVNet1 -g TestBGPRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24 
 
az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestBGPRG1 
 
az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestBGPRG1 --address-prefix 10.12.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestBGPRG1 --address-prefix 10.12.255.0/27 
```

### <a name="step-2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Шаг 2. Создание VPN-шлюза для TestVNet1 с параметрами BGP

#### <a name="1-create-the-public-ip-address"></a>1. Создайте общедоступный IP-адрес.

Запросите общедоступный IP-адрес. VPN-шлюзу, созданному для виртуальной сети, выделяется общедоступный IP-адрес.

```azurecli
az network public-ip create -n GWPubIP -g TestBGPRG1 --allocation-method Dynamic 
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Создайте VPN-шлюз с номером AS.

Создайте шлюз для виртуальной сети TestVNet1. BGP требует VPN-шлюз на основе маршрута. Необходим также дополнительный параметр `-Asn`, чтобы задать номер автономной системы для TestVNet1. Создание шлюза может занять некоторое время (45 минут или более). 

При выполнении этой команды с использованием параметра `--no-wait` вы не увидите ответа или выходных данных. Параметр `--no-wait` позволяет создать шлюз в фоновом режиме. Это не означает, что создание VPN-шлюза завершается немедленно.

```azurecli
az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address GWPubIP -g TestBGPRG1 --vnet TestVNet1 --gateway-type Vpn --sku HighPerformance --vpn-type RouteBased --asn 65010 --no-wait
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. получение IP-адреса узла BGP Azure

После создания шлюза необходимо получить IP-адрес для узла BGP на VPN-шлюзе Azure. Этот адрес нужен, чтобы VPN-шлюз мог выполнять роль узла BGP для локальных VPN-устройств.

Выполните следующую команду и проверьте раздел `bgpSettings` в верхней части выходных данных:

```azurecli
az network vnet-gateway list -g TestBGPRG1 
 
  
"bgpSettings": { 
      "asn": 65010, 
      "bgpPeeringAddress": "10.12.255.30", 
      "peerWeight": 0 
    }
```

С помощью созданного шлюза можно установить подключение между локальными или виртуальными сетями с использованием BGP.

## <a name="establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembgp"></a>Создание подключения между локальными сетями с использованием BGP

Чтобы установить подключение между локальными сетями, нужно создать локальный сетевой шлюз, который будет представлять локальное VPN-устройство. Затем необходимо создать подключение между VPN-шлюзом и шлюзом локальной сети. Хотя эти шаги похожи на создание других подключений, они включают дополнительные свойства, необходимые для указания параметров конфигурации BGP.

![BGP между локальными сетями](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)


### <a name="step-1-create-and-configure-the-local-network-gateway"></a>Шаг 1. Создание и настройка локального сетевого шлюза

В этом подразделе мы продолжим создание конфигурации, которая представлена на схеме. Не забудьте заменить значения теми, которые вы хотите использовать для конфигурации. При работе со шлюзами локальной сети учитывайте следующее:

* Локальный сетевой шлюз может находиться в том же расположении, что и VPN-шлюз, или в любом другом. Это же справедливо в отношении групп ресурсов. В нашем примере шлюзы располагаются в разных группах ресурсов и расположениях.
* Минимальный префикс, который необходимо объявить для локального сетевого шлюза — это IP-адрес узла BGP на вашем VPN-устройстве. В этом случае это префикс/32 10.51.255.254/32.
* Не забывайте, что для локальных сетей и виртуальной сети Azure должны быть указаны разные номера ASN BGP. Если они совпадают, а локальное VPN-устройство уже использует свой ASN для связи с другими соседями BGP, необходимо изменить ASN вашей виртуальной сети.

Прежде чем продолжить, убедитесь, что выполнены шаги из раздела [Включение BGP для VPN-шлюза](#enablebgp) этой статьи и что вы по-прежнему подключены к подписке 1. Обратите внимание, что в этом примере создается группа ресурсов. Обратите внимание также на два дополнительных параметра локального сетевого шлюза: `Asn` и `BgpPeerAddress`.

```azurecli
az group create -n TestBGPRG5 -l eastus2 
 
az network local-gateway create --gateway-ip-address 23.99.221.164 -n Site5 -g TestBGPRG5 --local-address-prefixes 10.51.255.254/32 --asn 65050 --bgp-peering-address 10.51.255.254
```

### <a name="step-2-connect-the-vnet-gateway-and-local-network-gateway"></a>Шаг 2. Подключение шлюза виртуальной сети к локальному сетевому шлюзу

На этом шаге вы создадите подключение между TestVNet1 и Site5. Необходимо указать параметр `--enable-bgp`, чтобы включить протокол BGP для этого подключения. 

В этом примере шлюз виртуальной и локальной сети находятся в разных группах ресурсов. Если шлюзы находятся в разных группах ресурсов, необходимо полностью указать идентификатор ресурса двух шлюзов, чтобы настроить подключение между виртуальными сетями.

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Получение идентификатора ресурса VNet1GW

Используйте выходные данные следующей команды, чтобы получить идентификатор ресурса для VNet1GW:

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

В выходных данных найдите строку `"id":`. Значения в кавычках понадобятся для создания подключения в следующем разделе.

Выходные данные примера:

```
{ 
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65010, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
  }, 
  "enableBgp": true, 
  "etag": "W/\"<your etag number>\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
```

Скопируйте значения после `"id":` в текстовый редактор, например Блокнот, чтобы их можно было быстро вставить при создании подключения. 

```
"id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
```

#### <a name="2-get-the-resource-id-of-site5"></a>2. Получите идентификатор ресурса site5

Выполните следующую команду, чтобы получите идентификатор ресурса Site5 из выходных данных:

```azurecli
az network local-gateway show -n Site5 -g TestBGPRG5
```

#### <a name="3-create-the-testvnet1-to-site5-connection"></a>3. Создание подключения TestVNet1-to-site5

На этом шаге вы создадите подключение между TestVNet1 и Site5. Как обсуждалось ранее, на одном VPN-шлюзе можно одновременно создавать подключения с использованием BGP и без него. Если BGP не указано в свойствах подключения явным образом, Azure не будет использовать BGP для этого подключения, даже если параметры BGP настроены для обоих шлюзов. Замените идентификатор подписки собственным идентификатором.

```azurecli
az network vpn-connection create -n VNet1ToSite5 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "abc123" --local-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG5/providers/Microsoft.Network/localNetworkGateways/Site5 --no-wait
```

В следующем примере перечислены параметры, которые следует ввести в разделе конфигурации BGP на локальном VPN-устройстве для нашего упражнения:

```
Site5 ASN            : 65050
Site5 BGP IP         : 10.52.255.254
Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
Azure VNet ASN       : 65010
Azure VNet BGP IP    : 10.12.255.30
Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Подключение должно установиться через несколько минут. Сразу после создания подключения IPsec начнется сеанс пиринга BGP.

## <a name="establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Создание подключения между виртуальными сетями с использованием BGP.

В этом разделе мы добавим подключение между виртуальными сетями с использованием BGP, как показано на следующей схеме: 

![BGP между виртуальными сетями](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Приведенные ниже инструкции продолжают действия, описанные в предыдущих разделах. Выполните шаги из раздела [Включение BGP для VPN-шлюза](#enablebgp), чтобы создать и настроить TestVNet1 и VPN-шлюз с BGP.

### <a name="step-1-create-testvnet2-and-the-vpn-gateway"></a>Шаг 1. Создание TestVNet2 и VPN-шлюза

Важно убедиться в том, что пространство IP-адресов для новой виртуальной сети TestVNet2 не пересекается с диапазонами любой из ваших виртуальных сетей.

В этом примере виртуальные сети относятся к одной подписке. Вы можете создавать подключения между виртуальным сетями из разных подписок. Дополнительные сведения см. в статье [Настройка подключения VPN-шлюза между виртуальными сетями с помощью Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md). Чтобы использовать для подключения протокол BGP, обязательно укажите параметр `-EnableBgp $True` при создании подключения.

#### <a name="1-create-a-new-resource-group"></a>1. Создайте новую группу ресурсов.

```azurecli
az group create -n TestBGPRG2 -l westus
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Создайте TestVNet2 в новой группе ресурсов.

Первая команда создает адресное пространство переднего плана и подсеть FrontEnd. Вторая команда создает дополнительное адресное пространство для подсети BackEnd. Третья и четвертая команды создают подсеть BackEnd и GatewaySubnet.

```azurecli
az network vnet create -n TestVNet2 -g TestBGPRG2 --address-prefix 10.21.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.21.0.0/24 
 
az network vnet update -n TestVNet2 --address-prefixes 10.21.0.0/16 10.22.0.0/16 -g TestBGPRG2 
 
az network vnet subnet create --vnet-name TestVNet2 -n BackEnd -g TestBGPRG2 --address-prefix 10.22.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet2 -n GatewaySubnet -g TestBGPRG2 --address-prefix 10.22.255.0/27
```

#### <a name="3-create-the-public-ip-address"></a>3. Создайте общедоступный IP-адрес.

Запросите общедоступный IP-адрес. VPN-шлюзу, созданному для виртуальной сети, выделяется общедоступный IP-адрес.

```azurecli
az network public-ip create -n GWPubIP2 -g TestBGPRG2 --allocation-method Dynamic
```

#### <a name="4-create-the-vpn-gateway-with-the-as-number"></a>4. Создайте VPN-шлюз с номером AS.

Создайте шлюз для виртуальной сети TestVNet2. Номер ASN по умолчанию необходимо переопределить для ваших VPN-шлюзов Azure. Номера ASN для подключенных виртуальных сетей должны быть разными, чтобы работал протокол BGP и транзитная маршрутизация.
 
```azurecli
az network vnet-gateway create -n VNet2GW -l westus --public-ip-address GWPubIP2 -g TestBGPRG2 --vnet TestVNet2 --gateway-type Vpn --sku Standard --vpn-type RouteBased --asn 65020 --no-wait
```

### <a name="step-2-connect-the-testvnet1-and-testvnet2-gateways"></a>Шаг 2. Подключение шлюзов TestVNet1 и TestVNet2

На этом шаге вы создадите подключение между TestVNet1 и Site5. Необходимо указать параметр `--enable-bgp`, чтобы включить протокол BGP для этого подключения.

В следующем примере шлюз виртуальной и локальной сети находятся в разных группах ресурсов. Если шлюзы находятся в разных группах ресурсов, необходимо полностью указать идентификатор ресурса двух шлюзов, чтобы настроить подключение между виртуальными сетями. 

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Получение идентификатора ресурса VNet1GW 

Получите идентификатор ресурса VNet1GW из выходных данных следующей команды:

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

#### <a name="2-get-the-resource-id-of-vnet2gw"></a>2. Получите идентификатор ресурса VNet2GW

Получите идентификатор ресурса VNet2GW из выходных данных следующей команды:

```azurecli
az network vnet-gateway show -n VNet2GW -g TestBGPRG2
```

#### <a name="3-create-the-connections"></a>3. Создание подключений

Создайте подключение из TestVNet1 к TestVNet2, а также подключение из TestVNet2 к TestVNet1. Замените идентификатор подписки собственным идентификатором.

```azurecli
az network vpn-connection create -n VNet1ToVNet2 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW
```

```azurecli
az network vpn-connection create -n VNet2ToVNet1 -g TestBGPRG2 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW --enable-bgp -l westus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
```

> [!IMPORTANT]
> Включите BGP для *обоих* подключений.
> 
> 

Подключение будет установлено через несколько минут после выполнения этих действий. Сразу после создания подключения между виртуальными сетями начнется сеанс пиринга BGP.

## <a name="next-steps"></a>Дальнейшие действия

Установив подключение, можно добавить виртуальные машины в виртуальные сети. Пошаговые инструкции см. в статье о [создании виртуальной машины](../virtual-machines/windows/quick-create-portal.md).
