---
title: Azure ExpressRoute. Примеры конфигурации маршрутизатора
description: Используйте эти примеры конфигурации интерфейса и маршрутизации для маршрутизаторов серии Cisco IOS-XE и Juniper MX в качестве примеров для работы с Azure ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 03/26/2020
ms.author: duau
ms.openlocfilehash: 3bc850f02884ae0547c2ecf56a46a57a4e66a752
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89397412"
---
# <a name="router-configuration-samples-to-set-up-and-manage-routing"></a>Примеры конфигурации маршрутизатора для настройки и управления маршрутизацией
Эта страница содержит примеры конфигурации интерфейса и маршрутизации для маршрутизаторов серий Cisco IOS-XE и Juniper MX при работе с Azure ExpressRoute.

> [!IMPORTANT]
> Примеры на этой странице предназначены исключительно для справки. Для получения подходящей конфигурации, которая удовлетворяет вашим потребностям, необходимо провести совместную работу со специалистами по продажам или техническими специалистами поставщика и вашими сетевыми специалистами. Корпорация Майкрософт не предоставляет поддержку по вопросам, связанным с конфигурациями, перечисленными на этой странице. Для получения поддержки обратитесь к поставщику устройства.
> 
> 

## <a name="mtu-and-tcp-mss-settings-on-router-interfaces"></a>Параметры MTU и TCP MSS в интерфейсах маршрутизатора
Значение MTU (максимальный передаваемый блок данных) для интерфейса ExpressRoute составляет 1500, что является типичным значением MTU по умолчанию для интерфейса Ethernet на маршрутизаторе. Если маршрутизатор не имеет другого значения MTU по умолчанию, то и значение в интерфейсе маршрутизатора можно не указывать.

В отличие от VPN-шлюза Azure, MSS (максимальный размер сегмента) TCP для канала ExpressRoute не нужно указывать.

Приведенные в этой статье примеры конфигурации маршрутизатора применяются ко всем пирингам. Для получения дополнительных сведений о маршрутизации обратитесь к статьям [Сеансы пиринга ExpressRoute](expressroute-circuit-peerings.md) и [Требования ExpressRoute к маршрутизации](expressroute-routing.md).


## <a name="cisco-ios-xe-based-routers"></a>Маршрутизаторы на основе Cisco IOS-XE
Примеры из этого раздела можно применить к любому маршрутизатору, работающему под управлением ОС семейства IOS-XE.

### <a name="configure-interfaces-and-subinterfaces"></a>Настройка интерфейсов и подчиненных интерфейсов
Вам потребуется один подчиненный интерфейс для каждого пиринга в каждом маршрутизаторе при подключении к Майкрософт. Подчиненный интерфейс можно идентифицировать с помощью идентификатора виртуальной локальной сети или с помощью накопленной пары идентификаторов виртуальных сетей и IP-адреса.

**Определение интерфейса Dot1Q**

В этом примере приведено определение подчиненного интерфейса для подчиненного интерфейса с одним идентификатором виртуальной локальной сети. Идентификатор виртуальной локальной сети уникален в пределах одного пиринга. Последний октет адреса IPv4 всегда будет нечетным числом.

```console
interface GigabitEthernet<Interface_Number>.<Number>
 encapsulation dot1Q <VLAN_ID>
 ip address <IPv4_Address><Subnet_Mask>
```

**Определение интерфейса QinQ**

В этом примере приведено определение подчиненного интерфейса для подчиненного интерфейса с двумя идентификаторами виртуальной локальной сети. Идентификатор внешней виртуальной локальной сети (s-тег) остается постоянным для всех пирингов, если используется. Идентификатор внешней виртуальной локальной сети (c-тег) уникален для каждого пиринга. Последний октет адреса IPv4 всегда будет нечетным числом.

```console
interface GigabitEthernet<Interface_Number>.<Number>
 encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
 ip address <IPv4_Address><Subnet_Mask>
```

### <a name="set-up-ebgp-sessions"></a>Настройка сеансов eBGP
Для каждого пиринга необходимо настроить сеанс BGP с Майкрософт. Настройте сеанс BGP, используя следующий пример. Если IPv4-адрес, используемый для подчиненного интерфейса, a.b.c.d, то IP-адрес соседа BGP (Майкрософт) будет таким: a.b.c.d+1. Последний октет IPv4-адреса соседа BGP всегда будет четным числом.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
 neighbor <IP#2_used_by_Azure> activate
 exit-address-family
!
```

### <a name="set-up-prefixes-to-be-advertised-over-the-bgp-session"></a>Настройка префиксов, объявляемых в сеансе BGP
Настройте маршрутизатор на объявление выбранных префиксов в Майкрософт, используя следующий пример.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  network <Prefix_to_be_advertised> mask <Subnet_mask>
  neighbor <IP#2_used_by_Azure> activate
 exit-address-family
!
```

