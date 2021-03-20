---
title: Управление пользовательскими политиками с помощью PowerShell
titleSuffix: Azure AD B2C
description: Используйте командлет PowerShell Azure Active Directory (Azure AD) для программного управления пользовательскими политиками Azure AD B2C. Создание, чтение, обновление и удаление настраиваемых политик с помощью PowerShell.
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: fcc482e215e646fec20516f35641bd05398d2f2d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96928721"
---
# <a name="manage-azure-ad-b2c-custom-policies-with-azure-powershell"></a>Управление Azure AD B2C пользовательскими политиками с помощью Azure PowerShell

Azure PowerShell предоставляет несколько командлетов для управления пользовательскими политиками на основе командной строки и сценариев в клиенте Azure AD B2C. Узнайте, как использовать модуль Azure AD PowerShell для:

* Перечисление пользовательских политик в клиенте Azure AD B2C
* Скачивание политики из клиента
* Обновление существующей политики путем перезаписи содержимого
* Отправка новой политики в клиент Azure AD B2C
* Удаление настраиваемой политики из клиента

## <a name="prerequisites"></a>Предварительные условия

* [Azure AD B2C клиент](tutorial-create-tenant.md)и учетные данные для пользователя в каталоге с ролью [администратора политики B2C инфраструктура процедур идентификации](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Пользовательские политики](custom-policy-get-started.md) , отправленные в клиент
* [**Модуль предварительной версии** Azure AD PowerShell для Graph](/powershell/azure/active-directory/install-adv2)

## <a name="connect-powershell-session-to-b2c-tenant"></a>Подключение сеанса PowerShell к клиенту B2C

Для работы с пользовательскими политиками в клиенте Azure AD B2C сначала необходимо подключить сеанс PowerShell к клиенту с помощью команды [Connect-AzureAD][Connect-AzureAD] .

Выполните следующую команду, заменив `{b2c-tenant-name}` именем клиента Azure AD B2C. Войдите под учетной записью, которой назначена роль [администратора политики B2C инфраструктура процедур идентификации](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator) в каталоге.

```PowerShell
Connect-AzureAD -Tenant "{b2c-tenant-name}.onmicrosoft.com"
```

Пример выходных данных команды, показывающей успешный вход в систему:

```Console
PS C:\> Connect-AzureAD -Tenant "contosob2c.onmicrosoft.com"

Account               Environment TenantId                             TenantDomain                 AccountType
-------               ----------- --------                             ------------                 -----------
azureuser@contoso.com AzureCloud  00000000-0000-0000-0000-000000000000 contosob2c.onmicrosoft.com   User
```

## <a name="list-all-custom-policies-in-the-tenant"></a>Перечисление всех пользовательских политик в клиенте

Обнаружение настраиваемых политик позволяет администратору Azure AD B2C просматривать, управлять и добавлять бизнес-логику в свои операции. Используйте команду [Get-азуреадмструстфрамеворкполици][Get-AzureADMSTrustFrameworkPolicy] , чтобы получить список идентификаторов пользовательских политик в клиенте Azure AD B2C.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy
```

Пример выходных данных команды:

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy

Id
--
B2C_1A_TrustFrameworkBase
B2C_1A_TrustFrameworkExtensions
B2C_1A_signup_signin
B2C_1A_ProfileEdit
B2C_1A_PasswordReset
```

## <a name="download-a-policy"></a>Скачать политику

