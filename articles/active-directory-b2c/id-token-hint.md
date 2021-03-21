---
title: Определение технического профиля указания маркера идентификации в настраиваемой политике
titleSuffix: Azure AD B2C
description: Определение технического профиля указания маркера идентификации в пользовательской политике в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/16/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: d77e145cabcef2931d5fe6e76599da7931e576e8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97669165"
---
# <a name="define-an-id-token-hint-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Определение технического профиля указания маркера идентификации в настраиваемой политике Azure Active Directory B2C

Azure AD B2C позволяет приложениям проверяющей стороны передавать входящие JWT в рамках запроса авторизации OAuth2. Маркер JWT может быть выдан приложением проверяющей стороны или поставщиком удостоверений и может передавать указание о пользователе или запросе на авторизацию. Azure AD B2C проверяет подпись, имя издателя и аудиторию маркера и извлекает утверждение из входящего токена.

## <a name="use-cases"></a>Варианты использования

Это решение можно использовать для отправки данных в Azure AD B2C, инкапсулированных в один маркер JWT. [Решение регистрация с приглашением по электронной почте](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md), в котором системный администратор может отправить подписанное приглашение пользователям, зависит от id_token_hint. Только пользователи с доступом к приглашению на приглашение могут создать учетную запись в каталоге.

## <a name="token-signing-approach"></a>Подход к подписыванию маркеров

С id_token_hint поставщик маркера (приложение проверяющей стороны или поставщик удостоверений) формирует маркер, а затем подписывает его с помощью ключа подписывания, чтобы доказать, что маркер поступает из надежного источника. Ключ подписывания может быть симметричным или асимметричным. Симметричное шифрование или шифрование с закрытым ключом использует общий секрет для подписи и проверки подписи. Асимметричная криптография или шифрование с открытым ключом — это система шифрования, использующая как закрытый, так и открытый ключ. Закрытый ключ известен только издателю маркера и используется для подписания маркера. Открытый ключ используется совместно с политикой Azure AD B2C для проверки сигнатуры маркера.

## <a name="token-format"></a>Формат токена

Id_token_hint должен быть допустимым маркером JWT. В следующей таблице перечислены утверждения, которые являются обязательными. Дополнительные утверждения являются необязательными.

| Имя | Утверждение | Пример значения | Описание |
| ---- | ----- | ------------- | ----------- |
| Аудитория | `aud` | `a489fc44-3cc0-4a78-92f6-e413cd853eae` | Определяет целевого получателя маркера. Аудитория — это произвольная строка, определяемая издателем маркера. Azure AD B2C проверяет это значение и отклоняет токен, если он не совпадает.  |
| Издатель | `iss` |`https://localhost` | Идентифицирует службу маркеров безопасности (поставщик маркеров). Издатель — это произвольный URI, определяемый издателем маркера. Azure AD B2C проверяет это значение и отклоняет токен, если он не совпадает.  |
| Время окончания срока действия | `exp` | `1600087315` | Время окончания срока действия маркера в виде времени эпохи. Azure AD B2C проверяет это значение и отклоняет маркер, если срок действия маркера истек.|
| Не ранее | `nbf` | `1599482515` | Время начала срока действия маркера в виде времени эпохи. Оно обычно совпадает со временем выдачи маркера. Azure AD B2C проверяет это значение и отклоняет маркер, если время существования маркера недопустимо. |

 Следующий токен является примером допустимого маркера идентификации:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}.{
  "displayName": " John Smith",
  "userId": "john.s@contoso.com",
  "nbf": 1599482515,
  "exp": 1600087315,
  "iss": "https://localhost",
  "aud": "a489fc44-3cc0-4a78-92f6-e413cd853eae"
}
```

## <a name="protocol"></a>Протокол

Атрибуту **Name** элемента **Protocol** необходимо присвоить значение `None`. Например, протокол для **IdTokenHint_ExtractClaims** технического профиля `None` :

```xml
<TechnicalProfile Id="IdTokenHint_ExtractClaims">
  <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
  <Protocol Name="None" />
  ...
