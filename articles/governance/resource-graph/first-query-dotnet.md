---
title: Краткое руководство. Ваш первый запрос .NET Core
description: В этом кратком руководстве показано, как включить пакеты NuGet Resource Graph для .NET Core и выполнить первый запрос.
ms.date: 01/27/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 0135dfd499af48b3c60314679f4c9b635a5ce15a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98917577"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-net-core"></a>Краткое руководство. Первый запрос Resource Graph с помощью .NET Core

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлены ли необходимые пакеты для .NET Core. В этом кратком руководстве показано, как добавить пакеты в установку .NET Core.

В результате вы добавите пакеты в установку .NET Core и выполните запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Субъект-служба Azure, включая _clientId_ и _clientSecret_. Если у вас нет субъекта-службы для использования с Resource Graph или вы хотите создать новый субъект-службу, см. раздел [Библиотеки управления Azure для аутентификации в .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Пропустите этот шаг, чтобы установить пакеты .NET Core, так как мы выполним это действие на следующих шагах.

## <a name="create-the-resource-graph-project"></a>Создание проекта Resource Graph

Чтобы обеспечить отправку запросов из .NET Core в Azure Resource Graph, создайте новое консольное приложение и установите нужные пакеты.

1. Убедитесь, что установлена последняя версия .NET Core (не ниже **3.1.5**). Если она еще не установлена, скачайте ее на сайте [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Инициализируйте новое консольное приложение .NET Core с именем argQuery:

   ```dotnetcli
   dotnet new console --name "argQuery"
   ```

1. Измените каталоги на соответствующие папке нового проекта и установите нужные пакеты для Azure Resource Graph:

   ```dotnetcli
   # Add the Resource Graph package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceGraph --version 2.0.0

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
   using Microsoft.Azure.Management.ResourceGraph;
   using Microsoft.Azure.Management.ResourceGraph.Models;

   namespace argQuery
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strQuery = args[4];

               AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/" + strTenant);
               AuthenticationResult authResult = await authContext.AcquireTokenAsync("https://management.core.windows.net", new ClientCredential(strClientId, strClientSecret));
               ServiceClientCredentials serviceClientCreds = new TokenCredentials(authResult.AccessToken);

               ResourceGraphClient argClient = new ResourceGraphClient(serviceClientCreds);
               QueryRequest request = new QueryRequest();
               request.Subscriptions = new List<string>(){ strSubscriptionId };
               request.Query = strQuery;

               QueryResponse response = argClient.Resources(request);
               Console.WriteLine("Records: " + response.Count);
               Console.WriteLine("Data:\n" + response.Data);
           }
       }
   }
   ```

1. Скомпилируйте и опубликуйте консольное приложение `argQuery`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда консольное приложение .NET Core создано и опубликовано, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

В каждом вызове к `argQuery` есть переменные, которые необходимо заменить собственными значениями:

- `{tenantId}` — замените своим идентификатором клиента.
- `{clientId}` — замените идентификатором клиента своего субъекта-службы.
- `{clientSecret}` — замените секретом клиента своего субъекта-службы.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.

1. Измените каталоги на папку `{run-folder}`, определенную с помощью предыдущей команды `dotnet publish`.

1. Выполните первый запрос к Azure Resource Graph, используя скомпилированное консольное приложение .NET Core:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Измените значение последнего параметра на `argQuery.exe` и измените запрос, чтобы выполнить сортировку (`order by`) по свойству **Name**:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Измените значение последнего параметра на `argQuery.exe`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить вывод (`limit`) первыми пятью результатами:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить консольное приложение .NET Core и установленные пакеты, удалите папку проекта `argQuery`.

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого краткого руководства вы создали консольное приложение .NET Core с необходимыми пакетами Resource Graph и выполнили свой первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).