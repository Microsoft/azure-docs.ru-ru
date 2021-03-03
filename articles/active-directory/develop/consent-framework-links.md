---
title: Как действует согласие для приложения
description: Дополнительные сведения о действии инфраструктуры согласия Azure AD, а также о том, как вы можете использовать ее при разработке приложений в Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ROBOTS: NOINDEX
ms.openlocfilehash: 9fa910dee2830f6749f0fbd36f065c31dafa6757
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101646254"
---
# <a name="how-application-consent-works"></a>Как действует согласие для приложения

Эта статья содержит дополнительные сведения о действии инфраструктуры согласия Azure AD, используя которые вы сможете оптимизировать разработку приложений.

## <a name="recommended-documents"></a>Рекомендуемые документы

- Получите общее представление о том, [как владелец ресурса может управлять доступом приложения к ресурсам благодаря согласию](./developer-glossary.md#consent).
- Ознакомьтесь с пошаговым руководством по [реализации согласия в инфраструктуре согласия Azure AD](./quickstart-register-app.md).
- Получите более подробные сведения об [использовании инфраструктуры согласия в мультитенантном приложении](./howto-convert-app-to-be-multi-tenant.md), чтобы реализовать согласие пользователя и администратора, с поддержкой нескольких дополнительных шаблонов многоуровневого приложения.
- Узнайте подробнее о том, [как согласие поддерживается на уровне протокола OAuth 2.0 во время предоставления кода авторизации.](../azuread-dev/v1-protocols-oauth-code.md#request-an-authorization-code)

## <a name="next-steps"></a>Дальнейшие действия
[AzureAD Microsoft Q&A](/answers/topics/azure-active-directory.html)