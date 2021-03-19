---
title: Утверждения клиента (MSAL.NET) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о поддержке подписанных утверждений клиентов для конфиденциальных клиентских приложений в библиотеке проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/18/2021
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 8fb4ecb8fa8d6938e9afbc77064380b7b213029a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578742"
---
# <a name="confidential-client-assertions"></a>Конфиденциальные утверждения клиентов

Чтобы доказать свою личность, конфиденциальные клиентские приложения обмениваются секретом с Azure AD. Секрет может быть следующим:
- Секрет клиента (пароль приложения).
- Сертификат, который используется для создания подписанного утверждения, содержащего стандартные утверждения.

Этот секрет также может быть подписанным утверждением напрямую.

MSAL.NET состоит из четырех методов предоставления учетных данных или утверждений для конфиденциального клиентского приложения:
- `.WithClientSecret()`
- `.WithCertificate()`
- `.WithClientAssertion()`
- `.WithClientClaims()`

> [!NOTE]
> Хотя можно использовать `WithClientAssertion()` API для получения маркеров для конфиденциального клиента, мы не рекомендуем использовать его по умолчанию, так как он более сложный и предназначен для работы с очень специфичными сценариями, которые не являются общими. Использование `.WithCertificate()` API позволит MSAL.NET справиться с этим. Этот API предлагает возможность настроить запрос проверки подлинности, если это необходимо, но утверждение по умолчанию, созданное, `.WithCertificate()` будет достаточно для большинства сценариев проверки подлинности. Этот API также можно использовать в качестве обходного решения в некоторых случаях, когда MSAL.NET не может выполнить операцию подписывания внутренним образом.

### <a name="signed-assertions"></a>Подписанные утверждения

Подписанное утверждение клиента принимает форму подписанного JWT с полезной нагрузкой, содержащей необходимые утверждения проверки подлинности, заданные в Azure AD, в кодировке Base64. Чтобы ее использовать, сделайте следующее:

```csharp
string signedClientAssertion = ComputeAssertion();
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientAssertion(signedClientAssertion)
                                          .Build();
```

Можно также использовать форму делегата, которая позволяет вычислять утверждение только вовремя:

```csharp
string signedClientAssertion = ComputeAssertion();
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientAssertion(() => { return GetSignedClientAssertion(); } )
                                          .Build();
```

[Утверждения, ожидаемые Azure AD](active-directory-certificate-credentials.md) в подписанном утверждении:

