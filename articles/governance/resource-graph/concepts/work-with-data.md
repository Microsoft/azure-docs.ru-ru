---
title: Работа с большими наборами данных
description: Узнайте, как получать, форматировать, разбивать на страницы и пропускать записи в больших наборах данных в Azure Resource Graph.
ms.date: 09/30/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 59f69738bf9fe25cb739539b7a1f93e4499d781a
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826022"
---
# <a name="working-with-large-azure-resource-data-sets"></a>Работа с большими наборами данных ресурса Azure

Azure Resource Graph предназначен для работы и получения сведений о ресурсах в вашей среде Azure. Resource Graph дает возможность получать эти данные быстро, даже при запросе тысяч записей. Resource Graph имеет несколько параметров для работы с этими большими наборами данных.

Рекомендации по работе с запросами с высокой частотой см. в статье [Руководство по регулируемым запросам](./guidance-for-throttled-requests.md).

## <a name="data-set-result-size"></a>Размер результата набора данных

По умолчанию Resource Graph ограничивает любой запрос возвращением до **100** записей. Этот элемент управления защищает как пользователя, так и службу от непреднамеренных запросов, которые могут привести к большим наборам данных. Это событие чаще всего происходит, когда клиент экспериментирует с запросами, чтобы найти и отфильтровать ресурсы в соответствии с их конкретными потребностями. Этот элемент управления отличается от использования операторов языка Azure Data Explorer [top](/azure/kusto/query/topoperator) или [limit](/azure/kusto/query/limitoperator) для ограничения результатов.

> [!NOTE]
> При использовании **First** рекомендуется упорядочивать результаты как `asc` или `desc` по крайней мере по одному столбцу. Без сортировки возвращаемые результаты будут случайными и не повторяемыми.

Ограничение по умолчанию может быть отменено всеми методами взаимодействия с Resource Graph. Следующие примеры демонстрируют изменение ограничения размера набора данных на _200_.

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --first 200 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -First 200
```

В [REST API](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources) элемент управления является **$top** и частью **QueryRequestOptions**.

Элемент управления, который является _наиболее строгим_, победит. Например, если ваш запрос использует операторы **top** или **limit** и приводит к большему количеству записей, чем **First**, максимальное количество возвращаемых записей будет равно **First**. Аналогичным образом, если **top** или **limit** меньше, чем **First**, возвращенный набор записей будет меньшим значением, настроенным с помощью **top** или **limit**.

**First** в настоящее время имеет максимальное допустимое значение _5000_, которое достигается при одновременном [разбиении по страницам](#paging-results) _1000_ результирующих записей.

> [!IMPORTANT]
> Если для **First** настроено значение больше _1000_ записей, запрос должен **проецировать** поле **id**, чтобы сработало разбиение на страницы. Если оно отсутствует в запросе, ответ не будет [разбит на страницы](#paging-results) и результаты будут ограничены _1000_ записей.

## <a name="skipping-records"></a>Пропуск записей

Следующий параметр для работы с большими наборами данных — элемент управления **Skip**. Этот элемент управления позволяет пропустить или перейти определенное число записей перед возвратом результатов. Параметр **Skip** полезен для запросов, которые сортируют результаты удобным образом, где намерением является получение записей в середине результирующего набора. Если необходимые результаты находятся в конце возвращенного набора данных, более эффективно использовать другую конфигурацию сортировки и извлекать результаты из верхней части набора данных.

> [!NOTE]
> При использовании **Skip** рекомендуется упорядочивать результаты как `asc` или `desc` по крайней мере по одному столбцу. Без сортировки возвращаемые результаты будут случайными и не повторяемыми. Если `limit` `take` в запросе используются или, **пропуск** игнорируется.

В следующих примерах показано, как пропустить первые _10_ записей, в результате которых будет получен запрос, вместо этого начиная возвращаемый набор результатов с 11-й записи.

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --skip 10 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -Skip 10
```

В [REST API](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources) элемент управления является **$skip** и частью **QueryRequestOptions**.

## <a name="paging-results"></a>Разбиение результатов по страницам

