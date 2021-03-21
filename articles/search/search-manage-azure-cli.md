---
title: Azure CLI сценариев, использующих модуль поиска AZ
titleSuffix: Azure Cognitive Search
description: Создание и Настройка службы Когнитивный поиск Azure с Azure CLI. Вы можете масштабировать службу по мере увеличения или уменьшения масштаба, управлять ключами API администрирования и запросов, а также запрашивать сведения о системе.
manager: luisca
author: DerekLegenzoff
ms.author: delegenz
ms.service: cognitive-search
ms.devlang: azurecli
ms.topic: conceptual
ms.date: 02/17/2021
ms.openlocfilehash: ee6b0e1b745e86c72843af88c0f6d17f91512e15
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102176762"
---
# <a name="manage-your-azure-cognitive-search-service-with-the-azure-cli"></a>Управление службой Когнитивный поиск Azure с помощью Azure CLI
> [!div class="op_single_selector"]
> * [Портал](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> * [Azure CLI](search-manage-azure-cli.md)
> * [REST API](/rest/api/searchmanagement/)
> * [Пакет SDK для .NET](/dotnet/api/microsoft.azure.management.search)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)

Вы можете выполнять команды Azure CLI и сценарии в Windows, macOS, Linux или в [Azure Cloud Shell](../cloud-shell/overview.md) для создания и настройки когнитивный Поиск Azure. Модуль [**поиска AZ**](/cli/azure/search) расширяет [Azure CLI](/cli/) с полным контролем четности для API- [интерфейсов RESTful управления поиском](/rest/api/searchmanagement) и возможностью выполнять следующие задачи:

