---
title: Протоколы проверки подлинности платформы Microsoft Identity
description: Обзор протоколов проверки подлинности, поддерживаемых платформой идентификации Майкрософт
author: rwike77
services: active-directory
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 12/18/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: hirsin
ROBOTS: NOINDEX
ms.openlocfilehash: 803f52666fb09a836eae3294410584ebbd71a1b6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98755885"
---
# <a name="microsoft-identity-platform-authentication-protocols"></a>Протоколы проверки подлинности платформы Microsoft Identity

Платформа Microsoft Identity поддерживает несколько наиболее широко используемых протоколов проверки подлинности и авторизации. В подразделах этого раздела описываются поддерживаемые протоколы и их реализация на платформе Microsoft Identity. Эти разделы содержат обзор поддерживаемых типов утверждений, основные сведения об использовании метаданных федерации, подробную документацию по протоколам OAuth 2.0. и SAML 2.0, а также советы по устранению неполадок.

## <a name="authentication-protocols-articles-and-reference"></a>Статьи и Справочник по протоколам проверки подлинности

* [Важная информация о смене ключей подписывания на платформе Microsoft Identity](active-directory-signing-key-rollover.md) . сведения о частоте смены ключей подписывания для платформы идентификации Майкрософт, изменения, которые можно внести для автоматического обновления ключа, и обсуждение способов обновления наиболее распространенных сценариев приложений.
* [Поддерживаемые типы маркеров и утверждений](id-tokens.md) . сведения об утверждениях в маркерах, устраняемых платформой идентификации Майкрософт.
* [Oauth 2,0 на платформе Microsoft Identity](v2-oauth2-auth-code-flow.md) — сведения о реализации OAuth 2,0 на платформе Microsoft Identity.
* [OpenID Connect 1.0](v2-protocols-oidc.md) — сведения об использовании протокола авторизации OAuth 2.0 для проверки подлинности.
* [Служба обслуживания вызовов с помощью учетных данных клиента](v2-oauth2-client-creds-grant-flow.md) — сведения об использовании потока предоставления учетных данных клиента OAuth 2.0 для вызовов между службами.
* [Служба обслуживания вызовов с помощью потока On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md) — сведения об использовании потока On-Behalf-Of OAuth 2.0 для вызовов между службами.
* [Справочник по протоколу SAML](active-directory-saml-protocol-reference.md) . сведения об одном Sign-On и профили SAML единого выхода для платформы Microsoft Identity.

## <a name="see-also"></a>См. также раздел

* [Общие сведения о платформе удостоверений Майкрософт](v2-overview.md)
* [Примеры кода Active Directory](sample-v2-code.md)