После просмотра списка идентификаторов политик можно ориентироваться на определенную политику с помощью [Get-азуреадмструстфрамеворкполици][Get-AzureADMSTrustFrameworkPolicy] , чтобы скачать его содержимое.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy [-Id <policyId>]
```

В этом примере загружается политика с ИДЕНТИФИКАТОРом *B2C_1A_signup_signin* :

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0" TenantId="contosob2c.onmicrosoft.com" PolicyId="B2C_1A_signup_signin" PublicPolicyUri="http://contosob2c.onmicrosoft.com/B2C_1A_signup_signin" TenantObjectId="00000000-0000-0000-0000-000000000000">
  <BasePolicy>
    <TenantId>contosob2c.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

Чтобы изменить содержимое политики локально, отправьте выходные данные команды по конвейеру в файл с `-OutputFilePath` аргументом, а затем откройте файл в любом удобном редакторе.

Пример команды, отправляющей выходные данные в файл:

```PowerShell
# Download and send policy output to a file
Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -OutputFilePath C:\RPPolicy.xml
```

## <a name="update-an-existing-policy"></a>Обновление существующей политики

После изменения созданного или скачанного файла политики можно опубликовать обновленную политику в Azure AD B2C с помощью команды [Set-азуреадмструстфрамеворкполици][Set-AzureADMSTrustFrameworkPolicy] .

Если вы выпустили `Set-AzureADMSTrustFrameworkPolicy` команду с идентификатором политики, которая уже существует в клиенте Azure AD B2C, содержимое этой политики перезаписывается.

```PowerShell
Set-AzureADMSTrustFrameworkPolicy [-Id <policyId>] -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Пример команды:

```PowerShell
# Update an existing policy from file
Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
```

Дополнительные примеры см. в справочнике по командам [Set-азуреадмструстфрамеворкполици][Set-AzureADMSTrustFrameworkPolicy] .

## <a name="upload-a-new-policy"></a>Отправка новой политики

При внесении изменений в настраиваемую политику, которая выполняется в рабочей среде, может потребоваться опубликовать несколько версий политики для сценариев тестирования отката или A/B. Вы также можете создать копию существующей политики, изменить ее с помощью нескольких небольших изменений, а затем передать ее в качестве новой политики для использования другим приложением.

Используйте команду [New-азуреадмструстфрамеворкполици][New-AzureADMSTrustFrameworkPolicy] , чтобы отправить новую политику:

```PowerShell
New-AzureADMSTrustFrameworkPolicy -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Пример команды:

```PowerShell
# Add new policy from file
New-AzureADMSTrustFrameworkPolicy -InputFilePath C:\SignUpOrSignInv2.xml
```

## <a name="delete-a-custom-policy"></a>Удаление настраиваемой политики

Для поддержания жизненного цикла операций очистки рекомендуется периодически удалять неиспользуемые пользовательские политики. Например, может потребоваться удалить старые версии политик после выполнения миграции на новый набор политик и проверки функциональности новых политик. Кроме того, при попытке публикации набора пользовательских политик и получения сообщения об ошибке может иметь смысл удалить политики, созданные в составе неисправного выпуска.

Удалите политику из клиента с помощью команды [Remove-азуреадмструстфрамеворкполици][Remove-AzureADMSTrustFrameworkPolicy] .

```PowerShell
Remove-AzureADMSTrustFrameworkPolicy -Id <policyId>
```

Пример команды:

```PowerShell
# Delete an existing policy
Remove-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
```

## <a name="troubleshoot-policy-upload"></a>Устранение неполадок при отправке политики

При попытке публикации новой настраиваемой политики или обновления существующей политики неправильное форматирование и ошибки XML в цепочке наследования файлов политики могут привести к сбоям при проверке.

Например, при обновлении политики с содержимым, содержащим неправильно сформированный XML (для краткости выводятся результаты усечения):

```Console
PS C:\> Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
Set-AzureADMSTrustFrameworkPolicy : Error occurred while executing PutTrustFrameworkPolicy
Code: AADB2C
Message: Validation failed: 1 validation error(s) found in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com".Schema validation error found at line
14 col 55 in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com": The element 'OutputClaims' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06' cannot contain text. List of possible elements expected: 'OutputClaim' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06'.
...
```

Сведения об устранении неполадок настраиваемых политик см. в разделе [Устранение неполадок Azure AD B2C пользовательских политик и инфраструктуры процедур идентификации](./troubleshoot-custom-policies.md).

## <a name="next-steps"></a>Дальнейшие действия

Сведения об использовании PowerShell для развертывания настраиваемых политик в рамках конвейера непрерывной интеграции и непрерывной поставки (CI/CD) см. в статье [развертывание настраиваемых политик в конвейере Azure DevOps](deploy-custom-policies-devops.md).

<!-- LINKS - External -->
[Connect-AzureAD]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[Get-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[New-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/new-azureadmstrustframeworkpolicy
[Remove-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/remove-azureadmstrustframeworkpolicy
[Set-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/set-azureadmstrustframeworkpolicy
