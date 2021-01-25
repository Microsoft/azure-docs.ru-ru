---
title: Перемещение управляющего приложения, вызывающего веб-API, в рабочую среду | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как переместить управляющее приложение, которое вызывает веб-API в рабочую среду.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 99fd79fb6c51f577d9b62d15ac006b068a685bcf
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2021
ms.locfileid: "98756543"
---
# <a name="daemon-app-that-calls-web-apis---move-to-production"></a>Приложение управляющей программы, вызывающее веб-API — переместить в рабочую среду

Теперь, когда вы узнали, как получить и использовать маркер для вызова между службами, Узнайте, как переместить приложение в рабочую среду.

## <a name="deployment---multitenant-daemon-apps"></a>Развертывание — многоклиентские приложения управляющей программы

Если вы являетесь независимым поставщиком программного обеспечения, создающим управляющее приложение, которое может работать в нескольких клиентах, необходимо убедиться в том, что администратор клиента выполняет следующие действия.

- Подготавливает субъект-службу для приложения.
- Предоставляет согласие для приложения.

Вам необходимо объяснить своим заказчикам, как выполнять эти операции. Дополнительные сведения см. в разделе [запрос согласия для всего клиента](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Следующие шаги

Вот несколько ссылок, которые помогут вам получить дополнительные сведения:

# <a name="net"></a>[.NET](#tab/dotnet)

- Краткое руководство. [Получение маркера и вызов Microsoft Graph API из консольного приложения с помощью удостоверения приложения](./quickstart-v2-netcore-daemon.md).
- Справочная документация по:
  - Создание экземпляра [конфидентиалклиентаппликатион](/dotnet/api/microsoft.identity.client.confidentialclientapplicationbuilder).
  - Вызов [аккуиретокенфорклиент](/dotnet/api/microsoft.identity.client.acquiretokenforclientparameterbuilder).
- Другие примеры и учебники:
  - [Microsoft-Identity-Platform-Console-DAEMON —](https://github.com/Azure-Samples/microsoft-identity-platform-console-daemon) это простое консольное приложение управляющей программы .NET Core, в котором отображаются пользователи запросов клиентов Microsoft Graph.

    ![Пример топологии приложения управляющей программы](media/scenario-daemon-app/daemon-app-sample.svg)

    В этом же примере также показан вариант с сертификатами:

    ![Пример топологии приложения управляющей программы — сертификаты](media/scenario-daemon-app/daemon-app-sample-with-certificate.svg)

  - [Microsoft-Identity-Platform-ASPNET-webapp-DAEMON](https://github.com/Azure-Samples/microsoft-identity-platform-aspnet-webapp-daemon) включает в себя веб-приложение ASP.NET MVC, которое синхронизирует данные из Microsoft Graph с помощью удостоверения приложения, а не от имени пользователя. Этот пример также иллюстрирует процесс согласия администратора.

    ![Топология](media/scenario-daemon-app/damon-app-sample-web.svg)

# <a name="python"></a>[Python](#tab/python)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения Python, используя удостоверение приложения](./quickstart-v2-python-daemon.md).

# <a name="java"></a>[Java](#tab/java)

MSAL Java в настоящее время находится в общедоступной предварительной версии. Дополнительные сведения см. в статье [MSAL Java dev Samples](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples).

---