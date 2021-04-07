---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: a5f829fc0e153c3a427fd1c7cc7d5c613cd624f6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "83648271"
---
Функции Azure поддерживают общий доступ к ресурсам независимо от источника (CORS). CORS настраивается [на портале](../articles/azure-functions/functions-how-to-use-azure-function-app-settings.md#cors) и через [Azure CLI](/cli/azure/functionapp/cors). Список разрешенных источников CORS применяется на уровне приложения-функции. При включении CORS ответы включают заголовок `Access-Control-Allow-Origin`. Дополнительные сведения см. в статье [об общем доступе к ресурсам независимо от источника](../articles/azure-functions/functions-how-to-use-azure-function-app-settings.md#cors). 