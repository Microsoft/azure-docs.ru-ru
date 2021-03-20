---
title: включить файл
titleSuffix: Azure
description: включить файл
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 34a23ce76ed0e9285a686073e1cbeb95347f7b7d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81678675"
---
Чтобы получить список пиринга, выполните команду **Get-азпиринг** .

```powershell
Get-AzPeering ResourceGroupName "PeeringResourceGroup" -Name "SeattleExchangePeering"
```

В этом примере ответа показано, когда закончена успешная подготовка.

```powershell
    Name                     : SeattleExchangePeering
    Sku                      : Basic_Exchange_Free
    Kind                     : Exchange
    PeeringLocation          : Seattle
    ProvisioningState        : Succeeded
    PeerAsn                  : 65000
    Connection               : ------------------------
    PeerSessionIPv4Address   : 10.21.31.100
    MicrosoftIPv4Address     : 10.21.31.50
    SessionStateV4           : Established
    MaxPrefixesAdvertisedV4  : 20000
    PeerSessionIPv6Address   : fe01::3e:100
    MicrosoftIPv6Address     : fe01::3e:50
    SessionStateV6           : Established
    MaxPrefixesAdvertisedV6  : 2000
    ConnectionState          : Active
    Connection               : ------------------------
    PeerSessionIPv4Address   : 10.21.31.101
    MicrosoftIPv4Address     : 10.21.31.51
    SessionStateV4           : Established
    MaxPrefixesAdvertisedV4  : 20000
    PeerSessionIPv6Address   : fe01::3e:101
    MicrosoftIPv6Address     : fe01::3e:51
    SessionStateV6           : Established
    MaxPrefixesAdvertisedV6  : 2000
    ConnectionState          : Active
```