---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/29/2019
ms.author: glenga
ms.openlocfilehash: f16852cdc18053a3a8e375dc6f14e39bb069afef
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "72329618"
---
Функции Azure позволяют выполнять подключение служб Azure и других ресурсов к функциям без необходимости написания кода для интеграции. Эти *привязки*, которые представляют как входные, так и выходные данные, объявляются в определении функции. Данные привязок предоставляются функции в качестве параметров. *Триггер* является специальным типом входных привязок. Хотя функция обладает только одним триггером, она может состоять из нескольких входных и выходных привязок. Дополнительные сведения см. в статье [Основные понятия триггеров и привязок в Функциях Azure](../articles/azure-functions/functions-triggers-bindings.md).