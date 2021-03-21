---
title: Поведение экспорта метрик с нулевыми и нулевыми значениями
description: Обсуждение значений NULL и нулей при экспорте метрик и указателя на список метрик, которые не могут быть экспортированы.
services: azure-monitor
ms.topic: reference
ms.date: 07/22/2020
ms.openlocfilehash: 47b98fe46ac1f2a3e2f3f1a8078ad9ca6f867554
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102048851"
---
# <a name="azure-monitor-platform-metrics-exportable-via-diagnostic-settings"></a>Метрики платформы Azure Monitor, которые можно экспортировать с помощью параметров диагностики

Azure Monitor предоставляет [метрики платформы](../essentials/data-platform-metrics.md) по умолчанию без конфигурации. Она обеспечивает несколько способов взаимодействия с метриками платформы, включая создание диаграмм метрик на портале, доступ к метрикам через REST API или запрос метрик с помощью PowerShell или интерфейса командной строки. Полный список метрик платформы, доступных в настоящее время с помощью конвейера консолидированной метрики Azure Monitor, см. в разделе [метрики — поддерживаемые](./metrics-supported.md). Чтобы запросить эти метрики и получить к ним доступ, воспользуйтесь [версией API 2018-01-01](/rest/api/monitor/metricdefinitions). Другие метрики могут быть доступны на портале или при использовании интерфейсов API прежних версий.

## <a name="metrics-not-exportable-via-diagnostic-settings"></a>Метрики, которые не экспортируются с помощью параметров диагностики

