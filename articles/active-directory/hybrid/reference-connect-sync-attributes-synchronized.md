---
title: Атрибуты, синхронизируемые службой Azure AD Connect | Документация Майкрософт
description: В этой статье перечислены атрибуты, которые синхронизируются с Azure Active Directory.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: c2bb36e0-5205-454c-b9b6-f4990bcedf51
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 04/15/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6ec05c4160c6502904644bf7035bda0bed66cc33
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94413196"
---
# <a name="azure-ad-connect-sync-attributes-synchronized-to-azure-active-directory"></a>Службы синхронизации Azure AD Connect: атрибуты, синхронизируемые с Azure Active Directory
В этой статье перечислены атрибуты, которые синхронизируются при помощи служб синхронизации Azure AD Connect.  
Атрибуты сгруппированы по соответствующим приложениям Azure AD.

## <a name="attributes-to-synchronize"></a>Атрибуты для синхронизации
Распространенный вопрос состоит в том, *какой минимальный перечень атрибутов следует синхронизировать*. По умолчанию и рекомендуемый подход заключается в том, чтобы хранить атрибуты по умолчанию, чтобы полный глобальный список адресов (Global Address List) можно было создать в облаке и получить все функции в Microsoft 365 рабочих нагрузках. В некоторых случаях существует несколько атрибутов, которые Организация не хочет синхронизировать с облаком, так как эти атрибуты содержат конфиденциальные персональные данные, как в этом примере:  
![Некорректные атрибуты](./media/reference-connect-sync-attributes-synchronized/badextensionattribute.png)

