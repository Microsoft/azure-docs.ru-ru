---
title: Руководство. Создание безопасного веб-приложения в Службе приложений Azure | Azure
description: Из этого руководства вы узнаете, как создать веб-приложение с помощью Службы приложений Azure, включить проверку подлинности, вызвать службу хранилища Azure и вызвать Microsoft Graph.
services: active-directory, app-service-web, storage, microsoft-graph
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 04/02/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 5b94000ce59b3a115e53ecdcd0849b97aae552c5
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221846"
---
# <a name="tutorial-enable-authentication-in-app-service-and-access-storage-and-microsoft-graph"></a>Руководство по Включение проверки подлинности в Службе приложений и получение доступа к хранилищу и Microsoft Graph

В этом руководстве описан типичный сценарий работы с приложением:

- [Настройка проверки подлинности для веб-приложения](scenario-secure-app-authentication-app-service.md) и ограничение доступа для пользователей в организации (A на схеме).
- [Безопасный доступ к службе хранилища Azure](scenario-secure-app-access-storage.md) от имени веб-приложения с помощью управляемых удостоверений (B на схеме).
- Доступ к данным в Microsoft Graph от имени [выполнившего вход пользователя](scenario-secure-app-access-microsoft-graph-as-user.md) или [веб-приложения](scenario-secure-app-access-microsoft-graph-as-app.md) с помощью управляемых удостоверений (C на схеме).
- [Очистка ресурсов](scenario-secure-app-clean-up-resources.md), созданных для работы с этим руководством.

:::image type="content" source="./media/scenario-secure-app-overview/web-app.svg" alt-text="Схема: сценарии использования приложений на платформе удостоверений Майкрософт" border="false":::

Чтобы начать работу, узнайте, как включить проверку подлинности для веб-приложения.

> [!div class="nextstepaction"]
> [Настройка проверки подлинности для веб-приложения](scenario-secure-app-authentication-app-service.md)
