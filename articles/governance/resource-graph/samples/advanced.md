---
title: Примеры расширенных запросов
description: С помощью Azure Resource Graph можно выполнять расширенные запросы, такие как работа со столбцами, вывод списка используемых тегов и сопоставление ресурсов с регулярными выражениями.
ms.date: 03/23/2021
ms.topic: sample
ms.openlocfilehash: c6a140b0392affea252e05d63055232532305c75
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104949861"
---
# <a name="advanced-resource-graph-query-samples"></a>Примеры расширенных запросов к Resource Graph

Первым шагом на пути к пониманию запросов к Azure Resource Graph является общее понимание [языка запросов](../concepts/query-language.md). Если вы еще не знакомы с [обозревателем данных Azure](/azure/data-explorer/data-explorer-overview), рекомендуется просмотреть основные сведения, чтобы понять, как составлять запросы к ресурсам, которые вы ищете.

Мы рассмотрим следующие продвинутые запросы:

- [Отображение версии API для каждого типа ресурсов](#apiversion)
- [Получение сведений о емкости и производительности масштабируемого набора виртуальных машин](#vmss-capacity)
- [Удаление столбцов из результатов](#remove-column)
- [Вывести список всех названий тегов](#list-all-tags)
- [Виртуальные машины, сопоставленные по регулярному выражению](#vm-regex)
- [Список Cosmos DB с конкретным указанием расположений записи](#mvexpand-cosmosdb)
- [Хранилища ключей с именем подписки](#join)
- [Список баз данных SQL и их эластичных пулов](#join-sql)
- [Список виртуальных машин с их сетевым интерфейсом и общедоступным IP-адресом](#join-vmpip)
- [Список всех расширений, установленных на виртуальной машине](#join-vmextension)
- [Поиск учетных записей хранения с указанным тегом в группе ресурсов](#join-findstoragetag)
- [Объединение результатов из двух запросов в один результат](#unionresults)
- [Суммирование виртуальных машин по расширенному свойству состояния питания](#vm-powerstate)
- [Число несоответствующих назначений гостевой конфигурации](#count-gcnoncompliant)
- [Сведения о запросах для отчетов о назначениях гостевой конфигурации](#query-gcreports)
- [Поиск всех причин, по которым компьютер не соответствует требованиям для назначений гостевой конфигурации](#query-gcmachinedetails)

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.

## <a name="language-support"></a>Поддержка языков

Azure CLI (с помощью расширения) и Azure PowerShell (с помощью модуля) поддерживают Azure Resource Graph. Перед выполнением любого из следующих запросов убедитесь, что среда готова. В разделах об [Azure CLI](../first-query-azurecli.md#add-the-resource-graph-extension) и [Azure PowerShell](../first-query-powershell.md#add-the-resource-graph-module) приведены инструкции по установке и проверке выбранной вами среды оболочки.

## <a name="show-resource-types-and-api-versions"></a><a name="apiversion"></a>Отображение типов ресурсов и версий API

Resource Graph в основном использует самую новую не предварительную версию API поставщика ресурсов для получения (`GET`) свойств ресурсов во время обновления. В некоторых случаях используемая версия API была переопределена, чтобы предоставить больше текущих или широко используемых свойств в результатах. Следующий запрос содержит сведения о версии API, используемой для сбора свойств каждого типа ресурсов:

```kusto
Resources
| distinct type, apiVersion
| where isnotnull(apiVersion)
| order by type asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | distinct type, apiVersion | where isnotnull(apiVersion) | order by type asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | distinct type, apiVersion | where isnotnull(apiVersion) | order by type asc"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.cn</a>

---

## <a name="get-virtual-machine-scale-set-capacity-and-size"></a><a name="vmss-capacity"></a>Получение сведений о емкости и производительности масштабируемого набора виртуальных машин

Этот запрос выполняет поиск ресурсов масштабируемого набора виртуальных машин и возвращает разные сведения, включая размер виртуальной машины и емкость масштабируемого набора. В запросе используется функция `toint()` для приведения значения емкости в число для сортировки. Наконец, столбцы переименовываются в пользовательские именованные свойства.

```kusto
Resources
| where type=~ 'microsoft.compute/virtualmachinescalesets'
| where name contains 'contoso'
| project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name
| order by Capacity desc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.cn</a>

---

## <a name="remove-columns-from-results"></a><a name="remove-column"></a>Удаление столбцов из результатов

Следующий запрос использует `summarize` для подсчета ресурсов по подписке, `join` — для их объединения с данными о подписке из таблицы _ResourceContainers_, а затем `project-away` — для удаления некоторых столбцов.

```kusto
Resources
| summarize resourceCount=count() by subscriptionId
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project-away subscriptionId, subscriptionId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | summarize resourceCount=count() by subscriptionId | join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId| project-away subscriptionId, subscriptionId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | summarize resourceCount=count() by subscriptionId | join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId| project-away subscriptionId, subscriptionId1"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.cn</a>

---

## <a name="list-all-tag-names"></a><a name="list-all-tags"></a>Вывести список всех названий тегов

Этот запрос начинается с тега и строит объект JSON, перечисляющий все уникальные имена тегов и их соответствующие типы.

```kusto
Resources
| project tags
| summarize buildschema(tags)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | project tags | summarize buildschema(tags)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project tags | summarize buildschema(tags)"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.cn</a>

---

## <a name="virtual-machines-matched-by-regex"></a><a name="vm-regex"></a>Виртуальные машины, сопоставленные по регулярному выражению

Этот запрос выполняет поиск виртуальных машин, которые соответствуют [регулярному выражению](/dotnet/standard/base-types/regular-expression-language-quick-reference) (известному как _регулярное выражение_). **matches regex\@** позволяет определить регулярное выражение для сопоставления, которое равно `^Contoso(.*)[0-9]+$`.
Дополнительные сведения об определении регулярного выражения см. в разделе:

- `^` — Сопоставление должно начаться в начале строки.
- `Contoso` — Строка с учетом регистра.
- `(.*)` — Сопоставление части выражения:
  - `.` — Сопоставляется с любым единичным символом (кроме новой строки).
  - `*` — Сопоставляется с предыдущим элементом ноль или более раз.
- `[0-9]` — Сопоставление группы символ для чисел от 0 до 9.
- `+` — Сопоставляется с предыдущим элементом один или более раз.
- `$` — Сопоставление предыдущего элемента должно находиться в конце строки.

После сопоставления по имени запрос проектирует имя и порядок по возрастанию имени.

```kusto
Resources
| where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$'
| project name
| order by name asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.cn</a>

---

## <a name="list-cosmos-db-with-specific-write-locations"></a><a name="mvexpand-cosmosdb"></a>Список Cosmos DB с конкретным указанием расположений записи

Следующий запрос ограничивает ресурсы Cosmos DB, использует `mv-expand` для расширения пакета свойств для **properties.writeLocations**, а затем проецирует конкретные поля и ограничивает результаты до значения **properties.writeLocations.locationName**, совпадающего с "Восточная часть США" или "Западная часть США".

```kusto
Resources
| where type =~ 'microsoft.documentdb/databaseaccounts'
| project id, name, writeLocations = (properties.writeLocations)
| mv-expand writeLocations
| project id, name, writeLocation = tostring(writeLocations.locationName)
| where writeLocation in ('East US', 'West US')
| summarize by id, name
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.documentdb/databaseaccounts' | project id, name, writeLocations = (properties.writeLocations) | mv-expand writeLocations | project id, name, writeLocation = tostring(writeLocations.locationName) | where writeLocation in ('East US', 'West US') | summarize by id, name"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.documentdb/databaseaccounts' | project id, name, writeLocations = (properties.writeLocations) | mv-expand writeLocations | project id, name, writeLocation = tostring(writeLocations.locationName) | where writeLocation in ('East US', 'West US') | summarize by id, name"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.cn</a>

---

## <a name="key-vaults-with-subscription-name"></a><a name="join"></a>Хранилища ключей с именем подписки

В следующем запросе показано сложное использование `join` с **kind** в качестве _leftouter_. Запрос ограничивает объединенную таблицу до ресурсов подписки, а `project` включает только исходное поле _subscriptionId_ и поле _name_, переименованное в _SubName_. Переименование полей позволяет избежать `join`, добавляя его как _name1_, так как поле уже существует в _resources_. Исходная таблица фильтруется с `where`, а следующая `project` включает столбцы из обеих таблиц. Результатом запроса являются все хранилища ключей, отображающие тип, имя хранилища ключей и имя подписки, в которой оно находится.

```kusto
Resources
| join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| where type == 'microsoft.keyvault/vaults'
| project type, name, SubName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.cn</a>

---

## <a name="list-sql-databases-and-their-elastic-pools"></a><a name="join-sql"></a>Список баз данных SQL и их эластичных пулов

В следующем запросе для объединения ресурсов Базы данных SQL и связанных с ними эластичных пулов (если имеются) используется **leftouter** `join`.

```kusto
Resources
| where type =~ 'microsoft.sql/servers/databases'
| project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId))
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.sql/servers/elasticpools'
    | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state)
on elasticPoolId
| project-away elasticPoolId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.sql/servers/databases' | project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId)) | join kind=leftouter ( Resources | where type =~ 'microsoft.sql/servers/elasticpools' | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state) on elasticPoolId | project-away elasticPoolId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.sql/servers/databases' | project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId)) | join kind=leftouter ( Resources | where type =~ 'microsoft.sql/servers/elasticpools' | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state) on elasticPoolId | project-away elasticPoolId1"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.cn</a>

---

## <a name="list-virtual-machines-with-their-network-interface-and-public-ip"></a><a name="join-vmpip"></a>Список виртуальных машин с их сетевым интерфейсом и общедоступным IP-адресом

В этом запросе для объединения виртуальных машин, которые созданы с помощью модели развертывания Resource Manager, связанных с ними сетевых интерфейсов и любых общедоступных IP-адресов, связанных с этими сетевыми интерфейсами, используются две команды **leftouter** `join`.

```kusto
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend nics=array_length(properties.networkProfile.networkInterfaces) 
| mv-expand nic=properties.networkProfile.networkInterfaces 
| where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) 
| project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) 
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/networkinterfaces'
    | extend ipConfigsCount=array_length(properties.ipConfigurations) 
    | mv-expand ipconfig=properties.ipConfigurations 
    | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true'
    | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id))
on nicId
| project-away nicId1
| summarize by vmId, vmName, vmSize, nicId, publicIpId
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/publicipaddresses'
    | project publicIpId = id, publicIpAddress = properties.ipAddress)
on publicIpId
| project-away publicIpId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.compute/virtualmachines' | extend nics=array_length(properties.networkProfile.networkInterfaces) | mv-expand nic=properties.networkProfile.networkInterfaces | where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) | project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) | join kind=leftouter ( Resources | where type =~ 'microsoft.network/networkinterfaces' | extend ipConfigsCount=array_length(properties.ipConfigurations) | mv-expand ipconfig=properties.ipConfigurations | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true' | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)) on nicId | project-away nicId1 | summarize by vmId, vmName, vmSize, nicId, publicIpId | join kind=leftouter ( Resources | where type =~ 'microsoft.network/publicipaddresses' | project publicIpId = id, publicIpAddress = properties.ipAddress) on publicIpId | project-away publicIpId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.compute/virtualmachines' | extend nics=array_length(properties.networkProfile.networkInterfaces) | mv-expand nic=properties.networkProfile.networkInterfaces | where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) | project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) | join kind=leftouter ( Resources | where type =~ 'microsoft.network/networkinterfaces' | extend ipConfigsCount=array_length(properties.ipConfigurations) | mv-expand ipconfig=properties.ipConfigurations | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true' | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)) on nicId | project-away nicId1 | summarize by vmId, vmName, vmSize, nicId, publicIpId | join kind=leftouter ( Resources | where type =~ 'microsoft.network/publicipaddresses' | project publicIpId = id, publicIpAddress = properties.ipAddress) on publicIpId | project-away publicIpId1"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.cn</a>

---

## <a name="list-all-extensions-installed-on-a-virtual-machine"></a><a name="join-vmextension"></a>Список всех расширений, установленных на виртуальной машине

Сначала этот запрос применяет `extend` к типу ресурса виртуальных машин, чтобы получить идентификатор, указанный прописными буквами (`toupper()`), имя и тип операционной системы, а также размер виртуальной машины.
Получение идентификатора ресурса, указанного прописными буквами, очень удобно при подготовке к объединению с другим свойством. Затем этот запрос применяет `join` со значением _leftouter_ для параметра **kind**, чтобы получить список расширений виртуальной машины по сопоставлению с `substring` идентификатора расширения, указанного прописными буквами. Сегмент идентификатора перед строкой "/extensions/\<ExtensionName\>" имеет тот же формат, что и идентификатор виртуальных машин, поэтому мы используем это свойство для `join`. Затем применяется `summarize` в сочетании с `make_list` для имени расширения виртуальной машины, чтобы объединить в одно свойство массива все имена расширений, у которых совпадают значения _ID_, _OSName_, _OSType_ и _VMSize_. Наконец, мы применим `order by` к значению _OSName_ (строчными буквами) с параметром **asc**. По умолчанию `order by` использует порядок по убыванию.

```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| extend
    JoinID = toupper(id),
    OSName = tostring(properties.osProfile.computerName),
    OSType = tostring(properties.storageProfile.osDisk.osType),
    VMSize = tostring(properties.hardwareProfile.vmSize)
| join kind=leftouter(
    Resources
    | where type == 'microsoft.compute/virtualmachines/extensions'
    | extend 
        VMId = toupper(substring(id, 0, indexof(id, '/extensions'))),
        ExtensionName = name
) on $left.JoinID == $right.VMId
| summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize
| order by tolower(OSName) asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' | extend JoinID = toupper(id), OSName = tostring(properties.osProfile.computerName), OSType = tostring(properties.storageProfile.osDisk.osType), VMSize = tostring(properties.hardwareProfile.vmSize) | join kind=leftouter( Resources | where type == 'microsoft.compute/virtualmachines/extensions' | extend VMId = toupper(substring(id, 0, indexof(id, '/extensions'))), ExtensionName = name ) on $left.JoinID == $right.VMId | summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize | order by tolower(OSName) asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.compute/virtualmachines' | extend JoinID = toupper(id), OSName = tostring(properties.osProfile.computerName), OSType = tostring(properties.storageProfile.osDisk.osType), VMSize = tostring(properties.hardwareProfile.vmSize) | join kind=leftouter( Resources | where type == 'microsoft.compute/virtualmachines/extensions' | extend VMId = toupper(substring(id, 0, indexof(id, '/extensions'))), ExtensionName = name ) on $left.JoinID == $right.VMId | summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize | order by tolower(OSName) asc"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.cn</a>

---

## <a name="find-storage-accounts-with-a-specific-tag-on-the-resource-group"></a><a name="join-findstoragetag"></a>Поиск учетных записей хранения с указанным тегом в группе ресурсов

В приведенном ниже запросе используется **inner** `join` для подключения учетных записей хранения к группам ресурсов с указанными именем с учетом регистра и значением тега.

```kusto
Resources
| where type =~ 'microsoft.storage/storageaccounts'
| join kind=inner (
    ResourceContainers
    | where type =~ 'microsoft.resources/subscriptions/resourcegroups'
    | where tags['Key1'] =~ 'Value1'
    | project subscriptionId, resourceGroup)
on subscriptionId, resourceGroup
| project-away subscriptionId1, resourceGroup1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | where tags['Key1'] =~ 'Value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | where tags['Key1'] =~ 'Value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.cn</a>

---

Если вам необходимо найти нечувствительное к регистру имя и значение тега, используйте `mv-expand` с параметром **bagexpansion**. Этот запрос использует больше квот, чем предыдущий, поэтому используйте `mv-expand` только при необходимости.

```kusto
Resources
| where type =~ 'microsoft.storage/storageaccounts'
| join kind=inner (
    ResourceContainers
    | where type =~ 'microsoft.resources/subscriptions/resourcegroups'
    | mv-expand bagexpansion=array tags
    | where isnotempty(tags)
    | where tags[0] =~ 'key1' and tags[1] =~ 'value1'
    | project subscriptionId, resourceGroup)
on subscriptionId, resourceGroup
| project-away subscriptionId1, resourceGroup1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | mv-expand bagexpansion=array tags | where isnotempty(tags) | where tags[0] =~ 'key1' and tags[1] =~ 'value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | mv-expand bagexpansion=array tags | where isnotempty(tags) | where tags[0] =~ 'key1' and tags[1] =~ 'value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.cn</a>

---

## <a name="combine-results-from-two-queries-into-a-single-result"></a><a name="unionresults"></a>Объединение результатов из двух запросов в один результат

Следующий запрос использует `union`, чтобы получить результаты из таблицы _ResourceContainers_ и добавить их в результаты из таблицы _Resources_.

```kusto
ResourceContainers
| where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5
| union  (Resources | project name, type | limit 5)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5 | union  (Resources | project name, type | limit 5)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5 | union  (Resources | project name, type | limit 5)"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.cn</a>

---

## <a name="summarize-virtual-machine-by-the-power-states-extended-property"></a><a name="vm-powerstate"></a>Суммирование виртуальных машин по расширенному свойству состояния питания

Этот запрос использует [расширенные свойства](../concepts/query-language.md#extended-properties) на виртуальных машинах для суммирования по состоянию питания.


```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' | summarize count() by tostring(properties.extended.instanceView.powerState.code)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.compute/virtualmachines' | summarize count() by tostring(properties.extended.instanceView.powerState.code)"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.com</a>
- Портал Azure для государственных организаций: <a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.us</a>
- Портал Azure для Китая (21Vianet): <a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.cn</a>

---

## <a name="count-of-non-compliant-guest-configuration-assignments"></a><a name="count-gcnoncompliant"></a>Число несоответствующих назначений гостевой конфигурации

Отображает число несоответствующих компьютеров для [каждой причины назначения гостевой конфигурации](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration). Ограничивает результаты первыми 100 для повышения производительности.

```kusto
GuestConfigurationResources
| extend vmid = split(properties.targetResourceId,'/')
| where properties.complianceStatus == 'NonCompliant'
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| project machine = tostring(vmid[(-1)]),
    type = tostring(vmid[(-3)]),
    name,
    status = tostring(properties.complianceStatus),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase)
| summarize count() by resource, name
| order by count_
| limit 100
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | where properties.complianceStatus == 'NonCompliant' | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase) | summarize count() by resource, name | order by count_ | limit 100"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | where properties.complianceStatus == 'NonCompliant' | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase) | summarize count() by resource, name | order by count_ | limit 100"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C%22%2F%22)%20%7C%20where%20properties.complianceStatus%20%3D%3D%20'NonCompliant'%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20project%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%2C%20type%20%3D%20tostring(vmid%5B(-3)%5D)%2C%20name%2C%20status%20%3D%20tostring(properties.complianceStatus)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20phrase%20%3D%20tostring(properties_latestAssignmentReport_resources_reasons.phrase)%20%7C%20summarize%20count()%20by%20resource%2C%20name%20%7C%20order%20by%20count_%20%7C%20limit%20100" target="_blank">portal.azure.com</a>

---

## <a name="query-details-of-guest-configuration-assignment-reports"></a><a name="query-gcreports"></a>Сведения о запросах для отчетов о назначениях гостевой конфигурации

Отображает отчет из сведений о [причине назначения гостевой конфигурации](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration). В следующем примере запрос возвращает только результаты, в которых имя гостевого назначения имеет значение `installed_application_linux`, а выходные данные содержат строку `Python`, то есть перечисляет все компьютеры Linux, на которых установлен пакет, имя которого содержит слово **Python**. Чтобы запросить сведения о соответствии всех компьютеров определенному назначению, удалите второе предложение `where`.

```kusto
GuestConfigurationResources
| extend vmid = split(properties.targetResourceId,'/')
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| where name in ('installed_application_linux')
| where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python'
| project machine = tostring(vmid[(-1)]),
    type = tostring(vmid[(-3)]),
    name,
    status = tostring(properties.complianceStatus),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | where name in ('installed_application_linux') | where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring (properties_latestAssignmentReport_resources_reasons.phrase)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | where name in ('installed_application_linux') | where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring (properties_latestAssignmentReport_resources_reasons.phrase)"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C'%2F')%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20where%20name%20in%20('installed_application_linux')%20%7C%20where%20properties_latestAssignmentReport_resources_reasons.phrase%20contains%20'Python'%20%7C%20project%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%2C%20type%20%3D%20tostring(vmid%5B(-3)%5D)%2C%20name%2C%20status%20%3D%20tostring(properties.complianceStatus)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20phrase%20%3D%20tostring%20(properties_latestAssignmentReport_resources_reasons.phrase)" target="_blank">portal.azure.com</a>

---

## <a name="find-all-reasons-a-machine-is-non-compliant-for-guest-configuration-assignments"></a><a name="query-gcmachinedetails"></a>Поиск всех причин, по которым компьютер не соответствует требованиям для назначений гостевой конфигурации

Отображает все [причины назначений гостевой конфигурации](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration) для определенного компьютера. Удалите первое предложение `where`, чтобы также включить проверки, по итогам которых компьютер соответствует требованиям.

```kusto
GuestConfigurationResources
| where properties.complianceStatus == 'NonCompliant'
| extend vmid = split(properties.targetResourceId,'/')
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| extend machine = tostring(vmid[(-1)])
| where machine == 'MACHINENAME'
| project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    name,
    machine,
    resourceGroup,
    subscriptionId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | where properties.complianceStatus == 'NonCompliant' | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | extend machine = tostring(vmid[(-1)]) | where machine == 'MACHINENAME' | project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase), resource = tostring(properties_latestAssignmentReport_resources.resourceId), name, machine, resourceGroup, subscriptionId"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | where properties.complianceStatus == 'NonCompliant' | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | extend machine = tostring(vmid[(-1)]) | where machine == 'MACHINENAME' | project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase), resource = tostring(properties_latestAssignmentReport_resources.resourceId), name, machine, resourceGroup, subscriptionId"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png"::: Попробуйте выполнить следующий запрос в обозревателе Azure Resource Graph:

- Портал Azure: <a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20where%20properties.complianceStatus%20%3D%3D%20'NonCompliant'%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C'%2F')%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20extend%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%20%7C%20where%20machine%20%3D%3D%20'MACHINENAME'%20%7C%20project%20phrase%20%3D%20tostring(properties_latestAssignmentReport_resources_reasons.phrase)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20name%2C%20machine%2C%20resourceGroup%2C%20subscriptionId" target="_blank">portal.azure.com</a>

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с примерами [начальных запросов](starter.md).
- Узнайте больше о [языке запросов](../concepts/query-language.md).
- Узнайте больше о том, как [изучать ресурсы](../concepts/explore-resources.md).