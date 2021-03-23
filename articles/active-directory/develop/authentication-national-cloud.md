---
title: Проверка подлинности с помощью Azure AD и национальные облака | Azure
titleSuffix: Microsoft identity platform
description: Узнайте о конечных точках для регистрации и проверки подлинности приложений в национальных облаках.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: negoe
ms.reviewer: marsma, negoe,celested
ms.custom: aaddev,references_regions
ms.openlocfilehash: 719d89fafe57e7f5132698404bcc0a41a9f9db37
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104780078"
---
# <a name="national-clouds"></a>Национальные облака

Национальные облака — это физически изолированные экземпляры Azure. Эти регионы Azure призваны обеспечить соблюдение требований по размещению и независимости данных, а также законодательных требований в пределах географических границ.

Кроме глобального облака, Azure Active Directory (Azure AD) развертывается в следующих национальных облаках:  

- Azure для государственных организаций
- Azure для Германии
- Azure China 21Vianet

Национальные облака являются уникальными и отличаются от глобального Azure средой. Важно помнить о ключевых различиях при разработке приложений для этих сред. Отличия наблюдаются в регистрации приложений, получении маркеров и настройке конечных точек.

## <a name="app-registration-endpoints"></a>Конечные точки регистрации приложений

Для каждого национального облака существует отдельный портал Azure. Чтобы интегрировать приложения с платформой удостоверений Microsoft в национальном облаке, необходимо зарегистрировать каждое приложение отдельно на соответствующем портале Azure в зависимости от среды.

В следующей таблице перечислены базовые URL-адреса конечных точек Azure AD, используемых для регистрации приложения в каждом из национальных облаков.

| Национальное облако | Конечная точка портала Azure AD |
|----------------|--------------------------|
| Azure AD для US Gov организаций | `https://portal.azure.us` |
| Azure AD для Германии | `https://portal.microsoftazure.de` |
| Azure AD для Китая через 21Vianet | `https://portal.azure.cn` |
| Azure AD (глобальная служба) |`https://portal.azure.com` |

## <a name="azure-ad-authentication-endpoints"></a>Конечные точки Azure AD для проверки подлинности

Все национальные облака проводят проверку подлинности пользователей в каждой среде отдельно и имеют отдельные конечные точки для проверки подлинности.

В следующей таблице перечислены базовые URL-адреса конечных точек Azure AD для получения маркеров для каждого национального облака.

| Национальное облако | Конечная точка проверки подлинности Azure AD |
|----------------|-------------------------|
| Azure AD для US Gov организаций | `https://login.microsoftonline.us` |
| Azure AD для Германии| `https://login.microsoftonline.de` |
| Azure AD для Китая через 21Vianet | `https://login.partner.microsoftonline.cn/common` |
| Azure AD (глобальная служба)| `https://login.microsoftonline.com` |

Запросы к конечным точкам Azure AD для авторизации или получения маркеров проверки подлинности можно составлять, используя базовый URL-адрес соответствующего региона. Например, для Azure для Германии:

- стандартная конечная точка авторизации — `https://login.microsoftonline.de/common/oauth2/v2.0/authorize`;
- стандартная конечная точка токена — `https://login.microsoftonline.de/common/oauth2/v2.0/token`.

В приложениях с одним клиентом замените общее в указанных выше URL-адресах идентификатором или именем клиента. Например, `https://login.microsoftonline.de/contoso.com`.

## <a name="microsoft-graph-api"></a>API Microsoft Graph

Сведения о процедуре вызова API Microsoft Graph в среде национального облака см. в разделе [Развертывание Microsoft Graph в национальном облаке](/graph/deployments).

> [!IMPORTANT]
> Некоторые функции и службы, существующие в отдельных регионах глобальной службы, могут быть недоступными для всех национальных облаков Azure. Узнать, какие службы доступны, можно в разделе [Доступные продукты по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=all&regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia,china-non-regional,china-east,china-east-2,china-north,china-north-2,germany-non-regional,germany-central,germany-northeast).

Чтобы узнать, как создать приложение, используя платформу удостоверений Microsoft, изучите [учебник по библиотеке проверки подлинности Microsoft (MSAL)](msal-national-cloud.md). В частности, приложение авторизует пользователя и получает маркер доступа для вызова API Microsoft Graph.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать [библиотеку проверки подлинности Майкрософт (MSAL) в национальной облачной среде](msal-national-cloud.md).

Документация по национальной облаку:

- [Azure для государственных организаций](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure для Германии](../../germany/index.yml)
