---
title: Руководство. Настройка пиринга для канала ExpressRoute — Azure CLI
description: В этом руководстве показано, как создать и подготовить частный пиринг, общедоступный пиринг или пиринг Майкрософт для канала ExpressRoute. а также показано, как проверить состояние, обновить или удалить пиринги для канала.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/09/2020
ms.author: duau
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 2c56e847e3b112d50285cd2c116c8f22efbc507f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101715536"
---
# <a name="tutorial-create-and-modify-peering-for-an-expressroute-circuit-using-cli"></a>Руководство. Создание и изменение пиринга для канала ExpressRoute с помощью интерфейса командной строки

В этом руководстве показано, как с помощью CLI и модели развертывания Resource Manager создать конфигурацию маршрутизации (пиринга) для канала ExpressRoute и управлять ею. Вы также сможете проверить состояние, обновить, удалить и отозвать пиринги для канала ExpressRoute. Если вы хотите использовать для работы с каналом другой метод, выберите подходящую статью из списка ниже.

> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [Общедоступный пиринг](about-public-peering.md)
> * [Видео — частный пиринг](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Видео — пиринг Майкрософт](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-routing-classic.md)
> 

В этом руководстве описано следующее:
> [!div class="checklist"]
> - настройка, обновление и удаление пиринга Майкрософт для канала;
> - настройка, обновление и удаление частного пиринга Azure для канала.

## <a name="prerequisites"></a>Предварительные требования

