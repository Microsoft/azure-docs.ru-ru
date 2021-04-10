---
title: Добавление мультитенантного приложения в коллекцию приложений Azure AD
description: В этой статье объясняется, как включить специально разработанное мультитенантное приложение в коллекцию приложений Azure Active Directory.
services: active-directory
documentationCenter: na
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.reviewer: jeedes
ROBOTS: NOINDEX
ms.openlocfilehash: de0231a49c4f806660ea0cb305fdc1a70e2b36d3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98063133"
---
# <a name="add-a-multitenant-application-to-the-azure-ad-application-gallery"></a>Добавление мультитенантного приложения в коллекцию приложений Azure Active Directory

## <a name="what-is-the-azure-ad-application-gallery"></a>Что такое коллекция приложений Azure AD?

Azure Active Directory (Azure AD) — это облачная служба идентификации. [Коллекция приложений Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory) представляет собой хранилище приложений Azure Marketplace, в котором публикуются все соединители приложений для единого входа и подготовки пользователей. Клиенты, которые используют Azure AD в качестве поставщика удостоверений, находят опубликованные здесь различные соединители приложений SaaS. Администраторы ИТ-систем добавляют соединители из коллекции приложений, а затем настраивают и используют эти соединители для единого входа и подготовки. Azure Active Directory поддерживает для функции единого входа все основные протоколы федерации, в том числе SAML 2.0, OpenID Connect, OAuth и WS-Fed. 

## <a name="if-your-application-supports-saml-or-openidconnect"></a>Если приложение поддерживает SAML или OpenIDConnect
Если у вас есть многопользовательское приложение, которое вы хотите включить в коллекцию приложений Azure Active Directory, сначала убедитесь, что оно поддерживает одну из следующих технологий единого входа:

- **OpenID Connect**. чтобы приложение отображалось здесь, создайте мультитенантное приложение в Azure Active Directory и реализуйте в нем [инфраструктуру согласия Azure Active Directory](./consent-framework.md). Направляйте запрос на вход к общедоступной конечной точке, чтобы любой клиент мог предоставить согласие для приложения. Доступом пользователей можно управлять на основе идентификатора клиента и имени участника-пользователя, которые передаются в маркере. Отправьте приложение, следуя инструкциям в статье [Добавление приложения в коллекцию приложений Azure Active Directory](./v2-howto-app-gallery-listing.md).

- **SAML**: если приложение поддерживает SAML 2.0, оно может отображаться в коллекции. Следуйте инструкциям в статье [Добавление приложения в коллекцию приложений Azure Active Directory](./v2-howto-app-gallery-listing.md).

## <a name="if-your-application-does-not-support-saml-or-openidconnect"></a>Если приложение не поддерживает SAML или OpenIDConnect
Приложения, которые не поддерживают SAML и OpenIDConnect, все равно можно интегрировать в коллекцию приложений, используя технологию единого входа на основе пароля.

Единый вход на основе пароля (также называется хранение пароля) позволяет управлять доступом и паролями для веб-приложений, которые не поддерживают федерацию удостоверений. Также он полезен в ситуациях, когда несколько пользователей работают под одной учетной записью (например, с корпоративными страницами в социальных сетях). 

Чтобы ваше приложение поддерживало эту технологию, сделайте следующее:
1. Создайте веб-приложение с HTML-страницей единого входа, чтобы [настроить единый вход на основе пароля](../manage-apps/what-is-single-sign-on.md). 
2. Оправьте запрос, следуя инструкциям в статье [Добавление приложения в коллекцию приложений Azure Active Directory](./v2-howto-app-gallery-listing.md).

## <a name="escalations"></a>Эскалация вопросов

При эскалации любых вопросов отправьте электронное письмо [команде интеграции единого входа Azure Active Directory](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). Мы ответим вам как можно скорее.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как [добавить приложение в коллекцию приложений Azure Active Directory](./v2-howto-app-gallery-listing.md).