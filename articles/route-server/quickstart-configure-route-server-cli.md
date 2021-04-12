---
title: Краткое руководство. Создание и настройка сервера маршрутизации с помощью Azure CLI
description: В этом кратком руководстве показано, как создать и настроить сервер маршрутизации с помощью Azure CLI.
services: route-server
author: duongau
ms.service: route-server
ms.topic: quickstart
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: e9c583db7493afc04b2c66553801f62d364b0a80
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103419614"
---
# <a name="quickstart-create-and-configure-route-server-using-azure-cli"></a>Краткое руководство. Создание и настройка сервера маршрутизации с помощью Azure CLI 

Эта статья поможет вам настроить сервер маршрутизации Azure для пиринга с виртуальными сетевыми модулями (NVA) в виртуальной сети с помощью Azure CLI. Сервер маршрутизации Azure будет анализировать маршруты от NVA и программировать их для виртуальных машин в виртуальной сети. Он также будет объявлять маршруты виртуальной сети к NVA. См. дополнительные сведения о [сервере маршрутизации Azure](overview.md).

> [!IMPORTANT]
> Сервер маршрутизации Azure (предварительная версия) сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

##  <a name="prerequisites"></a>Предварительные требования 

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
* Убедитесь, что у вас есть последняя версия Azure CLI, или перейдите к Azure Cloud Shell на портале. 
* Ознакомьтесь с [ограничениями службы для сервера маршрутизации Azure](route-server-faq.md#limitations). 

##  <a name="create-a-route-server"></a>Создание сервера маршрутизации 

###  <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку. 

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

### <a name="create-a-resource-group-and-virtual-network"></a>Создание группы ресурсов и виртуальной сети 

Перед созданием сервера маршрутизации Azure нужно создать виртуальную сеть для размещения развертывания. Чтобы создать группу ресурсов и виртуальную сеть, выполните приведенную ниже команду. Если у вас уже есть виртуальная сеть, можно перейти к следующему разделу.

```azurecli-interactive
az group create -n "RouteServerRG" -l "westus" 
az network vnet create -g "RouteServerRG" -n "myVirtualNetwork" --address-prefix "10.0.0.0/16" 
``` 

### <a name="add-a-subnet"></a>Добавление подсети 

1. Добавьте подсеть с именем *RouteServerSubnet*, чтобы развернуть в ней сервер маршрутизации Azure. Эта подсеть является выделенной подсетью только для сервера маршрутизации Azure. Для подсети RouteServerSubnet требуется префикс /27 или более короткий (например, /26 или /25), иначе при добавлении сервера маршрутизации Azure появится сообщение об ошибке.

    ```azurecli-interactive 
    az network vnet subnet create -g "RouteServerRG" --vnet-name "myVirtualNetwork" --name "RouteServerSubnet" --address-prefix "10.0.0.0/24"  
    ``` 

1. Получите идентификатор RouteServerSubnet. Чтобы просмотреть идентификатор ресурса всех подсетей в виртуальной сети, выполните следующую команду: 

    ```azurecli-interactive 
    $subnet_id = $(az network vnet subnet show -n "RouteServerSubnet" --vnet-name "myVirtualNetwork" -g "RouteServerRG" --query id -o tsv) 
    ``` 

Идентификатор RouteServerSubnet выглядит следующим образом: 

`/subscriptions/<subscriptionID>/resourceGroups/RouteServerRG/providers/Microsoft.Network/virtualNetworks/myVirtualNetwork/subnets/RouteServerSubnet`

## <a name="create-the-route-server"></a>Создание сервера маршрутизации 

Создайте сервер маршрутизации, выполнив следующую команду: 

```azurecli-interactive
az network routeserver create -n "myRouteServer" -g "RouteServerRG" --hosted-subnet $subnet_id  
``` 

Расположение должно совпадать с расположением виртуальной сети. HostedSubnet — это идентификатор RouteServerSubnet, полученный в предыдущем разделе. 

## <a name="create-peering-with-an-nva"></a>Создание пиринга с NVA 

Чтобы установить пиринг между сервером маршрутизации и NVA, выполните следующую команду: 

```azurecli-interactive 

az network routeserver peering create --routeserver-name "myRouteServer" -g "RouteServerRG" --peer-ip "nva_ip" --peer-asn "nva_asn" -n "NVA1_name" 

``` 

nva_ip — это IP-адрес виртуальной сети, назначенный NVA. nva_asn — это номер ASN, настроенный в NVA. Номер ASN может быть любым 16-разрядным числом, отличающимся от значений в диапазоне 65515–65520. Этот диапазон ASN зарезервирован корпорацией Майкрософт. 

Чтобы настроить пиринг с разными NVA или другим экземпляром того же NVA для обеспечения избыточности, выполните следующую команду:

```azurecli-interactive 

az network routeserver peering create --routeserver-name "myRouteServer" -g "RouteServerRG" --peer-ip "nva_ip" --peer-asn "nva_asn" -n "NVA2_name" 
``` 

## <a name="complete-the-configuration-on-the-nva"></a>Завершение настройки NVA 

Чтобы завершить настройку NVA и включить сеансы BGP, требуется IP-адрес и номер ASN сервера маршрутизации Azure. Эти сведения можно получить, выполнив следующую команду: 

```azurecli-interactive 
az network routeserver show -g "RouteServerRG" -n "myRouteServer" 
``` 

В выходных данных будут содержаться указанные ниже сведения. 

```azurecli-interactive 
RouteServerAsn  : 65515 

RouteServerIps  : {10.5.10.4, 10.5.10.5}  "virtualRouterAsn": 65515, 

  "virtualRouterIps": [ 

    "10.0.0.4", 

    "10.0.0.5" 

  ], 

``` 

## <a name="configure-route-exchange"></a>Настройка обмена маршрутами 

Если у вас есть шлюз ExpressRoute и VPN-шлюз Azure в одной и той же виртуальной сети и вы хотите, чтобы они могли обмениваться маршрутами, можно включить обмен маршрутами на сервере маршрутизации Azure.

> [!IMPORTANT]
> Для новых развертываний перед созданием сервера маршрутизации Azure необходимо создать VPN-шлюз Azure. Иначе развертывание VPN-шлюза Azure завершится ошибкой.
> 

1. Чтобы включить обмен маршрутами между сервером маршрутизации Azure и шлюзами, выполните следующую команду:

```azurecli-interactive 
az network routeserver update -g "RouteServerRG" -n "myRouteServer" --allow-b2b-traffic true 

``` 

2. Чтобы отключить обмен маршрутами между сервером маршрутизации Azure и шлюзами, выполните следующую команду:

```azurecli-interactive
az network routeserver update -g "RouteServerRG" -n "myRouteServer" --allow-b2b-traffic false 
``` 

## <a name="troubleshooting"></a>Устранение неполадок 

Вы можете просмотреть маршруты, объявленные и полученные сервером маршрутизации Azure, выполнив следующую команду:

```azurecli-interactive 
az network routeserver peering list-advertised-routes -g RouteServerRG --vrouter-name myRouteServer -n NVA1_name 
az network routeserver peering list-learned-routes -g RouteServerRG --vrouter-name myRouteServer -n NVA1_name 
``` 

## <a name="clean-up-resources"></a>Очистка ресурсов

Если сервер маршрутизации Azure больше не нужен, удалите пиринг BGP и сервер маршрутизации, выполнив следующие команды: 

1. Удалите пиринг BGP между сервером маршрутизации Azure и NVA, выполнив следующую команду:

```azurecli-interactive
az network routeserver peering delete --routeserver-name "myRouteServer" -g "RouteServerRG" -n "NVA2_name" 
``` 

2. Удалите сервер маршрутизации Azure, выполнив следующую команду: 

```azurecli-interactive 
az network routeserver delete -n "myRouteServer" -g "RouteServerRG" 
``` 

## <a name="next-steps"></a>Дальнейшие действия

Создав сервер маршрутизации Azure, вы можете продолжить изучение способов взаимодействия между сервером маршрутизации Azure с ExpressRoute и VPN-шлюзами: 

> [!div class="nextstepaction"]
> [Поддержка Azure ExpressRoute и VPN-шлюза Azure](expressroute-vpn-support.md)
 
