---
title: Переместить одностраничное приложение в рабочую среду
titleSuffix: Microsoft identity platform
description: Узнайте, как создать одностраничное приложение (переместить в рабочую среду)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 0c1e9e6da02478f5e4703676b74fc8247a4f619b
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2021
ms.locfileid: "98753523"
---
# <a name="single-page-application-move-to-production"></a>Одностраничное приложение: переход в рабочую среду

Теперь, когда вы узнали, как получить маркер для вызова веб-API, Узнайте, как перейти в рабочую среду.

## <a name="improve-your-app"></a>Улучшение приложения

[Включите ведение журнала](msal-logging.md) , чтобы подготовить приложение к работе.

## <a name="test-your-integration"></a>Тестирование интеграции

Протестируйте интеграцию согласно [контрольному списку для интеграции платформы удостоверений Майкрософт](identity-platform-integration-checklist.md).

## <a name="deploy-your-app"></a>Развертывание приложения

Ознакомьтесь с [примером развертывания](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter3) , чтобы узнать, как развернуть проекты SPA и веб-API с помощью службы хранилища Azure и служб приложений Azure соответственно. 

## <a name="next-steps"></a>Следующие шаги

- Подробное описание примера быстрого запуска, в котором объясняется, как войти в систему пользователей и получить маркер доступа для вызова **Microsoft Graph API** с помощью **MSAL.js**: руководство по [JavaScript Spa](./tutorial-v2-javascript-spa.md).

- Пример, демонстрирующий получение маркеров для собственного серверного веб-API (ASP.NET Core) с помощью **MSAL.js**: [Spa с серверной стороны ASP.NET](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi).

- Пример, демонстрирующий проверку маркеров доступа для серверного веб-API (Node.js) с помощью интерфейса **Passport-Azure-AD**: [Node.js веб-API (Azure AD](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2)).

- Пример, в котором показано, как использовать **MSAL.js** для входа пользователей в приложение, зарегистрированное в **Azure Active Directory B2C** (Azure AD B2C): [Spa с Azure AD B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp).

- Пример, в котором показано, как использовать **Passport-Azure-AD** для проверки маркеров доступа для приложений, зарегистрированных с помощью **Azure Active Directory B2C** (Azure AD B2C): [Node.js веб-API (Azure AD B2C)](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi).
