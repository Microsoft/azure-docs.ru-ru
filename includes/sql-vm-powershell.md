---
author: MikeRayMSFT
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 11/25/2018
ms.author: mikeray
ms.openlocfilehash: 27da28073b01c6bc43868172d6c989d9518d8f53
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95559602"
---
## <a name="start-your-powershell-session"></a>Запуск сеанса PowerShell
 

Выполните командлет [**Connect-Az Account**](/powershell/module/Az.Accounts/Connect-AzAccount). Откроется окно входа, в котором нужно ввести свои учетные данные. Используйте для входа те же учетные данные, что и для входа на портал Azure.

```powershell
Connect-AzAccount
```

Если у вас несколько подписок, используйте командлет [**Set-AzContext**](/powershell/module/az.accounts/set-azcontext), чтобы выбрать подписку, которую будет использовать сеанс PowerShell. Чтобы узнать, какую подписку использует текущий сеанс PowerShell, выполните командлет [**Get-AzContext**](/powershell/module/az.accounts/get-azcontext). Чтобы просмотреть все подписки, выполните командлет [**Get-AzSubscription**](/powershell/module/az.accounts/get-azsubscription).

```powershell
Set-AzContext -SubscriptionId '4cac86b0-1e56-bbbb-aaaa-000000000000'
```