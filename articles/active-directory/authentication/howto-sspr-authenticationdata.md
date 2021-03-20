---
title: Предварительное заполнение контактной информации для самостоятельного сброса пароля — Azure Active Directory
description: Узнайте, как предварительно заполнить контактные данные пользователей Azure Active Directory самостоятельного сброса пароля (SSPR), чтобы они могли использовать эту функцию без завершения процесса регистрации.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 10/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ed93944c2a94370250fb75a679c1ac37a2a8418c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96861159"
---
# <a name="pre-populate-user-authentication-contact-information-for-azure-active-directory-self-service-password-reset-sspr"></a>Предварительное заполнение контактных данных для проверки подлинности пользователя для Azure Active Directory самостоятельного сброса пароля (SSPR)

Чтобы использовать Azure Active Directory (Azure AD) самостоятельный сброс пароля (SSPR), необходимо наличие контактной информации для проверки подлинности пользователя. В некоторых организациях пользователи регистрируют свои данные для проверки подлинности самостоятельно. Другие организации предпочитают синхронизировать данные проверки подлинности, которые уже существуют в домен Active Directory Services (AD DS). Эти синхронизированные данные становятся доступными для Azure AD и SSPR без вмешательства пользователя. Если пользователям необходимо изменить или сбросить пароль, они могут сделать это, даже если они ранее не зарегистрировали свои контактные данные.

Вы можете предварительно заполнить контактную информацию для проверки подлинности, если вы отвечаете следующим требованиям.

* Вы правильно отформатируете данные в локальном каталоге.
* Вы настроили [Azure AD Connect](../hybrid/how-to-connect-install-express.md) для клиента Azure AD.

Номера телефонов должны быть в формате *+ Каунтрикоде PhoneNumber*, например *+ 1 4251234567*.

> [!NOTE]
> Между кодом страны и номером телефона должно быть пробела.
>
> Функция сброса пароля не поддерживает добавочные номера. Даже добавочные номера в формате *+1 4251234567X12345* будут удаляться.

## <a name="fields-populated"></a>Заполненные поля

Если вы используете параметры по умолчанию в Azure AD Connect, то для заполнения контактной информации для проверки подлинности для SSPR будут выполнены следующие сопоставления:

| Локальная служба Active Directory | Azure AD     |
|------------------------------|--------------|
| TelephoneNumber              | Рабочий телефон |
| mobile                       | Мобильный телефон |

После того как пользователь проверит свой номер мобильного телефона, в поле *Телефон* в разделе **Контактная информация для проверки подлинности** в Azure AD также заносится этот номер.

## <a name="authentication-contact-info"></a>Контактная информация для проверки подлинности

На странице **методы проверки подлинности** для пользователя Azure AD в портал Azure глобальный администратор может вручную задать контактную информацию для проверки подлинности. Вы можете просмотреть существующие методы в разделе *использование методов проверки подлинности* или **добавить методы проверки подлинности**, как показано на следующем примере снимка экрана:

:::image type="content" source="media/howto-sspr-authenticationdata/user-authentication-contact-info.png" alt-text="Управление методами проверки подлинности из портал Azure":::

Следующие рекомендации относятся к контактным сведениям для проверки подлинности:

* Если поле " *Телефон* " заполнено и в политике SSPR включен *мобильный телефон* , он увидит это число на странице регистрации сброса пароля и в ходе рабочего процесса сброса пароля.
* Если поле *электронной почты* заполнено и в политике SSPR включена *Электронная почта* , пользователь увидит это сообщение на странице регистрации сброса пароля и в ходе рабочего процесса сброса пароля.

## <a name="security-questions-and-answers"></a>Контрольные вопросы и ответы на них

