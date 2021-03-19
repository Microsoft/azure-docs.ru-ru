---
title: 'Создание и Настройка настраиваемых политик IPsec для "точка — сеть": PowerShell'
titleSuffix: Azure VPN Gateway
description: Эта статья поможет вам создать и настроить пользовательские политики IPSec для конфигурации P2S VPN-шлюза.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/09/2020
ms.author: cherylmc
ms.openlocfilehash: a77a870304e20c179e22b4a4ffe404315894d8a0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91743698"
---
# <a name="create-and-set-custom-ipsec-policies-for-point-to-site-preview"></a>Создание и Настройка настраиваемых политик IPsec для "точка — сеть" (Предварительная версия)

Если вашей среде требуется пользовательская политика IPsec для шифрования, можно легко настроить объект политики с необходимыми параметрами. Эта статья поможет вам создать объект настраиваемой политики, а затем задать его с помощью PowerShell.

## <a name="before-you-begin"></a>Перед началом

### <a name="prerequisites"></a>Предварительные условия

Убедитесь, что среда соответствует следующим предварительным требованиям:

* Уже настроена работоспособная VPN типа "точка — сеть". Если этого не сделать, настройте его, выполнив действия, описанные в статье **Создание VPN-подключения типа "точка — сеть**  " с помощью [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)или [портал Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="1-set-variables"></a><a name="signin"></a>1. Задание переменных

Объявите переменные, которые вы хотите использовать. Используйте следующий пример, заменяя собственные значения при необходимости. Если вы закроете сеанс PowerShell/Cloud Shell в любой момент во время выполнения упражнения, просто скопируйте и вставьте значения, чтобы повторно объявить переменные.

  ```azurepowershell-interactive
  $RG = "TestRG"
  $GWName = "VNet1GW"
  ```

## <a name="2-create-policy-object"></a><a name="create"></a>2. Создание объекта политики

Создайте объект настраиваемой политики IPsec. Можно изменить значения в соответствии с требуемыми критериями.

```azurepowershell-interactive
$vpnclientipsecpolicy = New-AzVpnClientIpsecPolicy -IpsecEncryption AES256 -IpsecIntegrity SHA256 -SALifeTime 86471 -SADataSize 429496 -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup2 -PfsGroup PFS2
```

## <a name="3-update-gateway-and-set-policy"></a><a name="update"></a>3. обновление шлюза и Настройка политики

На этом шаге обновите существующий VPN-шлюз P2S и настройте политику IPsec.

```azurepowershell-interactive
$gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway -VpnClientIpsecPolicy $vpnclientipsecpolicy
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о конфигурациях P2S см. в статье [о VPN-подключениях типа "точка — сеть](point-to-site-about.md)".
