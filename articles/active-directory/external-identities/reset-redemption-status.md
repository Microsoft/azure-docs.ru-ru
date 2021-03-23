---
title: Сброс состояния активации гостевого пользователя в Azure AD
description: Узнайте, как сбросить состояние активации приглашения для гостевых пользователей Azure Active Directory B2B в внешних удостоверениях Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/03/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2998c3ea0d65bd3c96bd1ac5bdfa8ff148c6c4cc
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104780435"
---
# <a name="reset-redemption-status-for-a-guest-user"></a>Сброс состояния активации для гостевого пользователя

После того как гостевой пользователь запросит ваше приглашение к службе совместной работы B2B, в некоторых случаях потребуется обновить данные для входа, например:

- Пользователь хочет войти в систему, используя другой адрес электронной почты и поставщик удостоверений
- Учетная запись пользователя в своем домашнем клиенте удалена и создана повторно
- Пользователь переместился в другую компанию, но ему по-прежнему нужен тот же доступ к вашим ресурсам.
- Обязанности пользователя переданы другому пользователю

Чтобы управлять этими сценариями ранее, необходимо вручную удалить учетную запись гостевого пользователя из каталога и повторно пригласить пользователя. Теперь вы можете использовать PowerShell или API приглашения Microsoft Graph, чтобы сбросить состояние активации пользователя и повторно пригласить пользователя, сохраняя при этом идентификатор объекта пользователя, членство в группах и назначения приложений. Когда пользователь активирует новое приглашение, UPN пользователя не изменяется, но имя входа пользователя меняется на новое сообщение. Затем пользователь может войти в систему с помощью нового электронного письма или электронного письма, добавленного в `otherMails` свойство объекта пользователя.

## <a name="reset-the-email-address-used-for-sign-in"></a>Сброс адреса электронной почты, используемого для входа

Если пользователь хочет выполнить вход, используя другой адрес электронной почты:

1. Убедитесь, что новый адрес электронной почты добавлен в `mail` свойство или `otherMails` объекта пользователя. 
2.  Замените адрес электронной почты в `InvitedUserEmailAddress` свойстве на новый адрес электронной почты.
3. Используйте один из следующих методов, чтобы сбросить состояние активации пользователя.

> [!NOTE]
>Во время общедоступной предварительной версии при сбросе адреса электронной почты пользователя рекомендуется присвоить `mail` свойству новый адрес электронной почты. Таким образом пользователь может активировать приглашение, войдя в каталог в дополнение к использованию ссылки активации в приглашении.
>
## <a name="use-powershell-to-reset-redemption-status"></a>Сброс состояния активации с помощью PowerShell

Установите последнюю версию модуля PowerShell AzureADPreview и создайте новое приглашение, указав `InvitedUserEmailAddress` Новый адрес электронной почты, и `ResetRedemption` Задайте для него значение `true` .

```powershell  
Uninstall-Module AzureADPreview 
Install-Module AzureADPreview 
Connect-AzureAD 
$ADGraphUser = Get-AzureADUser -objectID "UPN of User to Reset"  
$msGraphUser = New-Object Microsoft.Open.MSGraph.Model.User -ArgumentList $ADGraphUser.ObjectId 
New-AzureADMSInvitation -InvitedUserEmailAddress <<external email>> -SendInvitationMessage $True -InviteRedirectUrl "http://myapps.microsoft.com" -InvitedUser $msGraphUser -ResetRedemption $True 
```

## <a name="use-microsoft-graph-api-to-reset-redemption-status"></a>Использование API Microsoft Graph для сброса состояния активации

Используя [API приглашения Microsoft Graph](/graph/api/resources/invitation?view=graph-rest-1.0), задайте `resetRedemption` для свойства значение `true` и укажите новый адрес электронной почты в `invitedUserEmailAddress` свойстве.

```json
POST https://graph.microsoft.com/beta/invitations  
Authorization: Bearer eyJ0eX...  
ContentType: application/json  
{  
   "invitedUserEmailAddress": "<<external email>>",  
   "sendInvitationMessage": true,  
   "invitedUserMessageInfo": {  
      "messageLanguage": "en-US",  
      "ccRecipients": [  
         {  
            "emailAddress": {  
               "name": null,  
               "address": "<<optional additional notification email>>"  
            }  
         } 
      ],  
      "customizedMessageBody": "<<custom message>>"  
},  
"inviteRedirectUrl": "https://myapps.microsoft.com?tenantId=",  
"invitedUser": {  
   "id": "<<ID for the user you want to reset>>"  
}, 
"resetRedemption": true 
}
```

## <a name="next-steps"></a>Следующие шаги

- [Добавление пользователей службы совместной работы Azure Active Directory B2B с помощью PowerShell](customize-invitation-api.md#powershell)
- [Свойства гостевого пользователя Azure AD B2B](user-properties.md)
