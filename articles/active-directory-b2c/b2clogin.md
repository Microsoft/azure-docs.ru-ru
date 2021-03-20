---
title: Перенос приложений и API в b2clogin.com
titleSuffix: Azure AD B2C
description: Сведения об использовании b2clogin.com в качестве URL-адреса перенаправления для Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/27/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 20df5fc3a4d7c392be62df2b7778854d1e2e1cba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97109068"
---
# <a name="set-redirect-urls-to-b2clogincom-for-azure-active-directory-b2c"></a>Установка b2clogin.com в качестве URL-адреса перенаправления для Azure Active Directory B2C

При настройке поставщика удостоверений для регистрации и входа в приложение Azure Active Directory B2C (Azure AD B2C) необходимо указать URL-адрес перенаправления. Больше не следует ссылаться на *Login.microsoftonline.com* в приложениях и API для проверки подлинности пользователей с помощью Azure AD B2C. Вместо этого используйте *b2clogin.com* для всех новых приложений и перенесите существующие приложения из *Login.microsoftonline.com* в *b2clogin.com*.

## <a name="deprecation-of-loginmicrosoftonlinecom"></a>Нерекомендуемость login.microsoftonline.com

**Обновление за октябрь 2020 г.:** Мы расширяем льготный период для клиентов, которые не соответствуют первоначально объявленной дате устаревания (04 декабря 2020). Прекращение использования login.microsoftonline.com будет выполнено не раньше **14 января 2021.**

