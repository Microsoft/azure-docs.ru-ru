---
title: Проверка подлинности на основе сертификата Android — Azure Active Directory
description: Узнайте о поддерживаемых сценариях и требованиях к настройке аутентификации на основе сертификата в решениях на устройствах Android.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: annaba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1001d5524fe99783cda4d5b77bdaceacc6791848
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96861379"
---
# <a name="azure-active-directory-certificate-based-authentication-on-android"></a>Аутентификация на основе сертификата в Azure Active Directory на устройстве Android

Для устройств Android можно применять аутентификацию на основе сертификатов (CBA) в Azure Active Directory. Для этого на устройстве используется сертификат клиента при подключении к:

* мобильным приложениям Office, таким как Microsoft Outlook и Microsoft Word;
* клиентам Exchange ActiveSync (EAS).

Настройка данной функции избавляет от необходимости ввода имени пользователя и пароля в определенных почтовых клиентах и приложениях Microsoft Office на мобильных устройствах.

В этом разделе приводятся требования и поддерживаемые сценарии для настройки CBA на устройстве Android для пользователей клиентов в планах Office 365 Enterprise, Business, образовательных, США, Китая и Германии.

В тарифных планах Office 365 US Government Defense и Federal доступна предварительная версия этой функции.

## <a name="microsoft-mobile-applications-support"></a>Поддержка мобильных приложений Microsoft

| Приложения | Поддержка |
| --- | --- |
| Приложение Azure Information Protection |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Intune Портал компании |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Microsoft Teams |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| OneNote |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| OneDrive |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Outlook |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Power BI |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Skype для бизнеса |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Word/Excel/PowerPoint |![Установите флажок, обозначающий поддержку для этого приложения][1] |
| Yammer |![Установите флажок, обозначающий поддержку для этого приложения][1] |

### <a name="implementation-requirements"></a>Требования к реализации

На устройстве должна быть установлена ОС Android версии 5.0 (Lollipop) и выше.

Необходимо настроить сервер федерации.

Чтобы служба Azure Active Directory могла отзывать сертификат клиента, маркер AD FS должен иметь следующие утверждения:

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>` (серийный номер сертификата клиента);
* `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>` (строка для издателя сертификата клиента).

Azure Active Directory добавляет эти утверждения в маркер обновления, если они доступны в маркере AD FS (или любом другом токене SAML). Когда требуется проверить маркер обновления, эта информация используется для проверки отзыва.

Рекомендуем обновить страницы ошибок AD FS вашей организации следующими сведениями:

* требованием установки Microsoft Authenticator для Android;
* инструкциями о получении сертификата пользователя.

Дополнительные сведения см. в статье о [настройке страниц входа AD FS](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn280950(v=ws.11)).

Некоторые приложения Office (с включенной современной проверкой подлинности) отправляют запрос "*Prompt = login*" в Azure AD в своем запросе. По умолчанию Azure AD преобразует "*Prompt = login*" в запрос в ADFS как "*wauth = усернамепассвордури*" (запрашивает ADFS для проверки подлинности U/P) и "*wfresh = 0*" (запрашивает ADFS игнорировать состояние единого входа и выполняет новую проверку подлинности). Чтобы включить проверку подлинности на основе сертификатов для этих приложений, необходимо изменить поведение Azure AD по умолчанию. Задайте для параметра "*промптлогинбехавиор*" в параметрах федеративного домена значение "*disabled" (отключено*).
Для выполнения этой задачи можно использовать командлет [MSOLDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings):

`Set-MSOLDomainFederationSettings -domainname <domain> -PromptLoginBehavior Disabled`

## <a name="exchange-activesync-clients-support"></a>Поддержка клиентов Exchange ActiveSync

Поддерживаются некоторые приложения Exchange ActiveSync, работающие на Android 5.0 (Lollipop) или более поздней версии. Чтобы определить, поддерживает ли почтовая программа эту функцию, обратитесь к разработчику приложения.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы настроить аутентификацию на основе сертификата в своей среде, ознакомьтесь с инструкциями в статье [Get started with certificate-based authentication in Azure Active Directory](active-directory-certificate-based-authentication-get-started.md) (Приступая к работе с аутентификацией на основе сертификата в Azure Active Directory).

<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-android/ic195031.png
