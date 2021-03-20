---
title: API совместной работы B2B и настройка — Azure Active Directory
description: Служба совместной работы Azure Active Directory B2B поддерживает взаимодействие между компаниями, позволяя предоставлять бизнес-партнерам выборочный доступ к вашим корпоративным приложениям.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/03/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8160859bb782ee8ffc4fef5ee03b61b6f54be1bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99548667"
---
# <a name="azure-active-directory-b2b-collaboration-api-and-customization"></a>API службы совместной работы Azure Active Directory B2B и настройка

Очень многие клиенты высказали пожелание, чтобы процесс приглашения был более индивидуальным и соответствовал потребностям конкретной организации. С помощью нашего интерфейса API это можно сделать. [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](/graph/api/resources/invitation)

## <a name="capabilities-of-the-invitation-api"></a>Возможности API приглашения

Интерфейс API предоставляет следующие возможности:

1. Пригласите внешнего пользователя, используя *любой* адрес электронной почты.

    ```
    "invitedUserDisplayName": "Sam"
    "invitedUserEmailAddress": "gsamoogle@gmail.com"
    ```

2. Настройте, куда будет перенаправляться пользователь после принятия приглашения.

    ```
    "inviteRedirectUrl": "https://myapps.microsoft.com/"
    ```

3. Выберите отправку стандартного приглашения через нас

    ```
    "sendInvitationMessage": true
    ```

   с сообщением для получателя, которое можно настроить.

    ```
    "customizedMessageBody": "Hello Sam, let's collaborate!"
    ```

4. Выберите также, кому отправить копию. Это могут быть пользователи, которых вы хотите известить о приглашении данного сотрудника.

5. Или полностью настройте свой вариант приглашения и рабочего процесса регистрации сотрудников. При этом уведомления через Azure AD не будут отсылаться.

    ```
    "sendInvitationMessage": false
    ```

   В этом случае вы получите URL-адрес активации из API, который можно вставить в шаблон электронного сообщения, отправить в мгновенном сообщении или распространить другим способом на свое усмотрение.

6. Наконец, если вы являетесь администратором, то вы можете пригласить пользователя в качестве участника.

    ```
    "invitedUserType": "Member"
    ```

## <a name="determine-if-a-user-was-already-invited-to-your-directory"></a>Определение того, был ли пользователь приглашен к вашему каталогу

Вы можете использовать API приглашения, чтобы определить, существует ли пользователь в клиенте ресурсов. Это может быть полезно при разработке приложения, которое использует API приглашения для приглашения пользователя. Если пользователь уже существует в каталоге ресурсов, он не получит приглашения, поэтому можно сначала выполнить запрос, чтобы определить, существует ли уже сообщение электронной почты в качестве имени участника-пользователя или другого свойства входа.

1. Убедитесь, что домен электронной почты пользователя не входит в проверенный домен клиента ресурса.
2. В клиенте ресурса используйте следующий запрос на получение пользователя, где {0} — это адрес электронной почты, который вы приглашаете:

   ```
   “userPrincipalName eq '{0}' or mail eq '{0}' or proxyAddresses/any(x:x eq 'SMTP:{0}') or signInNames/any(x:x eq '{0}') or otherMails/any(x:x eq '{0}')"
   ```

## <a name="authorization-model"></a>Модель авторизации

API можно запустить в следующих режимах авторизации:

### <a name="app--user-mode"></a>Приложение + пользователь

В этом режиме приложение или пользователь, использующий API, должен иметь разрешения на создание приглашений в службу B2B.

### <a name="app-only-mode"></a>Только приложение

В контексте "только приложение" для успешного выполнения приглашения приложению требуется область User.Invite.All.

Дополнительные сведения см. в этой статье: https://developer.microsoft.com/graph/docs/authorization/permission_scopes.


## <a name="powershell"></a>PowerShell

Вы можете добавлять и приглашать в организацию внешних пользователей, теперь можно использовать PowerShell. Создайте новое приглашение, используя приведенный ниже командлет.

```powershell
New-AzureADMSInvitation
```

Можно использовать следующие параметры.

* -InvitedUserDisplayName
* -InvitedUserEmailAddress
* -SendInvitationMessage
* -InvitedUserMessageInfo

### <a name="invitation-status"></a>Статус приглашения

После отправки приглашения, внешний пользователь может использовать командлет **Get-AzureADUser**, чтобы убедится, принят ли он. Следующие свойства Get-AzureADUser заполняются в том случае, когда внешнему пользователю отправляется приглашение:

* **UserState** указывает, является ли приглашение **PendingAcceptance** или **принятым**.
* **UserStateChangedOn** показывает метку времени для последних изменений в свойстве **UserState**.

Можно использовать параметр **Filter** (Фильтр), чтобы отфильтровать результаты по **UserState**. В приведенном ниже примере показано, как фильтровать результаты, чтобы показывать только тех пользователей, у которых есть ожидающее приглашения. В примере также показан параметр **Format-List**, который позволяет указывать свойства для отображения. 
 

```powershell
Get-AzureADUser -Filter "UserState eq 'PendingAcceptance'" | Format-List -Property DisplayName,UserPrincipalName,UserState,UserStateChangedOn
```

> [!NOTE]
> Убедитесь, что у вас установлена последняя версия модуля AzureAD PowerShell или AzureADPreview PowerShell. 

## <a name="see-also"></a>См. также раздел

Ознакомьтесь с справочником по API приглашения в [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](/graph/api/resources/invitation) .

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое служба совместной работы Azure AD B2B?](what-is-b2b.md)
- [Элементы сообщения с приглашением в службу совместной работы B2B](invitation-email-elements.md)
- [Активация приглашения службы совместной работы B2B](redemption-experience.md)
- [Добавление пользователей службы совместной работы B2B без приглашения](add-user-without-invite.md)