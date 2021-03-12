---
title: Основные сведения о языке запросов
description: Описание таблиц Resource Graph и доступных типов данных, операторов и функций Kusto, которые можно использовать с Azure Resource Graph.
ms.date: 03/10/2021
ms.topic: conceptual
ms.openlocfilehash: f6cb13814fe725ff0253a0a5bf0098f0080fa407
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102633807"
---
# <a name="understanding-the-azure-resource-graph-query-language"></a>Общие сведения о языке запросов графика ресурсов Azure

Язык запросов для графика ресурсов Azure поддерживает ряд операторов и функций. Каждый из них работает так же, как элементы [языка запросов Kusto (KQL)](/azure/kusto/query/index). Чтобы ознакомиться с языком запросов, используемом в Resource Graph, начните с [учебника по KQL](/azure/kusto/query/tutorial).

В этой статье рассматриваются компоненты языка, поддерживаемые Resource Graph:

- [таблицы Resource Graph;](#resource-graph-tables)
- [Пользовательские элементы языка графа ресурсов](#resource-graph-custom-language-elements)
- [поддерживаемые элементы языка KQL;](#supported-kql-language-elements)
- [Область запроса](#query-scope)
- [escape-символы.](#escape-characters)

## <a name="resource-graph-tables"></a>Таблицы Resource Graph

Граф ресурсов предоставляет несколько таблиц для хранения данных о Azure Resource Manager типах ресурсов и их свойствах. Некоторые таблицы можно использовать с `join` `union` операторами или для получения свойств из связанных типов ресурсов. Ниже приведен список таблиц, доступных в Resource Graph.

|Таблица графиков ресурсов |Могут `join` ли другие таблицы? |Описание |
|---|---|---|
|Ресурсы |Да |Таблица по умолчанию, если таблица в запросе не определена. В ней содержится большинство типов ресурсов Resource Manager и их свойств. |
|ResourceContainers |Да |Содержит подписку (в предварительной версии — `Microsoft.Resources/subscriptions`) и типы ресурсов и данные группы ресурсов (`Microsoft.Resources/subscriptions/resourcegroups`). |
|AdvisorResources |Да (предварительная версия) |Содержит ресурсы, _связанные_ с `Microsoft.Advisor`. |
|AlertsManagementResources |Да (предварительная версия) |Содержит ресурсы, _связанные_ с `Microsoft.AlertsManagement`. |
|екстендедлокатионресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.ExtendedLocation`. |
|гуестконфигуратионресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.GuestConfiguration`. |
|кубернетесконфигуратионресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.KubernetesConfiguration`. |
|MaintenanceResources |Частично, только _для_ объединения. (предварительная версия) |Содержит ресурсы, _связанные_ с `Microsoft.Maintenance`. |
|патчассессментресаурцес|Нет |Включает ресурсы, _относящиеся_ к оценке исправлений на виртуальных машинах Azure. |
|патчинсталлатионресаурцес|Нет |Включает ресурсы, _связанные_ с установкой исправлений для виртуальных машин Azure. |
|полициресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.PolicyInsights`. (**Предварительная версия**)|
|рековерисервицесресаурцес |Частично, только _для_ объединения. (предварительная версия) |Включает ресурсы, _связанные_ с `Microsoft.DataProtection` и `Microsoft.RecoveryServices` . |
|SecurityResources |Частично, только _для_ объединения. (предварительная версия) |Содержит ресурсы, _связанные_ с `Microsoft.Security`. |
|сервицехеалсресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.ResourceHealth`. |
|ворклоадмониторресаурцес |Нет |Содержит ресурсы, _связанные_ с `Microsoft.WorkloadMonitor`. |

Полный список, включая типы ресурсов, см. в разделе [reference: Supported Tables and Resource Types](../reference/supported-tables-resources.md).

> [!NOTE]
> _Resources_ — это таблица по умолчанию. При запросе таблицы _Resources_ не требуется указывать имя таблицы, если не используется `join` или `union`. Однако рекомендуется всегда включать в запрос начальную таблицу.

Используйте обозреватель Resource Graph на портале, чтобы узнать, какие типы ресурсов доступны в каждой таблице. В качестве альтернативы можно использовать такой запрос, как `<tableName> | distinct type`, чтобы получить список имеющихся в среде типов ресурсов, которые поддерживаются данной таблицей Resource Graph.

В приведенном ниже запросе показана простая операция `join`. В результате запроса столбцы объединяются, а ко всем дублирующимся именам столбцов из соединяемой таблицы (в этом примере _ResourceContainers_) добавляется **1**. Так как таблица _ресаурцеконтаинерс_ имеет типы для подписок и групп ресурсов, для подключения к ресурсу из таблицы _Resources_ может использоваться любой тип.

```kusto
Resources
| join ResourceContainers on subscriptionId
| limit 1
```

В приведенном ниже запросе показано более сложное использование `join`. Сначала запрос использует `project` для получения полей из _ресурсов_ для типа ресурса Azure Key Vault хранилищ. Следующий шаг используется `join` для слияния результатов с _ресаурцеконтаинерс_ , где тип — это подписка _на_ свойство, которое является и в первой таблице, `project` и в соединенной таблице `project` . Переименование поля позволяет избежать `join` добавления его в качестве _name1_ , так как свойство уже проецируется из _ресурсов_. Результатом запроса является одно хранилище ключей, отображающее тип, имя, расположение и группу ресурсов хранилища ключей, а также имя подписки, в которой он находится.

```kusto
Resources
| where type == 'microsoft.keyvault/vaults'
| project name, type, location, subscriptionId, resourceGroup
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project type, name, location, resourceGroup, SubName
| limit 1
```

> [!NOTE]
> При ограничении результатов `join` с помощью `project` свойство, используемое `join` для связывания двух таблиц (_subscriptionId_ в приведенном выше примере), должно включаться в `project`.

## <a name="extended-properties-preview"></a><a name="extended-properties"></a>Расширенные свойства (Предварительная версия)

В качестве функции _предварительной версии_ некоторые типы ресурсов в графе ресурсов имеют дополнительные свойства, связанные с типом, которые могут запрашиваться за пределами свойств, предоставляемых Azure Resource Manager. Этот набор значений, известный как _Расширенные свойства_, существует в поддерживаемом типе ресурсов в `properties.extended` . Чтобы узнать, какие типы ресурсов имеют _Расширенные свойства_, используйте следующий запрос:

```kusto
Resources
| where isnotnull(properties.extended)
| distinct type
| order by type asc
```

Пример. Получение числа виртуальных машин по следующим параметрам `instanceView.powerState.code` :

```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

## <a name="resource-graph-custom-language-elements"></a>Пользовательские элементы языка графа ресурсов

### <a name="shared-query-syntax-preview"></a><a name="shared-query-syntax"></a>Синтаксис общего запроса (Предварительная версия)

В качестве функции предварительной версии [общий доступ к общему запросу](../tutorials/create-share-query.md) можно получить непосредственно в запросе графа ресурсов. Этот сценарий позволяет создавать стандартные запросы как общие запросы и использовать их повторно. Чтобы вызвать общий запрос в запросе графа ресурсов, используйте `{{shared-query-uri}}` синтаксис. URI общего запроса — это _идентификатор ресурса_ общего запроса на странице **параметров** этого запроса. В этом примере наш URI общего запроса — `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS` .
Этот URI указывает на подписку, группу ресурсов и полное имя общего запроса, на который мы хотим ссылаться в другом запросе. Этот запрос аналогичен тому, который был создан в [руководстве: создание и совместное использование запроса](../tutorials/create-share-query.md).

> [!NOTE]
> Нельзя сохранить запрос, который ссылается на общий запрос, в качестве общего запроса.

Пример 1. использование только общего запроса

Результаты этого запроса графа ресурсов совпадают с результатами запроса, хранящегося в общем запросе.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
```

Пример 2. Включение общего запроса в состав более крупного запроса

Сначала этот запрос использует общий запрос, а затем использует `limit` для ограничения результатов.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
| where properties_storageProfile_osDisk_osType =~ 'Windows'
```

## <a name="supported-kql-language-elements"></a>Поддерживаемые элементы языка KQL

Граф ресурсов поддерживает подмножество [типов данных](/azure/kusto/query/scalar-data-types/)ККЛ, [скалярных функций](/azure/kusto/query/scalarfunctions), [скалярных операторов](/azure/kusto/query/binoperators)и [агрегатных функций](/azure/kusto/query/any-aggfunction). Resource Graph поддерживает определенные [табличные операторы](/azure/kusto/query/queries), некоторые из которых имеют разное поведение.

### <a name="supported-tabulartop-level-operators"></a>Поддерживаемые табличные операторы и операторы верхнего уровня

Ниже приведен список табличных операторов KQL, поддерживаемых Resource Graph, с примерами.

|KQL |Пример запроса Resource Graph |Примечания |
|---|---|---|
|[count](/azure/kusto/query/countoperator) |[Число хранилищ ключей](../samples/starter.md#count-keyvaults) | |
|[distinct](/azure/kusto/query/distinctoperator) |[Отображение ресурсов, которые содержат хранилище](../samples/starter.md#show-storage) | |
|[extend](/azure/kusto/query/extendoperator) |[Подсчет виртуальных машин по типу ОС](../samples/starter.md#count-os) | |
|[join](/azure/kusto/query/joinoperator) |[Хранилище ключей с именем подписки](../samples/advanced.md#join) |Поддерживаемые разновидности оператора соединения: [innerunique](/azure/kusto/query/joinoperator#default-join-flavor), [inner](/azure/kusto/query/joinoperator#inner-join), [leftouter](/azure/kusto/query/joinoperator#left-outer-join). Ограничение в 3 `join` в одном запросе, 1 из которых может быть перекрестной таблицей `join` . Если между ресурсами и Ресаурцеконтаинерс используется все перекрестные таблицы `join` , то 3 перекрестные таблицы   `join` разрешены. Пользовательские стратегии соединения, такие как широковещательное соединение, не допускаются. Сведения о том, какие таблицы могут использоваться `join` , см. в разделе [таблицы Graph ресурсов](#resource-graph-tables). |
|[limit](/azure/kusto/query/limitoperator) |[Вывод списка общедоступных IP-адресов](../samples/starter.md#list-publicip) |Синоним `take` . Не работает с [Skip](./work-with-data.md#skipping-records). |
|[mvexpand](/azure/kusto/query/mvexpandoperator) | | Устаревший оператор, используйте вместо него `mv-expand`. Максимальное значение _RowLimit_ равно 400. Значение по умолчанию — 128. |
|[mv-expand](/azure/kusto/query/mvexpandoperator) |[Список Cosmos DB с конкретным указанием расположений записи](../samples/advanced.md#mvexpand-cosmosdb) |Максимальное значение _RowLimit_ равно 400. Значение по умолчанию — 128. Не более 3 `mv-expand` в одном запросе.|
|[order](/azure/kusto/query/orderoperator) |[Вывод списка ресурсов, отсортированных по имени](../samples/starter.md#list-resources) |Синоним `sort` |
|[project](/azure/kusto/query/projectoperator) |[Вывод списка ресурсов, отсортированных по имени](../samples/starter.md#list-resources) | |
|[project-away](/azure/kusto/query/projectawayoperator) |[Удаление столбцов из результатов](../samples/advanced.md#remove-column) | |
|[sort](/azure/kusto/query/sortoperator) |[Вывод списка ресурсов, отсортированных по имени](../samples/starter.md#list-resources) |Синоним `order` |
|[summarize](/azure/kusto/query/summarizeoperator) |[Подсчет ресурсов Azure](../samples/starter.md#count-resources) |Только первая упрощенная страница |
|[take](/azure/kusto/query/takeoperator) |[Вывод списка общедоступных IP-адресов](../samples/starter.md#list-publicip) |Синоним `limit` . Не работает с [Skip](./work-with-data.md#skipping-records). |
|[В начало](/azure/kusto/query/topoperator) |[Отображение первых пяти виртуальных машин по имени и типу ОС](../samples/starter.md#show-sorted) | |
|[union](/azure/kusto/query/unionoperator) |[Объединение результатов из двух запросов в один результат](../samples/advanced.md#unionresults) |Допускается одна таблица: _T_ `| union` \[`kind=` `inner`\|`outer`\] \[`withsource=`_ColumnName_\] _Table_. Не более 3 разветвлений `union` в одном запросе. Нечеткое разрешение таблиц разветвлений `union` не допускается. Может использоваться в одной таблице или между таблицами _Resources_ и _ResourceContainers_. |
|[where](/azure/kusto/query/whereoperator) |[Отображение ресурсов, которые содержат хранилище](../samples/starter.md#show-storage) | |

`join`В одном запросе пакета SDK для Graph используется по умолчанию ограничение в 3 и 3 `mv-expand` оператора. Вы можете запросить увеличение этих ограничений для вашего клиента с помощью **справки и поддержки**.

Для поддержки возможностей портала "открыть запрос" Обозреватель графа ресурсов Azure имеет более высокий уровень глобальных ограничений, чем пакет SDK для Graph.

## <a name="query-scope"></a>Область запроса

Область подписок, из которой ресурсы возвращаются запросом, зависит от метода доступа к графу ресурсов. Azure CLI и Azure PowerShell заполнить список подписок, включаемых в запрос, в зависимости от контекста полномочного пользователя. Список подписок можно задать вручную для каждой **подписки** и параметров **подписки** соответственно.
В REST API и всех других пакетах SDK список подписок, которые должны включать ресурсы из, должен быть явно определен как часть запроса.

В качестве **предварительной** версии REST API версия `2020-04-01-preview` добавляет свойство в область запроса в [группу управления](../../management-groups/overview.md). Этот API предварительной версии также делает свойство подписки необязательным. Если группа управления или список подписок не определены, областью запроса являются все ресурсы, включая делегированные ресурсы [Azure лигхсаусе](../../../lighthouse/concepts/azure-delegated-resource-management.md) , доступ к которым может получить пользователь, прошедший проверку подлинности. Новое `managementGroupId` свойство принимает идентификатор группы управления, который отличается от имени группы управления. Если `managementGroupId` указан параметр, включаются ресурсы из первой 5000 подписки в или в указанной иерархии группы управления. `managementGroupId` нельзя использовать в то же время, что и `subscriptions` .

Пример. запрос всех ресурсов в иерархии группы управления с именем "Моя группа управления" с ИДЕНТИФИКАТОРом "Мимг".

- Универсальный код ресурса (URI) REST API

  ```http
  POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2020-04-01-preview
  ```

- Текст запроса

  ```json
  {
      "query": "Resources | summarize count()",
      "managementGroupId": "myMG"
  }
  ```

## <a name="escape-characters"></a>Escape-символы

Некоторые имена свойств, например включающие в себя `.` или `$`, должны заключаться в оболочку или экранироваться в запросе, иначе они будут интерпретироваться неправильно и давать непредвиденные результаты.

- `.` — заключите имя свойства следующим образом: `['propertyname.withaperiod']`
  
  Пример запроса, в котором заключается свойство _odata.type_:

  ```kusto
  where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.['odata.type']
  ```

- `$` — экранируйте символ в имени свойства. Используемый escape-символ зависит от оболочки, из которой выполняется Resource Graph.

  - **bash** - `\`

    Пример запроса, в котором экранируется свойство _\$type_ в bash:

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.\$type
    ```

  - **cmd** — не экранируйте символ `$`.

  - **PowerShell** - ``` ` ```

    Пример запроса, в котором экранируется свойство _\$type_ в PowerShell:

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.`$type
    ```

## <a name="next-steps"></a>Дальнейшие действия

- См. описание используемого языка в разделе [Запросы для начинающих](../samples/starter.md).
- См. описание расширенных вариантов использования в разделе [Расширенные запросы](../samples/advanced.md).
- Узнайте больше о том, как [изучать ресурсы](explore-resources.md).
