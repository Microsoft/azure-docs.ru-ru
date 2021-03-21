---
title: Azure AD B2C и MSAL.NET
titleSuffix: Microsoft identity platform
description: Рекомендации по использованию Azure AD B2C с библиотекой проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2020
ms.author: jeferrie
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: bdb9e12fdf721204ce98d23e5d5aeea535ddf23d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100574797"
---
# <a name="use-msalnet-to-sign-in-users-with-social-identities"></a>Использование MSAL.NET для входа пользователей с удостоверениями социальных сетей

MSAL.NET можно использовать для входа пользователей с удостоверениями социальных сетей с помощью [Azure Active Directory B2C (Azure AD B2C)](../../active-directory-b2c/overview.md). Azure AD B2C построены на основе понятия политик. В MSAL.NET указание политики преобразуется в предоставление центра.

- При создании экземпляра общедоступного клиентского приложения укажите политику в качестве части центра.
- Если необходимо применить политику, вызовите переопределение метода `AcquireTokenInteractive` , принимающего `authority` параметр.

Эта статья относится к MSAL.NET 3. x. Сведения о MSAL.NET 2. x см. в разделе [особенности Azure AD B2C в MSAL 2. x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-Specifics-MSAL-2.x) на вики-сайте MSAL.NET на GitHub.

## <a name="authority-for-an-azure-ad-b2c-tenant-and-policy"></a>Центр для клиента Azure AD B2C и политики

Формат центра для Azure AD B2C: `https://{azureADB2CHostname}/tfp/{tenant}/{policyName}`

- `azureADB2CHostname` — Имя клиента Azure AD B2C и узла. Например, *contosob2c.b2clogin.com*.
- `tenant` — Имя домена или идентификатор каталога (клиента) Azure AD B2C клиента. Например, *contosob2c.onmicrosoft.com* или GUID соответственно.
- `policyName` — Имя пользовательского потока или настраиваемой политики для применения. Например, политика регистрации и входа, например *b2c_1_susi*.

Дополнительные сведения о Azure AD B2Cных центрах см. [в разделе Set Redirect URL to b2clogin.com](../../active-directory-b2c/b2clogin.md).

## <a name="instantiating-the-application"></a>Создание экземпляра приложения

Предоставьте центр, вызвав `WithB2CAuthority()` при создании объекта приложения:

```csharp
// Azure AD B2C Coordinates
public static string Tenant = "fabrikamb2c.onmicrosoft.com";
public static string AzureADB2CHostname = "fabrikamb2c.b2clogin.com";
public static string ClientID = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
public static string PolicySignUpSignIn = "b2c_1_susi";
public static string PolicyEditProfile = "b2c_1_edit_profile";
public static string PolicyResetPassword = "b2c_1_reset";

public static string AuthorityBase = $"https://{AzureADB2CHostname}/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
public static string AuthorityEditProfile = $"{AuthorityBase}{PolicyEditProfile}";
public static string AuthorityPasswordReset = $"{AuthorityBase}{PolicyResetPassword}";

application = PublicClientApplicationBuilder.Create(ClientID)
               .WithB2CAuthority(Authority)
               .Build();
```

## <a name="acquire-a-token-to-apply-a-policy"></a>Получение маркера для применения политики

Получение маркера для защищенного Azure AD B2C API в общедоступном клиентском приложении требует использования переопределений с центром сертификации:

```csharp
AuthenticationResult authResult = null;
IEnumerable<IAccount> accounts = await application.GetAccountsAsync(policy);
IAccount account = accounts.FirstOrDefault();
try
{
    authResult = await application.AcquireTokenSilent(scopes, account)
                      .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    authResult = await application.AcquireTokenInteractive(scopes)
                        .WithAccount(account)
                        .WithParentActivityOrWindow(ParentActivityOrWindow)
                        .ExecuteAsync();
}  
```

В предыдущем фрагменте кода:

