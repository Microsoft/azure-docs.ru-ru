---
title: Маршрутизация сетевого трафика для обеспечения высокой доступности — Azure PowerShell
description: Пример скрипта Azure PowerShell для маршрутизация трафика с целью обеспечения высокой доступности приложений.
services: traffic-manager
documentationcenter: traffic-manager
author: asudbring
manager: KumudD
ms.service: traffic-manager
ms.devlang: powershell
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: traffic-manager
ms.date: 05/16/2017
ms.author: allensu
ms.openlocfilehash: ce7dd8d5398274b08eae887cf811e397a2797959
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98231980"
---
# <a name="route-traffic-for-high-availability-of-applications---azure-powershell"></a>Маршрутизация трафика для обеспечения высокой доступности приложений — Azure PowerShell

Этот скрипт создает группу ресурсов, два плана службы приложений, два веб-приложения, профиль и две конечные точки диспетчера трафика. Диспетчер трафика направляет трафик в приложение в основном регионе. Если оно недоступно, трафик направляется в дополнительный регион. Перед выполнением этого скрипта необходимо задать уникальные в Azure значения MyWebApp, MyWebAppL1 и MyWebAppL2. После выполнения вы сможете подключиться к приложению в основном регионе, используя URL-адрес mywebapp.trafficmanager.net.

При необходимости установите Azure PowerShell с помощью инструкции, приведенной в [руководстве Azure PowerShell](/powershell/azure/), а затем выполните команду `Connect-AzAccount`, чтобы создать подключение к Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/traffic-manager/direct-traffic-for-increased-application-availability/direct-traffic-for-increased-application-availability.ps1 "Route traffic for high availability")]


Выполните следующую команду, чтобы удалить группу ресурсов, виртуальную машину и все связанные с ней ресурсы.

```powershell
Remove-AzResourceGroup -Name myResourceGroup1
Remove-AzResourceGroup -Name myResourceGroup2
```


## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, веб-приложения, профиля диспетчера трафика и всех связанных ресурсов этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)  | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Создает план службы приложений. Это как ферма сервера для веб-приложения Azure. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Создает веб-приложение Azure в плане службы приложений. |
| [Set-AzResource](/powershell/module/az.resources/new-azresource) | Создает веб-приложение Azure в плане службы приложений. |
| [New-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile) | Создает профиль диспетчера трафика Azure. |
| [New-AzTrafficManagerEndpoint](/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) | Добавление конечной точки в профиль диспетчера трафика Azure. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры скриптов PowerShell для сетей см. в [обзорной документации по сетям Azure](../powershell-samples.md?toc=%2fazure%2fnetworking%2ftoc.json).