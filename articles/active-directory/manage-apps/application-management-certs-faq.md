---
title: Часто задаваемые вопросы о Azure Active Directory сертификатах управления приложениями
description: Получите ответы на часто задаваемые вопросы об управлении сертификатами для приложений с помощью Azure Active Directory в качестве поставщика удостоверений (IdP).
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: reference
ms.date: 03/19/2021
ms.author: kenwith
ms.reviewer: secherka, mifarca, shchaur, shravank, sureshja
ms.openlocfilehash: 928bf02e2d628379738483b40631e36f0f929176
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104803806"
---
# <a name="azure-active-directory-azure-ad-application-management-certificates-frequently-asked-questions"></a>Часто задаваемые вопросы о сертификатах управления приложениями Azure Active Directory (Azure AD)

На этой странице приведены часто задаваемые вопросы об управлении сертификатами для приложений с помощью Azure Active Directory (Azure AD) в качестве поставщика удостоверений (IdP).

## <a name="is-there-a-way-to-generate-a-list-of-expiring-saml-signing-certificates"></a>Существует ли способ создания списка истечения срока действия сертификатов подписи SAML?

Вы можете экспортировать все регистрации приложений с истекшими секретами, сертификатами и владельцами для указанных приложений из каталога в CSV-файле с помощью [сценариев PowerShell](app-management-powershell-samples.md). 

## <a name="where-can-i-find-the-information-about-soon-to-expire-certificates-renewal-steps"></a>Где можно найти сведения о скором окончании срока действия обновления сертификатов?

Инструкции можно найти [здесь](manage-certificates-for-federated-single-sign-on.md#renew-a-certificate-that-will-soon-expire).

## <a name="how-can-i-customize-the-expiration-date-for-the-certificates-issued-by-azure-ad"></a>Как настроить дату истечения срока действия сертификатов, выданных Azure AD?

По умолчанию Azure AD настраивает срок действия сертификата в течение трех лет, когда он создается автоматически во время настройки единого входа SAML. Так как вы не можете изменить дату сертификата после его сохранения, необходимо создать новый сертификат. Дополнительные сведения о том, как это сделать, см. в разделе [Настройка срока действия сертификата Федерации и перенесите его на новый сертификат](manage-certificates-for-federated-single-sign-on.md#customize-the-expiration-date-for-your-federation-certificate-and-roll-it-over-to-a-new-certificate).

## <a name="how-can-i-automate-the-certificates-expiration-notifications"></a>Как можно автоматизировать уведомления об истечении срока действия сертификатов?

Azure AD будет отправлять по электронной почте уведомление 60, 30 и 7 дней до истечения срока действия сертификата SAML. Вы можете добавить более одного адреса электронной почты для получения уведомлений. 

> [!NOTE]
> В список уведомлений можно добавить до 5 адресов электронной почты (включая адрес электронной почты администратора, который добавил это приложение). Если вам требуется больше пользователей, чтобы получать уведомления, используйте список рассылки по электронной почте. 

Чтобы указать сообщения электронной почты, на которые должны отправляться уведомления, см. раздел [Добавление адресов уведомлений по электронной почте для истечения срока действия сертификата](manage-certificates-for-federated-single-sign-on.md#add-email-notification-addresses-for-certificate-expiration).

Нет возможности изменить или настроить эти уведомления по электронной почте, полученные от `aadnotification@microsoft.com` . Однако вы можете экспортировать регистрации приложений с истечением срока действия секретов и сертификатов с помощью [сценариев PowerShell](app-management-powershell-samples.md).

## <a name="who-can-update-the-certificates"></a>Кто может обновлять сертификаты?

Владелец приложения или глобальный администратор или администратор приложения может обновить сертификаты с помощью портал Azure пользовательского интерфейса, PowerShell или Microsoft Graph.

## <a name="i-need-more-details-about-certificate-signing-options"></a>Мне нужны дополнительные сведения о параметрах подписывания сертификата.

В Azure AD можно настроить параметры подписи сертификата и алгоритм подписи сертификата. Дополнительные сведения см. в разделе [Дополнительные параметры подписи сертификата маркера SAML для приложений Azure AD](certificate-signing-options.md).

## <a name="i-need-to-replace-the-certificate-for-azure-ad-application-proxy-applications-and-need-more-instructions"></a>Мне нужно заменить сертификат для приложений Azure AD Application Proxy и потребуются дополнительные инструкции.

Сведения о замене сертификатов для приложений Azure AD Application Proxy см. [в статье пример PowerShell. Замена сертификата в приложениях прокси приложения](scripts/powershell-get-custom-domain-replace-cert.md).

## <a name="how-do-i-manage-certificates-for-custom-domains-in-azure-ad-application-proxy"></a>Разделы справки управлять сертификатами для пользовательских доменов в AD Application Proxy Azure?

Чтобы настроить локальное приложение на использование личного домена, необходимы проверенный личный домен Azure Active Directory, PFX-сертификат для личного домена и локальное приложение для настройки. Дополнительные сведения см. [в разделе Личные домены в Azure AD application proxy](application-proxy-configure-custom-domain.md). 

## <a name="i-need-to-update-the-token-signing-certificate-on-the-application-side-where-can-i-get-it-on-azure-ad-side"></a>Мне нужно обновить сертификат подписи маркера на стороне приложения. Где можно получить его на стороне Azure AD?

Вы можете продлить сертификат SAML X. 509, см. раздел [сертификат подписи SAML](configure-saml-single-sign-on.md#saml-signing-certificate).

## <a name="what-is-azure-ad-signing-key-rollover"></a>Что такое смена ключей подписывания Azure AD?

Дополнительные сведения можно найти [здесь](../develop/active-directory-signing-key-rollover.md). 

## <a name="how-do-i-renew-application-token-encryption-certificate"></a>Разделы справки продлить сертификат шифрования маркеров приложений?

Чтобы продлить сертификат шифрования маркеров приложений, см. статью [как продлить сертификат шифрования маркеров для корпоративного приложения](howto-saml-token-encryption.md). 

## <a name="how-do-i-renew-application-token-signing-certificate"></a>Разделы справки продлить сертификат подписи маркера приложения?

Чтобы продлить сертификат подписи маркера приложения, см. статью [как продлить сертификат подписи маркера для корпоративного приложения](manage-certificates-for-federated-single-sign-on.md).

## <a name="how-do-i-update-azure-ad-after-changing-my-federation-certificates"></a>Разделы справки обновить Azure AD после изменения сертификатов федерации?

Сведения об обновлении Azure AD после изменения сертификатов Федерации см. в статье [продление сертификатов федерации для Microsoft 365 и Azure Active Directory](../hybrid/how-to-connect-fed-o365-certs.md).
