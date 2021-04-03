---
title: Краткое руководство. Создание и изменение канала ExpressRoute в Azure CLI
description: В этой статье показано, как создать, подготовить, проверить, обновить, удалить и отозвать канал ExpressRoute с помощью Azure CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: quickstart
ms.date: 10/05/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: a1d50c3f8f94fbfd7dbcb9b25e051b7f2951c518
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91969098"
---
# <a name="quickstart-create-and-modify-an-expressroute-circuit-using-azure-cli"></a>Краткое руководство. Создание и изменение канала ExpressRoute с помощью Azure CLI

В этом кратком руководстве описано, как создать канал Azure ExpressRoute с помощью интерфейса командной строки (CLI), а также показано, как проверить состояние, обновить или удалить и отозвать канал.

## <a name="prerequisites"></a>Предварительные требования

* Изучите [предварительные требования](expressroute-prerequisites.md) и [рабочие процессы](expressroute-workflows.md), прежде чем приступить к настройке.
* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Установите последнюю версию команд интерфейса командной строки (версии 2.0 или более позднюю). См. дополнительные сведения об [установке Azure CLI 2.0](/cli/azure/install-azure-cli) и [начале работы с Azure CLI 2.0](/cli/azure/get-started-with-azure-cli).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Создание и предоставление канала ExpressRoute

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.

Чтобы начать настройку, войдите в свою учетную запись Azure. Если вы открыли Cloud Shell с помощью кнопки "Попробовать", вход выполняется автоматически. Для подключения используйте следующие примеры:

```azurecli-interactive
az login
```

Просмотрите подписки учетной записи.

```azurecli-interactive
az account list
```

Выберите подписку, для которой требуется создать канал ExpressRoute.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

### <a name="get-the-list-of-supported-providers-locations-and-bandwidths"></a>Получение списка поддерживаемых поставщиков, расположений и значений пропускной способности

Перед созданием канала ExpressRoute потребуется список поддерживаемых поставщиков услуг подключения, расположений и вариантов пропускной способности. Команда `az network express-route list-service-providers` в CLI возвращает эти сведения, которые будут использоваться в последующих шагах.

```azurecli-interactive
az network express-route list-service-providers
```

Ответ будет выглядеть примерно так:

```output
[
  {
    "bandwidthsOffered": [
      {
        "offerName": "50Mbps",
        "valueInMbps": 50
      },
      {
        "offerName": "100Mbps",
        "valueInMbps": 100
      },
      {
        "offerName": "200Mbps",
        "valueInMbps": 200
      },
      {
        "offerName": "500Mbps",
        "valueInMbps": 500
      },
      {
        "offerName": "1Gbps",
        "valueInMbps": 1000
      },
      {
        "offerName": "2Gbps",
        "valueInMbps": 2000
      },
      {
        "offerName": "5Gbps",
        "valueInMbps": 5000
      },
      {
        "offerName": "10Gbps",
        "valueInMbps": 10000
      }
    ],
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/expressRouteServiceProviders/",
    "location": null,
    "name": "AARNet",
    "peeringLocations": [
      "Melbourne",
      "Sydney"
    ],
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "tags": null,
    "type": "Microsoft.Network/expressRouteServiceProviders"
  },
```

Проверьте, указан ли в ответе ваш поставщик услуг подключения. Запишите следующие сведения, которые потребуются при создании канала:

* Имя
* PeeringLocations
* BandwidthsOffered

Теперь все готово к созданию канала ExpressRoute.

### <a name="create-an-expressroute-circuit"></a>Создание канала ExpressRoute

> [!IMPORTANT]
> Выставление счетов за использование ExpressRoute начинается после получения ключа службы. Выполните эту операцию, когда поставщик услуг подключения будет готов предоставить канал.
>
>

Перед созданием канала ExpressRoute необходимо создать группу ресурсов (если вы этого еще не сделали) Чтобы создать группу ресурсов, выполните следующую команду:

```azurecli-interactive
az group create -n ExpressRouteResourceGroup -l "West US"
```

В приведенном ниже примере показано, как создать канал ExpressRoute со скоростью 200 Мбит/с каналом через Equinix в Кремниевой долине. Если вы используете другой поставщик и другие параметры, подставьте в запрос соответствующие данные.

Убедитесь, что указаны правильный уровень SKU и семейство SKU:

