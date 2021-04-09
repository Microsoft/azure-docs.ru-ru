---
title: включить файл
description: включить файл
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 12/13/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c0c05eb88df27ba9e71c0f61bad75457d893a7ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95562830"
---
> [!NOTE]
> Эти примеры не применяются при одновременном использовании конфигураций S2S и ExpressRoute.
> Дополнительные сведения о работе со шлюзами при одновременном использовании конфигураций см. в разделе о [настройке параллельных подключений](../articles/expressroute/expressroute-howto-coexist-classic.md#gw).

## <a name="add-a-gateway"></a>Добавление шлюза

При добавлении шлюза в виртуальную сеть с помощью ресурсов классической модели файл конфигурации сети изменяется непосредственно перед созданием шлюза. Чтобы создать шлюз, значения в приведенных ниже примерах должны присутствовать в файле. Если с вашей виртуальной сетью ранее был связан шлюз, некоторые из этих значений уже будут присутствовать. Измените файл с учетом значений ниже.

### <a name="download-the-network-configuration-file"></a>Загрузка файла конфигурации сети

1. Скачайте файл конфигурации сети, используя шаги, описанные в статье о [файле конфигурации сети](/previous-versions/azure/virtual-network/virtual-networks-using-network-configuration-file). Откройте файл в текстовом редакторе.
2. Добавьте в файл локальный сетевой сайт. Можно использовать любой допустимый префикс адреса. Можно добавить любой допустимый IP-адрес для VPN-шлюза. Значения адресов в этом разделе не используются для операций с ExpressRoute, но необходимы для проверки файла. В примере "branch1" – это имя узла. Вы можете использовать другое имя, но обязательно укажите то же значение в разделе файла Gateway (Шлюз).

   ```
   <VirtualNetworkConfiguration>
    <Dns />
    <LocalNetworkSites>
      <LocalNetworkSite name="branch1">
        <AddressSpace>
          <AddressPrefix>165.3.1.0/27</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>3.2.1.4</VPNGatewayAddress>
    </LocalNetworkSite>
   ```
3. Перейдите к VirtualNetworkSites и измените поля.

   * Убедитесь, что подсеть шлюза существует для вашей виртуальной сети. Если ее нет, ее можно сразу добавить. Задайте имя "GatewaySubnet".
   * Убедитесь, что в файле существует раздел Gateway (Шлюз). Если нет, добавьте его. Он необходим для связи виртуальной сети с локальным сетевым сайтом (представляющим сеть, к которой вы подключаетесь).
   * Убедитесь, что тип подключения = Dedicated (Выделенный). Он необходим для подключения ExpressRoute.

   ```
   </LocalNetworkSites>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="myAzureVNET" Location="East US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="default">
            <AddressPrefix>10.0.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.1.0/27</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="branch1">
              <Connection type="Dedicated" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
   </VirtualNetworkConfiguration>
   </NetworkConfiguration>
   ```
4. Сохраните и передайте файл в Azure.

### <a name="create-the-gateway"></a>Создание шлюза

Используйте следующую команду, чтобы создать шлюз. Подставьте собственные значения.

```powershell
New-AzureVNetGateway -VNetName "MyAzureVNET" -GatewayType DynamicRouting -GatewaySKU  Standard
```

## <a name="verify-the-gateway-was-created"></a>Проверка создания шлюза

Используйте следующую команду, чтобы проверить, был ли создан шлюз. Эта команда также получает идентификатор шлюза, который нужен для других операций.

```powershell
Get-AzureVNetGateway
```

## <a name="resize-a-gateway"></a>Изменение размера шлюза

Существует несколько [номеров SKU шлюзов](../articles/expressroute/expressroute-about-virtual-network-gateways.md). Чтобы изменить SKU шлюза в любое время, можно использовать следующую команду.

> [!IMPORTANT]
> Эта команда не работает для шлюза UltraPerformance. Чтобы заменить шлюз на UltraPerformance, сначала удалите существующий шлюз ExpressRoute, а затем создайте шлюз UltraPerformance. Чтобы заменить шлюз UltraPerformance обычным шлюзом, сначала удалите шлюз UltraPerformance, а затем создайте новый шлюз.
>
>

```powershell
Resize-AzureVNetGateway -GatewayId <Gateway ID> -GatewaySKU HighPerformance
```

## <a name="remove-a-gateway"></a>Удаление шлюза

Используйте следующую команду, чтобы удалить шлюз.

```powershell
Remove-AzureVnetGateway -GatewayId <Gateway ID>
```