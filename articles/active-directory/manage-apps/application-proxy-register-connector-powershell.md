---
title: Автоматическая установка соединителя прокси приложения Azure AD | Документация Майкрософт
description: Описывается, как выполнить автоматическую установку соединителя прокси приложения Azure AD для обеспечения безопасного удаленного доступа к локальным приложениям.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/24/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0ac442e895850de04bedf7673ffe267ac8697d26
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258156"
---
# <a name="create-an-unattended-installation-script-for-the-azure-ad-application-proxy-connector"></a>Создание скрипта автоматической установки для соединителя прокси приложения Azure AD

Эта статья поможет вам создать скрипт Windows PowerShell, позволяющий выполнить автоматическую установку и регистрацию соединителя прокси приложения Azure AD.

Эта возможность полезна в тех случаях, когда требуется:

* установить соединитель на серверах Windows, на которых отключен пользовательский интерфейс или к которым не удается получить доступ через удаленный рабочий стол;
* установить и зарегистрировать много соединителей одновременно;
* интегрировать установку и регистрацию соединителя в другую процедуру;
* создать стандартный образ сервера, который содержит биты соединителя, но не зарегистрирован.

Чтобы [соединитель прокси приложения](application-proxy-connectors.md) работал, он должен быть зарегистрирован в каталоге Azure AD с помощью администратора приложения и пароля. Обычно эти сведения вводятся во всплывающем окне во время установки соединителя, но для автоматизации этого процесса можно использовать PowerShell.

Этот процесс состоит из двух этапов. Сначала выполняется установка соединителя, а затем он регистрируется в Azure AD.

> [!IMPORTANT]
> Если вы устанавливаете соединитель для облака Azure для государственных организаций, проверьте [Предварительные требования](../hybrid/reference-connect-government-cloud.md#allow-access-to-urls) и [шаги по установке](../hybrid/reference-connect-government-cloud.md#install-the-agent-for-the-azure-government-cloud). Для этого необходимо включить доступ к другому набору URL-адресов и дополнительный параметр для запуска установки.

## <a name="install-the-connector"></a>Установка соединителя
Чтобы установить соединитель без регистрации, сделайте следующее:

1. Откройте командную строку.
2. Выполните следующую команду, в которой параметр /q означает, что установка осуществляется в автоматическом режиме и вам не нужно принимать лицензионное соглашение.

   ```
   AADApplicationProxyConnectorInstaller.exe REGISTERCONNECTOR="false" /q
   ```

## <a name="register-the-connector-with-azure-ad"></a>Регистрация соединителя в Azure AD
Существуют два метода регистрации соединителя:

* Регистрация соединителя с помощью объекта учетных данных Windows PowerShell
* Регистрация соединителя с помощью маркера, созданного в автономном режиме

### <a name="register-the-connector-using-a-windows-powershell-credential-object"></a>Регистрация соединителя с помощью объекта учетных данных Windows PowerShell
1. Создайте объект учетных данных Windows PowerShell `$cred`, содержащий имя пользователя и пароль администратора для вашего каталога. Выполните следующую команду, заменив *\<username\>* и *\<password\>* :

   ```powershell
   $User = "<username>"
   $PlainPassword = '<password>'
   $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
   $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $User, $SecurePassword
   ```
2. Перейдите в каталог **C:\Program Files\Microsoft AAD App Proxy Connector** и запустите следующий скрипт с использованием созданного объекта `$cred`.

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature ApplicationProxy
   ```

### <a name="register-the-connector-using-a-token-created-offline"></a>Регистрация соединителя с помощью маркера, созданного в автономном режиме
1. Создайте автономный маркер с помощью класса AuthenticationContext, используя значения, указанные во фрагменте кода или командлете PowerShell ниже:

   **Использование C#:**

   ```csharp
   using System;
   using System.Linq;
   using System.Collections.Generic;
   using Microsoft.Identity.Client;

   class Program
   {
   #region constants
   /// <summary>
   /// The AAD authentication endpoint uri
   /// </summary>
   static readonly string AadAuthenticationEndpoint = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize";

   /// <summary>
   /// The application ID of the connector in AAD
   /// </summary>
   static readonly string ConnectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";
 
   /// <summary>
   /// The AppIdUri of the registration service in AAD
   /// </summary>
   static readonly string RegistrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation";

   #endregion

   #region private members
   private string token;
   private string tenantID;
   #endregion

   public void GetAuthenticationToken()
   {
    
   IPublicClientApplication clientApp = PublicClientApplicationBuilder
      .Create(ConnectorAppId)
      .WithDefaultRedirectUri() // will automatically use the default Uri for native app
      .WithAuthority(AadAuthenticationEndpoint)
      .Build();

      AuthenticationResult authResult = null;
            
      IAccount account = null;

      IEnumerable<string> scopes = new string[] { RegistrationServiceAppIdUri };

      try
      {
       authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
      }
       catch (MsalUiRequiredException ex)
      {
       authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();
      }


      if (authResult == null || string.IsNullOrEmpty(authResult.AccessToken) || string.IsNullOrEmpty(authResult.TenantId))
      {
       Trace.TraceError("Authentication result, token or tenant id returned are null");
       throw new InvalidOperationException("Authentication result, token or tenant id returned are null");
      }

      token = authResult.AccessToken;
      tenantID = authResult.TenantId;
      }
      ```

   **С помощью PowerShell:**

   ```powershell
   # Load MSAL (Tested with version 4.7.1) 

   Add-Type -Path "..\MSAL\Microsoft.Identity.Client.dll" 
        
   # The AAD authentication endpoint uri
        
   $authority = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize"

   #The application ID of the connector in AAD

   $connectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";

   #The AppIdUri of the registration service in AAD
   $registrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation"

   # Define the resources and scopes you want to call 

   $scopes = New-Object System.Collections.ObjectModel.Collection["string"] 

   $scopes.Add($registrationServiceAppIdUri)

   $app = [Microsoft.Identity.Client.PublicClientApplicationBuilder]::Create($connectorAppId).WithAuthority($authority).WithDefaultRedirectUri().Build()

   [Microsoft.Identity.Client.IAccount] $account = $null

   # Acquiring the token 

   $authResult = $null

   $authResult = $app.AcquireTokenInteractive($scopes).WithAccount($account).ExecuteAsync().ConfigureAwait($false).GetAwaiter().GetResult()

   # Check AuthN result
   If (($authResult) -and ($authResult.AccessToken) -and ($authResult.TenantId)) {
        
   $token = $authResult.AccessToken
   $tenantId = $authResult.TenantId

   Write-Output "Success: Authentication result returned."
        
   }
   Else {
         
   Write-Output "Error: Authentication result, token or tenant id returned with null."
        
   } 
   ```

2. После создания маркера создайте с его помощью SecureString.

   ```powershell
   $SecureToken = $Token | ConvertTo-SecureString -AsPlainText -Force
   ```

3. Выполните следующую команду Windows PowerShell, заменив \<tenant GUID\> идентификатором каталога:

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Token -Token $SecureToken -TenantId <tenant GUID> -Feature ApplicationProxy
   ```

## <a name="next-steps"></a>Дальнейшие действия
* [Публикация приложений с помощью доменного имени](application-proxy-configure-custom-domain.md)
* [Включение единого входа](application-proxy-configure-single-sign-on-with-kcd.md)
* [Устранение неполадок с прокси приложения](application-proxy-troubleshoot.md)