- `policy` — Это строка, содержащая имя пользовательского потока Azure AD B2C или настраиваемой политики (например, `PolicySignUpSignIn` ).
- `ParentActivityOrWindow` требуется для Android (действие) и является необязательным для других платформ, поддерживающих родительский пользовательский интерфейс, такой как Windows в Microsoft Windows и UIViewController в iOS. Дополнительные сведения о диалоговом окне пользовательского интерфейса см. в разделе [виспарентактивитйорвиндов](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Acquiring-tokens-interactively#withparentactivityorwindow) на вики-сайте MSAL.

Применение пользовательского потока или настраиваемой политики (например, предоставление пользователю возможности изменять свой профиль или сброс пароля) в настоящее время выполняется путем вызова `AcquireTokenInteractive` . Для этих двух политик не используется возвращаемый результат маркера или проверки подлинности.

## <a name="profile-edit-policies"></a>Политики изменения профиля

Чтобы пользователи могли входить с помощью социальных удостоверений, а затем изменять их профиль, примените политику изменения профиля Azure AD B2C.

Это можно сделать, вызвав `AcquireTokenInteractive` с полномочиями для этой политики. Так как пользователь уже выполнил вход и имеет активный сеанс cookie, используйте, `Prompt.NoPrompt` чтобы запретить отображение диалогового окна выбора учетной записи.

```csharp
private async void EditProfileButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await application.GetAccountsAsync(PolicyEditProfile);
    IAccount account = accounts.FirstOrDefault();
    try
    {
        var authResult = await application.AcquireTokenInteractive(scopes)
                            .WithPrompt(Prompt.NoPrompt),
                            .WithAccount(account)
                            .WithB2CAuthority(AuthorityEditProfile)
                            .ExecuteAsync();
     }
    catch
    {
    }
}
```

## <a name="resource-owner-password-credentials-ropc"></a>Учетные данные владельца ресурса (РОПК)

Дополнительные сведения о потоке РОПК см. [в разделе Вход с использованием пароля владельца ресурса предоставление учетных данных](v2-oauth-ropc.md).

**Не рекомендуется использовать** поток ропк, так как запрос пользователя к паролю в приложении не является безопасным. Дополнительные сведения об этой проблеме см. в статье [что такое решение для растущей проблемы паролей?](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).

Используя имя пользователя и пароль в потоке РОПК, вы пожертвают несколько вещей:

- Основные принципы современного удостоверения. пароль можно заменять или воспроизводить, так как можно перехватывать общий секрет. По определению РОПК несовместим с потоками без пароля.
- Пользователи, которым требуется выполнить MFA, не смогут войти в систему (так как взаимодействие отсутствует).
- Пользователи не смогут использовать единый вход (SSO).

### <a name="configure-the-ropc-flow-in-azure-ad-b2c"></a>Настройка потока РОПК в Azure AD B2C

В клиенте Azure AD B2C создайте новый поток пользователя и выберите **Вход с помощью ропк** , чтобы включить ропк для потока пользователя. Дополнительные сведения см. [в статье Настройка потока учетных данных для пароля владельца ресурса](../../active-directory-b2c/add-ropc-policy.md).

`IPublicClientApplication` содержит `AcquireTokenByUsernamePassword` метод:

```csharp
AcquireTokenByUsernamePassword(
            IEnumerable<string> scopes,
            string username,
            SecureString password)
```

Этот `AcquireTokenByUsernamePassword` метод принимает следующие параметры:

- *Области* , для которых необходимо получить маркер доступа.
- *Имя пользователя*.
- *Пароль* SecureString для пользователя.

### <a name="limitations-of-the-ropc-flow"></a>Ограничения потока РОПК

Поток РОПК **работает только для локальных учетных записей**, где пользователи зарегистрировались в Azure AD B2C с помощью адреса электронной почты или имени пользователя. Этот поток не работает при создании Федерации для внешнего поставщика удостоверений, поддерживаемого Azure AD B2C (Facebook, Google и т. д.).

## <a name="google-auth-and-embedded-webview"></a>Google auth и Embedded WebView

Если вы используете Google в качестве поставщика удостоверений, мы рекомендуем использовать системный браузер, так как Google не разрешает [проверку подлинности из внедренных](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html)веб – представлений. В настоящее время `login.microsoftonline.com` является доверенным центром Google и будет работать с внедренными WebView. Но не `b2clogin.com` является доверенным центром Google, поэтому пользователи не смогут пройти проверку подлинности.

Мы предоставим обновление этой [проблемы](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/688) в случае изменения.

## <a name="token-caching-in-msalnet"></a>Кэширование маркеров в MSAL.NET

### <a name="known-issue-with-azure-ad-b2c"></a>Известная ошибка Azure AD B2C

MSAL.NET поддерживает [кэш маркеров](/dotnet/api/microsoft.identity.client.tokencache). Ключ кэширования маркера основан на утверждениях, возвращенных поставщиком удостоверений (IdP).

В настоящее время MSAL.NET требуется два утверждения для создания ключа кэша маркера:

- `tid` (идентификатор клиента Azure AD)
- `preferred_username`

В Azure AD B2C сценарии могут отсутствовать оба этих утверждения, так как не все поставщики удостоверений социальных сетей (Facebook, Google и другие) возвращают их в токены, которые они возвращают в Azure AD B2C.

Симптомом такого сценария является то, что MSAL.NET возвращается `Missing from the token response` при доступе к `preferred_username` значению утверждения в маркерах, выдаваемых Azure AD B2C. MSAL использует `Missing from the token response` значение для `preferred_username` для поддержки кросс-совместимости кэша между библиотеками.

### <a name="workarounds"></a>Методы обхода проблемы

#### <a name="mitigation-for-missing-tenant-id"></a>Устранение рисков для отсутствующего идентификатора клиента

Предлагаемый обходной путь заключается в использовании [кэширования согласно политике,](#acquire-a-token-to-apply-a-policy) описанной выше.

Кроме того, можно использовать `tid` утверждение, если вы используете [пользовательские политики](../../active-directory-b2c/custom-policy-get-started.md) в Azure AD B2C. Пользовательские политики могут возвращать дополнительные утверждения приложению с помощью [преобразования утверждений](../../active-directory-b2c/claims-transformation-technical-profile.md).

#### <a name="mitigation-for-missing-from-the-token-response"></a>Устранение рисков "отсутствует в ответе маркера"

Один из вариантов — использовать `name` утверждение вместо `preferred_username` . Чтобы включить `name` утверждение в токены идентификации, выданные Azure AD B2C, выберите **Отображаемое имя** при настройке потока пользователя.

Дополнительные сведения об указании утверждений, возвращаемых потоками пользователей, см. [в разделе Учебник. Создание пользовательских потоков в Azure AD B2C](../../active-directory-b2c/tutorial-create-user-flows.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о получении маркеров в интерактивном режиме с помощью MSAL.NET для Azure AD B2C приложений приведены в следующем примере.

| Образец | Платформа | Описание|
|------ | -------- | -----------|
|[Active-Directory-B2C-Xamarin-Native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Xamarin iOS, Xamarin Android, UWP | Приложение Xamarin Forms, использующее MSAL.NET для проверки подлинности пользователей с помощью Azure AD B2C и последующего доступа к веб-API с возвращаемыми маркерами.|
