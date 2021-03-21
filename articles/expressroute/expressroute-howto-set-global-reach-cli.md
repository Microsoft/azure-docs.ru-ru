---
title: 'Azure ExpressRoute: Настройка Global Reach ExpressRoute: CLI'
description: Узнайте, как связать каналы ExpressRoute вместе, чтобы создать частную сеть между локальными сетями и включить Global Reach с помощью Azure CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 01/07/2021
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 55f1e4c030dc0e7c8d0011dcff0cc1c9f8fec06e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99260237"
---
# <a name="configure-expressroute-global-reach-by-using-the-azure-cli"></a>Настройка Global Reach ExpressRoute с помощью Azure CLI

Эта статья поможет вам настроить Azure ExpressRoute Global Reach путем использования Azure CLI. См. дополнительные сведения об [ExpressRoute Global Reach](expressroute-global-reach.md).
 
Перед началом настройки выполните следующие требования.

* Установите последнюю версию Azure CLI. Дополнительные сведения см. в руководстве по [установке Azure CLI](/cli/azure/install-azure-cli), а также в руководстве по [началу работы с Azure CLI](/cli/azure/get-started-with-azure-cli).
* Ознакомьтесь с [рабочими процессами](expressroute-workflows.md) подготовки канала ExpressRoute.
* Убедитесь, что каналы ExpressRoute подготовлены.
* Убедитесь, что частный пиринг Azure настроен для каналов ExpressRoute.  

### <a name="sign-in-to-your-azure-account"></a>Вход в учетную запись Azure

Чтобы начать настройку, войдите в свою учетную запись Azure. Следующая команда открывает браузер по умолчанию, в котором отображается запрос на введение учетных данных для входа в учетную запись Azure.  

```azurecli
az login
```

Если у вас есть несколько подписок Azure, проверьте подписки для этой учетной записи.

```azurecli
az account list
```

Укажите подписку, которую нужно использовать.

```azurecli
az account set --subscription <your subscription ID>
```

### <a name="identify-your-expressroute-circuits-for-configuration"></a>Определение каналов ExpressRoute для конфигурации

Можно включить Global Reach ExpressRoute между любыми двумя каналами ExpressRoute. Эти цепи должны находиться в поддерживаемых странах и регионах и были созданы в разных расположениях пиринга. Если ваша подписка владеет обеими цепями, вы можете выбрать одну из них для запуска конфигурации. Однако если две цепи находятся в разных подписках Azure, необходимо создать ключ авторизации из одной из каналов. Используя ключ авторизации, созданный на основе первого канала, можно включить Global Reach во втором канале.

## <a name="enable-connectivity-between-your-on-premises-networks"></a>Настройка подключения между локальными сетями

При выполнении команды для настройки подключения следует учитывать следующие требования для значений параметров:

* Параметр *peer-circuit* должен представлять полный идентификатор ресурса. Пример:

  > /субскриптионс/{your_subscription_id}/Ресаурцеграупс/{your_resource_group}/Провидерс/Микрософт.Нетворк/експрессраутеЦиркуитс/{your_circuit_name}/peerings/AzurePrivatePeering

* *префикс адреса* должен быть подсетью IPv4 "/29" (например, "10.0.0.0/29"). Мы используем IP-адреса в этой подсети, чтобы установить подключение между двумя каналами ExpressRoute. Вы не можете использовать адреса в этой подсети в виртуальных сетях Azure или в локальных сетях.

Запустите следующую команду CLI для подключения двух каналов ExpressRoute.

```azurecli
az network express-route peering connection create -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName> --peer-circuit <Circuit2ResourceID> --address-prefix <__.__.__.__/29>
```

Выходные данные CLI выглядят следующим образом:

```output
{
  "addressPrefix": "<__.__.__.__/29>",
  "authorizationKey": null,
  "circuitConnectionStatus": "Connected",
  "etag": "W/\"48d682f9-c232-4151-a09f-fab7cb56369a\"",
  "expressRouteCircuitPeering": {
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit1Name>/peerings/AzurePrivatePeering",
    "resourceGroup": "<ResourceGroupName>"
  },
  "id": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit1Name>/peerings/AzurePrivatePeering/connections/<ConnectionName>",
  "name": "<ConnectionName>",
  "peerExpressRouteCircuitPeering": {
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/<Circuit2ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit2Name>/peerings/AzurePrivatePeering",
    "resourceGroup": "<Circuit2ResourceGroupName>"
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "<ResourceGroupName>",
  "type": "Microsoft.Network/expressRouteCircuits/peerings/connections"
}
```

После завершения этой операции должно появиться подключение между локальными сетями с обеих сторон через два канала ExpressRoute.

## <a name="enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions"></a>Настройка подключения между каналами ExpressRoute в разных подписках Azure

Если два канала находятся в разных подписках Azure, вам потребуется выполнить авторизацию. В следующей конфигурации вы создаете авторизацию в подписке канала 2. Затем вы передаете ключ авторизации в цепь 1.

1. Создайте ключ авторизации.

   ```azurecli
   az network express-route auth create --circuit-name <Circuit2Name> -g <Circuit2ResourceGroupName> -n <AuthorizationName>
   ```

   Выходные данные CLI выглядят следующим образом:

   ```output
   {
     "authorizationKey": "<authorizationKey>",
     "authorizationUseStatus": "Available",
     "etag": "W/\"cfd15a2f-43a1-4361-9403-6a0be00746ed\"",
     "id": "/subscriptions/<SubscriptionID>/resourceGroups/<Circuit2ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit2Name>/authorizations/<AuthorizationName>",
     "name": "<AuthorizationName>",
     "provisioningState": "Succeeded",
     "resourceGroup": "<Circuit2ResourceGroupName>",
     "type": "Microsoft.Network/expressRouteCircuits/authorizations"
   }
   ```

1. Запишите идентификатор ресурса и ключ авторизации для канала 2.

1. Выполните следующую команду для канала 1, передав идентификатор ресурса канала 2 и ключ авторизации.

   ```azurecli
   az network express-route peering connection create -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName> --peer-circuit <Circuit2ResourceID> --address-prefix <__.__.__.__/29> --authorization-key <authorizationKey>
   ```

После завершения этой операции должно появиться подключение между локальными сетями с обеих сторон через два канала ExpressRoute.

## <a name="get-and-verify-the-configuration"></a>Получение и проверка конфигурации

Используйте следующую команду для проверки конфигурации на канале, на котором она создана (в приведенном выше примере это канал 1).

```azurecli
az network express-route show -n <CircuitName> -g <ResourceGroupName>
```

В выходных данных CLI вы увидите *CircuitConnectionStatus*. Так вы узнаете, установлено ли подключение между двумя каналами: на основе состояния Connected (Подключено) или Disconnected (Отключено). 

## <a name="disable-connectivity-between-your-on-premises-networks"></a>Отключение подключения между локальными сетями

Чтобы отключить подключение, выполните следующую команду на канале, на котором создана конфигурация (в предыдущем примере это канал 1).

```azurecli
az network express-route peering connection delete -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName>
```

Используйте команду ```show```, чтобы проверить состояние.

После завершения этой операции вы потеряете подключение к локальным сетям через каналы ExpressRoute.

## <a name="next-steps"></a>Дальнейшие действия

* [Получение дополнительных сведений об ExpressRoute Global Reach](expressroute-global-reach.md).
* [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Подключение виртуальной сети к каналу ExpressRoute](expressroute-howto-linkvnet-arm.md)
