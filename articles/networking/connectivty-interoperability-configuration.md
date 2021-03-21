---
title: Взаимодействие между функциями подключения к серверной части в Azure. Анализ на уровне конфигурации | Документация Майкрософт
description: В этой статье подробно описана конфигурация тестовой системы, с помощью которой можно выполнять анализ взаимодействия между ExpressRoute, VPN типа "сеть — сеть" и пирингом между виртуальными сетями в Azure.
documentationcenter: na
services: networking
author: rambk
manager: tracsman
ms.service: virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/18/2018
ms.author: rambala
ms.openlocfilehash: 50fd5641c61d08939eca1f2cbafb1d077254e37d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98233276"
---
# <a name="interoperability-in-azure-back-end-connectivity-features-test-configuration-details"></a>Взаимодействие между функциями подключения к серверной части в Azure. Сведения о тестовой конфигурации

В этой статье подробно описана конфигурация [тестовой системы][Setup]. Эта тестовая система помогает выполнять анализ взаимодействия сетевых служб Azure на уровне управления и на уровне данных.

## <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>Подключение к периферийной виртуальной сети с использованием пиринга

На следующем рисунке изображена схема пиринга виртуальной сети Azure для периферийных виртуальных сетей. Чтобы узнать, как настроить пиринг между двумя виртуальными сетями, см. статью [Создание, изменение и удаление пиринга в виртуальной сети][VNet-Config]. Если требуется, чтобы периферийная виртуальная сеть использовала шлюзы, подключенные к центральной виртуальной сети, установите флажок **Использовать удаленные шлюзы**.

[![1]][1]

Ниже представлена страница сведений о пиринге для центральной виртуальной сети. Если вы хотите, чтобы виртуальная сеть концентратора позволяла периферийной виртуальной сети использовать шлюзы концентратора, установите флажок **Разрешить транзит шлюза**.

[![2]][2]

## <a name="branch-vnet-connectivity-by-using-a-site-to-site-vpn"></a>Подключение к виртуальной сети филиала с помощью VPN типа "сеть — сеть"

Подключение между центральной виртуальной сетью и виртуальными сетями филиалов с использованием VPN типа "сеть — сеть" настраивается с помощью VPN-шлюзов в службе "VPN-шлюз Azure". По умолчанию VPN-шлюзы и шлюзы Azure ExpressRoute используют ASN (номер автономной системы) **65515**. Вы можете изменить значение ASN в службе "VPN-шлюз Azure". В тестовой конфигурации значение ASN для VPN-шлюза виртуальной сети филиала изменено на **65516**, чтобы поддерживать маршрутизацию eBGP между центральной виртуальной сетью и виртуальными сетями филиалов.


[![3-5]][3]


## <a name="on-premises-location-1-connectivity-by-using-expressroute-and-a-site-to-site-vpn"></a>Подключение к локальному расположению 1 через ExpressRoute и VPN типа "сеть — сеть"

### <a name="expressroute-1-configuration-details"></a>Сведения о конфигурации ExpressRoute 1

На следующем изображении показана конфигурация канала ExpressRoute в регионе Azure 1 для подключения к граничным пользовательским маршрутизаторам (CE) локального расположения 1.

[![4]][4]

На следующем изображении показана конфигурация подключения между каналом ExpressRoute1 и центральной виртуальной сетью.

[![5.0]][5]

Ниже перечислены основные настройки маршрутизатора CE для подключения через частный пиринг ExpressRoute. (Маршрутизаторы Cisco ASR1000 используются в настройках тестирования в качестве маршрутизаторов CE). Если VPN-подключение типа "сеть — сеть" и каналы ExpressRoute настроены параллельно для подключения локальной сети к Azure, Azure назначает по умолчанию канал ExpressRoute. Чтобы избежать асимметричной маршрутизации, в локальной сети подключения ExpressRoute также должны иметь более высокий приоритет, чем VPN-подключение типа "сеть — сеть". Следующая конфигурация устанавливает приоритеты с помощью атрибута BGP **local-preference**:

```config
interface TenGigabitEthernet0/0/0.300
 description Customer 30 private peering to Azure
 encapsulation dot1Q 30 second-dot1q 300
 ip vrf forwarding 30
 ip address 192.168.30.17 255.255.255.252
!
interface TenGigabitEthernet1/0/0.30
 description Customer 30 to south bound LAN switch
 encapsulation dot1Q 30
 ip vrf forwarding 30
 ip address 192.168.30.0 255.255.255.254
 ip ospf network point-to-point
!
router ospf 30 vrf 30
 router-id 10.2.30.253
 redistribute bgp 65021 subnets route-map BGP2OSPF
 network 192.168.30.0 0.0.0.1 area 0.0.0.0
default-information originate always
 default-metric 10
!
router bgp 65021
 !
 address-family ipv4 vrf 30
  network 10.2.30.0 mask 255.255.255.128
  neighbor 192.168.30.18 remote-as 12076
  neighbor 192.168.30.18 activate
  neighbor 192.168.30.18 next-hop-self
  neighbor 192.168.30.18 soft-reconfiguration inbound
  neighbor 192.168.30.18 route-map prefer-ER-over-VPN in
  neighbor 192.168.30.18 prefix-list Cust30_to_Private out
 exit-address-family
!
route-map prefer-ER-over-VPN permit 10
 set local-preference 200
!
ip prefix-list Cust30_to_Private seq 10 permit 10.2.30.0/25
!
```

