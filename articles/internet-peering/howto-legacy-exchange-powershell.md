---
title: Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью PowerShell
titleSuffix: Azure
description: Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 12/15/2020
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: acc32f4916f5f7f8fe22eebdd1e72db297cac94c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97590211"
---
# <a name="convert-a-legacy-exchange-peering-to-an-azure-resource-by-using-powershell"></a>Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью PowerShell

В этой статье описывается, как преобразовать имеющийся устаревший пиринг Exchange в ресурс Azure с помощью командлетов PowerShell.

При желании вы можете выполнить это пошаговое руководством с помощью [портала](howto-legacy-exchange-portal.md)Azure.

## <a name="before-you-begin"></a>Перед началом
* Перед началом конфигурации изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по пирингу Exchange](walkthrough-exchange-all.md).

### <a name="work-with-azure-powershell"></a>Работа с Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="convert-a-legacy-exchange-peering-to-an-azure-resource"></a>Преобразование устаревшего пиринга Exchange в ресурс Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-legacy-exchange-peering-for-conversion"></a><a name= get></a>Получение устаревшего пиринга Exchange для преобразования
В этом примере показано, как получить устаревший пиринг Exchange в расположении пиринга в Сиэтле:

```powershell
$legacyPeering = Get-AzLegacyPeering -Kind Exchange -PeeringLocation "Seattle"
$legacyPeering
```

Ответ выглядит примерно так:
```powershell
    Kind                     : Exchange
    PeeringLocation          : Seattle
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
```

### <a name="convert-legacy-peering"></a>Преобразование устаревшего пиринга
Эту команду можно использовать для преобразования устаревшего пиринга Exchange в ресурс Azure:

```powershell
$legacyPeering[0] | New-AzPeering `
    -Name "SeattleExchangePeering" `
    -ResourceGroupName "PeeringResourceGroup"

```

&nbsp;
> [!IMPORTANT] 
> При преобразовании устаревшего пиринга в ресурс Azure изменения не поддерживаются.
&nbsp;

В этом примере ответа показано, когда закончена успешная подготовка.

```powershell
    Name                     : SeattleExchangePeering
    Kind                     : Exchange
    Sku                      : Basic_Exchange_Free
    PeeringLocation          : Seattle
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
```
## <a name="additional-resources"></a>Дополнительные ресурсы
Подробное описание всех параметров можно получить, выполнив следующую команду.

```powershell
Get-Help Get-AzPeering -detailed
```
Дополнительные сведения см. в разделе [Часто задаваемые вопросы об интернет-пиринге](faqs.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение пиринга Exchange с помощью PowerShell](howto-exchange-powershell.md)
