---
title: Как платформа Microsoft Identity использует протокол SAML
description: В этой статье содержится обзор профилей SAML для единого входа и единого выхода в Azure Active Directory.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/05/2018
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 4e9d63b8fe7fc281a87deb27ddadd794e14fa04d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98755626"
---
# <a name="how-the-microsoft-identity-platform-uses-the-saml-protocol"></a>Как платформа Microsoft Identity использует протокол SAML

Платформа Microsoft Identity использует протокол SAML 2,0 для предоставления приложениям возможности единого входа пользователям. Профили [единого входа](single-sign-on-saml-protocol.md) и [единого выхода](single-sign-out-saml-protocol.md) SAML в Azure AD содержат сведения о том, как в службе поставщика удостоверений используются утверждения SAML, протоколы и привязки.

Протокол SAML требует, чтобы поставщик удостоверений (платформа идентификации Майкрософт) и поставщик услуг (приложение) могли обмениваться информацией о себе.

При регистрации приложения в Azure AD разработчик приложения регистрирует в Azure AD сведения, относящиеся к федерации. К ним относятся **URI перенаправления** и **URI метаданных** для приложения.

Платформа Microsoft Identity использует **URI метаданных** облачной службы для получения ключа подписывания и URI выхода. Пользователь может открыть приложение в разделе **Azure AD -> Регистрация приложения**, а затем обновить URL-адрес выхода в разделе **Параметры -> Свойства**. Таким образом платформа Microsoft Identity может отправить ответ на правильный URL-адрес. 

Azure Active Directory предоставляет клиентские и общие (единые для всех клиентов) конечные точки единого входа и единого выхода. Эти URL-адреса представляют собой адресуемые расположения, а не просто идентификаторы. Вы можете обратиться по ним к конечной точке для чтения метаданных.

* Конечная точка конкретного клиента находится по адресу `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`. *\<TenantDomainName>* Заполнитель представляет зарегистрированное доменное имя или идентификатор GUID TenantID клиента Azure AD. Например, метаданные федерации клиента contoso.com находится здесь: https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* Независимая от клиента конечная точка расположена по адресу `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`. В этом адресе конечной точки вместо доменного имени или идентификатора клиента содержится ключевое слово **common**.

Сведения о документах метаданных федерации, публикуемых Azure AD, см. в разделе [метаданные федерации](../azuread-dev/azure-ad-federation-metadata.md).