### <a name="site-to-site-vpn-configuration-details"></a>Сведения о конфигурации VPN типа "сеть — сеть"

Ниже перечислены основные настройки маршрутизатора CE для подключения через VPN типа "сеть — сеть".

```config
crypto ikev2 proposal Cust30-azure-proposal
 encryption aes-cbc-256 aes-cbc-128 3des
 integrity sha1
 group 2
!
crypto ikev2 policy Cust30-azure-policy
 match address local 66.198.12.106
 proposal Cust30-azure-proposal
!
crypto ikev2 keyring Cust30-azure-keyring
 peer azure
  address 52.168.162.84
  pre-shared-key local IamSecure123
  pre-shared-key remote IamSecure123
!
crypto ikev2 profile Cust30-azure-profile
 match identity remote address 52.168.162.84 255.255.255.255
 identity local address 66.198.12.106
 authentication local pre-share
 authentication remote pre-share
 keyring local Cust30-azure-keyring
!
crypto ipsec transform-set Cust30-azure-ipsec-proposal-set esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile Cust30-azure-ipsec-profile
 set transform-set Cust30-azure-ipsec-proposal-set
 set ikev2-profile Cust30-azure-profile
!
interface Loopback30
 ip address 66.198.12.106 255.255.255.255
!
interface Tunnel30
 ip vrf forwarding 30
 ip address 10.2.30.125 255.255.255.255
 tunnel source Loopback30
 tunnel mode ipsec ipv4
 tunnel destination 52.168.162.84
 tunnel protection ipsec profile Cust30-azure-ipsec-profile
!
router bgp 65021
 !
 address-family ipv4 vrf 30
  network 10.2.30.0 mask 255.255.255.128
  neighbor 10.10.30.254 remote-as 65515
  neighbor 10.10.30.254 ebgp-multihop 5
  neighbor 10.10.30.254 update-source Tunnel30
  neighbor 10.10.30.254 activate
  neighbor 10.10.30.254 soft-reconfiguration inbound
 exit-address-family
!
ip route vrf 30 10.10.30.254 255.255.255.255 Tunnel30
```

## <a name="on-premises-location-2-connectivity-by-using-expressroute"></a>Подключение к локальному расположению 2 через ExpressRoute

Второй канал ExpressRoute, расположенный близко к локальному расположению 2, подключает локальное расположение 2 к центральной виртуальной сети. На следующем изображении показана конфигурация второго канала ExpressRoute.

[![6]][6]

На следующем изображении показана конфигурация подключения между вторым каналом ExpressRoute и центральной виртуальной сетью.

[![7]][7]

Канал ExpressRoute1 соединяет центральную виртуальную сеть и локальное расположение 1 с удаленной виртуальной сетью в другом регионе Azure.

[![8]][8]

## <a name="expressroute-and-site-to-site-vpn-connectivity-in-tandem"></a>Параллельное использование подключений ExpressRoute и VPN типа "сеть — сеть"

###  <a name="site-to-site-vpn-over-expressroute"></a>VPN типа "сеть — сеть" через ExpressRoute

VPN типа "сеть — сеть" можно настроить с использованием пиринга Microsoft ExpressRoute, чтобы организовать закрытый обмен данными между локальной сетью и виртуальными сетями Azure. Такая конфигурация гарантирует конфиденциальность, подлинность и целостность при обмене данными. Обмен данными также защищается от повторов. Дополнительные сведения о настройке VPN IPSec типа "сеть — сеть" в туннельном режиме с использованием пиринга Microsoft ExpressRoute см. [в этой статье][S2S-Over-ExR]. 

Основным ограничением настройки VPN типа "сеть — сеть" с использованием пиринга Майкрософт является пропускная способность. Пропускная способность туннеля IPsec ограничена емкостью VPN-шлюза. Пропускная способность VPN-шлюза всегда ниже пропускной способности ExpressRoute. В такой ситуации можно оптимизировать использование пропускной способности ExpressRoute, используя туннель IPSec для трафика, требующего высокого уровня безопасности, и частный пиринг для всего остального трафика.

### <a name="site-to-site-vpn-as-a-secure-failover-path-for-expressroute"></a>VPN типа "сеть — сеть" в качестве защищенного пути отработки отказа для ExpressRoute

