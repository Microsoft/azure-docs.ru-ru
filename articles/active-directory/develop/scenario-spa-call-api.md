---
title: Создание одностраничного приложения, вызывающего веб-API
titleSuffix: Microsoft identity platform
description: Узнайте, как создать одностраничное приложение, вызывающее веб-API.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/06/2019
ms.author: ryanwi
ms.custom: aaddev
ms.openlocfilehash: 4dd4a40233fc1c030581fd1ae2827061435a0ab3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98753543"
---
# <a name="single-page-application-call-a-web-api"></a>Одностраничное приложение: вызов веб-API

Рекомендуется вызывать `acquireTokenSilent` метод для получения или продления маркера доступа перед вызовом веб-API. После создания маркера можно вызвать защищенный веб-API.

## <a name="call-a-web-api"></a>Вызов веб-API

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Используйте полученный маркер доступа в качестве носителя в HTTP-запросе для вызова любого веб-API, например Microsoft Graph API. Пример:

```javascript
    var headers = new Headers();
    var bearer = "Bearer " + access_token;
    headers.append("Authorization", bearer);
    var options = {
         method: "GET",
         headers: headers
    };
    var graphEndpoint = "https://graph.microsoft.com/v1.0/me";

    fetch(graphEndpoint, options)
        .then(function (response) {
             //do something with response
        }
```

# <a name="angular"></a>[Angular](#tab/angular)

Угловая оболочка MSAL использует преимущества перехватчика HTTP для автоматического получения маркеров доступа и их присоединения к API-запросам. Дополнительные сведения см. в разделе [Получение маркера для вызова API](scenario-spa-acquire-token.md).

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [перейдите в рабочую среду](scenario-spa-production.md).