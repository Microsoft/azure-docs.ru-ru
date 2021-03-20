---
title: 'Подключение виртуальной сети к нескольким сайтам с помощью VPN-шлюза: классическая модель'
description: Подключение нескольких локальных сайтов к классической виртуальной сети с использованием VPN-шлюза
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.openlocfilehash: 168bb9e06c73ec27ec1304813023889c9549b8e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94660701"
---
# <a name="add-a-site-to-site-connection-to-a-vnet-with-an-existing-vpn-gateway-connection-classic"></a>Добавление подключения типа "сеть-сеть" к виртуальной сети с помощью существующего подключения VPN-шлюза (классическая модель)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

> [!div class="op_single_selector"]
> * [Портал Azure](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (классическая модель)](vpn-gateway-multi-site.md)
>
>

В этой статье рассматривается использование PowerShell для добавления подключений типа "сеть — сеть" (S2S) к VPN-шлюзу с имеющимся подключением. Этот тип подключения часто называется "многосайтовым". Действия из этой статьи относятся к виртуальным сетям, созданным с помощью классической модели развертывания (также называется управлением службами). Эти действия не применяются к параллельно существующим конфигурациям подключений ExpressRoute и "сеть — сеть".

### <a name="deployment-models-and-methods"></a>Модели и методы развертывания

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

Мы обновляем эту таблицу по мере выпуска новых статей и дополнительных инструментов для этой конфигурации. При появлении статьи мы указываем прямую ссылку на нее в этой таблице.

[!INCLUDE [vpn-gateway-table-multi-site](../../includes/vpn-gateway-table-multisite-include.md)]

## <a name="about-connecting"></a>О подключении

Вы можете подключить несколько локальных сайтов к одной виртуальной сети. Это особенно привлекательно для создания гибридных облачных решений. Создание многосайтового подключения к шлюзу виртуальной сети Azure похоже на создание других подключений типа "сеть-сеть". На самом деле вы можете использовать имеющийся VPN-шлюз Azure, если это шлюз с динамической маршрутизацией (на основе маршрутов).

Если к вашей виртуальной сети уже подключен статический шлюз, вы можете изменить его тип на динамический, не перестраивая виртуальную сеть, чтобы обеспечить многосайтовое подключение. Прежде чем менять тип маршрутизации, убедитесь, что локальный VPN-шлюз поддерживает конфигурации VPN на основе маршрутов.

![схема многосайтового подключения](./media/vpn-gateway-multi-site/multisite.png "несколько сайтов")

## <a name="points-to-consider"></a>Что необходимо учесть

**Для внесения изменений в эту виртуальную сеть портал не подходит.** Нужно внести в файл конфигурации сети, а не использовать для этого портал. Если внести изменения на портале, они заменят параметры ссылок на несколько сайтов для этой виртуальной сети.

К завершению процедуры многосайтового подключения вы научитесь уверенно использовать файл конфигурации сети. Однако при наличии нескольких людей, работающих над конфигурацией сети, необходимо убедиться, что все знают об этом ограничении. Это не значит, что использовать портал Azure вообще не нужно. Вы можете использовать его для всех действий, кроме изменения конфигурации этой конкретной виртуальной сети.

## <a name="before-you-begin"></a>Перед началом

Перед началом настройки убедитесь, что выполнены следующие предварительные требования:

