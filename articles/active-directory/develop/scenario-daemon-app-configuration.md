---
title: Настройка управляющих приложений, вызывающих веб-API — платформа Microsoft Identity | Службы
description: Узнайте, как настроить код для приложения управляющей программы, вызывающего веб-API (Конфигурация приложения).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/19/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 3c52d4d80fd3c77cff5e335967fc9d109212ce29
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578436"
---
# <a name="daemon-app-that-calls-web-apis---code-configuration"></a>Управляющее приложение, вызывающее веб-API — конфигурация кода

Узнайте, как настроить код для управляющего приложения, вызывающего веб-API.

## <a name="microsoft-libraries-supporting-daemon-apps"></a>Библиотеки Майкрософт, поддерживающие управляющие приложения

Следующие библиотеки Microsoft поддерживают приложения управляющей программы:

[!INCLUDE [active-directory-develop-libraries-daemon](../../../includes/active-directory-develop-libraries-daemon.md)]

## <a name="configure-the-authority"></a>Настройка центра

Управляющие приложения используют разрешения приложения, а не делегированные разрешения. Поэтому их поддерживаемый тип учетной записи не может быть учетной записью в любом каталоге организации или в личных учетная запись Майкрософтах (например, Skype, Xbox, Outlook.com). Нет прав администратора клиента для предоставления согласия на управляющее приложение для личной учетной записи Майкрософт. Вам потребуется выбрать *учетные записи в моей организации* или *учетных записях в любой организации*.

Центр, указанный в конфигурации приложения, должен быть клиентом (указание идентификатора клиента или доменного имени, связанного с вашей организацией).

Даже если требуется предоставить многоклиентское средство, следует использовать идентификатор клиента или доменное имя, а **не** `common` или `organizations` с этим потоком, так как служба не может надежно определить, какой клиент следует использовать.

## <a name="configure-and-instantiate-the-application"></a>Настройка и создание экземпляра приложения

В библиотеках MSAL учетные данные клиента (секрет или сертификат) передаются в качестве параметра для создания конфиденциального клиентского приложения.

> [!IMPORTANT]
> Даже если приложение является консольным приложением, которое работает как служба, то, если это управляющее приложение, оно должно быть конфиденциальным клиентским приложением.

### <a name="configuration-file"></a>Файл конфигурации

Файл конфигурации определяет:

- Облачный экземпляр и идентификатор клиента, которые вместе образуют *центр*.
- Идентификатор клиента, полученный при регистрации приложения.
- Секрет клиента или сертификат.

# <a name="net"></a>[.NET](#tab/dotnet)

Ниже приведен пример определения конфигурации в [*appsettings.js*](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/master/1-Call-MSGraph/daemon-console/appsettings.json) файла. Этот пример взят из примера кода [управляющей программы консоли .NET Core](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) на сайте GitHub.

```json
{
  "Instance": "https://login.microsoftonline.com/{0}",
  "Tenant": "[Enter here the tenantID or domain name for your Azure AD tenant]",
  "ClientId": "[Enter here the ClientId for your application]",
  "ClientSecret": "[Enter here a client secret for your application]",
  "CertificateName": "[Or instead of client secret: Enter here the name of a certificate (from the user cert store) as registered with your application]"
}
```

Вы предоставляете либо `ClientSecret` `CertificateName` . Эти параметры являются эксклюзивными.

# <a name="java"></a>[Java](#tab/java)

```Java
 private final static String CLIENT_ID = "";
 private final static String AUTHORITY = "https://login.microsoftonline.com/<tenant>/";
 private final static String CLIENT_SECRET = "";
 private final static Set<String> SCOPE = Collections.singleton("https://graph.microsoft.com/.default");
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Параметры конфигурации для [ примера управляющей программыNode.js](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-console/) находятся в файле *. env* :

```Text 
# Credentials
TENANT_ID=Enter_the_Tenant_Info_Here
CLIENT_ID=Enter_the_Application_Id_Here
CLIENT_SECRET=Enter_the_Client_Secret_Here

