---
title: Краткое руководство. Ваш первый запрос Go
description: В этом кратком руководстве показано, как включить пакет Resource Graph для Go и выполнить первый запрос.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: 4949801b3dc97904680d09e685fd225812a0e14d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98920068"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-go"></a>Краткое руководство. Первый запрос Resource Graph с помощью Go

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлены ли необходимые пакеты для Go. В этом кратком руководстве показано, как добавить пакеты в установку Go.

В результате вы добавите пакеты в установку Go и выполните запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="add-the-resource-graph-package"></a>Добавление пакета Resource Graph

Чтобы можно было отправлять запросы с помощью Go в Resource Graph, необходимо добавить соответствующий пакет. Этот пакет работает во всех средах, где можно использовать Go, в том числе в [Bash для Windows 10](/windows/wsl/install-win10). Также его можно установить локально.

1. Убедитесь, что установлена последняя версия Go (не ниже **1.14**). Если она еще не установлена, скачайте ее на сайте [Golang.org](https://golang.org/dl/).

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.5.1**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI требуется для того, чтобы в Go можно было использовать метод `auth.NewAuthorizerFromCLI()` в следующем примере. См. сведения о других параметрах в разделе [Azure SDK для Go — дополнительные сведения о проверке подлинности](https://github.com/Azure/azure-sdk-for-go#more-authentication-details).

1. Проверка подлинности с помощью Azure CLI.

   ```azurecli
   az login
   ```

1. В выбранной вами среде Go установите необходимые пакеты для Azure Resource Graph.

   ```bash
   # Add the Resource Graph package for Go
   go get -u github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph

   # Add the Azure auth package for Go
   go get -u github.com/Azure/go-autorest/autorest/azure/auth
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда пакеты Go добавлены в выбранную среду, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

1. Создайте приложение Go и сохраните следующий исходный код как `argQuery.go`.

   ```Go
   package main
   
   import (
      "fmt"
      "os"
      "context"
      "strconv"
      arg "github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph"
      "github.com/Azure/go-autorest/autorest/azure/auth"
   )
   
   func main() {
       // Get variables from command line arguments
       var query = os.Args[1]
       var subList = os.Args[2:]
   
       // Create and authorize a ResourceGraph client
       argClient := arg.New()
       authorizer, err := auth.NewAuthorizerFromCLI()
       if err == nil {
           argClient.Authorizer = authorizer
       } else {
           fmt.Printf(err.Error())
       }
     
       // Set options
       RequestOptions := arg.QueryRequestOptions {
           ResultFormat: "objectArray",
       }
     
       // Create the query request
       Request := arg.QueryRequest {
           Subscriptions: &subList,
           Query: &query,
           Options: &RequestOptions,
       }
     
       // Run the query and get the results
       var results, queryErr = argClient.Resources(context.Background(), Request)
       if queryErr == nil {
           fmt.Printf("Resources found: " + strconv.FormatInt(*results.TotalRecords, 10) + "\n")
           fmt.Printf("Results: " + fmt.Sprint(results.Data) + "\n")
       } else {
           fmt.Printf(queryErr.Error())
       }
   }
   ```

1. Выполните сборку приложения Go.

   ```bash
   go build argQuery.go
   ```

1. Запустите первый запрос к Azure Resource Graph, используя скомпилированное приложение Go. Замените `<SubID>` идентификатором своей подписки.

   ```bash
   argQuery "Resources | project name, type | limit 5" "<SubID>"
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Измените значение первого параметра на `argQuery` и измените запрос, чтобы выполнить сортировку (`order by`) по свойству **Name**. Замените `<SubID>` идентификатором своей подписки.

   ```bash
   argQuery "Resources | project name, type | limit 5 | order by name asc" "<SubID>"
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Измените значение первого параметра на `argQuery`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить вывод (`limit`) пятью первыми результатами. Замените `<SubID>` идентификатором своей подписки.

   ```bash
   argQuery "Resources | project name, type | order by name asc | limit 5" "<SubID>"
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные пакеты из среды Go, это можно сделать с помощью следующей команды.

```bash
# Remove the installed packages from the Go environment
go clean -i github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph
go clean -i github.com/Azure/go-autorest/autorest/azure/auth
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили пакеты Resource Graph в среду Go и выполнили первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).