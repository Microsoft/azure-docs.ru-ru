---
title: Примеры кода для платформы удостоверений Майкрософт
description: Список доступных примеров кода для платформы удостоверений Майкрософт, упорядоченный по сценариям.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: sample
ms.workload: identity
ms.date: 11/04/2020
ms.author: marsma
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: a19ca14a37bd24e499fdf0681b3510e9a4a8ea3f
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106075038"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Примеры кода для платформы удостоверений Майкрософт (конечная точка версии 2.0)

Возможности платформы удостоверений Майкрософт:

- добавление возможностей аутентификации и авторизации в веб-приложения и веб-API;
- требование маркера доступа для доступа к защищенным веб-API.

В этой статье приведены краткое описание и ссылки на примеры для платформы удостоверений Майкрософт. В этих примерах показано, как работает эта платформа, и предложены фрагменты кода для использования в реальных приложениях. На странице примеров кода приведены ссылки на статьи с подробными сведениями о требованиях, установке и настройке. Код содержит комментарии, которые помогают понять наиболее важные разделы.

Чтобы понять основной сценарий для каждого типа примера, см. статью [Типы приложений для платформы удостоверений Майкрософт](v2-app-types.md).

Вы также можете поделиться своими примерами на сайте GitHub. Чтобы узнать, как это сделать, обратитесь к разделу [Примеры и документация Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Одностраничные приложения

В этих примерах показано, как создать одностраничное приложение, защищенное с помощью платформы удостоверений Майкрософт. В этих примерах используется одна из разновидностей MSAL.js.

| Платформа | Описание | Ссылка |
| -------- | --------------------- | -------- |
| ![На этом изображении показан логотип JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js).](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | SPA вызывает Microsoft Graph |[javascript-graphapi-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) |
| ![На изображении показан логотип JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA вызывает Microsoft Graph с помощью потока кода проверки подлинности с PKCE |[javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2) |
| ![На этом изображении показан логотип JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js).](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | SPA вызывает B2C |[b2c-javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) |
| ![На изображении показан логотип JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA вызывает B2C с помощью потока кода проверки подлинности с PKCE |[b2c-javascript-spa](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa) |
| ![На изображении показан логотип JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA вызывает пользовательский веб-API, который в свою очередь вызывает Microsoft Graph  | [ms-identity-javascript-tutorial-chapter4-obo](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/4-AdvancedGrants/1-call-api-graph) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA вызывает Microsoft Graph  | [active-directory-javascript-singlepageapp-angular](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA вызывает Microsoft Graph с помощью потока кода проверки подлинности с PKCE | [ms-identity-javascript-angular-spa](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA вызывает пользовательский веб-API | [ms-identity-javascript-angular-spa-aspnetcore-webapi](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA вызывает B2C |[active-directory-b2c-javascript-angular-spa](https://github.com/Azure-Samples/active-directory-b2c-javascript-angular-spa) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA вызывает пользовательский веб-API с ролями приложений и группами безопасности |[ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups) |
| ![На этом изображении показан логотип React](media/sample-v2-code/logo_react.png) [React (MSAL React)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)| SPA вызывает Microsoft Graph с помощью потока кода проверки подлинности с PKCE | [ms-identity-javascript-react-spa](https://github.com/Azure-Samples/ms-identity-javascript-react-spa) |
| ![На этом изображении показан логотип React](media/sample-v2-code/logo_react.png) [React (MSAL React)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)| SPA вызывает пользовательский веб-API | [ms-identity-javascript-react-tutorial](https://github.com/Azure-Samples/ms-identity-javascript-react-tutorial/tree/main/3-Authorization-II/1-call-api) |
| ![На этом изображении показан логотип React](media/sample-v2-code/logo_react.png) [React (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)| SPA вызывает пользовательский веб-API, который в свою очередь вызывает Microsoft Graph  | [ms-identity-javascript-react-spa-dotnetcore-webapi-obo](https://github.com/Azure-Samples/ms-identity-javascript-react-spa-dotnetcore-webapi-obo) |
| ![На этом изображение показан логотип Blazor](media/sample-v2-code/logo-blazor.png) [Blazor WebAssembly (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | Учебник по Blazor WebAssembly, в котором содержатся сведения о том, как реализовать вход пользователей и вызов API с помощью Azure Active Directory |[ms-identity-blazor-wasm](https://github.com/Azure-Samples/ms-identity-blazor-wasm) |

## <a name="web-applications"></a>Веб-приложения

В следующем примере показаны веб-приложения для входа пользователей. В некоторых примерах приложение вызывает Microsoft Graph или веб-API с использованием удостоверения пользователя.

| Платформа | Только вход пользователей | Выполняет вход пользователей и вызывает Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![На этом изображении показан логотип ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | [Руководство по входу пользователей в систему для веб-приложений ASP.NET Core](https://aka.ms/aspnetcore-webapp-sign-in) | Тот же пример на этапе [Веб-приложение ASP.NET Core вызывает Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph)</p>Расширенный пример [Доступ к кэшу маркеров выполнившего вход пользователя из фоновых приложений, интерфейсов API и служб](https://github.com/Azure-Samples/ms-identity-dotnet-advanced-token-cache) |
| ![На этом изображении показан логотип ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p>ASP.NET Core | [Сборник схем для разработчиков, посвященный миграции приложений из AD FS в Azure AD](https://github.com/Azure-Samples/ms-identity-dotnet-adfs-to-aad). Узнайте, как безопасно переносить приложения, интегрированные со службами федерации Active Directory (AD FS), в Azure Active Directory (Azure AD). | |
| ![На этом изображении показан логотип ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [Краткое руководство по ASP.NET](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> [msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp) |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  |[Учебник по Java Servlet. Глава 1.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Вход с помощью AAD| |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  |[Учебник по Java Servlet. Глава 1.2](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Вход с помощью B2C |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  | | [Учебник по Java Servlet. Глава 2.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Вход с помощью AAD и графа вызовов|
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  |[Учебник по Java Servlet. Глава 3.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Вход с помощью AAD и контроль доступа с помощью утверждения ролей| |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  | | [Учебник по Java Servlet. Глава 3.2](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Вход с помощью AAD и контроль доступа с помощью утверждения групп|
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | |[Учебник по Java Servlet. Глава 4.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Развертывание в Службе приложений Azure|
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  | | [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp) |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png)  | [ms-identity-b2c-java-servlet-webapp-authentication](https://github.com/Azure-Samples/ms-identity-b2c-java-servlet-webapp-authentication)|  |
| ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (MSAL Node) | [Учебник по быстрому входу в веб-приложения для пользователей](https://github.com/Azure-Samples/ms-identity-node) | |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | [Учебник по Python Flask. Глава 1.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Вход с помощью AAD  |  |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | [Учебник по Python Flask. Глава 1.2](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Вход с помощью B2C                    |  |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | | [Учебник по Python Flask. Глава 2.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Вход с помощью AAD и графа вызовов |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | |[Учебник по Python Flask. Глава 3.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Развертывание в Службе приложений Azure  |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | [Учебник по Python Django. Глава 1.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Вход с помощью AAD  | |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | [Учебник по Python Django. Глава 1.2](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Вход с помощью B2C                    |  |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | | [Учебник по Python Django. Глава 2.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Вход с помощью AAD и графа вызовов|
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | | [Учебник по Python Django. Глава 3.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Развертывание в Службе приложений Azure                    |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)  | | [Веб-приложения Python Flask](https://github.com/Azure-Samples/ms-identity-python-webapp) |
| ![На этом изображении показан логотип Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |
| ![На этом изображении показан логотип Blazor](media/sample-v2-code/logo-blazor.png)</p>Blazor Server | Учебник по реализации входа пользователей в [приложение Blazor Server](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-OIDC) | [Вызов Microsoft Graph приложением Blazor Server](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-graph-user/Call-MSGraph)</p>Учебник, разделенный на главы: [Предоставление приложению Blazor Server возможности реализовать вход пользователей и вызов API с помощью Azure Active Directory](https://github.com/Azure-Samples/ms-identity-blazor-server) |

## <a name="desktop-and-mobile-public-client-apps"></a>Общедоступные клиентские приложения — классические и мобильные

В приведенных ниже примерах показаны общедоступные клиентские приложения (классические или мобильные), которые обращаются к API Microsoft Graph или веб-API от имени пользователя. Кроме примера *классического (консольного) приложения с WAM*, все эти клиентские приложения используют библиотеку проверки подлинности Майкрософт (MSAL).

| Клиентское приложение | Платформа | Поток или способ предоставления | Вызывает Microsoft Graph | Вызывает веб-API ASP.NET Core |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| Руководство по классическим приложениям (.NET Core) — необязательное использование:</p>кросс-платформенного кэша маркеров;</p>пользовательского веб-интерфейса. | ![На этом изображении показан логотип .NET/C#](media/sample-v2-code/logo_NETcore.png) | [Код авторизации](msal-authentication-flows.md#authorization-code)| [ms-identity-dotnet-desktop-tutorial](https://github.com/azure-samples/ms-identity-dotnet-desktop-tutorial) | |
| Классическое приложение (WPF)      | ![На этом изображении показан логотип .NET/C# (для классического приложения)](media/sample-v2-code/logo_NET.png) | [Код авторизации](msal-authentication-flows.md#authorization-code)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| Классическое приложение (консольное)   | ![Изображение: логотип .NET/C# (для классического приложения)](media/sample-v2-code/logo_NET.png) | [Встроенная аутентификация Windows](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | [Встроенная аутентификация Windows](msal-authentication-flows.md#integrated-windows-authentication) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Классическое приложение (консольное)   | ![Логотип .NET/C# (для классического приложения)](media/sample-v2-code/logo_NETcore.png) | [Имя пользователя или пароль](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| Классическое (консольное) приложение с WAM  | ![Логотип .NET/C# (для классического приложения)](media/sample-v2-code/logo_NETcore.png) | Интерактивный с использованием [диспетчера учетных веб-записей](/windows/uwp/security/web-account-manager) (WAM) |[dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) |  |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | [Имя пользователя или пароль](msal-authentication-flows.md#usernamepassword) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png) | [Имя пользователя или пароль](msal-authentication-flows.md#usernamepassword) |[ms-identity-python-desktop](https://github.com/Azure-Samples/ms-identity-python-desktop) |  |
| Классическое (Electron)   | ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (MSAL Node) | [Код авторизации (PKCE)](msal-authentication-flows.md#authorization-code) |[ms-identity-javascript-nodejs-desktop](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-desktop) |  |
| Мобильное приложение (Android, iOS, UWP)   | ![На этом изображении показан логотип .NET/C# (для Xamarin)](media/sample-v2-code/logo_xamarin.png) | [Код авторизации](msal-authentication-flows.md#authorization-code) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| Мобильное приложение (iOS)       | ![На этом изображении показаны iOS/Objective-C или SWIFT](media/sample-v2-code/logo_iOS.png) | [Код авторизации](msal-authentication-flows.md#authorization-code) |[ios-swift-objc-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| Классическое приложение (для macOS)       | macOS | [Код авторизации](msal-authentication-flows.md#authorization-code) |[macOS-swift-objc-native-v2](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) |  |
| Мобильное приложение (для Android и Java)   | ![На этом изображении показан логотип Android](media/sample-v2-code/logo_Android.png) | [Код авторизации](msal-authentication-flows.md#authorization-code) |  [android-Java](https://github.com/Azure-Samples/ms-identity-android-java) |  |
| Мобильное приложение (для Android и Kotlin)   | ![На этом изображении показан логотип Android](media/sample-v2-code/logo_Android.png) | [Код авторизации](msal-authentication-flows.md#authorization-code) |  [android-Kotlin](https://github.com/Azure-Samples/ms-identity-android-kotlin) |  |

## <a name="daemon-applications"></a>Управляющие программы

В следующем примере показано приложение, которое обращается к Microsoft Graph с использованием собственного удостоверения (без пользователя).

| Клиентское приложение | Платформа | Поток или предоставление | Вызывает Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| Консоль | ![На этом изображении показан логотип .NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [Учетные данные клиента](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Веб-приложение | ![Снимок экрана: логотип ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [Учетные данные клиента](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |
| Консоль | ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | [Учетные данные клиента](msal-authentication-flows.md#client-credentials) | [ms-identity-java-daemon](https://github.com/Azure-Samples/ms-identity-java-daemon) |
| Консоль | ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (MSAL Node)| [Учетные данные клиента](msal-authentication-flows.md#client-credentials) | [ms-identity-javascript-nodejs-console](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-console) |
| Консоль | ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png) | [Учетные данные клиента](msal-authentication-flows.md#client-credentials) | [ms-identity-python-daemon](https://github.com/Azure-Samples/ms-identity-python-daemon) |

## <a name="headless-applications"></a>Приложения удаленного управления

В следующем примере показано общедоступное клиентское приложение, запущенное на устройстве без веб-браузера. Это приложение может быть программой командной строки или приложением для Linux или Mac либо приложением Интернета вещей. В этом примере представлено приложение, которое обращается к API Microsoft Graph от имени пользователя, который в интерактивном режиме выполняет вход на другом устройстве (например, на мобильном телефоне). Это клиентское приложение использует библиотеку проверки подлинности Майкрософт (MSAL).

| Клиентское приложение | Платформа | Поток или предоставление | Вызывает Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип .NET/C# (для классического приложения)](media/sample-v2-code/logo_NETcore.png) | [Поток кода устройства](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | [Поток кода устройства](msal-authentication-flows.md#device-code) |[ms-identity-java-devicecodeflow](https://github.com/Azure-Samples/ms-identity-java-devicecodeflow) |
| Классическое приложение (консольное)   | ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png) | [Поток кода устройства](msal-authentication-flows.md#device-code) |[ms-identity-python-devicecodeflow](https://github.com/Azure-Samples/ms-identity-python-devicecodeflow) |

## <a name="multi-tenant-saas-applications"></a>Мультитенантные приложения SaaS

В следующих примерах показано, как настроить приложение таким образом, чтобы оно поддерживало вход из любого клиента Azure AD (AD). Настройка приложения как *мультитенантного* означает, что вы можете предлагать **программное обеспечение как услугу** (SaaS) многим организациям, позволяя пользователям входить в приложение после предоставления согласия.

| Платформа | Описание | Ссылка |
| -------- | --------------------- | -------- |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | Мультитенантное одностраничное приложение вызывает API Graph |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter1) |
| ![На этом изображении показан логотип Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | Мультитенантное SPA вызывает мультитенантный пользовательский веб-API |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter2) |
| ![На этом изображении показан логотип ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | Веб-приложение ASP.NET Core MVC вызывает API Graph |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-3-Multi-Tenant) |
| ![На этом изображении показан логотип ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | Веб-приложение ASP.NET Core MVC вызывает веб-API ASP.NET Core |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-3-AnyOrg) |

## <a name="web-apis"></a>Веб-API

В следующих примерах показано, как защитить веб-API с помощью платформы удостоверений Майкрософт и как вызывать из веб-API подчиненный API.

| Платформа | Образец |
| -------- | ------------------- |
| ![На этом изображении показан логотип ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | Веб-API (служба) ASP.NET Core [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph)  |
| ![На этом изображении показан логотип ASP.NET](media/sample-v2-code/logo_NET.png)</p>ASP.NET MVC 3 | Веб-API (служба) для [ms-identity-aspnet-webapi-onbehalfof](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof) |
| ![На этом изображении показан логотип Java](media/sample-v2-code/logo_java.png) | Веб-API (служба) для [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi) |
| ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| Веб-API (служба) для [active-directory-javascript-nodejs-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2) |
| ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| B2C Веб-API (служба) для [active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) |

## <a name="azure-functions-as-web-apis"></a>Функции Azure в качестве интерфейсов веб-API

В следующих примерах показано, как защитить функцию Azure с помощью HttpTrigger и веб-API с помощью платформы удостоверений Майкрософт и как вызывать из веб-API подчиненный API.

| Платформа | Образец |
| -------- | ------------------- |
| ![На этом изображении показан логотип ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | Веб-API (служба) ASP.NET Core Функций Azure для [dotnet-native-aspnetcore-v2](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)  |
| ![На этом изображении показан логотип Python](media/sample-v2-code/logo_python.png)</p>Python | Веб-API (служба) для [Python](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions) |
| ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| Веб-API (служба) для [Node.js и passport-azure-ad](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions) |
| ![На этом изображении показан логотип Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| Веб-API (служба) для [Node.js и passport-azure-ad с использованием от имени пользователя](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions) |

## <a name="other-microsoft-graph-samples"></a>Другие примеры Microsoft Graph

Дополнительные сведения о [примерах](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) и учебниках, в которых приводятся различные шаблоны использования API Microsoft Graph, включая аутентификацию в Azure AD, см. в статье [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Примеры и учебники сообщества Microsoft Graph).

## <a name="see-also"></a>См. также раздел

[Основные понятия и справочные материалы по API Microsoft Graph](/graph/use-the-api?context=graph%2fapi%2fbeta&view=graph-rest-beta&preserve-view=true)