* Перед началом работы установите последнюю версию команд интерфейса командной строки (версию 2.0 или более позднюю). Сведения об установке команд CLI см. в руководстве по [установке Azure](/cli/azure/install-azure-cli).
* Просмотрите [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [описание рабочих процессов](expressroute-workflows.md), прежде чем приступать к настройке.
* Вам потребуется активный канал ExpressRoute. Приступая к работе, [создайте канал ExpressRoute](howto-circuit-cli.md). Его должен включить поставщик услуг подключения. Для выполнения команд, описанных в статье, нужно подготовить и включить канал ExpressRoute.

Эти инструкции распространяются только на каналы от поставщиков, предоставляющих услуги подключения второго уровня. Если ваш поставщик услуг подключения предлагает услуги третьего уровня (обычно это IPVPN, например MPLS), он отвечает за настройку маршрутизации и управление ею.

Вы можете настроить частный пиринг и пиринг Майкрософт для канала ExpressRoute. Пиринги можно настраивать в любом порядке, главное, выполнять их конфигурацию по очереди. Дополнительную информацию о доменах маршрутизации и пиринге см. в статье [Каналы ExpressRoute и домены маршрутизации](expressroute-circuit-peerings.md).

## <a name="microsoft-peering"></a><a name="msft"></a>Пиринг Майкрософт

Этот раздел поможет создать, получить, обновить и (или) удалить конфигурацию пиринга Майкрософт для канала ExpressRoute.

> [!IMPORTANT]
> Пиринг Майкрософт для каналов ExpressRoute, которые были настроены до 1 августа 2017 г., позволяет объявлять все префиксы служб, даже если фильтры маршрутов не определены. Пиринг Майкрософт для каналов ExpressRoute, настроенных 1 августа 2017 г. или позднее, не будет объявлять префиксы, пока к каналу не будет присоединен фильтр маршрутов. Дополнительные сведения см. в руководстве по [настройке фильтра маршрута для пиринга Майкрософт](how-to-routefilter-powershell.md).
> 


### <a name="to-create-microsoft-peering"></a>Создание пиринга Майкрософт

1. Установите последнюю версию Azure CLI. При работе с этим руководством нужно использовать именно ее.

   ```azurecli
   az login
   ```

   Выберите подписку, для которой будете создавать канал ExpressRoute.

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```

1. Создайте канал ExpressRoute. Выполните инструкции по созданию [канала ExpressRoute](howto-circuit-cli.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения оказывает услуги третьего уровня, он может включить для вас пиринг Майкрософт. В этом случае инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией, после создания канала сделайте следующее. 

1. Убедитесь, что канал ExpressRoute подготовлен и включен. Используйте следующий пример:

   ```azurecli
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

4. Настройте пиринг Майкрософт для этого канала. Прежде чем продолжить, убедитесь, что у вас есть следующие сведения.

   * Подсеть /30 для основной ссылки. Блок адресов должен быть представлен префиксом допустимого общедоступного адреса IPv4, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Подсеть /30 для дополнительной ссылки. Блок адресов должен быть представлен префиксом допустимого общедоступного адреса IPv4, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * Объявленные префиксы: предоставьте список всех префиксов, которые вы планируете объявить во время сеанса BGP. Допускаются только общедоступные префиксы IP-адресов. Если вы планируете отправить набор префиксов, их можно оформить в виде списка, разделенного запятыми. Эти префиксы должны быть зарегистрированы в RIR/IRR на ваше имя.
   * **Необязательно.** Клиент ASN: для объявления префиксов, не зарегистрированных с номером AS для пиринга, можно указать номер AS, с которым они зарегистрированы.
   * Имя реестра маршрутизации: можно указать RIR/IRR, в котором зарегистрированы номер AS и префиксы.
   * **Необязательно.** Хэш MD5, если вы решите его использовать.

   Чтобы настроить пиринг Майкрософт для своего канала, выполните следующий пример кода:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 123.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 123.0.0.4/30 --vlan-id 300 --peering-type MicrosoftPeering --advertised-public-prefixes 123.1.0.0/24
   ```

### <a name="to-view-microsoft-peering-details"></a><a name="getmsft"></a>Просмотр сведений о пиринге Майкрософт

Для получения сведений о конфигурации можно использовать следующий пример кода:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzureMicrosoftPeering
```
> [!IMPORTANT]
> Корпорация Майкрософт проверяет, назначены ли вам указанные объявленные открытые префиксы (Advertised public prefixes) и одноранговый узел ASN (Peer ASN) или ASN клиента (Customer ASN) в реестре маршрутизации в Интернете. Если вы получаете открытые префиксы из другой сущности и если назначение не записано в реестре маршрутизации, автоматическая проверка не будет завершена и потребуется проверка вручную. Если результат автоматической проверки отрицательный, в выходных данных вышеуказанной команды параметр AdvertisedPublicPrefixesState получит значение Validation needed (Требуется проверка). 
> 
> Если вы видите это сообщение, соберите документы, подтверждающие, что открытые префиксы назначены вашей организации сущностью, указанной в качестве владельца префиксов в реестре маршрутизации. Отправьте эти документы для проверки вручную, открыв запрос в службе поддержки. 
> 
>

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzureMicrosoftPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": {
    "advertisedPublicPrefixes": [
        ""
      ],
     "advertisedPublicPrefixesState": "",
     "customerASN": ,
     "routingRegistryName": ""
  }
  "name": "AzureMicrosoftPeering",
  "peerAsn": ,
  "peeringType": "AzureMicrosoftPeering",
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

### <a name="to-update-microsoft-peering-configuration"></a><a name="updatemsft"></a>Обновление конфигурации пиринга Майкрософт

Вы можете обновить любую часть конфигурации. Следующий пример кода изменяет объявленные для канала префиксы с 123.1.0.0/24 на 124.1.0.0/24:

```azurecli
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroup --peering-type MicrosoftPeering --advertised-public-prefixes 124.1.0.0/24
```

### <a name="to-add-ipv6-microsoft-peering-settings-to-an-existing-ipv4-configuration"></a><a name="addIPv6msft"></a>Добавление параметров пиринга Майкрософт IPv6 к существующей конфигурации IPv4

```azurecli
az network express-route peering update -g ExpressRouteResourceGroup --circuit-name MyCircuit --peering-type MicrosoftPeering --ip-version ipv6 --primary-peer-subnet 2002:db00::/126 --secondary-peer-subnet 2003:db00::/126 --advertised-public-prefixes 2002:db00::/126
```