Содержимое, используемое в этом расположении, перемещено в [список поддерживаемых метрик Azure Monitor](./metrics-supported.md#exporting-platform-metrics-to-other-locations).

При экспорте метрик с помощью параметров диагностики существуют ограничения. Все метрики могут быть экспортированы с помощью REST API. 

## <a name="exported-zero-vs-null-values"></a>Экспортированные и нулевые значения NULL 

При работе со значениями 0 или NULL метрики имеют разную функциональность.  Некоторые метрики сообщают 0, если данные не получены, например метрики по ошибкам HTTP. Другие метрики хранят значения NULL, если данные не получены, так как это может означать, что ресурс находится в автономном режиме. Вы можете увидеть разницу при создании диаграммы этих метрик со значениями NULL, отображаемыми в виде [пунктирных линий](metrics-troubleshoot.md#chart-shows-dashed-line). 

Если метрики платформы можно экспортировать с помощью параметров диагностики, они соответствуют поведению метрики. То есть они экспортируют значения NULL, когда ресурс не отправляет данные.  Они экспортируют "0" только в том случае, если они действительно были выданы базовым ресурсом. 

При удалении группы ресурсов или определенного ресурса данные метрик из затронутых ресурсов больше не отправляются в целевые объекты экспорта параметров диагностики. Это значит, что он больше не отображается в концентраторах событий, учетных записях хранения Azure и Log Analytics рабочих областях.

### <a name="metrics-that-used-to-export-zero-for-null"></a>Метрики, которые использовались для экспорта нулевого значения для NULL

До 1 июня 2020 метрики ниже, **используемые для** выдачи "0", если данные отсутствуют. Затем эти нули можно экспортировать в последующие системы, такие как Log Analytics и служба хранилища Azure.  Такое поведение привело к путанице между реальным "0" (порожденным ресурсом) и интерпретированными знаками "0" (NULL), поэтому поведение было изменено таким образом, чтобы оно соответствовало значению базовой метрики, как упоминалось в предыдущем разделе. 

Это изменение произошло во всех общедоступных и частных облаках.

Изменение не повлияло на поведение любого из следующих интерфейсов: 
   - Журналы ресурсов платформы, экспортированные с помощью параметров диагностики
   - создание диаграмм метрик в обозревателе метрик;
   - предупреждения о метриках платформы.
 
Ниже приведен список метрик, поведение которых изменилось. 

| ResourceType                    | Метрика               |  MetricDisplayName  | 
|---------------------------------|----------------------|---------------------|
| Microsoft.ApiManagement/service | UnauthorizedRequests |  Неавторизованные запросы к шлюзу (не рекомендуется)  | 
| Microsoft.ApiManagement/service | TotalRequests |  Всего запросов к шлюзу (не рекомендуется)  | 
| Microsoft.ApiManagement/service | SuccessfulRequests |  Успешные запросы к шлюзу (не рекомендуется)  | 
| Microsoft.ApiManagement/service | Requests |  Requests  | 
| Microsoft.ApiManagement/service | OtherRequests |  Прочие запросы к шлюзу (не рекомендуется)  | 
| Microsoft.ApiManagement/service | FailedRequests |  Неудачные запросы к шлюзу (не рекомендуется)  | 
| Microsoft.ApiManagement/service | EventHubTotalFailedEvents |  События EventHub с ошибками  | 
| Microsoft.ApiManagement/service | EventHubTotalEvents |  Всего событий EventHub  | 
| Microsoft.ApiManagement/service | EventHubTotalBytesSent |  Размер событий EventHub  | 
| Microsoft.ApiManagement/service | EventHubTimedoutEvents |  События истекшего времени ожидания EventHub  | 
| Microsoft.ApiManagement/service | EventHubThrottledEvents |  События регулировки EventHub  | 
| Microsoft.ApiManagement/service | EventHubSuccessfulEvents |  Успешные события EventHub  | 
| Microsoft.ApiManagement/service | EventHubRejectedEvents |  Отклоненные события EventHub  | 
| Microsoft.ApiManagement/service | EventHubDroppedEvents |  Удаленные события EventHub  | 
| Microsoft.ApiManagement/service | Duration |  Общая длительность запросов шлюза  | 
| Microsoft.ApiManagement/service | BackendDuration |  Длительность внутренних запросов  | 
| Microsoft.DBforMariaDB/servers | storage_used |  Storage used  | 
| Microsoft.DBforMariaDB/servers | storage_percent |  Процент хранилища  | 
| Microsoft.DBforMariaDB/servers | storage_limit |  Storage limit  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_usage |  Используемый объем хранилища для журнала сервера  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_percent |  Процент хранилища для журнала сервера  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_limit |  Максимальный объем хранилища для журнала сервера  | 
| Microsoft.DBforMariaDB/servers | seconds_behind_master |  Задержка репликации в секундах  | 
| Microsoft.DBforMariaDB/servers | network_bytes_ingress |  Сеть (входящий трафик)  | 
| Microsoft.DBforMariaDB/servers | network_bytes_egress |  Сеть (исходящий трафик)  | 
| Microsoft.DBforMariaDB/servers | memory_percent |  Процент памяти  | 
| Microsoft.DBforMariaDB/servers | io_consumption_percent |  Процент ввода-вывода данных  | 
| Microsoft.DBforMariaDB/servers | cpu_percent |  Нагрузка ЦП  | 
| Microsoft.DBforMariaDB/servers | connections_failed |  Неудачные подключения  | 
| Microsoft.DBforMariaDB/servers | backup_storage_used |  Используемое хранилище резервных копий  | 
| Microsoft.DBforMariaDB/servers | active_connections |  Активные подключения  | 
| Microsoft.DBforMySQL/servers | storage_used |  Storage used  | 
| Microsoft.DBforMySQL/servers | storage_percent |  Процент хранилища  | 
| Microsoft.DBforMySQL/servers | storage_limit |  Storage limit  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_usage |  Используемый объем хранилища для журнала сервера  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_percent |  Процент хранилища для журнала сервера  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_limit |  Максимальный объем хранилища для журнала сервера  | 
| Microsoft.DBforMySQL/servers | seconds_behind_master |  Задержка репликации в секундах  | 
| Microsoft.DBforMySQL/servers | network_bytes_ingress |  Сеть (входящий трафик)  | 
| Microsoft.DBforMySQL/servers | network_bytes_egress |  Сеть (исходящий трафик)  | 
| Microsoft.DBforMySQL/servers | memory_percent |  Процент памяти  | 
| Microsoft.DBforMySQL/servers | io_consumption_percent |  Процент ввода-вывода данных  | 
| Microsoft.DBforMySQL/servers | cpu_percent |  Нагрузка ЦП  | 
| Microsoft.DBforMySQL/servers | connections_failed |  Неудачные подключения  | 
| Microsoft.DBforMySQL/servers | backup_storage_used |  Используемое хранилище резервных копий  | 
| Microsoft.DBforMySQL/servers | active_connections |  Активные подключения  | 
| Microsoft.Devices/Account | digitaltwins.telemetry.nodes |  Заполнитель данных телеметрии для узла Digital Twins  | 
| Microsoft.Devices/IotHubs | twinQueries.success |  Успешные запросы двойников.  | 
| Microsoft.Devices/IotHubs | twinQueries.resultSize |  Размер результатов запросов двойников.  | 
| Microsoft.Devices/IotHubs | twinQueries.failure |  Неудачные запросы двойников.  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.success |  Успешные запросы заданий.  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.failure |  Неудачные запросы заданий.  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.success |  Успешные вызовы получения списка заданий.  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.failure |  Неудачные вызовы получения списка заданий.  | 
| Microsoft.Devices/IotHubs | jobs.failed |  Неудачные задания.  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.success |  Успешные операции создания заданий обновления двойников.  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.failure |  Неудачные операции создания заданий обновления двойников.  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.success |  Успешные операции создания заданий вызова методов.  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.failure |  Неудачные операции создания заданий вызова методов.  | 
| Microsoft.Devices/IotHubs | jobs.completed |  Завершенные задания  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.success |  Успешные отмены заданий.  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.failure |  Неудачные отмены заданий.  | 
| Microsoft.Devices/IotHubs | EventGridLatency |  Задержка сетки событий (предварительная версия)  | 
| Microsoft.Devices/IotHubs | EventGridDeliveries |  Доставки сетки событий (предварительная версия)  | 
| Microsoft.Devices/IotHubs | devices.totalDevices |  Total devices (deprecated) (Всего устройств (не рекомендуется))  | 
| Microsoft.Devices/IotHubs | devices.connectedDevices.allProtocol |  Connected devices (deprecated) (Подключенные устройства (не рекомендуется))   | 
| Microsoft.Devices/IotHubs | deviceDataUsageV2 |  Total device data usage (preview) (Общий объем использования данных устройствами (предварительная версия))  | 
| Microsoft.Devices/IotHubs | deviceDataUsage |  Общий объем использования данных устройствами  | 
| Microsoft.Devices/IotHubs | dailyMessageQuotaUsed |  Общее количество используемых сообщений  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.success |  Успешные обновления двойников, инициированные устройством.  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.size |  Размер обновлений двойников, инициированных устройством.  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.failure |  Неудачные обновления двойников, инициированные устройством.  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.success |  Успешные операции чтения с двойников, инициированные устройством.  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.size |  Размер ответа операций чтения с двойников, инициированных устройством.  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.failure |  Неудачные операции чтения с двойников, инициированные устройством.  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.success |  Число отправленных сообщений телеметрии.  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.sendThrottle |  Количество ошибок регулирования  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.allProtocol |  Число предпринятых попыток отправки сообщений телеметрии.  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.success |  Routing: telemetry messages delivered (Маршрутизация: доставлено сообщений телеметрии)  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.orphaned |  Routing: telemetry messages orphaned (Маршрутизация: потеряно сообщений телеметрии)   | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.invalid |  Routing: telemetry messages incompatible (Маршрутизация: несовместимых сообщений телеметрии)  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.fallback |  Routing: messages delivered to fallback (Маршрутизация: доставлено сообщений на резервный маршрут)  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.dropped |  Routing: telemetry messages dropped (Маршрутизация: отброшено сообщений телеметрии)   | 
| Microsoft.Devices/IotHubs | d2c.Endpoints.latency.Storage |  Routing: message latency for storage (Маршрутизация: задержка сообщений для хранилища)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusTopics |  Routing: message latency for Service Bus Topic (Маршрутизация: задержка сообщений для раздела служебной шины)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusQueues |  Routing: message latency for Service Bus Queue (Маршрутизация: задержка сообщений для очереди служебной шины)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.eventHubs |  Routing: message latency for Event Hub (Маршрутизация: задержка сообщений для концентратора событий)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.builtIn.events |  Routing: message latency for messages/events (Маршрутизация: задержка сообщений для messages/events)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage.bytes |  Routing: data delivered to storage (Маршрутизация: доставлено данных в хранилище)  | 
| Microsoft.Devices/IotHubs | d2c.Endpoints.egress.Storage.BLOBs |  Routing: blobs delivered to storage (Маршрутизация: доставлено BLOB-объектов в хранилище)  | 
| Microsoft.Devices/IotHubs | d2c.Endpoints.egress.Storage |  Routing: messages delivered to storage (Маршрутизация: доставлено сообщений в хранилище)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusTopics |  Routing: messages delivered to Service Bus Topic (Маршрутизация: доставлено сообщений в раздел служебной шины)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusQueues |  Routing: messages delivered to Service Bus Queue (Маршрутизация: доставлено сообщений в очередь служебной шины)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.eventHubs |  Routing: messages delivered to Event Hub (Маршрутизация: доставлено сообщений в концентратор событий)  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.builtIn.events |  Routing: messages delivered to messages/events (Маршрутизация: доставлено сообщений на конечную точку messages/events)  | 
| Microsoft.Devices/IotHubs | конфигурации |  Configuration Metrics (Метрики конфигурации)  | 
| Microsoft.Devices/IotHubs | C2DMessagesExpired |  Сообщения из облака на устройство (C2D) с истекшим сроком действия (предварительная версия)  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.success |  Успешные обновления двойников, инициированные из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.size |  Размер обновлений двойников, инициированных из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.failure |  Неудачные обновления двойников, инициированные из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.success |  Успешные операции чтения с двойников, инициированные из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.size |  Размер ответа операций чтения с двойников, инициированных из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.failure |  Неудачные операции чтения с двойников, инициированные из серверной части.  | 
| Microsoft.Devices/IotHubs | c2d.methods.success |  Успешные вызовы прямых методов.  | 
| Microsoft.Devices/IotHubs | c2d.methods.responseSize |  Размер ответа вызовов прямых методов.  | 
| Microsoft.Devices/IotHubs | c2d.methods.requestSize |  Размер запроса вызовов прямых методов.  | 
| Microsoft.Devices/IotHubs | c2d.methods.failure |  Неудачные вызовы прямых методов.  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.reject.success |  Отклоненные сообщения из облака на устройство (C2D)  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.complete.success |  Доставленные сообщения из облака на устройство (C2D)  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.abandon.success |  Отброшенные сообщения из облака на устройство (C2D)  | 
| Microsoft.Devices/provisioningServices | RegistrationAttempts |  Попытки регистрации  | 
| Microsoft.Devices/provisioningServices | DeviceAssignments |  Назначенные устройства  | 
| Microsoft.Devices/provisioningServices | AttestationAttempts |  Попытки аттестации  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequestUnits |  Общее количество единиц запросов  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequests |  Общее количество запросов  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequests |  Запросы Mongo  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequestCharge |  Запросить оплату Mongo  | 
| Microsoft.EventGrid/domains | PublishSuccessLatencyInMs |  Задержка успешной публикации  | 
| Microsoft.EventGrid/domains | PublishSuccessCount |  Published Events (Опубликованные события)  | 
| Microsoft.EventGrid/domains | PublishFailCount |  События с ошибками публикации  | 
| Microsoft.EventGrid/domains | MatchedEventCount |  Соответствующие события  | 
| Microsoft.EventGrid/domains | DroppedEventCount |  Удаленные события  | 
| Microsoft.EventGrid/domains | DeliverySuccessCount |  Delivered Events (Доставленные события)  | 
| Microsoft.EventGrid/domains | DeadLetteredCount |  Dead Lettered Events (Невостребованные события)  | 
| Microsoft.EventGrid/eventSubscriptions | MatchedEventCount |  Соответствующие события  | 
| Microsoft.EventGrid/eventSubscriptions | DroppedEventCount |  Удаленные события  | 
| Microsoft.EventGrid/eventSubscriptions | DeliverySuccessCount |  Delivered Events (Доставленные события)  | 
| Microsoft.EventGrid/eventSubscriptions | DeadLetteredCount |  Dead Lettered Events (Невостребованные события)  | 
| Microsoft.EventGrid/extensionTopics | UnmatchedEventCount |  Unmatched Events (Несоответствующие события)  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessLatencyInMs |  Задержка успешной публикации  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessCount |  Published Events (Опубликованные события)  | 
| Microsoft.EventGrid/extensionTopics | PublishFailCount |  События с ошибками публикации  | 
| Microsoft.EventGrid/topics | UnmatchedEventCount |  Unmatched Events (Несоответствующие события)  | 
| Microsoft.EventGrid/topics | PublishSuccessLatencyInMs |  Задержка успешной публикации  | 
| Microsoft.EventGrid/topics | PublishSuccessCount |  Published Events (Опубликованные события)  | 
| Microsoft.EventGrid/topics | PublishFailCount |  События с ошибками публикации  | 
| Microsoft.EventHub/clusters | OutgoingMessages |  Исходящие сообщения  | 
| Microsoft.EventHub/clusters | OutgoingBytes |  Исходящие байты.  | 
| Microsoft.EventHub/clusters | IncomingRequests |  Входящих запросов  | 
| Microsoft.EventHub/clusters | IncomingMessages |  Входящие сообщения  | 
| Microsoft.EventHub/clusters | IncomingBytes |  Входящие байты.  | 
| Microsoft.EventHub/namespaces | SVRBSY |  Ошибки из-за занятости сервера (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | SUCCREQ |  Успешные запросы (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | OUTMSGS |  Исходящие сообщения (устаревшие) (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | OutgoingMessages |  Исходящие сообщения  | 
| Microsoft.EventHub/namespaces | OutgoingBytes |  Исходящие байты.  | 
| Microsoft.EventHub/namespaces | MISCERR |  Прочие ошибки (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | INTERR |  Внутренние ошибки сервера (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | INREQS |  Входящие запросы (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | INMSGS |  Входящие сообщения (устаревшие) (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | IncomingRequests |  Входящих запросов  | 
| Microsoft.EventHub/namespaces | IncomingMessages |  Входящие сообщения  | 
| Microsoft.EventHub/namespaces | IncomingBytes |  Входящие байты.  | 
| Microsoft.EventHub/namespaces | FAILREQ |  Неудачные запросы (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHOUTMSGS |  Исходящие сообщения (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHOUTMBS |  Исходящие байты (устаревшие) (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHOUTBYTES |  Исходящие байты (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHINMSGS |  Входящие сообщения (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHINMBS |  Входящие байты (устаревшие) (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHINBYTES |  Входящие байты (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHAMSGS |  Архивные сообщения (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHAMBS |  Пропускная способность архивных сообщений (не рекомендуется)  | 
| Microsoft.EventHub/namespaces | EHABL |  Архивные сообщения невыполненной работы (не рекомендуется)  | 
| Microsoft.HDInsight/clusters | NumActiveWorkers |  Число активных рабочих ролей  | 
| Microsoft.HDInsight/clusters | GatewayRequests |  Запросы к шлюзу  | 
| Microsoft.HDInsight/clusters | CategorizedGatewayRequests |  Запросы к шлюзу по категориям  | 
| Microsoft.Insights/AutoscaleSettings | ScaleActionsInitiated |  Инициированные действия масштабирования  | 
| Microsoft.Insights/Components | traces/count |  Трассировки  | 
| Microsoft.Insights/Components | performanceCounters/requestsPerSecond |  Скорость HTTP-запроса  | 
| Microsoft.Insights/Components | performanceCounters/requestsInQueue |  HTTP requests in application queue (HTTP-запросы в очереди приложений)  | 
| Microsoft.Insights/Components | performanceCounters/exceptionsPerSecond |  Частота исключений  | 
| Microsoft.Insights/Components | pageViews/count |  Просмотры страниц  | 
| Microsoft.Insights/Components | exceptions/count |  Исключения  | 
| Microsoft.Kusto/Clusters | StreamingIngestResults |  Результат потокового приема  | 
| Microsoft.Kusto/Clusters | StreamingIngestDuration |  Длительность потокового приема  | 
| Microsoft.Kusto/Clusters | StreamingIngestDataRate |  Скорость данных потокового приема  | 
| Microsoft.Kusto/Clusters | SteamingIngestRequestRate |  Частота запросов потокового приема  | 
| Microsoft.Kusto/Clusters | QueryDuration |  Query duration (Длительность запросов)  | 
| Microsoft.Kusto/Clusters | KeepAlive |  Проверка активности  | 
| Microsoft.Kusto/Clusters | IngestionVolumeInMB |  Объем приема (в МБ)  | 
| Microsoft.Kusto/Clusters | IngestionUtilization |  Использование приема  | 
| Microsoft.Kusto/Clusters | IngestionResult |  Результат приема  | 
| Microsoft.Kusto/Clusters | IngestionLatencyInSeconds |  Задержка приема (в секундах)  | 
| Microsoft.Kusto/Clusters | ExportUtilization |  Использование экспорта  | 
| Microsoft.Kusto/Clusters | EventsProcessedForEventHubs |  Обработанные события (для концентраторов событий и Центров Интернета вещей)  | 
| Microsoft.Kusto/Clusters | ЦП |  ЦП  | 
| Microsoft.Kusto/Clusters | ContinuousExportResult |  Результат непрерывного экспорта  | 
| Microsoft.Kusto/Clusters | ContinuousExportPendingCount |  Число ожидающих непрерывного экспорта  | 
| Microsoft.Kusto/Clusters | ContinuousExportNumOfRecordsExported |  Непрерывный экспорт — число экспортированных записей  | 
| Microsoft.Kusto/Clusters | ContinuousExportMaxLatenessMinutes |  Максимальная просрочка непрерывного экспорта в минутах  | 
| Microsoft.Kusto/Clusters | CacheUtilization |  Использование кэша  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggerThrottledEvents |  Trigger Throttled Events  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSucceeded |  Triggers Succeeded   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersStarted |  Triggers Started   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSkipped |  Triggers Skipped  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFired |  Triggers Fired   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFailed |  Triggers Failed   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersCompleted |  Triggers Completed   | 
| Microsoft.Logic/integrationServiceEnvironments | RunThrottledEvents |  Run Throttled Events  | 
| Microsoft.Logic/integrationServiceEnvironments | RunStartThrottledEvents |  События регулировки начала запуска  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsSucceeded |  Runs Succeeded  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsStarted |  Runs Started  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsFailed |  Runs Failed  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCompleted |  Runs Completed  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCancelled |  Отмененные запуски  | 
| Microsoft.Logic/integrationServiceEnvironments | RunFailurePercentage |  Run Failure Percentage  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionThrottledEvents |  Action Throttled Events  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSucceeded |  Actions Succeeded   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsStarted |  Actions Started   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSkipped |  Actions Skipped   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsFailed |  Actions Failed   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsCompleted |  Actions Completed   | 
| Microsoft.Logic/workflows | TriggerThrottledEvents |  Trigger Throttled Events  | 
| Microsoft.Logic/workflows | TriggersSucceeded |  Triggers Succeeded   | 
| Microsoft.Logic/workflows | TriggersStarted |  Triggers Started   | 
| Microsoft.Logic/workflows | TriggersSkipped |  Triggers Skipped  | 
| Microsoft.Logic/workflows | TriggersFired |  Triggers Fired   | 
| Microsoft.Logic/workflows | TriggersFailed |  Triggers Failed   | 
| Microsoft.Logic/workflows | TriggersCompleted |  Triggers Completed   | 
| Microsoft.Logic/workflows | TotalBillableExecutions |  Total Billable Executions  | 
| Microsoft.Logic/workflows | RunThrottledEvents |  Run Throttled Events  | 
| Microsoft.Logic/workflows | RunStartThrottledEvents |  События регулировки начала запуска  | 
| Microsoft.Logic/workflows | RunsSucceeded |  Runs Succeeded  | 
| Microsoft.Logic/workflows | RunsStarted |  Runs Started  | 
| Microsoft.Logic/workflows | RunsFailed |  Runs Failed  | 
| Microsoft.Logic/workflows | RunsCompleted |  Runs Completed  | 
| Microsoft.Logic/workflows | RunsCancelled |  Отмененные запуски  | 
| Microsoft.Logic/workflows | RunFailurePercentage |  Run Failure Percentage  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Выставление счетов за операции с использованием хранилища  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Выставление счетов за операции с использованием хранилища  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Выставление счетов за операции стандартного соединителя  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Выставление счетов за операции стандартного соединителя  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Выставление счетов за внутренние операции  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Выставление счетов за внутренние операции  | 
| Microsoft.Logic/workflows | BillableTriggerExecutions |  Billable Trigger Executions  | 
| Microsoft.Logic/workflows | BillableActionExecutions |  Billable Action Executions  | 
| Microsoft.Logic/workflows | ActionThrottledEvents |  Action Throttled Events  | 
| Microsoft.Logic/workflows | ActionsSucceeded |  Actions Succeeded   | 
| Microsoft.Logic/workflows | ActionsStarted |  Actions Started   | 
| Microsoft.Logic/workflows | ActionsSkipped |  Actions Skipped   | 
| Microsoft.Logic/workflows | ActionsFailed |  Actions Failed   | 
| Microsoft.Logic/workflows | ActionsCompleted |  Actions Completed   | 
| Microsoft.Network/frontdoors | WebApplicationFirewallRequestCount |  Web Application Firewall Request Count (Число запросов к брандмауэру веб-приложения)  | 
| Microsoft.Network/frontdoors | TotalLatency |  Total Latency (Общая задержка)  | 
| Microsoft.Network/frontdoors | ResponseSize |  Размер ответа  | 
| Microsoft.Network/frontdoors | RequestSize |  Размер запроса  | 
| Microsoft.Network/frontdoors | RequestCount |  Число запросов  | 
| Microsoft.Network/frontdoors | BillableResponseSize |  Оплачиваемый размер ответа  | 
| Microsoft.Network/frontdoors | BackendRequestLatency |  Backend Request Latency (Задержка запросов к серверным частям)  | 
| Microsoft.Network/frontdoors | BackendRequestCount |  Число запросов к серверному компоненту  | 
| Microsoft.Network/frontdoors | BackendHealthPercentage |  Работоспособность серверного компонента (в процентах)  | 
| Microsoft.Network/trafficManagerProfiles | QpsByEndpoint |  Запросы, выполняемые возвращаемой конечной точкой  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | scheduled.pending |  Запланированных уведомлений в состоянии ожидания  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.update |  Операции обновления регистрации  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.get |  Операции чтения регистрации  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.delete |  Операции удаления регистрации  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.create |  Операции создания регистрации  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.all |  Операции регистрации  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.wrongtoken |  Ошибки авторизации WNS (неправильный маркер)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.tokenproviderunreachable |  Ошибки авторизации WNS (нет доступа)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.throttled |  Регулируемые уведомления WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.success |  Успешные уведомления WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.pnserror |  Ошибки WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidtoken |  Ошибки авторизации WNS (недопустимый маркер)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationsize |  Ошибка из-за недопустимого размера уведомления WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationformat |  Недопустимый формат уведомления WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidcredentials |  Ошибки авторизации WNS (недопустимые учетные данные)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.expiredchannel |  Ошибка из-за просроченного канала WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.dropped |  Пропущенные уведомления WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channelthrottled |  Канал WNS регулируется  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channeldisconnected |  Канал WNS отсоединен  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.badchannel |  Ошибка из-за поврежденного канала WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.authenticationerror |  Ошибки проверки подлинности WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.throttled |  Регулируемые уведомления MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.success |  Успешные уведомления MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.pnserror |  Ошибки MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidnotificationformat |  Недопустимый формат уведомления MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidcredentials |  Недопустимые учетные данные MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.dropped |  Пропущенные уведомления MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.channeldisconnected |  Канал MPNS отсоединен  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.badchannel |  Ошибка из-за поврежденного канала MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.authenticationerror |  Ошибки проверки подлинности MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.wrongchannel |  Ошибка из-за неправильного канала GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.throttled |  Регулируемые уведомления GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.success |  Успешные уведомления GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.pnserror |  Ошибки GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationsize |  Ошибка из-за недопустимого размера уведомления GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationformat |  Недопустимый формат уведомления GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidcredentials |  Ошибки авторизации GCM (недопустимые учетные данные)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.expiredchannel |  Ошибка из-за просроченного канала GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.badchannel |  Ошибка из-за поврежденного канала GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.authenticationerror |  Ошибки проверки подлинности GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.success |  Успешные уведомления APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.pnserror |  Ошибки APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidnotificationsize |  Ошибка из-за недопустимого размера уведомления APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidcredentials |  Ошибки авторизации APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.expiredchannel |  Ошибка из-за просроченного канала APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.badchannel |  Ошибка из-за поврежденного канала APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.success |  Успешные уведомления  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.pnserror |  Ошибки внешней системы уведомлений  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.invalidpayload |  Ошибки полезных данных  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.channelerror |  Ошибки канала  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | notificationhub.pushes |  Все исходящие уведомления  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.upsert |  Операции создания или обновления установки  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.patch |  Операции исправления установки  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.get |  Операции получения установки  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.delete |  Операции удаления установки  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.all |  Операции управления установкой  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled.cancel |  Отменено запланированных push-уведомлений  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled |  Отправлено запланированных push-уведомлений  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.all.requests |  Все входящие запросы  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | Incoming.all.failedrequests |  Все неудачные входящие запросы  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming |  Входящие сообщения  | 
| Microsoft.OperationalInsights/workspaces | Пульс |  Пульс  | 
| Microsoft.Relay/namespaces | BytesTransferred |  BytesTransferred  | 
| Microsoft.ServiceBus/namespaces | OutgoingMessages |  Исходящие сообщения  | 
| Microsoft.ServiceBus/namespaces | IncomingRequests |  Входящих запросов  | 
| Microsoft.ServiceBus/namespaces | IncomingMessages |  Входящие сообщения  | 
| Microsoft.SignalRService/SignalR | UserErrors |  Ошибки пользователей  | 
| Microsoft.SignalRService/SignalR | SystemErrors |  Системные ошибки  | 
| Microsoft.SignalRService/SignalR | OutboundTraffic |  Outbound Traffic (Исходящий трафик)  | 
| Microsoft.SignalRService/SignalR | MessageCount |  Количество сообщений  | 
| Microsoft.SignalRService/SignalR | InboundTraffic |  Inbound Traffic (Входящий трафик)  | 
| Microsoft.SignalRService/SignalR | ConnectionCount |  Число подключений  | 
| Microsoft.Sql/managedInstances | avg_cpu_percent |  Average CPU percentage (Средний процент использования ЦП)  | 
| Microsoft.Sql/servers | dtu_used |  DTU used  | 
| Microsoft.Sql/servers | dtu_consumption_percent |  Процент использования DTU  | 
| Microsoft.Sql/servers/databases | xtp_storage_percent |  Процент хранилища выполняющейся в памяти OLTP  | 
| Microsoft.Sql/servers/databases | workers_percent |  Workers percentage  | 
| Microsoft.Sql/servers/databases | sessions_percent |  Процент использования сеансов  | 
| Microsoft.Sql/servers/databases | physical_data_read_percent |  Процент операций ввода/вывода данных  | 
| Microsoft.Sql/servers/databases | log_write_percent |  Log IO percentage  | 
| Microsoft.Sql/servers/databases | dwu_used |  DWU used  | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent |  DWU percentage  | 
| Microsoft.Sql/servers/databases | dtu_used |  DTU used  | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent |  Процент использования DTU  | 
| Microsoft.Sql/servers/databases | взаимоблокировка |  Взаимоблокировки  | 
| Microsoft.Sql/servers/databases | cpu_used |  Загрузка ЦП  | 
| Microsoft.Sql/servers/databases | cpu_percent |  Процент использования ЦП  | 
| Microsoft.Sql/servers/databases | connection_successful |  Успешные подключения  | 
| Microsoft.Sql/servers/databases | connection_failed |  Неудачные подключения  | 
| Microsoft.Sql/servers/databases | cache_hit_percent |  Cache hit percentage (Процент попаданий в кэш)  | 
| Microsoft.Sql/servers/databases | blocked_by_firewall |  Заблокировано брандмауэром  | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent |  Процент хранилища выполняющейся в памяти OLTP  | 
| Microsoft.Sql/servers/elasticPools | workers_percent |  Workers percentage  | 
| Microsoft.Sql/servers/elasticPools | sessions_percent |  Процент использования сеансов  | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent |  Процент операций ввода/вывода данных  | 
| Microsoft.Sql/servers/elasticPools | log_write_percent |  Log IO percentage  | 
| Microsoft.Sql/servers/elasticPools | eDTU_used |  eDTU used  | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent |  Процент использования DTU  | 
| Microsoft.Sql/servers/elasticPools | cpu_percent |  Процент использования ЦП  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | TotalFrontEnds |  Общее число внешних интерфейсов  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | SmallAppServicePlanInstances |  Рабочие роли плана службы приложений уровня "Малый"  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Requests |  Requests  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MemoryPercentage |  Процент использования памяти  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MediumAppServicePlanInstances |  Рабочие процессы плана службы приложений уровня "Средний"  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | LargeAppServicePlanInstances |  Рабочие процессы плана службы приложений уровня "Крупный"  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | HttpQueueLength |  Длина очереди HTTP:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http5xx |  Ошибки HTTP-сервера:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http4xx |  HTTP 4xx:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http406 |  HTTP 406:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http404 |  HTTP 404:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http403 |  HTTP 403:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http401 |  HTTP 401:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http3xx |  HTTP 3xx:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http2xx |  HTTP 2xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http101 |  Http 101  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | DiskQueueLength |  Длина дисковой очереди  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | CpuPercentage |  Процент использования ЦП  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesSent |  Выходные данные  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesReceived |  Входящие данные:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | AverageResponseTime |  Среднее время ответа:  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | ActiveRequests |  Активные запросы  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersUsed |  Использованные рабочие процессы  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersTotal |  Общее количество рабочих процессов  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersAvailable |  Доступные рабочие процессы  | 
| Microsoft.Web/hostingEnvironments/workerPools | MemoryPercentage |  Процент использования памяти  | 
| Microsoft.Web/hostingEnvironments/workerPools | CpuPercentage |  Процент использования ЦП  | 
| Microsoft.Web/serverfarms | TcpTimeWait |  Состояние TCP TIME-WAIT  | 
| Microsoft.Web/serverfarms | TcpSynSent |  Состояние TCP SYN-SENT  | 
| Microsoft.Web/serverfarms | TcpSynReceived |  Состояние TCP SYN-RECEIVED  | 
| Microsoft.Web/serverfarms | TcpLastAck |  Состояние TCP LAST-ACK  | 
| Microsoft.Web/serverfarms | TcpFinWait2 |  Состояние TCP FIN-WAIT-2  | 
| Microsoft.Web/serverfarms | TcpFinWait1 |  Состояние TCP FIN-WAIT-1  | 
| Microsoft.Web/serverfarms | TcpEstablished |  Состояние TCP ESTABLISHED  | 
| Microsoft.Web/serverfarms | TcpClosing |  Состояние TCP CLOSING  | 
| Microsoft.Web/serverfarms | TcpCloseWait |  Состояние TCP CLOSE-WAIT  | 
| Microsoft.Web/serverfarms | MemoryPercentage |  Процент использования памяти  | 
| Microsoft.Web/serverfarms | HttpQueueLength |  Длина очереди HTTP:  | 
| Microsoft.Web/serverfarms | DiskQueueLength |  Длина дисковой очереди  | 
| Microsoft.Web/serverfarms | CpuPercentage |  Процент использования ЦП  | 
| Microsoft.Web/serverfarms | BytesSent |  Выходные данные  | 
| Microsoft.Web/serverfarms | BytesReceived |  Входящие данные:  | 
| Microsoft.Web/sites | TotalAppDomainsUnloaded |  Всего выгруженных доменов приложений  | 
| Microsoft.Web/sites | TotalAppDomains |  Всего доменов приложений  | 
| Microsoft.Web/sites | Потоки |  Счетчик потоков  | 
| Microsoft.Web/sites | RequestsInApplicationQueue |  Запросов в очереди приложений  | 
| Microsoft.Web/sites | Requests |  Requests  | 
| Microsoft.Web/sites | PrivateBytes |  Байты исключительного пользования  | 
| Microsoft.Web/sites | MemoryWorkingSet |  рабочий набор памяти;  | 
| Microsoft.Web/sites | IoWriteOperationsPerSecond |  Операции ввода-вывода: операций записи в секунду  | 
| Microsoft.Web/sites | IoWriteBytesPerSecond |  Операции ввода-вывода: запись, байт в секунду  | 
| Microsoft.Web/sites | IoReadOperationsPerSecond |  Операции ввода-вывода: операций чтения в секунду  | 
| Microsoft.Web/sites | IoReadBytesPerSecond |  Операции ввода-вывода: чтение, байт в секунду  | 
| Microsoft.Web/sites | IoOtherOperationsPerSecond |  Операции ввода-вывода: прочих операций в секунду  | 
| Microsoft.Web/sites | IoOtherBytesPerSecond |  Операции ввода-вывода: прочие, байт в секунду  | 
| Microsoft.Web/sites | HttpResponseTime |  Время ответа  | 
| Microsoft.Web/sites | Http5xx |  Ошибки HTTP-сервера:  | 
| Microsoft.Web/sites | Http4xx |  HTTP 4xx:  | 
| Microsoft.Web/sites | Http406 |  HTTP 406:  | 
| Microsoft.Web/sites | Http404 |  HTTP 404:  | 
| Microsoft.Web/sites | Http403 |  HTTP 403:  | 
| Microsoft.Web/sites | Http401 |  HTTP 401:  | 
| Microsoft.Web/sites | Http3xx |  HTTP 3xx:  | 
| Microsoft.Web/sites | Http2xx |  HTTP 2xx  | 
| Microsoft.Web/sites | Http101 |  Http 101  | 
| Microsoft.Web/sites | HealthCheckStatus |  Состояние проверки работоспособности  | 
| Microsoft.Web/sites | Маркеры |  Счетчик дескрипторов  | 
| Microsoft.Web/sites | Gen2Collections |  Сборок мусора поколения 2  | 
| Microsoft.Web/sites | Gen1Collections |  Сборок мусора поколения 1  | 
| Microsoft.Web/sites | Gen0Collections |  Сборок мусора поколения 0  | 
| Microsoft.Web/sites | FunctionExecutionUnits |  Function Execution Units  | 
| Microsoft.Web/sites | FunctionExecutionCount |  Function Execution Count  | 
| Microsoft.Web/sites | CurrentAssemblies |  Текущее число сборок  | 
| Microsoft.Web/sites | CpuTime |  Время ЦП:  | 
| Microsoft.Web/sites | BytesSent |  Выходные данные  | 
| Microsoft.Web/sites | BytesReceived |  Входящие данные:  | 
| Microsoft.Web/sites | AverageResponseTime |  Среднее время ответа:  | 
| Microsoft.Web/sites | AverageMemoryWorkingSet |  средний размер рабочего набора памяти;  | 
| Microsoft.Web/sites | AppConnections |  Соединения  | 
| Microsoft.Web/sites/slots | TotalAppDomainsUnloaded |  Всего выгруженных доменов приложений  | 
| Microsoft.Web/sites/slots | TotalAppDomains |  Всего доменов приложений  | 
| Microsoft.Web/sites/slots | Потоки |  Счетчик потоков  | 
| Microsoft.Web/sites/slots | RequestsInApplicationQueue |  Запросов в очереди приложений  | 
| Microsoft.Web/sites/slots | Requests |  Requests  | 
| Microsoft.Web/sites/slots | PrivateBytes |  Байты исключительного пользования  | 
| Microsoft.Web/sites/slots | MemoryWorkingSet |  рабочий набор памяти;  | 
| Microsoft.Web/sites/slots | IoWriteOperationsPerSecond |  Операции ввода-вывода: операций записи в секунду  | 
| Microsoft.Web/sites/slots | IoWriteBytesPerSecond |  Операции ввода-вывода: запись, байт в секунду  | 
| Microsoft.Web/sites/slots | IoReadOperationsPerSecond |  Операции ввода-вывода: операций чтения в секунду  | 
| Microsoft.Web/sites/slots | IoReadBytesPerSecond |  Операции ввода-вывода: чтение, байт в секунду  | 
| Microsoft.Web/sites/slots | IoOtherOperationsPerSecond |  Операции ввода-вывода: прочих операций в секунду  | 
| Microsoft.Web/sites/slots | IoOtherBytesPerSecond |  Операции ввода-вывода: прочие, байт в секунду  | 
| Microsoft.Web/sites/slots | HttpResponseTime |  Время ответа  | 
| Microsoft.Web/sites/slots | Http5xx |  Ошибки HTTP-сервера:  | 
| Microsoft.Web/sites/slots | Http4xx |  HTTP 4xx:  | 
| Microsoft.Web/sites/slots | Http406 |  HTTP 406:  | 
| Microsoft.Web/sites/slots | Http404 |  HTTP 404:  | 
| Microsoft.Web/sites/slots | Http403 |  HTTP 403:  | 
| Microsoft.Web/sites/slots | Http401 |  HTTP 401:  | 
| Microsoft.Web/sites/slots | Http3xx |  HTTP 3xx:  | 
| Microsoft.Web/sites/slots | Http2xx |  HTTP 2xx  | 
| Microsoft.Web/sites/slots | Http101 |  Http 101  | 
| Microsoft.Web/sites/slots | HealthCheckStatus |  Состояние проверки работоспособности  | 
| Microsoft.Web/sites/slots | Маркеры |  Счетчик дескрипторов  | 
| Microsoft.Web/sites/slots | Gen2Collections |  Сборок мусора поколения 2  | 
| Microsoft.Web/sites/slots | Gen1Collections |  Сборок мусора поколения 1  | 
| Microsoft.Web/sites/slots | Gen0Collections |  Сборок мусора поколения 0  | 
| Microsoft.Web/sites/slots | FunctionExecutionUnits |  Function Execution Units  | 
| Microsoft.Web/sites/slots | FunctionExecutionCount |  Function Execution Count  | 
| Microsoft.Web/sites/slots | CurrentAssemblies |  Текущее число сборок  | 
| Microsoft.Web/sites/slots | CpuTime |  Время ЦП:  | 
| Microsoft.Web/sites/slots | BytesSent |  Выходные данные  | 
| Microsoft.Web/sites/slots | BytesReceived |  Входящие данные:  | 
| Microsoft.Web/sites/slots | AverageResponseTime |  Среднее время ответа:  | 
| Microsoft.Web/sites/slots | AverageMemoryWorkingSet |  средний размер рабочего набора памяти;  | 
| Microsoft.Web/sites/slots | AppConnections |  Соединения  | 
| Microsoft.Sql/servers/databases | cpu_percent | Процент использования ЦП | 
| Microsoft.Sql/servers/databases | physical_data_read_percent | Процент операций ввода/вывода данных | 
| Microsoft.Sql/servers/databases | log_write_percent | Log IO percentage | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent | Процент использования DTU | 
| Microsoft.Sql/servers/databases | connection_successful | Успешные подключения | 
| Microsoft.Sql/servers/databases | connection_failed | Неудачные подключения | 
| Microsoft.Sql/servers/databases | blocked_by_firewall | Заблокировано брандмауэром | 
| Microsoft.Sql/servers/databases | взаимоблокировка | Взаимоблокировки | 
| Microsoft.Sql/servers/databases | xtp_storage_percent | Процент хранилища выполняющейся в памяти OLTP | 
| Microsoft.Sql/servers/databases | workers_percent | Workers percentage | 
| Microsoft.Sql/servers/databases | sessions_percent | Процент использования сеансов | 
| Microsoft.Sql/servers/databases | dtu_used | DTU used | 
| Microsoft.Sql/servers/databases | cpu_used | Загрузка ЦП | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent | DWU percentage | 
| Microsoft.Sql/servers/databases | dwu_used | DWU used | 
| Microsoft.Sql/servers/databases | cache_hit_percent | Cache hit percentage (Процент попаданий в кэш) | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_system_percent | Распределение группы рабочей нагрузки по процентам системы | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_wlg_effective_cap_percent | Распределение группы рабочей нагрузки по максимальному проценту ресурсов | 
| Microsoft.Sql/servers/databases | wlg_active_queries | Активные запросы группы рабочей нагрузки | 
| Microsoft.Sql/servers/databases | wlg_queued_queries | Запросы в очереди группы рабочей нагрузки | 
| Microsoft.Sql/servers/databases | active_queries | Активные запросы | 
| Microsoft.Sql/servers/databases | queued_queries | Запросы в очереди | 
| Microsoft.Sql/servers/databases | wlg_active_queries_timeouts | Время ожидания запросов группы рабочей нагрузки | 
| Microsoft.Sql/servers/databases | wlg_queued_queries_timeouts | Время ожидания запросов в очереди группы рабочей нагрузки | 
| Microsoft.Sql/servers/databases | wlg_effective_min_resource_percent | Действующий минимальный процент ресурсов | 
| Microsoft.Sql/servers/databases | wlg_effective_cap_resource_percent | Действующее ограничение процента ресурсов | 
| Microsoft.Sql/servers/elasticPools | cpu_percent | Процент использования ЦП | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent | Процент операций ввода/вывода данных | 
| Microsoft.Sql/servers/elasticPools | log_write_percent | Log IO percentage | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent | Процент использования DTU | 
| Microsoft.Sql/servers/elasticPools | workers_percent | Workers percentage | 
| Microsoft.Sql/servers/elasticPools | sessions_percent | Процент использования сеансов | 
| Microsoft.Sql/servers/elasticPools | eDTU_used | eDTU used | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent | Процент хранилища выполняющейся в памяти OLTP | 
| Microsoft.Sql/servers | dtu_consumption_percent | Процент использования DTU | 
| Microsoft.Sql/servers | dtu_used | DTU used | 
| Microsoft.Sql/managedInstances | avg_cpu_percent | Average CPU percentage (Средний процент использования ЦП) |