В этом случае начните со списка атрибутов в этом разделе и найдите эти атрибуты, которые будут содержать персональные данные и не могут быть синхронизированы. Отмените их выбор во время установки с помощью [фильтра приложений и атрибутов Azure AD](how-to-connect-install-custom.md#azure-ad-app-and-attribute-filtering).

> [!WARNING]
> При отмене выбора атрибутов необходимо соблюдать осторожность, чтобы снять флажки только тех из них, которые абсолютно невозможно синхронизировать. Отмена выбора других атрибутов может отрицательно повлиять на функции.
>
>

## <a name="microsoft-365-apps-for-enterprise"></a>Приложения Microsoft 365 для предприятия
| Имя атрибута | Пользователь | Комментировать |
| --- |:---:| --- |
| AccountEnabled |X |Определяет, включена ли учетная запись. |
| cn |X | |
| displayName |X | |
| objectSID |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| pwdLastSet |X |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
|samAccountName|X| |
| sourceAnchor |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| usageLocation |X |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |

## <a name="exchange-online"></a>Exchange Online
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| assistant |X |X | | |
| altRecipient |X | | |Требуется Azure AD Connect сборки 1.1.552.0 или более поздних версий. |
| authOrig |X |X |X | |
| с |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| description; | | |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| homePhone |X |X | | |
| сведения |X |X |X |Сейчас этот атрибут не используется для групп. |
| Инициалы |X |X | | |
| l |X |X | | |
| legacyExchangeDN |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| член | | |X | |
| mobile |X |X | | |
| msDS-HABSeniorityIndex |X |X |X | |
| msDS-PhoneticDisplayName |X |X |X | |
| msExchArchiveGUID |X | | | |
| msExchArchiveName |X | | | |
| msExchAssistantName |X |X | | |
| msExchAuditAdmin |X | | | |
| msExchAuditDelegate |X | | | |
| msExchAuditDelegateAdmin |X | | | |
| msExchAuditOwner |X | | | |
| msExchBlockedSendersHash |X |X | | |
| msExchBypassAudit |X | | | |
| msExchBypassModerationLink | | |X |Доступно в версии в Azure AD Connect 1.1.524.0 |
| msExchCoManagedByLink | | |X | |
| msExchDelegateListLink |X | | | |
| msExchELCExpirySuspensionEnd |X | | | |
| msExchELCExpirySuspensionStart |X | | | |
| msExchELCMailboxFlags |X | | | |
| msExchEnableModeration |X | |X | |
| msExchExtensionCustomAttribute1 |X |X |X |Сейчас этот атрибут не используется Exchange Online. |
| msExchExtensionCustomAttribute2 |X |X |X |Сейчас этот атрибут не используется Exchange Online. |
| msExchExtensionCustomAttribute3 |X |X |X |Сейчас этот атрибут не используется Exchange Online. |
| msExchExtensionCustomAttribute4 |X |X |X |Сейчас этот атрибут не используется Exchange Online. |
| msExchExtensionCustomAttribute5 |X |X |X |Сейчас этот атрибут не используется Exchange Online. |
| msExchHideFromAddressLists |X |X |X | |
| msExchImmutableID |X | | | |
| msExchLitigationHoldDate |X |X |X | |
| msExchLitigationHoldOwner |X |X |X | |
| msExchMailboxAuditEnable |X | | | |
| msExchMailboxAuditLogAgeLimit |X | | | |
| msExchMailboxGuid |X | | | |
| msExchModeratedByLink |X |X |X | |
| msExchModerationFlags |X |X |X | |
| msExchRecipientDisplayType |X |X |X | |
| msExchRecipientTypeDetails |X |X |X | |
| msExchRemoteRecipientType |X | | | |
| msExchRequireAuthToSendTo |X |X |X | |
| msExchResourceCapacity |X | | | |
| msExchResourceDisplay |X | | | |
| msExchResourceMetaData |X | | | |
| msExchResourceSearchProperties |X | | | |
| msExchRetentionComment |X |X |X | |
| msExchRetentionURL |X |X |X | |
| msExchSafeRecipientsHash |X |X | | |
| msExchSafeSendersHash |X |X | | |
| msExchSenderHintTranslations |X |X |X | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| msExchUserHoldPolicies |X | | | |
| msOrg-IsOrganizational | | |X | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| proxyAddresses |X |X |X | |
| publicDelegates |X |X |X | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется как в синхронизации паролей, так и в федерации. |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| sn |X |X | | |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| TelephoneNumber |X |X | | |
| thumbnailphoto |X |X | |синхронизируется только один раз из Azure AD в Exchange Online, после чего Exchange Online получает источник полномочий для этого атрибута, и все последующие изменения не могут быть синхронизированы из локальной системы. Дополнительные сведения см. в разделе ([КБ](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| unauthOrig |X |X |X | |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userCertificate |X |X | | |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |
| userSMIMECertificates |X |X | | |
| wWWHomePage |X |X | | |

## <a name="sharepoint-online"></a>SharePoint Online
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| authOrig |X |X |X | |
| с |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| description; |X |X |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| hideDLMembership | | |X | |
| homePhone |X |X | | |
| сведения |X |X |X | |
| Initials |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| член | | |X | |
| middleName |X |X | | |
| mobile |X |X | | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointLinkedBy |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherIpPhone |X |X | | |
| otherMobile |X |X | | |
| otherPager |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| postOfficeBox |X |X | |Сейчас этот атрибут не используется SharePoint Online. |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| sn |X |X | | |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| TelephoneNumber |X |X | | |
| thumbnailphoto |X |X | |синхронизируется только один раз из Azure AD в Exchange Online, после чего Exchange Online получает источник полномочий для этого атрибута, и все последующие изменения не могут быть синхронизированы из локальной системы. Дополнительные сведения см. в разделе ([КБ](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| unauthOrig |X |X |X | |
| url |X |X | | |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя
. Используется для назначения лицензии. |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |
| wWWHomePage |X |X | | |

## <a name="teams-and-skype-for-business-online"></a>Команды и Skype для бизнеса Online
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| с |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| department |X |X | | |
| description; |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X |X | |
| givenName |X |X | | |
| homePhone |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| член | | |X | |
| mobile |X |X | | |
| msExchHideFromAddressLists |X |X |X | |
| msRTCSIP-ApplicationOptions |X | | | |
| msRTCSIP-DeploymentLocator |X |X | | |
| msRTCSIP-Line |X |X | | |
| msRTCSIP-OptionFlags |X |X | | |
| msRTCSIP-OwnerUrn |X | | | |
| msRTCSIP-PrimaryUserAddress |X |X | | |
| msRTCSIP-UserEnabled |X |X | | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| otherTelephone |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
| sn |X |X | | |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| TelephoneNumber |X |X | | |
| thumbnailphoto |X |X | |синхронизируется только один раз из Azure AD в Exchange Online, после чего Exchange Online получает источник полномочий для этого атрибута, и все последующие изменения не могут быть синхронизированы из локальной системы. Дополнительные сведения см. в разделе ([КБ](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |
| wWWHomePage |X |X | | |

## <a name="azure-rms"></a>Azure RMS
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| cn |X | |X |Общее имя или псевдоним. Чаще всего префикс значения [mail]. |
| displayName |X |X |X |Строка, представляющая имя. Часто отображается как понятное имя (имя и фамилия). |
| mail |X |X |X |Полный адрес электронной почты. |
| член | | |X | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| proxyAddresses |X |X |X |Механическое свойство. Используется в Azure AD. Содержит все дополнительные адреса электронной почты пользователя. |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X | | |Это имя участника-пользователя является именем пользователя для входа. Чаще всего соответствует значению [mail]. |

## <a name="intune"></a>Intune
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| с |X |X | | |
| cn |X | |X | |
| description; |X |X |X | |
| displayName |X |X |X | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| член | | |X | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |

## <a name="dynamics-crm"></a>Dynamics CRM
| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| с |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| description; |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| l |X |X | | |
| managedBy | | |X | |
| manager |X |X | | |
| член | | |X | |
| mobile |X |X | | |
| objectSID |X | |X |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
| sn |X |X | | |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| TelephoneNumber |X |X | | |
| title |X |X | | |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |

## <a name="3rd-party-applications"></a>Сторонние приложения
Эта группа представляет собой минимальный набор атрибутов, необходимых для универсальной рабочей нагрузки или приложения. Он может использоваться для рабочей нагрузки, не указанной в другом разделе, или приложения стороннего производителя. Явным образом он используется для следующего:

* Yammer (используется только пользователь);
* [гибридные сценарии межорганизационного взаимодействия бизнес-бизнес, обеспечиваемые такими ресурсами, как SharePoint.](/sharepoint/create-b2b-extranet)

Эта группа представляет собой набор атрибутов, которые можно использовать, если каталог Azure AD не используется для поддержки Microsoft 365, Dynamics или Intune. Это небольшой набор основных атрибутов. Обратите внимание, что единый вход или подготовка к некоторым сторонним приложениям требует настройки синхронизации атрибутов в дополнение к описанным здесь атрибутам. Требования к приложениям описаны в руководстве по приложениям [SaaS](../saas-apps/tutorial-list.md) для каждого приложения.

| Имя атрибута | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |
| AccountEnabled |X | | |Определяет, включена ли учетная запись. |
| cn |X | |X | |
| displayName |X |X |X | |
| employeeID |X |  |  | |
| givenName |X |X | | |
| mail |X | |X | |
| managedBy | | |X | |
| mailNickName |X |X |X | |
| член | | |X | |
| objectSID |X | | |Механическое свойство. Идентификатор пользователя AD, используемый для обеспечения синхронизации между Azure AD и AD. |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |Механическое свойство. Используется, чтобы определить, когда необходимо сделать недействительными уже выданные маркеры. Используется для синхронизации хэша паролей, сквозной аутентификации и федерации. |
| sn |X |X | | |
| sourceAnchor |X |X |X |Механическое свойство. Неизменяемый идентификатор для поддержания связи между доменными службами Active Directory и Azure AD. |
| usageLocation |X | | |Механическое свойство. Страна или регион пользователя. Используется для назначения лицензии. |
| userPrincipalName |X | | |Имя участника-пользователя является именем для входа. Чаще всего соответствует значению [mail]. |

## <a name="windows-10"></a>Windows 10
Компьютер (устройство), присоединенное к домену Windows 10, синхронизирует некоторые атрибуты с Azure AD. Дополнительные сведения о сценариях см. в статье [Подключение присоединенных к домену устройств к Azure AD для работы в Windows 10](../devices/hybrid-azuread-join-plan.md). Эти атрибуты всегда синхронизируются, и Windows 10 отображается как приложение, выбор которого нельзя отменить. Компьютер, присоединенный к домену Windows 10, определяется заполненным атрибутом userCertificate.

| Имя атрибута | Устройство | Комментировать |
| --- |:---:| --- |
| AccountEnabled |X | |
| deviceTrustType |X |Жестко запрограммированное значение для компьютеров, присоединенных к домену. |
| displayName |X | |
| MS-DS-CreatorSID |X |Другое название registeredOwnerReference. |
| objectGUID |X |Другое название deviceID. |
| objectSID |X |Другое название onPremisesSecurityIdentifier. |
| operatingSystem |X |Другое название deviceOSType. |
| operatingSystemVersion |X |Другое название deviceOSVersion. |
| userCertificate |X | |

Эти атрибуты для **пользователя** представлены в дополнение к другим выбранным приложениям.  

| Имя атрибута | Пользователь | Комментировать |
| --- |:---:| --- |
| domainFQDN |X |Другое название dnsDomainName. Например, contoso.com. |
| domainNetBios |X |Другое название netBiosName. Например, CONTOSO. |
| msDS-KeyCredentialLink |X |После регистрации пользователя в Windows Hello для бизнеса. | 

## <a name="exchange-hybrid-writeback"></a>Гибридная обратная запись Exchange
Эти атрибуты записываются из Azure AD в локальную службу Active Directory при активации **гибридного развертывания Exchange**. В зависимости от установленной версии Exchange может синхронизироваться меньшее количество атрибутов.

| Имя атрибута (локальная служба AD) | Имя атрибута (пользовательский интерфейс Connect) | Пользователь | Contact | Group | Комментировать |
| --- |:---:|:---:|:---:| --- |---|
| msDS-ExternalDirectoryObjectID| ms-DS-External-Directory-Object-Id |X | | |Производный от cloudAnchor в Azure AD. Это новый атрибут в Exchange 2016 и Windows Server 2016 AD. |
| msExchArchiveStatus| ms-Exch-ArchiveStatus |X | | |Серверный архив: позволяет клиентам архивировать почту. |
| msExchBlockedSendersHash| ms-Exch-BlockedSendersHash |X | | |Фильтрация: записывает обратно данные локальной фильтрации и сетевые данные о безопасных и заблокированных отправителях от клиентов. |
| msExchSafeRecipientsHash| MS-Exch-SafeRecipientsHash  |X | | |Фильтрация: записывает обратно данные локальной фильтрации и сетевые данные о безопасных и заблокированных отправителях от клиентов. |
| msExchSafeSendersHash| ms-Exch-SafeSendersHash  |X | | |Фильтрация: записывает обратно данные локальной фильтрации и сетевые данные о безопасных и заблокированных отправителях от клиентов. |
| msExchUCVoiceMailSettings| ms-Exch-UCVoiceMailSettings |X | | |Включение единой системы обмена сообщениями (сетевая голосовая почта): используется при интеграции сервера Microsoft Lync Server, указывая локальному серверу Lync Server, что у пользователя в онлайн-службах есть голосовая почта. |
| msExchUserHoldPolicies| MS-дов-УсерхолдполиЦиес |X | | |Хранение для судебного разбирательства: позволяет облачным службам определять, на каких пользователей действует хранение для судебного разбирательства. |
| proxyAddresses| proxyAddresses |X |X |X |Вставляется только адрес x500 из Exchange Online. |
| publicDelegates| ms-Exch-Public-Delegates  |X | | |Позволяет в почтовом ящике Exchange Online предоставлять права SendOnBehalfTo пользователям с локальными почтовыми ящиками Exchange. Требуется Azure AD Connect сборки 1.1.552.0 или более поздних версий. |

## <a name="exchange-mail-public-folder"></a>Общедоступная папка почты Exchange
Эти атрибуты синхронизируются из локального экземпляра Active Directory в Azure AD при включении **общедоступных почтовых папок Exchange**.

| Имя атрибута | PublicFolder | Комментировать |
| --- | :---:| --- |
| displayName | X |  |
| mail | X |  |
| msExchRecipientTypeDetails | X |  |
| objectGUID | X |  |
| proxyAddresses | X |  |
| targetAddress | X |  |

## <a name="device-writeback"></a>Обратная запись устройств
Объекты устройств создаются в Active Directory. Это могут быть устройства, присоединенные к Azure AD, или компьютеры Windows 10, присоединенные к домену.

| Имя атрибута | Устройство | Комментировать |
| --- |:---:| --- |
| altSecurityIdentities |X | |
| displayName |X | |
| Различающееся имя |X | |
| msDS-CloudAnchor |X | |
| msDS-DeviceID |X | |
| msDS-DeviceObjectVersion |X | |
| msDS-DeviceOSType |X | |
| msDS-DeviceOSVersion |X | |
| msDS-DevicePhysicalIDs |X | |
| msDS-KeyCredentialLink |X |Только со схемой Windows Server 2016 AD |
| msDS-IsCompliant |X | |
| msDS-IsEnabled |X | |
| msDS-IsManaged |X | |
| msDS-RegisteredOwner |X | |

## <a name="notes"></a>Примечания
* Если используется альтернативный идентификатор, то локальный атрибут userPrincipalName синхронизируется с атрибутом Azure AD onPremisesUserPrincipalName. Атрибут "Альтернативный идентификатор", например почта, синхронизируется с атрибутом Azure AD userPrincipalName.
* В списках выше тип объекта **User** также относится к типу объекта **iNetOrgPerson**.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше о настройке [службы синхронизации Azure AD Connect](how-to-connect-sync-whatis.md) .

Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).