ExpressRoute предоставляется в виде пары каналов, что обеспечивает избыточность и высокий уровень доступности. Вы можете настроить географически избыточное подключение ExpressRoute в разных регионах Azure. Как и в нашей тестовой конфигурации, вы можете создать путь отработки отказа для подключения ExpressRoute в пределах определенного региона Azure с помощью VPN типа "сеть — сеть". Если одни и те же префиксы объявляются через ExpressRoute и VPN типа "сеть — сеть", Azure отдает предпочтение ExpressRoute. Чтобы избежать асимметричной маршрутизации между ExpressRoute и VPN типа "сеть — сеть", в локальной сети следует также настроить конфигурацию ExpressRoute, прежде чем использовать VPN типа "сеть — сеть".

Дополнительные сведения о настройке параллельных подключений ExpressRoute и VPN типа "сеть — сеть" см. в [этой статье][ExR-S2S-CoEx].

## <a name="extend-back-end-connectivity-to-spoke-vnets-and-branch-locations"></a>Предоставление периферийным виртуальным сетям и расположениям филиалов доступа к подключениям серверной части

### <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>Подключение к периферийной виртуальной сети с использованием пиринга

Звездообразная архитектура виртуальных сетей нашла широкое применение. Центральная виртуальная сеть в Azure выполняет роль центральной точки для подключения между периферийными виртуальными сетями и локальной сетью. Периферийными называются виртуальные сети, которые устанавливают пиринг с центральной сетью и применяются для изоляции рабочих нагрузок. Трафик передается между локальным центром обработки данных и центральной сетью через подключение ExpressRoute и (или) VPN. Дополнительные сведения об этой архитектуре см. в статье [Реализация звездообразной топологии сети в Azure][Hub-n-Spoke].

Пиринг виртуальных сетей в пределах региона позволяет периферийным виртуальным сетям использовать шлюзы центральной виртуальной сети (шлюз VPN или ExpressRoute) для подключения к удаленным сетям.

### <a name="branch-vnet-connectivity-by-using-site-to-site-vpn"></a>Подключение к виртуальной сети филиала с помощью VPN типа "сеть — сеть"

Через центральную виртуальную сеть можно подключить друг к другу виртуальные сети филиалов из разных регионов и локальные сети. Обычно в Azure решением для этой конфигурации является VPN типа "сеть — сеть". Альтернативой может быть использование в центральной сети виртуального сетевого модуля (NVA) для маршрутизации.

Дополнительные сведения см. в статьях [Сведения о VPN-шлюзе][VPN] и [Развертывание высокодоступных виртуальных сетевых модулей][Deploy-NVA].

## <a name="next-steps"></a>Дальнейшие действия

Изучите [сведения об анализе уровня управления ][Control-Analysis] для тестовой системы и представления разных виртуальных сетей или виртуальных локальных сетей в топологии.

Сведения об анализе на уровне данных для конфигурации теста и представлениях функций мониторинга сети Azure см. в [этой статье][Data-Analysis].

В статье [Вопросы и ответы по ExpressRoute][ExR-FAQ] представлены следующие сведения:
-   количество каналов ExpressRoute, которые можно подключить к шлюзу ExpressRoute;
-   количество шлюзов ExpressRoute, которые можно подключить к каналу ExpressRoute;
-   дополнительные ограничения масштаба для ExpressRoute.


<!--Image References-->
[1]: ./media/backend-interoperability/SpokeVNet_peering.png "Пиринг для периферийной виртуальной сети"
[2]: ./media/backend-interoperability/HubVNet-peering.png "Пиринг для центральной виртуальной сети"
[3]: ./media/backend-interoperability/BranchVNet-VPNGW.png "Конфигурация VPN-шлюза для виртуальной сети филиала"
[4]: ./media/backend-interoperability/ExR1.png "Конфигурация ExpressRoute 1"
[5]: ./media/backend-interoperability/ExR1-Hub-Connection.png "Конфигурация подключения ExpressRoute 1 к шлюзу ExpressRoute центральной виртуальной сети"
[6]: ./media/backend-interoperability/ExR2.png "Конфигурация ExpressRoute 2"
[7]: ./media/backend-interoperability/ExR2-Hub-Connection.png "Конфигурация подключения ExpressRoute 2 к шлюзу ExpressRoute центральной виртуальной сети"
[8]: ./media/backend-interoperability/ExR2-Remote-Connection.png "Конфигурация подключения ExpressRoute 2 к шлюзу ExpressRoute удаленной виртуальной сети"

<!--Link References-->
[Setup]: ./connectivty-interoperability-preface.md
[ExpressRoute]: ../expressroute/expressroute-introduction.md
[VPN]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[VNet]: ../virtual-network/tutorial-connect-virtual-networks-portal.md
[Control-Analysis]: ./connectivty-interoperability-control-plane.md
[Data-Analysis]: ./connectivty-interoperability-data-plane.md
[ExR-FAQ]: ../expressroute/expressroute-faqs.md
[S2S-Over-ExR]: ../expressroute/site-to-site-vpn-over-microsoft-peering.md
[ExR-S2S-CoEx]: ../expressroute/expressroute-howto-coexist-resource-manager.md
[Hub-n-Spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
[Deploy-NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[VNet-Config]: ../virtual-network/virtual-network-manage-peering.md