---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 872164dfee9f35881fca97b0339bd95c64a2f5ff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99070220"
---
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. С помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) Azure PowerShell создайте группу ресурсов с именем *myResourceGroup* в регионе *eastus*. 

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```
