---
title: Краткое руководство. Добавление гостевого пользователя с помощью PowerShell (Azure AD)
description: В этом кратком руководстве вы узнаете, как использовать PowerShell для отправки приглашения внешнему пользователю службы совместной работы Azure AD B2B.
services: active-directory
author: msmimart
ms.author: mimart
manager: celestedg
ms.reviewer: mal
ms.date: 08/28/2018
ms.topic: quickstart
ms.service: active-directory
ms.subservice: B2B
ms.custom:
- it-pro
- seo-update-azuread-jan
- mode-api
ms.collection: M365-identity-device-management
ms.openlocfilehash: 765d8ea99167512e553cef9cc6f7fed583eff469
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529821"
---
# <a name="quickstart-add-a-guest-user-with-powershell"></a>Краткое руководство. Добавление гостевого пользователя с помощью PowerShell

Существует много способов пригласить внешних партнеров для использования приложений и служб с помощью службы совместной работы Azure Active Directory B2B. В предыдущем кратком руководстве вы узнали, как добавить гостевых пользователей непосредственно на портале администрирования Azure Active Directory. Можно также использовать PowerShell, чтобы добавить гостевых пользователей по одному или сразу несколько. В этом кратком руководстве вы используете команду New-AzureADMSInvitation, чтобы добавить одного гостевого пользователя в клиент Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу. 

## <a name="prerequisites"></a>Предварительные требования

### <a name="install-the-latest-azureadpreview-module"></a>Установка последнего модуля AzureADPreview
Убедитесь, что вы установили последнюю версию модуля Azure AD PowerShell для Graph (AzureADPreview). 

Сначала проверьте, какие модули вы установили. Для этого откройте Windows PowerShell от имени пользователя (функция "Запуск от имени администратора") и выполните следующую команду.
 
```powershell  
Get-Module -ListAvailable AzureAD*
```

Если модуль AzureADPreview отображается без сообщения о том, что имеется более поздняя версия, то все готово. В противном случае, на основе выходных данных выполните одно из следующих действий.

- Если результаты не отображаются, выполните следующую команду, чтобы установить модуль AzureADPreview.
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Если в результатах отображается только модуль AzureAD, выполните следующие команды, чтобы установить модуль AzureADPreview. 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Если в результатах отображается только модуль AzureADPreview, но вы получили сообщение о том, что существует более поздняя версия, выполните следующие команды для обновления модуля. 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
  ```

Вы можете получить запрос на установку модуля из ненадежного репозитория. Это происходит, если вы ранее не устанавливали репозиторий PSGallery в качестве доверенного. Нажмите клавишу **Y**, чтобы установить модуль.

### <a name="get-a-test-email-account"></a>Получение тестовой учетной записи электронной почты

Требуется тестовая учетная запись электронной почты, на которую можно отправить приглашение. Эта учетная запись должна находиться за пределами вашей организации. Вы можете использовать любой тип учетной записи, включая учетные записи социальных сетей, например адрес gmail.com или outlook.com.

## <a name="sign-in-to-your-tenant"></a>Вход в клиент

Выполните следующую команду для подключения к домену клиента.

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
Например, `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

При появлении запроса введите свои учетные данные.

## <a name="send-an-invitation"></a>Отправка приглашения

1. Чтобы отправить приглашение на тестовую учетную запись электронной почты, выполните приведенную ниже команду PowerShell (замените **Sanda** и **sanda\@fabrikam.com** именем и адресом электронной почты тестовой учетной записи). 

   ```powershell
   New-AzureADMSInvitation -InvitedUserDisplayName "Sanda" -InvitedUserEmailAddress sanda@fabrikam.com -InviteRedirectURL https://myapps.microsoft.com -SendInvitationMessage $true
   ```
2. Команда отправляет приглашение на указанный адрес электронной почты. Проверьте результат. Он должен выглядеть примерно следующим образом.

   ![Выходные данные PowerShell с запросом, ожидающим подтверждения пользователя](media/quickstart-invite-powershell/powershell-azureadmsinvitation-result.png)

## <a name="verify-the-user-exists-in-the-directory"></a>Проверка существования пользователя в каталоге

1. Чтобы убедиться, что приглашенный пользователь добавлен в Azure AD, выполните следующую команду.
 
   ```powershell
   Get-AzureADUser -Filter "UserType eq 'Guest'"
   ```
3. Просмотрите результат, чтобы убедиться, что приглашенный пользователь указан в списке с именем участника-пользователя (UPN) в формате *адрес_электронной_почты*#EXT#\@*домен*. Например, *sanda_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, где contoso.onmicrosoft.com — это организация, из которой отправляется приглашение.

   ![Выходные данные PowerShell с добавленным гостевым пользователем](media/quickstart-invite-powershell/powershell-guest-user-added.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если тестовая учетная запись пользователя в каталоге больше не нужна, ее можно удалить. Выполните следующую команду для удаления учетной записи пользователя.

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
Например: `Remove-AzureADUser -ObjectId "sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>Дальнейшие действия
В этом кратком руководстве вы пригласили одного гостевого пользователя и добавили его в каталог с помощью PowerShell. Далее узнайте, как массово пригласить гостевых пользователей с помощью PowerShell.

> [!div class="nextstepaction"]
> [Руководство по Массовое приглашение пользователей службы совместной работы Azure Active Directory B2B](tutorial-bulk-invite.md)