* Уровень SKU определяет ценовую категорию канала ExpressRoute: [Локальный](expressroute-faqs.md#expressroute-local), "Стандартный" или [Премиум](expressroute-faqs.md#expressroute-premium). Вы можете указать *Локальный*, *"Стандартный" или *Премиум*. Изменить номер SKU *Стандартный / Премиум* на *Локальный* нельзя.
* Семейство SKU определяет тип выставления счетов. Выберите *MeteredData* для тарифного плана с оплатой за трафик или *UnlimitedData* для безлимитного тарифного плана. Тип выставления счетов можно изменить с *MeteredData* на *UnlimitedData*, но не с *UnlimitedData* на *MeteredData*. Для канала уровня *Локальный* используется только *UnlimitedData*.


Выставление счетов за использование ExpressRoute начинается после получения ключа службы. Ниже приведен пример запроса нового ключа службы:

```azurecli-interactive
az network express-route create --bandwidth 200 -n MyCircuit --peering-location "Silicon Valley" -g ExpressRouteResourceGroup --provider "Equinix" -l "West US" --sku-family MeteredData --sku-tier Standard
```

Ответ будет содержать ключ службы.

### <a name="list-all-expressroute-circuits"></a>Получение списка всех каналов ExpressRoute

Чтобы получить список всех созданных вами каналов ExpressRoute, выполните команду `az network express-route list`. Вы можете получить эти сведения в любое время с помощью этой команды. Чтобы получить список всех каналов, сделайте вызов без параметров.

```azurecli-interactive
az network express-route list
```

Ваш ключ службы внесен в поле *ServiceKey* ответа.

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
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

Подробное описание всех параметров можно получить, выполнив команду с параметром -h.

```azurecli-interactive
az network express-route list -h
```

### <a name="send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>Отправка ключа службы поставщику услуг подключения для подготовки

Параметр ServiceProviderProvisioningState предоставляет сведения о текущем состоянии подготовки на стороне поставщика услуг. Параметр Status предоставляет состояние на стороне Майкрософт. Дополнительные сведения см. в разделе [Состояния подготовки канала ExpressRoute](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Вновь созданный канал ExpressRoute будет имеет следующее состояние:

```output
"serviceProviderProvisioningState": "NotProvisioned"
"circuitProvisioningState": "Enabled"
```

Канал, включаемый поставщиком услуг подключения, переходит в следующее состояние:

```output
"serviceProviderProvisioningState": "Provisioning"
"circuitProvisioningState": "Enabled"
```

Используемый канал ExpressRoute должен находиться в следующем состоянии:

```output
"serviceProviderProvisioningState": "Provisioned"
"circuitProvisioningState": "Enabled
```

### <a name="periodically-check-the-status-and-the-state-of-the-circuit-key"></a>Периодическая проверка состояния и статуса ключа канала

Проверяя состояние ключа службы, вы можете узнать, когда поставщик подготовит ваш канал. После настройки канала значение параметра *ServiceProviderProvisioningState* изменится на *Provisioned*, как показано в примере ниже.

```azurecli-interactive
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
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

### <a name="create-your-routing-configuration"></a>Создание конфигурации маршрутизации

Пошаговые инструкции по созданию и изменению пиринга каналов см. в статье [Создание и изменение маршрутизации для канала ExpressRoute](howto-routing-cli.md).

> [!IMPORTANT]
> Эти инструкции распространяются только на каналы от поставщиков, предоставляющих услуги подключения второго уровня. Если ваш поставщик услуг подключения предлагает услуги третьего уровня (обычно это IPVPN, например MPLS), то он возьмет на себя настройку маршрутизации и управление ею.
>
>

### <a name="link-a-virtual-network-to-an-expressroute-circuit"></a>Связывание виртуальной сети с каналом ExpressRoute

Теперь свяжите виртуальную сеть с каналом ExpressRoute. Дополнительные сведения см. в статье [Connect a virtual network to an ExpressRoute circuit using CLI](howto-linkvnet-cli.md) (Подключение виртуальной сети к каналу ExpressRoute с помощью CLI).

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Изменение канала ExpressRoute

Некоторые свойства канала ExpressRoute можно изменить, не повлияв на подключение. Можно вносить следующие изменения без простоя:

* Включать и отключать надстройку ExpressRoute "Премиум" для канала ExpressRoute. Изменение SKU с уровня *Стандартный / Премиум* на *Локальный* не поддерживается.
* Увеличивать пропускную способность канала ExpressRoute, если в порту имеется доступная емкость. Но снижение уровня пропускной способности канала не поддерживается.
* Перейти с тарифного плана с оплатой за трафик на безлимитный тарифный план. Но переход с безлимитного тарифного плана на тарифный план с оплатой за трафик не поддерживается.
* Параметр *Allow Classic Operations*(Разрешить классические операции) можно включать и отключать.

Дополнительные сведения об ограничениях см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).

### <a name="to-enable-the-expressroute-premium-add-on"></a>Включение надстройки ExpressRoute "Премиум"

Вы можете включить надстройку ExpressRoute "Премиум" для имеющегося канала ExpressRoute с помощью следующей команды:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Premium
```

Теперь для канала включены функции надстройки ExpressRoute "Премиум". С момента успешного выполнения команды мы начнем выставлять счета по тарифам надстройки уровня "Премиум".

### <a name="to-disable-the-expressroute-premium-add-on"></a>Отключение надстройки ExpressRoute "Премиум"

> [!IMPORTANT]
> Операция может завершиться ошибкой, если использовать больше ресурсов, чем разрешено для канала "Стандартный".
>
>

Перед отключением надстройки ExpressRoute "Премиум" ознакомьтесь со следующими требованиями:

* Прежде чем переходить с канала "Премиум" на "Стандартный", убедитесь, что к каналу привязано менее 10 виртуальных сетей. Если этого не сделать, запрос на обновление завершится ошибкой и мы будем выставлять вам счета по тарифам ценовой категории "Премиум".
* Все связи с виртуальными сетями в других геополитических регионах необходимо разорвать. Если этого не сделать, запрос на обновление завершится ошибкой, и мы будем выставлять вам счета по тарифам уровня "Премиум".
* Для частного пиринга таблица маршрутов должна содержать менее 4000 маршрутов. Если таблица маршрутов содержит больше 4000 маршрутов, сеанс BGP будет прерван. Его можно будет снова включить, когда количество объявленных префиксов станет меньше 4000.

Вы можете отключить надстройку ExpressRoute "Премиум" для имеющегося канала с помощью следующей команды:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Standard
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>Обновление пропускной способности канала ExpressRoute

Параметры пропускной способности, поддерживаемые для вашего поставщика, приведены в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md). Можно выбрать любой размер, больший, чем размер существующего канала.

> [!IMPORTANT]
> Может потребоваться заново создать канал ExpressRoute, если существующий порт не обеспечивает достаточную емкость. Канал невозможно обновить, если в его расположении нет доступной дополнительной емкости.
>
> Уменьшить пропускную способность канала ExpressRoute без прерывания его работы нельзя. Для снижения пропускной способности нужно будет отозвать канал ExpressRoute и повторно подготовить новый канал ExpressRoute.
>

Выберите необходимый размер и используйте следующую команду для изменения размера канала:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --bandwidth 1000
```

Размер канала будет изменен на стороне Майкрософт. Затем свяжитесь с поставщиком услуг подключения и попросите обновить конфигурации с его стороны в соответствии с этим изменением. Когда вы сообщите поставщику услуг об изменении размера канала, мы будем выставлять счета за обновленную пропускную способность.

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>Перевод SKU с измеряемых на неограниченные данные

Изменить SKU канала ExpressRoute можно, используя следующий пример:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-family UnlimitedData
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>Управление доступом к классической среде и среде диспетчера ресурсов

Инструкции см. в статье [Перемещение каналов ExpressRoute из классической модели развертывания в модель Resource Manager](expressroute-howto-move-arm.md).

## <a name="deprovisioning-an-expressroute-circuit"></a><a name="delete"></a>Отзыв канала ExpressRoute

Чтобы отменить подготовку и удалить канал ExpressRoute, ознакомьтесь со следующими требованиями:

* Связь между ExpressRoute и всеми виртуальными сетями необходимо разорвать. Если операция завершится ошибкой, проверьте, не привязаны ли к каналу какие-либо виртуальные сети.
* Если подготовка поставщика услуг канала ExpressRoute находится в состоянии **Идет подготовка** или **Подготовлено** то свяжитесь с поставщиком услуг, чтобы отозвать канал с его стороны. Мы будем резервировать ресурсы и выставлять вам счета до тех пор, пока поставщик услуг не завершит отзыв канала и не отправит нам соответствующее уведомление.
* Когда поставщик услуг отзовет канал (состояние подготовки поставщика услуг изменится на **Не подготовлено**), вы сможете удалить этот канал. Выставление счетов за этот канал будет приостановлено.

## <a name="clean-up-resources"></a>Очистка ресурсов

Для удаления канала ExpressRoute выполните следующую команду:

```azurecli-interactive
az network express-route delete  -n MyCircuit -g ExpressRouteResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Когда вы вместе с поставщиком создадите и подготовите канал, перейдите к следующему шагу, чтобы настроить пиринг:

> [!div class="nextstepaction"]
> [Создание и изменение маршрутизации для канала ExpressRoute](howto-routing-cli.md)
