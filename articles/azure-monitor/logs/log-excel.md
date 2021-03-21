---
title: Интеграция Log Analytics и Excel
description: Получение Log Analytics запроса к Excel и обновление результатов в Excel.
ms.topic: conceptual
author: roygalMS
ms.author: roygal
ms.date: 11/03/2020
ms.openlocfilehash: f2834e9bd91ecbbf32e0321179c2359862a5b605
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102041116"
---
# <a name="integrate-log-analytics-and-excel"></a>Интеграция Log Analytics и Excel

Azure Monitor Log Analytics и Microsoft Excel можно интегрировать с помощью запросов M и API Log Analytics. Такая интеграция позволяет передавать до 500 000 записей в Excel, если общий объем результатов не превышает 61MiB.

> [!NOTE]
> Поскольку Excel является локальным клиентским приложением, ограничения на локальное оборудование и программное обеспечение влияют на производительность и возможность обработки больших наборов данных.

## <a name="create-your-m-query-in-log-analytics"></a>Создание запроса M в Log Analytics 

1. **Создайте и выполните запрос** в log Analytics, как обычно. Не беспокойтесь, если достигнуто ограничение 10 000 записей в пользовательском интерфейсе.  Рекомендуется использовать относительные даты, например функцию "назад" или средство выбора времени пользовательского интерфейса, поэтому Excel обновляет правильный набор данных.
  
2. **Экспорт запроса** . когда вы довольны запросом и его результатами, экспортируйте запрос в M с помощью log Analytics **Экспорт в Power BI (m Query)** в меню *Экспорт* :

:::image type="content" source="media/log-excel/export-query.png" alt-text="Log Analytics запрос с параметром Data and Export" border="true":::



При выборе этого параметра загружается txt-файл, содержащий код M, который можно использовать в Excel.

Показанный выше запрос экспортирует следующий код M. Ниже приведен пример кода M, экспортированного для запроса в нашем примере:

```m
/*
The exported Power Query Formula Language (M Language ) can be used with Power Query in Excel
and Power BI Desktop.
For Power BI Desktop follow the instructions below: 
1) Download Power BI Desktop from https://powerbi.microsoft.com/desktop/
2) In Power BI Desktop select: 'Get Data' -> 'Blank Query'->'Advanced Query Editor'
3) Paste the M Language script into the Advanced Query Editor and select 'Done'
*/

let AnalyticsQuery =
let Source = Json.Document(Web.Contents("https://api.loganalytics.io/v1/workspaces/ddcfc599-cae0-48ee-9026-fffffffffffff/query", 
[Query=[#"query"="

Heartbeat 
| summarize dcount(ComputerIP) by bin(TimeGenerated, 1h)    
| render timechart
",#"x-ms-app"="OmsAnalyticsPBI",#"timespan"="P1D",#"prefer"="ai.response-thinning=true"],Timeout=#duration(0,0,4,0)])),
TypeMap = #table(
{ "AnalyticsTypes", "Type" }, 
{ 
{ "string",   Text.Type },
{ "int",      Int32.Type },
{ "long",     Int64.Type },
{ "real",     Double.Type },
{ "timespan", Duration.Type },
{ "datetime", DateTimeZone.Type },
{ "bool",     Logical.Type },
{ "guid",     Text.Type },
{ "dynamic",  Text.Type }
}),
DataTable = Source[tables]{0},
Columns = Table.FromRecords(DataTable[columns]),
ColumnsWithType = Table.Join(Columns, {"type"}, TypeMap , {"AnalyticsTypes"}),
Rows = Table.FromRows(DataTable[rows], Columns[name]), 
Table = Table.TransformColumnTypes(Rows, Table.ToList(ColumnsWithType, (c) => { c{0}, c{3}}))
in
Table
in AnalyticsQuery
```

## <a name="connect-query-to-excel"></a>Подключение запроса к Excel 

Для импорта запроса. 

1. Откройте Microsoft Excel. 
1. На ленте перейдите в меню **данные** . Выберите **получить данные**. В **других источниках** выберите **пустой запрос**:
 
   :::image type="content" source="media/log-excel/excel-import-blank-query.png" alt-text="Параметр &quot;импортировать с пустым&quot; в Excel" border="true":::

1. В окне Power Query выберите **Расширенный редактор**:

   :::image type="content" source="media/log-excel/advanced-editor.png" alt-text="Расширенный редактор запросов Excel" border="true":::

 
1. Замените текст в расширенном редакторе запросом, экспортированным из Log Analytics:

   :::image type="content" source="media/log-excel/advanced-editor-2.png" alt-text="Создание пустого запроса" border="true":::
 
1. Нажмите кнопку **Готово**, а затем — **загрузить и закрыть**. Excel выполняет запрос, используя API log Analytics и результирующий набор, а затем показывает.
 

   :::image type="content" source="media/log-excel/excel-query-result.png" alt-text="Результаты запроса в Excel" border="true":::

> [!Note]
> Если число записей меньше ожидаемого, объем результатов может превысить ограничение 61MiB. Попробуйте использовать `project` или `project-away` в запросе, чтобы ограничить столбцы нужными столбцами.

##  <a name="refreshing--data"></a>Обновление данных

Вы можете обновить данные непосредственно из Excel. В группе меню **данные** на ленте Excel нажмите кнопку **Обновить** .
 
## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об интеграции Excel с внешними источниками данных см. в разделе [Импорт данных из внешних источников данных (Power Query)](https://support.office.com/article/import-data-from-external-data-sources-power-query-be4330b3-5356-486c-a168-b68e9e616f5a) .