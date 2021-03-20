---
title: Изменения, вносимые в проект MVC при подключении к Azure AD
description: Описывает, что происходит с проектом MVC при подключении к приложению Azure AD с помощью подключенных служб Visual Studio
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: ab72512b2e5d915dbd4b8a3d7d8446a5932eef9e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88165453"
---
# <a name="what-happened-to-my-mvc-project-visual-studio-azure-active-directory-connected-service"></a>Что произошло с моим проектом MVC в подключенной службе Visual Studio Azure Active Directory?

> [!div class="op_single_selector"]
> - [Начало работы](vs-active-directory-dotnet-getting-started.md)
> - [Что произошло?](vs-active-directory-dotnet-what-happened.md)

В этой статье описаны конкретные изменения, которые вносятся в проекты ASP.NET MVC при добавлении [подключенной службы Azure Active Directory через Visual Studio](vs-active-directory-add-connected-service.md).

Сведения о работе с подключенной службой см. в статье [Начало работы с Azure Active Directory и подключенными службами Visual Studio (проекты WebApi)](vs-active-directory-dotnet-getting-started.md).

## <a name="added-references"></a>Добавленные ссылки

Это изменение влияет на файлы проекта (ссылки *.NET) и `packages.config` (ссылки NuGet).

| Тип | Справочник |
| --- | --- |
| .NET; NuGet | Microsoft.IdentityModel.Protocol.Extensions |
| .NET; NuGet | Microsoft.Owin |
| .NET; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET; NuGet | Microsoft.Owin.Security |
| .NET; NuGet | Microsoft.Owin.Security.Cookies |
| .NET; NuGet | Microsoft.Owin.Security.OpenIdConnect |
| .NET; NuGet | Owin |
| .NET        | System.IdentityModel |
| .NET; NuGet | System.IdentityModel.Tokens.Jwt |
| .NET        | System.Runtime.Serialization |

Дополнительные ссылки, если выбран параметр **Чтение данных каталога**

| Тип | Справочник |
| --- | --- |
| .NET; NuGet | EntityFramework |
| .NET        | EntityFramework.SqlServer (только для Visual Studio 2015) |
| .NET; NuGet | Microsoft.Azure.ActiveDirectory.GraphClient |
| .NET; NuGet | Microsoft.Data.Edm |
| .NET; NuGet | Microsoft.Data.OData |
| .NET; NuGet | Microsoft.Data.Services.Client |
| .NET; NuGet | Microsoft.IdentityModel.Clients.ActiveDirectory; |
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms (только для Visual Studio 2015) |
| .NET; NuGet | System.Spatial |

Следующие ссылки удалены (только для проектов ASP.NET 4, например в Visual Studio 2015):

| Тип | Справочник |
| --- | --- |
| .NET; NuGet | Microsoft.AspNet.Identity.Core |
| .NET; NuGet | Microsoft.AspNet.Identity.EntityFramework |
| .NET; NuGet | Microsoft.AspNet.Identity.Owin |

## <a name="project-file-changes"></a>Изменения в файле проекта

- Для свойства `IISExpressSSLPort` установлено конкретное число.
- Для свойства `WebProject_DirectoryAccessLevelKey` установлено значение 0 (или 1, если выбран параметр **Чтение данных каталога**).
- Для свойства `IISUrl` установлено значение `https://localhost:<port>/`, где `<port>` соответствует значению `IISExpressSSLPort`.

## <a name="webconfig-or-appconfig-changes"></a>Изменения в файлах web.config или app.config

- Добавлены следующие записи конфигурации:

    ```xml
    <appSettings>
        <add key="ida:ClientId" value="<ClientId from the new Azure AD app>" />
        <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
        <add key="ida:Domain" value="<your selected Azure domain>" />
        <add key="ida:TenantId" value="<the Id of your selected Azure AD tenant>" />
        <add key="ida:PostLogoutRedirectUri" value="<project start page, such as https://localhost:44335>" />
    </appSettings>
    ```

