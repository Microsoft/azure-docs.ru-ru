---
title: Краткое руководство. Ваш первый запрос Azure CLI
description: В этом кратком руководстве приведены шаги по включению расширения Resource Graph для Azure CLI и выполнению первого запроса.
ms.date: 01/27/2021
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5111f59eb760afda4e206837ca5bdf8bcc201338
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98917847"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-cli"></a>Краткое руководство. Выполните первый запрос графа ресурсов с помощью Azure CLI

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлено ли расширение для [Azure CLI](/cli/azure/). В этом кратком руководстве описывается процесс добавления расширения к установке Azure CLI. Можно использовать расширение Azure CLI, установленное локально или через [Azure Cloud Shell](https://shell.azure.com).

В конце этого процесса будет добавлено расширение к установке Azure CLI по выбору и выполнен первого запрос графа ресурсов.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-extension"></a>Добавление расширения "Граф ресурсов"

Чтобы позволить Azure CLI выполнять запрос графических ресурсов Azure, необходимо добавить расширение. Это расширение работает везде, где может использоваться оболочка Azure CLI, включая [bash на операционной системе Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (как автономно, так и внутри портала), [образ Azure CLI Docker](https://hub.docker.com/_/microsoft-azure-cli) или установленную локально.

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.0.76**). Если она еще не установлена, выполните [эти инструкции](/cli/azure/install-azure-cli-windows).

1. В выбранной вами среде Azure CLI импортируйте ее с помощью следующей команды:

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. Проверьте установку расширения и номер версии (по крайней мере **1.0.0**):

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда расширение для Azure CLI добавлено в выбранную среду, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с **Именем** и **Типом ресурса** для каждого ресурса.

1. Запустите ваш первый запрос графа ресурсов Azure, используя `graph` расширение и `query` команду :

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Run Azure Resource Graph query
   az graph query -q 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Обновите запрос свойством `order by`**Имя**:

   ```azurecli-interactive
   # Run Azure Resource Graph query with 'order by'
   az graph query -q 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Обновите запрос, чтобы сначала выполнить сортировку (`order by`) по свойству **Имя**, а затем ограничить (`limit`) вывод пятью первыми результатами:

   ```azurecli-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   az graph query -q 'Resources | project name, type | order by name asc | limit 5'
   ```

Повторные выполнения последней версии запроса, если ничего не изменяется в самой среде, возвращают стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить расширение "Граф ресурсов" из среды Azure CLI, это можно сделать с помощью следующей команды:

```azurecli-interactive
# Remove the Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили расширение Resource Graph в среду Azure CLI и выполнили первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
