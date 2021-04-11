---
title: Руководство по массовому приглашению пользователей B2B для совместной работы в Azure Active Directory | Документация Майкрософт
description: Из этого руководства вы узнаете, как с помощью PowerShell и CSV-файла отправлять массовые приглашения для внешних гостевых пользователей службы совместной работы Azure AD B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: tutorial
ms.date: 02/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 19b4ea9f9f159efe974be54238843f5b9e3faaca
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106056697"
---
# <a name="tutorial-use-powershell-to-bulk-invite-azure-ad-b2b-collaboration-users"></a>Руководство по Массовое приглашение пользователей службы совместной работы Azure AD B2Bс помощью PowerShell

Если вы используете службу совместной работы Azure Active Directory (Azure AD) B2B для работы с внешними партнерами, вы можете одновременно пригласить несколько гостевых пользователей к вашей организации. Из этого руководства вы узнаете, как использовать PowerShell для отправки массовых приглашений внешним пользователям. В частности, вы выполните описанные ниже действия:

> [!div class="checklist"]
> * подготовите файл с разделителями-запятыми (CSV), который вмещает данные о пользователе;
> * запустите сценарий PowerShell для отправки приглашений;
> * убедитесь, что пользователи добавлены в каталог.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу. 

## <a name="prerequisites"></a>Предварительные требования

### <a name="install-the-latest-azureadpreview-module"></a>Установка последнего модуля AzureADPreview

Убедитесь, что вы установили последнюю версию модуля Azure AD PowerShell для Graph (AzureADPreview). 

Сначала проверьте, какие модули вы установили. Для этого откройте Windows PowerShell от имени пользователя (функция "Запуск от имени администратора") и выполните следующую команду.

```powershell
Get-Module -ListAvailable AzureAD*
```

На основе выходных данных выполните одно из следующих действий.

- Если результаты не отображаются, выполните следующую команду, чтобы установить модуль AzureADPreview:
  
   ```powershell
   Install-Module AzureADPreview
   ```

- Если в результатах отображается только модуль AzureAD, выполните следующие команды, чтобы установить модуль AzureADPreview: 

   ```powershell
   Uninstall-Module AzureAD
   Install-Module AzureADPreview
   ```

- Если в результатах отображается только модуль AzureADPreview, но вы получили сообщение о том, что у вас установлена более поздняя версия, выполните следующие команды для обновления модуля:

   ```powershell
   Uninstall-Module AzureADPreview
   Install-Module AzureADPreview
   ```

Вы можете получить запрос на установку модуля из ненадежного репозитория. Это происходит, если вы ранее не устанавливали репозиторий PSGallery в качестве доверенного. Нажмите клавишу **Y**, чтобы установить модуль.

### <a name="get-test-email-accounts"></a>Получение тестовых учетных записей электронной почты

Требуется две или больше тестовых учетных записей электронной почты, с которых можно отправлять приглашения. Учетные записи должны находиться за пределами вашей организации. Вы можете использовать любой тип учетной записи, включая учетные записи социальных сетей, такие как адреса gmail.com или outlook.com.

## <a name="prepare-the-csv-file"></a>Подготовка CSV-файла

В Microsoft Excel создайте CSV-файл со списком имен приглашенных пользователей и адресами электронной почты. Не забудьте включить заголовки столбцов **Name** и **InvitedUserEmailAddress**.

Например, создайте лист в следующем формате:

![Выходные данные PowerShell с запросом, ожидающим подтверждения пользователя](media/tutorial-bulk-invite/AddUsersExcel.png)

Сохраните файл как **C:\BulkInvite\Invitations.csv**. 

Если у вас нет Excel, CSV-файл можно создать в любом текстовом редакторе, таком как Блокнот. Каждое значение разделяется запятой, а каждая запись начинается с новой строки. 

## <a name="sign-in-to-your-tenant"></a>Вход в клиент

Выполните следующую команду для подключения к домену клиента.

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```

Например, `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

При появлении запроса введите свои учетные данные.

## <a name="send-bulk-invitations"></a>Отправка массовых приглашений

Чтобы отправить приглашения, выполните следующий сценарий PowerShell (где **c:\bulkinvite\invitations.csv** — это путь к CSV-файлу):

```powershell
$invitations = import-csv c:\bulkinvite\invitations.csv

$messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo

$messageInfo.customizedMessageBody = "Hello. You are invited to the Contoso organization."

foreach ($email in $invitations)
   {New-AzureADMSInvitation `
      -InvitedUserEmailAddress $email.InvitedUserEmailAddress `
      -InvitedUserDisplayName $email.Name `
      -InviteRedirectUrl https://myapps.microsoft.com `
      -InvitedUserMessageInfo $messageInfo `
      -SendInvitationMessage $true
   }
```

Этот сценарий отправляет приглашение на электронные адреса, указанные в файле invitations.csv. Вы должны увидеть результат, аналогичный приведенному ниже для каждого пользователя:

![Снимок экрана: выходные данные PowerShell с запросом, ожидающим подтверждения пользователя.](media/tutorial-bulk-invite/B2BBulkImport.png)

## <a name="verify-users-exist-in-the-directory"></a>Проверка существования пользователей в каталоге

Чтобы убедиться, что приглашенные пользователи добавлены в Azure AD, выполните следующую команду:

```powershell
 Get-AzureADUser -Filter "UserType eq 'Guest'"
```

Вы должны увидеть приглашенных пользователей в списке с именем участника-пользователя (UPN) в формате *emailaddress*#EXT#\@*domain*. Например *lstokes_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, где contoso.onmicrosoft.com — это организация, из которой отправляется приглашение.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если учетная запись тестового пользователя в каталоге больше не нужна, ее можно удалить. Выполните следующую команду для удаления учетной записи пользователя.

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```

Например: `Remove-AzureADUser -ObjectId "lstokes_fabrikam.com#EXT#@contoso.onmicrosoft.com"`

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого руководства вы отправили массовые приглашения гостевым пользователям за пределами своей организации. Затем узнайте, как работает процесс активации приглашения.

> [!div class="nextstepaction"]
> [Активация приглашения службы совместной работы Azure Active Directory B2B](redemption-experience.md)