Контрольные вопросы и ответы на них надежно хранятся в клиенте Azure AD. Эти данные доступны пользователям только на [портале регистрации SSPR](https://aka.ms/ssprsetup). Администраторы не могут видеть, настраивать или изменять содержимое вопросов и ответов других пользователей.

## <a name="what-happens-when-a-user-registers"></a>Что происходит, когда пользователь проходит регистрацию?

Когда пользователь регистрируется, на странице регистрации будут заполнены следующие поля:

* **Телефон проверки подлинности**
* **Адрес электронной почты для проверки подлинности**
* **Контрольные вопросы и ответы на них**

Если вы указали значение для *мобильного телефона* или *альтернативного адреса электронной почты*, пользователи могут сразу же использовать эти значения для сброса паролей, даже если они не зарегистрированы для службы.

Пользователи также видят эти значения при первой регистрации, а также могут изменить их при необходимости. После успешной регистрации эти значения сохраняются в полях *Телефон для проверки* подлинности и *адрес электронной почты для проверки подлинности* соответственно.

## <a name="set-and-read-the-authentication-data-through-powershell"></a>Установка и чтение данных проверки подлинности с помощью PowerShell

С помощью PowerShell можно заполнить следующие поля:

* *Дополнительный адрес электронной почты*
* *Мобильный телефон*;
* *Рабочий телефон*.
    * Можно задать, только если синхронизация выполняется с локальным каталогом.

> [!IMPORTANT]
> Существует известная нехватка четности в функциях команд между PowerShell v1 и PowerShell v2. [Microsoft Graph REST API (бета-версия) для методов проверки подлинности](/graph/api/resources/authenticationmethods-overview) — это текущий инженерный фокус для обеспечения современного взаимодействия.

### <a name="use-powershell-version-1"></a>Использование PowerShell версии 1

Чтобы приступить к работе, [скачайте и установите модуль Azure AD PowerShell](/previous-versions/azure/jj151815(v=azure.100)#bkmk_installmodule). После установки выполните следующие действия, чтобы настроить каждое поле.

#### <a name="set-the-authentication-data-with-powershell-version-1"></a>Настройка данных проверки подлинности с помощью PowerShell версии 1

```PowerShell
Connect-MsolService

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com")
Set-MsolUser -UserPrincipalName user@domain.com -MobilePhone "+1 4251234567"
Set-MsolUser -UserPrincipalName user@domain.com -PhoneNumber "+1 4252345678"

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com") -MobilePhone "+1 4251234567" -PhoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-1"></a>Считывание данных проверки подлинности с помощью PowerShell версии 1

```PowerShell
Connect-MsolService

Get-MsolUser -UserPrincipalName user@domain.com | select AlternateEmailAddresses
Get-MsolUser -UserPrincipalName user@domain.com | select MobilePhone
Get-MsolUser -UserPrincipalName user@domain.com | select PhoneNumber

Get-MsolUser | select DisplayName,UserPrincipalName,AlternateEmailAddresses,MobilePhone,PhoneNumber | Format-Table
```

#### <a name="read-the-authentication-phone-and-authentication-email-options"></a>Считывание параметров "Телефон для проверки подлинности" и "Адрес электронной почты для проверки подлинности"

Для считывания **телефона для проверки подлинности** и **адреса электронной почты для проверки подлинности** при использовании PowerShell версии 1 выполните следующие команды:

```PowerShell
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select PhoneNumber
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select Email
```

### <a name="use-powershell-version-2"></a>Использование PowerShell версии 2

Чтобы приступить к работе, [скачайте и установите модуль PowerShell для Azure AD версии 2](/powershell/module/azuread/).

Чтобы быстро установить из последних версий PowerShell, поддерживающих `Install-Module` , выполните следующие команды. В первой строке проверяется, установлен ли модуль.

```PowerShell
Get-Module AzureADPreview
Install-Module AzureADPreview
Connect-AzureAD
```

После установки модуля выполните следующие действия, чтобы настроить каждое поле.

#### <a name="set-the-authentication-data-with-powershell-version-2"></a>Настройка данных проверки подлинности с помощью PowerShell версии 2

```PowerShell
Connect-AzureAD

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("email@domain.com")
Set-AzureADUser -ObjectId user@domain.com -Mobile "+1 4251234567"
Set-AzureADUser -ObjectId user@domain.com -TelephoneNumber "+1 4252345678"

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("emails@domain.com") -Mobile "+1 4251234567" -TelephoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-2"></a>Считывание данных проверки подлинности с помощью PowerShell версии 2

```PowerShell
Connect-AzureAD

Get-AzureADUser -ObjectID user@domain.com | select otherMails
Get-AzureADUser -ObjectID user@domain.com | select Mobile
Get-AzureADUser -ObjectID user@domain.com | select TelephoneNumber

Get-AzureADUser | select DisplayName,UserPrincipalName,otherMails,Mobile,TelephoneNumber | Format-Table
```

## <a name="next-steps"></a>Дальнейшие действия

После того как контактные данные для проверки подлинности предварительно заполнены для пользователей, следуйте указаниям в следующем руководстве по включению самостоятельного сброса пароля.

> [!div class="nextstepaction"]
> [Включение самостоятельного сброса пароля Azure AD](tutorial-enable-sspr.md)
