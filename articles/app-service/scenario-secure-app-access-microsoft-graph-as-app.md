---
title: Учебник. Доступ веб-приложения к Microsoft Graph от имени приложения | Azure
description: Из этого учебника вы узнаете, как получить доступ к данным в Microsoft Graph с помощью управляемых удостоверений.
services: microsoft-graph, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 01/28/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 06837ab0f4685787f8d2615e81d0405fdb8ec711
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99062565"
---
# <a name="tutorial-access-microsoft-graph-from-a-secured-app-as-the-app"></a>Руководство по доступу к Microsoft Graph из защищенного приложения от имени приложения

Узнайте, как получить доступ к Microsoft Graph из веб-приложения, работающего в Службе приложений Azure.

:::image type="content" alt-text="Диаграмма, в которой показан доступ к Microsoft Graph." source="./media/scenario-secure-app-access-microsoft-graph/web-app-access-graph.svg" border="false":::

Необходимо вызвать Microsoft Graph для веб-приложения. Чтобы предоставить веб-приложению доступ к данным, можно использовать [управляемое удостоверение, назначаемое системой](../active-directory/managed-identities-azure-resources/overview.md). Управляемое удостоверение из Azure Active Directory позволяет Службе приложений получать доступ к ресурсам через управление доступом на основе ролей (RBAC) без учетных данных приложения. Когда вы назначаете веб-приложению управляемое удостоверение, Azure создает и распространяет сертификат. Вам не придется беспокоиться об управлении секретами или учетными данными приложения.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> * создавать для веб-приложения управляемое удостоверение, назначаемое системой;
> * добавлять разрешения API Microsoft Graph в управляемое удостоверение;
> * вызывать Microsoft Graph из веб-приложения с использованием управляемых удостоверений.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Обязательные условия

* Веб-приложение, выполняющееся в Службе приложений Azure, с [включенным модулем проверки подлинности или авторизации Службы приложений](scenario-secure-app-authentication-app-service.md).

## <a name="enable-managed-identity-on-app"></a>Включение управляемого удостоверения в приложении

Если вы создавали и публиковали веб-приложение с помощью Visual Studio, управляемое удостоверение уже включено для этого приложения. В Службе приложений в области слева выберите **Удостоверение**, а затем — **Назначаемое системой**. Убедитесь, что для параметра **Состояние** задано значение **Вкл**. В противном случае выберите **Сохранить**, а затем — **Да**, чтобы включить управляемое удостоверение, назначаемое системой. Если управляемое удостоверение включено, для состояния устанавливается значение **Вкл.** и будет доступен идентификатор объекта.

Запишите значение **идентификатора объекта**, которое потребуется на следующем шаге.

:::image type="content" alt-text="Снимок экрана: удостоверение, назначенное системой." source="./media/scenario-secure-app-access-microsoft-graph/create-system-assigned-identity.png":::

## <a name="grant-access-to-microsoft-graph"></a>Предоставление доступа к Microsoft Graph

При доступе к Microsoft Graph управляемому удостоверению требуются соответствующие разрешения для выполнения операции. В настоящее время нет возможности назначать такие разрешения с помощью портала Azure. С помощью приведенного ниже скрипта можно добавить запрошенные разрешения API Microsoft Graph в объект субъекта-службы управляемого удостоверения.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Install the module. (You need admin on the machine.)
# Install-Module AzureAD.

# Your tenant ID (in the Azure portal, under Azure Active Directory > Overview).
$TenantID="<tenant-id>"
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp-20201102125811"

# Get the ID of the managed identity for the web app.
$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid

# Check the Microsoft Graph documentation for the permission you need for the operation.
$PermissionName = "User.Read.All"

Connect-AzureAD -TenantId $TenantID

# Get the service principal for Microsoft Graph.
# First result should be AppId 00000003-0000-0000-c000-000000000000
$GraphServicePrincipal = Get-AzureADServicePrincipal -SearchString "Microsoft Graph" | Select-Object -first 1

# Assign permissions to the managed identity service principal.
$AppRole = $GraphServicePrincipal.AppRoles | `
Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}

New-AzureAdServiceAppRoleAssignment -ObjectId $spID -PrincipalId $spID `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole.Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az login

webAppName="SecureWebApp-20201106120003"

spId=$(az resource list -n $webAppName --query [*].identity.principalId --out tsv)

graphResourceId=$(az ad sp list --display-name "Microsoft Graph" --query [0].objectId --out tsv)

appRoleId=$(az ad sp list --display-name "Microsoft Graph" --query "[0].appRoles[?value=='User.Read.All' && contains(allowedMemberTypes, 'Application')].id" --output tsv)

uri=https://graph.microsoft.com/v1.0/servicePrincipals/$spId/appRoleAssignments

body="{'principalId':'$spId','resourceId':'$graphResourceId','appRoleId':'$appRoleId'}"

