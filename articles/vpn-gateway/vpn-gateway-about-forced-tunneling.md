---
title: 'VPN-шлюз Azure: Настройка принудительного туннелирования для подключений типа "сеть — сеть": классическая модель'
description: Узнайте, как настроить принудительное туннелирование для виртуальных сетей, созданных с помощью классической модели развертывания.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 10/15/2020
ms.author: cherylmc
ms.openlocfilehash: af4359efb48898c12bb8ee7ffb882448b5012d19
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92151352"
---
# <a name="configure-forced-tunneling-using-the-classic-deployment-model"></a>Настройка принудительного туннелирования с помощью классической модели развертывания

Оно позволяет перенаправлять или "принудительно направлять" весь Интернет-трафик обратно в локальное расположение через VPN типа "сеть — сеть" для проверки и аудита. Это критически важное требование безопасности, имеющееся в большинстве корпоративных ИТ-политик. Без принудительного туннелирования Интернет-трафик из виртуальных машин в Azure будет всегда поступать из инфраструктуры сети Azure непосредственно в Интернет, без возможности его проверки или аудита. Неавторизованный доступ в Интернет может привести к раскрытию информации или другим нарушениям безопасности.

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

В этой статье описана настройка принудительного туннелирования для виртуальных сетей, созданных с помощью классической модели развертывания. Принудительное туннелирование можно настроить с помощью PowerShell, но не на портале. Если вы хотите настроить принудительное туннелирование для модели развертывания диспетчер ресурсов, выберите диспетчер ресурсов статью из следующего раскрывающегося списка:

> [!div class="op_single_selector"]
> * [Классический](vpn-gateway-about-forced-tunneling.md)
> * [Resource Manager](vpn-gateway-forced-tunneling-rm.md)
> 

## <a name="requirements-and-considerations"></a>Требования и рекомендации

В Azure принудительное туннелирование настраивается с помощью определяемых пользователем маршрутов виртуальной сети (UDR). Перенаправление трафика на локальный сайт выполняется с помощью маршрута по умолчанию к VPN-шлюзу Azure. В разделе ниже перечислены текущие ограничения таблицы маршрутизации и маршрутов для виртуальной сети Azure.

* Каждая подсеть виртуальной сети имеет встроенные системные таблицы маршрутизации. Системная таблица маршрутизации содержит три указанные ниже группы маршрутов.

  * **Маршруты локальной виртуальной сети.** Они ведут непосредственно к виртуальным машинам назначения в той же виртуальной сети.
  * **Локальные маршруты.** Ведут к VPN-шлюзу Azure.
  * **Маршрут по умолчанию.** Ведет напрямую в Интернет. Пакеты, предназначенные для частных IP-адресов, не входящих в предыдущие два маршрута, будут удалены.
* Задав пользовательские маршруты, вы сможете создать таблицу маршрутизации для добавления маршрута по умолчанию, а затем сопоставить таблицу маршрутизации с подсетями вашей виртуальной сети для включения принудительного туннелирования в этих подсетях.
* Из числа локальных межорганизационных сайтов, подключенных к виртуальной сети, необходимо выбрать "сайт по умолчанию".
* Принудительное туннелирование должно быть сопоставлено с виртуальной сетью, в которой есть VPN-шлюз с динамической маршрутизацией (а не статический шлюз).
* С помощью этого механизма невозможно настроить принудительное туннелирование ExpressRoute. Такое туннелирование включается, когда предлагается маршрут по умолчанию с помощью сеансов пиринга BGP ExpressRoute. Дополнительные сведения см. в разделе [что такое ExpressRoute?](../expressroute/expressroute-introduction.md).

## <a name="configuration-overview"></a>Общие сведения о настройке

В следующем примере для интерфейсной подсети не применяется принудительное туннелирование. Рабочие нагрузки в интерфейсной подсети могут продолжать принимать запросы клиентов непосредственно из Интернета и отвечать на них. Для подсетей среднего уровня и внутренних подсетей применяется принудительное туннелирование. Любые исходящие подключения из этих двух подсетей к Интернету будут принудительно перенаправлены обратно на локальный сайт через один из VPN-туннелей, работающих по протоколу S2S.

Это позволяет ограничивать и проверять доступ в Интернет с виртуальных машин или облачных служб в Azure, при этом поддерживая необходимую многоуровневую архитектуру служб. Кроме того, вы можете применять принудительное туннелирование для всех виртуальных сетей, если в них нет рабочих нагрузок, требующих взаимодействия с Интернетом.

![Принудительное туннелирование](./media/vpn-gateway-about-forced-tunneling/forced-tunnel.png)

## <a name="prerequisites"></a>Предварительные условия

Перед началом настройки убедитесь, что у вас есть следующее:

* Подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/).
* Настроенная виртуальная сеть. 
* [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="configure-forced-tunneling"></a>Настройка принудительного туннелирования

Выполнив описанную ниже процедуру, вы сможете настроить принудительное туннелирование в виртуальной сети. Рекомендации по настройке относятся к файлу конфигурации виртуальной сети.  В этом примере в виртуальную сеть MultiTier-VNet входят три подсети (Frontend, Midtier и Backend) с четырьмя распределенными подключениями DefaultSiteHQ и тремя ветвями Branch.


```xml
<VirtualNetworkSite name="MultiTier-VNet" Location="North Europe">
     <AddressSpace>
      <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Frontend">
            <AddressPrefix>10.1.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Midtier">
            <AddressPrefix>10.1.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Backend">
            <AddressPrefix>10.1.2.0/23</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.1.200.0/28</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="DefaultSiteHQ">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch2">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch3">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSite>
```

Следующие шаги задают "DefaultSiteHQ" в качестве подключения к сайту по умолчанию для принудительного туннелирования, а также настраивают подсети Midtier и серверных подсетей для использования принудительного туннелирования.

1. Откройте консоль PowerShell с повышенными правами. Подключитесь к учетной записи, используя следующий пример:

   ```powershell
   Add-AzureAccount
   ```

1. Создайте таблицу маршрутизации. Для этого воспользуйтесь указанным ниже командлетом.

   ```powershell
   New-AzureRouteTable –Name "MyRouteTable" –Label "Routing Table for Forced Tunneling" –Location "North Europe"
   ```

1. Добавьте маршрут по умолчанию в таблицу маршрутизации. 

   В примере ниже показано, как добавить маршрут по умолчанию в таблицу маршрутизации, созданную в действии 1. Обратите внимание, что поддерживается только маршрут с указанием префикса места назначения "0.0.0.0/0" для значения VPNGateway свойства NextHop.

   ```powershell
   Get-AzureRouteTable -Name "MyRouteTable" | Set-AzureRoute –RouteTable "MyRouteTable" –RouteName "DefaultRoute" –AddressPrefix "0.0.0.0/0" –NextHopType VPNGateway
   ```

1. Сопоставьте таблицу маршрутизации с подсетями. 

   После создания таблицы маршрутизации и добавления маршрута с помощью примера ниже добавьте таблицу маршрутизации в подсеть виртуальной сети или сопоставьте таблицу с этой подсетью. В примере ниже показано, как добавить таблицу маршрутизации MyRouteTable в подсети Midtier и Backend виртуальной сети MultiTier-VNet.

   ```powershell
   Set-AzureSubnetRouteTable -VirtualNetworkName "MultiTier-VNet" -SubnetName "Midtier" -RouteTableName "MyRouteTable"
   Set-AzureSubnetRouteTable -VirtualNetworkName "MultiTier-VNet" -SubnetName "Backend" -RouteTableName "MyRouteTable"
   ```

1. Назначьте сайт по умолчанию для принудительного туннелирования. 

   В предыдущем действии с помощью сценариев с командлетами была создана таблица маршрутизации, которая затем была сопоставлена с двумя подсетями виртуальной сети. В последнем действии среди подключений к нескольким сайтам в виртуальной сети необходимо выбрать локальный сайт, который будет сайтом по умолчанию или туннелем.

   ```powershell
   $DefaultSite = @("DefaultSiteHQ")
   Set-AzureVNetGatewayDefaultSite –VNetName "MultiTier-VNet" –DefaultSite "DefaultSiteHQ"
   ```

## <a name="additional-powershell-cmdlets"></a>Дополнительные командлеты PowerShell

### <a name="to-delete-a-route-table"></a>Удаление таблицы маршрутизации

```powershell
Remove-AzureRouteTable -Name <routeTableName>
```
  
### <a name="to-list-a-route-table"></a>Отображение таблицы маршрутизации

```powershell
Get-AzureRouteTable [-Name <routeTableName> [-DetailLevel <detailLevel>]]
```

### <a name="to-delete-a-route-from-a-route-table"></a>Удаление маршрута из таблицы маршрутизации

```powershell
Remove-AzureRouteTable –Name <routeTableName>
```

### <a name="to-remove-a-route-from-a-subnet"></a>Удаление маршрута из подсети

```powershell
Remove-AzureSubnetRouteTable –VirtualNetworkName <virtualNetworkName> -SubnetName <subnetName>
```

### <a name="to-list-the-route-table-associated-with-a-subnet"></a>Отображение таблицы маршрутизации, сопоставленной с подсетью

```powershell
Get-AzureSubnetRouteTable -VirtualNetworkName <virtualNetworkName> -SubnetName <subnetName>
```

### <a name="to-remove-a-default-site-from-a-vnet-vpn-gateway"></a>Удаление сайта по умолчанию из VPN-шлюза виртуальной сети

```powershell
Remove-AzureVnetGatewayDefaultSite -VNetName <virtualNetworkName>
```
