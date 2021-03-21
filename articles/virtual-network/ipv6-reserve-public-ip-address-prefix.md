---
title: Резервирование общедоступных IPv6-адресов и диапазонов адресов в виртуальной сети Azure
titlesuffix: Azure Virtual Network
description: Узнайте, как зарезервировать общедоступные IPv6-адреса и диапазоны адресов в виртуальной сети Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: ac79e1eb5c4f7448dc17804cd8aac3cba582497e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88509949"
---
# <a name="reserve-public-ipv6-address-prefix"></a>Зарезервируйте префикс общедоступного IPv6-адреса
Протокол IPv6 для виртуальной сети Azure (VNet) позволяет размещать приложения на платформе Azure с поддержкой адресации IPv6 и IPv4 как в виртуальной сети, так и с доступом в Интернет и из него. Помимо резервирования отдельных IPv6-адресов, можно зарезервировать непрерывные диапазоны IPv6-адресов Azure (называемых префиксом IP) для использования. В этой статье описывается создание общедоступных IP-адресов IPv6 и диапазонов адресов с помощью Azure PowerShell и CLI.


## <a name="create-a-single-reserved-ipv6-public-ip"></a>Создание одного зарезервированного общедоступного IP-адреса IPv6

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

Вы можете создать один зарезервированный (статический) общедоступный IP-адрес IPv6, используя Azure PowerShell с параметром [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress) следующим образом:

```azurepowershell
 $myOwnIPv6Address = New-AzPublicIpAddress `
 -name PIPv6_WestUS `
 -ResourceGroup MyRG `
 -Location "West US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6
 ```

### <a name="using-azure-cli"></a>Использование Azure CLI

 Вы можете создать один зарезервированный (статический) общедоступный IP-адрес IPv6 Azure CLI с помощью команды [AZ Network public-IP Create](/cli/azure/network/public-ip) , как показано ниже.

```azurecli
 az network public-ip create \
 --name dsPublicIP_v6 \
 --resource-group UpgradeInPlace_CLI_RG1 \
 --location WestUS \
 --sku Standard  \
 --allocation-method static  \
 --version IPv6
```

## <a name="create-a-reserved-ipv6-prefix-range"></a>Создание зарезервированного префикса IPv6 (диапазон)

Чтобы зарезервировать префикс IPv6, добавьте семейство IP-адресов IPv6 в ту же команду, которая использовалась для создания префиксов IPv4. Следующие команды создают префикс размера/125 (8 адресов IPv6).

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

Вы можете создать общедоступный IPv6-адрес, используя Azure CLI с помощью команды [AZ Network public-IP Create](/powershell/module/az.network/new-azpublicipprefix) , как показано ниже.
```azurepowershell
 $myOwnIPv6Prefix = New-AzPublicIpPrefix `
 -name IPv6PrefixWestUS `
 -ResourceGroupName MyRG `
 -Location "West US" `
 -Sku Standard `
 -IpAddressVersion IPv6 `
 -PrefixLength 125
```

### <a name="using-azure-cli"></a>Использование Azure CLI

Общедоступный IPv6-адрес можно создать с помощью Azure CLI следующим образом.

```azurecli
az network public-ip prefix create \
--name IPv6PrefixWestUS \
--resource-group MyRG \
--location WestUS \
--version IPv6 \
--length 125
```

## <a name="allocate-a-public-ip-address-from-a-reserved-ipv6-prefix"></a>Выделение общедоступного IP-адреса из зарезервированного префикса IPv6

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

 Вы создадите статический общедоступный IP-адрес IPv6 из зарезервированного префикса, добавив `-PublicIpPrefix` аргумент при создании общедоступного IP-адреса с помощью Azure PowerShell. В следующем примере предполагается, что префикс был создан и сохранен в переменной PowerShell с именем: *$MyOwnIPv 6prefix*.

```azurepowershell
 $MyIPv6PublicIPFromMyReservedPrefix = New-AzPublicIpAddress \
 -name PIPv6_fromPrefix `
 -ResourceGroup DsStdLb04 `
 -Location "West Central US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6 `
 -PublicIpPrefix $myOwnIPv6Prefix
```

### <a name="using-azure-cli"></a>Использование Azure CLI

В следующем примере предполагается, что префикс был создан и сохранен в переменной CLI с именем: *IPv6PrefixWestUS*.

```azurecli
az network public-ip create \
--name dsPublicIP_v6 \
--resource-group UpgradeInPlace_CLI_RG1 \
--location WestUS \
--sku Standard \
--allocation-method static \
--version IPv6 \
--public-ip-prefix  IPv6PrefixWestUS
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об [префиксе IPv6-адреса](ipv6-public-ip-address-prefix.md).
- Дополнительные сведения об [IPv6-адресах](ipv6-overview.md).