Если необходимо разбить результирующий набор на меньшие наборы записей для обработки или если он превышает максимально допустимое значение возвращаемых записей (_1000_), используйте разбиение на страницы. [REST API](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources) 
 **куериреспонсе** предоставляет значения, указывающие на то, что набор результатов был разбит: **ресулттрункатед** и **$skipToken**. **resultTruncated** является логическим значением, которое оповещает потребителя, если существуют дополнительные записи, которые не возвращаются в ответе. Это условие также можно определить, когда свойство **count** меньше, чем свойство **totalRecords**. **totalRecords** определяет, сколько записей соответствует запросу.

 **ресулттрункатед** имеет **значение true** , если разбиение по страницам отключено или невозможно, так как ни один `id` столбец не существует, или если доступно меньше ресурсов, чем запрашивает запрос. Если **ресулттрункатед** имеет **значение true**, свойство **$skipToken** не задано.

В следующих примерах показано, как **пропустить** первые 3000 записей и вернуть **первую** 1000 записей после пропущенных с помощью Azure CLI и Azure PowerShell:

```azurecli-interactive
az graph query -q "Resources | project id, name | order by id asc" --first 1000 --skip 3000
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project id, name | order by id asc" -First 1000 -Skip 3000
```

> [!IMPORTANT]
> Запрос должен **проецировать** поле **id**, чтобы сработала нумерация страниц. Если оно отсутствует в запросе, ответ не будет содержать **$skipToken**.

Пример см. в разделе [Запрос следующей страницы](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#next-page-query) в документации REST API.

## <a name="formatting-results"></a>Результаты форматирования

Результаты запроса к Resource Graph предоставляются в двух форматах: _Table_ и _ObjectArray_. Этот формат настраивается с помощью параметра **resultFormat** запроса. Формат _Table_ является значением по умолчанию для **resultFormat**.

Результаты из Azure CLI предоставляются по умолчанию в формате JSON. Результаты в Azure PowerShell по умолчанию представляют собой объект **PSCustomObject**, но их можно быстро преобразовать в формат JSON с помощью командлета `ConvertTo-Json`. Для других пакетов SDK можно настроить вывод результатов запроса в формате _ObjectArray_.

### <a name="format---table"></a>Формат Table

Формат по умолчанию _Table_ возвращает результаты в формате JSON, предназначенном для выделения структуры столбцов и значений строк свойств, возвращаемых запросом. Этот формат очень похож на то, как данные определяются в структурированной или электронной таблице: сначала определяются столбцы, а затем каждая строка с данными для этих столбцов.

Вот пример результата запроса в формате _Table_:

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": {
        "columns": [{
                "name": "name",
                "type": "string"
            },
            {
                "name": "type",
                "type": "string"
            },
            {
                "name": "location",
                "type": "string"
            },
            {
                "name": "subscriptionId",
                "type": "string"
            }
        ],
        "rows": [
            [
                "veryscaryvm2-nsg",
                "microsoft.network/networksecuritygroups",
                "eastus",
                "11111111-1111-1111-1111-111111111111"
            ]
        ]
    },
    "facets": [],
    "resultTruncated": "true"
}
```

### <a name="format---objectarray"></a>Формат ObjectArray

Формат _ObjectArray_ также возвращает результаты в формате JSON. Однако он соответствует отношению "ключ-значение", часто используемому в JSON и предполагающему сопоставление данных столбцов и строк в группах массива.

Вот пример результата запроса в формате _ObjectArray_:

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": [{
        "name": "veryscaryvm2-nsg",
        "type": "microsoft.network/networksecuritygroups",
        "location": "eastus",
        "subscriptionId": "11111111-1111-1111-1111-111111111111"
    }],
    "facets": [],
    "resultTruncated": "true"
}
```

Вот несколько примеров настройки **resultFormat** для использования формата _ObjectArray_:

```csharp
var requestOptions = new QueryRequestOptions( resultFormat: ResultFormat.ObjectArray);
var request = new QueryRequest(subscriptions, "Resources | limit 1", options: requestOptions);
```

```python
request_options = QueryRequestOptions(
    result_format=ResultFormat.object_array
)
request = QueryRequest(query="Resources | limit 1", subscriptions=subs_list, options=request_options)
response = client.resources(request)
```

## <a name="next-steps"></a>Дальнейшие действия

- См. описание используемого языка в разделе [Запросы для начинающих](../samples/starter.md).
- См. описание расширенных вариантов использования в разделе [Расширенные запросы](../samples/advanced.md).
- Узнайте больше о том, как [изучать ресурсы](explore-resources.md).
