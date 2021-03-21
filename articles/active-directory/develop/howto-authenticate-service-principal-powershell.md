---
title: Создание удостоверения приложения Azure (PowerShell) | Службы
titleSuffix: Microsoft identity platform
description: Использование Azure PowerShell для создания приложения Azure Active Directory и субъекта-службы с последующим предоставлением доступа к ресурсам посредством управления доступом на основе ролей. В статье показано, как выполнять аутентификацию приложения с помощью сертификата.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev , devx-track-azurepowershell
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.date: 02/22/2021
ms.author: ryanwi
ms.reviewer: tomfitz
ms.openlocfilehash: b27af53d615fa9c0c46699a52a004098dc46b7b2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101688541"
---
# <a name="how-to-use-azure-powershell-to-create-a-service-principal-with-a-certificate"></a>Практическое руководство по использованию Azure PowerShell для создания субъекта-службы с сертификатом

При наличии приложения или сценария, которому требуется доступ к ресурсам, можно настроить удостоверение для приложения и аутентифицировать его с помощью собственных учетных данных. Этот идентификатор известен как субъект-служба. Такой подход позволяет выполнить следующие действия:

* Назначить удостоверению приложения разрешения, которые отличаются от ваших разрешений. Как правило, приложение получает именно те разрешения, которые требуются для его работы.
* Использовать сертификат для аутентификации при выполнении автоматического сценария.

> [!IMPORTANT]
> Вместо создания субъекта-службы вы можете применить управляемые удостоверения ресурсов Azure в качестве удостоверения приложения. Если код выполняется в службе, которая поддерживает управляемые удостоверения и обращается к ресурсам, которые поддерживают проверку подлинности Azure Active Directory (Azure AD), то управляемые удостоверения будут оптимальным выбором. Дополнительные сведения об управляемых удостоверениях для ресурсов Azure, включая службы, которые в настоящее время поддерживают их, см. в разделе [Что такое управляемые удостоверения для ресурсов Azure?](../managed-identities-azure-resources/overview.md)

В этой статье показано, как создать субъект-службу, который выполняет аутентификацию с помощью сертификата. Настройка субъекта-службы с паролем описана в статье [Создание субъекта-службы Azure с помощью Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).

Для выполнения задач из этой статься требуется Azure PowerShell [последней версии](/powershell/azure/install-az-ps).

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="required-permissions"></a>Необходимые разрешения

Для работы с этой статьей у вас должен быть достаточный уровень разрешений в подписках Azure AD и Azure. В частности, вы должны иметь право на создание приложения в Azure AD и назначение роли субъекту-службе.

