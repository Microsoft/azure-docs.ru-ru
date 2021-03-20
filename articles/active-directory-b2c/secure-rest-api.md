---
title: Защита службы с поддержкой RESTful в Azure AD B2C
titleSuffix: Azure AD B2C
description: Защита обмена пользовательскими утверждениями REST API в Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: f6907db7f6e53247a8f2fc0042e8c8e6b081dbd3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97516372"
---
# <a name="secure-your-restful-services"></a>Защита служб с поддержкой RESTful 

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

При интеграции REST API через путь взаимодействия пользователя Azure AD B2C вам нужно защитить конечную точку REST API с помощью аутентификации. Так вы сможете гарантировать, что вызовы к конечной точке REST API смогут выполнять только те службы, у которых есть правильные учетные данные, например Azure AD B2C.

Сведения о том, как интегрировать REST API в путь взаимодействия пользователя Azure AD B2C, можно найти в статье о [проверке вводимых пользователем данных](custom-policy-rest-api-claims-validation.md) и в статье [Добавление возможности обмена утверждениями REST API в настраиваемые политики](custom-policy-rest-api-claims-exchange.md).

В этой статье рассматривается защита REST API с помощью нескольких механизмов аутентификации: обычная через HTTP, на основе сертификата клиента и OAuth2. 

## <a name="prerequisites"></a>Предварительные требования

Выполните действия, приведенные в одном из следующих кратких руководств.

- [Интеграция обмена утверждениями REST API в пути взаимодействия пользователя Azure AD B2C как проверка входных данных.](custom-policy-rest-api-claims-validation.md)
- [Добавление возможности обмена утверждениями REST API в настраиваемые политики.](custom-policy-rest-api-claims-exchange.md)

## <a name="http-basic-authentication"></a>Обычная аутентификация через HTTP

