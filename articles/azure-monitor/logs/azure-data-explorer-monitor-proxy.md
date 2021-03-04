---
title: Запрос данных в Azure Monitor с помощью обозреватель данных Azure (Предварительная версия)
description: Используйте обозреватель данных Azure для выполнения перекрестных запросов между обозреватель данных Azure, Log Analytics рабочими областями и классическими Application Insights приложениями в Azure Monitor.
author: osalzberg
ms.author: bwren
ms.reviewer: bwren
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: 65dba60a798b1157a44a7a198b8eba7de1e8fe81
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102031265"
---
# <a name="query-data-in-azure-monitor-using-azure-data-explorer-preview"></a>Запрос данных в Azure Monitor с помощью Azure Data Explorer (предварительная версия)

Обозреватель данных Azure поддерживает запросы между службами между Azure обозреватель данных, [Application Insights (AI)](../app/app-insights-overview.md)и [log Analytics (La)](./data-platform-logs.md). Затем вы можете запросить рабочую область Log Analytics/Application Insights с помощью средств обозреватель данных Azure и обратиться к ней в запросе кросс-обслуживания. В этой статье показано, как создать запрос между службами и как добавить рабочую область Log Analytics/Application Insights в веб-интерфейс Azure обозреватель данных.

Поток запросов между службами в Azure обозреватель данных: :::image type="content" source="media\azure-data-explorer-monitor-proxy\azure-data-explorer-monitor-flow.png" alt-text="поток прокси обозревателя данных Azure.":::

> [!NOTE]
> * Возможность запрашивать Azure Monitor данных из Azure обозреватель данных или напрямую из клиентских средств Azure обозреватель данных или косвенно, выполняя запрос в кластере Azure обозреватель данных, находится в режиме предварительного просмотра.
>* Обратитесь к группе по [запросам обслуживания](mailto:adxproxy@microsoft.com) с любыми вопросами.

## <a name="add-a-log-analyticsapplication-insights-workspace-to-azure-data-explorer-client-tools"></a>Добавление рабочей области Log Analytics/Application Insights в клиентские средства Azure Data Explorer

1. Прежде чем подключиться к кластеру Log Analytics или Application Insights, убедитесь, что в меню слева отображается собственный кластер Azure Data Explorer (например, кластер *help*).

:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-data-explorer-web-ui-help-cluster.png" alt-text="Azure обозреватель данных собственный кластер.":::

 В пользовательском интерфейсе Azure Data Explorer (https://dataexplorer.azure.com/clusters) ) выберите **Добавить кластер**.

2. В окне **Добавление кластера** добавьте URL-адрес кластера LA или AI.

    * Для LA: `https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>`
    * Для AI: `https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>`

    * Выберите **Добавить**.

:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-proxy-add-cluster.png" alt-text="Добавить кластер.":::
 
>[!NOTE]
>Если вы добавляете подключение к нескольким областям Log Analytics или Application Insights, дайте каждому из них другое имя. В противном случае в левой области они будут отображаться с одним именем.

 Когда подключение будет установлено, рабочая область Log Analytics или Application Insights отобразится на панели слева с вашим собственным кластером Azure Data Explorer.

:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-azure-data-explorer-clusters.png" alt-text="Log Analytics и кластеры обозреватель данных Azure.":::
 
> [!NOTE]
> Количество рабочих областей Azure Monitor, которые могут быть сопоставлены, ограничено до 100.

## <a name="create-queries-using-azure-monitor-data"></a>Создание запросов с помощью Azure Monitor данных

Запросы можно выполнять с помощью клиентских средств, поддерживающих запросы Kusto, например следующих: Kusto Explorer, веб-интерфейс Azure Data Explorer, Jupyter Kqlmagic, Flow, Power Query, PowerShell, Lens, REST API.

> [!NOTE]
> Возможность запросов между службами используется только для извлечения данных. Дополнительные сведения см. в разделе [Поддержка функций](#function-supportability).

> [!TIP]
> * Имя базы данных должно совпадать с именем ресурса, указанным в запросе Cross Service. В именах учитывается регистр.
> * При использовании межкластерных запросов убедитесь в правильности именования приложений Application Insights и рабочих областей Log Analytics.
> * Если имена содержат специальные символы, они заменяются на кодировку URL-адресов в запросе Cross Service.
> * Если имена содержат символы, которые не соответствуют [правилам для именования идентификаторов KQL](/azure/data-explorer/kusto/query/schema-entities/entity-names), они заменяются символом тире **-** .

### <a name="direct-query-on-your-log-analytics-or-application-insights-workspaces-from-azure-data-explorer-client-tools"></a>Отправка прямых запросов к рабочим областям Log Analytics или Application Insights из клиентских средств Azure Data Explorer

Выполнение запросов к рабочим областям Log Analytics или Application Insights. Убедитесь, что рабочая область выбрана на панели слева.
 
```kusto
Perf | take 10 // Demonstrate cross service query on the Log Analytics workspace
```

:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-proxy-query-la.png" alt-text="Запрос Log Analytics рабочей области.":::

### <a name="cross-query-of-your-log-analytics-or-application-insights-and-the-azure-data-explorer-native-cluster"></a>Перекрестный запрос Log Analytics или Application Insights и собственного кластера Azure обозреватель данных

При выполнении запросов между кластерами убедитесь, что на панели слева выбран собственный кластер Azure Data Explorer. В следующих примерах демонстрируется объединение таблиц кластера Azure обозреватель данных [с помощью объединения](/azure/data-explorer/kusto/query/unionoperator) с рабочей областью log Analytics.

```kusto
union StormEvents, cluster('https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>').database('<workspace-name>').Perf
| take 10
```

```kusto
let CL1 = 'https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>';
union <Azure Data Explorer table>, cluster(CL1).database(<workspace-name>).<table name>
```

:::image type="content" source="media\azure-data-explorer-monitor-proxy\azure-data-explorer-cross-query-proxy.png" alt-text="Запрос между службами из обозреватель данных Azure.":::

Использование [оператора `join`](/azure/data-explorer/kusto/query/joinoperator) вместо union может потребовать наличия [`hint`](/azure/data-explorer/kusto/query/joinoperator#join-hints) для его выполнения в собственном кластере Azure Data Explorer.

### <a name="join-data-from-an-azure-data-explorer-cluster-in-one-tenant-with-an-azure-monitor-resource-in-another"></a>Объединение данных из кластера Azure Data Explorer в одном арендаторе с ресурсом Azure Monitor в другом

Межклиентские запросы между службами не поддерживаются. Вы выполняете вход в один арендатор для выполнения запроса с двумя ресурсами.

Если ресурс Azure Data Explorer находится в арендаторе A, а рабочая область Log Analytics — в арендаторе B, используйте один из следующих двух методов:

1. Azure Data Explorer позволяет добавлять роли для субъектов в разные арендаторы. Добавьте свой идентификатор пользователя в арендаторе B в качестве полномочного пользователя в кластере Azure Data Explorer. Убедитесь, что свойство *[TrustedExternalTenant](/powershell/module/az.kusto/update-azkustocluster)* в кластере Azure Data Explorer содержит арендатор B. Выполните полный перекрестный запрос в арендаторе B.

2. Используйте [Lighthouse](../../lighthouse/index.yml) для проецирования ресурса Azure Monitor в арендатор A.
### <a name="connect-to-azure-data-explorer-clusters-from-different-tenants"></a>Подключение к кластерам Azure Data Explorer из разных арендаторов

Kusto Explorer автоматически выполняет за вас вход в арендатор, которому изначально принадлежит учетная запись пользователя. Чтобы получить доступ к ресурсам в других арендаторах с помощью одной учетной записи пользователя, для `tenantId` необходимо явно указать в строке подключения следующее: `Data Source=https://ade.applicationinsights.io/subscriptions/SubscriptionId/resourcegroups/ResourceGroupName;Initial Catalog=NetDefaultDB;AAD Federated Security=True;Authority ID=`**TenantId**

## <a name="function-supportability"></a>Поддержка функций

Запросы перекрестных служб в Azure обозреватель данных поддерживают функции как для Application Insights, так и для Log Analytics.
Эта возможность позволяет выполнять межкластерные запросы для указания прямых ссылок на табличную функцию Azure Monitor.
В запросе Cross Service поддерживаются следующие команды:

* `.show functions`
* `.show function {FunctionName}`
* `.show database {DatabaseName} schema as json`

На следующем рисунке показан пример запрашивания табличной функции из пользовательского веб-интерфейса Azure Data Explorer.
Чтобы использовать функцию, запустите ее по имени в окне "Запрос".

:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-proxy-function-query.png" alt-text="Запрос табличной функции из пользовательского веб-интерфейса Azure обозреватель данных.":::

## <a name="additional-syntax-examples"></a>Дополнительные примеры синтаксиса

При вызове кластеров Log Analytics или Application Insights доступны следующие параметры синтаксиса:

|Описание синтаксиса  |Application Insights  |Log Analytics  |
|----------------|---------|---------|
| База данных в кластере, которая содержит только определенный ресурс в этой подписке (**рекомендуется для межкластерных запросов**) |   cluster(`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>').database('<ai-app-name>`) | cluster(`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>').database('<workspace-name>`)     |
| Кластер, содержащий все приложения и рабочие области в этой подписке    |     cluster(`https://ade.applicationinsights.io/subscriptions/<subscription-id>`)    |    cluster(`https://ade.loganalytics.io/subscriptions/<subscription-id>`)     |
|Кластер, содержащий все приложения и рабочие области в подписке, которые входят в эту группу ресурсов    |   cluster(`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>`)      |    cluster(`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>`)      |
|Кластер, содержащий только определенный ресурс в этой подписке      |    cluster(`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>`)    |  cluster(`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>`)     |

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [структуре данных рабочих областей log Analytics и Application Insights](data-platform-logs.md).
- Узнайте, как [писать запросы в Azure обозреватель данных](/azure/data-explorer/write-queries).