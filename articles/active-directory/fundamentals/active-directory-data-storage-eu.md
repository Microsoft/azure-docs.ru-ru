---
title: Хранилище данных удостоверений для европейских клиентов — Azure AD
description: Узнайте, где Azure Active Directory хранит данные, связанные с идентификаторами клиентов из ЕС.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 09/15/2020
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7a8b013723707c4a3a087a90674227c3d41c5108
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94836943"
---
# <a name="identity-data-storage-for-european-customers-in-azure-active-directory"></a>Хранение данных удостоверений клиентов из ЕС — Azure Active Directory
Данные удостоверений хранятся в службе Azure AD в географическом расположении на основе адреса, предоставленного вашей организацией, при подписке на службу Microsoft Online Service, например Microsoft 365 и Azure. Сведения о том, где хранятся данные удостоверений, можно найти в разделе [где находятся ваши данные?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) центра управления безопасностью Майкрософт.

Для клиентов, которые указали адрес в Европе, Azure AD хранит большую часть данных удостоверений в европейских центрах данных. В этом документе содержатся сведения о любых данных, хранящихся за пределами Европы службами Azure AD.

## <a name="microsoft-azure-ad-multi-factor-authentication"></a>Многофакторная проверка подлинности Microsoft Azure AD

Для облачной многофакторной идентификации Azure AD проверка подлинности завершается в ближайшем центре обработки данных для пользователя. Центры обработки данных для многофакторной идентификации Azure AD существуют в Северная Америка, Европе и Азиатско-Тихоокеанский регион.

* Многофакторная идентификация, использующая телефонные звонки, исходит от центров обработки данных США и направляется глобальными поставщиками.
* Многофакторная идентификация с помощью SMS направляется глобальными поставщиками.
* Запросы многофакторной проверки подлинности, использующие Microsoft Authenticator приложения push-уведомления, полученные из центров обработки данных ЕС, обрабатываются в центрах обработки данных ЕС.
    * Службы, зависящие от поставщика устройств, такие как push-уведомления Apple, могут находиться за пределами Европы.
* Запросы многофакторной проверки подлинности, использующие OATH-коды, полученные из центров обработки данных ЕС, проверяются в ЕС.

Дополнительные сведения о том, какие сведения о пользователях собираются с помощью Azure сервер Многофакторной идентификации (сервер MFA) и в облачной службе Azure AD MFA, см. в статье [сбор данных пользователей многофакторной идентификации Azure](../authentication/howto-mfa-reporting-datacollection.md).

## <a name="password-based-single-sign-on-for-enterprise-applications"></a>Единый Sign-On на основе пароля для корпоративных приложений
 
Если клиент создает новое корпоративное приложение (с помощью коллекции Azure AD или без галереи) и включает единый вход на основе пароля, то URL-адрес входа приложения и поля пользовательского входа в систему сохраняются в США. Дополнительные сведения об этой функции см. в статье [Настройка единого входа на основе пароля](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md) .

## <a name="microsoft-azure-active-directory-b2c-azure-ad-b2c"></a>Microsoft Azure Active Directory B2C (Active Directory B2C)

Данные конфигурации политики Azure AD B2C и контейнеры ключей хранятся в центрах обработки данных США. Они не содержат персональных данных пользователя. Дополнительные сведения о конфигурациях политики см. в статье [Azure Active Directory B2C: встроенные политики](../../active-directory-b2c/user-flow-overview.md).

## <a name="microsoft-azure-active-directory-b2b-azure-ad-b2b"></a>Microsoft Azure Active Directory B2B (Active Directory B2B) 
    
Azure AD B2B хранит приглашения со ссылками для активации и URL-адресом перенаправления в центрах обработки данных США. Кроме того, адреса электронной почты пользователей, которые отменяют подписываться на получение приглашений B2B, также хранятся в центрах обработки данных США.

## <a name="microsoft-azure-active-directory-domain-services-azure-ad-ds"></a>Доменные службы Microsoft Azure Active Directory (Azure AD DS)

Azure AD DS хранит данные пользователей в том же расположении, где размещена виртуальная сеть Azure, выбранная клиентом. Таким образом, если сеть находится за пределами Европы, то данные реплицируются и хранятся за пределами Европы.

## <a name="federation-in-microsoft-exchange-server-2013"></a>Федерация в Microsoft Exchange Server 2013
    
- Идентификатор приложения (AppID) — уникальный номер, созданный системой проверки подлинности Azure Active Directory для идентификации организаций Exchange.
- Список утвержденных федеративных доменов для приложения
- Открытый ключ подписи маркера приложения 

Дополнительные сведения о Федерации в Microsoft Exchange Server см. в статье [интеграция: Exchange 2013](/exchange/federation-exchange-2013-help) .


## <a name="other-considerations"></a>Другие замечания

Службы и приложения, которые интегрируются с Azure AD, имеют доступ к данным удостоверений. Оцените каждую службу и приложение, используемые для определения того, как данные удостоверений обрабатываются этой конкретной службой и приложением, а также соответствуют ли они требованиям к хранению данных вашей компании.

Дополнительные сведения о местонахождении данных служб Майкрософт см. в разделе [Где находятся ваши данные?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) Центра управления безопасностью Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об этих функциях и возможностях, описанных выше, смотрите в следующих статьях.
- [Что такое Многофакторная идентификация Azure?](../authentication/concept-mfa-howitworks.md)

- [Самостоятельный сброс пароля в Azure AD](../authentication/concept-sspr-howitworks.md)

- [Что такое Azure Active Directory B2C?](../../active-directory-b2c/overview.md)

- [Что такое служба совместной работы Azure AD B2B?](../external-identities/what-is-b2b.md)

- [Доменные службы Azure Active Directory (AD)](../../active-directory-domain-services/overview.md)