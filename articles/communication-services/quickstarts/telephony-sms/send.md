---
title: Краткое руководство. Отправка SMS-сообщения
titleSuffix: An Azure Communication Services quickstart
description: Сведения о том, как реализовать отправку SMS-сообщений с помощью Служб коммуникации Azure.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.custom: tracking-python, devx-track-js
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 061a7787183918aaee529b686c1478cd4d4b396b
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101661602"
---
# <a name="quickstart-send-an-sms-message"></a>Краткое руководство. Отправка SMS-сообщения

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

> [!IMPORTANT]
> SMS можно получать и отправлять с номеров телефонов в США. Номера телефонов из других регионов пока не поддерживаются SMS Службы коммуникации.
> Дополнительные сведения см. на странице **[Типы номеров телефона](../../concepts/telephony-sms/plan-solution.md)** .

::: zone pivot="programming-language-csharp"
[!INCLUDE [Send SMS with .NET client library](./includes/send-sms-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Send SMS with JavaScript client library](./includes/send-sms-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Send SMS with Python client library](./includes/send-sms-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Send SMS with Java client library](./includes/send-sms-java.md)]
::: zone-end

## <a name="troubleshooting"></a>Устранение неполадок

Для устранения проблем, связанных с доставкой текстовых сообщений, вы можете [включить отчеты о доставке в Сетке событий](./handle-sms-events.md), чтобы сохранять сведения о доставке.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как реализовать отправку SMS-сообщений с помощью Служб коммуникации Azure.

> [!div class="nextstepaction"]
> [Оформление подписки на события SMS](./handle-sms-events.md)

> [!div class="nextstepaction"]
> [Типы номеров телефона](../../concepts/telephony-sms/plan-solution.md)

> [!div class="nextstepaction"]
> [Дополнительные сведения об SMS](../../concepts/telephony-sms/concepts.md)