```

Технический профиль вызывается из шага оркестрации с типом `GetClaims` .

```xml
<OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
``` 

## <a name="output-claims"></a>Исходящие утверждения

Элемент **OutputClaims** содержит список заявок, извлекаемых из маркера JWT. Может потребоваться связать имя утверждения, определенного в политике, с именем, определенным в маркере JWT. Можно также включить утверждения, которые не возвращаются маркером JWT, при условии, что задан `DefaultValue` атрибут.

## <a name="metadata"></a>Метаданные

При использовании симметричного ключа важны следующие метаданные. 

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| issuer | Да | Идентифицирует службу маркеров безопасности (поставщик маркеров). Это значение должно совпадать с `iss` утверждением в утверждении маркера JWT. | 
| IdTokenAudience | Да | Определяет целевого получателя маркера. Должно совпадать с `aud` утверждением в утверждении маркера JWT. | 

При использовании асимметричного ключа важны следующие метаданные. 

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| METADATA| Да | URL-адрес, указывающий на документ конфигурации издателя маркера, который также называется OpenID Connect хорошо известной конечной точкой конфигурации.   |
| issuer | Нет | Идентифицирует службу маркеров безопасности (поставщик маркеров). Это значение можно использовать для перезаписи значения, настроенного в метаданных, и должно совпадать с `iss` утверждением в утверждении токена JWT. |  
| IdTokenAudience | Нет | Определяет целевого получателя маркера. Должно совпадать с `aud` утверждением в утверждении маркера JWT. |  

## <a name="cryptographic-keys"></a>Криптографические ключи

При использовании симметричного ключа элемент **криптографиккэйс** содержит следующий атрибут:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| client_secret | Да | Криптографический ключ, используемый для проверки подписи токена JWT.|


## <a name="how-to-guide"></a>Практическое руководство

### <a name="issue-a-token-with-symmetric-keys"></a>Выдача маркера с симметричными ключами

#### <a name="step-1-create-a-shared-key"></a>Шаг 1. Создание общего ключа 

Создайте ключ, который можно использовать для подписания маркера. Например, используйте следующий код PowerShell для создания ключа.

```powershell
$bytes = New-Object Byte[] 32
$rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rand.GetBytes($bytes)
$rand.Dispose()
$newClientSecret = [System.Convert]::ToBase64String($bytes)
$newClientSecret
```

Этот код создает секретную строку, например `VK62QTn0m1hMcn0DQ3RPYDAr6yIiSvYgdRwjZtU5QhI=` .

#### <a name="step-2-add-the-signing-key-to-azure-ad-b2c"></a>Шаг 2. Добавьте ключ подписывания в Azure AD B2C

Тот же ключ, который используется издателем маркера, необходимо создать в ключах политики Azure AD B2C.  

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" в разделе **Политики** выберите **Identity Experience Framework**.
1. Выбор **ключей политики** 
1. Выберите **вручную**.
1. Для параметра **Имя** используйте значение `IdTokenHintKey`.  
   Префикс `B2C_1A_` может быть добавлен автоматически.
1. В поле **секрет** введите созданный ранее ключ входа.
1. Для параметра **Использование ключа** задайте значение **Шифрование**.
1. Нажмите кнопку **создания**.
1. Убедитесь, что вы создали ключ `B2C_1A_IdTokenHintKey`.


#### <a name="step-3-add-the-id-token-hint-technical-profile"></a>Шаг 3. Добавление технического профиля указания маркера идентификации

Следующий технический профиль проверяет маркер и извлекает утверждения. 

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <Item Key="IdTokenAudience">a489fc44-3cc0-4a78-92f6-e413cd853eae</Item>
        <Item Key="issuer">https://localhost</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_IdTokenHintKey" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-4-prepare-your-policy"></a>Шаг 4. Подготовка политики

Завершите шаг [Настройка политики](#configure-your-policy) .

#### <a name="step-5-prepare-the-code"></a>Шаг 5. Подготовка кода  

[Пример GitHub](https://github.com/azure-ad-b2c/id_token_hint/tree/master/dotnet_core_symmetric_key) — это веб-приложение ASP.NET и консольное приложение, которое создает маркер идентификации, подписанный с помощью симметричного ключа. 


### <a name="issue-a-token-with-asymmetric-keys"></a>Выдача маркера с асимметричными ключами

При использовании асимметричного ключа маркер подписывается с помощью сертификатов RSA. В этом приложении размещается конечная точка с открытым ИДЕНТИФИКАТОРом Connect и конечная точка JSON Web Keys (JWKs), которая используется Azure AD B2C для проверки подписи маркера идентификации.

Издатель маркера должен предоставлять следующие конечные точки:

* `/.well-known/openid-configuration` — Хорошо известная конечная точка конфигурации с релевантной информацией о токене, например имя поставщика маркера и ссылка на конечную точку ЖВК. 
* `/.well-known/keys` — Конечная точка JSON Web Key (ЖВК) с открытым ключом, используемой для подписания ключа (с помощью закрытого ключа).

См. пример контроллера MVC .NET [токенметадатаконтроллер. CS](https://github.com/azure-ad-b2c/id-token-builder/blob/master/source-code/B2CIdTokenBuilder/Controllers/TokenMetadataController.cs) .

#### <a name="step-1-prepare-a-self-signed-certificate"></a>Шаг 1. Подготовка самозаверяющего сертификата

Если у вас еще нет сертификата, вы можете использовать самозаверяющий сертификат для этого руководства. В Windows для создания сертификата используется командлет PowerShell [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate).

Выполните эту команду PowerShell, чтобы создать самозаверяющий сертификат. Измените аргумент `-Subject`, указав реальные значения приложения и имени клиента Azure  AD B2C. Можно также скорректировать дату `-NotAfter`, чтобы указать другой срок действия сертификата.

```PowerShell
New-SelfSignedCertificate `
    -KeyExportPolicy Exportable `
    -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -KeyUsage DigitalSignature `
    -NotAfter (Get-Date).AddMonths(12) `
    -CertStoreLocation "Cert:\CurrentUser\My"