**Background**: на 04 декабря 2019 мы первоначально [объявили](https://azure.microsoft.com/updates/b2c-deprecate-msol/) о запланированном прекращении поддержки Login.microsoftonline.com в Azure AD B2C 4 декабря 2020. В этом случае существующие клиенты предоставили один (1) год для миграции на b2clogin.com. Новые клиенты, созданные после 04 декабря 2019, не будут принимать запросы от login.microsoftonline.com. Все функциональные возможности остаются неизменными в конечной точке b2clogin.com.

Нерекомендуемое значение login.microsoftonline.com не влияет на Azure Active Directory клиентов. Это изменение затрагивает только Azure Active Directory B2C клиентов.

## <a name="what-endpoints-does-this-apply-to"></a>Какие конечные точки применяются к
Переход на b2clogin.com применяется только к конечным точкам проверки подлинности, которые используют политики Azure AD B2C (потоки пользователей или пользовательские политики) для проверки подлинности пользователей. Эти конечные точки имеют `<policy-name>` параметр, указывающий политику, которую Azure AD B2C следует использовать. Дополнительные [сведения о политиках Azure AD B2C](technical-overview.md#identity-experiences-user-flows-or-custom-policies). 

Эти конечные точки могут выглядеть следующим образом:
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/<b>\<policy-name\></b>/oauth2/v2.0/authorize</code>

- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/<b>\<policy-name\></b>/oauth2/v2.0/token</code>

Кроме того, `<policy-name>` может передаваться как параметр запроса:
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/oauth2/v2.0/authorize?<b>p=\<policy-name\></b></code>
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/oauth2/v2.0/token?<b>p=\<policy-name\></b></code>

> [!IMPORTANT]
> Необходимо обновить конечные точки, использующие параметр "Policy", а также [URL-адреса перенаправления поставщика удостоверений](#change-identity-provider-redirect-urls).

Некоторые Azure AD B2C клиенты используют общие возможности корпоративных клиентов Azure AD, такие как учетные данные клиента OAuth 2,0. Доступ к этим функциям осуществляется с помощью конечных точек login.microsoftonline.com Azure AD, *которые не содержат параметр политики*. __Эти конечные точки не затрагиваются__.

## <a name="benefits-of-b2clogincom"></a>Преимущества b2clogin.com

При использовании *b2clogin.com* в качестве URL-адреса перенаправления:

* Пространство, используемое службами Майкрософт в заголовке файлов cookie, уменьшается.
* URL-адреса перенаправления больше не должны включать ссылку на Майкрософт.
* Код на стороне клиента JavaScript поддерживается (в настоящее время находится на [этапе предварительной версии](javascript-and-page-layout.md)) на настраиваемых страницах. Из-за ограничений безопасности код JavaScript и HTML-элементы формы удаляются из пользовательских страниц, если используется *Login.microsoftonline.com*.

## <a name="overview-of-required-changes"></a>Общие сведения о необходимых изменениях

Существует несколько изменений, которые может потребоваться выполнить для переноса приложений в *b2clogin.com*:

* Измените URL-адрес перенаправления в приложениях поставщика удостоверений, чтобы он ссылался на *b2clogin.com*.
* Обновите приложения Azure AD B2C, чтобы использовать *b2clogin.com* в своих потоках пользователей и ссылках на конечные точки маркеров. Это может быть обновление использования библиотеки проверки подлинности, такой как библиотека проверки подлинности Майкрософт (MSAL).
* Обновите все **Разрешенные источники** , определенные в параметрах CORS для [настройки пользовательского интерфейса](customize-ui-with-html.md).

Старая конечная точка может выглядеть следующим образом:
- <b><code>https://login.microsoft.com/</b>\<tenant-name\>.onmicrosoft.com/\<policy-name\>/oauth2/v2.0/authorize</code>

Соответствующая обновленная конечная точка будет выглядеть следующим образом:
- <code><b>https://\<tenant-name\>.b2clogin.com/</b>\<tenant-name\>.onmicrosoft.com/\<policy-name\>/oauth2/v2.0/authorize</code>


## <a name="change-identity-provider-redirect-urls"></a>Изменение URL-адресов перенаправления поставщика удостоверений

На каждом веб-сайте поставщика удостоверений, в котором вы создали приложение, измените все доверенные URL-адреса, чтобы они перенаправлялись на `your-tenant-name.b2clogin.com` *Login.microsoftonline.com*.

Для URL-адресов перенаправления b2clogin.com можно использовать два формата. Первый предоставляет преимущества отсутствия "Майкрософт" в URL-адресе, используя идентификатор клиента (GUID) вместо имени домена клиента:

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-id}/oauth2/authresp
```

Во втором варианте используется доменное имя клиента в формате `your-tenant-name.onmicrosoft.com` . Пример:

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp
```

Для обоих форматов:

* Замените `{your-tenant-name}` именем вашего клиента Azure AD B2C.
* Удалите `/te` , если он существует в URL-адресе.

## <a name="update-your-applications-and-apis"></a>Обновление приложений и API-интерфейсов

Код в приложениях с поддержкой Azure AD B2C и API-интерфейсах может ссылаться `login.microsoftonline.com` в нескольких местах. Например, код может иметь ссылки на потоки пользователей и конечные точки маркеров. Вместо этого обновите следующие ссылки `your-tenant-name.b2clogin.com` :

* Конечная точка авторизации
* Конечная точка маркера
* Поставщик токена

Например, конечная точка центра для политики регистрации и входа в Contoso теперь будет выглядеть так:

```
https://contosob2c.b2clogin.com/00000000-0000-0000-0000-000000000000/B2C_1_signupsignin1
```

Сведения о миграции веб-приложений на основе OWIN в b2clogin.com см. в статье [Миграция веб-API на основе OWIN в b2clogin.com](multiple-token-endpoints.md).

Сведения о переносе API-интерфейсов службы управления API Azure, защищенных Azure AD B2C, см. в разделе " [Миграция в b2clogin.com](secure-api-management.md#migrate-to-b2clogincom) " статьи [Защита API управления API azure с помощью Azure AD B2C](secure-api-management.md).

## <a name="microsoft-authentication-library-msal"></a>Библиотека проверки подлинности Майкрософт (MSAL)

### <a name="msalnet-validateauthority-property"></a>MSAL.NET Валидатеаусорити, свойство

Если вы используете [MSAL.NET][msal-dotnet] v2 или более раннюю версию, установите свойство **валидатеаусорити** в значение `false` On Client, чтобы разрешить перенаправление в *b2clogin.com*. Установка этого значения в `false` MSAL.NET v3 и более поздних версий не требуется.

```csharp
ConfidentialClientApplication client = new ConfidentialClientApplication(...); // Can also be PublicClientApplication
client.ValidateAuthority = false; // MSAL.NET v2 and earlier **ONLY**
```

### <a name="msal-for-javascript-validateauthority-property"></a>MSAL для свойства Валидатеаусорити JavaScript

Если вы используете [MSAL для JavaScript][msal-js] v 1.2.2 или более ранней версии, задайте для свойства **валидатеаусорити** значение `false` .

```JavaScript
// MSAL.js v1.2.2 and earlier
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: false // Required in MSAL.js v1.2.2 and earlier **ONLY**
  }
);
```

Если задано значение `validateAuthority: true` в MSAL.js 1.3.0 + (значение по умолчанию), необходимо также указать допустимый издатель маркера с помощью `knownAuthorities` :

```JavaScript
// MSAL.js v1.3.0+
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: true, // Supported in MSAL.js v1.3.0+
    knownAuthorities: ['tenant-name.b2clogin.com'] // Required if validateAuthority: true
  }
);
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о миграции веб-приложений на основе OWIN в b2clogin.com см. в статье [Миграция веб-API на основе OWIN в b2clogin.com](multiple-token-endpoints.md).

Сведения о переносе API-интерфейсов службы управления API Azure, защищенных Azure AD B2C, см. в разделе " [Миграция в b2clogin.com](secure-api-management.md#migrate-to-b2clogincom) " статьи [Защита API управления API azure с помощью Azure AD B2C](secure-api-management.md).

<!-- LINKS - External -->
[msal-dotnet]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[msal-dotnet-b2c]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics
[msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
[msal-js-b2c]: ../active-directory/develop/msal-b2c-overview.md
