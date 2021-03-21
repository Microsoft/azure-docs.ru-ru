---
title: Переход на MSAL.NET
titleSuffix: Microsoft identity platform
description: Сведения о различиях между библиотекой проверки подлинности Microsoft для .NET (MSAL.NET) и библиотекой аутентификация Azure AD для .NET (ADAL.NET) и о переходе на MSAL.NET.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/10/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 64107c3f667dd7e59fcf6d191e83457029b3a277
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100546352"
---
# <a name="migrating-applications-to-msalnet"></a>Перенос приложений на MSAL.NET

Как библиотека проверки подлинности Microsoft для .NET (MSAL.NET), так и библиотека аутентификация Azure AD для .NET (ADAL.NET) используются для проверки подлинности сущностей Azure AD и маркеров запросов из Azure AD. До настоящего времени большинство разработчиков использовали для аутентификации удостоверений Azure AD (рабочих и учебных учетных записей) платформу Azure AD для разработчиков (версия 1.0), запрашивая маркеры через библиотеку аутентификации Azure AD (ADAL). Использование MSAL:

- Вы можете проверить подлинность более широкого набора удостоверений Майкрософт (удостоверений Azure AD и учетных записей Майкрософт, а также социальных и локальных учетных записей с помощью Azure AD B2C), так как в нем используется платформа Microsoft Identity.
- пользователи получат лучшие возможности единого входа.
- приложение может включить добавочное согласие, и поддержка условного доступа упрощается
- преимущества инноваций.

**Теперь MSAL.NET является рекомендуемой библиотекой проверки подлинности для использования с платформой Microsoft Identity**. Новые функции не будут реализованы в ADAL.NET. Усилия посвящены улучшению MSAL.

В этой статье описываются различия между библиотекой проверки подлинности Microsoft для .NET (MSAL.NET) и библиотекой аутентификация Azure AD для .NET (ADAL.NET) и помогает переходить на MSAL.

## <a name="differences-between-adal-and-msal-apps"></a>Различия между приложениями ADAL и MSAL

В большинстве случаев необходимо использовать MSAL.NET и платформу Microsoft Identity, которая является последним поколением библиотек проверки подлинности Майкрософт. Используя MSAL.NET, вы получаете маркеры для пользователей, входящих в приложение с учетной записью Azure AD (рабочей или учебной), учетной записью Майкрософт (личной MSA) или через Azure AD B2C.

Если вы уже знакомы с конечной точкой Azure AD для разработчиков (v 1.0) (и ADAL.NET), вам может потребоваться ознакомиться с [другими сведениями о платформе Microsoft Identity](../azuread-dev/azure-ad-endpoint-comparison.md).

