---
title: Включение и управление метриками Аналитика Службы хранилища Azure (классическая модель) | Документация Майкрософт
description: Узнайте, как включать, изменять и просматривать метрики Аналитика Службы хранилища Azure.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/29/2021
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: 784929e50d25a07ae92cf388be5ac14f6fa820a6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99221645"
---
# <a name="enable-and-manage-azure-storage-analytics-metrics-classic"></a>Включение и управление метриками Аналитика Службы хранилища Azure (классическая модель)

[Аналитика службы хранилища Azure](storage-analytics.md) предоставляет метрики для всех служб хранилища для больших двоичных объектов, очередей и таблиц. Вы можете использовать [портал Azure](https://portal.azure.com) для настройки метрик, записываемых в учетную запись, и настройки диаграмм, предоставляющих визуальные представления данных метрик. В этой статье показано, как включить метрики и управлять ими. Сведения о том, как включить журналы, см. в статье [Включение и управление журналами аналитика службы хранилища Azure (классическая модель)](manage-storage-analytics-logs.md).

Рекомендуем ознакомиться со статьей [Azure Monitor для хранилищ](../../azure-monitor/insights/storage-insights-overview.md) (предварительная версия). Это функция Azure Monitor, которая обеспечивает комплексный мониторинг ваших учетных записей службы хранилища Azure, создавая единое представление данных о производительности, ресурсах и доступности ваших служб хранилища Azure. Для этого вам не требуется ничего включать или настраивать. Кроме того, вы можете мгновенно просматривать эти метрики в предварительно определенных интерактивных диаграммах и прочих доступных визуализациях.

> [!NOTE]
> За изучение данных мониторинга на портале Azure взимается плата. Дополнительные сведения см. в [этой статье](storage-analytics.md).
>
> Учетные записи хранения для блоков производительности Premium не поддерживают метрики Аналитика Службы хранилища. Если вы хотите просмотреть метрики с учетными записями хранилища BLOB-объектов высокой производительности уровня "Премиум", рассмотрите возможность использования [метрик хранилища Azure в Azure Monitor](../blobs/monitor-blob-storage.md).
>
> Дополнительные указания о функциях аналитики хранилища и других инструментах идентификации, диагностики и устранения неполадок, связанных со службой хранилища Azure, см. в статье [Мониторинг, диагностика и устранение неполадок службы хранилища Microsoft Azure](storage-monitoring-diagnosing-troubleshooting.md).
>

<a id="Enable-metrics"></a>

## <a name="enable-metrics"></a>Включить метрики

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://portal.azure.com) выберите **Учетные записи хранения**, а затем щелкните имя учетной записи хранения, чтобы открыть ее панель мониторинга.

2. Выберите **параметры диагностики (классическая модель)** в разделе **мониторинг (классический)** в колонке меню.
   
   ![Снимок экрана, посвященный параметру "параметры диагностики" (классический) в разделе "мониторинг (классический)".](./media/manage-storage-analytics-metrics/storage-enable-metrics-00.png)

3. Выберите **тип** данных метрик для каждой **службы**, которую нужно отслеживать, и **политику хранения** для данных. Можно также отключить мониторинг, задав для параметра **Состояние** значение **Откл**.

   > [!div class="mx-imgBorder"]
   > ![Настройка ведения журнала на портале Azure](./media/manage-storage-analytics-logs/enable-diagnostics.png) 

   Чтобы задать политику хранения данных, переместите ползунок **Хранение (дни)** или введите число дней хранения данных в диапазоне от 1 до 365. Значение по умолчанию для новых учетных записей хранения составляет семь дней. Если политику хранения задавать не нужно, введите ноль. Если политика хранения отсутствует, удаление данных мониторинга выполняется вручную.

   > [!WARNING]
   > Метрики хранятся в виде данных в учетной записи. Данные метрик могут накапливаться в вашей учетной записи с течением времени, что может увеличить стоимость хранилища. Если данные метрик требуются в течение небольшого периода времени, можно уменьшить затраты, изменив политику хранения данных. Устаревшие данные метрик (данные старше политики хранения) удаляются системой. Рекомендуется настроить политику хранения в зависимости от того, как долго требуется хранить данные метрик для вашей учетной записи. Дополнительные сведения см. в разделе [Выставление счетов за метрики хранилища](storage-analytics-metrics.md#billing-on-storage-metrics).
   >

4. Закончив настройку мониторинга, нажмите кнопку **Сохранить**.

Набор метрик по умолчанию отображается в диаграммах в колонке **Обзор** , а также в колонке **метрики (классическая модель)** . После включения метрик для службы может пройти до часа, прежде чем их данные отобразятся на диаграммах. Вы можете выбрать **Изменить** на любой диаграмме метрики, чтобы настроить метрики, отображаемые на диаграмме.

Можно отключить сбор метрик и ведение журнала, задав для параметра **Состояние** значение **Откл**.

> [!NOTE]
> Служба хранилища Azure использует [хранилище таблиц](storage-introduction.md#table-storage) для хранения метрик вашей учетной записи хранения и хранит метрики в таблицах, расположенных в вашей учетной записи. Дополнительные сведения см. в разделе [Как хранятся метрики](storage-analytics-metrics.md#how-metrics-are-stored).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Откройте командное окно Windows PowerShell.

2. Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

3. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

5. Получите контекст учетной записи хранения, определяющий необходимую учетную запись хранения.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Замените значение заполнителя `<resource-group-name>` именем своей группы ресурсов.

   * Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения. 

6. Вы можете использовать PowerShell на локальном компьютере, чтобы настроить метрики хранилища в учетной записи хранения. Чтобы изменить текущие параметры, используйте командлет Azure PowerShell **Set-азсторажесервицеметрикспроперти** . 

   Приведенная ниже команда переключается по минутным метрикам для службы BLOB-объектов в учетной записи хранения с периодом хранения, равным пяти дням.

   ```powershell
   Set-AzStorageServiceMetricsProperty -MetricsType Minute -ServiceType Blob -MetricsLevel ServiceAndApi  -RetentionDays 5 -Context $ctx
   ```   

   Этот командлет использует следующие параметры:  

   - **ServiceType**. Возможные значения **Blob**, **Queue**, **Table** и **File**.
   - **MetricsType**. Возможные значения: **Hour** и **Minute**.  
   - **MetricsLevel**. Возможные значения:
      - **None**. Отключает мониторинг.
      - **Service**. Сбор метрик, таких как исходящие и входящие данные, доступность, задержка и процент успешных операций, для служб больших двоичных объектов, очередей, таблиц и файлов.
      - **ServiceAndApi**. Дополнительно к метрикам служб обеспечивается сбор одинаковых метрик для каждой операции с хранилищем в API службы хранилища Azure.

   Следующая команда получает текущий уровень часовых метрик и длительность периода хранения в днях для службы BLOB-объектов в учетной записи хранения по умолчанию:  

   ```powershell
   Get-AzStorageServiceMetricsProperty -MetricsType Hour -ServiceType Blob -Context $storagecontext.Context
   ```  

   Дополнительные сведения о настройке командлетов Azure PowerShell для работы с подпиской Azure и о выборе учетной записи хранения по умолчанию см. в разделе [Установка и настройка Azure PowerShell](/powershell/azure/).  

### <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_EnableDiagnosticLogs":::

Дополнительные сведения о настройке метрик хранилища с помощью языка .NET см. в статье [API-интерфейсы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).  

Общие сведения о настройке метрик хранилища с использованием API REST см. в статье [Включение и настройка аналитики хранилища](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics). 

### <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)  

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.HourMetrics.MetricsLevel = MetricsLevel.Service;  
serviceProperties.HourMetrics.RetentionDays = 10;  

queueClient.SetServiceProperties(serviceProperties);  
```  

Дополнительные сведения о настройке метрик хранилища с помощью языка .NET см. в статье [API-интерфейсы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).  

Общие сведения о настройке метрик хранилища с использованием API REST см. в статье [Включение и настройка аналитики хранилища](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics). 

---

<a id="view-metrics"></a>

## <a name="view-metrics-in-a-chart"></a>Просмотр метрик в диаграмме

После настройки метрик аналитики хранилища для мониторинга учетной записи хранения функция аналитики хранилища записывает метрики в набор известных таблиц в учетной записи хранения. Можно настроить диаграммы для просмотра почасовых метрик в [портал Azure](https://portal.azure.com).

Чтобы выбрать метрики хранилища для отображения на диаграмме метрик, выполните следующие действия.

1. Сначала отобразите диаграмму метрики службы хранилища на портале Azure. Диаграммы можно найти в **колонке учетной записи хранения** и в колонке **метрики (классическая модель)** .

   В этом примере используется приведенная ниже диаграмма, которая отображается в **колонке учетной записи хранения**.

   ![Выбор диаграммы на портале Azure](./media/manage-storage-analytics-metrics/stg-customize-chart-00.png)

2. Щелкните любое место диаграммы, чтобы изменить ее.

3. Выберите **диапазон времени** метрик для отображения на диаграмме и **службу** (больших двоичных объектов, очередей, таблиц, файлов), метрики которой нужно отобразить. Выбраны отображение метрики службы BLOB-объектов за прошлую неделю.

   ![Выбор диапазона времени и службы в колонке "Правка диаграммы"](./media/manage-storage-analytics-metrics/storage-edit-metric-time-range.png)

4. Выберите отдельные **метрики**, которые нужно отобразить на диаграмме, затем нажмите кнопку **ОК**.

   ![Выбор отдельных метрик в колонке "Правка диаграммы"](./media/manage-storage-analytics-metrics/storage-edit-metric-selections.png)

Настройки диаграммы не влияют на сбор, статистическую обработку и хранение данных мониторинга в учетной записи хранения.

#### <a name="metrics-availability-in-charts"></a>Доступность метрики на диаграммах

Список доступных метрик меняется в зависимости от того, какая служба выбрана из раскрывающегося списка и какой тип единиц использует изменяемая диаграмма. Например, процентные метрики, такие как *PercentNetworkError* и *PercentThrottlingError*, можно выбрать только в том случае, если требуется изменить диаграмму, показывающую единицы в процентах.

![Ошибка запроса к процентной диаграмме на портале Azure](./media/manage-storage-analytics-metrics/stg-customize-chart-04.png)

#### <a name="metrics-resolution"></a>Разрешение метрик

Метрики, выбранные в колонке **Диагностика**, определяют разрешение метрик, доступных для вашей учетной записи.

* **Сводный** мониторинг позволяет собирать такие метрики, как метрики исходящего и входящего трафика, доступности, задержки и процента успешных операций. Эти метрики вычисляются для служб BLOB-объектов, очередей, таблиц и файлов.
* Мониторинг **каждого API** обеспечивает более точное разрешение, помимо статистических метрик уровня службы предоставляя метрики отдельных операций хранилища.

## <a name="download-metrics-to-archive-or-analyze-locally"></a>Скачать метрики для локального архива или анализа

Чтобы скачивать метрики для длительного хранения или анализа в локальной среде, необходимо использовать соответствующий инструмент или написать код для чтения таблиц. Эти таблицы не отображаются при выводе списка таблиц в учетной записи хранения. Однако к ним можно обращаться напрямую по имени. Многие инструменты обзора хранилищ поддерживают эти таблицы и позволяют просматривать их напрямую. Список доступных инструментов указан в разделе [Клиентские инструменты службы хранилища Azure](./storage-explorers.md).

|Метрики|Имена таблиц|Примечания| 
|-|-|-|  
|Часовые метрики|$MetricsHourPrimaryTransactionsBlob<br /><br /> $MetricsHourPrimaryTransactionsTable<br /><br /> $MetricsHourPrimaryTransactionsQueue<br /><br /> $MetricsHourPrimaryTransactionsFile|В версиях, выпущенных до 15 августа 2013 года, эти таблицы назывались так:<br /><br /> $MetricsTransactionsBlob<br /><br /> $MetricsTransactionsTable<br /><br /> $MetricsTransactionsQueue<br /><br /> Метрики для службы файлов доступны начиная с версии от 5 апреля 2015 года.|  
|Минутные метрики|$MetricsMinutePrimaryTransactionsBlob<br /><br /> $MetricsMinutePrimaryTransactionsTable<br /><br /> $MetricsMinutePrimaryTransactionsQueue<br /><br /> $MetricsMinutePrimaryTransactionsFile|Могут быть включены только с помощью PowerShell или программно.<br /><br /> Метрики для службы файлов доступны начиная с версии от 5 апреля 2015 года.|  
|Capacity|$MetricsCapacityBlob|Только для службы BLOB-объектов.|  

Подробные сведения о схемах для этих таблиц см. в разделе [Схема таблицы метрик аналитики хранилища](/rest/api/storageservices/storage-analytics-metrics-table-schema). В примере строк ниже показана только часть доступных столбцов, однако они иллюстрируют некоторые важные возможности сохранения метрик с помощью метрик хранилища.  

|PartitionKey|RowKey|Отметка времени|TotalRequests|TotalBillableRequests|TotalIngress|TotalEgress|Доступность|AverageE2ELatency|AverageServerLatency|PercentSuccess| 
|-|-|-|-|-|-|-|-|-|-|-|  
|20140522T1100|user;All|2014-05-22T11:01:16.7650250Z|7|7|4003|46801|100|104.4286|6.857143|100|  
|20140522T1100|user;QueryEntities|2014-05-22T11:01:16.7640250Z|5|5|2694|45951|100|143.8|7.8|100|  
|20140522T1100|user;QueryEntity|2014-05-22T11:01:16.7650250Z|1|1|538|633|100|3|3|100|  
|20140522T1100|user;UpdateEntity|2014-05-22T11:01:16.7650250Z|1|1|771|217|100|9|6|100|  

В этом примере данных минутной метрики для ключа раздела используется время с минутным разрешением. Ключ строки определяет тип информации, хранящейся в строке. Эта информация включает в себя тип доступа и тип запроса.  

-   Типом доступа является либо **user**, либо **system**, где **user** означает все запросы пользователей к службе хранилища, а **system** — запросы, выполненные с помощью Аналитики Службы хранилища.  
-   Типом запроса является либо **all**, в случае чего это строка сводки, либо он определяет конкретный API, например **QueryEntity** или **UpdateEntity**.  

В примере данных выше показаны все записи за одну минуту (начиная с 11:00). Таким образом, если сложить количество запросов **QueryEntities**, количество запросов **QueryEntity** и количество запросов **UpdateEntity**, в сумме получится семь. Эта сумма отображается в строке **user:All**. Аналогичным образом можно определить среднюю сквозную задержку 104,4286 в строке **user:All**, вычислив выражение ((143,8 * 5) + 3 + 9)/7.  

## <a name="view-metrics-data-programmatically"></a>Программное Просмотр данных метрик

В следующем списке показан пример кода C#, в котором реализован доступ к минутным метрикам для диапазона минут с отображением результатов в окне консоли. В этом примере кода используется клиентская библиотека службы хранилища Azure версии 4, включающая в себя класс **CloudAnalyticsClient**, упрощающий доступ к таблицам метрик в хранилище. 

> [!NOTE]
> Класс **CloudAnalyticsClient** не включен в клиентскую библиотеку хранилища BLOB-объектов Azure версии 12 для .NET. С **31 августа 2023** аналитика службы хранилища метрики, также называемые *классическими метриками* , будут прекращены. Дополнительные сведения см. в [официальном объявлении](https://azure.microsoft.com/updates/azure-storage-classic-metrics-will-be-retired-on-31-august-2023/). Если вы используете классические метрики, рекомендуется перейти к метрикам в Azure Monitor до этой даты. 

```csharp
private static void PrintMinuteMetrics(CloudAnalyticsClient analyticsClient, DateTimeOffset startDateTime, DateTimeOffset endDateTime)  
{  
 // Convert the dates to the format used in the PartitionKey.  
 var start = startDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  
 var end = endDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  

 var services = Enum.GetValues(typeof(StorageService));  
 foreach (StorageService service in services)  
 {  
     Console.WriteLine("Minute Metrics for Service {0} from {1} to {2} UTC", service, start, end);  
     var metricsQuery = analyticsClient.CreateMinuteMetricsQuery(service, StorageLocation.Primary);  
     var t = analyticsClient.GetMinuteMetricsTable(service);  
     var opContext = new OperationContext();  
     var query =  
             from entity in metricsQuery  
             // Note, you can't filter using the entity properties Time, AccessType, or TransactionType  
             // because they are calculated fields in the MetricsEntity class.  
             // The PartitionKey identifies the DataTime of the metrics.  
             where entity.PartitionKey.CompareTo(start) >= 0 && entity.PartitionKey.CompareTo(end) <= 0   
             select entity;  

     // Filter on "user" transactions after fetching the metrics from Azure Table storage.  
     // (StartsWith is not supported using LINQ with Azure Table storage.)  
     var results = query.ToList().Where(m => m.RowKey.StartsWith("user"));  
     var resultString = results.Aggregate(new StringBuilder(), (builder, metrics) => builder.AppendLine(MetricsString(metrics, opContext))).ToString();  
     Console.WriteLine(resultString);  
 }  
}  

private static string MetricsString(MetricsEntity entity, OperationContext opContext)  
{  
 var entityProperties = entity.WriteEntity(opContext);  
 var entityString =  
         string.Format("Time: {0}, ", entity.Time) +  
         string.Format("AccessType: {0}, ", entity.AccessType) +  
         string.Format("TransactionType: {0}, ", entity.TransactionType) +  
         string.Join(",", entityProperties.Select(e => new KeyValuePair<string, string>(e.Key.ToString(), e.Value.PropertyAsObject.ToString())));  
 return entityString;  
}  
```  

<a id="add-metrics-to-dashboard"></a>

## <a name="add-metrics-charts-to-the-portal-dashboard"></a>Добавление диаграмм метрик на панель мониторинга портала

Диаграммы метрик службы хранилища Azure для любых учетных записей хранения можно добавить на панель мониторинга портала.

1. Откройте панель мониторинга на [портале Azure](https://portal.azure.com) и щелкните **Изменить панель мониторинга**.
1. В колонке **Коллекция плиток** выберите **Найти плитки по** > **Тип**.
1. Выберите **Тип** > **Учетные записи хранения**.
1. В колонке **ресурсы** выберите учетную запись, метрики которой вы хотите добавить на панель мониторинга.
1. Выберите **Категории** > **Мониторинг**.
1. Перетащите на панель мониторинга элемент диаграммы для метрики, которую нужно отобразить. Повторите это для всех метрик, которые требуется отобразить на панели мониторинга. На следующем рисунке в качестве примера выбрана диаграмма "Большие двоичные объекты — всего запросов", но на панель мониторинга можно добавить любую диаграмму.

   ![Коллекция элементов на портале Azure](./media/manage-storage-analytics-metrics/storage-customize-dashboard.png)
1. Завершив добавление диаграммы, выберите **Настройка выполнена** в верхней части панели мониторинга.

После добавления диаграмм на панель мониторинга их можно настроить, как описано в разделе "Настройка диаграмм метрик".

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о Аналитика Службы хранилища см. в разделе [аналитика службы хранилища](storage-analytics.md) for аналитика службы хранилища.
* [Настройка журналов аналитика службы хранилища](manage-storage-analytics-logs.md).
* Дополнительные сведения о схеме метрик. См. раздел [схема таблицы метрик аналитика службы хранилища](/rest/api/storageservices/storage-analytics-metrics-table-schema).