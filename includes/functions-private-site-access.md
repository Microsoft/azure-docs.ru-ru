---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: 3c9679f3d66d58c7a6c847b54c84438c979ecd39
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97936872"
---
[Частная конечная точка Azure](../articles/private-link/private-endpoint-overview.md) — это сетевой интерфейс, который обеспечивает безопасное подключение к службе с помощью Приватного канала Azure.  Частная конечная точка использует частный IP-адрес из виртуальной сети, эффективно предоставляя доступ к службе из виртуальной сети.

Вы можете использовать закрытую конечную точку для функций, размещенных в планах [службы приложений](../articles/azure-functions/dedicated-plan.md) уровня " [премиум](../articles/azure-functions/functions-premium-plan.md) " и "Услуга".

При создании входящего подключения к частной конечной точке для функций потребуется также запись DNS для разрешения частного адреса.  По умолчанию частная запись DNS будет создана при создании частной конечной точки с помощью портал Azure.

Дополнительные сведения см. в статье [использование частных конечных точек для веб-приложений](../articles/app-service/networking/private-endpoint.md).