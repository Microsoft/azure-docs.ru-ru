---
title: Краткое руководство. Ваш первый запрос Python
description: В этом кратком руководстве приведены шаги по включению библиотеки Resource Graph для Python и выполнению первого запроса.
ms.date: 01/27/2021
ms.topic: quickstart
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: 0bae0639e3f9913bc47b18fbc0b1d3d9ef1ad324
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533024"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-python"></a>Краткое руководство. Первый запрос Resource Graph с помощью Python

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлены ли необходимые библиотеки для Python. В этом кратком руководстве описывается процесс добавления библиотек к установке Python.

В конце этого процесса будут добавлены библиотеки к установке Python и выполнен запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-library"></a>Добавление библиотеки Resource Graph

Чтобы позволить Python запрашивать Azure Resource Graph, необходимо добавить библиотеку. Эта библиотека работает во всех средах, где можно использовать Python, в том числе в [оболочке Bash для Windows 10](/windows/wsl/install-win10). Также ее можно установить локально.

1. Убедитесь, что установлена последняя версия Python (не ниже **3.8**). Если он еще не установлен, скачайте его на сайте [Python.org](https://www.python.org/downloads/).

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.5.1**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI требуется для того, чтобы с помощью Python можно было использовать **проверки подлинности на основе CLI** в следующих примерах. Дополнительные сведения о других параметрах см. в статье о [проверке подлинности с помощью библиотек управления Azure для Python](/azure/developer/python/azure-sdk-authenticate).

1. Проверка подлинности с помощью Azure CLI.

   ```azurecli
   az login
   ```

1. В выбранной вами среде Python установите необходимые библиотеки для Azure Resource Graph:

   ```bash
   # Add the Resource Graph library for Python
   pip install azure-mgmt-resourcegraph

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Если Python установлен для всех пользователей, эти команды необходимо запустить из консоли с повышенными привилегиями.

1. Убедитесь, что библиотеки установлены. `azure-mgmt-resourcegraph` должна быть версии **2.0.0** или более поздней, `azure-mgmt-resource` ⁠— **9.0.0** или более поздней, а `azure-cli-core` ⁠— **2.5.0** или более поздней.

   ```bash
   # Check each installed library
   pip show azure-mgmt-resourcegraph azure-mgmt-resource azure-cli-core
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда библиотеки Python добавлены в выбранную среду, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Name** и **Resource Type** для каждого ресурса.

1. Запустите ваш первый запрос Azure Resource Graph, используя установленные библиотеки и метод `resources`:

   ```python
   # Import Azure Resource Graph library
   import azure.mgmt.resourcegraph as arg
   
   # Import specific methods and models from other libraries
   from azure.common.credentials import get_azure_cli_credentials
   from azure.common.client_factory import get_client_from_cli_profile
   from azure.mgmt.resource import SubscriptionClient
   
   # Wrap all the work in a function
   def getresources( strQuery ):
       # Get your credentials from Azure CLI (development only!) and get your subscription list
       subsClient = get_client_from_cli_profile(SubscriptionClient)
       subsRaw = []
       for sub in subsClient.subscriptions.list():
           subsRaw.append(sub.as_dict())
       subsList = []
       for sub in subsRaw:
           subsList.append(sub.get('subscription_id'))
       
       # Create Azure Resource Graph client and set options
       argClient = get_client_from_cli_profile(arg.ResourceGraphClient)
       argQueryOptions = arg.models.QueryRequestOptions(result_format="objectArray")
       
       # Create query
       argQuery = arg.models.QueryRequest(subscriptions=subsList, query=strQuery, options=argQueryOptions)
       
       # Run query
       argResults = argClient.resources(argQuery)
   
       # Show Python object
       print(argResults)
   
   getresources("Resources | project name, type | limit 5")
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Обновите запрос к `getresources` и измените запрос на `order by` для свойства **Name**:

   ```python
   getresources("Resources | project name, type | limit 5 | order by name asc")
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Обновите вызов `getresources`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить (`limit`) вывод пятью первыми результатами.

   ```python
   getresources("Resources | project name, type | order by name asc | limit 5")
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные библиотеки из среды Python, это можно сделать с помощью следующей команды:

```bash
# Remove the installed libraries from the Python environment
pip uninstall azure-mgmt-resourcegraph azure-mgmt-resource azure-cli-core
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили библиотеки Resource Graph в среду Python и выполнили первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