# Endpoints
AAD_ENDPOINT=Enter_the_Cloud_Instance_Id_Here
GRAPH_ENDPOINT=Enter_the_Graph_Endpoint_Here
```

# <a name="python"></a>[Python](#tab/python)

При создании конфиденциального клиента с секретами клиента [parameters.js](https://github.com/Azure-Samples/ms-identity-python-daemon/blob/master/1-Call-MsGraph-WithSecret/parameters.json) в файле конфигурации в образце [управляющей программы Python](https://github.com/Azure-Samples/ms-identity-python-daemon) выглядит следующим образом:

```Json
{
  "authority": "https://login.microsoftonline.com/<your_tenant_id>",
  "client_id": "your_client_id",
  "scope": [ "https://graph.microsoft.com/.default" ],
  "secret": "The secret generated by AAD during your confidential app registration",
  "endpoint": "https://graph.microsoft.com/v1.0/users"
}
```

При создании конфиденциального клиента с сертификатами [parameters.js](https://github.com/Azure-Samples/ms-identity-python-daemon/blob/master/2-Call-MsGraph-WithCertificate/parameters.json) в файле конфигурации в образце [управляющей программы Python](https://github.com/Azure-Samples/ms-identity-python-daemon) выглядит следующим образом:

```Json
{
  "authority": "https://login.microsoftonline.com/<your_tenant_id>",
  "client_id": "your_client_id",
  "scope": [ "https://graph.microsoft.com/.default" ],
  "thumbprint": "790E... The thumbprint generated by AAD when you upload your public cert",
  "private_key_file": "server.pem",
  "endpoint": "https://graph.microsoft.com/v1.0/users"
}
```

---

### <a name="instantiate-the-msal-application"></a>Создание экземпляра приложения MSAL

Чтобы создать экземпляр приложения MSAL, добавьте, создайте ссылку или импортируйте пакет MSAL (в зависимости от языка).

Конструкция отличается в зависимости от того, используете ли вы клиентские секреты или сертификаты (или, в качестве расширенного сценария, подписанные утверждения).

#### <a name="reference-the-package"></a>Ссылка на пакет

Сослаться на пакет MSAL в коде приложения.

# <a name="net"></a>[.NET](#tab/dotnet)

Добавьте в приложение пакет NuGet [Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client) , а затем добавьте `using` в код директиву для ссылки на него.

В MSAL.NET конфиденциальное клиентское приложение представлено `IConfidentialClientApplication` интерфейсом.

```csharp
using Microsoft.Identity.Client;
IConfidentialClientApplication app;
```

# <a name="java"></a>[Java](#tab/java)

```java
import com.microsoft.aad.msal4j.ClientCredentialFactory;
import com.microsoft.aad.msal4j.ClientCredentialParameters;
import com.microsoft.aad.msal4j.ConfidentialClientApplication;
import com.microsoft.aad.msal4j.IAuthenticationResult;
import com.microsoft.aad.msal4j.IClientCredential;
import com.microsoft.aad.msal4j.MsalException;
import com.microsoft.aad.msal4j.SilentParameters;
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Просто установите пакеты, запустив `npm install` в папке, в которой находится *package.js* файла. Затем импортируйте пакет **msal-node** :

```JavaScript 
const msal = require('@azure/msal-node');
```

# <a name="python"></a>[Python](#tab/python)

```python
import msal
import json
import sys
import logging
```

---

#### <a name="instantiate-the-confidential-client-application-with-a-client-secret"></a>Создание экземпляра конфиденциального клиентского приложения с секретом клиента

Вот код для создания экземпляра конфиденциального клиентского приложения с секретом клиента:

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
           .WithClientSecret(config.ClientSecret)
           .WithAuthority(new Uri(config.Authority))
           .Build();
```

`Authority`— Это объединение облачного экземпляра и идентификатора клиента, например `https://login.microsoftonline.com/contoso.onmicrosoft.com` или `https://login.microsoftonline.com/eb1ed152-0000-0000-0000-32401f3f9abd` . В файле *appsettings.js* , показанном в разделе [файл конфигурации](#configuration-file) , они представлены `Instance` `Tenant` значениями и соответственно.

В образце кода, взятом из предыдущего фрагмента, `Authority` является свойством класса  [аусентикатионконфиг](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/ffc4a9f5d9bdba5303e98a1af34232b434075ac7/1-Call-MSGraph/daemon-console/AuthenticationConfig.cs#L61-L70) и определяется следующим образом:

```csharp
/// <summary>
/// URL of the authority
/// </summary>
public string Authority
{
    get
    {
        return String.Format(CultureInfo.InvariantCulture, Instance, Tenant);
    }
}
```

# <a name="java"></a>[Java](#tab/java)

```Java
IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```JavaScript

const msalConfig = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: process.env.AAD_ENDPOINT + process.env.TENANT_ID,
        clientSecret: process.env.CLIENT_SECRET,
    }
};

