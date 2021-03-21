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
ms.openlocfilehash: d903f04055d1607ee782bd502d99a8fd9cde87ca
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578453"
---
# <a name="daemon-app-that-calls-web-apis---move-to-production"></a>Приложение управляющей программы, вызывающее веб-API — переместить в рабочую среду

Теперь, когда вы узнали, как получить и использовать маркер для вызова между службами, Узнайте, как переместить приложение в рабочую среду.

## <a name="deployment---multitenant-daemon-apps"></a>Развертывание — многоклиентские приложения управляющей программы

Если вы являетесь независимым поставщиком программного обеспечения, создающим управляющее приложение, которое может работать в нескольких клиентах, убедитесь, что администратор клиента:

- Подготавливает субъект-службу для приложения.
- Предоставляет согласие для приложения.

Вам необходимо объяснить своим заказчикам, как выполнять эти операции. Дополнительные сведения см. в разделе [запрос согласия для всего клиента](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="code-samples"></a>Примеры кода

# <a name="net"></a>[.NET](#tab/dotnet)

- Справочная документация по:
  - Создание экземпляра [конфидентиалклиентаппликатион](/dotnet/api/microsoft.identity.client.confidentialclientapplicationbuilder).
  - Вызов [аккуиретокенфорклиент](/dotnet/api/microsoft.identity.client.acquiretokenforclientparameterbuilder).
- Другие примеры и учебники:
  - [Microsoft-Identity-Platform-Console-DAEMON —](https://github.com/Azure-Samples/microsoft-identity-platform-console-daemon) это небольшое консольное приложение управляющей программы .NET Core, в котором отображаются пользователи запросов клиентов Microsoft Graph.

    ![Пример топологии приложения управляющей программы](media/scenario-daemon-app/daemon-app-sample.svg)

    В этом же примере также показан вариант с сертификатами:

    ![Пример топологии приложения управляющей программы — сертификаты](media/scenario-daemon-app/daemon-app-sample-with-certificate.svg)

  - [Microsoft-Identity-Platform-ASPNET-webapp-DAEMON](https://github.com/Azure-Samples/microsoft-identity-platform-aspnet-webapp-daemon) включает в себя веб-приложение ASP.NET MVC, которое синхронизирует данные из Microsoft Graph с помощью удостоверения приложения, а не от имени пользователя. Этот пример также иллюстрирует процесс согласия администратора.

    ![Топология](media/scenario-daemon-app/damon-app-sample-web.svg)

# <a name="java"></a>[Java](#tab/java)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения Java, используя удостоверение приложения](quickstart-v2-java-daemon.md).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

- Дополнительные сведения см. в разделе:
  - Общие сведения о [конфигурации](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md)
  - Создание экземпляра [конфидентиалклиентаппликатион](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md)
  - [FAQ](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/faq.md)
- Другие примеры и учебники:
  - [Пример управляющей программы консоли узла MSAL](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-console)

# <a name="python"></a>[Python](#tab/python)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения Python, используя удостоверение приложения](quickstart-v2-python-daemon.md).

---

## <a name="next-steps"></a>Дальнейшие действия

Вот несколько ссылок, которые помогут вам получить дополнительные сведения:

# <a name="net"></a>[.NET](#tab/dotnet)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения .NET Core, используя удостоверение приложения](quickstart-v2-netcore-daemon.md).

# <a name="java"></a>[Java](#tab/java)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения Java, используя удостоверение приложения](quickstart-v2-java-daemon.md).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Попробуйте [получить маркер и вызвать Microsoft Graph API из Node.js консольного приложения с помощью удостоверения приложения](quickstart-v2-nodejs-console.md).

# <a name="python"></a>[Python](#tab/python)

Попробуйте [получить маркер и вызвать Microsoft Graph API из консольного приложения Python, используя удостоверение приложения](quickstart-v2-python-daemon.md).

---
