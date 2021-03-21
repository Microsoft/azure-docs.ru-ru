---
title: Добавление поставщика удостоверений — Azure Active Directory B2C | Документация Майкрософт
description: Узнайте, как добавить поставщик удостоверений в клиент Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 03/03/2021
ms.custom: mvc
ms.topic: how-to
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: dbb2aeefc22fae4a3c83956dd7681b49fc435500
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102034002"
---
# <a name="add-an-identity-provider-to-your-azure-active-directory-b2c-tenant"></a>Добавление поставщика удостоверений в клиент Azure Active Directory B2C

Azure AD B2C позволяет разрешить пользователям вход в приложение с учетными данными внешних поставщиков удостоверений (социальных сетей или предприятий). Azure AD B2C поддерживает Facebook, учетные записи Майкрософт, Google, Твиттера и других внешних поставщиков удостоверений, совместимых с протоколами OAuth 1.0, OAuth 2.0, OpenID Connect и SAML.

Благодаря федерации с внешними поставщиками удостоверений вы сможете предложить потребителям возможность входа с их существующими учетными записями социальных сетей или предприятий, не вынуждая создавать новую учетную запись только для своего приложения.

На странице регистрации и входа в Azure AD B2C представлен список внешних поставщиков удостоверений, которые пользователь может выбрать для входа. Выбрав один из внешних поставщиков удостоверений, пользователь попадает на веб-сайт этого поставщика, где сможет завершить процесс входа. После успешного входа пользователь возвращается в Azure AD B2C для аутентификации учетной записи в вашем приложении.

![Пример входа с мобильного устройства с использованием учетной записи социальной сети (Facebook)](media/add-identity-provider/external-idp.png)

С помощью портала Azure вы можете добавить поставщики удостоверений, которые поддерживает Azure Active Directory B2C (Azure AD B2C), в свои [потоки пользователя](user-flow-overview.md). Поставщики удостоверений также можно добавить в [пользовательские политики](custom-policy-get-started.md).

## <a name="select-an-identity-provider"></a>Выберите поставщик удостоверений

Обычно в приложениях используется только один поставщик удостоверений, но у вас есть возможность добавить еще. В статьях с инструкциями ниже показано, как создать приложение поставщика удостоверений, добавить поставщик удостоверений в клиент и добавить поставщик удостоверений в пользовательский поток или настраиваемую политику.

* [AD FS](identity-provider-adfs.md)
* [Amazon](identity-provider-amazon.md)
* [Apple](identity-provider-apple-id.md);
* [Azure AD (один клиент)](identity-provider-azure-ad-single-tenant.md)
* [Azure AD (несколько клиентов)](identity-provider-azure-ad-multi-tenant.md)
* [Azure AD B2C](identity-provider-azure-ad-b2c.md)
* [Facebook](identity-provider-facebook.md)
* [Универсальный поставщик удостоверений](identity-provider-generic-openid-connect.md)
* [GitHub](identity-provider-github.md)
* [ID.me](identity-provider-id-me.md)
* [Google](identity-provider-google.md)
* [LinkedIn](identity-provider-linkedin.md)
* [Учетная запись Майкрософт](identity-provider-microsoft-account.md)
* [QQ](identity-provider-qq.md)
* [Salesforce](identity-provider-salesforce.md)
* [SalesForce (протокол SAML)](identity-provider-salesforce-saml.md)
* [Twitter](identity-provider-twitter.md)
* [WeChat](identity-provider-wechat.md)
* [Weibo](identity-provider-weibo.md)
