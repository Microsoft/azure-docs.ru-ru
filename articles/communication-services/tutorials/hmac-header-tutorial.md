---
title: Как подписать HTTP-запрос с помощью C#
titleSuffix: An Azure Communication Services tutorial
description: Сведения о том, как подписать HTTP-запрос для Служб коммуникации Azure с помощью C#.
author: alexandra142
manager: soricos
services: azure-communication-services
ms.author: apistrak
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 51b38a7344e3a54f565c55f17b7c8970bbef3a1b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105727779"
---
# <a name="sign-an-http-request"></a>Подписывание HTTP-запроса

В этом учебнике описывается, как подписать HTTP-запрос с помощью подписи HMAC.

[!INCLUDE [Sign an HTTP request C#](./includes/hmac-header-csharp.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить и удалить подписку на Службы коммуникации, удалите ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. дополнительные сведения об [очистке ресурсов Служб коммуникации Azure](../quickstarts/create-communication-resource.md#clean-up-resources) и [очистке ресурсов Функций Azure](../../azure-functions/create-first-function-vs-code-csharp.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление функции голосового вызова в приложение](../quickstarts/voice-video-calling/getting-started-with-calling.md)

Кроме того, вам может понадобиться следующее:

- [Добавление чата в приложение](../quickstarts/chat/get-started.md)
- [Создание маркеров доступа пользователей](../quickstarts/access-tokens.md)
- [Сведения об архитектуре клиент-сервер](../concepts/client-and-server-architecture.md)
- [Сведения о проверке подлинности](../concepts/authentication.md)
