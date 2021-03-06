---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/22/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 0eb509d543bc9b4a8846319126185eb1f27ec7a2
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104953462"
---
### <a name="is-bgp-supported-on-all-azure-vpn-gateway-skus"></a>VPN-шлюзы Azure поддерживают BGP для всех классов SKU?

Протокол BGP поддерживается для всех SKU VPN-шлюза Azure, за исключением SKU уровня "Базовый".

### <a name="can-i-use-bgp-with-azure-policy-vpn-gateways"></a>Можно ли использовать BGP с VPN-шлюзами на основе Политики Azure?

Нет, BGP поддерживает только VPN-шлюзы с управлением на основе маршрута.

### <a name="what-asns-autonomous-system-numbers-can-i-use"></a>Какие номера ASN можно использовать?

Вы можете использовать собственные общедоступные или частные ASN как для локальных сетей, так и для виртуальных сетей Azure. Диапазоны, зарезервированные Azure или IANA, использовать нельзя.

Следующие номера ASN зарезервированы для Azure и IANA.
* Номера ASN, зарезервированные для Azure:

  * общедоступные ASN — 8074, 8075, 12076;
  * частные ASN — 65515, 65517, 65518, 65519, 65520.
* Номера ASN, [зарезервированные для IANA](http://www.iana.org/assignments/iana-as-numbers-special-registry/iana-as-numbers-special-registry.xhtml):

  * 23456, 64496-64511, 65535-65551 и 429496729.

При подключении к VPN-шлюзам Azure эти номера ASN нельзя указывать для локальных VPN-устройств.

### <a name="can-i-use-32-bit-4-byte-asns"></a>Можно ли использовать 32-разрядный (4-байтовый) номер ASN?

Да, VPN-шлюз теперь поддерживает 32-разрядный (4-байтовый) номер ASN. Чтобы выполнить настройку с помощью ASN в десятичном формате, используйте PowerShell, Azure CLI или пакет Azure SDK.

### <a name="what-private-asns-can-i-use"></a>Какие частные ASN можно использовать?

Доступные диапазоны частных ASN:

* 64512–65514 и 65521–65534

Эти ASN не зарезервированы для использования IANA или Azure, поэтому их можно назначить VPN-шлюзу Azure.

### <a name="what-address-does-vpn-gateway-use-for-bgp-peer-ip"></a>Какой адрес VPN-шлюз использует в качестве IP-адреса узла BGP?

По умолчанию VPN-шлюз выделяет из диапазона *GatewaySubnet* один IP-адрес для VPN-шлюзов в режиме "активный — резервный" или два IP-адреса для VPN-шлюзов в режиме "активный — активный". Эти адреса выделяются автоматически при создании VPN-шлюза. Вы можете получить фактический выделенный IP-адрес BGP с помощью PowerShell или на портале Azure. В PowerShell используйте **Get-AzVirtualNetworkGateway** и найдите свойство **bgpPeeringAddress**. На портале Azure на странице **Конфигурация шлюза** просмотрите свойство **Настройка BGP ASN**.

Если локальные маршрутизаторы VPN используют IP-адреса **APIPA** (169.254.x.x) в качестве IP-адресов BGP, необходимо указать дополнительный **IP-адрес BGP APIPA Azure** на VPN-шлюзе Azure. VPN-шлюз Azure выберет адрес APIPA для использования с локальным узлом BGP APIPA, указанным в шлюзе локальной сети, или частным IP-адресом для локального узла BGP, не связанного с APIPA. См. статью [Настройка BGP на VPN-шлюзах Azure](../articles/vpn-gateway/bgp-howto.md).

### <a name="what-are-the-requirements-for-the-bgp-peer-ip-addresses-on-my-vpn-device"></a>Каковы требования к IP-адресам узла BGP на VPN-устройстве?

Локальный узел BGP не должен иметь общедоступный IP-адрес VPN-устройства или входить в диапазон IP-адресов виртуальной сети VPN-шлюза. Используйте в качестве IP-адреса узла BGP другой IP-адрес VPN-устройства. Это может быть адрес, назначенный интерфейсу внутреннего замыкания на устройстве (обычный IP-адрес или адрес APIPA). Если устройство использует для BGP адрес APIPA, необходимо указать IP-адрес BGP APIPA на VPN-шлюзе Azure, как описано в статье [Настройка BGP](../articles/vpn-gateway/bgp-howto.md). Укажите этот адрес на локальном сетевом шлюзе, который представляет это расположение.

### <a name="what-should-i-specify-as-my-address-prefixes-for-the-local-network-gateway-when-i-use-bgp"></a>Что нужно указать в качестве префикса адреса локального сетевого шлюза, если используется BGP?

> [!IMPORTANT]
>
>Это изменение из задокументированного ранее требования. При использовании BGP для подключения следует оставить поле **Адресное пространство** пустым для соответствующего ресурса шлюза локальной сети. VPN-шлюз Azure добавит маршрут узла в IP-адрес локального узла BGP через туннель IPsec. Не добавляйте маршрут /32 в поле **Диапазон адресов**. Он является избыточным. Если вы используете адрес APIPA в качестве IP-адреса BGP VPN-устройства, его нельзя добавлять в это поле. Другие префиксы будут добавлены в поле **Диапазон адресов** как статические маршруты на VPN-шлюзе Azure, наряду с маршрутами, полученными через BGP.
>

### <a name="can-i-use-the-same-asn-for-both-on-premises-vpn-networks-and-azure-virtual-networks"></a>Можно ли использовать один номер ASN одновременно для локальных VPN-сетей и виртуальных сетей Azure?

Нет, следует назначить различные ASN для локальных сетей и виртуальных сетей Azure, если они обмениваются данными с использованием BGP. VPN-шлюзы Azure по умолчанию получают значение ASN 65515, независимо от использования BGP для подключений между локальными сетями. Вы можете переопределить это значение по умолчанию, назначив другой номер ASN при создании VPN-шлюза или изменив его после создания шлюза. Номер ASN, который используется для локальной сети, нужно будет назначить соответствующему локальному сетевому шлюзу Azure.

### <a name="what-address-prefixes-will-azure-vpn-gateways-advertise-to-me"></a>Какие префиксы адресов будут объявлять для меня VPN-шлюзы Azure?

VPN-шлюз объявляет следующие маршруты для локальных устройств BGP:

* префиксы адресов виртуальной сети;
* префиксы адресов от каждого из локальных сетевых шлюзов, подключенных к VPN-шлюзу Azure;
* маршруты, полученные от сеансов пиринга BGP с другими устройствами, подключенными к VPN-шлюзу Azure, за исключением маршрута по умолчанию и маршрутов, пересекающихся с префиксами любой виртуальной сети.

### <a name="how-many-prefixes-can-i-advertise-to-azure-vpn-gateway"></a>Сколько префиксов можно объявлять VPN-шлюзу Azure?

VPN-шлюз Azure поддерживает до 4000 префиксов. Если количество префиксов превысит лимит, сеанс BGP будет сброшен.

### <a name="can-i-advertise-default-route-00000-to-azure-vpn-gateways"></a>Можно ли объявить маршрут по умолчанию (0.0.0.0/0) к VPN-шлюзам Azure?

Да. Обратите внимание, что в результате весь исходящий трафик виртуальной сети будет направлен на локальный сайт. Кроме того, виртуальные машины в виртуальной сети не смогут принимать общедоступный трафик из Интернета напрямую, например RDP или SSH из Интернета на виртуальные машины.

### <a name="can-i-advertise-the-exact-prefixes-as-my-virtual-network-prefixes"></a>Можно ли объявить точно такие же префиксы, как в моей виртуальной сети?

Нет. Azure будет блокировать или фильтровать объявление таких же префиксов, как префиксы адреса виртуальной сети. Тем нее менее вы можете объявить префикс, который представляет собой супермножество адресов внутри виртуальной сети. 

Например, если ваша виртуальная сеть использует адресное пространство 10.0.0.0/16, вы можете объявить 10.0.0.0/8. При этом нельзя объявлять 10.0.0.0/16 или 10.0.0.0/24.

### <a name="can-i-use-bgp-with-my-connections-between-virtual-networks"></a>Можно ли использовать BGP с подключениями между виртуальными сетями?

Да, вы можете использовать BGP и для подключений между организациями, и для подключений между виртуальными сетями.

### <a name="can-i-mix-bgp-with-non-bgp-connections-for-my-azure-vpn-gateways"></a>Можно ли сочетать подключения с BGP и подключения без BGP на VPN-шлюзах Azure?

Да, вы можете использовать подключения с BGP и без BGP на одном VPN-шлюзе Azure.

### <a name="does-azure-vpn-gateway-support-bgp-transit-routing"></a>Поддерживает ли VPN-шлюз Azure транзитную маршрутизацию BGP?

Да, транзитная маршрутизация BGP поддерживается, но с одним исключением: VPN-шлюзы Azure не будут объявлять маршруты по умолчанию для других узлов BGP. Чтобы включить транзитную маршрутизацию между несколькими VPN-шлюзами Azure, необходимо включить BGP на всех промежуточных подключениях между виртуальными сетями. Дополнительные сведения см. в статье [Обзор использования BGP с VPN-шлюзами Azure](../articles/vpn-gateway/vpn-gateway-bgp-overview.md).

### <a name="can-i-have-more-than-one-tunnel-between-an-azure-vpn-gateway-and-my-on-premises-network"></a>Можно ли создать несколько туннелей между VPN-шлюзом Azure и локальной сетью?

Да, вы можете создать несколько VPN-туннелей типа "сеть – сеть" (S2S) между VPN-шлюзом Azure и локальной сетью. Обратите внимание, что все эти туннели будут учитываться в общем числе туннелей на ваших VPN-шлюзах Azure. Кроме того, вы должны включить BGP для обоих туннелей.

Например, если вы настроите для избыточности два туннеля между VPN-шлюзом Azure и одной из локальных сетей, они будут учтены как два туннеля в общей квоте для VPN-шлюза Azure.

### <a name="can-i-have-multiple-tunnels-between-two-azure-virtual-networks-with-bgp"></a>Можно ли создать несколько туннелей между двумя виртуальными сетями Azure с использованием BGP?

Да, но хотя бы один из шлюзов виртуальной сети должен быть в конфигурации "активный — активный".

### <a name="can-i-use-bgp-for-s2s-vpn-in-an-azure-expressroute-and-s2s-vpn-coexistence-configuration"></a>Можно ли использовать BGP для S2S VPN в конфигурации сосуществования ExpressRoute и S2S VPN?

Да.

### <a name="what-should-i-add-to-my-on-premises-vpn-device-for-the-bgp-peering-session"></a>Что следует добавить в настройки локального VPN-устройства для сеансов пиринга BGP?

Добавьте маршрут узла для IP-адреса узла BGP Azure на VPN-устройство. Этот маршрут указывает на туннель S2S VPN IPsec. Например если IP-адрес VPN-узла Azure имеет значение 10.12.255.30, добавьте для адреса 10.12.255.30 узловой маршрут, в котором интерфейсом следующего прыжка будет соответствующий интерфейс туннелирования IPsec на VPN-устройстве.

### <a name="does-the-virtual-network-gateway-support-bfd-for-s2s-connections-with-bgp"></a>Поддерживает ли шлюз виртуальной сети BFD для подключений S2S с BGP?

Нет. Протокол BFD можно использовать с BGP, чтобы обнаруживать простои соседей быстрее, чем при использовании стандартных проверок активности BGP. BFD использует таймеры с точностью измерения до доли секунды, предназначенные для работы в средах LAN, но не с подключениями через общедоступный Интернет или глобальную сеть.

Для подключений через общедоступный Интернет задержка отправки или даже удаление определенных пакетов не является необычным, поэтому внедрение этих агрессивных таймеров приведет к нестабильной работе. Такая нестабильность может привести к подавлению маршрутов протоколом BGP. В качестве альтернативы можно настроить локальное устройство с таймерами с частотой проверки активности менее 60 секунд и таймером удержания на 180 секунд. Это приводит к ускоренной конвергенции.

### <a name="do-azure-vpn-gateways-initiate-bgp-peering-sessions-or-connections"></a>Инициируют ли VPN-шлюзы Azure сеансы пиринга или подключения BGP?

Шлюз будет инициировать сеансы пиринга BGP с локальными IP-адресами однорангового узла BGP, указанными в ресурсах шлюза локальной сети, использующих частные IP-адреса в VPN-шлюзах. Это не зависит от того, находятся ли локальные IP-адреса BGP в диапазоне APIPA или являются обычными частными IP-адресами. Если ваши локальные VPN-устройства используют адреса APIPA в качестве IP-адресов BGP, для инициирования подключений вам нужно настроить узел BGP.