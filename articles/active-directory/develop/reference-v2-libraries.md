---
title: Библиотеки проверки подлинности платформы Microsoft Identity
description: Совместимые клиентские библиотеки и библиотеки промежуточного слоя сервера, а также связанные библиотеки, источник и образцы ссылок для конечной точки платформы Microsoft Identity.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 07/25/2019
ms.author: negoe
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev
ms.openlocfilehash: a7792e989444c87ba80088f8a850cacb4aa0dc3c
ms.sourcegitcommit: 2488894b8ece49d493399d2ed7c98d29b53a5599
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2021
ms.locfileid: "98064578"
---
# <a name="microsoft-identity-platform-authentication-libraries"></a>Библиотеки проверки подлинности платформы Microsoft Identity

[Конечная точка платформы Microsoft Identity](../azuread-dev/azure-ad-endpoint-comparison.md) поддерживает стандартные протоколы OAuth 2,0 и openid connect Connect 1,0. Библиотека проверки подлинности Microsoft (MSAL) предназначена для работы с конечной точкой платформы Microsoft Identity. Также можно использовать библиотеки с открытым исходным кодом, поддерживающие OAuth 2,0 и OpenID Connect Connect 1,0.

Рекомендуется использовать библиотеки, написанные экспертами по домену протокола, которые соответствуют методологии жизненного цикла разработки безопасности (SDL). Такие методологии включают [в себя ту, что следует корпорации Майкрософт][Microsoft-SDL]. При наличии кода для протоколов следует следовать такой методологии, как Microsoft SDL. Обратите особое внимание на вопросы безопасности в спецификациях стандартов для каждого протокола.

> [!NOTE]
> Вы ищете библиотеку проверки подлинности Azure Active Directory (ADAL)? Ознакомьтесь с [руководством по библиотеке ADAL](../azuread-dev/active-directory-authentication-libraries.md).

## <a name="types-of-libraries"></a>Типы библиотек

Конечная точка платформы Microsoft Identity работает с двумя типами библиотек:

* **Клиентские библиотеки**. собственные клиенты и серверы используют клиентские библиотеки для получения маркеров доступа для вызова ресурса, например Microsoft Graph.
* **Серверные библиотеки ПО промежуточного слоя**. Веб-приложения используют серверные библиотеки ПО промежуточного слоя для выполнения входа пользователей. Веб-API используют серверные библиотеки ПО промежуточного слоя для проверки маркеров, отправляемых собственными клиентами или другими серверами.

## <a name="library-support"></a>Поддержка библиотек

Для библиотек предусмотрено две категории поддержки:

* **Библиотеки, поддерживаемые корпорацией Майкрософт**. Корпорация Майкрософт предоставляет исправления для этих библиотек, к тому же она провела для них комплексную экспертизу жизненного цикла разработки защищенных приложений.
* **Совместимо**: Корпорация Майкрософт проверила эти библиотеки в основных сценариях и подтвердила, что они работают с конечной точкой платформы Microsoft Identity. Корпорация Майкрософт не предоставляет исправления для этих библиотек и не выполнила проверку этих библиотек. Проблемы и запросы возможностей следует отправлять в проекты с открытым кодом библиотеки.

Список библиотек, которые работают с конечной точкой платформы идентификации Майкрософт, см. в следующих разделах.

## <a name="microsoft-supported-client-libraries"></a>Клиентские библиотеки, поддерживаемые корпорацией Майкрософт

Используйте библиотеки проверки подлинности клиента, чтобы получить маркер для вызова защищенного веб-API.

