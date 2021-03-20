---
title: Наблюдение за центром Интернета вещей Azure
description: Начните с этого раздела, чтобы узнать, как отслеживать центр Интернета вещей Azure
author: robinsh
ms.author: robinsh
ms.topic: conceptual
ms.service: iot-hub
ms.custom: subject-monitoring
ms.date: 11/06/2020
ms.openlocfilehash: c9e6107e981d2bae96c5d9f368c8122d3ce245f0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100570953"
---
# <a name="monitoring-azure-iot-hub"></a>Наблюдение за центром Интернета вещей Azure

При наличии критически важных приложений и бизнес-процессов, использующих ресурсы Azure, необходимо отслеживать эти ресурсы на предмет их доступности, производительности и работы. В этой статье описываются данные мониторинга, созданные центром Интернета вещей Azure, а также способы использования функций Azure Monitor для анализа и оповещения об этих данных.

## <a name="monitor-overview"></a>Общие сведения о мониторинге

На странице **Обзор** в портал Azure для каждого центра Интернета вещей содержатся диаграммы, которые предоставляют некоторые метрики использования, например количество используемых сообщений и число устройств, подключенных к центру Интернета вещей.

:::image type="content" source="media/monitor-iot-hub/overview-portal.png" alt-text="Диаграммы метрик по умолчанию на странице обзора центра Интернета вещей.":::

Имейте в виду, что значение числа сообщений может быть задержано на 1 минуту и, по соображениям, при работе с инфраструктурой службы центра Интернета вещей, иногда значение может передаваться между более высоким и меньшим значениями при обновлении. Этот счетчик должен быть неправильным только для значений, начисленных за последнюю минуту.

Сведения, представленные в области Обзор, полезны, но представляют лишь небольшой объем данных мониторинга, доступных для центра Интернета вещей. Некоторые данные мониторинга собираются автоматически и доступны для анализа сразу после создания центра Интернета вещей. Вы можете включить дополнительные типы сбора данных с помощью определенной конфигурации.

## <a name="what-is-azure-monitor"></a>Общие сведения об Azure Monitor

Центр Интернета вещей Azure создает данные мониторинга с помощью [Azure Monitor](../azure-monitor/overview.md), которая представляет собой полную службу мониторинга стека в Azure, которая предоставляет полный набор функций для мониторинга ресурсов Azure в дополнение к ресурсам в других облаках и локальных средах.

Начните со статьи [Мониторинг ресурсов Azure с помощью Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md), в которой описаны следующие понятия:

- Общие сведения об Azure Monitor
- затраты, связанные с мониторингом;
- данные мониторинга, собираемые в Azure;
- настройка сбора данных;
- стандартные средства Azure для анализа данных мониторинга и оповещения.

В следующих разделах описывается работа с данными, собранными для центра Интернета вещей Azure, и приводятся примеры настройки сбора данных и анализа этих данных с помощью средств Azure.

## <a name="monitoring-data"></a>Данные мониторинга

Центр Интернета вещей Azure собирает данные мониторинга тех же типов, что и другие ресурсы Azure, описанные в разделе [мониторинг данных из ресурсов Azure](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data).

Подробные сведения о метриках и журналах, созданных центром Интернета вещей Azure, см. в статье [мониторинг справочника по данным центра Интернета вещей Azure](monitor-iot-hub-reference.md) .

> [!IMPORTANT]
> Не гарантируется надежность или упорядочение событий, созданных службой центра Интернета вещей с помощью Azure Monitor журналов ресурсов. Некоторые события могут быть потеряны или доставлены не по порядку. Журналы ресурсов также не предназначены для работы в режиме реального времени, и для регистрации событий в выбранном месте может потребоваться несколько минут.

## <a name="collection-and-routing"></a>Сбор и маршрутизация

Метрики платформы и журнал действий собираются и сохраняются автоматически, но их можно направить в другие расположения с помощью параметра диагностики.

Журналы ресурсов не собираются и не сохраняются, пока вы не создадите параметр диагностики и не направите их в одно или несколько расположений.