Тип утверждения | Значение | Описание
---------- | ---------- | ----------
aud | `https://login.microsoftonline.com/{tenantId}/v2.0` | Утверждение "AUD" (аудитория) определяет получателей, для которых предназначено JWT (здесь Azure AD), см. [RFC 7519, раздел 4.1.3](https://tools.ietf.org/html/rfc7519#section-4.1.3).  В этом случае получатель — это сервер входа (login.microsoftonline.com).
exp | 1601519414 | Утверждение "exp" (время окончания срока действия) указывает время окончания срока действия или время, после которого маркер JWT НЕ ДОЛЖЕН приниматься в обработку. См. статью [RFC 7519 в разделе 4.1.4](https://tools.ietf.org/html/rfc7519#section-4.1.4).  Это позволяет использовать утверждение до тех пор, пока оно не пойдет в короткий (5-10 мин `nbf` .).  Azure AD не помещает ограничения на `exp` Текущее время. 
iss | ClientID | Утверждение "ISS" (издатель) определяет участника, который выдал JWT, в данном случае клиентское приложение.  Используйте идентификатор приложения GUID.
jti | (GUID) | Утверждение "ЖТИ" (идентификатор JWT) предоставляет уникальный идентификатор для JWT. Значение идентификатора должно быть назначено способом, гарантирующим незначительную вероятность того, что одно и то же значение будет случайно назначено другому объекту данных. Если приложение использует несколько издателей, то конфликты между значениями, создаваемыми разными издателями, должны быть предотвращены. Значение "ЖТИ" является строкой с учетом регистра. [RFC 7519, раздел 4.1.7](https://tools.ietf.org/html/rfc7519#section-4.1.7)
nbf | 1601519114 | Утверждение "nbf" (не ранее) определяет время, до которого маркер JWT НЕ ДОЛЖЕН приниматься в обработку. [RFC 7519, раздел 4.1.5](https://tools.ietf.org/html/rfc7519#section-4.1.5).  Использование текущего времени приемлемо. 
sub | ClientID | Утверждение "подсистема" (тема) определяет тему JWT, в данном случае — и приложение. Используйте то же значение, что и `iss` . 

Ниже приведен пример создания этих утверждений.

```csharp
private static IDictionary<string, string> GetClaims()
{
      //aud = https://login.microsoftonline.com/ + Tenant ID + /v2.0
      string aud = $"https://login.microsoftonline.com/{tenantId}/v2.0";

      string ConfidentialClientID = "00000000-0000-0000-0000-000000000000" //client id
      const uint JwtToAadLifetimeInSeconds = 60 * 10; // Ten minutes
      DateTime validFrom = DateTime.UtcNow;
      var nbf = ConvertToTimeT(validFrom);
      var exp = ConvertToTimeT(validFrom + TimeSpan.FromSeconds(JwtToAadLifetimeInSeconds));

      return new Dictionary<string, string>()
           {
                { "aud", aud },
                { "exp", exp.ToString() },
                { "iss", ConfidentialClientID },
                { "jti", Guid.NewGuid().ToString() },
                { "nbf", nbf.ToString() },
                { "sub", ConfidentialClientID }
            };
}
```

Вот как можно создавать подписанное утверждение клиента:

```csharp
string Encode(byte[] arg)
{
    char Base64PadCharacter = '=';
    char Base64Character62 = '+';
    char Base64Character63 = '/';
    char Base64UrlCharacter62 = '-';
    char Base64UrlCharacter63 = '_';

    string s = Convert.ToBase64String(arg);
    s = s.Split(Base64PadCharacter)[0]; // RemoveAccount any trailing padding
    s = s.Replace(Base64Character62, Base64UrlCharacter62); // 62nd char of encoding
    s = s.Replace(Base64Character63, Base64UrlCharacter63); // 63rd char of encoding

    return s;
}

string GetSignedClientAssertion()
{
    //Signing with SHA-256
    string rsaSha256Signature = "http://www.w3.org/2001/04/xmldsig-more#rsa-sha256";
     X509Certificate2 certificate = new X509Certificate2("Certificate.pfx", "Password", X509KeyStorageFlags.EphemeralKeySet);

    //Create RSACryptoServiceProvider
    var x509Key = new X509AsymmetricSecurityKey(certificate);
    var privateKeyXmlParams = certificate.PrivateKey.ToXmlString(true);
    var rsa = new RSACryptoServiceProvider();
    rsa.FromXmlString(privateKeyXmlParams);

    //alg represents the desired signing algorithm, which is SHA-256 in this case
    //kid represents the certificate thumbprint
    var header = new Dictionary<string, string>()
         {
              { "alg", "RS256"},
              { "kid", Encode(Certificate.GetCertHash()) }
         };

    //Please see the previous code snippet on how to craft claims for the GetClaims() method
    string token = Encode(Encoding.UTF8.GetBytes(JObject.FromObject(header).ToString())) + "." + Encode(Encoding.UTF8.GetBytes(JObject.FromObject(GetClaims())));

    string signature = Encode(rsa.SignData(Encoding.UTF8.GetBytes(token), new SHA256Cng()));
    string signedClientAssertion = string.Concat(token, ".", signature);
    return signedClientAssertion;
}
```

### <a name="alternative-method"></a>Альтернативный метод

Вы также можете использовать [Microsoft.IdentityModel.Jsонвебтокенс](https://www.nuget.org/packages/Microsoft.IdentityModel.JsonWebTokens/) , чтобы создать утверждение. Код будет более элегантным, как показано в следующем примере:

```csharp
        string GetSignedClientAssertion()
        {
            var cert = new X509Certificate2("Certificate.pfx", "Password", X509KeyStorageFlags.EphemeralKeySet);

            //aud = https://login.microsoftonline.com/ + Tenant ID + /v2.0
            string aud = $"https://login.microsoftonline.com/{tenantID}/v2.0";

            // client_id
            string confidentialClientID = "00000000-0000-0000-0000-000000000000";

            // no need to add exp, nbf as JsonWebTokenHandler will add them by default.
            var claims = new Dictionary<string, object>()
            {
                { "aud", aud },
                { "iss", confidentialClientID },
                { "jti", Guid.NewGuid().ToString() },
                { "sub", confidentialClientID }
            };

            var securityTokenDescriptor = new SecurityTokenDescriptor
            {
                Claims = claims,
                SigningCredentials = new X509SigningCredentials(cert)
            };

            var handler = new JsonWebTokenHandler();
            var signedClientAssertion = handler.CreateToken(securityTokenDescriptor);
        }
```

После подписанного утверждения клиента его можно использовать с API-интерфейсами MSAL, как показано ниже.

```csharp
            string signedClientAssertion = GetSignedClientAssertion();

            var confidentialApp = ConfidentialClientApplicationBuilder
                .Create(ConfidentialClientID)
                .WithClientAssertion(signedClientAssertion)
                .Build();
```

### <a name="withclientclaims"></a>висклиентклаимс

`WithClientClaims(X509Certificate2 certificate, IDictionary<string, string> claimsToSign, bool mergeWithDefaultClaims = true)` по умолчанию будет создаваться подписанное утверждение, содержащее утверждения, ожидаемые Azure AD, и дополнительные клиентские утверждения, которые требуется отправить. Ниже приведен фрагмент кода для этого.

```csharp
string ipAddress = "192.168.1.2";
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithAuthority(new Uri(config.Authority))
                                          .WithClientClaims(certificate, 
                                                                      new Dictionary<string, string> { { "client_ip", ipAddress } })
                                          .Build();

```

Если одно из утверждений в словаре, которое вы передали, совпадает с одним из обязательных утверждений, учитывается значение дополнительного утверждения. Он переопределит утверждения, вычисленные с помощью MSAL.NET.

Если вы хотите предоставить собственные утверждения, включая обязательные утверждения, ожидаемые Azure AD, передайте `false` `mergeWithDefaultClaims` параметр.