> [!div class="checklist"]
> * [Перечисление служб поиска в подписке](#list-search-services)
> * [Возврат сведений о службе](#get-search-service-information)
> * [Создание или удаление службы](#create-or-delete-a-service)
> * [Создание службы с закрытой конечной точкой](#create-a-service-with-a-private-endpoint)
> * [Повторное создание ключей API администратора](#regenerate-admin-keys)
> * [Создание или удаление ключей API запроса](#create-or-delete-query-keys)
> * [Увеличение или уменьшение масштаба с помощью реплик и секций](#scale-replicas-and-partitions)
> * [Создание общего ресурса частной ссылки](#create-a-shared-private-link-resource)

Иногда вопросы задаются *не* в приведенном выше списке. В настоящее время нельзя использовать либо модуль **поиска AZ** , либо REST API управления для изменения имени сервера, региона или уровня. Выделенные ресурсы выделяются при создании службы. Таким образом, для изменения базового оборудования (расположения или типа узла) требуется новая служба. Аналогично, отсутствуют средства или интерфейсы API для передачи содержимого, например индекса, из одной службы в другую.

В рамках службы создание и управление содержимым осуществляется с помощью [Служба поиска REST API](/rest/api/searchservice/) или [пакета SDK для .NET](/dotnet/api/overview/azure/search.documents-readme). Хотя для содержимого нет выделенных команд PowerShell, можно написать скрипты, которые вызывают интерфейсы API RESTFUL или .NET для создания и загрузки индексов.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

<a name="list-search-services"></a>

## <a name="list-services-in-a-subscription"></a>Перечисление служб в подписке

Следующие команды относятся к [**ресурсу AZ**](/cli/azure/resource), возвращая сведения о существующих ресурсах и службах, уже подготовленных в вашей подписке. Если вы не уверены, сколько служб поиска уже создано, эти команды возвращают эту информацию, экономя на портале.

Первая команда возвращает все службы поиска.

```azurecli-interactive
az resource list --resource-type Microsoft.Search/searchServices --output table
```

В списке служб верните сведения о конкретном ресурсе.

```azurecli-interactive
az resource list --name <service-name>
```

## <a name="list-all-az-search-commands"></a>Вывод списка всех команд AZ Search

Сведения о подгруппах и командах, доступных в [**поиске AZ**](/cli/azure/search) , можно просмотреть в интерфейсе командной строки. Кроме того, можно ознакомиться с [документацией](/cli/azure/search).

Чтобы просмотреть подгруппы, доступные в `az search` , выполните следующую команду.

```azurecli-interactive
az search --help
```

Ответ должен выглядеть примерно так, как показано ниже.

```
Group
    az search : Manage Azure Search services, admin keys and query keys.
        WARNING: This command group is in preview and under development. Reference and support
        levels: https://aka.ms/CLI_refstatus
Subgroups:
    admin-key                    : Manage Azure Search admin keys.
    private-endpoint-connection  : Manage Azure Search private endpoint connections.
    private-link-resource        : Manage Azure Search private link resources.
    query-key                    : Manage Azure Search query keys.
    service                      : Manage Azure Search services.
    shared-private-link-resource : Manage Azure Search shared private link resources.

For more specific examples, use: az find "az search"
```

В каждой подгруппе доступно несколько команд. Чтобы просмотреть доступные команды для `service` подгруппы, запустите следующую строку.

```azurecli-interactive
az search service --help
```

Можно также просмотреть аргументы, доступные для конкретной команды.

```azurecli-interactive
az search service create --help
```

## <a name="get-search-service-information"></a>Получение сведений о службе поиска

Если известно, что группа ресурсов содержит службу поиска, выполните команду [**AZ Search Service**](/cli/azure/search/service#az_search_service_show) , чтобы вернуть определение службы, включая имя, регион, уровень, реплику и число разделов.

```azurecli-interactive
az search service show --name <service-name> --resource-group <resource-group-name>
```

## <a name="create-or-delete-a-service"></a>Создание или удаление службы

Чтобы [создать новую службу поиска](search-create-service-portal.md), используйте команду [**AZ Search Service Create**](/cli/azure/search/service#az_search_service_show) .

```azurecli-interactive
az search service create \
    --name <service-name> \
    --resource-group <resource-group-name> \
    --sku Standard \
    --partition-count 1 \
    --replica-count 1
``` 

Результаты должны выглядеть примерно так, как показано ниже.

```
{
  "hostingMode": "default",
  "id": "/subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp",
  "identity": null,
  "location": "West US",
  "name": "my-demo-searchapp",
  "networkRuleSet": {
    "bypass": "None",
    "ipRules": []
  },
  "partitionCount": 1,
  "privateEndpointConnections": [],
  "provisioningState": "succeeded",
  "publicNetworkAccess": "Enabled",
  "replicaCount": 1,
  "resourceGroup": "demo-westus",
  "sharedPrivateLinkResources": [],
  "sku": {
    "name": "standard"
  },
  "status": "running",
  "statusDetails": "",
  "tags": null,
  "type": "Microsoft.Search/searchServices"
}
```

### <a name="create-a-service-with-ip-rules"></a>Создание службы с правилами IP-адресов

В зависимости от требований безопасности может потребоваться создать службу поиска с [настроенным брандмауэром IP](service-configure-firewall.md). Для этого передайте аргументу общедоступные IP-адреса (v4) или диапазоны CIDR, `ip-rules` как показано ниже. Правила должны быть разделены запятыми ( `,` ) или точкой с запятой ( `;` ).

```azurecli-interactive
az search service create \
    --name <service-name> \
    --resource-group <resource-group-name> \
    --sku Standard \
    --partition-count 1 \
    --replica-count 1 \
    --ip-rules "55.5.63.73;52.228.215.197;101.37.221.205"
```

### <a name="create-a-service-with-a-system-assigned-managed-identity"></a>Создание службы с управляемым удостоверением, назначенным системой

В некоторых случаях, например при [использовании управляемого удостоверения для подключения к источнику данных](search-howto-managed-identities-storage.md), необходимо включить [управляемое удостоверение, назначенное системой](../active-directory/managed-identities-azure-resources/overview.md). Это можно сделать, добавив `--identity-type SystemAssigned` к команде.

```azurecli-interactive
az search service create \
    --name <service-name> \
    --resource-group <resource-group-name> \
    --sku Standard \
    --partition-count 1 \
    --replica-count 1 \
    --identity-type SystemAssigned
```

## <a name="create-a-service-with-a-private-endpoint"></a>Создание службы с закрытой конечной точкой

[Частные конечные точки](../private-link/private-endpoint-overview.md) для Azure когнитивный Поиск позволяют клиенту в виртуальной сети безопасно получать доступ к данным в индексе поиска по [закрытой ссылке](../private-link/private-link-overview.md). Частная конечная точка использует IP-адрес из [адресного пространства виртуальной сети](../virtual-network/private-ip-addresses.md) для службы поиска. Сетевой трафик между клиентом и службой поиска проходит по виртуальной сети и частной связи в магистральной сети Майкрософт, что устраняет уязвимость в общедоступном Интернете. Дополнительные сведения см. в документации по [созданию частной конечной точки для Azure когнитивный Поиск](service-create-private-endpoint.md)

В следующем примере показано, как создать службу поиска с закрытой конечной точкой.

Сначала разверните службу поиска со `PublicNetworkAccess` значением `Disabled` .

```azurecli-interactive
az search service create \
    --name <service-name> \
    --resource-group <resource-group-name> \
    --sku Standard \
    --partition-count 1 \
    --replica-count 1 \
    --public-access Disabled
```

Затем создайте виртуальную сеть и закрытую конечную точку.

```azurecli-interactive
# Create the virtual network
az network vnet create \
    --resource-group <resource-group-name> \
    --location "West US" \
    --name <virtual-network-name> \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name <subnet-name> \
    --subnet-prefixes 10.1.0.0/24

# Update the subnet to disable private endpoint network policies
az network vnet subnet update \
    --name <subnet-name> \
    --resource-group <resource-group-name> \
    --vnet-name <virtual-network-name> \
    --disable-private-endpoint-network-policies true

# Get the id of the search service
id=$(az search service show \
    --resource-group <resource-group-name> \
    --name <service-name> \
    --query [id] \
    --output tsv)

# Create the private endpoint
az network private-endpoint create \
    --name <private-endpoint-name> \
    --resource-group <resource-group-name> \
    --vnet-name <virtual-network-name> \
    --subnet <subnet-name> \
    --private-connection-resource-id $id \
    --group-id searchService \
    --connection-name <private-link-connection-name>  
```

Наконец, создайте частную зону DNS. 

```azurecli-interactive
## Create private DNS zone
az network private-dns zone create \
    --resource-group <resource-group-name> \
    --name "privatelink.search.windows.net"

## Create DNS network link
az network private-dns link vnet create \
    --resource-group <resource-group-name> \
    --zone-name "privatelink.search.windows.net" \
    --name "myLink" \
    --virtual-network <virtual-network-name> \
    --registration-enabled false

## Create DNS zone group
az network private-endpoint dns-zone-group create \
   --resource-group <resource-group-name>\
   --endpoint-name <private-endpoint-name> \
   --name "myZoneGroup" \
   --private-dns-zone "privatelink.search.windows.net" \
   --zone-name "searchServiceZone"
```

Дополнительные сведения о создании частных конечных точек в PowerShell см. в этом [кратком руководстве по этой частной ссылке](https://docs.microsoft.com/azure/private-link/create-private-endpoint-cli) .

### <a name="manage-private-endpoint-connections"></a>Управление подключениями к частным конечным точкам

Кроме создания подключения к частной конечной точке, можно также `show` `update` и `delete` подключение.

Чтобы получить подключение к частной конечной точке и просмотреть его состояние, используйте команду [**AZ Search частный-Endpoint-Connection Показать**](/cli/azure/search/private-endpoint-connection#az_search_private_endpoint_connection_show).

```azurecli-interactive
az search private-endpoint-connection show \
    --name <pe-connection-name> \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> 
```

Чтобы обновить подключение, используйте команду [**AZ Search Private-Endpoint-Connection**](/cli/azure/search/private-endpoint-connection#az_search_private_endpoint_connection_update). В следующем примере задается отклонение подключения к частной конечной точке:

```azurecli-interactive
az search private-endpoint-connection show \
    --name <pe-connection-name> \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> 
    --status Rejected \
    --description "Rejected" \
    --actions-required "Please fix XYZ"
```

Чтобы удалить подключение к частной конечной точке, используйте команду [**AZ Search Private-Endpoint-Connection Delete**](/cli/azure/search/private-endpoint-connection#az_search_private_endpoint_connection_delete).

```azurecli-interactive
az search private-endpoint-connection delete \
    --name <pe-connection-name> \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> 
```

## <a name="regenerate-admin-keys"></a>Повторное создание ключей администратора

Чтобы выполнить откат [ключей API](search-security-api-keys.md)администратора, используйте команду [**AZ Search Admin-Key продление**](/cli/azure/search/admin-key#az_search_admin_key_renew). Для каждой службы создаются два ключа администратора для доступа с проверкой подлинности. Для каждого запроса требуются ключи. Оба ключа администратора функционально эквивалентны, они предоставляют полный доступ для записи службе поиска с возможностью получения любых сведений, а также создания и удаления любого объекта. Существует два ключа, которые можно использовать при замене другого. 

Его можно создать только по одному, указанному в качестве `primary` `secondary` ключа или. При использовании непрерывной службы не забудьте обновить весь код клиента для использования вторичного ключа при последовательном переходе по первичному ключу. Старайтесь не менять ключи, пока операции находятся в полете.

Как вы можете ожидать, при повторном создании ключей без обновления клиентского кода запросы, использующие старый ключ, завершатся ошибкой. Повторное создание всех новых ключей не приводит к окончательной блокировке вашей службы, и вы по-прежнему можете обращаться к службе через портал. После повторного создания первичного и вторичного ключей можно обновить код клиента, чтобы использовать новые ключи, и операции будут возобновлены соответствующим образом.

Значения ключей API создаются службой. Вы не можете предоставить настраиваемый ключ для Azure Когнитивный поиск. Точно так же не существует определяемого пользователем имени для ключей API администратора. Ссылки на ключ являются фиксированными строками: `primary` или `secondary` . 

```azurecli-interactive
az search admin-key renew \
    --resource-group <resource-group-name> \
    --service-name <search-service-name> \
    --key-kind primary
```

Результаты должны выглядеть следующим образом. Возвращаются оба ключа, несмотря на то, что вы изменяете их только один раз.

```   
{
  "primaryKey": <alphanumeric-guid>,
  "secondaryKey": <alphanumeric-guid>  
}
```

## <a name="create-or-delete-query-keys"></a>Создание или удаление ключей запроса

Чтобы создать [ключи API](search-security-api-keys.md) запросов для доступа только для чтения из клиентских приложений к индексу Azure когнитивный Поиск, используйте команду [**AZ Search Query-Key Create**](/cli/azure/search/query-key#az_search_query_key_create). Ключи запроса используются для проверки подлинности определенного индекса с целью получения результатов поиска. Ключи запроса не предоставляют доступ только для чтения к другим элементам службы, таким как индекс, источник данных или индексатор.

Вы не можете предоставить ключ для использования Когнитивный поиск Azure. Ключи API создаются службой.

```azurecli-interactive
az search query-key create \
    --name myQueryKey \
    --resource-group <resource-group-name> \
    --service-name <search-service-name>
```

## <a name="scale-replicas-and-partitions"></a>Масштабирование реплик и секций

Чтобы [увеличить или уменьшить реплики и секции,](search-capacity-planning.md) используйте [**AZ Search Service Update**](/cli/azure/search/service#az_search_service_update). При увеличении количества реплик или секций добавляется к счету, у которого есть фиксированная и переменная плата. При наличии временной потребности в дополнительной вычислительной мощности можно увеличить реплики и секции для обработки рабочей нагрузки. В области мониторинг на странице Обзор портала находятся плитки о задержке запросов, запросах в секунду и регулированиях, указывающих, достаточно ли текущих мощностей.

Добавление или удаление перебора может занять некоторое время. Корректировки производительности выполняются в фоновом режиме, что позволяет продолжать работу существующих рабочих нагрузок. Дополнительная емкость используется для входящих запросов, как только она будет готова, без необходимости дополнительной настройки. 

Удаление емкости может нарушить работу. Остановка всех заданий индексирования и индексирования перед уменьшением емкости рекомендуется, чтобы избежать отброшенных запросов. Если это нецелесообразно, можно потребовать более постепенного снижения емкости по одной реплике и секции, пока не будут достигнуты новые целевые уровни.

После отправки команды нет возможности завершить ее с самого посредине. Перед изменением счетчиков необходимо будет дождаться завершения команды.

```azurecli-interactive
az search service update \
    --name <service-name> \
    --resource-group <resource-group-name> \
    --partition-count 6 \
    --replica-count 6
```

Кроме обновления счетчиков реплик и секций, можно также обновить `ip-rules` , `public-access` и `identity-type` .

## <a name="create-a-shared-private-link-resource"></a>Создание общего ресурса частной ссылки

Частные конечные точки защищенных ресурсов, созданные с помощью Azure Когнитивный поиск API, называются *общими ресурсами частной ссылки*. Это связано с тем, что вы используете общий доступ к ресурсу, например к учетной записи хранения, интегрированной со [службой частной связи Azure](https://azure.microsoft.com/services/private-link/).

Если вы используете индексатор для индексирования данных в Когнитивный поиск Azure, а источник данных находится в частной сети, можно создать исходящее [Подключение к частной конечной точке](../private-link/private-endpoint-overview.md) для достижения данных.

Полный список ресурсов Azure, для которых можно создать исходящие частные конечные точки из Когнитивный поиск Azure, можно найти [здесь](search-indexer-howto-access-private.md#shared-private-link-resources-management-apis) вместе со связанными значениями **идентификаторов групп** .

Чтобы создать ресурс общей частной ссылки, используйте команду [**AZ Search Shared-Private-Link-Resource Create**](/cli/azure/search/shared-private-link-resource#az_search_shared_private_link_resource_list). Помните, что перед выполнением этой команды может потребоваться определенная конфигурация для источника данных.

```azurecli-interactive
az search shared-private-link-resource create \
    --name <spl-name> \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> \
    --group-id blob \
    --resource-id "/subscriptions/<alphanumeric-subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/myBlobStorage"  \
    --request-message "Please approve" 
```


Чтобы получить общие ресурсы частной ссылки и просмотреть их состояние, используйте команду [**AZ Search Shared-Private-Link-Resource List**](/cli/azure/search/shared-private-link-resource#az_search_shared_private_link_resource_list).

```azurecli-interactive
az search shared-private-link-resource list \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> 
```

Прежде чем можно будет использовать это подключение, необходимо одобрить его с помощью следующей команды. Идентификатор подключения к частной конечной точке должен быть получен из дочернего ресурса. В этом случае мы получаем идентификатор подключения из AZ Storage.

```azurecli-interactive
id = (az storage account show -n myBlobStorage --query "privateEndpointConnections[0].id")

az network private-endpoint-connection approve --id $id
```

Чтобы удалить ресурс общей частной ссылки, выполните команду [**AZ Search Shared-Private-Link-Resource Delete**](/cli/azure/search/shared-private-link-resource#az_search_shared_private_link_resource_delete).

```azurecli-interactive
az search shared-private-link-resource delete \
    --name <spl-name> \
    --service-name <search-service-name> \
    --resource-group <resource-group-name> 
```

Дополнительные сведения о настройке общих ресурсов частной связи см. в документации по [созданию подключений индексатора через закрытую конечную точку](search-indexer-howto-access-private.md).

## <a name="next-steps"></a>Дальнейшие действия

Создайте [индекс](search-what-is-an-index.md), [запросите индекс](search-query-overview.md) с помощью портала, API-интерфейсов и пакета SDK для .NET.

* [Создание индекса службы "Когнитивный поиск Azure" на портале Azure](search-get-started-portal.md)
* [Настройка индексатора для загрузки данных из других служб](search-indexer-overview.md)
* [Запрос индекса Когнитивный поиск Azure с помощью обозревателя поиска в портал Azure](search-explorer.md)
* [Использование Когнитивный поиск Azure в .NET](search-howto-dotnet-sdk.md)
