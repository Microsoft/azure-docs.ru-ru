---
title: Учетные данные сертификата платформы удостоверений Майкрософт
titleSuffix: Microsoft identity platform
description: В этой статье рассматривается регистрация и использование учетных данных сертификата для проверки подлинности приложения.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/3/2020
ms.author: hirsin
ms.reviewer: nacanuma, jmprieur
ms.custom: aaddev
ms.openlocfilehash: cfbcc8523ff1d5858317a3654b58ec7b2d23607a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99582032"
---
# <a name="microsoft-identity-platform-application-authentication-certificate-credentials"></a>Учетные данные сертификата проверки подлинности приложения платформы удостоверений Майкрософт

Платформа Microsoft Identity позволяет приложению использовать собственные учетные данные для проверки подлинности в любом месте, где можно использовать секрет клиента, например в потоке  [предоставления учетных данных клиента](v2-oauth2-client-creds-grant-flow.md) OAuth 2,0 и потоке " [от имени](v2-oauth2-on-behalf-of-flow.md) " (OBO).

Одним из форм учетных данных, которые приложение может использовать для проверки подлинности, является утверждение [JSON Web Token](./security-tokens.md#json-web-tokens-and-claims) (JWT), подписанное с помощью сертификата, которому принадлежит приложение.

## <a name="assertion-format"></a>Формат утверждения

Чтобы вычислить утверждение, можно использовать одну из многих библиотек JWT на выбранном языке — [MSAL поддерживает это с помощью `.WithCertificate()` ](msal-net-client-assertions.md). Эта информация переносится маркером в [заголовке](#header), [утверждениях](#claims-payload)и [сигнатуре](#signature).

### <a name="header"></a>Заголовок

| Параметр |  Комментарий |
| --- | --- |
| `alg` | Должен иметь значение **RS256** |
| `typ` | Должен иметь значение **JWT** |
| `x5t` | Хэш сертификата X. 509 (также известный как *отпечаток* SHA-1) кодируется в шестнадцатеричное представление в виде строкового значения Base64url. Например, если получен хэш сертификата X. 509 `84E05C1D98BCE3A5421D225B140B36E86A3D5534` (HEX), `x5t` утверждение будет равно `hOBcHZi846VCHSJbFAs26Go9VTQ=` (Base64url). |

### <a name="claims-payload"></a>Утверждения (полезные данные)

Тип утверждения | Значение | Описание
---------- | ---------- | ----------
aud | `https://login.microsoftonline.com/{tenantId}/v2.0` | Утверждение "AUD" (аудитория) определяет получателей, для которых предназначено JWT (здесь Azure AD), см. [RFC 7519, раздел 4.1.3](https://tools.ietf.org/html/rfc7519#section-4.1.3).  В этом случае получатель — это сервер входа (login.microsoftonline.com).
exp | 1601519414 | Утверждение "exp" (время окончания срока действия) указывает время окончания срока действия или время, после которого маркер JWT НЕ ДОЛЖЕН приниматься в обработку. См. статью [RFC 7519 в разделе 4.1.4](https://tools.ietf.org/html/rfc7519#section-4.1.4).  Это позволяет использовать утверждение до тех пор, пока оно не пойдет в короткий (5-10 мин `nbf` .).  Azure AD не помещает ограничения на `exp` Текущее время. 
iss | ClientID | Утверждение "ISS" (издатель) определяет участника, который выдал JWT, в данном случае клиентское приложение.  Используйте идентификатор приложения GUID.
jti | (GUID) | Утверждение "ЖТИ" (идентификатор JWT) предоставляет уникальный идентификатор для JWT. Значение идентификатора должно быть назначено способом, гарантирующим незначительную вероятность того, что одно и то же значение будет случайно назначено другому объекту данных. Если приложение использует несколько издателей, то конфликты между значениями, создаваемыми разными издателями, должны быть предотвращены. Значение "ЖТИ" является строкой с учетом регистра. [RFC 7519, раздел 4.1.7](https://tools.ietf.org/html/rfc7519#section-4.1.7)
nbf | 1601519114 | Утверждение "nbf" (не ранее) определяет время, до которого маркер JWT НЕ ДОЛЖЕН приниматься в обработку. [RFC 7519, раздел 4.1.5](https://tools.ietf.org/html/rfc7519#section-4.1.5).  Использование текущего времени приемлемо. 
sub | ClientID | Утверждение "подсистема" (тема) определяет тему JWT, в данном случае — и приложение. Используйте то же значение, что и `iss` . 

### <a name="signature"></a>Подпись

Подпись вычисляются путем применения сертификата, как описано в [спецификации JSON Web Token RFC7519](https://tools.ietf.org/html/rfc7519).

## <a name="example-of-a-decoded-jwt-assertion"></a>Пример декодированного утверждения JWT

```JSON
{
  "alg": "RS256",
  "typ": "JWT",
  "x5t": "gx8tGysyjcRqKjFPnd7RFwvwZI0"
}
.
{
  "aud": "https: //login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token",
  "exp": 1484593341,
  "iss": "97e0a5b7-d745-40b6-94fe-5f77d35c6e05",
  "jti": "22b3bb26-e046-42df-9c96-65dbd72c1c81",
  "nbf": 1484592741,
  "sub": "97e0a5b7-d745-40b6-94fe-5f77d35c6e05"
}
.
"Gh95kHCOEGq5E_ArMBbDXhwKR577scxYaoJ1P{a lot of characters here}KKJDEg"
```

## <a name="example-of-an-encoded-jwt-assertion"></a>Пример закодированного утверждения JWT

Ниже приводится строка, которая является примером закодированного утверждения. При внимательном рассмотрении Обратите внимание на три раздела, разделенные точками ( `.` ):

* Первый раздел кодирует *заголовок*
* Второй раздел кодирует *утверждения* (полезные данные)
* Последний раздел — это *подпись* , вычисленная с использованием сертификатов из содержимого первых двух разделов.

```
"eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJhdWQiOiJodHRwczpcL1wvbG9naW4ubWljcm9zb2Z0b25saW5lLmNvbVwvam1wcmlldXJob3RtYWlsLm9ubWljcm9zb2Z0LmNvbVwvb2F1dGgyXC90b2tlbiIsImV4cCI6MTQ4NDU5MzM0MSwiaXNzIjoiOTdlMGE1YjctZDc0NS00MGI2LTk0ZmUtNWY3N2QzNWM2ZTA1IiwianRpIjoiMjJiM2JiMjYtZTA0Ni00MmRmLTljOTYtNjVkYmQ3MmMxYzgxIiwibmJmIjoxNDg0NTkyNzQxLCJzdWIiOiI5N2UwYTViNy1kNzQ1LTQwYjYtOTRmZS01Zjc3ZDM1YzZlMDUifQ.
Gh95kHCOEGq5E_ArMBbDXhwKR577scxYaoJ1P{a lot of characters here}KKJDEg"
```

## <a name="register-your-certificate-with-microsoft-identity-platform"></a>Зарегистрируйте сертификат на платформе удостоверений Майкрософт

Учетные данные сертификата можно связать с клиентским приложением на платформе Microsoft Identity с помощью портал Azure одним из следующих способов.

### <a name="uploading-the-certificate-file"></a>Передача файла сертификата

При регистрации приложения Azure для клиентского приложения сделайте следующее.
1. Выберите **Сертификаты и секреты**.
2. Нажмите **Отправить сертификат** и выберите файл сертификата для отправки.
3. Нажмите кнопку **Добавить**.
  После передачи сертификата отображаются значения отпечатка сертификата, даты начала и истечения срока действия.

### <a name="updating-the-application-manifest"></a>Обновление манифеста приложения

Выполнив получение сертификата, вычислите следующие значения:

- `$base64Thumbprint` — Значение хэша сертификата в кодировке Base64
- `$base64Value` — Значение необработанных данных сертификата в кодировке Base64

Укажите идентификатор GUID для задания ключа в манифесте приложения ( `$keyId` ).

При регистрации приложения Azure для клиентского приложения сделайте следующее.
1. Щелкните **Манифест**, чтобы открыть манифест приложения.
2. Замените свойство *keyCredentials* данными нового сертификата, используя приведенную ниже схему.

   ```JSON
   "keyCredentials": [
       {
           "customKeyIdentifier": "$base64Thumbprint",
           "keyId": "$keyid",
           "type": "AsymmetricX509Cert",
           "usage": "Verify",
           "value":  "$base64Value"
       }
   ]
   ```
3. Сохраните изменения в манифесте приложения и передайте его на платформу удостоверений Майкрософт.

   Свойство `keyCredentials` является многозначным, поэтому для расширенного управления ключами можно передать несколько сертификатов.
   
## <a name="using-a-client-assertion"></a>Использование утверждения клиента

Утверждения клиента можно использовать в любом месте, где будет использоваться секрет клиента.  Например, в [потоке кода авторизации](v2-oauth2-auth-code-flow.md)можно передать в, `client_secret` чтобы доказать, что запрос поступает из приложения. Это можно заменить на `client_assertion` Параметры и `client_assertion_type` . 

| Параметр | Значение | Описание|
|-----------|-------|------------|
|`client_assertion_type`|`urn:ietf:params:oauth:client-assertion-type:jwt-bearer`| Это фиксированное значение, указывающее, что используются учетные данные сертификата. |
|`client_assertion`| JWT |Это созданное выше JWT. |

## <a name="next-steps"></a>Дальнейшие действия

[Библиотека MSAL.NET обрабатывает этот сценарий](msal-net-client-assertions.md) в одной строке кода.

[Консольное приложение управляющей программы .NET Core, использующее пример кода платформы идентификаторов Microsoft Identity](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) на GitHub, показывает, как приложение использует собственные учетные данные для проверки подлинности. Здесь также показано, как [создать самозаверяющий сертификат](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/tree/master/1-Call-MSGraph#optional-use-the-automation-script) с помощью `New-SelfSignedCertificate` командлета PowerShell. Вы также можете использовать [скрипты создания приложений](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/master/1-Call-MSGraph/AppCreationScripts-withCert/AppCreationScripts.md) в примере репозитория для создания сертификатов, вычислить отпечаток и т. д.