Метрики и журналы можно направлять в несколько расположений, в том числе:
- Azure Monitor хранилище журналов с помощью связанной Log Analytics рабочей области. Там их можно проанализировать с помощью Log Analytics.
- Служба хранилища Azure для архивирования и автономного анализа 
- Конечная точка концентратора событий, в которой они могут считываться внешними приложениями, например сторонние средства SIEM.

В портал Azure можно выбрать **параметры диагностики** в разделе **мониторинг** на левой панели центра Интернета вещей, а затем **Добавить параметр диагностики** , чтобы создать параметры диагностики, областью действия которых являются журналы и метрики платформы, созданные центром Интернета вещей.

На следующем снимке экрана показан параметр диагностики для маршрутизации *операций подключения* типа журнала ресурсов и всех метрик платформы в log Analytics рабочей области.

:::image type="content" source="media/monitor-iot-hub/diagnostic-setting-portal.png" alt-text="Область параметров диагностики для центра Интернета вещей.":::

Подробный процесс создания параметров диагностики с помощью портала Azure, интерфейса командной строки или PowerShell см. в статье [Создание параметров диагностики для отправки журналов платформы и метрик в разные места назначения](../azure-monitor/essentials/diagnostic-settings.md). Создавая параметр диагностики, нужно указать, какие категории журналов должны собираться. Категории для центра Интернета вещей Azure перечислены в разделе [журналы ресурсов в справочнике по данным мониторинга центра Интернета вещей Azure](monitor-iot-hub-reference.md#resource-logs).

При маршрутизации метрик платформы центра Интернета вещей в другие расположения имейте в виду следующее.

- Следующие метрики платформы не могут быть экспортированы с помощью параметров диагностики: *подключенные устройства (Предварительная версия)* и *Общее число устройств (Предварительная версия)*.

- Многомерные метрики, например некоторые [метрики маршрутизации](monitor-iot-hub-reference.md#routing-metrics), в настоящее время экспортируются как плоские одномерные метрики, агрегированные по значениям измерений. Дополнительные сведения см. [в разделе Экспорт метрик платформы в другие расположения](../azure-monitor/essentials/metrics-supported.md#exporting-platform-metrics-to-other-locations).

## <a name="analyzing-metrics"></a>Анализ метрик

Вы можете анализировать метрики центра Интернета вещей Azure с метриками из других служб Azure, используя обозреватель метрик, открыв **метрики** из меню **Azure Monitor** . Подробные сведения об использовании этого средства см. в статье [Начало работы с обозревателем метрик Azure](../azure-monitor/essentials/metrics-getting-started.md).

В портал Azure можно выбрать **метрики** в разделе **мониторинг** на левой панели центра Интернета вещей, чтобы открыть обозреватель метрик по умолчанию для метрик платформы, созданных центром Интернета вещей:

:::image type="content" source="media/monitor-iot-hub/metrics-portal.png" alt-text="Страница обозревателя метрик для центра Интернета вещей.":::

Список метрик платформы, собранных для центра Интернета вещей Azure, см. [в разделе метрики в справочнике по данным мониторинга центра Интернета вещей Azure](monitor-iot-hub-reference.md#metrics). Список метрик платформы, собранных для всех служб Azure, см. в разделе [Поддерживаемые метрики с помощью Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

Для метрик платформы центра Интернета вещей, собранных в единицах счетчика, некоторые агрегаты могут быть недоступны или использоваться. Дополнительные сведения см. в разделе [Поддерживаемые статистические функции статьи мониторинг справочника по данным центра Интернета вещей Azure](monitor-iot-hub-reference.md#supported-aggregations).

Некоторые метрики центра Интернета вещей, такие как [метрики маршрутизации](monitor-iot-hub-reference.md#routing-metrics), являются многомерными. Для этих метрик можно применить [фильтры](../azure-monitor/essentials/metrics-charts.md#filters) и [разделение](../azure-monitor/essentials/metrics-charts.md#apply-splitting) на диаграммы на основе измерения.

## <a name="analyzing-logs"></a>анализ журналов;

Данные в журналах Azure Monitor хранятся в таблицах, и каждая таблица имеет собственный набор уникальных свойств. Данные в этих таблицах связаны с рабочей областью Log Analytics и могут запрашиваться в Log Analytics. Дополнительные сведения о журналах Azure Monitor см. в разделе [Общие сведения о Azure Monitor журналах](../azure-monitor/logs/data-platform-logs.md) в Azure Monitor документации. 

Чтобы направить данные в журналы Azure Monitor, необходимо создать параметр диагностики для отправки журналов ресурсов или метрик платформы в Log Analytics рабочую область. Дополнительные сведения см. в разделе [сбор и маршрутизация](#collection-and-routing).

В портал Azure можно выбрать **журналы** в разделе **мониторинг** на левой панели центра Интернета вещей, чтобы выполнять log Analytics запросы по умолчанию в журналы и метрики, собранные в журналах Azure Monitor для центра Интернета вещей.

:::image type="content" source="media/monitor-iot-hub/logs-portal.png" alt-text="Страница журналов для центра Интернета вещей.":::

Список таблиц, используемых Azure Monitor журналами и запрашиваемых Log Analytics, см. [в разделе таблицы Azure Monitor журналов статьи мониторинг справочника по данным центра Интернета вещей Azure](monitor-iot-hub-reference.md#azure-monitor-logs-tables).

Все журналы ресурсов в Azure Monitor имеют те же поля, за которыми следуют поля, связанные со службой. Общая схема описана в [Azure Monitor схеме журнала ресурсов](../azure-monitor/essentials/resource-logs-schema.md#top-level-common-schema). Схему и категории журналов ресурсов, собранных для центра Интернета вещей Azure, можно найти в [журналах ресурсов в справочнике по сведениям о мониторинге центра Интернета вещей Azure](monitor-iot-hub-reference.md#resource-logs).

[Журнал действий](../azure-monitor/essentials/activity-log.md) — это журнал платформы в Azure, который предоставляет подробные сведения о событиях уровня подписки. Вы можете просмотреть их независимо или направить в журналы Azure Monitor, где можно выполнять гораздо более сложные запросы с помощью Log Analytics.  

При маршрутизации метрик платформы центра Интернета вещей для Azure Monitor журналов имейте в виду следующее:

- Следующие метрики платформы не могут быть экспортированы с помощью параметров диагностики: *подключенные устройства (Предварительная версия)* и *Общее число устройств (Предварительная версия)*.

- Многомерные метрики, например некоторые [метрики маршрутизации](monitor-iot-hub-reference.md#routing-metrics), в настоящее время экспортируются как плоские одномерные метрики, агрегированные по значениям измерений. Дополнительные сведения см. [в разделе Экспорт метрик платформы в другие расположения](../azure-monitor/essentials/metrics-supported.md#exporting-platform-metrics-to-other-locations).

Некоторые распространенные запросы к центру Интернета вещей см. в статье [примеры запросов Kusto](#sample-kusto-queries). Подробные сведения об использовании запросов Log Analytics см. [в разделе Обзор запросов журналов в Azure Monitor](../azure-monitor/logs/log-query-overview.md).

### <a name="sdk-version-in-iot-hub-logs"></a>Версия пакета SDK в журналах центра Интернета вещей

Некоторые операции в журналах ресурсов центра Интернета вещей возвращают `sdkVersion` свойство в своем `properties` объекте. Для этих операций, когда устройство или серверное приложение использует один из пакетов SDK для Интернета вещей Azure, это свойство содержит сведения об используемом пакете SDK, версии пакета SDK и платформе, на которой выполняется пакет SDK. В следующем примере показано `sdkVersion` свойство, порожденное для [`deviceConnect`](monitor-iot-hub-reference.md#connections) операции при использовании пакета SDK для устройства Node.js: `"azure-iot-device/1.17.1 (node v10.16.0; Windows_NT 10.0.18363; x64)"` . Ниже приведен пример значения, созданного для .NET (C#) SDK: `".NET/1.21.2 (.NET Framework 4.8.4200.0; Microsoft Windows 10.0.17763 WindowsProduct:0x00000004; X86)"` .

В следующей таблице показано имя пакета SDK, используемого для различных пакетов SDK для Azure IoT.

| Имя пакета SDK в свойстве Сдкверсион | Язык |
|----------|----------|
| .NET | .NET (C#) |
| Microsoft. Azure. Devices | Пакет SDK для служб .NET (C#) |
| Microsoft. Azure. Devices. Client | Пакет SDK для устройств .NET (C#) |
| iothubclient | Пакет SDK для устройств C или Python v1 (не рекомендуется) |
| иосубсервицеклиент | Пакет SDK для C или Python v1 (не рекомендуется) |
| Azure-IOT-Device-iothub-корректировка | Пакет SDK для устройства Python |
| azure-iot-device | Пакет SDK для Node.js устройства |
| azure-iothub | Пакет SDK для службы Node.js |
| com. Microsoft. Azure. iothub-Java-Client | Пакет SDK для устройств Java |
| com. Microsoft. Azure. iothub. Service. SDK | Пакет SDK для службы Java |
| com. Microsoft. Azure. SDK. IOT. IOT-Device-Client | Пакет SDK для устройств Java |
| com. Microsoft. Azure. SDK. IOT. IOT-Service-Client | Пакет SDK для службы Java |
| C | Embedded C; |
| C + (Оссимплифиед = Azure RTO) | ОСРВ Azure |

Вы можете извлечь свойство версии пакета SDK при выполнении запросов к журналам ресурсов центра Интернета вещей. Например, следующий запрос извлекает свойство версии пакета SDK (и идентификатор устройства) из свойств, возвращаемых операциями подключения. Эти два свойства записываются в результаты вместе со временем операции и ИДЕНТИФИКАТОРом ресурса центра Интернета вещей, к которому подключается устройство.

```kusto
// SDK version of devices
// List of devices and their SDK versions that connect to IoT Hub
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
| where Category == "Connections"
| extend parsed_json = parse_json(properties_s) 
| extend SDKVersion = tostring(parsed_json.sdkVersion) , DeviceId = tostring(parsed_json.deviceId)
| distinct DeviceId, SDKVersion, TimeGenerated, _ResourceId
```

### <a name="sample-kusto-queries"></a>Примеры запросов Kusto

> [!IMPORTANT]
> При выборе **журналов** в меню центра интернета вещей log Analytics открывается с областью запроса, заданной для текущего центра Интернета вещей. Это означает, что запросы к журналам будут содержать данные только из этого ресурса. Если требуется выполнить запрос, который включает данные из других центров Интернета вещей или данных из других служб Azure, выберите **журналы** в меню **Azure Monitor** . Подробные сведения см. в статье [Область запросов журнала и временной диапазон в Azure Monitor Log Analytics](../azure-monitor/logs/scope.md).

Ниже приведены запросы, которые можно использовать для мониторинга центра Интернета вещей.

- Ошибки подключения: выявление ошибок подключения устройства.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" and Level == "Error"
    ```

- Ошибки регулирования: выявление устройств, которые выполнили наибольшее количество запросов, что приводит к ошибкам регулирования.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where ResultType == "429001"
    | extend DeviceId = tostring(parse_json(properties_s).deviceId)
    | summarize count() by DeviceId, Category, _ResourceId
    | order by count_ desc
    ```

- Неработающие конечные точки: выявление неработоспособных конечных точек по количеству, по которому была обнаружена Данная ошибка, а также причина.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Routes" and OperationName in ("endpointDead", "endpointUnhealthy")
    | extend parsed_json = parse_json(properties_s)
    | extend Endpoint = tostring(parsed_json.endpointName), Reason = tostring(parsed_json.details) 
    | summarize count() by Endpoint, OperationName, Reason, _ResourceId
    | order by count_ desc
    ```

- Сводка ошибок: количество ошибок по всем операциям по типу.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Level == "Error"
    | summarize count() by ResultType, ResultDescription, Category, _ResourceId
    ```

- Недавно подключенные устройства: список устройств, к которым центр Интернета вещей обнаружил подключение за указанный период времени.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" and OperationName == "deviceConnect"
    | extend DeviceId = tostring(parse_json(properties_s).deviceId)
    | summarize max(TimeGenerated) by DeviceId, _ResourceId
    ```

- Версия пакета SDK для устройств: список устройств и их версии пакета SDK для подключений устройств или устройств к облачным двойника операциям.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" or Category == "D2CTwinOperations"
    | extend parsed_json = parse_json(properties_s)
    | extend SDKVersion = tostring(parsed_json.sdkVersion) , DeviceId = tostring(parsed_json.deviceId)
    | distinct DeviceId, SDKVersion, TimeGenerated, _ResourceId
    ```

### <a name="read-logs-from-azure-event-hubs"></a>Чтение журналов из Центров событий Azure

После настройки ведения журнала событий в параметрах диагностики можно создавать приложения, которые считывают журналы, что позволяет выполнять действия на основе информации, содержащейся в них. В этом примере кода журналы извлекаются из концентратора событий:

```csharp
class Program
{ 
    static string connectionString = "{your AMS eventhub endpoint connection string}";
    static string monitoringEndpointName = "{your AMS event hub endpoint name}";
    static EventHubClient eventHubClient;
    //This is the Diagnostic Settings schema
    class AzureMonitorDiagnosticLog
    {
        string time { get; set; }
        string resourceId { get; set; }
        string operationName { get; set; }
        string category { get; set; }
        string level { get; set; }
        string resultType { get; set; }
        string resultDescription { get; set; }
        string durationMs { get; set; }
        string callerIpAddress { get; set; }
        string correlationId { get; set; }
        string identity { get; set; }
        string location { get; set; }
        Dictionary<string, string> properties { get; set; }
    };

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformationAsync().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }
        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }
            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));
            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
                var deserializer = new JavaScriptSerializer();
                //deserialize json data to azure monitor object
                AzureMonitorDiagnosticLog message = new JavaScriptSerializer().Deserialize<AzureMonitorDiagnosticLog>(result);
            }
        }
    }
}
```

## <a name="alerts"></a>видны узлы

Оповещения Azure Monitor заблаговременно уведомляют вас при обнаружении важных условий в данных мониторинга. Они позволяют выявлять и устранять проблемы в системе до того, как ваши клиенты заметят их. Оповещения можно настроить для [метрик](../azure-monitor/alerts/alerts-metric-overview.md), [журналов](../azure-monitor/alerts/alerts-unified-log.md) и [журнала действий](../azure-monitor/alerts/activity-log-alerts.md). Различные типы оповещений имеют преимущества и недостатки.

При создании правила генерации оповещений на основе метрик платформы следует учитывать, что для метрик платформы центра Интернета вещей, собранных в единицах счетчика, некоторые агрегаты могут быть недоступны или использоваться. Дополнительные сведения см. в разделе [Поддерживаемые статистические функции статьи мониторинг справочника по данным центра Интернета вещей Azure](monitor-iot-hub-reference.md#supported-aggregations).

## <a name="monitor-per-device-disconnects-with-event-grid"></a>Мониторинг отключения каждого устройства с помощью сетки событий

Azure Monitor предоставляет метрику, *подключенные устройства*, которые можно использовать для наблюдения за числом устройств, подключенных к центру Интернета вещей, и запуска оповещений, когда число подключенных устройств падает ниже порогового значения. Хотя это может быть достаточно для некоторых сценариев, служба " [Сетка событий Azure](../event-grid/index.yml) " предоставляет решение для мониторинга с низкой задержкой, которое можно использовать для отслеживания подключений устройств для критически важных устройств и инфраструктуры.

С помощью сетки событий вы можете подписываться на события [ **Девицеконнектед** и **девицедисконнектед**](iot-hub-event-grid.md#event-types) центра Интернета вещей, чтобы активировать предупреждения и отслеживать состояние подключения устройства. Сетка событий обеспечивает большую задержку событий, чем Azure Monitor, и вы можете наблюдать за каждым устройством, а не с общим числом подключенных устройств. Эти факторы делают сетку событий предпочтительным методом мониторинга подключений для критически важных устройств и инфраструктуры. Мы настоятельно рекомендуем использовать сетку событий для отслеживания подключений устройств в рабочих средах.

Более подробные сведения о мониторинге подключений устройств с помощью службы "Сетка событий" и Azure Monitor см. [в статье мониторинг, диагностика и устранение неполадок при отключении с помощью центра Интернета вещей Azure](iot-hub-troubleshoot-connectivity.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о метриках, журналах и других важных значениях, созданных [Service Name], см. в статье [мониторинг справочника по данным центра Интернета вещей Azure](monitor-iot-hub-reference.md) .

- Подробные сведения о мониторинге ресурсов Azure см. в статье [Мониторинг ресурсов Azure с помощью Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