Проверить, есть ли у вас соответствующие разрешения, проще всего на портале. Ознакомьтесь с [проверкой наличия необходимых разрешений](howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="assign-the-application-to-a-role"></a>Назначение приложению роли
Чтобы обеспечить доступ к ресурсам в подписке, необходимо назначить приложению роль. Укажите, какая роль предоставляет приложению необходимые разрешения. Дополнительные сведения о доступных ролях см. в статье [встроенные роли Azure](../../role-based-access-control/built-in-roles.md).

Вы можете задать область действия на уровне подписки, группы ресурсов или ресурса. Разрешения наследуют более низкие уровни области действия. Например, Добавление приложения в роль *читатель* для группы ресурсов означает, что она может читать группу ресурсов и все содержащиеся в ней ресурсы. Чтобы разрешить приложению выполнять такие действия, как перезагрузка, запуск и завершение экземпляров, выберите роль *участник* .

## <a name="create-service-principal-with-self-signed-certificate"></a>Создание субъекта-службы с самозаверяющим сертификатом

Ниже приводится простой пример сценария. Для создания субъекта-службы с самозаверяющим сертификатом используются [новые азадсервицепринЦипал](/powershell/module/az.resources/new-azadserviceprincipal) . с помощью [New-азролеассигнмент](/powershell/module/az.resources/new-azroleassignment) можно назначить роль [читателя](../../role-based-access-control/built-in-roles.md#reader) субъекту-службе. Назначение ролей ограничивается текущей выбранной подпиской Azure. Чтобы выбрать другую подписку, выполните командлет [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext).

> [!NOTE]
> Командлет New-SelfSignedCertificate и модуль PKI в настоящее время не поддерживаются в PowerShell Core. 

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" `
  -Subject "CN=exampleappScriptCert" `
  -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

$sp = New-AzADServicePrincipal -DisplayName exampleapp `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
Sleep 20
New-AzRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $sp.ApplicationId
```

Пример бездействует 20 секунд, чтобы данные нового субъекта-службы распространились в Azure AD. Если период ожидания окажется недостаточным, появится следующее сообщение об ошибке: "Субъект {идентификатор} не существует в каталоге {идентификатор_каталога}". Чтобы устранить эту ошибку, подождите немного и выполните команду **New-AzRoleAssignment** еще раз.

Вы можете задать область назначения ролей для определенной группы ресурсов с помощью параметра **ResourceGroupName**. Кроме того, вы можете задать область для определенного ресурса с помощью параметров **ResourceType** и **ResourceName**. 

Если у вас **нет Windows 10 или Windows Server 2016**, скачайте [командлет New-СЕЛФСИГНЕДЦЕРТИФИКАТИКС](https://www.pkisolutions.com/tools/pspki/New-SelfSignedCertificateEx/) из решений PKI. Извлеките содержимое сценария и импортируйте требуемый командлет.

```powershell
# Only run if you could not use New-SelfSignedCertificate
Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
```

Замените в сценарии следующие две строки, чтобы создать сертификат.

```powershell
New-SelfSignedCertificateEx -StoreLocation CurrentUser `
  -Subject "CN=exampleapp" `
  -KeySpec "Exchange" `
  -FriendlyName "exampleapp"
$cert = Get-ChildItem -path Cert:\CurrentUser\my | where {$PSitem.Subject -eq 'CN=exampleapp' }
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Предоставление сертификата с помощью автоматизированного сценария PowerShell

При каждом входе в качестве субъекта-службы укажите идентификатор клиента каталога для приложения AD. Клиент — это экземпляр Azure AD.

```powershell
$TenantId = (Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
$ApplicationId = (Get-AzADApplication -DisplayNameStartWith exampleapp).ApplicationId

$Thumbprint = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -eq "CN=exampleappScriptCert" }).Thumbprint
Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

## <a name="create-service-principal-with-certificate-from-certificate-authority"></a>Создание субъекта-службы с помощью сертификата из центра сертификации

В приведенном ниже примере создается субъект-служба с сертификатом, выданным центром сертификации. Назначение ограничено указанной подпиской Azure. Он добавляет субъект-службу к роли [читателя](../../role-based-access-control/built-in-roles.md#reader) . Если при назначении ролей возникнет ошибка, назначение выполняется повторно.

```powershell
Param (
 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName,

 [Parameter(Mandatory=$true)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword
 )

 Connect-AzAccount
 Import-Module Az.Resources
 Set-AzContext -Subscription $SubscriptionId

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force

 $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
 $KeyValue = [System.Convert]::ToBase64String($PFXCert.GetRawCertData())

 $ServicePrincipal = New-AzADServicePrincipal -DisplayName $ApplicationDisplayName
 New-AzADSpCredential -ObjectId $ServicePrincipal.Id -CertValue $KeyValue -StartDate $PFXCert.NotBefore -EndDate $PFXCert.NotAfter
 Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $ServicePrincipal.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzRoleAssignment -ObjectId $ServicePrincipal.Id -ErrorAction SilentlyContinue
    $Retries++;
 }

 $NewRole
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Предоставление сертификата с помощью автоматизированного сценария PowerShell
При каждом входе в качестве субъекта-службы укажите идентификатор клиента каталога для приложения AD. Клиент — это экземпляр Azure AD.

```powershell
Param (

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword,

 [Parameter(Mandatory=$true)]
 [String] $ApplicationId,

 [Parameter(Mandatory=$true)]
 [String] $TenantId
 )

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
 $PFXCert = New-Object `
  -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 `
  -ArgumentList @($CertPath, $CertPassword)
 $Thumbprint = $PFXCert.Thumbprint

 Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

Идентификаторы приложения и клиента не являются конфиденциальными, поэтому их можно внедрить непосредственно в скрипт. Чтобы получить идентификатор клиента, используйте следующий командлет:

```powershell
(Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
```

Чтобы получить идентификатор приложения, используйте следующий командлет:

```powershell
(Get-AzADApplication -DisplayNameStartWith {display-name}).ApplicationId
```

## <a name="change-credentials"></a>Изменение учетных данных

Чтобы изменить учетные данные для приложения AD из-за нарушения безопасности или истечения их срока действия, используйте командлеты [Remove-AzADAppCredential](/powershell/module/az.resources/remove-azadappcredential) и [New-AzADAppCredential](/powershell/module/az.resources/new-azadappcredential).

Чтобы удалить все учетные данные для приложения, используйте следующую команду.

```powershell
Get-AzADApplication -DisplayName exampleapp | Remove-AzADAppCredential
```

Чтобы добавить значение сертификата, создайте самозаверяющий сертификат, как показано в этой статье. Затем используйте следующую команду.

```powershell
Get-AzADApplication -DisplayName exampleapp | New-AzADAppCredential `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
```

## <a name="debug"></a>Отладка

При создании субъекта-службы могут возникнуть следующие ошибки.

* **"Authentication_Unauthorized"** (Аутентификация не выполнена) или **"No subscription found in the context"** (В контексте не удалось найти подписку). Эта ошибка может отобразиться, если учетная запись не имеет [необходимых разрешений](#required-permissions) в Azure AD для регистрации приложения. Как правило, эта ошибка возникает, когда только пользователи с правами администратора в Azure Active Directory могут регистрировать приложения, а ваша учетная запись не является администратором. Попросите администратора либо назначить вам роль администратора, либо разрешить пользователям регистрировать приложения.

* Ваша учетная запись **"не авторизована для выполнения действия 'Microsoft.Authorization/roleAssignments/write' с областью '/subscriptions/{guid}'."** Эта ошибка возникает, когда учетная запись не имеет достаточно разрешений для назначения роли удостоверению. Попросите администратора подписки назначить вам роль администратора доступа пользователей.

## <a name="next-steps"></a>Дальнейшие действия

* Настройка субъекта-службы с паролем описана в статье [Создание субъекта-службы Azure с помощью Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).
* Более подробное описание приложений и субъектов-служб см. в статье [Объекты приложений и объекты участников-служб](app-objects-and-service-principals.md).
* Дополнительные сведения о проверке подлинности Azure AD см. [здесь](./authentication-vs-authorization.md).
* Сведения о работе с регистрацией приложений с помощью **Microsoft Graph** см. в справочнике по API [приложений](/graph/api/resources/application) .
