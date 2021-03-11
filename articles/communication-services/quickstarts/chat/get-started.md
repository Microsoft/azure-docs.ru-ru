---
title: Краткое руководство. Добавление чата в приложение
titleSuffix: An Azure Communication Services quickstart
description: В этом кратком руководстве показано, как добавить в приложение чат Служб коммуникации.
author: fanche
manager: phans
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python-swift-android
ms.openlocfilehash: 4c2e0ab3170779234c1a1d0b2676632022b9dfcd
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102489729"
---
# <a name="quickstart-add-chat-to-your-app"></a>Краткое руководство. Добавление чата в приложение

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]


Начало работы со Службами коммуникации Azure с помощью клиентской библиотеки чатов Служб коммуникации для добавления в приложение чата с поддержкой реального времени. В этом кратком руководстве показано, как использовать клиентскую библиотеку чатов, чтобы создавать цепочки чатов, с помощью которых пользователи смогут общаться между собой. См. сведения о [чатах](../../concepts/chat/concepts.md) в документации.

::: zone pivot="programming-language-javascript"
[!INCLUDE [Chat with JavaScript client library](./includes/chat-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Chat with Python client library](./includes/chat-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Chat with Java client library](./includes/chat-java.md)]
::: zone-end

::: zone pivot="programming-language-android"
[!INCLUDE [Chat with Android client library](./includes/chat-android.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Chat with C# client library](./includes/chat-csharp.md)]
::: zone-end

::: zone pivot="programming-language-swift"
[!INCLUDE [Chat with iOS client library](./includes/chat-swift.md)]
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как сделать следующее:

> [!div class="checklist"]
> * создать клиент чата;
> * создать цепочку с двумя пользователями;
> * отправить сообщение в цепочку;
> * получить сообщения из цепочки;
> * исключить пользователей из цепочки.

> [!div class="nextstepaction"]
> [Ознакомьтесь с главным имиджевым приложением чата](../../samples/chat-hero-sample.md)

Возможно, вы также захотите:

 - узнать о [понятиях, связанных с чатами](../../concepts/chat/concepts.md);
 - ознакомиться с нашей [клиентской библиотекой чатов](../../concepts/chat/sdk-features.md).
