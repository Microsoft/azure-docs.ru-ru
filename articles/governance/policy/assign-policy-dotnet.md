---
title: Краткое руководство. Создание назначения политики в .NET Core
description: В рамках этого краткого руководства с помощью .NET Core вы создадите назначение в службе "Политика Azure" для определения ресурсов, которые не соответствуют требованиям.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: b89a0db419b1e7213e9742e4a39d073445d05189
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106091792"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-net-core"></a>Краткое руководство. Создание назначения политики для идентификации ресурсов, не соответствующих требованиям, с помощью .NET Core

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние. В рамках этого краткого руководства вы создадите назначение политики для выявления виртуальных машин, которые не используют управляемые диски. После выполнения задач в руководстве вы сможете выявлять _несоответствующие_ виртуальные машины.

Библиотека .NET Core используется для управления ресурсами Azure. В этом руководство объясняется, как создать назначение политики с помощью библиотеки .NET Core для Политики Azure.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Субъект-служба Azure, включая _clientId_ и _clientSecret_. Если у вас нет субъекта-службы, который можно использовать для Политики Azure, или вы хотите создать новый субъект-службу, см. статью о [библиотеках управления Azure для аутентификации .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Пропустите этот шаг, чтобы установить пакеты .NET Core, так как мы выполним это действие на следующих шагах.

## <a name="create-the-azure-policy-project"></a>Создание проекта Политики Azure

Чтобы управлять Политикой Azure из .NET Core, создайте новое консольное приложение и установите нужные пакеты.

1. Убедитесь, что установлена последняя версия .NET Core (не ниже **3.1.8**). Если она еще не установлена, скачайте ее на сайте [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Инициализируйте новое консольное приложение .NET Core с именем policyAssignment:

   ```dotnetcli
   dotnet new console --name "policyAssignment"
   ```

1. Перейдите к папке нового проекта и установите нужные пакеты для Политики Azure:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceManager --version 3.10.0-preview

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
   using Microsoft.Azure.Management.ResourceManager;
   using Microsoft.Azure.Management.ResourceManager.Models;
   
   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];
               string strDisplayName = args[5];
               string strPolicyDefID = args[6];
               string strDescription = args[7];
               string strScope = args[8];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));

               using (var client = new PolicyClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyAssignment = new PolicyAssignment
                   {
                       DisplayName = strDisplayName,
                       PolicyDefinitionId = strPolicyDefID,
                       Description = strDescription
                   };
                   var response = await client.PolicyAssignments.CreateAsync(strScope, strName, policyAssignment);
               }
           }
       }
   }
   ```

1. Скомпилируйте и опубликуйте консольное приложение `policyAssignment`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-a-policy-assignment"></a>Создание назначения политики

С помощью этого краткого руководства вы создадите назначение политики и назначите определение **Аудит виртуальных машин, которые не используют управляемые диски** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Это определение политики идентифицирует ресурсы, которые не соответствуют заданным в нем условиям.

1. Измените каталоги на папку `{run-folder}`, определенную с помощью предыдущей команды `dotnet publish`.

1. Введите следующую команду в окне терминала.

   ```bash
   policyAssignment.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks" `
      "Audit VMs without managed disks Assignment" `
      "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      "Shows all virtual machines not using managed disks" `
      "{scope}"
   ```

В указанных выше командах используются следующие сведения:

- `{tenantId}` — замените своим идентификатором клиента.
- `{clientId}` — замените идентификатором клиента своего субъекта-службы.
- `{clientSecret}` — замените секретом клиента своего субъекта-службы.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.
- **name** — уникальное имя для объекта назначения политики. В приведенном выше примере это _audit-vm-manageddisks_.
- **displayName** — отображаемое имя назначения политики. В этом случае используется определение _аудита виртуальных машин без назначения управляемых дисков_.
- **policyDefID** — путь определения политики, на основе которого вы создаете назначение. В нашем случае это идентификатор определения политики _аудита виртуальных машин, которые не используют управляемые диски_.
- **description** — более подробное объяснение того, что делает политика или почему она относится к этой области.
- **scope** — область, которая определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Она может варьироваться от группы управления до отдельного ресурса. Обязательно замените `{scope}` на один из следующих шаблонов:
  - Группа управления: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Подписка: `/subscriptions/{subscriptionId}`
  - группу ресурсов `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`;
  - Ресурс: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`.

Теперь все готово к обнаружению ресурсов, которые не соответствуют требованиям, что позволит оценить состояние соответствия в среде.

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Итак, вы создали назначение политики и теперь можете определить ресурсы, не соответствующие требованиям.

1. Инициализируйте новое консольное приложение .NET Core с именем policyCompliance:

   ```dotnetcli
   dotnet new console --name "policyCompliance"
   ```

1. Перейдите к папке нового проекта и установите нужные пакеты для Политики Azure:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.PolicyInsights --version 3.1.0

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
   using Microsoft.Azure.Management.PolicyInsights;
   using Microsoft.Azure.Management.PolicyInsights.Models;
   
   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));
   
               using (var client = new PolicyInsightsClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyQueryOptions = new QueryOptions
                   {
                       Filter = $"IsCompliant eq false and PolicyAssignmentId eq '{strName}'",
                       Apply = "groupby(ResourceId)"
                   };
   
                   var response = await client.PolicyStates.ListQueryResultsForSubscriptionAsync(
                       "latest", strSubscriptionId, policyQueryOptions);
                   Console.WriteLine(response.Odatacount);
               }
           }
       }
   }
   ```

1. Скомпилируйте и опубликуйте консольное приложение `policyAssignment`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

1. Измените каталоги на папку `{run-folder}`, определенную с помощью предыдущей команды `dotnet publish`.

1. Введите следующую команду в окне терминала.

   ```bash
   policyCompliance.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks"
   ```

В указанных выше командах используются следующие сведения:

- `{tenantId}` — замените своим идентификатором клиента.
- `{clientId}` — замените идентификатором клиента своего субъекта-службы.
- `{clientSecret}` — замените секретом клиента своего субъекта-службы.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.
- **name** — уникальное имя для объекта назначения политики. В приведенном выше примере это _audit-vm-manageddisks_.

Результаты в `response` совпадают с отображающимися на вкладке **Соответствие ресурсов** для назначения политики в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

- С помощью портала удалите назначение политики _Аудит виртуальных машин без назначения управляемых дисков_. Определение политики встроено, поэтому его удалять не нужно.

- Если вы хотите удалить консольное приложение .NET Core и установленные пакеты, удалите папки проекта `policyAssignment` и `policyCompliance`.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Чтобы узнать больше об определениях политики назначения для проверки новых ресурсов на соответствие требованиям, см. следующее руководство:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)
