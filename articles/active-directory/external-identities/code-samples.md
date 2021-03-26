---
title: Примеры кода и команд PowerShell для службы "Совместная работа B2B" —Azure AD
description: Примеры кода и команд PowerShell для службы совместной работы Azure Active Directory B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: sample
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.custom: it-pro, seo-update-azuread-jan, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: fcf6e7a4fb5e76dddba6162bbabfdc5abc806a20
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87907047"
---
# <a name="azure-active-directory-b2b-collaboration-code-and-powershell-samples"></a>Примеры кода и команд PowerShell для службы совместной работы Azure Active Directory B2B

## <a name="powershell-example"></a>Пример PowerShell
Можно выполнить массовое приглашение внешних пользователей в организацию, используя электронные адреса, сохраненные в CSV-файле.

1. Подготовьте CSV-файл. Создайте CSV-файл и назовите его invitations.csv. В этом примере файл сохранен в папке C:\data. Он содержит следующие сведения:

   Имя                  |  InvitedUserEmailAddress
   --------------------- | --------------------------
   Приглашенный Gmail B2B     | b2binvitee@gmail.com
   Приглашенный B2B Outlook   | b2binvitee@outlook.com


2. Установите последнюю версию Azure AD PowerShell. Для использования новых командлетов необходимо установить обновленный модуль Azure AD PowerShell, который можно скачать на [странице выпуска модулей Powershell](https://www.powershellgallery.com/packages/AzureADPreview).

3. Войдите в свой клиент:

    ```powershell
    $cred = Get-Credential
    Connect-AzureAD -Credential $cred
    ```

4. Выполнение командлета PowerShell

   ```powershell
   $invitations = import-csv C:\data\invitations.csv
   $messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo
   $messageInfo.customizedMessageBody = "Hey there! Check this out. I created an invitation through PowerShell"
   foreach ($email in $invitations) {New-AzureADMSInvitation -InvitedUserEmailAddress $email.InvitedUserEmailAddress -InvitedUserDisplayName $email.Name -InviteRedirectUrl https://wingtiptoysonline-dev-ed.my.salesforce.com -InvitedUserMessageInfo $messageInfo -SendInvitationMessage $true}
   ```

Этот командлет отправляет приглашение на электронные адреса, указанные в файле invitations.csv. Дополнительные возможности этого командлета:
- настраиваемый текст электронного сообщения;
- добавление отображаемого имени для приглашенного пользователя;
- отправка копий сообщений или полное подавление электронных сообщений.

## <a name="code-sample"></a>Пример кода
Ниже показано, как вызвать API приглашения в режиме "только приложение" и получить URL-адрес активации для ресурса, в который вы приглашаете пользователя службы B2B. Цель этого действия — отправить настраиваемое сообщение с приглашением. Электронное сообщение можно составить, используя HTTP-клиент, чтобы настроить его внешний вид, и отправить его с помощью API Microsoft Graph.

```csharp
namespace SampleInviteApp
{
    using System;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    class Program
    {
        /// <summary>
        /// Microsoft Graph resource.
        /// </summary>
        static readonly string GraphResource = "https://graph.microsoft.com";

        /// <summary>
        /// Microsoft Graph invite endpoint.
        /// </summary>
        static readonly string InviteEndPoint = "https://graph.microsoft.com/v1.0/invitations";

        /// <summary>
        ///  Authentication endpoint to get token.
        /// </summary>
        static readonly string EstsLoginEndpoint = "https://login.microsoftonline.com";

        /// <summary>
        /// This is the tenantid of the tenant you want to invite users to.
        /// </summary>
        private static readonly string TenantID = "";

        /// <summary>
        /// This is the application id of the application that is registered in the above tenant.
        /// The required scopes are available in the below link.
        /// https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/invitation_post
        /// </summary>
        private static readonly string TestAppClientId = "";

        /// <summary>
        /// Client secret of the application.
        /// </summary>
        private static readonly string TestAppClientSecret = @"";

        /// <summary>
        /// This is the email address of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserEmailAddress = @"";

        /// <summary>
        /// This is the display name of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserDisplayName = @"";

        /// <summary>
        /// Main method.
        /// </summary>
        /// <param name="args">Optional arguments</param>
        static void Main(string[] args)
        {
            Invitation invitation = CreateInvitation();
            SendInvitation(invitation);
        }

        /// <summary>
        /// Create the invitation object.
        /// </summary>
        /// <returns>Returns the invitation object.</returns>
        private static Invitation CreateInvitation()
        {
            // Set the invitation object.
            Invitation invitation = new Invitation();
            invitation.InvitedUserDisplayName = InvitedUserDisplayName;
            invitation.InvitedUserEmailAddress = InvitedUserEmailAddress;
            invitation.InviteRedirectUrl = "https://www.microsoft.com";
            invitation.SendInvitationMessage = true;
            return invitation;
        }

        /// <summary>
        /// Send the guest user invite request.
        /// </summary>
        /// <param name="invitation">Invitation object.</param>
        private static void SendInvitation(Invitation invitation)
        {
            string accessToken = GetAccessToken();

            HttpClient httpClient = GetHttpClient(accessToken);

            // Make the invite call.
            HttpContent content = new StringContent(JsonConvert.SerializeObject(invitation));
            content.Headers.Add("ContentType", "application/json");
            var postResponse = httpClient.PostAsync(InviteEndPoint, content).Result;
            string serverResponse = postResponse.Content.ReadAsStringAsync().Result;
            Console.WriteLine(serverResponse);
        }

        /// <summary>
        /// Get the HTTP client.
        /// </summary>
        /// <param name="accessToken">Access token</param>
        /// <returns>Returns the Http Client.</returns>
        private static HttpClient GetHttpClient(string accessToken)
        {
            // setup http client.
            HttpClient httpClient = new HttpClient();
            httpClient.Timeout = TimeSpan.FromSeconds(300);
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
            httpClient.DefaultRequestHeaders.Add("client-request-id", Guid.NewGuid().ToString());
            Console.WriteLine(
                "CorrelationID for the request: {0}",
                httpClient.DefaultRequestHeaders.GetValues("client-request-id").Single());
            return httpClient;
        }

        /// <summary>
        /// Get the access token for our application to talk to Microsoft Graph.
        /// </summary>
        /// <returns>Returns the access token for our application to talk to Microsoft Graph.</returns>
        private static string GetAccessToken()
        {
            string accessToken = null;

            // Get the access token for our application to talk to Microsoft Graph.
            try
            {
                AuthenticationContext testAuthContext =
                    new AuthenticationContext(string.Format("{0}/{1}", EstsLoginEndpoint, TenantID));
                AuthenticationResult testAuthResult = testAuthContext.AcquireTokenAsync(
                    GraphResource,
                    new ClientCredential(TestAppClientId, TestAppClientSecret)).Result;
                accessToken = testAuthResult.AccessToken;
            }
            catch (AdalException ex)
            {
                Console.WriteLine("An exception was thrown while fetching the token: {0}.", ex);
                throw;
            }

            return accessToken;
        }

        /// <summary>
        /// Invitation class.
        /// </summary>
        public class Invitation
        {
            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserDisplayName { get; set; }

            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserEmailAddress { get; set; }

            /// <summary>
            /// Gets or sets a value indicating whether Invitation Manager should send the email to InvitedUser.
            /// </summary>
            public bool SendInvitationMessage { get; set; }

            /// <summary>
            /// Gets or sets invitation redirect URL
            /// </summary>
            public string InviteRedirectUrl { get; set; }
        }
    }
}
```


## <a name="next-steps"></a>Дальнейшие действия

- [Что такое служба совместной работы Azure AD B2B?](what-is-b2b.md)