```


#### <a name="step-2-add-the-id-token-hint-technical-profile"></a>Шаг 2. Добавление технического профиля указания маркера идентификации 

Следующий технический профиль проверяет маркер и извлекает утверждения. Измените URI метаданных на хорошо известную конечную точку конфигурации издателя токена.  

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <!-- Replace with your endpoint location -->
        <Item Key="METADATA">https://your-app.azurewebsites.net/.well-known/openid-configuration</Item>
        <Item Key="IdTokenAudience">your_optional_audience</Item> -->
        <!-- <Item Key="issuer">your_optional_token_issuer_override</Item> -->
      </Metadata>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-3-prepare-your-policy"></a>Шаг 3. Подготовка политики

Завершите шаг [Настройка политики](#configure-your-policy) .

#### <a name="step-4-prepare-the-code"></a>Шаг 4. Подготовка кода 

Этот [Пример](https://github.com/azure-ad-b2c/id-token-builder) веб-приложения GitHub ASP.NET создает маркеры идентификаторов и размещает конечные точки метаданных, необходимые для использования параметра "id_token_hint" в Azure AD B2C.


### <a name="configure-your-policy"></a>Настройка политики

Для симметричных и асимметричных подходов `id_token_hint` технический профиль вызывается из шага оркестрации с типом `GetClaims` и должен указывать входные утверждения политики проверяющей стороны.

1. Добавьте IdTokenHint_ExtractClaims технический профиль в политику расширения.
1. Добавьте следующий шаг оркестрации к пути взаимодействия пользователя в качестве первого элемента.  

    ```xml
    <OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
    ``` 
1. В политике проверяющей стороны повторите те же входные утверждения, которые были настроены в техническом профиле IdTokenHint_ExtractClaims. Пример:
    ```xml
   <RelyingParty>
     <DefaultUserJourney ReferenceId="SignUp" />
     <TechnicalProfile Id="PolicyProfile">
       <DisplayName>PolicyProfile</DisplayName>
       <Protocol Name="OpenIdConnect" />
       <InputClaims>
         <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
        </InputClaims>
       <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
       </OutputClaims>
       <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```

В зависимости от бизнес-требований может потребоваться добавить проверки маркеров, например проверить формат адреса электронной почты. Для этого добавьте шаги оркестрации, которые вызывают [технический профиль преобразования утверждений](claims-transformation-technical-profile.md). Также добавьте [самостоятельно утвержденный технический профиль](self-asserted-technical-profile.md) для отображения сообщения об ошибке. 

### <a name="create-and-sign-a-token"></a>Создание и подписание маркера

В примерах GitHub показано, как создать такой маркер для выдаче JWT, который позже отправляется в качестве `id_token_hint` параметра строки запроса. Ниже приведен пример запроса авторизации с параметром id_token_hint
 
```html
https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/B2C_1A_signup_signin/oauth2/v2.0/authorize?client_id=63ba0d17-c4ba-47fd-89e9-31b3c2734339&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login&id_token_hint=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkaXNwbGF5TmFtZSI6IiBKb2huIFNtaXRoIiwidXNlcklkIjoiam9obi5zQGNvbnRvc28uY29tIiwibmJmIjoxNTk5NDgyNTE1LCJleHAiOjE2MDAwODczMTUsImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwiYXVkIjoiYTQ4OWZjNDQtM2NjMC00YTc4LTkyZjYtZTQxM2NkODUzZWFlIn0.nPmLXydI83PQCk5lRBYUZRu_aX58pL1khahHyQuupig
```

## <a name="next-steps"></a>Дальнейшие действия

- Проверьте [регистрацию с помощью решения "пригласить электронное письмо](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md) " в репозитории сообщества GitHub Azure AD B2C.
