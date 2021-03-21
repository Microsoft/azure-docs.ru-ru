---
title: 'Подключение локальных сетей к виртуальной сети: подключение типа "сеть — сеть" с помощью интерфейса командной строки'
description: Создайте подключение шлюза IPsec типа "сеть — сеть" из локальной сети к виртуальной сети Azure через общедоступный Интернет с помощью интерфейса командной строки.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/23/2020
ms.author: cherylmc
ms.openlocfilehash: 2c59c67eb7b5ae5b26ac5517afba433fe8c028fa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104611752"
---
# <a name="create-a-virtual-network-with-a-site-to-site-vpn-connection-using-cli"></a>Создание виртуальной сети с VPN типа "сеть — сеть" с помощью интерфейса командной строки

В этой статье показано, как с помощью Azure CLI создавать подключение типа "сеть — сеть" с использованием VPN-шлюза между вашей локальной сетью к виртуальной. Приведенные в этой статье инструкции относятся к модели развертывания с помощью Resource Manager. Эту конфигурацию также можно создать с помощью разных средств или моделей развертывания, выбрав вариант из следующего списка:<br>

> [!div class="op_single_selector"]
> * [Портал Azure](./tutorial-site-to-site-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Портал Azure (классический)](vpn-gateway-howto-site-to-site-classic-portal.md)

![Схема подключения типа "сеть — сеть" через VPN-шлюз](./media/vpn-gateway-howto-site-to-site-resource-manager-cli/site-to-site-diagram.png)

Подключение VPN-шлюза типа "сеть — сеть" используется для подключения между локальной сетью и виртуальной сетью Azure через туннель VPN по протоколу IPsec/IKE (IKEv1 или IKEv2). Для этого типа подключения требуется локальное VPN-устройство, которому назначен внешний общедоступный IP-адрес. Дополнительные сведения о VPN-шлюзах см. в [этой статье](vpn-gateway-about-vpngateways.md).

## <a name="before-you-begin"></a>Перед началом

Перед началом настройки убедитесь, что удовлетворены следующие требования:

* Убедитесь, что у вас есть совместимое VPN–устройство и пользователь, который может настроить его. Дополнительные сведения о совместимых устройствах VPN и их настройке см. в [этой статье](vpn-gateway-about-vpn-devices.md).
* Убедитесь, что у вас есть общедоступный IPv4–адрес для вашего VPN–устройства.
* Если вы не знаете диапазоны IP-адресов в своей конфигурации локальной сети, найдите того, кто сможет предоставить вам нужную информацию. При создании этой конфигурации необходимо указать префиксы диапазона IP-адресов, которые Azure будет направлять к локальному расположению. Ни одна из подсетей локальной сети не может перекрывать виртуальные подсети, к которым вы хотите подключиться.
[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
* Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

### <a name="example-values"></a><a name="example"></a>Примеры значений

Следующие значения можно использовать для создания тестовой среды или для лучшего понимания примеров в этой статье.

```
#Example values

VnetName                = TestVNet1 
ResourceGroup           = TestRG1 
Location                = eastus 
AddressSpace            = 10.11.0.0/16 
SubnetName              = Subnet1 
Subnet                  = 10.11.0.0/24 
GatewaySubnet           = 10.11.255.0/27 
LocalNetworkGatewayName = Site2 
LNG Public IP           = <VPN device IP address>
LocalAddrPrefix1        = 10.0.0.0/24
LocalAddrPrefix2        = 20.0.0.0/24   
GatewayName             = VNet1GW 
PublicIP                = VNet1GWIP 
VPNType                 = RouteBased 
GatewayType             = Vpn 
ConnectionName          = VNet1toSite2
```

## <a name="1-connect-to-your-subscription"></a><a name="Login"></a>1. подключение к подписке

Для локального выполнения интерфейса командной строки необходимо подключиться к подписке. Если вы используете Azure Cloud Shell в браузере, подключаться к подписке не требуется. В таком случае подключение к Azure Cloud Shell будет устанавливаться автоматически. Тем не менее при необходимости после подключения вы можете убедиться, что используется правильная подписка.

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

## <a name="2-create-a-resource-group"></a><a name="rg"></a>2. Создание группы ресурсов

В следующем примере создается группа ресурсов с именем TestRG1 в расположении eastus. Если у вас уже есть группа ресурсов в регионе, где вы хотите создать виртуальную сеть, вы можете воспользоваться ею.

```azurecli-interactive
az group create --name TestRG1 --location eastus
```

## <a name="3-create-a-virtual-network"></a><a name="VNet"></a>3. Создание виртуальной сети

Если у вас еще нет виртуальной сети, создайте ее, используя команду [az network vnet create](/cli/azure/network/vnet). При создании виртуальной сети убедитесь, что указанные адресные пространства не перекрываются ни с одним из адресных пространств, которые используются в локальной сети.

>[!NOTE]
>Для подключения этой виртуальной сети к локальному расположению вам необходимо обратиться к администратору локальной сети, чтобы выделить диапазон IP-адресов, который будет использоваться специально для этой виртуальной сети. Если повторяющийся диапазон адресов существует на обеих сторонах VPN-подключения, трафик не будет маршрутизироваться должным образом. Кроме того, если вы хотите подключить виртуальную сеть к другой виртуальной сети, адресное пространство не должно пересекаться с другой виртуальной сетью. Спланируйте конфигурацию сети соответствующим образом.
>
>

В следующем примере создаются виртуальная сеть TestVNet1 и подсеть Subnet1.

```azurecli-interactive
az network vnet create --name TestVNet1 --resource-group TestRG1 --address-prefix 10.11.0.0/16 --location eastus --subnet-name Subnet1 --subnet-prefix 10.11.0.0/24
```

## <a name="4-create-the-gateway-subnet"></a>4. <a name="gwsub"></a> Создание подсети шлюза


[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-include.md)]

