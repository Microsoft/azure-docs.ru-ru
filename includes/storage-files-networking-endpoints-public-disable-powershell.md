---
title: включить файл
description: включить файл
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 6/2/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: a0a9bc29c3e20a025fb2c46a71c2f134c37bee04
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "84465114"
---
Следующая команда PowerShell блокирует весь трафик к общедоступной конечной точке учетной записи хранения. Обратите внимание, что в этой команде параметр `-Bypass` имеет значение `AzureServices`. Это значение разрешает доступ к учетной записи хранения через общедоступную конечную точку для доверенных служб корпорации Майкрософт, таких как Синхронизация файлов Azure.

```PowerShell
# This assumes $storageAccount is still defined from the beginning of this of this guide.
$storageAccount | Update-AzStorageAccountNetworkRuleSet `
        -DefaultAction Deny `
        -Bypass AzureServices `
        -WarningAction SilentlyContinue `
        -ErrorAction Stop | `
    Out-Null
```