---
title: Приступая к работе с библиотекой Azure CDN для .NET | Документация Майкрософт
description: Узнайте, как с помощью Visual Studio создавать приложения .NET для управления Azure CDN.
services: cdn
documentationcenter: .net
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 63cf4101-92e7-49dd-a155-a90e54a792ca
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: a812704c42a4da5ddf89fe6c5ba0c9a684047f75
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88919494"
---
# <a name="get-started-with-the-azure-cdn-library-for-net"></a>Приступая к работе с библиотекой Azure CDN для .NET
> [!div class="op_single_selector"]
> * [Node.js](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
>
>

С помощью [библиотеки Azure CDN для .NET](/dotnet/api/overview/azure/cdn) можно автоматизировать создание профилей и конечных точек CDN и управление ими.  В этом руководстве описывается создание простого консольного приложения .NET, которое демонстрирует некоторые из доступных операций.  Данный учебник не содержит подробных сведений о всех аспектах библиотеки Azure CDN для .NET.

Для работы с этим руководством требуется Visual Studio 2015.  [Visual Studio Community 2015](https://www.visualstudio.com/products/visual-studio-community-vs.aspx) .

> [!TIP]
> [Завершенный проект из этого учебника](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c) доступен для скачивания на сайте MSDN.
>
>

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-nuget-packages"></a>Создание проекта и добавление пакетов Nuget
Создав группу ресурсов для профилей CDN и назначив приложению Azure AD разрешения для управления профилями CDN и конечными точками в этой группе, мы можем приступить к созданию своего приложения.

В Visual Studio 2015 щелкните **Файл** > **Создать** > **Проект**. Откроется диалоговое окно создания проекта.  В области слева разверните узел **Visual C#** и выберите **Windows**.  В центральной области щелкните **Консольное приложение**.  Присвойте проекту имя и нажмите кнопку **ОК**.

![Создать проект](./media/cdn-app-dev-net/cdn-new-project.png)

В нашем проекте будут использоваться некоторые библиотеки Azure, содержащиеся в пакетах Nuget.  Давайте добавим их в проект.

1. В меню **Сервис** выберите **Диспетчер пакетов NuGet**, а затем — **Консоль диспетчера пакетов**.

    ![Управление пакетами NuGet](./media/cdn-app-dev-net/cdn-manage-nuget.png)
2. В консоли диспетчера пакетов выполните приведенную ниже команду, чтобы установить **библиотеку аутентификации Active Directory (ADAL)** .

    `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory`
3. Выполните следующую команду, чтобы установить **библиотеки управления Azure CDN**.

    `Install-Package Microsoft.Azure.Management.Cdn`

## <a name="directives-constants-main-method-and-helper-methods"></a>Директивы, константы, главный метод и вспомогательные методы
Давайте напишем базовую структуру нашей программы.

1. На вкладке Program.cs замените расположенные вверху директивы `using` на следующие.

    ```csharp
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Management.Cdn;
    using Microsoft.Azure.Management.Cdn.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```
2. Необходимо определить несколько констант, которые будут использоваться нашими методами.  В классе `Program` перед методом `Main` добавьте приведенный ниже код.  Обязательно замените заполнители, включая **&lt;угловые скобки&gt;** , собственными значениями.

    ```csharp
    //Tenant app constants
    private const string clientID = "<YOUR CLIENT ID>";
    private const string clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    private const string authority = "https://login.microsoftonline.com/<YOUR TENANT ID>/<YOUR TENANT DOMAIN NAME>";

    //Application constants
    private const string subscriptionId = "<YOUR SUBSCRIPTION ID>";
    private const string profileName = "CdnConsoleApp";
    private const string endpointName = "<A UNIQUE NAME FOR YOUR CDN ENDPOINT>";
    private const string resourceGroupName = "CdnConsoleTutorial";
    private const string resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. Кроме того, на уровне класса определите эти две переменные.  Мы воспользуемся ими позже, чтобы определить, существует ли профиль или конечная точка.

    ```csharp
    static bool profileAlreadyExists = false;
    static bool endpointAlreadyExists = false;
    ```
4. Замените метод `Main` следующим кодом.

   ```csharp
   static void Main(string[] args)
   {
       //Get a token
       AuthenticationResult authResult = GetAccessToken();

       // Create CDN client
       CdnManagementClient cdn = new CdnManagementClient(new TokenCredentials(authResult.AccessToken))
           { SubscriptionId = subscriptionId };

       ListProfilesAndEndpoints(cdn);

       // Create CDN Profile
       CreateCdnProfile(cdn);

       // Create CDN Endpoint
       CreateCdnEndpoint(cdn);

       Console.WriteLine();

       // Purge CDN Endpoint
       PromptPurgeCdnEndpoint(cdn);

       // Delete CDN Endpoint
       PromptDeleteCdnEndpoint(cdn);

       // Delete CDN Profile
       PromptDeleteCdnProfile(cdn);

       Console.WriteLine("Press Enter to end program.");
       Console.ReadLine();
   }
   ```
5. Некоторые из других методов будет запрашивать у пользователя ответ на вопрос в формате "Да/нет".  Добавьте следующий метод, чтобы упростить это.

    ```csharp
    private static bool PromptUser(string Question)
    {
        Console.Write(Question + " (Y/N): ");
        var response = Console.ReadKey();
        Console.WriteLine();
        if (response.Key == ConsoleKey.Y)
        {
            return true;
        }
        else if (response.Key == ConsoleKey.N)
        {
            return false;
        }
        else
        {
            // They pressed something other than Y or N.  Let's ask them again.
            return PromptUser(Question);
        }
    }
    ```

Написав базовую структуру программы, давайте создадим методы, вызываемые методом `Main` .

## <a name="authentication"></a>Аутентификация
Прежде чем можно будет использовать библиотеку управления Azure CDN, необходимо аутентифицировать наш субъект-службу и получить токен аутентификации.  Этот метод использует ADAL для получения токена.

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority);
    ClientCredential credential = new ClientCredential(clientID, clientSecret);
    AuthenticationResult authResult =
        authContext.AcquireTokenAsync("https://management.core.windows.net/", credential).Result;

    return authResult;
}
```

Если вы используете аутентификацию отдельных пользователей, метод `GetAccessToken` будет выглядеть немного иначе.

> [!IMPORTANT]
> Используйте приведенный пример кода, только если решили использовать аутентификацию отдельных пользователей, а не субъект-службу.
>
>

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority);
    AuthenticationResult authResult = authContext.AcquireTokenAsync("https://management.core.windows.net/",
        clientID, new Uri("http://<redirect URI>"), new PlatformParameters(PromptBehavior.RefreshSession)).Result;

    return authResult;
}
```

Обязательно замените `<redirect URI>` универсальным кодом ресурса (URI) перенаправления, введенным при регистрации приложения в Azure AD.

## <a name="list-cdn-profiles-and-endpoints"></a>Вывод списка профилей CDN и конечных точек
Теперь все готово к выполнению операций CDN.  Первое, что делает наш метод, — выводит список профилей и конечных точек в группе ресурсов и, если он обнаруживает совпадение с именами профиля и конечных точек, указанными в константах, то учитывает это на будущее, чтобы мы не пытались создать повторяющиеся значения.

```csharp
private static void ListProfilesAndEndpoints(CdnManagementClient cdn)
{
    // List all the CDN profiles in this resource group
    var profileList = cdn.Profiles.ListByResourceGroup(resourceGroupName);
    foreach (Profile p in profileList)
    {
        Console.WriteLine("CDN profile {0}", p.Name);
        if (p.Name.Equals(profileName, StringComparison.OrdinalIgnoreCase))
        {
            // Hey, that's the name of the CDN profile we want to create!
            profileAlreadyExists = true;
        }

        //List all the CDN endpoints on this CDN profile
        Console.WriteLine("Endpoints:");
        var endpointList = cdn.Endpoints.ListByProfile(p.Name, resourceGroupName);
        foreach (Endpoint e in endpointList)
        {
            Console.WriteLine("-{0} ({1})", e.Name, e.HostName);
            if (e.Name.Equals(endpointName, StringComparison.OrdinalIgnoreCase))
            {
                // The unique endpoint name already exists.
                endpointAlreadyExists = true;
            }
        }
        Console.WriteLine();
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>Создание профилей CDN и конечных точек
Далее мы создадим профиль.

```csharp
private static void CreateCdnProfile(CdnManagementClient cdn)
{
    if (profileAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating profile {0}.", profileName);
        ProfileCreateParameters profileParms =
            new ProfileCreateParameters() { Location = resourceLocation, Sku = new Sku(SkuName.StandardVerizon) };
        cdn.Profiles.Create(profileName, profileParms, resourceGroupName);
    }
}
```

После этого мы создадим конечную точку.

```csharp
private static void CreateCdnEndpoint(CdnManagementClient cdn)
{
    if (endpointAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating endpoint {0} on profile {1}.", endpointName, profileName);
        EndpointCreateParameters endpointParms =
            new EndpointCreateParameters()
            {
                Origins = new List<DeepCreatedOrigin>() { new DeepCreatedOrigin("Contoso", "www.contoso.com") },
                IsHttpAllowed = true,
                IsHttpsAllowed = true,
                Location = resourceLocation
            };
        cdn.Endpoints.Create(endpointName, endpointParms, profileName, resourceGroupName);
    }
}
```

> [!NOTE]
> В приведенном выше примере конечной точке назначается источник *Contoso* с именем узла `www.contoso.com`.  Это следует изменить, указав собственное имя узла источника.
>
>

## <a name="purge-an-endpoint"></a>Очистка конечной точки
После создания конечной точки одной из распространенных задач, которая может выполняться в нашей программе, является очистка содержимого в конечной точке.

```csharp
private static void PromptPurgeCdnEndpoint(CdnManagementClient cdn)
{
    if (PromptUser(String.Format("Purge CDN endpoint {0}?", endpointName)))
    {
        Console.WriteLine("Purging endpoint. Please wait...");
        cdn.Endpoints.PurgeContent(endpointName, profileName, resourceGroupName, new List<string>() { "/*" });
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

> [!NOTE]
> В приведенном выше примере строка `/*` означает, что я хочу очистить все содержимое в корне конечной точки.  Этот тоже самое, что установить флажок **Очистить все** в диалоговом окне "Очистить" на портале Azure. В методе `CreateCdnProfile` я создал профиль **Azure CDN от Verizon** с помощью кода `Sku = new Sku(SkuName.StandardVerizon)`, поэтому операция будет выполнена успешно.  Однако профили **Azure CDN от Akamai** не поддерживают функцию **Очистить все**. Если бы я использовал в этом руководстве профиль Akamai, мне пришлось бы указать конкретные расположения, в которых нужно выполнить очистку.
>
>

## <a name="delete-cdn-profiles-and-endpoints"></a>Удаление профилей CDN и конечных точек
Последние методы удаляют конечную точку и профиль.

```csharp
private static void PromptDeleteCdnEndpoint(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN endpoint {0} on profile {1}?", endpointName, profileName)))
    {
        Console.WriteLine("Deleting endpoint. Please wait...");
        cdn.Endpoints.DeleteIfExists(endpointName, profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}

private static void PromptDeleteCdnProfile(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN profile {0}?", profileName)))
    {
        Console.WriteLine("Deleting profile. Please wait...");
        cdn.Profiles.DeleteIfExists(profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

## <a name="running-the-program"></a>Запуск программы
Теперь можно скомпилировать и запустить программу, нажав в Visual Studio кнопку **Запустить** .

![Выполнение программы](./media/cdn-app-dev-net/cdn-program-running-1.png)

Когда программа дойдет до упомянутого выше запроса, вы сможете вернуться к группе ресурсов на портале Azure и увидеть, что профиль создан.

![Готово!](./media/cdn-app-dev-net/cdn-success.png)

Затем мы можем подтвердить запросы, чтобы была выполнена оставшаяся часть программы.

![Завершение программы](./media/cdn-app-dev-net/cdn-program-running-2.png)

## <a name="next-steps"></a>Next Steps
Чтобы просмотреть описываемый в этом руководстве готовый проект, [скачайте пример](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c).

Чтобы найти дополнительную документацию по библиотеке управления Azure CDN для .NET, воспользуйтесь [справкой на сайте MSDN](/dotnet/api/overview/azure/cdn).

Управление ресурсами CDN с помощью [PowerShell](cdn-manage-powershell.md).