### <a name="route-maps"></a>Карты маршрутизации
Для фильтрации префиксов, распространяемых в сети, используйте карты маршрутизации и списки префиксов. См. следующий пример и убедитесь, что настроены соответствующие списки префиксов.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  network <Prefix_to_be_advertised> mask <Subnet_mask>
  neighbor <IP#2_used_by_Azure> activate
  neighbor <IP#2_used_by_Azure> route-map <MS_Prefixes_Inbound> in
 exit-address-family
!
route-map <MS_Prefixes_Inbound> permit 10
 match ip address prefix-list <MS_Prefixes>
!
```

### <a name="configure-bfd"></a>Настройка BFD

BFD необходимо настроить в двух местах: один на уровне интерфейса, а другой — на уровне BGP. Ниже приведен пример для интерфейса QinQ. 

```console
interface GigabitEthernet<Interface_Number>.<Number>
 bfd interval 300 min_rx 300 multiplier 3
 encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
 ip address <IPv4_Address><Subnet_Mask>

router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  neighbor <IP#2_used_by_Azure> activate
  neighbor <IP#2_used_by_Azure> fall-over bfd
 exit-address-family
!
```


## <a name="juniper-mx-series-routers"></a>Маршрутизаторы серии Juniper MX
Примеры из этого раздела можно применять ко всем маршрутизаторам серии Juniper MX.

### <a name="configure-interfaces-and-subinterfaces"></a>Настройка интерфейсов и подчиненных интерфейсов

**Определение интерфейса Dot1Q**

В этом примере приведено определение подчиненного интерфейса для подчиненного интерфейса с одним идентификатором виртуальной локальной сети. Идентификатор виртуальной локальной сети уникален в пределах одного пиринга. Последний октет адреса IPv4 всегда будет нечетным числом.

```console
    interfaces {
        vlan-tagging;
        <Interface_Number> {
            unit <Number> {
                vlan-id <VLAN_ID>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }
            }
        }
    }
```


**Определение интерфейса QinQ**

В этом примере приведено определение подчиненного интерфейса для подчиненного интерфейса с двумя идентификаторами виртуальной локальной сети. Идентификатор внешней виртуальной локальной сети (s-тег) остается постоянным для всех пирингов, если используется. Идентификатор внешней виртуальной локальной сети (c-тег) уникален для каждого пиринга. Последний октет адреса IPv4 всегда будет нечетным числом.

```console
    interfaces {
        <Interface_Number> {
            flexible-vlan-tagging;
            unit <Number> {
                vlan-tags outer <S-tag> inner <C-tag>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }                           
            }                               
        }                                   
    }                           
```

### <a name="set-up-ebgp-sessions"></a>Настройка сеансов eBGP
Для каждого пиринга необходимо настроить сеанс BGP с Майкрософт. Настройте сеанс BGP, используя следующий пример. Если IPv4-адрес, используемый для подчиненного интерфейса, a.b.c.d, то IP-адрес соседа BGP (Майкрософт) будет таким: a.b.c.d+1. Последний октет IPv4-адреса соседа BGP всегда будет четным числом.

```console
    routing-options {
        autonomous-system <Customer_ASN>;
    }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="set-up-prefixes-to-be-advertised-over-the-bgp-session"></a>Настройка префиксов, объявляемых в сеансе BGP
Настройте маршрутизатор на объявление выбранных префиксов в Майкрософт, используя следующий пример.

```console
    policy-options {
        policy-statement <Policy_Name> {
            term 1 {
                from protocol OSPF;
                route-filter; 
                <Prefix_to_be_advertised/Subnet_Mask> exact;
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>;
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="route-policies"></a>Политики маршрутизации
Для фильтрации префиксов, распространяемых в сети, можно использовать карты маршрутизации и списки префиксов. См. следующий пример и убедитесь, что настроены соответствующие списки префиксов.

```console
    policy-options {
        prefix-list MS_Prefixes {
            <IP_Prefix_1/Subnet_Mask>;
            <IP_Prefix_2/Subnet_Mask>;
        }
        policy-statement <MS_Prefixes_Inbound> {
            term 1 {
                from {
                    prefix-list MS_Prefixes;
                }
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>;
                import <MS_Prefixes_Inbound>;
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="configure-bfd"></a>Настройка BFD
Настройте BFD в разделе протокола BGP.

```console
    protocols {
        bgp { 
            group <Group_Name> { 
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
                bfd-liveness-detection {
                       minimum-interval 3000;
                       multiplier 3;
                }
            }                               
        }                                   
    }
```

### <a name="configure-macsec"></a>Настройка MACSec
Для конфигурации MACSec ключ связи подключения (CAK) и имя ключа связи подключения (CKN) должны соответствовать значениям, настроенным с помощью команд PowerShell.

```console
    security {
        macsec {
            connectivity-association <Connectivity_Association_Name> {
                cipher-suite gcm-aes-xpn-128;
                security-mode static-cak;
                pre-shared-key {
                    ckn <Connectivity_Association_Key_Name>;
                    cak <Connectivity_Association_Key>; ## SECRET-DATA
                }
            }
            interfaces {
                <Interface_Number> {
                    connectivity-association <Connectivity_Association_Name>;
                }
            }
        }
    }
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения см. в разделе [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).



