---
title: 'Azure ExpressRoute: настройка ExpressRoute Direct для CLI'
description: Узнайте, как использовать Azure CLI, чтобы настроить Azure ExpressRoute Direct для прямого подключения к глобальной сети Майкрософт.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/14/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9f35a698510f8637c3fe66528e6d64e5cd87b693
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553827"
---
# <a name="configure-expressroute-direct-by-using-the-azure-cli"></a>Настройка ExpressRoute Direct с помощью интерфейса командной строки Azure

ExpressRoute Direct дает возможность подключения непосредственно в глобальной сети корпорации Майкрософт в стратегически распределенных по всему миру расположениях пиринга. Дополнительные сведения см. в разделе [About ExpressRoute Direct](expressroute-erdirect-about.md) (Общие сведения о подключении ExpressRoute Direct).

## <a name="before-you-begin"></a>Подготовка к работе

Перед использованием ExpressRoute Direct необходимо зарегистрировать подписку. Перед использованием ExpressRoute Direct необходимо зарегистрировать подписку. Для регистрации выполните следующие действия с помощью Azure PowerShell.
1.  Выполните вход в свою учетную запись Azure и выберите план подписки, которую планируете зарегистрировать.

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

2. Зарегистрируйте подписку для общедоступной предварительной версии с помощью следующей команды.
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowExpressRoutePorts -ProviderNamespace Microsoft.Network
    ```

После регистрации убедитесь, что поставщик ресурсов **Microsoft.Network** зарегистрирован в вашей подписке. Регистрация поставщика ресурсов настраивает подписку для работы с поставщиком ресурсов.

## <a name="create-the-resource"></a><a name="resources"></a>Создание ресурса

1. Войдите в Azure и выберите подписку, которая включает ExpressRoute. Ресурс ExpressRoute Direct и каналы ExpressRoute должны размещаться в одной подписке. Выполните следующие команды в Azure CLI:

   ```azurecli
   az login
   ```

   Просмотрите подписки учетной записи. 

   ```azurecli
   az account list 
   ```

   Выбор подписки для создания канала ExpressRoute:

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```

2. Повторно зарегистрируйте подписку в Microsoft.Network для доступа к API-интерфейсам expressrouteportslocation и expressrouteport

   ```azurecli
   az provider register --namespace Microsoft.Network
   ```
3. Вывод списка всех расположений, где поддерживается ExpressRoute Direct:
    
   ```azurecli
   az network express-route port location list
   ```

   **Пример выходных данных**
  
   ```output
   [
   {
    "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
    "location": null,
    "name": "Equinix-Ashburn-DC2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "1950 N. Stemmons Freeway, Suite 1039A, DA3, Dallas, TX 75207",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Dallas-DA3",
    "location": null,
    "name": "Equinix-Dallas-DA3",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "111 8th Avenue, New York, NY 10011",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-New-York-NY5",
    "location": null,
    "name": "Equinix-New-York-NY5",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "11 Great Oaks Blvd, SV1, San Jose, CA 95119",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-SV1",
    "location": null,
    "name": "Equinix-San-Jose-SV1",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "2001 Sixth Ave., Suite 350, SE2, Seattle, WA 98121",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Seattle-SE2",
    "location": null,
    "name": "Equinix-Seattle-SE2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ]
   ```
4. Проверка наличия доступной пропускной способности в любом из расположений, полученных на предыдущем шаге:

   ```azurecli
   az network express-route port location show -l "Equinix-Ashburn-DC2"
   ```

   **Пример выходных данных**

   ```output
   {
   "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
   "availableBandwidths": [
    {
      "offerName": "100 Gbps",
      "valueInGbps": 100
    }
   ],
   "contact": "support@equinix.com",
   "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
   "location": null,
   "name": "Equinix-Ashburn-DC2",
   "provisioningState": "Succeeded",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ```
