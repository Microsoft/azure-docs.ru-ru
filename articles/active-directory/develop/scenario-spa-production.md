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
ms.openlocfilehash: 9656da8be086724482f129efab323e02b73e117e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954972"
---
# <a name="single-page-application-move-to-production"></a>Одностраничное приложение: переход в рабочую среду

Теперь, когда вы узнали, как получить маркер для вызова веб-API, ниже приведены некоторые моменты, которые следует учитывать при переносе приложения в рабочую среду.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="deploy-your-app"></a>Развертывание приложения

Ознакомьтесь с [примером развертывания](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter3) , чтобы узнать, как развернуть проекты SPA и веб-API с помощью службы хранилища Azure и служб приложений Azure соответственно. 

## <a name="code-samples"></a>Примеры кода

В этих примерах кода демонстрируются несколько ключевых операций для одностраничного приложения.
- [Spa с серверной стороны ASP.NET](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi): как получить маркеры для собственного серверного веб-API (ASP.NET Core) с помощью **MSAL.js**.

- [Node.js веб-API (Azure AD](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2): Проверка маркеров доступа для серверного веб-api (Node.js) с помощью **Passport-Azure-AD**.

- [Spa с Azure AD B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp): как использовать **MSAL.js** для входа пользователей в приложение, зарегистрированное в **Azure Active Directory B2C** (Azure AD B2C).

- [Node.js веб-API (Azure AD B2C)](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi): как использовать **Passport-Azure-AD** для проверки маркеров доступа для приложений, зарегистрированных в **Azure Active Directory B2C** (Azure AD B2C).

## <a name="next-steps"></a>Дальнейшие действия

- [Руководство по JavaScript Spa](./tutorial-v2-javascript-spa.md). подробный обзор способов входа пользователей и получения маркера доступа для вызова **API Microsoft Graph** с помощью **MSAL.js**.