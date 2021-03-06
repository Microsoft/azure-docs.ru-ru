---
title: 'Настройка шифрования ExpressRoute: IPsec через ExpressRoute для виртуальной глобальной сети Azure'
description: Узнайте, как использовать виртуальную сеть Azure для создания VPN-подключения типа "сеть — сеть" через частный пиринг ExpressRoute.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: b8dde3ed76587e2343edaec8626287853ec6ef9b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96487513"
---
# <a name="expressroute-encryption-ipsec-over-expressroute-for-virtual-wan"></a>Шифрование ExpressRoute: IPsec через ExpressRoute для виртуальной глобальной сети

В этой статье показано, как использовать виртуальную сеть Azure для установления VPN-подключения IPsec/IKE из локальной сети к Azure через частный пиринг канала Azure ExpressRoute. Этот метод может обеспечить зашифрованное переработку между локальными сетями и виртуальными сетями Azure через ExpressRoute, не открывая общедоступный Интернет или не используя общедоступные IP-адреса.

## <a name="topology-and-routing"></a>Топология и маршрутизация

На следующей схеме показан пример VPN-подключения через частный пиринг ExpressRoute:

:::image type="content" source="./media/vpn-over-expressroute/vwan-vpn-over-er.png" alt-text="VPN через ExpressRoute":::

На схеме показана сеть в локальной сети, подключенная к VPN-шлюзу центра Azure через частный пиринг ExpressRoute. Установка подключения проста:

1. Установите подключение ExpressRoute к каналу ExpressRoute и частному пирингу.
2. Установите VPN-подключение, как описано в этой статье.

Важным аспектом этой конфигурации является маршрутизация между локальными сетями и Azure по путям ExpressRoute и VPN.

### <a name="traffic-from-on-premises-networks-to-azure"></a>Трафик из локальных сетей в Azure

Для трафика из локальных сетей в Azure префиксы Azure (включая виртуальный концентратор и все периферийные виртуальные сети, подключенные к концентратору) объявляются через частный пиринг пиринга ExpressRoute и BGP VPN. В результате мы получаем два сетевых маршрута (пути) к Azure из локальных сетей:

- Один через путь, защищенный с помощью IPsec
- Один непосредственно через ExpressRoute *без* защиты IPsec 

Чтобы применить шифрование к обмену данными, необходимо убедиться, что для сети, подключенной к VPN, в схеме маршруты Azure с помощью локального VPN-шлюза являются предпочтительными по отношению к непосредственному пути ExpressRoute.

### <a name="traffic-from-azure-to-on-premises-networks"></a>Трафик из Azure в локальные сети

Это же требование распространяется на трафик из Azure в локальные сети. Чтобы убедиться, что путь IPsec предпочтительнее, чем прямой путь ExpressRoute (без IPsec), у вас есть два варианта:

- Объявите более конкретные префиксы в VPN-сеансе BGP для сети, подключенной к VPN. Вы можете объявить больший диапазон, охватывающий подключенную по сети VPN сеть через частный пиринг ExpressRoute, а затем более конкретные диапазоны в VPN-сеансе BGP. Например, объявить 10.0.0.0/16 через ExpressRoute и 10.0.1.0/24 через VPN.

- Объявление необъединяемых префиксов для VPN и ExpressRoute. Если диапазоны сетей, подключенных к VPN, не являются отсоединенными от других подключенных к сети ExpressRoute сетей, можно объявить префиксы в сеансах BGP VPN и ExpressRoute соответственно. Например, объявить 10.0.0.0/24 через ExpressRoute и 10.0.1.0/24 через VPN.

В обоих примерах Azure будет передавать трафик 10.0.1.0/24 через VPN-подключение, а не напрямую через ExpressRoute без защиты VPN.

> [!WARNING]
> Если вы объявите *одни и те же* префиксы как для ExpressRoute, так и для VPN-подключений, Azure будет использовать путь ExpressRoute напрямую без защиты VPN.
>

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="1-create-a-virtual-wan-and-hub-with-gateways"></a><a name="openvwan"></a>1. Создание виртуальной глобальной сети и концентратора с шлюзами