const apiConfig = {
    uri: process.env.GRAPH_ENDPOINT + 'v1.0/users',
};

const tokenRequest = {
    scopes: [process.env.GRAPH_ENDPOINT + '.default'],
};

const cca = new msal.ConfidentialClientApplication(msalConfig);
```

# <a name="python"></a>[Python](#tab/python)

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential=config["secret"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

---

#### <a name="instantiate-the-confidential-client-application-with-a-client-certificate"></a>Создание экземпляра конфиденциального клиентского приложения с помощью сертификата клиента

Ниже приведен код для создания приложения с сертификатом.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
    .WithCertificate(certificate)
    .WithAuthority(new Uri(config.Authority))
    .Build();
```
# <a name="java"></a>[Java](#tab/java)

В MSAL Java для создания экземпляра конфиденциального клиентского приложения с сертификатами предусмотрено два строителя:

```Java

InputStream pkcs12Certificate = ... ; /* Containing PCKS12-formatted certificate*/
string certificatePassword = ... ;    /* Contains the password to access the certificate */

IClientCredential credential = ClientCredentialFactory.createFromCertificate(pkcs12Certificate, certificatePassword);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

или

```Java
PrivateKey key = getPrivateKey(); /* RSA private key to sign the assertion */
X509Certificate publicCertificate = getPublicCertificate(); /* x509 public certificate used as a thumbprint */

IClientCredential credential = ClientCredentialFactory.createFromCertificate(key, publicCertificate);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

В данный момент пример приложения не реализует инициализацию с сертификатами.

# <a name="python"></a>[Python](#tab/python)

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential={"thumbprint": config["thumbprint"], "private_key": open(config['private_key_file']).read()},
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

---

#### <a name="advanced-scenario-instantiate-the-confidential-client-application-with-client-assertions"></a>Расширенный сценарий: создание экземпляра конфиденциального клиентского приложения с утверждениями клиента

# <a name="net"></a>[.NET](#tab/dotnet)

Вместо секрета клиента или сертификата конфиденциальное клиентское приложение также может доказать свою личность с помощью утверждений клиента.

MSAL.NET имеет два метода для предоставления подписанных утверждений для конфиденциального клиентского приложения:

- `.WithClientAssertion()`
- `.WithClientClaims()`

При использовании `WithClientAssertion` Предоставьте подписанный JWT. Этот расширенный сценарий подробно описан в [проверочных утверждениях клиента](msal-net-client-assertions.md).

```csharp
string signedClientAssertion = ComputeAssertion();
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientAssertion(signedClientAssertion)
                                          .Build();
```

При использовании `WithClientClaims` MSAL.NET создает подписанное утверждение, которое содержит утверждения, ожидаемые Azure AD, а также дополнительные клиентские утверждения, которые требуется отправить.
В этом коде показано, как это сделать:

```csharp
string ipAddress = "192.168.1.2";
var claims = new Dictionary<string, string> { { "client_ip", ipAddress } };
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithAuthority(new Uri(config.Authority))
                                          .WithClientClaims(certificate, claims)
                                          .Build();
```

Дополнительные сведения см. в разделе [утверждения клиента](msal-net-client-assertions.md).

# <a name="java"></a>[Java](#tab/java)

```Java
IClientCredential credential = ClientCredentialFactory.createFromClientAssertion(assertion);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

В данный момент пример приложения не реализует инициализацию с утверждениями.

# <a name="python"></a>[Python](#tab/python)

В MSAL Python можно предоставить клиентские заявки с помощью утверждений, которые будут подписаны этим `ConfidentialClientApplication` закрытым ключом.

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential={"thumbprint": config["thumbprint"], "private_key": open(config['private_key_file']).read()},
    client_claims = {"client_ip": "x.x.x.x"}
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

Дополнительные сведения см. в справочной документации по Python MSAL для [конфидентиалклиентаппликатион](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.__init__).

---

## <a name="next-steps"></a>Дальнейшие действия

# <a name="net"></a>[.NET](#tab/dotnet)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для приложения](./scenario-daemon-acquire-token.md?tabs=dotnet).

# <a name="java"></a>[Java](#tab/java)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для приложения](./scenario-daemon-acquire-token.md?tabs=java).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для приложения](./scenario-daemon-acquire-token.md?tabs=nodejs).

# <a name="python"></a>[Python](#tab/python)

Перейдите к следующей статье в этом сценарии, чтобы [получить маркер для приложения](./scenario-daemon-acquire-token.md?tabs=python).

---
