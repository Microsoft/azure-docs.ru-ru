---
title: Преобразование устаревшего прямого пиринга в ресурс Azure с помощью PowerShell
titleSuffix: Azure
description: Преобразование устаревшего прямого пиринга в ресурс Azure с помощью PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d3e7cdf11e1e1e033b4e72b9579d8c63b28e6c88
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89071692"
---
# <a name="convert-a-legacy-direct-peering-to-an-azure-resource-by-using-powershell"></a>Преобразование устаревшего прямого пиринга в ресурс Azure с помощью PowerShell

В этой статье описывается, как преобразовать существующий прямой пиринг в ресурс Azure с помощью командлетов PowerShell.

При желании вы можете выполнить это пошаговое руководством с помощью [портала](howto-legacy-direct-portal.md)Azure.

## <a name="before-you-begin"></a>Перед началом
* Прежде чем начать настройку, изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по прямому пирингу](walkthrough-direct-all.md).

### <a name="work-with-azure-powershell"></a>Работа с Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="convert-a-legacy-direct-peering-to-an-azure-resource"></a>Преобразование устаревшего прямого пиринга в ресурс Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-a-legacy-direct-peering-for-conversion"></a><a name= get></a>Получение устаревшего прямого пиринга для преобразования
В этом примере показано, как получить устаревший прямой пиринг в расположении пиринга в Сиэтле.

```powershell
$legacyPeering = Get-AzLegacyPeering `
    -Kind Direct -PeeringLocation "Seattle"
$legacyPeering
```

Вот пример ответа:
```powershell
Name                       :
Sku                        : Basic_Direct_Free
Kind                       : Direct
PeeringLocation            : Seattle
UseForPeeringService       : False
PeerAsn.Id                 :
Connection                 : ------------------------
PeeringDBFacilityId        : 71
SessionPrefixIPv4          : 4.71.156.72/30
PeerSessionIPv4Address     : 4.71.156.73
MicrosoftIPv4Address       : 4.71.156.74
SessionStateV4             : Established
MaxPrefixesAdvertisedV4    : 20000
SessionPrefixIPv6          : 2001:1900:2100::1e10/126
MaxPrefixesAdvertisedV6    : 2000
ConnectionState            : Active
BandwidthInMbps            : 0
ProvisionedBandwidthInMbps : 20000
Connection                 : ------------------------
PeeringDBFacilityId        : 71
SessionPrefixIPv4          : 4.68.70.140/30
PeerSessionIPv4Address     : 4.68.70.141
MicrosoftIPv4Address       : 4.68.70.142
SessionStateV4             : Established
MaxPrefixesAdvertisedV4    : 20000
SessionPrefixIPv6          : 2001:1900:4:3::cc/126
PeerSessionIPv6Address     : 2001:1900:4:3::cd
MicrosoftIPv6Address       : 2001:1900:4:3::ce
SessionStateV6             : Established
MaxPrefixesAdvertisedV6    : 2000
ConnectionState            : Active
BandwidthInMbps            : 0
ProvisionedBandwidthInMbps : 20000
ProvisioningState          : Succeeded
```

### <a name="convert-a-legacy-direct-peering"></a>Преобразование устаревшего прямого пиринга

&nbsp;
> [!IMPORTANT]
> При преобразовании устаревшего пиринга в ресурс Azure изменения не поддерживаются. &nbsp;

Используйте эту команду для преобразования устаревшего прямого пиринга в ресурс Azure:

```powershell
$legacyPeering[0] | New-AzPeering `
    -Name "SeattleDirectPeering" `
    -ResourceGroupName "PeeringResourceGroup" `

```

Вот пример ответа:

```powershell
Name                 : SeattleDirectPeering
Sku.Name             : Basic_Direct_Free
Kind                 : Direct
Connections          : {11, 11}
PeerAsn.Id           : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/{asnNumber}
UseForPeeringService : False
PeeringLocation      : Seattle
ProvisioningState    : Succeeded
Location             : centralus
Id                   : /subscriptions/{subscriptionId}/resourceGroups/PeeringResourceGroup/providers/Microsoft.Peering/peerings/SeattleDirectPeering
Type                 : Microsoft.Peering/peerings
Tags                 : {}
```

## <a name="additional-resources"></a>Дополнительные ресурсы
Подробные описания всех параметров можно получить, выполнив следующую команду:

```powershell
Get-Help Get-AzPeering -detailed
```

Дополнительные сведения см. в разделе [Часто задаваемые вопросы об интернет-пиринге](faqs.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение прямого пиринга с помощью PowerShell](howto-direct-powershell.md)
