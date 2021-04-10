---
title: Краткое руководство. Создание общего запроса с помощью Azure CLI
description: В этом кратком руководстве приведены шаги по включению расширения Resource Graph для Azure CLI и созданию общего запроса.
ms.date: 02/05/2021
ms.topic: quickstart
ms.openlocfilehash: ec1b010771790339a13777624f04c7bd2db01f11
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99594389"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-cli"></a>Краткое руководство. Создание общего запроса к Resource Graph с помощью Azure CLI

При использовании Azure Resource Graph с [Azure CLI](/cli/azure/) в первую очередь необходимо проверить, установлено ли соответствующее расширение. В этом кратком руководстве описывается процесс добавления расширения к установке Azure CLI. Можно использовать расширение Azure CLI, установленное локально или через [Azure Cloud Shell](https://shell.azure.com).

Выполнив этот процесс, вы добавите расширение к выбранной установке Azure CLI и создадите общий запрос по Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-extension"></a>Добавление расширения "Граф ресурсов"

Чтобы обеспечить работу Azure CLI с Azure Resource Graph, необходимо добавить расширение. Это расширение работает везде, где может использоваться оболочка Azure CLI, включая [bash на операционной системе Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (как автономно, так и внутри портала), [образ Azure CLI Docker](https://hub.docker.com/_/microsoft-azure-cli) или установленную локально.

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.8.0**). Если она еще не установлена, выполните [эти инструкции](/cli/azure/install-azure-cli-windows).

1. В своей среде Azure CLI выполните команду [az extension add](/cli/azure/extension#az_extension_add), чтобы импортировать расширение Resource Graph с помощью следующей команды:

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. Убедитесь, что расширение установлено и имеет подходящий номер версии (не ниже **1.1.0**), выполнив команду [az extension list](/cli/azure/extension#az_extension_list):

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="create-a-resource-graph-shared-query"></a>Создание общего запроса к Resource Graph

Теперь, когда расширение для Azure CLI добавлено в выбранную среду, пора создать общий запрос к Resource Graph. Общий запрос представляет собой объект Azure Resource Manager, которому можно предоставить разрешения на доступ к обозревателю Azure Resource Graph или на выполнение в нем. Этот запрос вычисляет общее количество ресурсов с группировкой по _расположению_.

1. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create), чтобы сохранить общий запрос к Azure Resource Graph. Эта группа ресурсов имеет имя `resource-graph-queries` и расположение `westus2`.

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Create the resource group
   az group create --name 'resource-graph-queries' --location 'westus2'
   ```

1. Создайте общий запрос к Azure Resource Graph с помощью расширения `graph` и команды [az graph shared-query create](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_create).

   ```azurecli-interactive
   # Create the Azure Resource Graph shared query
   az graph shared-query create --name 'Summarize resources by location' \
      --description 'This shared query summarizes resources by location for a pinnable map graphic.' \
      --graph-query 'Resources | summarize count() by location' \
      --resource-group 'resource-graph-queries'
   ```

1. Получите список общих запросов в новой группе ресурсов. Команда [az graph shared-query list](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_list) возвращает массив значений.

   ```azurecli-interactive
   # List all the Azure Resource Graph shared queries in a resource group
   az graph shared-query list --resource-group 'resource-graph-queries'
   ```

1. Чтобы получить в качестве результата только один общий запрос, используйте команду [az graph shared-query show](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_show).

   ```azurecli-interactive
   # Show a specific Azure Resource Graph shared query
   az graph shared-query show --resource-group 'resource-graph-queries' \
      --name 'Summarize resources by location'
   ```

1. Выполните общий запрос в Azure CLI, используя синтаксис `{{shared-query-uri}}` в команде [az graph query](/cli/azure/ext/resource-graph/graph#ext_resource_graph_az_graph_query).
   Для начала скопируйте поле `id` из результата выполнения предыдущей команды `show`. Замените в этом примере текст `shared-query-uri` значением из поля `id`, но сохраните обрамляющие его символы `{{` и `}}`.

   ```azurecli-interactive
   # Run a Azure Resource Graph shared query
   az graph query --graph-query "{{shared-query-uri}}"
   ```

   > [!NOTE]
   > Синтаксис `{{shared-query-uri}}` находится в режиме **предварительной версии функции**.

Также вы можете найти общие запросы к Resource Graph на портале Azure. В строке поиска на портале введите строку "запросы к Resource Graph". Выберите общий запрос. На странице **Обзор** на вкладке **Запрос** отображается сохраненный запрос. Используйте кнопку **Изменить**, чтобы открыть его в [Обозревателе Resource Graph](./first-query-portal.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить общий запрос, группу ресурсов и расширение Resource Graph из среды Azure CLI, воспользуйтесь следующими командами:

- [az graph shared-query delete](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_delete);
- [az group delete](/cli/azure/group#az_group_delete)
- [az extension remove](/cli/azure/extension#az_extension_remove).

```azurecli-interactive
# Delete the Azure Resource Graph shared query
az graph shared-query delete --resource-group 'resource-graph-queries' \
   --name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group without prompting for confirmation
az group delete --resource-group 'resource-graph-queries' --yes

# Remove the Azure Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили расширение Resource Graph в среду Azure CLI и создали общий запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).