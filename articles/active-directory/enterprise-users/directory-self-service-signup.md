---
title: Самостоятельная регистрация проверенных с помощью электронной почты пользователей — Azure AD | Документация Майкрософт
description: Использование самостоятельной регистрации в организации Azure Active Directory (Azure AD).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: overview
ms.workload: identity
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 95625886ed11256a40e5993540d7e545134d6dd6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96860870"
---
# <a name="what-is-self-service-sign-up-for-azure-active-directory"></a>Что такое самостоятельная регистрация для Azure Active Directory?

В этой статье объясняется, как использовать самостоятельную регистрацию для заполнения организации в Azure Active Directory (Azure AD). Если вам нужно сменить доменное имя из неуправляемой организации Azure AD, см. статью [Смена неуправляемого каталога от имени администратора в Azure Active Directory](domains-admin-takeover.md).

## <a name="why-use-self-service-sign-up"></a>Зачем нужна самостоятельная регистрация?

* Предоставить клиентам нужные службы как можно быстрее.
* Создать предложения на основе электронной почты для службы.
* Создать потоки регистрации на основе электронной почты, которые позволяют пользователям быстро создавать удостоверения с помощью легко запоминаемых псевдонимов рабочих электронных адресов.
* Самостоятельно созданный каталог Azure AD можно преобразовать в управляемый каталог, который может использоваться для других служб.

## <a name="terms-and-definitions"></a>Термины и определения

* **Самостоятельная регистрация** метод, с помощью которого пользователь регистрируется в облачной службе, после чего для него в Azure Active Directory на основании домена электронной почты автоматически создается удостоверение.
* **Неуправляемый каталог Azure Active Directory**: каталог, где создается это удостоверение. Неуправляемый каталог — это каталог, в котором нет глобального администратора.
* **Проверяемый с помощью электронной почты пользователь**: тип учетной записи пользователя в Azure Active Directory. Пользователь, который получает автоматически созданное после регистрации для предложения самообслуживания удостоверение, называется проверяемым с помощью электронной почты пользователем. Проверяемый с помощью электронной почты пользователь является регулярным участником каталога с тегом creationmethod=EmailVerified.

## <a name="how-do-i-control-self-service-settings"></a>Как контролировать параметры самообслуживания?

На сегодняшний день у администраторов есть два способа самообслуживаемого управления. Они могут управлять:

* возможностью присоединения пользователей к каталогу с помощью электронной почты;
* лицензированием пользователями самих себя для приложений и служб.

### <a name="how-can-i-control-these-capabilities"></a>Как контролировать эти возможности?

Администратор может настраивать эти возможности с помощью следующих параметров командлета Azure AD Set-MsolCompanySettings:

* **AllowEmailVerifiedUsers** отвечает за то, может ли пользователь создать или присоединить каталог. Если задать для этого параметра значение $false, учетная запись пользователя, проверяемая с помощью электронной почты, не сможет присоединиться к каталогу.
* **AllowAdHocSubscriptions** управляет самостоятельной регистрацией. Если задать для этого параметра значение $false, пользователи не смогут выполнять самостоятельную регистрацию.
  
Параметры AllowEmailVerifiedUsers и AllowAdHocSubscriptions применяются на уровне каталога для управляемого или неуправляемого каталога. Ниже приведен пример.

* Вы администрируете каталог с проверенным домен, например contoso.com.
* Вы используете службу совместной работы B2B из другого каталога, чтобы пригласить пользователя, который еще не существует (userdoesnotexist@contoso.com) в домашнем каталоге constoso.com.
* Для домашнего каталога включен параметр AllowEmailVerifiedUsers.

Если эти условия выполнены, то в домашнем каталоге создается пользователь-участник, а в приглашающем каталоге создается гостевой пользователь B2B.

Дополнительные сведения о пробных подписках на Flow и PowerApps см. в следующих статьях:

* [Как запретить имеющимся пользователям начать работу с Power BI?](https://support.office.com/article/Power-BI-in-your-Organization-d7941332-8aec-4e5e-87e8-92073ce73dc5#bkmk_preventjoining)
* [Вопросы и ответы об использовании Flow в организации](/flow/organization-q-and-a)

### <a name="how-do-the-controls-work-together"></a>Как совместно использовать эти элементы управления?
Эти два параметра можно использовать совместно для избирательного управления самостоятельной регистрацией. Например, следующая команда позволит пользователям выполнять самостоятельную регистрацию, но только если у этих пользователей уже есть учетная запись в Azure AD (иными словами, пользователи, которым нужно создать проверенную с помощью электронной почты учетную запись, не смогут выполнить самостоятельную регистрацию).

```powershell
    Set-MsolCompanySettings -AllowEmailVerifiedUsers $false -AllowAdHocSubscriptions $true
```

На блок-схеме ниже показаны разные сочетания этих параметров и результирующие условия для каталога и самостоятельной регистрации.

![Блок-схема элементов управления самостоятельной регистрации](./media/directory-self-service-signup/SelfServiceSignUpControls.png)

Сведения об этом параметре можно получить с помощью командлета Get-MsolCompanyInformation в PowerShell. Дополнительные сведения об этом см. в статье [Get-MsolCompanyInformation](/powershell/module/msonline/get-msolcompanyinformation).

```powershell
    Get-MsolCompanyInformation | Select AllowEmailVerifiedUsers, AllowAdHocSubscriptions
```

Дополнительные сведения об этих параметрах и примеры их использования см. в статье [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings).

## <a name="next-steps"></a>Дальнейшие действия

* [Добавление имени личного домена в Azure Active Directory](../fundamentals/add-custom-domain.md)
* [Как установить и настроить Azure PowerShell](/powershell/azure/)
* [Azure PowerShell](/powershell/azure/)
* [Справка по командлетам Azure](/powershell/azure/get-started-azureps)
* [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings)
* [Закрытие рабочей или учебной учетной записи в неуправляемом каталоге](users-close-account.md)
