---
title: Краткое руководство. Использование пакета SDK, вызывающего службы связи Azure
titleSuffix: An Azure Communication Services quickstart
description: Узнайте о возможностях служб связи, вызывающих пакеты SDK.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: conceptual
ms.service: azure-communication-services
zone_pivot_groups: acs-plat-web-ios-android
ms.openlocfilehash: b5ade06e8338dd810651ccd606c7dc9a313b6fa9
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105107763"
---
# <a name="quickstart-use-the-communication-services-calling-sdk"></a>Краткое руководство. Использование пакета SDK, вызывающего коммуникационные службы

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]


Приступая к работе со службами связи Azure с помощью пакета SDK, вызывающего коммуникационные службы, для добавления голоса и видео, обращающегося к вашему приложению.

::: zone pivot="platform-web"
[!INCLUDE [Calling with JavaScript](./includes/calling-sdk-js.md)]
::: zone-end

::: zone pivot="platform-android"
[!INCLUDE [Calling with Android](./includes/calling-sdk-android.md)]
::: zone-end

::: zone pivot="platform-ios"
[!INCLUDE [Calling with iOS](./includes/calling-sdk-ios.md)]
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- Ознакомьтесь с нашим [главным примером функции вызовов](../../samples/calling-hero-sample.md).
- Узнайте больше о [принципе работы функции вызовов](../../concepts/voice-video-calling/about-call-types.md).