- Добавлены элементы `<dependentAssembly>` в узел `<runtime><assemblyBinding>` для `System.IdentityModel.Tokens.Jwt` и `Microsoft.IdentityModel.Protocol.Extensions`.

Дополнительные изменения, если выбран параметр **Чтение данных каталога**

- Добавлены следующие записи конфигурации в `<appSettings>`.

    ```xml
    <add key="ida:ClientSecret" value="<Azure AD app's new client secret>" />
    ```

- Добавлены следующие элементы в `<configuration>`. Значения будут разными для MDF-файла проекта и идентификатора каталога проекта:

    ```xml
    <configSections>
      <!-- For more information on Entity Framework configuration, visit https://go.microsoft.com/fwlink/?LinkID=237468 -->
      <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>

    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\<project-mdf-file>.mdf;Initial Catalog=<project-catalog-id>;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>

    <entityFramework>
      <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
        <parameters>
          <parameter value="mssqllocaldb" />
        </parameters>
      </defaultConnectionFactory>
      <providers>
        <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      </providers>
    </entityFramework>
    ```

- Добавлены элементы `<dependentAssembly>` в узел `<runtime><assemblyBinding>` для `Microsoft.Data.Services.Client`, `Microsoft.Data.Edm` и `Microsoft.Data.OData`.

## <a name="code-changes-and-additions"></a>Изменения и дополнения в коде

- Добавлен атрибут `[Authorize]` в `Controllers/HomeController.cs` и все остальные существующие контроллеры.

- Добавлен класс запуска `App_Start/Startup.Auth.cs` для аутентификации, который содержит логику запуска для аутентификации Azure AD. Если вы выбрали параметр **Чтение данных каталога**, этот файл содержит еще и код для получения кода OAuth и его обмена на маркер доступа.

- Добавлен класс контроллера `Controllers/AccountController.cs`, который содержит методы `SignIn` и `SignOut`.

- Добавлено частичное представление `Views/Shared/_LoginPartial.cshtml`, которое содержит ссылку на действие для `SignIn` и `SignOut`.

- Добавлено частичное представление `Views/Account/SignoutCallback.cshtml`, которое содержит HTML для пользовательского интерфейса выхода.

- В методе `Startup.Configuration` добавлен вызов `ConfigureAuth(app)`, если такой класс уже существовал. В противном случае добавлен класс `Startup` с вызовом этого метода.

- Добавлен файл `Connected Services/AzureAD/ConnectedService.json` (для Visual Studio 2017) или `Service References/Azure AD/ConnectedService.json` (для Visual Studio 2015), который содержит сведения, позволяющие Visual Studio отслеживать добавление подключенной службы.

- Если вы выбрали параметр **Чтение данных каталога**, добавлены `Models/ADALTokenCache.cs` и `Models/ApplicationDbContext.cs` для поддержки кэширования маркеров. Чтобы проиллюстрировать получение доступа к информации о пользовательском профиле с помощью интерфейсов API Graph Azure `Controllers/UserProfileController.cs`, `Views/UserProfile/Index.cshtml` и `Views/UserProfile/Relogin.cshtml`, добавлены дополнительные контроллер и представление.

### <a name="file-backup-visual-studio-2015"></a>Резервные копии файлов (для Visual Studio 2015)

При добавлении подключенной службы Visual Studio 2015 создает резервные копии всех измененных и удаленных файлов. Все эти файлы сохраняются в папке `Backup/AzureAD`. Visual Studio 2017 и более поздних версий не создает резервные копии.

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ManageViewModels.cs`
- `Views\Shared\_LoginPartial.cshtml`

## <a name="changes-on-azure"></a>Изменения в Azure

- Создано приложение Azure AD в домене, который вы выбрали при добавлении подключенной службы.
- В приложение добавлены разрешения на **чтение данных каталога**, если вы выбрали соответствующий параметр.

[Подробнее об Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## <a name="next-steps"></a>Дальнейшие действия

- [Сценарии аутентификации в Azure Active Directory](./authentication-vs-authorization.md)
- [Добавление возможности входа в веб-приложение ASP.NET с помощью учетной записи Майкрософт](quickstart-v2-aspnet-webapp.md)