az rest --method post --uri $uri --body $body --headers "Content-Type=application/json"
```

---

После выполнения скрипта на [портале Azure](https://portal.azure.com) можно проверить, что запрашиваемые разрешения API назначены управляемому удостоверению.

Перейдите в раздел **Azure Active Directory**, а затем выберите **Корпоративные приложения**. В этой области отображаются все субъект-службы клиента. В разделе **Все приложения** выберите субъект-службу для управляемого удостоверения. 

При работе с этим учебником используются два субъекта-службы с одинаковым отображаемым именем (например, SecureWebApp2020094113531). Субъект-служба с **URL-адресом домашней страницы** представляет собой веб-приложение в клиенте. Субъект-служба без **URL-адреса домашней страницы** представляет собой управляемое удостоверение, назначаемое системой, для веб-приложения. Значение **идентификатора объекта** для управляемого удостоверения совпадает с идентификатором объекта созданного ранее управляемого удостоверения.

Выберите субъект-службу для управляемого удостоверения.

:::image type="content" alt-text="Снимок экрана: параметр &quot;Все приложения&quot;." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-all-applications.png":::

В разделе **Обзор** выберите **Разрешения** и вы увидите добавленные разрешения для Microsoft Graph.

:::image type="content" alt-text="Снимок экрана: панель &quot;Разрешения&quot;." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-permissions.png":::

## <a name="call-microsoft-graph-net"></a>Вызов Microsoft Graph (.NET)

Класс [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) используется для получения учетных данных токена для кода, чтобы авторизовать запросы в Microsoft Graph. Создайте экземпляр класса [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), который использует управляемое удостоверение для получения токенов и их подключения к клиенту службы. Приведенный ниже пример кода получает учетные данные токена с пройденной проверкой подлинности и использует их для создания объекта клиента службы, который получает список пользователей в группе.

Просмотреть этот код как часть примера приложения можно на [сайте GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/3-WebApp-graphapi-managed-identity).

### <a name="install-the-microsoftidentitywebmicrosoftgraph-client-library-package"></a>Установка пакета клиентской библиотеки Microsoft.Identity.Web.MicrosoftGraph

Установите в проект пакет NuGet [Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) с помощью интерфейса командной строки .NET Core или консоли диспетчера пакетов в Visual Studio.

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Откройте командную строку и перейдите в каталог с файлом проекта.

Выполните команды установки.

```dotnetcli
dotnet add package Microsoft.Identity.Web.MicrosoftGraph
```

# <a name="package-manager"></a>[Диспетчер пакетов](#tab/package-manager)

Откройте проект или решение в Visual Studio, а затем — консоль, выбрав **Средства** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.

Выполните команды установки.
```powershell
Install-Package Microsoft.Identity.Web.MicrosoftGraph
```

---

### <a name="example"></a>Пример

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Azure.Identity;
using Microsoft.Graph.Core;
using System.Net.Http.Headers;

...

public IList<MSGraphUser> Users { get; set; }

public async Task OnGetAsync()
{
    // Create the Microsoft Graph service client with a DefaultAzureCredential class, which gets an access token by using the available Managed Identity.
    var credential = new DefaultAzureCredential();
    var token = credential.GetToken(
        new Azure.Core.TokenRequestContext(
            new[] { "https://graph.microsoft.com/.default" }));

    var accessToken = token.Token;
    var graphServiceClient = new GraphServiceClient(
        new DelegateAuthenticationProvider((requestMessage) =>
        {
            requestMessage
            .Headers
            .Authorization = new AuthenticationHeaderValue("bearer", accessToken);

            return Task.CompletedTask;
        }));

    List<MSGraphUser> msGraphUsers = new List<MSGraphUser>();
    try
    {
        var users =await graphServiceClient.Users.Request().GetAsync();
        foreach(var u in users)
        {
            MSGraphUser user = new MSGraphUser();
            user.userPrincipalName = u.UserPrincipalName;
            user.displayName = u.DisplayName;
            user.mail = u.Mail;
            user.jobTitle = u.JobTitle;

            msGraphUsers.Add(user);
        }
    }
    catch(Exception ex)
    {
        string msg = ex.Message;
    }

    Users = msGraphUsers;
}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы завершили работу с этим учебником и вам больше не требуется веб-приложение или связанные с ним ресурсы, необходимо [очистить созданные ресурсы](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * создавать для веб-приложения управляемое удостоверение, назначаемое системой;
> * добавлять разрешения API Microsoft Graph в управляемое удостоверение;
> * вызывать Microsoft Graph из веб-приложения с использованием управляемых удостоверений.

Узнайте, как подключить к базе данных приложения [.NET Core](tutorial-dotnetcore-sqldb-app.md), [Python](tutorial-python-postgresql-app.md), [Java](tutorial-java-spring-cosmosdb.md) или [Node.js](tutorial-nodejs-mongodb-app.md).