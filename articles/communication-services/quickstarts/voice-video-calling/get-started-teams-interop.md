---
title: Краткое руководство. Реализация взаимодействия в Teams с помощью Служб коммуникации Azure
titleSuffix: An Azure Communication Services quickstart
description: Из этого краткого руководства вы узнаете, как присоединиться к собранию в Teams с помощью пакета SDK вызовов для Служб коммуникации Azure.
author: chpalm
ms.author: chpalm
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-plat-web-ios-android
ms.openlocfilehash: a9ef74c04c1f709348ae1d6dd97558ee6bedccf3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104654974"
---
# <a name="quickstart-join-your-calling-app-to-a-teams-meeting"></a>Краткое руководство. Подключение приложения для звонков к собранию в Teams

> [!IMPORTANT]
> Чтобы включить или отключить [взаимодействие арендаторов в Teams](../../concepts/teams-interop.md), заполните [эту форму](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR21ouQM6BHtHiripswZoZsdURDQ5SUNQTElKR0VZU0VUU1hMOTBBMVhESS4u).

Приступите к работе со Службами коммуникации Azure, подключив решение для вызовов к Microsoft Teams с помощью клиентской библиотеки JavaScript.

::: zone pivot="platform-web"
[!INCLUDE [Calling with JavaScript](./includes/teams-interop-javascript.md)]
::: zone-end

::: zone pivot="platform-android"
[!INCLUDE [Calling with Android](./includes/teams-interop-android.md)]
::: zone-end

::: zone pivot="platform-ios"
[!INCLUDE [Calling with iOS](./includes/teams-interop-ios.md)]
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- Ознакомьтесь с нашим [главным примером функции вызовов](../../samples/calling-hero-sample.md).
- Изучите [возможности клиентской библиотеки для вызовов](./calling-client-samples.md).
- Узнайте больше о [принципе работы функции вызовов](../../concepts/voice-video-calling/about-call-types.md).
