---
title: Краткое руководство. Добавление чата в приложение
titleSuffix: An Azure Communication Services quickstart
description: В этом кратком руководстве показано, как добавить в приложение чат Служб коммуникации.
author: fanche
manager: phans
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python-swift-android
ms.openlocfilehash: 6d3f9f7fd30d2c6b1cbc3882a41546593ee1c156
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726662"
---
# <a name="quickstart-add-chat-to-your-app"></a>Краткое руководство. Добавление чата в приложение

Начало работы со Службами коммуникации Azure с помощью пакета SDK для чата Служб коммуникации для добавления в приложение чата с поддержкой реального времени. В этом кратком руководстве показано, как использовать пакет SDK для чата, чтобы создавать цепочки чатов, с помощью которых пользователи смогут общаться между собой. См. сведения о [чатах](../../concepts/chat/concepts.md) в документации.

::: zone pivot="programming-language-javascript"
[!INCLUDE [Chat with JavaScript SDK](./includes/chat-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Chat with Python SDK](./includes/chat-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Chat with Java SDK](./includes/chat-java.md)]
::: zone-end

::: zone pivot="programming-language-android"
[!INCLUDE [Chat with Android SDK](./includes/chat-android.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Chat with C# SDK](./includes/chat-csharp.md)]
::: zone-end

::: zone pivot="programming-language-swift"
[!INCLUDE [Chat with iOS SDK](./includes/chat-swift.md)]
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
 - познакомиться с [пакетом SDK для чата](../../concepts/chat/sdk-features.md)