Обычная аутентификация через HTTP определяется стандартом [RFC 2617](https://tools.ietf.org/html/rfc2617). Обычная аутентификация работает следующим образом: Azure AD B2C отправляет HTTP-запрос с учетными данными клиента в заголовке авторизации. Эти учетные данные имеют формат строки в кодировке Base64 со значениями "имя:пароль".  

### <a name="add-rest-api-username-and-password-policy-keys"></a>Добавление ключей политики для имени пользователя и пароля REST API

Чтобы настроить в техническом профиле REST API обычную аутентификацию через HTTP, создайте следующие криптографические ключи для хранения имени пользователя и пароля:

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. В пункте **Параметры** выберите **Manual** (Вручную).
1. В поле **Имя** введите **RestApiUsername**.
    Префикс *B2C_1A_* может быть добавлен автоматически.
1. В поле **Секрет** введите имя пользователя REST API.
1. Для параметра **Использование ключа** задайте значение **Шифрование**.
1. Нажмите кнопку **создания**.
1. Снова щелкните **Ключи политики**.
1. Выберите **Добавить**.
1. В пункте **Параметры** выберите **Manual** (Вручную).
1. В поле **Имя** введите **RestApiPassword**.
    Префикс *B2C_1A_* может быть добавлен автоматически.
1. В поле **Секрет** введите пароль REST API.
1. Для параметра **Использование ключа** задайте значение **Шифрование**.
1. Нажмите кнопку **создания**.

### <a name="configure-your-rest-api-technical-profile-to-use-http-basic-authentication"></a>Настройка обычной аутентификации через HTTP в техническом профиле REST API

После создания необходимых ключей настройте метаданные технического профиля REST API, чтобы они ссылались на эти учетные данные.

1. В рабочей папке откройте файл политики расширения (TrustFrameworkExtensions.xml).
1. Выполните поиск технического профиля REST API, например `REST-ValidateProfile` или `REST-GetProfile`.
1. Найдите элемент `<Metadata>`.
1. Установите для параметра *AuthenticationType* значение `Basic`.
1. Установите для параметра *AllowInsecureAuthInProduction* значение `false`.
1. Сразу после завершения элемента `</Metadata>` добавьте следующий фрагмент XML-кода:
    ```xml
    <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_RestApiUsername" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_RestApiPassword" />
    </CryptographicKeys>
    ```

Ниже приведен пример технического профиля RESTful,в котором настроена обычная аутентификация через HTTP.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_RestApiUsername" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_RestApiPassword" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="https-client-certificate-authentication"></a>Аутентификация через HTTPS на основе сертификата клиента

Аутентификация на основе сертификата клиента выполняется взаимно, и в ее ходе служба Azure AD B2C, выполняющая роль клиента, предоставляет серверу сертификат клиента для подтверждения личности. Это происходит в процессе приветствия SSL. Только службы с соответствующими сертификатами, например Azure AD B2C, могут получить доступ к службе REST API. Сертификат клиента — это цифровой сертификат X.509. В рабочих средах он должен быть подписан центром сертификации.

### <a name="prepare-a-self-signed-certificate-optional"></a>Подготовка самозаверяющего сертификата (необязательно)

Если у вас еще нет сертификата, для всех сред, кроме рабочей, можно использовать самозаверяющий сертификат. В Windows для создания сертификата используется командлет PowerShell [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate).

1. Выполните эту команду PowerShell, чтобы создать самозаверяющий сертификат. Измените аргумент `-Subject`, указав реальные значения приложения и имени клиента Azure  AD B2C. Можно также скорректировать дату `-NotAfter`, чтобы указать другой срок действия сертификата.
    ```powershell
    New-SelfSignedCertificate `
        -KeyExportPolicy Exportable `
        -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
        -KeyAlgorithm RSA `
        -KeyLength 2048 `
        -KeyUsage DigitalSignature `
        -NotAfter (Get-Date).AddMonths(12) `
        -CertStoreLocation "Cert:\CurrentUser\My"
    ```    
1. Откройте **Управление сертификатами пользователя** > **Текущий пользователь** > **Персональные** > **Сертификаты** > *имя_приложения.имя_клиента.onmicrosoft.com*.
1. Выберите сертификат, затем щелкните **Действие** > **Все задачи** > **Экспорт**.
1. Нажмите **Да** > **Далее** > **Да, экспортировать закрытый ключ** > **Далее**.
1. Примите значения по умолчанию для **формата файла экспорта**.
1. Укажите пароль для сертификата.

### <a name="add-a-client-certificate-policy-key"></a>Добавление ключа политики для сертификата клиента

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. В поле **Параметры** выберите **Отправить**.
1. В поле **Имя** введите **RestApiClientCertificate**.
    Префикс *B2C_1A_* добавляется автоматически.
1. В поле **Отправка файлов** выберите PFX-файл сертификата с закрытым ключом.
1. В поле **Пароль** введите пароль сертификата.
1. Нажмите кнопку **создания**.

### <a name="configure-your-rest-api-technical-profile-to-use-client-certificate-authentication"></a>Настройка аутентификации с использованием сертификата клиента в техническом профиле REST API

После создания необходимых ключей настройте метаданные технического профиля REST API, чтобы они ссылались на сертификат клиента.

1. В рабочей папке откройте файл политики расширения (TrustFrameworkExtensions.xml).
1. Выполните поиск технического профиля REST API, например `REST-ValidateProfile` или `REST-GetProfile`.
1. Найдите элемент `<Metadata>`.
1. Установите для параметра *AuthenticationType* значение `ClientCertificate`.
1. Установите для параметра *AllowInsecureAuthInProduction* значение `false`.
1. Сразу после завершения элемента `</Metadata>` добавьте следующий фрагмент XML-кода:
    ```xml
    <CryptographicKeys>
       <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_RestApiClientCertificate" />
    </CryptographicKeys>
    ```

Ниже приведен пример технического профиля RESTful,в котором настроена аутентификация с использованием сертификата клиента.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">ClientCertificate</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_RestApiClientCertificate" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="oauth2-bearer-authentication"></a>Аутентификация по носителю OAuth2 

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

Аутентификация по носителю OAuth2 определяется в стандарте [OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (OAuth2.0 Authorization Framework: использование маркера носителя (RFC 6750)). При использовании аутентификации по маркеру носителя OAuth2 служба Azure AD B2C отправляет HTTP-запрос с маркером в заголовке авторизации.

```http
Authorization: Bearer <token>
```

Маркер носителя является непрозрачной строкой. Это может быть маркер доступа JWT или любая строка, которую REST API ожидает от Azure AD B2C в качестве заголовка авторизации. Azure AD B2C поддерживает следующие типы:

- **Маркер носителя.** Чтобы иметь возможность отправить маркер носителя в технический профиль RESTful, политика должна сначала получить маркер носителя, а затем применить его в техническом профиле RESTful.  
- **Статический маркер носителя.** Используйте этот подход, если ваш REST API выдает долгосрочный маркер доступа. Чтобы использовать статический маркер носителя, создайте ключ политики и создайте ссылку на него в техническом профиле RESTful. 


## <a name="using-oauth2-bearer"></a>Использование носителя OAuth2  

В следующих шагах показано, как использовать учетные данные клиента для получения маркера носителя и передачи его в заголовке авторизации при вызовах REST API.  

### <a name="define-a-claim-to-store-the-bearer-token"></a>Определение утверждения для хранения маркера носителя

Утверждение предоставляет временное хранилище данных на время выполнения политики Azure AD B2C. [Схема утверждений](claimsschema.md) — это место, где вы объявляете свои утверждения. Маркер доступа должен храниться в утверждении для последующего использования. 

1. Откройте файл расширения для политики, например <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если такой элемент не существует, добавьте его.
1. Найдите элемент [ClaimsSchema](claimsschema.md). Если такой элемент не существует, добавьте его.
1. Добавьте следующие утверждения в элемент **ClaimsSchema**.  

```xml
<ClaimType Id="bearerToken">
  <DisplayName>Bearer token</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="grant_type">
  <DisplayName>Grant type</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="scope">
  <DisplayName>scope</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

### <a name="acquiring-an-access-token"></a>Получение маркера доступа 

Маркер доступа можно получить любым из нескольких способов: от [поставщика федеративного удостоверения](idp-pass-through-user-flow.md), с помощью REST API, который возвращает маркер доступа, с помощью [потока ROPC](../active-directory/develop/v2-oauth-ropc.md) или [потока учетных данных клиента](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md).  

В следующем примере используется технический профиль REST API для выполнения запроса к конечной точке маркера Azure AD с учетными данными клиента, которые передаются в формате обычной аутентификации через HTTP. Чтобы настроить этот механизм в Azure AD, см. статью [Платформа удостоверений Майкрософт и поток учетных данных клиента OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Возможно, вам придется изменить его с учетом методов взаимодействия с поставщиком удостоверений. 

В параметре ServiceUrl замените заполнитель your-tenant-name именем вашего клиента Azure AD. Доступные варианты описаны в документации [по техническому профилю RESTful](restful-technical-profile.md).

```xml
<TechnicalProfile Id="SecureREST-AccessToken">
  <DisplayName></DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://login.microsoftonline.com/your-tenant-name.onmicrosoft.com/oauth2/v2.0/token</Item>
    <Item Key="AuthenticationType">Basic</Item>
     <Item Key="SendClaimsIn">Form</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_SecureRESTClientId" />
    <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_SecureRESTClientSecret" />
  </CryptographicKeys>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="grant_type" DefaultValue="client_credentials" />
    <InputClaim ClaimTypeReferenceId="scope" DefaultValue="https://secureb2cfunction.azurewebsites.net/.default" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="bearerToken" PartnerClaimType="access_token" />
  </OutputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

### <a name="change-the-rest-technical-profile-to-use-bearer-token-authentication"></a>Изменение технического профиля RESTful для использования аутентификации по маркеру носителя

Для поддержи аутентификации по маркеру носителя в настраиваемой политике измените технический профиль REST API, сделав следующее:

1. В рабочей папке откройте файл политики расширения *TrustFrameworkExtensions.xml*.
1. Найдите узел `<TechnicalProfile>`, содержащий `Id="REST-API-SignUp"`.
1. Найдите элемент `<Metadata>`.
1. Измените значение *AuthenticationType* на *Bearer*, как показано ниже.
    ```xml
    <Item Key="AuthenticationType">Bearer</Item>
    ```
1. Для параметра *UseClaimAsBearerToken* укажите значение *bearerToken*, как показано ниже. Имя *bearerToken* обозначает утверждение, из которого будет получен маркер носителя (исходящее утверждение из `SecureREST-AccessToken`).

    ```xml
    <Item Key="UseClaimAsBearerToken">bearerToken</Item>
    ```
    
1. Обязательно добавьте в качестве входящего утверждения то утверждение, которое использовалось ранее:

    ```xml
    <InputClaim ClaimTypeReferenceId="bearerToken"/>
    ```    

После добавления представленных выше фрагментов кода технический профиль должен выглядеть аналогично следующему XML-коду:

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="UseClaimAsBearerToken">bearerToken</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="bearerToken"/>
      </InputClaims>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="using-a-static-oauth2-bearer"></a>Использование статического носителя OAuth2 

### <a name="add-the-oauth2-bearer-token-policy-key"></a>Добавление ключа политики маркера носителя OAuth2

Чтобы настроить REST API технический профиль с токеном носителя OAuth2, получите маркер доступа от владельца REST API. Затем создайте следующий криптографический ключ для хранения токена носителя.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. Для пункта **Параметры** выберите `Manual`.
1. Введите **имя** ключа политики. Например, `RestApiBearerToken`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
1. В поле **Секрет** введите ранее записанный секрет клиента.
1. Для параметра **Использование ключа** выберите `Encryption`.
1. Нажмите кнопку **создания**.

### <a name="configure-your-rest-api-technical-profile-to-use-the-bearer-token-policy-key"></a>Настройка технического профиля REST API для использования ключа политики маркера носителя

После создания необходимых ключей настройте метаданные технического профиля REST API, чтобы они ссылались на маркер носителя.

1. В рабочей папке откройте файл политики расширения (TrustFrameworkExtensions.xml).
1. Выполните поиск технического профиля REST API, например `REST-ValidateProfile` или `REST-GetProfile`.
1. Найдите элемент `<Metadata>`.
1. Установите для параметра *AuthenticationType* значение `Bearer`.
1. Установите для параметра *AllowInsecureAuthInProduction* значение `false`.
1. Сразу после завершения элемента `</Metadata>` добавьте следующий фрагмент XML-кода:
    ```xml
    <CryptographicKeys>
       <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_RestApiBearerToken" />
    </CryptographicKeys>
    ```

Ниже приведен пример технического профиля RESTful,в котором настроена аутентификация по маркеру носителя.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_RestApiBearerToken" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="api-key-authentication"></a>Проверка подлинности ключа API

Ключ API — это уникальный идентификатор, используемый для проверки подлинности пользователя при доступе к конечной точке REST API. Ключ отправляется в настраиваемом заголовке HTTP. Например, [триггер HTTP для функций Azure](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys) использует `x-functions-key` заголовок HTTP для обнаружения запрашивающей стороны.  

### <a name="add-api-key-policy-keys"></a>Добавление ключей политики ключей API

Чтобы настроить REST API технический профиль с проверкой подлинности с помощью ключа API, создайте следующий криптографический ключ для хранения ключа API:

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. В пункте **Параметры** выберите **Manual** (Вручную).
1. В качестве **имени** введите **рестапикэй**.
    Префикс *B2C_1A_* может быть добавлен автоматически.
1. В поле **секрет** введите ключ REST API.
1. Для параметра **Использование ключа** задайте значение **Шифрование**.
1. Нажмите кнопку **создания**.


### <a name="configure-your-rest-api-technical-profile-to-use-api-key-authentication"></a>Настройка REST API технического профиля для использования проверки подлинности с помощью ключа API

После создания нужного ключа настройте метаданные технического профиля REST API, чтобы они ссылались на учетные данные.

1. В рабочей папке откройте файл политики расширения (TrustFrameworkExtensions.xml).
1. Выполните поиск технического профиля REST API, например `REST-ValidateProfile` или `REST-GetProfile`.
1. Найдите элемент `<Metadata>`.
1. Установите для параметра *AuthenticationType* значение `ApiKeyHeader`.
1. Установите для параметра *AllowInsecureAuthInProduction* значение `false`.
1. Сразу после завершения элемента `</Metadata>` добавьте следующий фрагмент XML-кода:
    ```xml
    <CryptographicKeys>
        <Key Id="x-functions-key" StorageReferenceId="B2C_1A_RestApiKey" />
    </CryptographicKeys>
    ```

**Идентификатор** криптографического ключа определяет заголовок HTTP. В этом примере ключ API отправляется в виде **ключа x-functions-Key**.

Ниже приведен пример технического профиля RESTFUL, настроенного для вызова функции Azure с проверкой подлинности с помощью ключа API:

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">ApiKeyHeader</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="x-functions-key" StorageReferenceId="B2C_1A_RestApiKey" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="next-steps"></a>Дальнейшие действия

- См. дополнительные сведения об элементе [технического профиля RESTful](restful-technical-profile.md).