Прежде чем продолжать, необходимо выполнить следующие ресурсы Azure и соответствующие локальные конфигурации.

- Виртуальная глобальная сеть Azure
- Виртуальный концентратор глобальной сети с [шлюзом ExpressRoute](virtual-wan-expressroute-portal.md) и [VPN-шлюзом](virtual-wan-site-to-site-portal.md)

Действия по созданию виртуальной глобальной сети Azure и концентратора с Ассоциацией ExpressRoute см. в статье [Создание ассоциации expressroute с помощью виртуальной глобальной сети Azure](virtual-wan-expressroute-portal.md). Действия по созданию VPN-шлюза в виртуальной глобальной сети см. в статье [Создание подключения типа "сеть — сеть" с помощью виртуальной глобальной сети Azure](virtual-wan-site-to-site-portal.md).

## <a name="2-create-a-site-for-the-on-premises-network"></a><a name="site"></a>2. Создайте сайт для локальной сети.

Ресурс сайта совпадает с VPN-сайтами, отличными от ExpressRoute, для виртуальной глобальной сети. IP-адрес локального VPN-устройства теперь может быть частным IP-адресом или общедоступным IP-адресом в локальной сети, доступным через частный пиринг ExpressRoute, созданный на шаге 1.

> [!NOTE]
> IP-адрес для локального VPN-устройства *должен* быть частью префиксов адресов, объявленных в виртуальном КОНЦЕНТРАТОРЕ глобальной сети через частный пиринг Azure ExpressRoute.
>

1. Перейдите в портал Azure в браузере. 
1. Выберите созданный концентратор. На странице **Подключение** к концентратору ВИРТУАЛЬНОЙ глобальной сети выберите **VPN-сайты**.
1. На странице **VPN-узлы** выберите **+ создать сайт**.
1. На странице **создания сайта** заполните следующие поля:
   * **Подписка**: Проверьте подписку.
   * **Группа ресурсов**. Выберите или создайте группу ресурсов, которую вы хотите использовать.
   * **Регион**: введите регион Azure для ресурса сайта VPN.
   * **Имя**. Введите имя, по которому будет ссылаться на локальный сайт.
   * **Поставщик устройства**: введите поставщик локального VPN-устройства.
   * **Протокол BGP**. Выберите "включить", если локальная сеть использует BGP.
   * **Пространство частных адресов**: введите пространство IP-адресов, расположенное на локальном сайте. Трафик, предназначенный для этого адресного пространства, направляется в локальную сеть через VPN-шлюз.
   * **Концентраторы**. Выберите один или несколько концентраторов для подключения этого VPN-сайта. Для выбранных концентраторов должны быть уже созданы VPN-шлюзы.
1. Щелкните **Далее: ссылки >** для параметров VPN-канала:
   * **Имя ссылки**: имя, по которому должно быть сослаться это соединение.
   * **Имя поставщика**: имя поставщика услуг Интернета для этого сайта. Для локальной сети ExpressRoute это имя поставщика службы ExpressRoute.
   * **Speed**: скорость связи со службой Интернета или канал ExpressRoute.
   * **IP-адрес**: общедоступный IP-адрес VPN-устройства, расположенного на локальном сайте. Или для локальной сети ExpressRoute это частный IP-адрес VPN-устройства через ExpressRoute.

   Если BGP включен, он будет применяться ко всем подключениям, созданным для этого сайта в Azure. Настройка BGP в виртуальной глобальной сети аналогична настройке BGP на VPN-шлюзе Azure. 
   
   Адрес локального узла BGP *не должен* совпадать с IP-АДРЕСом VPN-устройства или адресным пространством виртуальной сети сайта VPN. Используйте в качестве IP-адреса узла BGP другой IP-адрес VPN-устройства. Можно использовать адрес интерфейса замыкания на себя. Однако это *не может* быть APIPA (169,254.*x*. *x*) адрес. Укажите этот адрес на соответствующем VPN-сайте, который представляет расположение. Предварительные требования BGP см. статье [About BGP with Azure VPN Gateway](../vpn-gateway/vpn-gateway-bgp-overview.md) (Сведения об использовании BGP с VPN-шлюзом Azure).