| Платформа | Библиотека | Скачать | Исходный код | Образец | Справочник | Основная документация | Схема действий |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ![JavaScript](media/sample-v2-code/logo_js.png) | MSAL.js  | [NPM](https://www.npmjs.com/package/msal) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/README.md) |  [Одностраничное приложение](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) | [Ссылки](https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/) | [Основные документы](msal-overview.md)| [Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
![Angular](media/sample-v2-code/logo_angular.png) | MSALный угловой | [NPM](https://www.npmjs.com/package/@azure/msal-angular) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | [Угловой SPA](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) | [Ссылки](https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-angular/) | [Основные документы](msal-overview.md) | [Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
| ![.NET Framework](media/sample-v2-code/logo_NET.png) ![UWP](media/sample-v2-code/logo_windows.png) ![Xamarin](media/sample-v2-code/logo_xamarin.png) | MSAL.NET  |[NuGet](https://www.nuget.org/packages/Microsoft.Identity.Client) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | [Классическое приложение](/windows/apps/desktop/) | [MSAL.NET](/dotnet/api/microsoft.identity.client?view=azure-dotnet-preview&preserve-view=true) |[Основные документы](msal-overview.md) | [Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap)
| ![Значок .NET Core](media/sample-v2-code/logo_NETCore.png) | Веб-сайт Microsoft Identity  |[NuGet](https://www.nuget.org/packages/Microsoft.Identity.Web) |[GitHub](https://github.com/AzureAD/microsoft-identity-web) | [Примеры](https://aka.ms/ms-id-web/samples) | [Microsoft. Identity. Web](/dotnet/api/microsoft.identity.web?view=azure-dotnet-preview&preserve-view=true) |[Основные документы](https://aka.ms/ms-id-web/conceptual-doc) | [Схема действий](https://github.com/AzureAD/microsoft-identity-web/wiki#roadmap)
| ![Python](media/sample-v2-code/logo_python.png) | MSAL Python | [PyPI](https://pypi.org/project/msal) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python) | [Примеры](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) | [реадседокс](https://msal-python.rtfd.io/) | [Вики](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | [Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Roadmap)
| ![Java](media/sample-v2-code/logo_java.png) | MSAL Java | [Maven](https://search.maven.org/artifact/com.microsoft.azure/msal4j) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-java) | [Примеры](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) | [Ссылки](https://javadoc.io/doc/com.microsoft.azure/msal4j/latest/index.html) | [Вики](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki) | [Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki)
| iOS и macOS | MSAL iOS и macOS | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [приложение iOS](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc), [приложение macOS](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) | [Ссылки](https://azuread.github.io/microsoft-authentication-library-for-objc/index.html)  | [Основные документы](msal-overview.md) | |
|![Android и Java](media/sample-v2-code/logo_Android.png) | MSAL Android | [Центральный репозиторий](https://repo1.maven.org/maven2/com/microsoft/identity/client/msal/) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android) | [Приложение Android](quickstart-v2-android.md) | [Javadocs](https://javadoc.io/doc/com.microsoft.identity.client/msal) | [Основные документы](msal-overview.md) |[Схема действий](https://github.com/AzureAD/microsoft-authentication-library-for-android/wiki/Roadmap)

## <a name="microsoft-supported-server-middleware-libraries"></a>Серверные библиотеки ПО промежуточного слоя, поддерживаемые корпорацией Майкрософт

Используйте библиотеки по промежуточного слоя для защиты веб-приложений и веб-API. Веб-приложения или веб-API, написанные с помощью ASP.NET или ASP.NET Core, используют библиотеки по промежуточного слоя.

| Платформа | Библиотека | Скачать | Исходный код | Образец | Справочник
| --- | --- | --- | --- | --- | --- |
| ![.NET](media/sample-v2-code/logo_NET.png) ![.NET Core](media/sample-v2-code/logo_NETcore.png) | Безопасность ASP.NET |[NuGet](https://www.nuget.org/packages/Microsoft.AspNet.Mvc/) |[GitHub](https://github.com/aspnet/AspNetCore) |[Приложение MVC](quickstart-v2-aspnet-webapp.md) |[Справочник по API для ASP.NET](/dotnet/api/?view=aspnetcore-2.0&preserve-view=true) |
| ![.NET](media/sample-v2-code/logo_NET.png)| Расширения модели идентификации для .NET| |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | [Приложение MVC](quickstart-v2-aspnet-webapp.md) |[Ссылки](/dotnet/api/overview/azure/activedirectory/client?view=azure-dotnet&preserve-view=true) |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | Azure AD Passport |[NPM](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [Веб-приложение](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs) | |

## <a name="microsoft-supported-libraries-by-os--language"></a>Библиотеки, поддерживаемые Майкрософт, по ОС и языку

В терминах поддерживаемых операционных систем и языков используется следующее сопоставление:

| Платформа    | Windows    | Linux      | macOS      | iOS | Android    |
|-------------|------------|------------|------------|------------|------------|
| ![JavaScript](media/sample-v2-code/logo_js.png)  |  MSAL.js | MSAL.js | MSAL.js | MSAL.js |  MSAL.js |
| <img alt="C#" src="../../cognitive-services/speech-service/media/index/logo_csharp.svg" width="64px" height="64px" /> | ASP.NET, ASP.NET Core, MSAL.Net (.NET FW, Core, UWP)| ASP.NET Core MSAL.Net (.NET Core) | ASP.NET Core, MSAL.Net (macOS)       | MSAL.Net (Xamarin. iOS) | MSAL.Net (Xamarin. Android)|
| Swift <br> Objective-C |            |            | [MSAL для iOS и MacOS](msal-overview.md) | [MSAL для iOS и MacOS](msal-overview.md) |            |
| ![Java](media/sample-v2-code/logo_java.png) Java | msal4j | msal4j | msal4j | | MSAL Android |
| ![Python](media/sample-v2-code/logo_python.png) Python | MSAL Python | MSAL Python | MSAL Python |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) Node.js | Паспорт. Node | Паспорт. Node | Паспорт. Node |

См. также [сценарии поддерживаемых платформ и языков](authentication-flows-app-scenarios.md#scenarios-and-supported-platforms-and-languages)

## <a name="compatible-client-libraries"></a>Совместимые клиентские библиотеки

| Платформа | Имя библиотеки | Проверенные версии | Исходный код | Образец |
|:---:|:---:|:---:|:---:|:---:|
|![JavaScript](media/sample-v2-code/logo_js.png)|[Hello.js](https://adodson.com/hello.js/) | Версия 1.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |[Безопасная проверка пароля](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) |
|![Vue](media/sample-v2-code/logo_vue.png)|[Vue MSAL](https://github.com/mvertopoulos/vue-msal) | Версия 3.0.3 |[Vue — msal](https://github.com/mvertopoulos/vue-msal) | |
| ![Java](media/sample-v2-code/logo_java.png) | [Scribe Java](https://github.com/scribejava/scribejava) | [Версия 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [ScribeJava](https://github.com/scribejava/scribejava/) | |
| ![Java](media/sample-v2-code/logo_java.png) | [Библиотека глуу OpenID Connect Connect](https://github.com/GluuFederation/oxAuth) | [Версия 3.0.2](https://github.com/GluuFederation/oxAuth/releases/tag/3.0.2) | [Библиотека глуу OpenID Connect Connect](https://github.com/GluuFederation/oxAuth) | |
| ![Python](media/sample-v2-code/logo_python.png) | [Запросы — Оауслиб](https://github.com/requests/requests-oauthlib) | [Версии 1.2.0](https://github.com/requests/requests-oauthlib/releases/tag/v1.2.0) | [Запросы — Оауслиб](https://github.com/requests/requests-oauthlib) | |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | [OpenID Connect — клиент](https://github.com/panva/node-openid-client) | [Версия 2.4.5](https://github.com/panva/node-openid-client/releases/tag/v2.4.5) | [OpenID Connect — клиент](https://github.com/panva/node-openid-client) | |
| ![PHP](media/sample-v2-code/logo_php.png) | [The PHP League (oauth2-client)](https://github.com/thephpleague/oauth2-client) | [Версия 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/) | |
| ![Ruby](media/sample-v2-code/logo_ruby.png) |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |омниаус: 1.3.1<br />омниаус-OAuth2:1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)<br />[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |  |
| iOS, macOS, & Android  | [Отреагировать на проверку подлинности собственного приложения](https://github.com/FormidableLabs/react-native-app-auth) | [Версия 4.2.0](https://github.com/FormidableLabs/react-native-app-auth/releases/tag/v4.2.0) | [Отреагировать на проверку подлинности собственного приложения](https://github.com/FormidableLabs/react-native-app-auth) | |

Для любой библиотеки, совместимой со стандартами, можно использовать конечную точку платформы Microsoft Identity. Важно иметь представление о том, где следует обращаться в службу поддержки:

* Проблемы и запросы на новые возможности в коде библиотеки отправляются владельцу библиотеки.
* Проблемы и запросы на новые возможности в реализации протокола на стороне службы отправляются в корпорацию Майкрософт.
* Отправьте [запрос функции](https://feedback.azure.com/forums/169401-azure-active-directory) для дополнительных функций, которые вы хотите увидеть в протоколе.
* [Создайте запрос в службу поддержки](../../azure-portal/supportability/how-to-create-azure-support-request.md) , если вы нашли ошибку, когда конечная точка платформы Microsoft Identity не соответствует требованиям OAuth 2,0 или openid connect Connect 1,0.

## <a name="related-content"></a>Связанное содержимое

Дополнительные сведения о конечной точке платформы идентификации Майкрософт см. в [статье Обзор платформы идентификации Майкрософт][AAD-App-Model-V2-Overview].

<!--Image references-->

<!--Reference style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[ClientLib-NET-Lib]: https://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-NET-Sample]: ./tutorial-v2-windows-desktop.md
[ClientLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ClientLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad
[ClientLib-Node-Sample]:/
[ClientLib-Iosmac-Lib]:/
[ClientLib-Iosmac-Repo]:/
[ClientLib-Iosmac-Sample]:/
[ClientLib-Android-Lib]:/
[ClientLib-Android-Repo]:/
[ClientLib-Android-Sample]:/
[ClientLib-Js-Lib]:/
[ClientLib-Js-Repo]:/
[ClientLib-Js-Sample]:/

[Microsoft-SDL]: https://www.microsoft.com/sdl/default.aspx
[ServerLib-Net4-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/
[ServerLib-Net4-Owin-Oidc-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oidc-Sample]: ./tutorial-v2-asp-webapp.md
[ServerLib-Net4-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth/
[ServerLib-Net4-Owin-Oauth-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oauth-Sample]: https://azure.microsoft.com/documentation/articles/active-directory-v2-devquickstarts-dotnet-api/
[ServerLib-Net-Jwt-Lib]: https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt
[ServerLib-Net-Jwt-Repo]: https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
[ServerLib-Net-Jwt-Sample]:/
[ServerLib-NetCore-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OpenIdConnect/
[ServerLib-NetCore-Owin-Oidc-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oidc-Sample]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore-v2
[ServerLib-NetCore-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OAuth/
[ServerLib-NetCore-Owin-Oauth-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oauth-Sample]:/
[ServerLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ServerLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad/
[ServerLib-Node-Sample]: https://azure.microsoft.com/documentation/articles/active-directory-v2-devquickstarts-node-web/
