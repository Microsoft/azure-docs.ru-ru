---
title: Microsoft 365 внешний общий доступ и совместная работа B2B — Azure AD
description: Обсуждение совместного использования ресурсов с внешними партнерами с помощью Microsoft 365 и Azure Active Directory службы совместной работы B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: d003008ea5b0d2591574f6f488b0145ee6f08a5e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100008134"
---
# <a name="microsoft-365-external-sharing-and-azure-active-directory-azure-ad-b2b-collaboration"></a>Microsoft 365 внешний общий доступ и Azure Active Directory (Azure AD) B2B для совместной работы

В службе совместной работы Azure AD B2B и Microsoft 365 внешнем совместном доступе (OneDrive, SharePoint Online, единые группы и т. д.) проверка подлинности внешних пользователей осуществляется с помощью Azure AD B2B.

## <a name="how-does-azure-ad-b2b-differ-from-external-sharing-in-sharepoint-online"></a>Чем использование Azure AD B2B отличается от внешнего общего доступа в SharePoint Online?

В OneDrive и SharePoint Online используется другой диспетчер приглашений. Поддержка внешнего общего доступа в OneDrive и SharePoint Online была реализована до того, как ее внедрили в Azure AD. Со временем функция внешнего общего доступа OneDrive и SharePoint Online обросла дополнительными компонентами, и миллионы пользователей используют встроенную модель общего доступа этого продукта. При этом существуют незначительные различия в принципах работы внешнего общего доступа OneDrive и SharePoint Online и службы совместной работы Azure AD B2B. См. дополнительные сведения о [внешнем доступе OneDrive и SharePoint Online](/sharepoint/external-sharing-overview). Процесс отличается от использования Azure AD B2B следующим образом:

- OneDrive и SharePoint Online добавляют пользователей в каталог после того, как пользователи активировали приглашения. Таким образом, до активации вы не видите пользователя на портале Azure AD. В то же время если пользователь приглашается из другого сайта, то создается новое приглашение. При использовании службы совместной работы Azure AD B2B пользователь добавляется непосредственно после отправки приглашения, поэтому он сразу везде отображается.

- Процесс активации приглашения в OneDrive и SharePoint Online отличается от аналогичной процедуры в службе совместной работы Azure AD B2B. После того как пользователь активирует приглашение, интерфейсы будут выглядеть одинаково.

- Пользователей, приглашенных в службу совместной работы Azure AD B2B, можно выбирать в диалоговых окнах общего доступа OneDrive и SharePoint Online. Приглашенные пользователи OneDrive и SharePoint Online также отображаются в Azure AD после того, как они активируют свои приглашения.

- Требования к лицензированию отличаются. Дополнительные сведения о лицензировании см. в статье [Лицензирование внешних удостоверений Azure AD](./external-identities-pricing.md) и [Обзор внешнего общего доступа к SharePoint Online](/sharepoint/external-sharing-overview).
Чтобы управлять внешним общим доступом в OneDrive и SharePoint Online с помощью службы совместной работы Azure AD B2B, задайте для параметра внешнего общего доступа OneDrive и SharePoint Online значение **Allow sharing only with the external users that already exist in your organization's directory** (Разрешить общий доступ только внешним пользователям, которые уже существуют в каталоге организации). Пользователи могут перейти на внешние сайты, предоставляющие общий доступ, и выбрать из внешних сотрудников, добавленных администратором. Администратор может добавлять внешних сотрудников с помощью интерфейсов API приглашения службы совместной работы B2B.


![Параметр внешнего общего доступа OneDrive или SharePoint](media/o365-external-user/odsp-sharing-setting.png)

Когда вы включите предоставление общего доступа для внешних пользователей, возможность поиска существующих гостевых пользователей в средстве выбора пользователей SharePoint Online (SPO) отключена по умолчанию для согласования с поведением предыдущих версий.

Вы можете включить эту функцию, используя параметр ShowPeoplePickerSuggestionsForGuestUsers на уровне семейства веб-сайтов и коллекции клиентов. Задать параметр этой функции можно с помощью командлетов SPOTenant и SPOSite, которые позволяют участникам искать всех существующих гостевых пользователей в каталоге. Изменения в области клиента не влияют на уже подготовленные сайты SPO.

## <a name="next-steps"></a>Дальнейшие действия

* [Что такое служба совместной работы Azure AD B2B?](what-is-b2b.md)
* [Добавление пользователя службы совместной работы Azure Active Directory B2B в роль](add-guest-to-role.md)
* [Делегирование приглашений для службы совместной работы Azure Active Directory B2B](delegate-invitations.md)
* [Динамические группы и служба совместной работы Azure Active Directory B2B](use-dynamic-groups.md)
* [Устранение неполадок службы совместной работы Azure Active Directory B2B](troubleshoot.md)
