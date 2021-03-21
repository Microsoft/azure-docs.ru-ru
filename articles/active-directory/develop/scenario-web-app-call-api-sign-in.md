---
title: Удалить учетные записи из кэша маркеров при выходе | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как удалить учетную запись из кэша маркеров при выходе
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 9fc271dfa9edbedac8527009dd2b2180b7c5e7cd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756263"
---
# <a name="a-web-app-that-calls-web-apis-remove-accounts-from-the-token-cache-on-global-sign-out"></a>Веб-приложение, вызывающее веб-API: удаление учетных записей из кэша маркеров при глобальном выходе

Вы узнали, как добавить вход в веб-приложение в [веб-приложении, которое входит в систему пользователей: вход и](scenario-web-app-sign-user-sign-in.md)выход.

Выход отличается для веб-приложения, которое вызывает веб-API. Когда пользователь выходит из приложения или из любого приложения, необходимо удалить маркеры, связанные с этим пользователем, из кэша маркеров.

## <a name="intercept-the-callback-after-single-sign-out"></a>Перехват обратного вызова после единого выхода

Чтобы очистить запись кэша маркеров, связанную с учетной записью, в которой выполнен выход из службы, приложение может перехватить `logout` событие After. Веб-приложения хранят маркеры доступа для каждого пользователя в кэше маркеров. Перехватывая `logout` ответный вызов после обратного вызова, веб-приложение может удалить пользователя из кэша.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Microsoft. Identity. Web берет на себя реализацию выхода. Дополнительные сведения см. в разделе [Исходный код Microsoft. Identity. Web](https://github.com/AzureAD/microsoft-identity-web/blob/c29f1a7950b940208440bebf0bcb524a7d6bee22/src/Microsoft.Identity.Web/WebAppExtensions/WebAppCallsWebApiAuthenticationBuilderExtensions.cs#L168-L176) .

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Пример ASP.NET не удаляет учетные записи из кэша при глобальном выходе.

# <a name="java"></a>[Java](#tab/java)

Пример Java не удаляет учетные записи из кэша при глобальном выходе.

# <a name="python"></a>[Python](#tab/python)

Пример Python не удаляет учетные записи из кэша при глобальном выходе.

---

## <a name="next-steps"></a>Дальнейшие действия

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для веб-приложения](./scenario-web-app-call-api-acquire-token.md?tabs=aspnetcore).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для веб-приложения](./scenario-web-app-call-api-acquire-token.md?tabs=aspnet).

# <a name="java"></a>[Java](#tab/java)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для веб-приложения](./scenario-web-app-call-api-acquire-token.md?tabs=java).

# <a name="python"></a>[Python](#tab/python)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для веб-приложения](./scenario-web-app-call-api-acquire-token.md?tabs=python).

---