5. Создайте ресурс ExpressRoute Direct на основе расположений, выбранных на предыдущих шагах.

   ExpressRoute Direct поддерживает инкапсуляцию как QinQ, так и Dot1Q. Если вы выберете QinQ, каждому каналу ExpressRoute динамически назначается S-тег и каналы будут уникальными в пределах ресурса ExpressRoute Direct. Каждый C-тег в канале должен быть уникальным в пределах канала, но не для всего ресурса ExpressRoute Direct.  

   Если вы выберете инкапсуляция Dot1Q, необходимо обеспечить уникальность C-тега (VLAN) в пределах всего ресурса ExpressRoute Direct.  

   > [!IMPORTANT]
   > Для ExpressRoute Direct можно выбрать только один тип инкапсуляции. Вы не сможете изменить тип инкапсуляции после создания ресурса ExpressRoute Direct.
   > 
 
   ```azurecli
   az network express-route port create -n $name -g $RGName --bandwidth 100 gbps  --encapsulation QinQ | Dot1Q --peering-location $PeeringLocationName -l $AzureRegion 
   ```

   > [!NOTE]
   > Атрибуту **Encapsulation** (Инкапсуляция) также можно присвоить значение **Dot1Q**. 
   >

   **Пример выходных данных**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "02ee21fe-4223-4942-a6bc-8d81daabc94f",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }  
   ```

## <a name="generate-the-letter-of-authorization-loa"></a><a name="resources"></a>Создание письма авторизации (LOA)

Введите недавно созданное имя ресурса ExpressRoute Direct, имя группы ресурсов и имя клиента, куда следует осуществить запись LOA и (необязательно) задайте расположение файла для хранения документа. Если на путь к файлу нет ссылки, документ будет загружен в текущий каталог.

```azurecli
az network express-route port generate-loa -n Contoso-Direct -g Contoso-Direct-rg --customer-name Contoso --destination C:\Users\SampleUser\Downloads\LOA.pdf
```

## <a name="change-adminstate-for-links"></a><a name="state"></a>Изменение AdminState для ссылок

Этот процесс позволяет провести тестирование уровня 1. Убедитесь, что все кроссовые подключения правильно подключены к маршрутизаторам на первичных и вторичных портах.

1. Для ссылок выберите режим **Enabled** (Включено). Повторите этот шаг для каждой ссылки, установив для всех значение **Enabled** (Включено).

   Ссылки [0] — это основной порт, а ссылки [1] — это дополнительный порт.

   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[0].adminState="Enabled"
   ```
   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[1].adminState="Enabled"
   ```
   **Пример выходных данных**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "<resourceGUID>",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }
   ```

   Чтобы выключить порты, используйте эту же процедуру с параметром `AdminState = "Disabled"`.

## <a name="create-a-circuit"></a><a name="circuit"></a>Создание цепи

По умолчанию в подписке, в которой есть ресурс ExpressRoute Direct, можно создать до 10 каналов. Это ограничение можно увеличить через службу поддержки Майкрософт. Отслеживание подготовленной и используемой пропускной способности вам нужно выполнять самостоятельно. Подготовленная пропускная способность вычисляется как сумма пропускной способности всех каналов в ресурсе ExpressRoute Direct. Используемая пропускная способность описывает фактическое потребление базовых физических интерфейсов.

Вы можете добавить дополнительную пропускную способность в ExpressRoute Direct только для тех сценариев, которые описаны в этой статье. Поддерживаются варианты пропускной способности 40 Гбит/с и 100 Гбит/с.

Параметр **SkuTier** может иметь значения "Local", "Standard" или "Premium".

Параметр **SkyFamily** имеет только значение "MeteredData". Значение "unlimited" не поддерживается в ExpressRoute Direct.

Создайте канал в ресурсе ExpressRoute Direct:

  ```azurecli
  az network express-route create --express-route-port "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct" -n "Contoso-Direct-ckt" -g "Contoso-Direct-rg" --sku-family MeteredData --sku-tier Standard --bandwidth 100 Gbps
  ```

  Также поддерживаются такие значения пропускной способности: 5 Гбит/с, 10 Гбит/с и 40 Гбит/с.

  **Пример выходных данных**

  ```output
  {
  "allowClassicOperations": false,
  "allowGlobalReach": false,
  "authorizations": [],
  "bandwidthInGbps": 100.0,
  "circuitProvisioningState": "Enabled",
  "etag": "W/\"<etagnumber>\"",
  "expressRoutePort": {
    "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
    "resourceGroup": "Contoso-Direct-rg"
  },
  "gatewayManagerEtag": "",
  "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRouteCircuits/ERDirect-ckt-cli",
  "location": "westus",
  "name": "ERDirect-ckt-cli",
  "peerings": [],
  "provisioningState": "Succeeded",
  "resourceGroup": "Contoso-Direct-rg",
  "serviceKey": "<serviceKey>",
  "serviceProviderNotes": null,
  "serviceProviderProperties": null,
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "MeteredData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "stag": null,
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits"
  }  
  ```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [сведений о подключении ExpressRoute Direct](expressroute-erdirect-about.md).