Но ADAL.NET пока остается обязательной для тех случаев, когда приложению нужно обрабатывать вход пользователей из более ранних версий [служб федерации Active Directory (ADFS)](/windows-server/identity/active-directory-federation-services). Дополнительные сведения см. в разделе [Поддержка ADFS](https://aka.ms/msal-net-adfs-support).

На следующем рисунке приведены некоторые различия между ADAL.NET и MSAL.NET: ![Сравнительный анализ кода](./media/msal-compare-msaldotnet-and-adaldotnet/differences.png)

### <a name="nuget-packages-and-namespaces"></a>Пакеты NuGet и пространства имен

Библиотека ADAL.NET предоставляется в составе пакета NuGet [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory). Для нее нужно использовать пространство имен `Microsoft.IdentityModel.Clients.ActiveDirectory`.

Чтобы использовать MSAL.NET, следует добавить пакет NuGet [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) и задействовать пространство имен `Microsoft.Identity.Client`.

### <a name="scopes-not-resources"></a>Области вместо ресурсов

ADAL.NET получает маркеры для *ресурсов*, а MSAL.NET — для *областей*. Для нескольких переопределений AcquireToken в MSAL.NET требуется параметр с именем scopes(`IEnumerable<string> scopes`). Этот параметр представляет собой простой список строк, которые объявляют нужные разрешения и запрашиваемые ресурсы. Например, часто используются [области Microsoft Graph](/graph/permissions-reference).

Из MSAL также можно получить доступ к ресурсам версии 1.0. Подробнее об этом см. в разделе [Области для веб-API, которые принимают маркеры версии 1.0](#scopes-for-a-web-api-accepting-v10-tokens).

### <a name="core-classes"></a>Основные классы

- ADAL.NET использует [AuthenticationContext](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/AuthenticationContext:-the-connection-to-Azure-AD) для представления подключения к службе токенов безопасности (STS) или серверу авторизации с помощью объекта Authority. Библиотека MSAL.NET, в свою очередь, основана на концепции [клиентских приложений](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Client-Applications). Она предоставляет два отдельных класса: `PublicClientApplication` и `ConfidentialClientApplication`.

- Получение маркеров: ADAL.NET и MSAL.NET имеют одинаковые вызовы проверки подлинности ( `AcquireTokenAsync` и `AcquireTokenSilentAsync` для ADAL.NET и `AcquireTokenInteractive` и `AcquireTokenSilent` в MSAL.NET), но с различными параметрами. В числе отличий важно отметить, что в MSAL.NET уже не нужно передавать в приложение `ClientID` при каждом вызове AcquireTokenXX. Теперь `ClientID` задается только один раз при создании `IPublicClientApplication` или `IConfidentialClientApplication`.

### <a name="iaccount-not-iuser"></a>IAccount вместо IUser

ADAL.NET управляет пользователями. Пользователем считается конкретный человек или агент программного обеспечения, но любому из них могут принадлежать одна или несколько учетных записей в системе идентификации Майкрософт (несколько учетных записей Azure AD, Azure AD B2C, личных учетных записей Майкрософт).

В MSAL.NET 2.x определена концепция учетной записи (через интерфейс IAccount). Это важное изменение предоставляет правильную семантику, то есть отражает возможность использования одним пользователем нескольких учетных записей в разных каталогах AAD. Также MSAL.NET предоставляет более подробные сведения в сценариях гостевого входа, в частности сведения о домашней учетной записи.

Дополнительные сведения о различиях между IUser и IAccount см. в описании [MSAL.NET 2.x](https://aka.ms/msal-net-2-released).

### <a name="exceptions"></a>Исключения

#### <a name="interaction-required-exceptions"></a>Взаимодействия требовали исключений

MSAL.NET более явно использует исключения. Например, при сбое автоматической аутентификации в ADAL процедура должна перехватывать исключение и выполнять поиск кода ошибки `user_interaction_required`:

```csharp
catch(AdalException exception)
{
 if (exception.ErrorCode == "user_interaction_required")
 {
  try
  {“try to authenticate interactively”}}
 }
}
```

Подробнее см. в описании [рекомендуемого метода получения маркера](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/AcquireTokenSilentAsync-using-a-cached-token#recommended-pattern-to-acquire-a-token) для ADAL.NET.

При работе с MSAL.NET следует перехватывать исключение `MsalUiRequiredException`, как описано в разделе [AcquireTokenSilent](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AcquireTokenSilentAsync-using-a-cached-token).

```csharp
catch(MsalUiRequiredException exception)
{
 try {“try to authenticate interactively”}
}
```

#### <a name="handling-claim-challenge-exceptions"></a>Обработка исключений по запросу утверждений

В ADAL.NET исключения по запросу утверждений обрабатываются следующим образом:

- Исключение `AdalClaimChallengeException`, производное от `AdalServiceException`, генерируется службой в том случае, если ресурсу нужны дополнительные утверждения от пользователя (например, двухфакторная аутентификация). Член `Claims` содержит некоторый фрагмент JSON с ожидаемыми утверждениями.
- Работающее с ADAL.NET общедоступное клиентское приложение, которое получит такое исключение, должно вызывать переопределение `AcquireTokenInteractive` с параметром утверждения. Это переопределение `AcquireTokenInteractive` никогда не обращается к кэшу, так как в этом нет необходимости. Дело в том, что маркер в кэше наверняка не имеет правильных утверждений (иначе не возникло бы исключение `AdalClaimChallengeException`). Это означает, что проверять кэш не нужно. Обратите внимание, что `ClaimChallengeException` можно получить в WebAPI, выполнив OBO, тогда как в `AcquireTokenInteractive` общедоступном клиентском приложении, вызывающем этот веб-API.
- Дополнительные сведения и примеры вы найдете в разделе [об обработке исключения AdalClaimChallengeException](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Exceptions-in-ADAL.NET#handling-adalclaimchallengeexception).

В MSAL.NET исключения по запросу утверждений обрабатываются следующим образом:

- Запросы `Claims` создаются в исключении `MsalServiceException`.
- Существует метод `.WithClaim(claims)`, который можно применить к построителю `AcquireTokenInteractive`.

### <a name="supported-grants"></a>Поддерживаемые предоставляемые разрешения

MSAL.NET и конечная точка версии 2.0 пока поддерживают не все предоставляемые разрешения. Ниже приведена сводка со сравнением предоставляемых разрешений, которые поддерживаются в ADAL.NET и MSAL.NET.

#### <a name="public-client-applications"></a>Общедоступные клиентские приложения

Следующие предоставляемые разрешения поддерживаются в ADAL.NET и MSAL.NET для классических и мобильных приложений.

Предоставить | ADAL.NET | MSAL.NET
----- |----- | -----
Интерактивно | [Интерактивная аутентификация](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Acquiring-tokens-interactively---Public-client-application-flows) | [Получение маркеров в интерактивном режиме через MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Acquiring-tokens-interactively)
Встроенная проверка подлинности Windows | [Встроенная аутентификация Windows (Kerberos)](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/AcquireTokenSilentAsync-using-Integrated-authentication-on-Windows-(Kerberos)) | [Встроенная аутентификация Windows](msal-authentication-flows.md#integrated-windows-authentication)
Имя пользователя и пароль | [Получение маркеров с использованием имени пользователя и пароля](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Acquiring-tokens-with-username-and-password)| [Аутентификация по имени пользователя и паролю](msal-authentication-flows.md#usernamepassword)
Поток кода устройства | [Профиль устройства для устройств без веб-браузеров](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Device-profile-for-devices-without-web-browsers) | [Поток кода устройства](msal-authentication-flows.md#device-code)

#### <a name="confidential-client-applications"></a>Конфиденциальные клиентские приложения

Ниже приведены права, поддерживаемые в ADAL.NET и MSAL.NET для веб-приложений, веб-API и управляющих приложений.

Тип приложения | Предоставить | ADAL.NET | MSAL.NET
----- | ----- | ----- | -----
Веб-приложение, веб-API, управляющая программа | Учетные данные клиента | [Потоки учетных данных клиента в ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Client-credential-flows) | [Потоки учетных данных клиента в MSAL.NET](msal-authentication-flows.md#client-credentials)
Веб-интерфейс API | От имени | [Вызовы между службами от имени пользователя через ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Service-to-service-calls-on-behalf-of-the-user) | [Вызов от имени через MSAL.NET](msal-authentication-flows.md#on-behalf-of)
Веб-приложение | Код аутентификации | [Получение маркеров с помощью кодов авторизации в веб-приложениях через ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Acquiring-tokens-with-authorization-codes-on-web-apps) | [Получение маркеров с помощью кодов авторизации в веб-приложениях через MSAL.NET](msal-authentication-flows.md#authorization-code)

### <a name="cache-persistence"></a>Сохраняемость кэша

ADAL.NET позволяет расширить класс `TokenCache`, чтобы реализовать нужные функции сохраняемости на платформах без защищенного хранилища (.NET Framework и .NET core) с помощью методов `BeforeAccess` и `BeforeWrite`. Дополнительные сведения см. в разделе [о сериализации кэша маркеров в ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization).

MSAL.NET предоставляет кэш маркеров в закрытом классе, то есть не позволяет расширить его. Это означает, что сохраняемость кэша маркеров следует реализовать в виде вспомогательного класса, который взаимодействует с закрытым кэшем маркеров. Такое взаимодействие описано в разделе [о сериализации кэша маркеров MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization).

## <a name="signification-of-the-common-authority"></a>Обозначение единого центра

Если вы используете центр `https://login.microsoftonline.com/common` в версии 1.0, пользователи могут входить с любой учетной записью AAD (любой организации). Подробнее см. в статье [о проверке в ADAL.NET](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/AuthenticationContext:-the-connection-to-Azure-AD#authority-validation).

Если вы используете центр `https://login.microsoftonline.com/common` в версии 2.0, пользователи смогут входить с любой корпоративной (AAD) или личной учетной записью Майкрософт (MSA). В MSAL.NET, если требуется ограничить вход в учетную запись AAD (то же поведение, что и в ADAL.NET), используйте `https://login.microsoftonline.com/organizations` . Дополнительные сведения см. в описании параметра `authority` в статье об [общедоступном клиентском приложении](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Client-Applications#publicclientapplication).

## <a name="v10-and-v20-tokens"></a>Маркеры версии 1.0 и версии 2.0

Существуют две версии маркеров:
- маркеры версии 1.0;
- маркеры версии 2.0.

Конечная точка версии 1.0 (используется в ADAL) выдает только маркеры версии 1.0.

Однако конечная точка версии 2.0 (используемая MSAL) выдает версию токена, принимаемого веб-API. Свойство манифеста приложения веб-API позволяет разработчикам выбрать версию токена, которая принимается. См. описание `accessTokenAcceptedVersion` в справочной документации [о манифесте приложения](reference-app-manifest.md).

Дополнительные сведения о маркерах версий 1.0 и 2.0 см. в статье [Microsoft identity platform access tokens](access-tokens.md) (Маркеры доступа к платформе удостоверений Майкрософт).

## <a name="scopes-for-a-web-api-accepting-v10-tokens"></a>Области для веб-API, принимающие токены версии 1.0

Разрешения OAuth2 задаются для областей разрешений, которые веб-API (ресурс) версии 1.0 предоставляет клиентским приложениям. Эти области действия разрешений могут быть назначены клиентским приложениям во время предоставления согласия. См. раздел об oauth2Permissions в документации по [манифесту приложения Azure Active Directory](./reference-app-manifest.md).

### <a name="scopes-to-request-access-to-specific-oauth2-permissions-of-a-v10-application"></a>Области для запроса доступа к конкретным разрешениям OAuth2 в приложении версии 1.0

Если вы хотите получить токены для приложения, принимающего токены версии 1.0 (например, Microsoft Graph API, то https://graph.microsoft.com) , что вам нужно создать, `scopes` путем сцепления требуемого идентификатора ресурса с нужным разрешением OAuth2 для этого ресурса.

Например, чтобы получить доступ в имени пользователя веб-API версии 1.0, для которого используется URI идентификатора приложения `ResourceId` , необходимо использовать:

```csharp
var scopes = new [] { ResourceId+"/user_impersonation" };
```

Если вы хотите считывать и записывать с помощью MSAL.NET Azure Active Directory с помощью Microsoft Graph API ( https://graph.microsoft.com/) , создайте список областей, например, в следующем фрагменте кода:

```csharp
string ResourceId = "https://graph.microsoft.com/"; 
string[] scopes = { ResourceId + "Directory.Read", ResourceId + "Directory.Write" }
```

#### <a name="warning-should-you-have-one-or-two-slashes-in-the-scope-corresponding-to-a-v10-web-api"></a>Предупреждение. должна ли быть одна или две косые черты в области, соответствующей веб-API версии 1.0

Если необходимо записать область, соответствующую Azure Resource Manager API ( https://management.core.windows.net/) , запросите следующую область (Обратите внимание на две косые черты).

```csharp
var scopes = new[] {"https://management.core.windows.net//user_impersonation"};
var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();

// then call the API: https://management.azure.com/subscriptions?api-version=2016-09-01
```

Это связано с тем, что API Resource Manager ожидает получить косую черту в утверждении audience (`aud`), а вторая косая черта отделяет имя API от области действия.

Azure AD использует следующую логику:
- Для конечной точки ADAL (версия 1.0) с маркером доступа версии 1.0 (единственный поддерживаемый вариант) aud=resource.
- Для конечной точки MSAL (версии 2.0), которая запрашивает маркер доступа для ресурса, принимающего маркеры версии 2.0, aud=resource.AppId.
- Для конечной точки MSAL (версии 2.0), которая запрашивает маркер доступа для ресурса, принимающего маркеры версии 1.0 (как в примере выше), AAD выполняет синтаксический анализ параметра audience из запрошенной области, используя все символы до последней косой черты в качестве идентификатора ресурса. Поэтому если https:\//database.windows.net ожидает для audience значение "https://database.windows.net/", следует запросить область https:\//database.windows.net//.default. См. также проблема No[747](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/747): Конечная косая черта URL-адреса ресурса пропущена, что привело к сбою проверки подлинности SQL #747


### <a name="scopes-to-request-access-to-all-the-permissions-of-a-v10-application"></a>Области для запроса доступа ко всем разрешениям в приложении версии 1.0

Например, если вы хотите получить маркер для всех статических областей приложения версии 1.0, необходимо использовать

```csharp
ResourceId = "someAppIDURI";
var scopes = new [] { ResourceId+"/.default" };
```

### <a name="scopes-to-request-in-the-case-of-client-credential-flow--daemon-app"></a>Области для запроса при использовании клиентского потока учетных данных или управляющей программы

Если используется клиентский поток учетных данных, следует передать область `/.default`. Эта область сообщает Azure AD: "все разрешения уровня приложения, которые администратор предоставил в регистрации приложения.

## <a name="adal-to-msal-migration"></a>Переход с ADAL на MSAL

В ADAL.NET версии 2.X стали доступны маркеры обновления, что позволяет создавать решения на основе таких маркеров, кэшируя их и с помощью методов `AcquireTokenByRefreshToken`, предоставляемых в ADAL 2.x.
Такие решения удобны для следующих сценариев:
* Долго выполняющиеся службы, которые выполняют действия (в том числе обновление панелей мониторинга) от имени пользователя, когда пользователь уже закрыл подключение.
* Предоставление клиенту возможности применить RT для веб-службы в сценарии WebFarm (кэширование выполняется на стороне клиента с использованием зашифрованного файла cookie, а не на стороне сервера).

MSAL.NET не предоставляет маркеры обновления в целях безопасности: MSAL обрабатывает маркеры обновления.

К счастью, у MSAL.NET теперь есть API, позволяющий перенести предыдущие маркеры обновления (полученные с помощью ADAL) в `IConfidentialClientApplication` :

```csharp
/// <summary>
/// Acquires an access token from an existing refresh token and stores it and the refresh token into
/// the application user token cache, where it will be available for further AcquireTokenSilent calls.
/// This method can be used in migration to MSAL from ADAL v2 and in various integration
/// scenarios where you have a RefreshToken available.
/// (see https://aka.ms/msal-net-migration-adal2-msal2)
/// </summary>
/// <param name="scopes">Scope to request from the token endpoint.
/// Setting this to null or empty will request an access token, refresh token and ID token with default scopes</param>
/// <param name="refreshToken">The refresh token from ADAL 2.x</param>
IByRefreshToken.AcquireTokenByRefreshToken(IEnumerable<string> scopes, string refreshToken);
```

Этот метод позволяет передавать ранее использованный маркер обновления вместе с любым нужными областями (ресурсами). Маркер обновления будет заменен на новый и повторно кэширован в приложении.

Так как этот метод предназначен для нетипичного сценария, его нельзя использовать для `IConfidentialClientApplication` без предварительного преобразования в `IByRefreshToken`.

Этот фрагмент кода демонстрирует перенос конфиденциального клиентского приложения. `GetCachedRefreshTokenForSignedInUser` получает маркер обновления, сохраненный в хранилище предыдущей версией приложения, которое использовало ADAL 2.x. `GetTokenCacheForSignedInUser` выполняет десериализацию кэша для пользователя, выполнившего вход (в конфиденциальном клиентском приложении обычно создается отдельный кэш для каждого пользователя).

```csharp
TokenCache userCache = GetTokenCacheForSignedInUser();
string rt = GetCachedRefreshTokenForSignedInUser();

IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.Create(clientId)
 .WithAuthority(Authority)
 .WithRedirectUri(RedirectUri)
 .WithClientSecret(ClientSecret)
 .Build();
IByRefreshToken appRt = app as IByRefreshToken;

AuthenticationResult result = await appRt.AcquireTokenByRefreshToken(null, rt)
                                         .ExecuteAsync()
                                         .ConfigureAwait(false);
```

Вы увидите, что в AuthenticationResult возвращаются маркер доступа и маркер идентификатора, а новый маркер обновления сохраняется в кэше.

Этот метод можно использовать в разных процессах интеграции, если у вас есть готовый маркер обновления.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об областях в [области, разрешениях и согласии можно найти на платформе Microsoft Identity](v2-permissions-and-consent.md) .
