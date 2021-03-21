---
title: Решение Azure VMware от CloudSimple — перенос локальной сети уровня 2 в частное облако
description: Описывает, как настроить VPN уровня 2 между NSX-T в частном облаке CloudSimple и локальным автономным пограничным клиентом NSX.
author: Ajayan1008
ms.author: v-hborys
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 06446b6c36e36466fe891d7327d8151603cdecd2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97899377"
---
# <a name="migrate-workloads-using-layer-2-stretched-networks"></a>Перенос рабочих нагрузок с помощью растянутых сетей уровня 2

В этом руководство вы узнаете, как использовать VPN уровня 2 (L2VPN) для переноса сети уровня 2 из локальной среды в частное облако CloudSimple. Это решение позволяет выполнять миграцию рабочих нагрузок, работающих в локальной среде VMware, в частное облако в Azure в рамках одного адресного пространства подсети без необходимости повторного создания IP-адресов рабочих нагрузок.

Перенос на основе L2VPN сетей уровня 2 может работать с сетями на основе NSX или без них в локальной среде VMware. Если у вас нет сетей на основе NSX для рабочих нагрузок в локальной среде, можно использовать изолированный шлюз пограничных служб NSX.

> [!NOTE]
> В этом руководстве рассматривается сценарий, в котором локальные и частные облачные центры обработки данных подключены через VPN типа "сеть — сеть".

## <a name="deployment-scenario"></a>Сценарий развертывания

Чтобы перенести локальную сеть с помощью L2VPN, необходимо настроить сервер L2VPN (конечный маршрутизатор NSX-T уровня 0) и клиент L2VPN (исходный автономный клиент).  

В этом сценарии развертывания ваше частное облако подключается к локальной среде через VPN-туннель типа "сеть — сеть", который позволяет локальным средам управления и подсетям vMotion обмениваться данными с подсетями управления частным облаком и vMotion. Такая структура необходима для Cross vCenter vMotion (xVC-vMotion). Маршрутизатор NSX-T уровня 0 развертывается в частном облаке как сервер L2VPN.

Автономный клиент NSX развертывается в локальной среде как клиент L2VPN и затем связывается с сервером L2VPN. Конечная точка туннеля GRE создается на каждой стороне и настроена для переноса локальной сети уровня 2 в частное облако. Данная конфигурация показана на следующем рисунке.

![Сценарий развертывания](media/l2vpn-deployment-scenario.png)

Дополнительные сведения о миграции с помощью VPN уровня 2 см. в разделе [Виртуальные частные сети](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) в документации по VMware.

## <a name="prerequisites-for-deploying-the-solution"></a>Необходимые условия для развертывания решения

Перед развертыванием и настройкой решения убедитесь в наличии следующих компонентов.

* Локальная версия vSphere — 6.7U1 или 6.5 P03 или более поздние.
* Локальная лицензия vSphere уровня Enterprise Plus (для распределенного коммутатора vSphere).
* Найдите сеть рабочей нагрузки уровня 2, которую нужно перенести в частное облако.
* Найдите сеть уровня 2 в локальной среде для развертывания клиентского устройства L2VPN.
* [Частное облако уже создано](create-private-cloud.md).
* Версия автономного пограничного устройства NSX-T совместима с версией NSX-T Manager (NSX-T 2.3.0), используемой в среде частного облака.
* В локальном vCenter создана группа магистральных портов с включенным параметром Forged Transmit.
* Общедоступный IP-адрес зарезервирован для использования в качестве IP-адреса канала исходящей связи автономного клиента NSX-T, и для преобразования между двумя адресами используется NAT 1:1.
* Переадресация DNS настроена на локальных DNS-серверах, чтобы домен az.cloudsimple.io указывал на DNS-серверы частного облака.
* Задержка при приеме и передаче составляет не более 150 мс для нормальной работы vMotion между двумя сайтами.

## <a name="limitations-and-considerations"></a>Ограничения и рекомендации

