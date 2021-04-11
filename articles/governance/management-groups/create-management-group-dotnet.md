---
title: Краткое руководство. Создание группы управления с помощью .NET Core
description: В этом кратком руководстве показано, как с помощью .NET Core создать группу управления для организации ресурсов в иерархии ресурсов.
ms.date: 02/05/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: a74cea9b142785c093b8ed235fc40049746f11a0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99592591"
---
# <a name="quickstart-create-a-management-group-with-net-core"></a>Краткое руководство. Создание группы управления с помощью .NET Core

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Субъект-служба Azure, включая _clientId_ и _clientSecret_. Если у вас нет субъекта-службы, который можно использовать для Политики Azure, или вы хотите создать новый субъект-службу, см. статью о [библиотеках управления Azure для аутентификации .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Пропустите этот шаг, чтобы установить пакеты .NET Core, так как мы выполним это действие на следующих шагах.

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Настройка приложения

Чтобы управлять группами управления из .NET Core, создайте новое консольное приложение и установите нужные пакеты.

1. Убедитесь, что установлена последняя версия .NET Core (не ниже **3.1.8**). Если она еще не установлена, скачайте ее на сайте [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Инициализируйте новое консольное приложение .NET Core с именем mgCreate:

   ```dotnetcli
   dotnet new console --name "mgCreate"
   ```

1. Перейдите к папке нового проекта и установите нужные пакеты для Политики Azure:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ManagementGroups --version 1.1.1-preview

   # Add the Azure app auth package for .NET Core
   dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.5.0
   ```

1. Замените файл `program.cs` по умолчанию следующим кодом и сохраните обновленный файл:

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ManagementGroups;
   using Microsoft.Azure.Management.ManagementGroups.Models;
   
   namespace mgCreate
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strGroupId = args[3];
               string strDisplayName = args[4];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                      "https://management.core.windows.net",
                      new ClientCredential(strClientId, strClientSecret));
   
               using (var client = new ManagementGroupsAPIClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var mgRequest = new CreateManagementGroupRequest
                   {
                       DisplayName = strDisplayName
                   };
                   var response = await client.ManagementGroups.CreateOrUpdateAsync(strGroupId, mgRequest);
               }
           }
       }
   }
   ```

1. Скомпилируйте и опубликуйте консольное приложение `mgCreate`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-the-management-group"></a>Создание группы управления

В рамках этого краткого руководства вы создадите новую группу управления в корневой группе управления.

1. Измените каталоги на папку `{run-folder}`, определенную с помощью предыдущей команды `dotnet publish`.

1. Введите следующую команду в окне терминала.

   ```bash
   mgCreate.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{groupID}" `
      "{displayName}"
   ```

В указанных выше командах используются следующие сведения:

- `{tenantId}` — замените своим идентификатором клиента.
- `{clientId}` — замените идентификатором клиента своего субъекта-службы.
- `{clientSecret}` — замените секретом клиента своего субъекта-службы.
- `{groupID}` — замените это значение идентификатором новой группы управления.
- `{displayName}` — замените это значение понятным именем для новой группы управления.

В результате выполнения в корневой группе управления появится новая группа управления.

## <a name="clean-up-resources"></a>Очистка ресурсов

- Удалите новую группу управления с помощью портала.

- Если вы хотите удалить консольное приложение .NET Core и установленные пакеты, удалите папку проекта `mgCreate`.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)