* Совместимое оборудование VPN для каждого местного расположения. Ознакомьтесь со статьей [VPN-устройства для создания виртуальных сетей](vpn-gateway-about-vpn-devices.md) , чтобы проверить, является ли совместимым устройство, которое вы хотите использовать.
* Внешний общедоступный IPv4-адрес для каждого VPN-устройства. Этот IP-адрес не может располагаться вне преобразования сетевых адресов (NAT). Это обязательное требование.
* Пользователь, имеющий опыт в настройке вашего оборудования VPN. Вам нужно будет хорошо понимать, как настраивать VPN-устройство, или воспользоваться помощью соответствующего специалиста.
* Диапазоны IP-адресов, которые вы хотите использовать для виртуальной сети (если она еще не создана).
* Диапазоны IP-адресов для каждого сайта локальной сети, к которым вы будете подключаться. Понадобится убедиться, что диапазоны IP-адресов сайтов локальной сети, к которым требуется подключиться, не перекрываются. В противном случае портал или REST API отклонит передаваемую конфигурацию.<br>Например, если два сайта локальной сети содержат диапазон IP-адресов 10.2.3.0/24 и имеется пакет с адресом назначения 10.2.3.3, Azure не сможет определить сайт, на который требуется отправить пакет, так как диапазоны адресов перекрываются. Для предотвращения проблем с маршрутизацией Azure не позволит передать файл конфигурации с перекрывающимися диапазонами адресов.

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="1-create-a-site-to-site-vpn"></a>1. Создание VPN типа "сеть — сеть"
Это отлично, если у вас уже есть подключение VPN типа "сеть — сеть" со шлюзом с динамической маршрутизацией. Можно перейти к [экспорту параметров конфигурации виртуальной сети](#export). Если нет, выполните следующее:

### <a name="if-you-already-have-a-site-to-site-virtual-network-but-it-has-a-static-policy-based-routing-gateway"></a>Если у вас уже есть виртуальная сеть типа "сеть — сеть", но со шлюзом со статической маршрутизацией (на основе политики)
1. Измените тип шлюза на шлюз с динамической маршрутизацией. Для многосайтового подключения VPN требуется шлюз с динамической маршрутизацией (также известный как шлюз на основе маршрутов). Чтобы изменить тип шлюза, необходимо сначала удалить существующий шлюз, а затем создать новый.
2. Настройте новый шлюз и создайте VPN-туннель. Инструкции см. в разделе [Указание SKU и типа VPN](vpn-gateway-howto-site-to-site-classic-portal.md#sku). Убедитесь, что для типа маршрутизации задано значение "Динамический".

### <a name="if-you-dont-have-a-site-to-site-virtual-network"></a>Если у вас нет виртуальной сети типа "сеть — сеть"
1. Создайте виртуальную сеть с VPN-подключением типа "сеть — сеть" с помощью указаний в [этой статье](./vpn-gateway-howto-site-to-site-classic-portal.md).  
2. Настройте шлюз с динамической маршрутизацией с помощью следующей процедуры: [Настройка VPN-шлюза](./vpn-gateway-howto-site-to-site-classic-portal.md). Не забудьте выбрать **динамическую маршрутизацию** в качестве типа шлюза.

## <a name="2-export-the-network-configuration-file"></a><a name="export"></a>2. Экспортируйте файл конфигурации сети.

Откройте консоль PowerShell с повышенными правами. Чтобы переключиться на управление службами, используйте следующую команду:

```powershell
azure config mode asm
```

Подключитесь к учетной записи. Для подключения используйте следующий пример кода:

```powershell
Add-AzureAccount
```

Экспортируйте файл конфигурации сети Azure, выполнив приведенную ниже команду. При необходимости можно изменить расположение файла для экспорта.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

## <a name="3-open-the-network-configuration-file"></a>3. Откройте файл конфигурации сети.
Откройте файл конфигурации сети, скачанный на последнем шаге. Используйте любой редактор XML. Файл должен выглядеть следующим образом:

```xml
<NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
  <VirtualNetworkConfiguration>
    <LocalNetworkSites>
      <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
          <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.2.3.4</VPNGatewayAddress>
      </LocalNetworkSite>
      <LocalNetworkSite name="Site2">
        <AddressSpace>
          <AddressPrefix>10.2.0.0/16</AddressPrefix>
          <AddressPrefix>10.3.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.4.5.6</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="VNet1" AffinityGroup="USWest">
        <AddressSpace>
          <AddressPrefix>10.20.0.0/16</AddressPrefix>
          <AddressPrefix>10.21.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="FE">
            <AddressPrefix>10.20.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="BE">
            <AddressPrefix>10.20.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.20.2.0/29</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="Site1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
  </VirtualNetworkConfiguration>
</NetworkConfiguration>
```

## <a name="4-add-multiple-site-references"></a>4. Добавление ссылок на несколько сайтов
При добавлении или удалении данных ссылок на сайты будут внесены изменения в элемент ConnectionsToLocalNetwork/LocalNetworkSiteRef. При добавлении новой ссылки на локальный сайт Azure инициирует создание нового туннеля. В следующем примере приведена конфигурации сети для подключения одного сайта. Сохраните файл после внесения изменений.

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

Чтобы добавить ссылки на сайты (создать многосайтовую конфигурацию), просто добавьте дополнительные строки "LocalNetworkSiteRef", как показано в следующем примере:

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
      <LocalNetworkSiteRef name="Site2"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

## <a name="5-import-the-network-configuration-file"></a>5. Импорт файла конфигурации сети
Импортируйте файл конфигурации сети. После импорта этого файла с изменениями будут добавлены новые туннели. Туннели будут использовать динамический шлюз, который был создан ранее. Для импорта файла можно использовать PowerShell.

## <a name="6-download-keys"></a>6. скачивание ключей
После добавления новых туннелей используйте командлет PowerShell "Get-AzureVNetGatewayKey", чтобы получить общие ключи IPsec/IKE для каждого туннеля.

Пример:

```powershell
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site1"
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site2"
```

Кроме того, при желании можно использовать REST API *получения общего ключа шлюза виртуальной сети* для получения общих ключей.

## <a name="7-verify-your-connections"></a>7. Проверка подключений
Проверьте состояние многосайтового туннеля. После скачивания ключей для каждого туннеля необходимо проверить подключения. Используйте командлет "Get-AzureVnetConnection", чтобы получить список туннелей виртуальной сети, как показано в примере ниже. VNet1 — имя виртуальной сети.

```powershell
Get-AzureVnetConnection -VNetName VNET1
```

Пример возвращаемых данных:

```
    ConnectivityState         : Connected
    EgressBytesTransferred    : 661530
    IngressBytesTransferred   : 519207
    LastConnectionEstablished : 5/2/2014 2:51:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site1' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site1
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7f68a8e6-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded

    ConnectivityState         : Connected
    EgressBytesTransferred    : 789398
    IngressBytesTransferred   : 143908
    LastConnectionEstablished : 5/2/2014 3:20:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site2' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site2
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7893b329-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы больше узнать о VPN-шлюзах, см. статью [Шлюзы VPN](vpn-gateway-about-vpngateways.md).