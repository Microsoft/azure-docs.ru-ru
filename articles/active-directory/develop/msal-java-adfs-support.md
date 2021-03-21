---
title: Поддержка AD FS (MSAL для Java)
titleSuffix: Microsoft identity platform
description: Сведения о поддержке службы федерации Active Directory (AD FS) (AD FS) в библиотеке проверки подлинности Майкрософт для Java (MSAL4j).
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/21/2019
ms.author: sagonzal
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: 82e3f3b32c6d96b83ec1d0402f344e7d65788c06
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98063711"
---
# <a name="active-directory-federation-services-support-in-msal-for-java"></a>Поддержка службы федерации Active Directory (AD FS) в MSAL для Java

Службы федерации Active Directory (AD FS) (AD FS) в Windows Server позволяет добавлять проверку подлинности OpenID Connect Connect и OAuth 2,0, а также авторизацию в библиотеке проверки подлинности Майкрософт для Java (MSAL for Java). После интеграции приложение может проверять подлинность пользователей в AD FS, Федеративных через Azure AD. Дополнительные сведения о сценариях см. в разделе [сценарии AD FS для разработчиков](/windows-server/identity/ad-fs/ad-fs-development).

Приложение, использующее MSAL для Java, будет взаимодействовать с Azure Active Directory (Azure AD), которое затем выполняет Федерацию в AD FS.

MSAL для Java подключается к Azure AD, который выполняет вход пользователей, управляемых в Azure AD (управляемых пользователей), или пользователей, управляемых другим поставщиком удостоверений, например AD FS (Федеративные пользователи). MSAL для Java не знает, что пользователь является федеративным. Это просто говорит с Azure AD.

В этом случае в качестве [центра](msal-client-application-configuration.md#authority) используется обычный центр (имя узла центра сертификации + клиент, общие или организации).

## <a name="acquire-a-token-interactively-for-a-federated-user"></a>Получение маркера в интерактивном режиме для федеративного пользователя

При вызове функции `ConfidentialClientApplication.AcquireToken()` или `PublicClientApplication.AcquireToken()` с помощью `AuthorizationCodeParameters` или `DeviceCodeParameters` , как правило, пользователь работает следующим образом:

1. Пользователь вводит идентификатор своей учетной записи.
2. В Azure AD кратко отображается «переход на страницу вашей организации», и пользователь перенаправляется на страницу входа поставщика удостоверений. Страница входа обычно настраивается с помощью логотипа Организации.

Поддерживаемые версии AD FS в этом Федеративной ситуации:
- службы федерации Active Directory (AD FS) FS v2
- Службы федерации Active Directory (AD FS) v3 (Windows Server 2012 R2)
- Службы федерации Active Directory (AD FS) v4 (AD FS 2016)

## <a name="acquire-a-token-via-username-and-password"></a>Получение маркера с помощью имени пользователя и пароля

При получении маркера с помощью `ConfidentialClientApplication.AcquireToken()` или `PublicClientApplication.AcquireToken()` с `IntegratedWindowsAuthenticationParameters` или `UsernamePasswordParameters` , MSAL для Java получает поставщик удостоверений для связи на основе имени пользователя. MSAL для Java получает маркер [маркера SAML 1,1](reference-saml-tokens.md) от поставщика удостоверений, который затем предоставляет Azure AD, который возвращает JSON Web Token (JWT).

## <a name="next-steps"></a>Дальнейшие действия

Сведения об объединенном варианте см. в статье [Настройка поведения при входе Azure Active Directory для приложения с помощью политики обнаружения домашней области](../manage-apps/configure-authentication-for-federated-users-portal.md) .