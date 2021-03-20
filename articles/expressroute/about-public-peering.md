---
title: Создание общедоступного пиринга Azure ExpressRoute и управление им
description: Сведения об общедоступном пиринга Azure и управлении им
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/16/2019
ms.author: duau
ms.openlocfilehash: 477145619e1b4d8b41c422389b57a46615597478
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92202554"
---
# <a name="create-and-manage-expressroute-public-peering"></a>Создание общедоступного пиринга ExpressRoute и управление им

> [!div class="op_single_selector"]
> * [Статья — общедоступный пиринг](about-public-peering.md)
> * [Видео — общедоступный пиринг](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Статья — пиринг Майкрософт](expressroute-circuit-peerings.md#microsoftpeering)
>

Эта статья поможет вам создать и управлять конфигурацией маршрутизации общедоступного пиринга для канала ExpressRoute. Вы также можете проверить состояние, обновить или удалить и отменить инициализацию пиринга. Эта статья относится к диспетчер ресурсовным каналам, созданным до объявления общедоступного пиринга. При наличии ранее существующего канала (созданного до использования общедоступного пиринга) можно управлять и настраивать общедоступные пиринга с помощью [Azure PowerShell](#powershell), [Azure CLI](#cli)и [портал Azure](#portal).

>[!NOTE]
>Общедоступный пиринг является устаревшим. Невозможно создать общедоступный пиринг для новых каналов ExpressRoute. Если вы используете новый канал ExpressRoute, используйте [пиринг Майкрософт](expressroute-circuit-peerings.md#microsoftpeering) для служб Azure.
>

## <a name="connectivity"></a>Возможность подключения

Подключение всегда инициируется из глобальной сети к службам Microsoft Azure. Службы Microsoft Azure не будут иметь возможность инициировать подключения к вашей сети через этот домен маршрутизации. Если канал ExpressRoute включен для общедоступного пиринга Azure, вы можете получить доступ к [диапазонам общедоступных IP-адресов, используемым в Azure](../virtual-network/public-ip-addresses.md#public-ip-addresses) по каналу.

После включения общедоступного пиринга можно подключиться к большинству служб Azure. Мы не позволяем выбирать службы, для которых объявляются маршруты.

* Службы, такие как служба хранилища Azure, базы данных SQL и веб-сайты, предлагаются на общедоступных IP-адресах.
* С помощью домена маршрутизации общедоступного пиринга можно частным образом подключаться к службам, размещенным на общедоступных IP-адресах, включая VIP облачных служб.
* Вы можете подключить домен общедоступного пиринга к своей сети периметра и подключаться ко всем службам Azure по их общедоступным IP-адресам из своей глобальной сети без необходимости подключения через Интернет.

## <a name="services"></a><a name="services"></a>Обслуживание

В этом разделе показаны службы, доступные через общедоступный пиринг. Поскольку общедоступный пиринг является устаревшим, нет необходимости добавлять новые или дополнительные службы в общедоступный пиринг. Если вы используете общедоступный пиринг и служба, которую вы хотите использовать, поддерживается только через пиринг Майкрософт, необходимо переключиться на пиринг Майкрософт. Список поддерживаемых служб см. в разделе [пиринг Майкрософт](expressroute-faqs.md#microsoft-peering) .

**Поддерживаются:**

* Power BI
* Большинство служб Azure поддерживается. Проверьте непосредственно в службе, которую вы хотите использовать для проверки поддержки.

**Не поддерживаются:**
  * CDN
  * Azure Front Door
  * Сервер Многофакторной проверки подлинности (устаревший)
  * Диспетчер трафика

Чтобы проверить доступность конкретной службы, ознакомьтесь с документацией по ней и определите, опубликован ли зарезервированный диапазон для этой службы. Затем можно найти диапазоны IP-адресов целевой службы и сравнить их с диапазонами, перечисленными в поле [диапазоны IP-адресов Azure и теги службы — общедоступный облачный XML-файл](https://www.microsoft.com/download/details.aspx?id=56519). Кроме того, можно открыть запрос в службу поддержки для уточнения.

## <a name="peering-comparison"></a><a name="compare"></a>Сравнение пирингов

[!INCLUDE [peering comparison](../../includes/expressroute-peering-comparison.md)]

> [!NOTE]
> Общедоступный пиринг Azure имеет 1 IP-адрес NAT, связанный с каждым сеансом BGP. Для более чем 2 IP-адресов NAT перейдите в пиринг Майкрософт. Пиринг Майкрософт позволяет настраивать собственные выделения NAT, а также использовать фильтры маршрутов для избирательных объявлений префиксов. Дополнительные сведения см. [в разделе Переход на пиринг Майкрософт](./how-to-move-peering.md).
>

## <a name="custom-route-filters"></a>Фильтры настраиваемых маршрутов

Вы можете задавать в своей сети настраиваемые фильтры маршрутов, чтобы использовать только нужные маршруты. Подробные сведения о настройке маршрутизации см. на странице [Маршрутизация](expressroute-routing.md).

## <a name="azure-powershell-steps"></a><a name="powershell"></a>Azure PowerShell действия


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

Поскольку общедоступный пиринг является устаревшим, вы не можете настроить общедоступный пиринг в новом канале ExpressRoute.

1. Убедитесь, что у вас есть канал ExpressRoute, который подготовлен и включен. Используйте следующий пример:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   Ответ будет выглядеть примерно так:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
   ProvisioningState                : Succeeded
   Sku                              : {
                                      "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             : 
   ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
2. Настройте для канала общедоступный пиринг Azure. Прежде чем продолжить, убедитесь, что у вас есть следующие сведения:

   * Подсеть /30 для основной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Подсеть /30 для дополнительной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * Необязательное действие:
   * Хэш MD5, если вы решите его использовать.

   Чтобы настроить для канала общедоступный пиринг Azure, выполните код из примера ниже

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Если вы решили использовать MD5-хэш, используйте следующий пример:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100  -SharedKey "A1B2C3D4"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   > [!IMPORTANT]
   > Номер AS должен быть указан в качестве ASN пиринга, а не ASN клиента.
   > 
   >

### <a name="to-get-azure-public-peering-details"></a><a name="getpublic"></a>Получение сведений об общедоступном пиринге Azure

Для получения сведений о конфигурации можно использовать следующий командлет.

```azurepowershell-interactive
  $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

  Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -Circuit $ckt
  ```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Обновление конфигурации общедоступного пиринга Azure

С помощью следующего примера можно обновить любую часть конфигурации. В приведенном ниже примере значение идентификатора виртуальной локальной сети для канала изменяется с 200 на 600.

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 600

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Удаление общедоступного пиринга Azure

Для удаления конфигурации пиринга выполните следующий пример кода:

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-cli-steps"></a><a name="cli"></a>Azure CLI действия


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

1. Убедитесь, что канал ExpressRoute подготовлен и включен. Используйте следующий пример:

   ```azurecli-interactive
   az network express-route list
   ```

   Ответ будет выглядеть примерно так:

   ```output
   "allowClassicOperations": false,
   "authorizations": [],
   "circuitProvisioningState": "Enabled",
   "etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
   "gatewayManagerEtag": null,
   "id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
   "location": "westus",
   "name": "MyCircuit",
   "peerings": [],
   "provisioningState": "Succeeded",
   "resourceGroup": "ExpressRouteResourceGroup",
   "serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
   "serviceProviderNotes": null,
   "serviceProviderProperties": {
    "bandwidthInMbps": 200,
    "peeringLocation": "Silicon Valley",
    "serviceProviderName": "Equinix"
   },
   "serviceProviderProvisioningState": "Provisioned",
   "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
   },
   "tags": null,
   "type": "Microsoft.Network/expressRouteCircuits]
   ```

2. Настройте для канала общедоступный пиринг Azure. Прежде чем продолжить, убедитесь, что у вас есть следующие сведения:

   * Подсеть /30 для основной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Подсеть /30 для дополнительной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * **Необязательно.** Хэш MD5, если вы решите его использовать.

   Чтобы настроить для канала общедоступный пиринг Azure, выполните код из следующего примера:

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering
   ```

   Если вы решили использовать MD5-хэш, используйте следующий пример:

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering --SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Номер AS должен быть указан в качестве ASN пиринга, а не ASN клиента.

### <a name="to-view-azure-public-peering-details"></a><a name="getpublic"></a>Просмотр сведений об общедоступном пиринге Azure

Для получения сведений о конфигурации можно использовать следующий пример:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePublicPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePublicPeering",
  "peerAsn": 7671,
  "peeringType": "AzurePublicPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Обновление конфигурации общедоступного пиринга Azure

С помощью следующего примера можно обновить любую часть конфигурации. В приведенном ниже примере значение идентификатора виртуальной локальной сети для канала изменяется с 200 на 600.

```azurecli-interactive
az network express-route peering update --vlan-id 600 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Удаление общедоступного пиринга Azure

Для удаления конфигурации пиринга выполните следующий пример кода:

```azurecli-interactive
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

## <a name="azure-portal-steps"></a><a name="portal"></a>Инструкции для портала Azure

Чтобы настроить пиринг, воспользуйтесь инструкциями PowerShell или CLI, которые содержатся в этой статье. Для управления пирингом можно использовать следующие разделы. В качестве справочных данных эти действия похожи на управление [пиринга Майкрософт на портале](expressroute-howto-routing-portal-resource-manager.md#msft).

### <a name="to-view-azure-public-peering-details"></a><a name="get"></a>Просмотр сведений об общедоступном пиринге Azure

Просмотрите свойства общедоступного пиринга Azure, выбрав пиринг на портале.

### <a name="to-update-azure-public-peering-configuration"></a><a name="update"></a>Обновление конфигурации общедоступного пиринга Azure

Выберите строку для пиринга, а затем измените свойства пиринга.

### <a name="to-delete-azure-public-peering"></a><a name="delete"></a>Удаление общедоступного пиринга Azure

Удалите конфигурацию пиринга, щелкнув значок Удалить.

## <a name="next-steps"></a>Дальнейшие действия

Следующий шаг — [связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md).

* Дополнительную информацию о рабочих процессах ExpressRoute см. в статье [Процедуры ExpressRoute для подготовки каналов и состояний каналов](expressroute-workflows.md).
* Дополнительную информацию о пиринге канала см. в статье [Каналы ExpressRoute и домены маршрутизации](expressroute-circuit-peerings.md).
* Подробнее о работе с виртуальными сетями см. в статье [Обзор виртуальных сетей](../virtual-network/virtual-networks-overview.md).