Используйте команду [azure network vnet subnet create](/cli/azure/network/vnet/subnet), чтобы создать шлюз подсети.

```azurecli-interactive
az network vnet subnet create --address-prefix 10.11.255.0/27 --name GatewaySubnet --resource-group TestRG1 --vnet-name TestVNet1
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="5-create-the-local-network-gateway"></a><a name="localnet"></a>5. Создание шлюза локальной сети

Обычно термин "шлюз локальной сети" означает локальное расположение. Присвойте сайту имя, по которому Azure может обращаться к этому сайту, а затем укажите IP-адрес локального VPN-устройства, к которому вы подключитесь. Вы можете также указать префиксы IP-адресов, которые будут направляться через VPN-шлюз к VPN-устройству. Указываемые префиксы адресов расположены в локальной сети. При изменении локальной сети вы сможете без проблем обновить эти префиксы.

Используйте следующие значения.

* *--Gateway-IP-Address* — это IP-адрес локального VPN-устройства.
* *--local-address-prefixes* — это локальные адресные пространства.

Используйте команду [az network local-gateway create](/cli/azure/network/local-gateway),чтобы добавить шлюз локальной сети с несколькими префиксами адресов:

```azurecli-interactive
az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --resource-group TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24
```

## <a name="6-request-a-public-ip-address"></a><a name="PublicIP"></a>6. запрос общедоступного IP-адреса

VPN-шлюз должен иметь общедоступный IP-адрес. Сначала запросите ресурс IP-адреса, а затем укажите его при создании шлюза виртуальной сети. IP-адрес динамически назначается ресурсу при создании VPN-шлюза. В настоящее время VPN-шлюз поддерживает выделение *динамических* общедоступных IP-адресов. Вы не можете запросить назначение статического общедоступного IP-адреса. Однако это не означает, что IP-адрес изменяется после назначения VPN-шлюзу. Общедоступный IP-адрес изменяется только после удаления и повторного создания шлюза. При изменении размера, сбросе или других внутренних операциях обслуживания или обновления IP-адрес VPN-шлюза не изменяется.

Используйте команду [az network public-ip create](/cli/azure/network/public-ip), чтобы запросить динамический общедоступный IP-адрес.

```azurecli-interactive
az network public-ip create --name VNet1GWIP --resource-group TestRG1 --allocation-method Dynamic
```

## <a name="7-create-the-vpn-gateway"></a><a name="CreateGateway"></a>7. Создание VPN-шлюза

Создайте VPN-шлюз виртуальной сети. Создание VPN-шлюза может занять от 45 минут и более.

Используйте следующие значения.

* Параметр *--Gateway-Type* для конфигурации типа "сеть — сеть" — *VPN*. Тип шлюза всегда зависит от реализуемой конфигурации. Дополнительные сведения см. в разделе [Типы шлюзов](vpn-gateway-about-vpn-gateway-settings.md#gwtype).
* У параметра *--vpn-type* может быть значение *RouteBased* (в некоторых документах такой шлюз называется шлюзом с динамической маршрутизацией) или *PolicyBased* (в некоторых документах — шлюз со статической маршрутизацией). Этот параметр зависит от требований устройства, к которому вы подключаетесь. Дополнительные сведения о VPN-шлюзах см. в [этой статье](vpn-gateway-about-vpn-gateway-settings.md#vpntype).
* Выберите SKU шлюза, который нужно использовать. К определенным номерам SKU применяются ограничения настройки. Дополнительные сведения см. в разделе о [номерах SKU шлюзов](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

Чтобы создать VPN-шлюз, используйте команду [az network vnet-gateway create](/cli/azure/network/vnet-gateway). При выполнении этой команды с использованием параметра --no-wait вы не увидите ответа или выходных данных. Этот параметр позволяет создать шлюз в фоновом режиме. Процесс создания шлюза занимает около 45 минут.

```azurecli-interactive
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWIP --resource-group TestRG1 --vnet TestVNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait 
```

## <a name="8-configure-your-vpn-device"></a><a name="VPNDevice"></a>8. Настройка VPN-устройства

Для подключения типа "сеть — сеть" к локальной сети требуется VPN-устройство. На этом этапе мы настроим VPN-устройство. Чтобы настроить локальное VPN-устройство, вам потребуется следующее:

- Общий ключ. Это тот же общий ключ, который указывается при создании VPN-подключения "сеть — сеть". В наших примерах мы используем простые общие ключи. Для практического использования рекомендуется создавать более сложные ключи.
- Общедоступный IP-адрес шлюза виртуальной сети. Общедоступный IP-адрес можно просмотреть с помощью портала Azure, PowerShell или CLI. Чтобы найти общедоступный IP-адрес шлюза виртуальной сети, используйте команду [az network public-ip list](/cli/azure/network/public-ip). Для удобства чтения выходные данные форматируются для отображения списка общедоступных IP-адресов в формате таблицы.

  ```azurecli-interactive
  az network public-ip list --resource-group TestRG1 --output table
  ```


[!INCLUDE [Configure VPN device](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]


## <a name="9-create-the-vpn-connection"></a><a name="CreateConnection"></a>9. Создание VPN-подключения

Создайте VPN-подключение типа "сеть — сеть" между шлюзом виртуальной сети и локальным VPN-устройством. Обратите особое внимание на значение общего ключа, которое должно соответствовать значению заданного общего ключа для VPN-устройства.

Создайте подключение с помощью команды [az network vpn-connection create](/cli/azure/network/vpn-connection).

```azurecli-interactive
az network vpn-connection create --name VNet1toSite2 --resource-group TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key abc123 --local-gateway2 Site2
```

Через некоторое время будет установлено подключение.

## <a name="10-verify-the-vpn-connection"></a><a name="toverify"></a>10. Проверка VPN-подключения

[!INCLUDE [verify connection](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

Если вы хотите использовать другой метод проверки подключения, см. статью [Проверка соединения VPN-шлюза](vpn-gateway-verify-connection-resource-manager.md).

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Подключение к виртуальной машине

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

## <a name="common-tasks"></a><a name="tasks"></a>Стандартные задачи

Этот раздел содержит общие команды, которые могут быть полезны при работе с конфигурациями подключения типа "сайт — сайт". Полный список сетевых команд интерфейса командной строки см. в разделе об [управлении ресурсами сети Azure](/cli/azure/network).

[!INCLUDE [local network gateway common tasks](../../includes/vpn-gateway-common-tasks-cli-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Установив подключение, можно добавить виртуальные машины в виртуальные сети. Дополнительные сведения см. в статье [виртуальные машины](../index.yml).
* Сведения о BGP см. в статьях [Обзор использования BGP с VPN-шлюзами Azure](vpn-gateway-bgp-overview.md) и [Настройка BGP на VPN-шлюзах Azure с помощью Azure Resource Manager и PowerShell](vpn-gateway-bgp-resource-manager-ps.md).
* Дополнительные сведения о принудительном туннелировании см. в статье [о принудительном туннелировании](vpn-gateway-forced-tunneling-rm.md).
* Сведения о высокодоступных подключениях в режиме "активный — активный" см. в статье [Настройка высокодоступных подключений: распределенных и между виртуальными сетями](vpn-gateway-highlyavailable.md).
* Список сетевых команд Azure CLI см. в статье об [Azure CLI](/cli/azure/network).
* Сведения о создании VPN-подключения типа "сеть — сеть" с помощью шаблона Azure Resource Manager см. [в разделе Создание VPN-подключения](https://azure.microsoft.com/resources/templates/101-site-to-site-vpn-create/)типа "сеть — сеть".
* Сведения о создании VPN-подключения между виртуальными сетями с помощью шаблона Azure Resource Manager см. в статье [развертывание георепликации HBase](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-replication-geo/).