В следующей таблице перечислены поддерживаемые версии vSphere и типы сетевых адаптеров.  

| Версия vSphere | Тип исходного vSwitch | Драйвер виртуального сетевого адаптера | Тип целевого vSwitch | Поддерживается? |
------------ | ------------- | ------------ | ------------- | ------------- 
| All | DVS | All | DVS | Да |
| vSphere 6.7UI или более поздней версии, 6.5P03 или более поздней версии | DVS | VMXNET3 | N-VDS | Да |
| vSphere 6.7UI или более поздней версии, 6.5P03 или более поздней версии | DVS | E1000 | N-VDS | [Не поддерживается для VMware](https://kb.vmware.com/s/article/56991) |
| vSphere 6.7UI или 6.5P03, NSX-V или версии до NSX-T 2.2, 6.5P03 или более поздние версии | All | All | N-VDS | [Не поддерживается для VMware](https://kb.vmware.com/s/article/56991) |

Начиная с версии VMware NSX-T 2.3

* Логический коммутатор на стороне частного облака, который переносится в локальную среду через L2VPN, нельзя маршрутизировать одновременно. Перенесенный логический коммутатор не может быть подключен к логическому маршрутизатору.
* Виртуальные частные сети IPSEC на основе маршрута и L2VPN можно настроить только с помощью вызовов API.

Дополнительные сведения см. в статье [Виртуальные частные сети](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) в документации по VMware.

### <a name="sample-l2-vpn-deployment-addressing"></a>Пример адресации VPN-развертывания уровня 2

### <a name="on-premises-network-where-the-standalone-esg-l2-vpn-client-is-deployed"></a>Локальная сеть, в которой развернут автономный ESG (VPN-клиент уровня 2)

| **Item** | **Значение** |
|------------|-----------------|
| Сетевое имя | MGMT_NET_VLAN469 |
| Виртуальная локальная сеть | 469 |
| CIDR| 10.250.0.0/24 |
| IP-адрес автономного пограничного устройства | 10.250.0.111 |
| IP-адрес NAT автономного пограничного устройства | 192.227.85.167 |

### <a name="on-premises-network-to-be-stretched"></a>Локальная сеть для переноса

| **Item** | **Значение** |
|------------|-----------------|
| Виртуальная локальная сеть | 472 |
| CIDR| 10.250.3.0/24 |

### <a name="private-cloud-ip-schema-for-nsx-t-tier0-router-l2-vpn-serve"></a>Схема IP частного облака для маршрутизатора NSX-T уровня 0 (сервер VPN уровня 2)

| **Item** | **Значение** |
|------------|-----------------|
| Интерфейс замыкания на себя | 192.168.254.254/32 |
| Туннельный интерфейс | 5.5.5.1/29 |
| Логический коммутатор (перенесенный) | Stretch_LS |
| Интерфейс замыкания на себя (IP-адрес NAT) | 104.40.21.81 |

### <a name="private-cloud-network-to-be-mapped-to-the-stretched-network"></a>Сеть частного облака, которая будет сопоставлена с перенесенной сетью

| **Item** | **Значение** |
|------------|-----------------|
| Виртуальная локальная сеть | 712 |
| CIDR| 10.200.15.0/24 |

## <a name="fetch-the-logical-router-id-needed-for-l2vpn"></a>Получение идентификатора логического маршрутизатора, необходимого для L2VPN

В следующих шагах показано, как получить идентификатор логического маршрутизатора для экземпляра логического маршрутизатора DR уровня 0 для служб IPsec и L2VPN. Идентификатор логического маршрутизатора потребуется позже при реализации L2VPN.

1. Войдите в NSX-T Manager `https://*nsx-t-manager-ip-address*` и выберите **Сети**  >  **Маршрутизаторы**  >  **Provider-LR**  >  **Обзор**. Для параметра **Режим высокого уровня доступности** выберите **Активный-резервный**. Откроется всплывающее окно, где отображается пограничная виртуальная машина, на которой в данный момент активен маршрутизатор уровня 0.

    ![Выбор конфигурации "активный — резервный"](media/l2vpn-fetch01.png)

2. Выберите **Fabric**  >  **Узлы**  >  **Границы**. Запишите IP-адрес управления активной пограничной виртуальной машины (Edge VM1), указанной на предыдущем шаге.

    ![Запись IP-адреса управления](media/l2vpn-fetch02.png)

3. Откройте сеанс SSH с IP-адресом управления пограничной виртуальной машины. Выполните команду ```get logical-router``` с именем пользователя **admin** и паролем **CloudSimple 123!** .

    ![Снимок экрана, на котором показан открытый сеанс SSH.](media/l2vpn-fetch03.png)

4. Если вы не видите запись "DR-Provider-LR", выполните следующие действия.

5. Создайте два логических коммутатора с наложением. Один логический коммутатор переносится в локальную среду, где находятся перенесенные рабочие нагрузки. Другой логический коммутатор является временным. Инструкции см. в разделе [Создание логического коммутатора](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-23194F9A-416A-40EA-B9F7-346B391C3EF8.html) в документации по VMware.

    ![Создание логического коммутатора](media/l2vpn-fetch04.png)

6. Подключите временный коммутатор к маршрутизатору уровня 1 с локальным IP-адресом канала или любой непересекающейся подсетью из локальной среды или частного облака. См. раздел [Добавление нисходящего порта для логического маршрутизатора уровня 1](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-E7EA867C-604C-4224-B61D-2A8EF41CB7A6.html) в документации по VMware.

    ![Присоединение временного коммутатора](media/l2vpn-fetch05.png)

7. Снова выполните команду `get logical-router` в сеансе SSH пограничной виртуальной машины. Отображается UUID логического маршрутизатора DR-Provider-LR. Запишите идентификатор UUID, который требуется при настройке L2VPN.

    ![Снимок экрана, показывающий UUID логического маршрутизатора.](media/l2vpn-fetch06.png)

## <a name="fetch-the-logical-switch-id-needed-for-l2vpn"></a>Получение идентификатора логического коммутатора, необходимого для L2VPN

1. Войдите в NSX-T Manager (`https://nsx-t-manager-ip-address`).
2. Выберите **Сети**  >  **Коммутирование**  >  **Коммутаторы**  >  **<\Логический коммутатор\>**  >  **Обзор**.
3. Запишите идентификатор UUID переносимого логического коммутатора, который требуется при настройке L2VPN.

    ![Получение выходных данных логического маршрутизатора](media/l2vpn-fetch-switch01.png)

## <a name="routing-and-security-considerations-for-l2vpn"></a>Рекомендации по маршрутизации и безопасности для L2VPN

Чтобы установить VPN на основе маршрутов IPsec между маршрутизатором NSX-T уровня 0 и автономным пограничным клиентом NSX, интерфейс замыкания на себя маршрутизатора NSX-T уровня 0 должен иметь возможность взаимодействовать с общедоступным IP-адресом автономного локального клиента NSX по протоколу UDP 500/4500.

### <a name="allow-udp-5004500-for-ipsec"></a>Разрешение UDP 500/4500 для IPsec

1. [Создайте общедоступный IP-адрес](public-ips.md) для интерфейса замыкания на себя NSX-T уровня 0 на портале CloudSimple.

2. [Создайте таблицу брандмауэра](firewall.md) с правилами с отслеживанием состояния, разрешающими входящий трафик UDP 500/4500, и подключите таблицу брандмауэра к подсети NSX-T HostTransport.

### <a name="advertise-the-loopback-interface-ip-to-the-underlay-network"></a>Объявление IP-адреса интерфейса замыкания на себя для базовой сети

1. Создайте маршрут со значением NULL для сети интерфейса замыкания на себя. Войдите в NSX-T Manager и выберите **Сети**  >  **Маршрутизация**  >  **Маршрутизаторы**  >  **Provider-LR**  >  **Маршрутизация**  >  **Статические маршруты**. Нажмите кнопку **Добавить**. Для параметра **Сеть** введите IP-адрес интерфейса замыкания на себя. Для параметра **Следующие прыжки** нажмите кнопку **Добавить**, укажите значение NULL для следующего прыжка и оставьте значение 1 по умолчанию для параметра "Административное расстояние".

    ![Добавление статического маршрута](media/l2vpn-routing-security01.png)

2. Создайте список префиксов IP-адресов. Войдите в NSX-T Manager и выберите **Сети**  >  **Маршрутизация**  >  **Маршрутизаторы**  >  **Provider-LR**  >  **Маршрутизация**  >  **Список префиксов IP-адресов**. Нажмите кнопку **Добавить**. Введите имя списка. Для параметра **Префиксы** дважды нажмите **Добавить**. В первой строке введите "0.0.0.0/0" для параметра **Сеть** и "Запретить" для параметра **Действие**. Во второй строке выберите **Любая** для параметра **Сеть** и **Разрешить** для параметра **Действие**.
3. Присоедините список префиксов IP-адресов к BGP-соседям (TOR). Присоединение списка префиксов IP-адресов к BGP-соседу предотвращает объявление маршрута по умолчанию в BGP для коммутаторов TOR. Однако любой другой маршрут, который содержит маршрут NULL, будет объявлять IP-адрес интерфейса замыкания на себя для коммутаторов TOR.

    ![Создание списка префиксов IP-адресов](media/l2vpn-routing-security02.png)

4. Войдите в NSX-T Manager и выберите **Сети**  >  **Маршрутизация**  >  **Маршрутизаторы**  >  **Provider-LR**  >  **Маршрутизация**  >  **BGP**  >  **Соседи**. Выберите первого соседа. Щелкните **Изменить**  >  **Семейства адресов**. В семействе IPv4 измените **Исходящий фильтр** и выберите созданный список префиксов IP-адресов. Выберите команду **Сохранить**. Повторите этот шаг для второго соседа.

    ![Подключение списка префиксов IP-адресов 1](media/l2vpn-routing-security03.png) ![Подключение списка префиксов IP-адресов 2](media/l2vpn-routing-security04.png)

5. Повторно распространите статический маршрут NULL в BGP. Чтобы объявить маршрут интерфейса замыкания для базовой сети, необходимо повторно распространить статический маршрут NULL в BGP. Войдите в NSX-T Manager и выберите **Сети**  >  **Маршрутизация**  >  **Маршрутизаторы**  >  **Provider-LR**  >  **Маршрутизация**  >  **Повторное распространение маршрута**  >  **Соседи**. Выберите **Provider-LR-Route_Redistribution** и нажмите **Изменить**. Установите флажок **Статический** и нажмите кнопку **Сохранить**.

    ![Повторное распространение статического маршрута NULL в BGP](media/l2vpn-routing-security05.png)

## <a name="configure-a-route-based-vpn-on-the-nsx-t-tier0-router"></a>Настройка VPN на основе маршрутов на маршрутизаторе NSX-T уровня 0

Используйте следующий шаблон, чтобы заполнить все сведения для настройки VPN на основе маршрутов на маршрутизаторе NSX-T уровня 0. Идентификаторы UUID необходимы во всех последующих вызовах POST. IP-адреса для интерфейсов замыкания на себя и туннеля для L2VPN должны быть уникальными и не должны перекрываться с локальными сетями или сетями частного облака.

IP-адреса, выбранные для интерфейсов замыкания на себя и туннеля для L2VPN, должны быть уникальными и не должны перекрываться с локальными сетями или сетями частного облака. Сеть интерфейса замыкания на себя всегда должна иметь значение /32.

```
Loopback interface ip : 192.168.254.254/32
Tunnel interface subnet : 5.5.5.0/29
Logical-router ID : UUID of Tier0 DR logical router obtained in section "Steps to fetch Logical-Router ID needed for L2VPN"
Logical-switch ID(Stretch) : UUID of Stretch Logical Switch obtained earlier
IPSec Service ID :
IKE profile ID :
DPD profile ID :
Tunnel Profile ID :
Local-endpoint ID :
Peer end-point ID :
IPSec VPN session ID (route-based) :
L2VPN service ID :
L2VPN session ID :
Logical-Port ID :
Peer Code :
```

Во всех приведенных ниже вызовах API замените IP-адрес своим IP-адресом NSX-T Manager. Все эти вызовы API можно выполнить из клиента POSTMAN или с помощью команд `curl`.

### <a name="enable-the-ipsec-vpn-service-on-the-logical-router"></a>Включение службы VPN IPSec на логическом маршрутизаторе

```
POST   https://192.168.110.201/api/v1/vpn/ipsec/services/
{
"resource_type": "IPSecVPNService",
"description": "Manage VPN service",
"display_name": "IPSec VPN service",
"logical_router_id": "Logical-router ID",
"ike_log_level": "INFO",
"enabled": true
}
```

### <a name="create-profiles-ike"></a>Создание профилей: IKE

```
POST https://192.168.110.201/api/v1/vpn/ipsec/ike-profiles
 
{
"resource_type": "IPSecVPNIKEProfile",
"description": "IKEProfile for siteA",
"display_name": "IKEProfile siteA",
"encryption_algorithms": ["AES_128"],
"ike_version": "IKE_V2",
"digest_algorithms": ["SHA2_256"],
"sa_life_time":21600,
"dh_groups": ["GROUP14"]
}
```

### <a name="create-profiles-dpd"></a>Создание профилей: DPD

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/dpd-profiles  

{
"resource_type": "IPSecVPNDPDProfile",
"display_name": "nsx-default-dpd-profile",
"enabled": true
}
```

### <a name="create-profiles-tunnel"></a>Создание профилей: туннель

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/tunnel-profiles
 
{
"resource_type": "IPSecVPNTunnelProfile",
"display_name": "nsx-default-tunnel-profile",
"enable_perfect_forward_secrecy": true,
"encryption_algorithms": ["AES_GCM_128"],
"digest_algorithms": [],
"sa_life_time":3600,
"dh_groups": ["GROUP14"],
"encapsulation_mode": "TUNNEL_MODE",
"transform_protocol": "ESP",
"df_policy": "COPY"
}
```

### <a name="create-a-local-endpoint"></a>Создание локальной конечной точки

``` 
POST https://192.168.110.201/api/v1/vpn/ipsec/local-endpoints
 
{
"resource_type": "IPSecVPNLocalEndpoint",
"description": "Local endpoint",
"display_name": "Local endpoint",
"local_id": "<Public IP of Loopback interface>",
"ipsec_vpn_service_id": {
"target_id": "IPSec VPN service ID"},
"local_address": "<IP of Loopback interface>",
"trust_ca_ids": [],
"trust_crl_ids": []
}
```

### <a name="create-a-peer-endpoint"></a>Создание конечной точки одноранговой сети

```
POST https://192.168.110.201/api/v1/vpn/ipsec/peer-endpoints

{
"resource_type": "IPSecVPNPeerEndpoint",
"description": "Peer endpoint for site B",
"display_name": "Peer endpoint for site B",
"connection_initiation_mode": "INITIATOR",
"authentication_mode": "PSK",
"ipsec_tunnel_profile_id": "IPSec Tunnel profile ID",
"dpd_profile_id": "DPD profile ID",
"psk":"nsx",
"ike_profile_id": "IKE profile ID",
"peer_address": "<Public IP of Standalone client",
"peer_id": "<Public IP of Standalone client>"
}
```

### <a name="create-a-route-based-vpn-session"></a>Создание VPN-сеанса на основе маршрутов

```
POST :  https://192.168.110.201/api/v1/vpn/ipsec/sessions
 
{
"resource_type": "RouteBasedIPSecVPNSession",
"peer_endpoint_id": "Peer Endpoint ID",
"ipsec_vpn_service_id": "IPSec VPN service ID",
"local_endpoint_id": "Local Endpoint ID",
"enabled": true,
"tunnel_ports": [
{
"ip_subnets": [
{
"ip_addresses": [
 "5.5.5.1"
],
"prefix_length": 24
}
  ]
}
]
}
```

## <a name="configure-l2vpn-on-nsx-t-tier0-router"></a>Настройка L2VPN на маршрутизаторе NSX-T уровня 0

Укажите следующие сведения после каждого вызова POST. Идентификаторы необходимы при последующих вызовах POST.

```
L2VPN Service ID:
L2VPN Session ID:
Logical Port ID:
```

### <a name="create-the-l2vpn-service"></a>Создание службы L2VPN

Выходные данные следующей команды GET будут пустыми, так как конфигурация еще не завершена.

```
GET : https://192.168.110.201/api/v1/vpn/l2vpn/services
```

Для следующей команды POST идентификатор логического маршрутизатора — это UUID логического маршрутизатора DR уровня 0, полученный ранее.

```
POST : https://192.168.110.201/api/v1/vpn/l2vpn/services

{
"logical_router_id": "Logical Router ID",
"enable_full_mesh" : true
}
```

### <a name="create-the-l2vpn-session"></a>Создание сеанса L2VPN

Для следующей команды POST идентификатор службы L2VPN — это только что полученный идентификатор, а идентификатор сеанса VPN-подключения IPsec — это идентификатор, полученный в предыдущем разделе.

```    
POST: https://192.168.110.201/api/v1/vpn/l2vpn/sessions

{
"l2vpn_service_id" : "L2VPN service ID",
"transport_tunnels" : [
    {
    "target_id" : "IPSec VPN session ID"
    }]
}
```

Эти вызовы создают конечную точку туннеля GRE. Для проверки состояния выполните следующую команду.

```
edge-2> get tunnel-port
Tunnel      : 44648dae-8566-5bc9-a065-b1c4e5c3e03f
IFUID       : 328
LOCAL       : 169.254.64.1
REMOTE      : 169.254.64.2
ENCAP       : GRE

Tunnel      : cf950ca1-5cf8-5438-9b1a-d2c8c8e7229b
IFUID       : 318
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.152
ENCAP       : GENEVE

Tunnel      : 63639321-87c5-529e-8a61-92c1939799b2
IFUID       : 304
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.156
ENCAP       : GENEVE
```

### <a name="create-logical-port-with-the-tunnel-id-specified"></a>Создание логического порта с указанным идентификатором туннеля

```
    POST https://192.168.110.201/api/v1/logical-ports/

{
"resource_type": "LogicalPort",
"display_name": "Extend logicalSwitch, port for service",
"logical_switch_id": "Logical switch ID",
"admin_state" : "UP",
"attachment": {
"attachment_type":"L2VPN_SESSION",
"id":"L2VPN session ID",
"context" : {
"resource_type" : "L2VpnAttachmentContext",
    "tunnel_id" : 10
}
    }
        }
```

## <a name="obtain-the-peer-code-for-l2vpn-on-the-nsx-t-side"></a>Получение кода однорангового узла для L2VPN на стороне NSX-T

Получение кода однорангового узла конечной точки NSX-T. При настройке удаленной конечной точки требуется код однорангового узла. <session-id> для L2VPN можно получить в предыдущем разделе. Дополнительные сведения см. в [руководстве по API NSX-T 2.3](https://www.vmware.com/support/nsxt/doc/nsxt_23_api.html).

```
GET https://192.168.110.201/api/v1/vpn/l2vpn/sessions/<session-id>/peer-codes
```

## <a name="deploy-the-nsx-t-standalone-client-on-premises"></a>Развертывание автономного клиента NSX-T (локально)

Перед развертыванием убедитесь, что локальные правила брандмауэра разрешают входящий и исходящий трафик UDP 500/4500 для обмена данными по общедоступному IP-адресу CloudSimple, который был зарезервирован ранее для интерфейса замыкания на себя маршрутизатора NSX-T уровня 0. 

1. [Скачайте OVF автономного пограничного клиента NSX](https://my.vmware.com/group/vmware/details?productId=673&rPId=33945&downloadGroup=NSX-T-230) и извлеките файлы из скачанного пакета в папку.

    ![Загрузка автономного пограничного клиента NSX](media/l2vpn-deploy-client01.png)

2. Перейдите в папку со всеми извлеченными файлами. Выберите все vmdk (NSX-l2t-client-large.mf и NSX-l2t-client-large.ovf для крупных устройств или NSX-l2t-client-Xlarge.mf и NSX-l2t-client-Xlarge.ovf для очень крупных устройств). Щелкните **Далее**.

    ![Выберите шаблон ](media/l2vpn-deploy-client02.png) ![ снимок экрана, на котором отображаются выбранные файлы VMDK.](media/l2vpn-deploy-client03.png)

3. Введите имя автономного клиента NSX-T и нажмите **Далее**.

    ![Ввод имени шаблона](media/l2vpn-deploy-client04.png)

4. Щелкните **Далее**, чтобы перейти к параметрам хранилища данных. Выберите соответствующее хранилище данных для автономного клиента NSX-T и нажмите **Далее**.

    ![Выбор хранилища данных](media/l2vpn-deploy-client06.png)

5. Выберите правильные группы портов для магистрали (магистральная группа портов), общедоступного канала (группа портов исходящей связи) и интерфейса высокой доступности (группа портов исходящей связи) для автономного клиента NSX-T. Щелкните **Далее**.

    ![Выбор групп портов](media/l2vpn-deploy-client07.png)

6. Укажите следующие сведения на экране **Настройка шаблона** и нажмите кнопку **Далее**.

    Разверните L2T.

    * **Адрес однорангового узла**. Введите IP-адрес, зарезервированный на портале Azure CloudSimple, для интерфейса замыкания на себя NSX-T уровня 0.
    * **Код однорангового узла**. Вставьте код однорангового узла, полученный на последнем шаге развертывания сервера L2VPN.
    * **Дочерние интерфейсы виртуальной ЛС (идентификатор туннеля)** . Введите идентификатор виртуальной ЛС для переноса. В скобках () введите идентификатор туннеля, который был настроен ранее.

    Разверните интерфейс исходящей связи.

    * **IP-адрес DNS**. Введите локальный IP-адрес DNS.
    * **Шлюз по умолчанию**.  Введите шлюз по умолчанию виртуальной ЛС, который будет использоваться в качестве шлюза по умолчанию для этого клиента.
    * **IP-адрес**. Введите IP-адрес канала исходящей связи автономного клиента.
    * **Длина префикса**. Введите длину префикса исходящей виртуальной ЛС/подсети.
    * **Пароль CLI admin/enable/root**. Задайте пароль для учетной записи admin /enable /root.

      ![Настройка шаблона](media/l2vpn-deploy-client08.png)
       ![Настройка шаблона — дополнительно](media/l2vpn-deploy-client09.png)

7. Проверьте параметры и нажмите кнопку **Готово**.

    ![Полная конфигурация](media/l2vpn-deploy-client10.png)

## <a name="configure-an-on-premises-sink-port"></a>Настройка порта локального приемника

Если на одном из сайтов VPN не развернут NSX, можно настроить VPN уровня 2, развернув автономный пограничный клиент NSX на этом сайте. Автономный пограничный клиент NSX развертывается с помощью OVF-файла на узле, который не управляется NSX. Это позволяет развернуть устройство шлюза служб пограничного клиента NSX для работы в качестве VPN-клиента уровня 2.

Если автономный пограничный магистральный клиент vNIC подключен к распределенному коммутатору vSphere, то для работы VPN уровня 2 требуется либо неизбирательный режим, либо порт приемника. Использование неизбирательного режима может привести к дублированию проверки связи и ответов. По этой причине используйте режим порта приемника в конфигурации автономного пограничного клиента NSX для VPN уровня 2. См. раздел [Настройка порта приемника](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.admin.doc/GUID-3CDA4346-E692-4592-8796-ACBEEC87C161.html) в документации по VMware.

## <a name="ipsec-vpn-and-l2vpn-verification"></a>Проверка IPsec VPN и L2VPN

Используйте следующие команды для проверки сеансов IPsec и L2VPN из автономного пограничного клиента NSX-T.

```
nsx-l2t-edge> show service ipsec
-----------------------------------------------------------------------
vShield Edge IPSec Service Status:
IPSec Server is running.
AESNI is enabled.
Total Sites: 1, 1 UP, 0 Down
Total Tunnels: 1, 1 UP, 0 Down
----------------------------------
Site:  10.250.0.111_0.0.0.0/0-104.40.21.81_0.0.0.0/0
Channel: PeerIp: 104.40.21.81    LocalIP: 10.250.0.111  Version: IKEv2  Status: UP
Tunnel: PeerSubnet: 0.0.0.0/0    LocalSubnet: 0.0.0.0/0   Status: UP
----------------------------------
```

```
nsx-l2t-edge> show service l2vpn
L2 VPN is running
----------------------------------------
L2 VPN type: Client/Spoke

SITENAME                       IPSECSTATUS          VTI                  GRE
1ecb00fb-a538-4740-b788-c9049e8cb6c6 UP                   vti-100              l2t-1
```

Используйте следующие команды для проверки сеансов IPsec и L2VPN из маршрутизатора NSX-T уровня 0.

```
edge-2> get ipsecvpn session
Total Number of Sessions: 1

IKE Session ID : 3
UUID           : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Type           : Route

Local IP       : 192.168.254.254      Peer IP        : 192.227.85.167
Local ID       : 104.40.21.81         Peer ID        : 192.227.85.167
Session Status : Up

Policy Rules
    VTI UUID       : 4bf96e3b-e50b-49cc-a16e-43a6390e3d53
    ToRule ID      : 560874406            FromRule ID    : 2708358054
    Local Subnet   : 0.0.0.0/0            Peer Subnet    : 0.0.0.0/0
    Tunnel Status  : Up
```

```
edge-2> get l2vpn session
Session       : f7147137-5dd0-47fe-9e53-fdc2b687b160
Tunnel        : b026095b-98c8-5932-96ff-dda922ffe316
IPSEC Session : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Status        : UP
```

Используйте следующие команды, чтобы проверить порт приемника на узле ESXi, где находится виртуальная машина автономного клиента NSX-T в локальной среде.

```
 [root@esxi02:~] esxcfg-vswitch -l |grep NSX
  53                  1           NSXT-client-large.eth2
  225                1           NSXT-client-large.eth1
  52                  1           NSXT-client-large.eth0
```

```
[root@esxi02:~] net-dvs -l | grep "port\ [0-9]\|SINK\|com.vmware.common.alias"
                com.vmware.common.alias = csmlab2DS ,   propType = CONFIG
        port 24:
        port 25:
        port 26:
        port 27:
        port 13:
        port 19:
        port 169:
        port 54:
        port 110:
        port 6:
        port 107:
        port 4:
        port 199:
        port 168:
        port 201:
        port 0:
        port 49:
        port 53:
        port 225:
                com.vmware.etherswitch.port.extraEthFRP =   SINK
        port 52:
```
