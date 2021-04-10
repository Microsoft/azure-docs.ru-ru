---
title: Краткое руководство. Создание и настройка сервера маршрутизации с помощью Azure PowerShell
description: В этом кратком руководстве показано, как создать и настроить сервер маршрутизации с помощью Azure PowerShell.
services: route-server
author: duongau
ms.service: route-server
ms.topic: quickstart
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: a3ab3a801872cc20b4e41bbff02ad6474c3bab8c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104655212"
---
# <a name="quickstart-create-and-configure-route-server-using-azure-powershell"></a>Краткое руководство. Создание и настройка сервера маршрутизации с помощью Azure PowerShell

Эта статья поможет вам настроить сервер маршрутизации Azure для пиринга с виртуальными сетевыми модулями (NVA) в виртуальной сети с помощью Azure PowerShell. Сервер маршрутизации Azure будет анализировать маршруты от NVA и программировать их для виртуальных машин в виртуальной сети. Он также будет объявлять маршруты виртуальной сети к NVA. См. дополнительные сведения о [сервере маршрутизации Azure](overview.md).

> [!IMPORTANT]
> Сервер маршрутизации Azure (предварительная версия) сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Убедитесь, что у вас есть последняя версия модулей PowerShell, или перейдите к Azure Cloud Shell на портале.
* Ознакомьтесь с [ограничениями службы для сервера маршрутизации Azure](route-server-faq.md#limitations).

## <a name="create-a-route-server"></a>Создание сервера маршрутизации

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.

[!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

### <a name="create-a-resource-group-and-virtual-network"></a>Создание группы ресурсов и виртуальной сети

Перед созданием сервера маршрутизации Azure нужно создать виртуальную сеть для размещения развертывания. Чтобы создать группу ресурсов и виртуальную сеть, выполните приведенную ниже команду. Если у вас уже есть виртуальная сеть, можно перейти к следующему разделу.

```azurepowershell-interactive
New-AzResourceGroup –Name "RouteServerRG” -Location “West US"
New-AzVirtualNetwork –ResourceGroupName "RouteServerRG" -Location "West US" -Name myVirtualNetwork –AddressPrefix 10.0.0.0/16
```

### <a name="add-a-subnet"></a>Добавление подсети

1. Добавьте подсеть с именем *RouteServerSubnet*, чтобы развернуть в ней сервер маршрутизации Azure. Эта подсеть является выделенной подсетью только для сервера маршрутизации Azure. Для подсети RouteServerSubnet требуется префикс /27 или более короткий (например, /26 или /25), иначе при добавлении сервера маршрутизации Azure появится сообщение об ошибке.

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork –Name "myVirtualNetwork" - ResourceGroupName "RouteServerRG"
    Add-AzVirtualNetworkSubnetConfig –Name "RouteServerSubnet" -AddressPrefix 10.0.0.0/24 -VirtualNetwork $vnet
    $vnet | Set-AzVirtualNetwork
    ```

1. Получите идентификатор RouteServerSubnet. Чтобы просмотреть идентификатор ресурса всех подсетей в виртуальной сети, выполните эту команду:

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork –Name "vnet_name" -ResourceGroupName "RouteServerRG"
    $vnet.Subnets
    ```

Идентификатор RouteServerSubnet выглядит следующим образом:

`/subscriptions/<subscriptionID>/resourceGroups/RouteServerRG/providers/Microsoft.Network/virtualNetworks/myVirtualNetwork/subnets/RouteServerSubnet`

## <a name="create-the-route-server"></a>Создание сервера маршрутизации

Создайте сервер маршрутизации, выполнив следующую команду:

```azurepowershell-interactive 
New-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG -Location "West US" -HostedSubnet "RouteServerSubnet_ID"
```

Расположение должно совпадать с расположением виртуальной сети. HostedSubnet — это идентификатор RouteServerSubnet, полученный в предыдущем разделе.

## <a name="create-peering-with-an-nva"></a>Создание пиринга с NVA

Чтобы установить пиринг BGP между сервером маршрутизации и NVA, выполните следующую команду:

```azurepowershell-interactive 
Add-AzRouteServerPeer -PeerName "myNVA" -PeerIp "nva_ip" -PeerAsn "nva_asn" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

nva_ip — это IP-адрес виртуальной сети, назначенный NVA. nva_asn — это номер ASN, настроенный в NVA. Номер ASN может быть любым 16-разрядным числом, отличающимся от значений в диапазоне 65515–65520. Этот диапазон ASN зарезервирован корпорацией Майкрософт.

Чтобы настроить пиринг с разными NVA или другим экземпляром того же NVA для обеспечения избыточности, выполните следующую команду:

```azurepowershell-interactive 
Add-AzRouteServerPeer -PeerName "NVA2_name" -PeerIp "nva2_ip" -PeerAsn "nva2_asn" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG 
```

## <a name="complete-the-configuration-on-the-nva"></a>Завершение настройки NVA

Чтобы завершить настройку NVA и включить сеансы BGP, требуется IP-адрес и номер ASN сервера маршрутизации Azure. Эти сведения можно получить, выполнив следующую команду:

```azurepowershell-interactive 
Get-AzRouteServer -RouterServerName myRouteServer -ResourceGroupName RouteServerRG
```

В выходных данных будут содержаться следующие сведения:

``` 
RouteServerAsn : 65515
RouteServerIps : {10.5.10.4, 10.5.10.5}
```

## <a name="configure-route-exchange"></a><a name = "route-exchange"></a>Настройка обмена маршрутами

Если у вас есть шлюз ExpressRoute и VPN-шлюз Azure в одной и той же виртуальной сети и вы хотите, чтобы они могли обмениваться маршрутами, можно включить обмен маршрутами на сервере маршрутизации Azure.

1. Чтобы включить обмен маршрутами между сервером маршрутизации Azure и шлюзами, выполните следующую команду:

```azurepowershell-interactive 
Update-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG -AllowBranchToBranchTraffic 
```

2. Чтобы отключить обмен маршрутами между сервером маршрутизации Azure и шлюзами, выполните следующую команду:

```azurepowershell-interactive 
Update-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

## <a name="troubleshooting"></a>Устранение неполадок

Вы можете просмотреть маршруты, объявленные и полученные сервером маршрутизации Azure, выполнив следующую команду:

```azurepowershell-interactive
Get-AzRouteServerPeerAdvertisedRoute
Get-AzRouteServerPeerLearnedRoute
```
## <a name="clean-up-resources"></a>Очистка ресурсов

Если сервер маршрутизации Azure больше не нужен, удалите пиринг BGP и сервер маршрутизации, выполнив следующие команды: 

1. Удалите пиринг BGP между сервером маршрутизации Azure и NVA, выполнив следующую команду:

```azurepowershell-interactive 
Remove-AzRouteServerPeer -PeerName "nva_name" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG 
```

2. Удалите сервер маршрутизации Azure, выполнив следующую команду:

```azurepowershell-interactive 
Remove-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

## <a name="next-steps"></a>Дальнейшие действия

Создав сервер маршрутизации Azure, вы можете продолжить изучение способов взаимодействия между сервером маршрутизации Azure с ExpressRoute и VPN-шлюзами: 

> [!div class="nextstepaction"]
> [Поддержка Azure ExpressRoute и VPN-шлюза Azure](expressroute-vpn-support.md)