1. Нажмите кнопку **Далее: Проверка и создание >** , чтобы проверить значения параметров и создать VPN-сайт. Если выбраны **концентраторы** для подключения, подключение будет установлено между локальной сетью и VPN-шлюзом концентратора.

## <a name="3-update-the-vpn-connection-setting-to-use-expressroute"></a><a name="hub"></a>3. обновите параметр VPN-подключения, чтобы использовать ExpressRoute.

После создания VPN-сайта и подключения к концентратору выполните следующие действия, чтобы настроить подключение для использования частного пиринга ExpressRoute.

1. Вернитесь на страницу виртуальной глобальной сети и выберите ресурс концентратора. Или перейдите с VPN-сайта на подключенный центр.

   :::image type="content" source="./media/vpn-over-expressroute/hub-selection.png" alt-text="Выберите концентратор":::
1. В разделе **Подключение** выберите **VPN (сеть — сеть)**.

   :::image type="content" source="./media/vpn-over-expressroute/vpn-select.png" alt-text="Выберите VPN (&quot;сеть — сеть&quot;).":::
1. Нажмите кнопку с многоточием (**...**) на VPN-сайте через ExpressRoute и выберите **изменить VPN-подключение к этому концентратору**.

   :::image type="content" source="./media/vpn-over-expressroute/config-menu.png" alt-text="Введите меню конфигурации":::
1. Для **использования частного IP-адреса Azure** выберите **Да**. Этот параметр настраивает VPN-шлюз концентратора для использования частных IP-адресов в пределах диапазона адресов концентратора для этого подключения, а не общедоступных IP-адресов. Это обеспечит передачу трафика из локальной сети через пути частного пиринга ExpressRoute вместо использования общедоступного Интернета для этого VPN-подключения. На следующем снимке экрана показан параметр:

   :::image type="content" source="./media/vpn-over-expressroute/vpn-link-configuration.png" alt-text="Настройка для использования частного IP-адреса для VPN-подключения" border="false":::
1. Щелкните **Сохранить**.

После сохранения изменений VPN-шлюз концентратора будет использовать частные IP-адреса VPN-шлюза, чтобы установить подключения IPsec/IKE к локальному VPN-устройству через ExpressRoute.

## <a name="4-get-the-private-ip-addresses-for-the-hub-vpn-gateway"></a><a name="associate"></a>4. получение частных IP-адресов для VPN-шлюза концентратора

Скачайте конфигурацию VPN-устройства, чтобы получить частные IP-адреса VPN-шлюза концентратора. Эти адреса необходимы для настройки локального VPN-устройства.

1. На странице центра выберите **VPN (сеть-сеть)** в разделе **Подключение**.
1. В верхней части страницы **обзора** выберите **скачать конфигурацию VPN**. 

   Azure создает учетную запись хранения в группе ресурсов "Microsoft-Network-[Расположение]", где *Location* — это расположение глобальной сети. После применения конфигурации к VPN-устройствам эту учетную запись хранения можно удалить.
1. После создания файла щелкните ссылку, чтобы скачать его.
1. Примените конфигурацию к VPN-устройству.

### <a name="vpn-device-configuration-file"></a>Файл конфигурации VPN-устройства

Файл конфигурации устройства содержит параметры, используемые при настройке локального VPN-устройства. При просмотре этого файла обратите внимание на следующие сведения:

