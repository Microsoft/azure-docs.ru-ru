---
title: Обзор библиотеки веб-аутентификации Microsoft Identity
titleSuffix: Microsoft identity platform
description: Сведения о Microsoft Identity Web, библиотеке проверки подлинности и авторизации для ASP.NET Core приложений, которые интегрируются с Azure Active Directory, Azure AD B2C, Microsoft Graph и другими веб-API.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/09/2020
ms.author: jmprieur
ms.reviewer: marsma
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 8bc9773fa7e1b3db2e249e803f4d2a3cf39edb48
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100102552"
---
# <a name="microsoft-identity-web-authentication-library"></a>Библиотека веб-аутентификации Microsoft Identity

Microsoft Identity Web — это набор библиотек ASP.NET Core, упрощающих Добавление поддержки проверки подлинности и авторизации в веб-приложения и веб-API, интегрированные с платформой Microsoft Identity. Он предоставляет удобный высокопроизводительный уровень API, связывающий ASP.NET Core, по промежуточного слоя для проверки подлинности и [библиотеку проверки подлинности Майкрософт (MSAL) для .NET](https://github.com/azuread/microsoft-authentication-library-for-dotnet).

## <a name="supported-application-scenarios"></a>Поддерживаемые сценарии приложений

Если вы создаете ASP.NET Core веб-приложений или веб-API и хотите использовать Azure Active Directory (Azure AD) или Azure AD B2C для управления удостоверениями и доступом (IAM), мы рекомендуем использовать Microsoft Identity Web для всех следующих сценариев:

- [Веб-приложение, которое поддерживает вход пользователей](scenario-web-app-sign-user-overview.md)
- [Веб-приложение, которое подписывает пользователей и вызывает веб-API от их имени](scenario-web-app-call-api-overview.md)
- [Защищенный веб-API, доступ к которому могут получить только пользователи, прошедшие проверку](scenario-protected-web-api-overview.md)
- [Защищенный веб-API, который вызывает другой (нисходящий) веб-API от имени пользователя, выполнившего вход](scenario-web-api-call-api-overview.md)

## <a name="get-the-library"></a>Получение библиотеки

Вы можете получить веб-сайт Microsoft Identity из [NuGet](#nuget), [шаблонов проектов .NET Core](#project-templates)и [GitHub](#github).

#### <a name="nuget"></a>NuGet

Веб-сайт Microsoft Identity доступен в NuGet в виде набора пакетов, которые предоставляют модульные функции в зависимости от потребностей вашего приложения. Используйте команду интерфейса командной строки .NET `dotnet add` или **Диспетчер пакетов NuGet** для Visual Studio, чтобы установить пакеты, подходящие для вашего проекта:

- [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web) — основной пакет. Требуется для всех приложений, использующих веб-сайт Microsoft Identity.
- [Microsoft. Identity. Web. UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) — необязательно. Добавляет пользовательский интерфейс для входа и выхода пользователя и связанного контроллера для веб-приложений.
- [Microsoft. Identity. Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) — необязательный. Предоставляет Упрощенное взаимодействие с Microsoft Graph API.
- [Microsoft. Identity. Web. микрософтграфбета](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraphBeta) — необязательный. Предоставляет Упрощенное взаимодействие с [конечной точкой бета-версии](/graph/api/overview?view=graph-rest-beta&preserve-view=true)Microsoft Graph API.

#### <a name="project-templates"></a>Шаблоны проектов

Шаблоны веб-проектов Microsoft Identity включены в .NET 5,0 и доступны для загрузки в проектах ASP.NET Core 3,1.

Если вы используете ASP.NET Core 3,1, установите шаблоны с помощью интерфейса командной строки .NET:

```dotnetcli
dotnet new --install Microsoft.Identity.Web.ProjectTemplates::1.0.0
```

На следующей схеме показано высокоуровневое представление поддерживаемых типов приложений и соответствующих им аргументов.

:::image type="content" source="media/microsoft-identity-web-overview/diagram-microsoft-identity-web-templates.png" lightbox="media/microsoft-identity-web-overview/diagram-microsoft-identity-web-templates.png" alt-text="Схема доступных шаблонов проектов CLI с точкой NET для Microsoft Identity Web":::
<br /><sup><b>*</b></sup>`MultiOrg`не поддерживается `webapi2` в, но его можно включить в *appsettings.js* , установив для параметра "клиент" значение `common` или`organizations`
<br /><sup><b>**</b></sup>`--calls-graph`не поддерживается для Azure AD B2C

В этом примере команды .NET CLI, взятые из [учебника по серверу блазор](tutorial-blazor-server.md), создается новый проект сервера блазор, включающий правильные пакеты и начальный код (показаны значения заполнителей):

```dotnetcli
dotnet new blazorserver2 --auth SingleOrg --calls-graph --client-id "00000000-0000-0000-0000-000000000000" --tenant-id "11111111-1111-1111-1111-111111111111" --output my-blazor-app
```

#### <a name="github"></a>GitHub

Microsoft Identity Web — это проект с открытым кодом, размещенный на сайте GitHub: <a href="https://github.com/AzureAD/microsoft-identity-web" target="_blank">AzureAD/Microsoft-Identity-Web</a>

[Вики-сайт репозитория](https://github.com/AzureAD/microsoft-identity-web/wiki) содержит дополнительную документацию, и если вам нужна помощь или обнаружение ошибки, можно [придать ему сообщение](https://github.com/AzureAD/microsoft-identity-web/issues).

## <a name="features"></a>Компоненты

Веб-сайт Microsoft Identity включает несколько функций, которые не предоставляются при использовании шаблонов проектов ASP.NET 3,1 по умолчанию.

| Компонент                                                                                  | ASP.NET Core 3.1                                                     | Веб-сайт Microsoft Identity                                                                                  |
|------------------------------------------------------------------------------------------|----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| [Вход пользователей](scenario-web-app-sign-user-app-configuration.md) в веб-приложениях             | <li>Рабочие или учебные учетные записи<li>Удостоверения социальных сетей (с Azure AD B2C) | <li>Рабочие или учебные учетные записи<li>Personal Microsoft accounts (Личные учетные записи Майкрософт)<li>Удостоверения социальных сетей (с Azure AD B2C)     |
| [Защита веб-API](scenario-protected-web-api-app-configuration.md#microsoftidentityweb) | <li>Рабочие или учебные учетные записи<li>Удостоверения социальных сетей (с Azure AD B2C) | <li>Рабочие или учебные учетные записи<li>Personal Microsoft accounts (Личные учетные записи Майкрософт)<li>Удостоверения социальных сетей (с Azure AD B2C)     |
| Проверка издателя в приложениях с несколькими клиентами                                                   | Нет                                                                   | Да, для [всех облаков](authentication-national-cloud.md) и [Azure AD B2C](../../active-directory-b2c/index.yml) |
| Веб-приложение или API [вызовы Microsoft Graph] [сценарий-API-Call-Graph]                             | Нет                                                                   | Да                                                                                                     |
| Веб-приложение или API [вызовы веб-API] [сценарий-API-Call-API]                                       | Нет                                                                   | Да                                                                                                     |
| Поддержка учетных данных сертификата                                                         | Нет                                                                   | Да, включая Azure Key Vault                                                                          |
| Добавочное согласие и поддержка условного доступа в веб-приложениях                           | Нет                                                                   | Да, в MVC, на страницах Razor и Блазор                                                                    |
| Сертификаты шифрования маркеров в веб-API                                                | Нет                                                                   | Да                                                                                                     |
| [Область действия/проверка роли приложения] [сценарий — API-проверка] в веб-API                        | Нет                                                                   | Да                                                                                                     |
| `WWW-Authenticate` Создание заголовков в веб-API                                         | Нет                                                                   | Да                                                                                                     |

## <a name="next-steps"></a>Дальнейшие действия

Чтобы просмотреть Microsoft Identity Web в действии, воспользуйтесь нашим руководством по Блазор Server.

[Руководство по Создание приложения Blazor Server, которое использует платформу удостоверений Майкрософт для аутентификации](tutorial-blazor-server.md)

Веб-вики Microsoft Identity на сайте GitHub содержит подробную справочную документацию по различным аспектам библиотеки. Например, использование сертификатов, добавочное согласие и ссылка на условный доступ можно найти здесь:

- <a href="https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates" target="_blank">Использование сертификатов с Microsoft. Identity. Web</a> (GitHub)
- <a href="https://github.com/AzureAD/microsoft-identity-web/wiki/Managing-incremental-consent-and-conditional-access" target="_blank">Поэтапное согласие и условный доступ</a> (GitHub)

<!-- LINKS -->
<!--  [miw-certs]: microsoft-identity-web-certificates.md  -->
<!--  [miw-certs-decrypt]: microsoft-identity-web-certificates.md#decryption-certificates  -->
<!--  [miw-inc-consent-ca-header]: microsoft-identity-web-consent-conditional-access.md#handling-incremental-consent-or-conditional-access-in-web-apis  -->
<!--  [miw-inc-consent-ca]: microsoft-identity-web-consent-conditional-access.md  -->
[сценарий-API-Call-API]: сценарий-Web-API-Call-API-Call-API. md # параметр-1 — вызов-Microsoft-Graph-с-The-SDK  
[сценарий-API-Call-Graph]: сценарий-Web-API-Call-API-Call-API. md # параметр-1-Call-Microsoft-Graph-с-The-SDK  
[сценарий-API-Validation]: scenario-protected-web-api-verification-scope-app-roles.md  