## <a name="azure-private-peering"></a><a name="private"></a>Частный пиринг Azure

Этот раздел поможет вам создать, получить, обновить и (или) удалить конфигурацию частного пиринга Azure для канала ExpressRoute.

### <a name="to-create-azure-private-peering"></a>Создание частного пиринга Azure

1. Установите последнюю версию Azure CLI.
1. 
   ```azurecli
   az login
   ```

   Выберите подписку, необходимую для создания канала ExpressRoute

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```
1. Создайте канал ExpressRoute. Выполните инструкции по созданию [канала ExpressRoute](howto-circuit-cli.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения предоставляет управляемые службы уровня 3, он может включить для вас частный пиринг Azure. В этом случае инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией, после создания канала сделайте следующее.

1. Убедитесь, что канал ExpressRoute подготовлен и включен. Используйте следующий пример:

   ```azurecli
   az network express-route show --resource-group ExpressRouteResourceGroup --name MyCircuit
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

1. Настройте для канала частный пиринг Azure. Прежде чем продолжить, убедитесь в наличии следующих элементов:

   * Пара подсетей, которые не являются частью какого-либо диапазона адресов, зарезервированного для виртуальных сетей. Одна подсеть будет использоваться в качестве основной ссылки, а вторая — в качестве дополнительной. Из каждой подсети вы назначите первый готовый к применению IP-адрес для своего маршрутизатора, так как корпорация Майкрософт использует второй IP-адрес для маршрутизатора Майкрософт. Существует три варианта пары подсетей:
       * IPv4: две подсети /30;
       * IPv6: две подсети /126;
       * Оба: две подсети /30 и две подсети /126.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS. Для этого пиринга можно использовать частный номер AS. Убедитесь, что вы не используете номер 65515.
   * **Необязательно.** Хэш MD5, если вы решите его использовать.

   Чтобы настроить для канала частный пиринг Azure, выполните код из следующего примера:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering
   ```

   Если вы решили использовать MD5-хэш, используйте следующий пример:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering --SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Номер AS должен быть указан в качестве ASN пиринга, а не ASN клиента.
   > 
   > 

### <a name="to-view-azure-private-peering-details"></a><a name="getprivate"></a>Просмотр сведений о частном пиринге Azure

Для получения сведений о конфигурации можно использовать следующий пример кода:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePrivatePeering",
  "ipv6PeeringConfig": null,
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePrivatePeering",
  "peerAsn": 7671,
  "peeringType": "AzurePrivatePeering",
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

### <a name="to-update-azure-private-peering-configuration"></a><a name="updateprivate"></a>Обновление конфигурации частного пиринга Azure

С помощью следующего примера можно обновить любую часть конфигурации. В приведенном ниже примере значение идентификатора виртуальной локальной сети для канала изменяется со 100 на 500.

```azurecli
az network express-route peering update --vlan-id 500 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="to-delete-microsoft-peering"></a><a name="deletemsft"></a>Удаление пиринга Майкрософт

Для удаления конфигурации пиринга выполните следующий пример кода:

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name MicrosoftPeering
```

### <a name="to-delete-azure-private-peering"></a><a name="deleteprivate"></a>Удаление частного пиринга Azure

Для удаления конфигурации пиринга выполните следующий пример кода:

> [!WARNING]
> Перед запуском этого примера обязательно убедитесь, что все виртуальные сети и подключения Global Reach к ExpressRoute удалены. 
> 
> 

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

## <a name="next-steps"></a>Следующие шаги

После настройки частного пиринга Azure вы можете связать виртуальные сети с каналом. Дополнительные сведения см. в следующей статье: 

> [!div class="nextstepaction"]
> [Связывание виртуальной сети с каналом ExpressRoute](howto-linkvnet-cli.md)