* **впнситеконфигуратион**. Этот раздел обозначает сведения об устройстве, настроенные как сайт, который подключается к виртуальной глобальной сети. Он включает имя и общедоступный IP-адрес устройства филиала.
* **впнситеконнектионс**. в этом разделе содержатся сведения о следующих параметрах.

    * Адресное пространство виртуальной сети виртуального концентратора.<br/>Пример
           ```
           "AddressSpace":"10.51.230.0/24"
           ```
    * Адресное пространство виртуальных сетей, подключенных к концентратору.<br>Пример
           ```
           "ConnectedSubnets":["10.51.231.0/24"]
            ```
    * IP-адреса VPN-шлюза виртуального концентратора. Так как каждое подключение VPN-шлюза состоит из двух туннелей в конфигурации "активный — активный", вы увидите оба IP-адреса, перечисленные в этом файле. В этом примере вы видите `Instance0` и `Instance1` для каждого сайта и они являются ЧАСТНЫМИ IP-адресами вместо общедоступных IP-адресов.<br>Пример
           ``` 
           "Instance0":"10.51.230.4"
           "Instance1":"10.51.230.5"
           ```
    * Сведения о конфигурации подключения VPN-шлюза, такие как BGP и общий ключ. Общий ключ создается автоматически. Вы всегда можете изменить подключение на странице **Обзор** для пользовательского общего ключа.
  
### <a name="example-device-configuration-file"></a>Пример файла конфигурации устройства

```
[{
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"5b096293-edc3-42f1-8f73-68c14a7c4db3"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-ER-site",
        "IPAddress":"172.24.127.211",
        "LinkName":"VPN-over-ER"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"10.51.230.4",
            "Instance1":"10.51.230.5"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
    },
    {
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"fbdb34ea-45f8-425b-9bc2-4751c2c4fee0"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-INet-site",
        "IPAddress":"13.75.195.234",
        "LinkName":"VPN-over-INet"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"51.143.63.104",
            "Instance1":"52.137.90.89"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
}]
```

### <a name="configuring-your-vpn-device"></a>Настройка VPN-устройства

Если вам нужны инструкции по настройке устройства, см. раздел [Загрузка сценариев конфигурации VPN-устройства из Azure](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts). При этом учтите следующее:

* Инструкции на странице VPN-устройства не записываются для виртуальной глобальной сети. Однако можно использовать значения виртуальной глобальной сети из файла конфигурации, чтобы вручную настроить VPN-устройство. 
* Загружаемые сценарии конфигурации устройств для VPN-шлюза не работают для виртуальной глобальной сети, так как конфигурация отличается.
* Новая виртуальная Глобальная сеть может поддерживать как IKEv1, так и IKEv2.
* Виртуальная глобальная сеть может использовать только VPN-устройства на основе маршрутов и инструкции для устройств.

## <a name="5-view-your-virtual-wan"></a><a name="viewwan"></a>5. Просмотр виртуальной глобальной сети

1. Перейдите к виртуальной глобальной сети.
1. На странице **Обзор** каждая точка на карте представляет собой концентратор.
1. В разделе **концентраторы и подключения** можно просматривать состояние концентратора, сайта, региона и VPN-подключения. Кроме того, можно просматривать и исходящие байты.

## <a name="6-monitor-a-connection"></a><a name="connectmon"></a>6. Мониторинг подключения

Создайте подключение для отслеживания обмена данными между виртуальной машиной Azure и удаленным сайтом. Сведения о настройке монитора подключения см. в статье [Руководство. Мониторинг сетевого взаимодействия между двумя виртуальными машинами с помощью портала Azure](~/articles/network-watcher/connection-monitor.md). Исходным полем является IP-адрес виртуальной машины в Azure, а целевым IP-адресом является IP-адрес сайта.

## <a name="7-clean-up-resources"></a><a name="cleanup"></a>7. Очистка ресурсов

Если эти ресурсы больше не нужны, можно удалить группу ресурсов и все содержащиеся в ней ресурсы с помощью команды [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup) . Выполните следующую команду PowerShell и замените `myResourceGroup` именем группы ресурсов:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

Эта статья поможет вам создать VPN-подключение через частный пиринг ExpressRoute с помощью виртуальной глобальной сети. Дополнительные сведения о виртуальной глобальной сети и связанных с ней функциях см. в [обзоре виртуальной глобальной сети](virtual-wan-about.md).
