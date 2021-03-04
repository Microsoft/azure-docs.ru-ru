---
title: Поддерживаемые службы и категории журналов Azure Monitor ресурсов
description: Справочник по Azure Monitor сведения о поддерживаемых службах и схеме событий для журналов ресурсов Azure.
ms.topic: reference
ms.date: 01/29/2021
ms.openlocfilehash: 9a04d0f470522dd4689d604756ffd25e70c5d456
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102033152"
---
# <a name="supported-categories-for-azure-resource-logs"></a>Поддерживаемые категории для журналов ресурсов Azure

> [!NOTE]
> Журналы ресурсов ранее назывались журналами диагностики. Имя было изменено в октябре 2019, так как типы журналов, собранные Azure Monitor перемещены, чтобы включать больше, чем просто ресурс Azure.

[Azure Monitor журналы ресурсов](../essentials/platform-logs-overview.md) — это журналы, создаваемые службами Azure, которые описывают работу этих служб или ресурсов. Все журналы ресурсов, доступные через Azure Monitor, используют общую схему верхнего уровня, обеспечивая гибкость для каждой службы, чтобы создавать уникальные свойства для собственных событий.

Сочетание типа ресурса (доступного в свойстве `resourceId`) и свойства `category` является уникальным идентификатором схемы. Существует общая схема для всех журналов ресурсов с полями для конкретных служб, которые затем добавляются для различных категорий журналов. Дополнительные сведения см. в статьях [Общие и зависящие от службы схемы для журналов ресурсов Azure]() .


## <a name="costs"></a>Затраты

Существуют затраты, связанные с отправкой и хранением данных в Log Analytics, службе хранилища Azure и (или) концентраторе событий. Вы можете платить за затраты на получение данных в эти места и на их хранение.  Журналы ресурсов — это один из типов данных, которые можно отправить в эти расположения. 

Экспорт некоторых категорий журналов ресурсов в эти расположения является дополнительной ценой. Эти журналы с затратами на экспорт перечислены в таблице ниже. Дополнительные сведения об этих ценах см. в разделе журналы платформы на [странице цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="supported-log-categories-per-resource-type"></a>Поддерживаемые категории журнала для каждого типа ресурса

Ниже приведен список типов журналов, доступных для каждого типа ресурсов. 

Некоторые категории могут поддерживаться только для определенных типов ресурсов. Если вы считаете, что ресурс отсутствует, см. документацию для конкретного ресурса. Например, категории Microsoft. SQL/Servers/databases недоступны для всех типов баз данных. Дополнительные сведения см. [в разделе сведения о журнале диагностики базы данных SQL](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md). 

Если вы считаете, что что-то отсутствует, можно открыть комментарий GitHub в нижней части этой статьи.
## <a name="microsoftaaddomainservices"></a>Microsoft. AAD/domainServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|аккаунтлогон|аккаунтлогон|Нет|
|AccountManagement|AccountManagement|Нет|
|детаилтраккинг|детаилтраккинг|Нет|
|директорисервицеакцесс|директорисервицеакцесс|Нет|
|логонлогофф|логонлогофф|Нет|
|обжектакцесс|обжектакцесс|Нет|
|полицичанже|полицичанже|Нет|
|привилежеусе|привилежеусе|Нет|
|системсекурити|системсекурити|Нет|


## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Подсистема|Подсистема|Нет|
|Служба|Служба|Нет|


## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|GatewayLogs|Журналы, относящихся к шлюзу ApiManagement.|Нет|


## <a name="microsoftappconfigurationconfigurationstores"></a>Microsoft.AppConfiguration/configurationStores

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|HttpRequest|HTTP-запросы|Да|


## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|аппликатионконсоле|Консоль приложения|Нет|
|системлогс|Системные журналы|Нет|


## <a name="microsoftattestationattestationproviders"></a>Microsoft.Attestation/attestationProviders

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AuditEvent|Категория журнала сообщений AuditEvent.|Нет|
|ERR|Категория журнала сообщений об ошибках.|Нет|
|INF|Категория журнала информационных сообщений.|Нет|
|WRN|Категория журнала предупреждающих сообщений.|Нет|


## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|DscNodeStatus|Состояние узла DSC.|Нет|
|JobLogs|Журналы заданий|Нет|
|JobStreams|Потоки заданий|Нет|


## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ServiceLog|Журналы служб|Нет|


## <a name="microsoftbatchaiworkspaces"></a>Microsoft.BatchAI/workspaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|баиклустеревент|баиклустеревент|Нет|
|баиклустернодивент|баиклустернодивент|Нет|
|баижобевент|баижобевент|Нет|


## <a name="microsoftblockchainblockchainmembers"></a>Microsoft.Blockchain/blockchainMembers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|блоккчаинаппликатион|Приложение блокчейн|Нет|
|фабрикордерер|Структурный заказ|Нет|
|фабрикпир|Одноранговый узел структуры|Нет|
|Proxy (Прокси)|Proxy (Прокси)|Нет|


## <a name="microsoftblockchaincordamembers"></a>Microsoft. Блокчейн/Кордамемберс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|блоккчаинаппликатион|Приложение блокчейн|Нет|


## <a name="microsoftbotservicebotservices"></a>microsoft.botservice/botservices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ботрекуест|Запросы от каналов к Bot|Нет|
|депенденцирекуест|Запросы к зависимостям|Нет|


## <a name="microsoftcdncdnwebapplicationfirewallpolicies"></a>Microsoft. CDN/кднвебаппликатионфиреваллполиЦиес

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|вебаппликатионфиревалллогс|Журналы брандмауэра веб-работающие|Нет|


## <a name="microsoftcdnprofiles"></a>Microsoft.Cdn/profiles

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|азурекднакцесслог|Журнал доступа Azure CDN|Нет|
|фронтдуракцесслог|Журнал доступа FrontDoor|Да|
|фронтдурхеалспробелог|Журнал зонда работоспособности FrontDoor|Да|
|фронтдурвебаппликатионфиревалллог|Журнал FrontDoor Вебаппликатионфиревалл|Да|


## <a name="microsoftcdnprofilesendpoints"></a>Microsoft.Cdn/profiles/endpoints

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|CoreAnalytics|Возвращает метрики конечной точки, например пропускную способность, исходящий трафик и т. д.|Нет|


## <a name="microsoftclassicnetworknetworksecuritygroups"></a>Microsoft.ClassicNetwork/networksecuritygroups

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Событие потока правил группы безопасности сети|Событие потока правил группы безопасности сети|Нет|


## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Журналы аудита|Нет|
|RequestResponse|Журналы запросов и ответов|Нет|
|Трассировка|Журналы трассировки|Нет|


## <a name="microsoftcommunicationcommunicationservices"></a>Microsoft. Communication/Коммуникатионсервицес

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|чатоператионал|Журналы операционного разговора|Нет|
|смсоператионал|Рабочие журналы SMS|Нет|
|Использование|Записи об использовании|Нет|


## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ContainerRegistryLoginEvents|События входа|Нет|
|ContainerRegistryRepositoryEvents|Журналы Репоситоревент|Нет|


## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|cluster-autoscaler|Средство автомасштабирования кластера Kubernetes|Нет|
|guard|guard|Нет|
|kube-apiserver|Сервер API Kubernetes|Нет|
|KUBE — аудит|Аудит Kubernetes|Нет|
|KUBE-Audit-Admin|Журналы администратора аудита Kubernetes|Нет|
|kube-controller-manager|Диспетчер контроллеров Kubernetes|Нет|
|kube-scheduler|Планировщик Kubernetes|Нет|


## <a name="microsoftcustomprovidersresourceproviders"></a>Microsoft. Кустомпровидерс/ресаурцепровидерс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AuditLogs|Журналы аудита для вызовов Минирп|Нет|


## <a name="microsoftd365customerinsightsinstances"></a>Microsoft. D365CustomerInsights/экземпляры

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Аудит событий|Нет|
|Операционный|Операционные события|Нет|


## <a name="microsoftdatabricksworkspaces"></a>Microsoft. кирпичы и рабочие области

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|accounts|Учетные записи кирпичей|нет|
|clusters|Кластеры кирпичей|Нет|
|dBFS|Файловая система Databricks|Нет|
|инстанцепулс|Пулы экземпляров|нет|
|jobs|Задания кирпичей|Нет|
|записная книжка|Записная книжка Databricks|нет|
|секретные коды|Секретные данные кирпичей|Нет|
|склпермиссионс|Склпермиссионсы блоков|Нет|
|ssh|SSH-сеансы|Нет|
|Рабочая область|Рабочая область "кирпичи"|Нет|


## <a name="microsoftdatacollaborationworkspaces"></a>Microsoft. сотрудничество и рабочие области

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|коллаборатионаудит|Аудит совместной работы|Да|
|Ресурсы ресурсов|Ресурсы данных|Нет|
|Конвейеры|Конвейеры|Нет|
|Предложений|Предложений|Нет|
|Сценарии|Сценарии|Нет|


## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories;

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ActivityRuns|Журнал выполнения действий конвейера|Нет|
|PipelineRuns|Журнал запусков конвейера|Нет|
|ссисинтегратионрунтимелогс|Журналы среды выполнения интеграции SSIS|Нет|
|ссиспаккажеевентмессажеконтекст|Контекст сообщения о событиях пакета служб SSIS|Нет|
|ссиспаккажеевентмессажес|Сообщения о событиях пакета служб SSIS|Нет|
|ссиспаккажеексекутаблестатистикс|Статистика исполняемых файлов пакета служб SSIS|Нет|
|ссиспаккажеексекутионкомпонентфасес|Этапы компонента выполнения пакетов служб SSIS|Нет|
|ссиспаккажеексекутиондатастатистикс|Статистика данных ексеутион пакетов служб SSIS|Нет|
|TriggerRuns|Журнал запусков триггера|Нет|


## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.DataLakeAnalytics/accounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Журналы аудита|Нет|
|Requests|Журналы запросов|Нет|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Журналы аудита|Нет|
|Requests|Журналы запросов|Нет|


## <a name="microsoftdatashareaccounts"></a>Microsoft.DataShare/accounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|рецеиведшареснапшотс|Получено моментальных снимков общих ресурсов|Нет|
|сентшареснапшотс|Отправленные моментальные снимки общих ресурсов|Нет|
|Shares (Общие папки)|Shares (Общие папки)|Нет|
|шаресубскриптионс|Предоставление общего доступа к подпискам|Нет|


## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|мисклаудитлогс|Журналы аудита MariaDB|Нет|
|MySqlSlowLogs|Журналы сервера MariaDB|Нет|


## <a name="microsoftdbformysqlflexibleservers"></a>Microsoft.DBforMySQL/flexibleServers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|мисклаудитлогс|Журналы аудита MySQL|Нет|
|MySqlSlowLogs|Замедляют журналы MySQL|Нет|


## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|мисклаудитлогс|Журналы аудита MySQL|Нет|
|MySqlSlowLogs|Журналы сервера MySQL|Нет|


## <a name="microsoftdbforpostgresqlflexibleservers"></a>Microsoft.DBforPostgreSQL/flexibleServers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|PostgreSQLLogs|Журналы сервера PostgreSQL|Нет|


## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|PostgreSQLLogs|Журналы сервера PostgreSQL|Нет|
|QueryStoreRuntimeStatistics|Статистика времени выполнения хранилища запросов PostgreSQL|Нет|
|QueryStoreWaitStatistics|Статистика ожидания хранилища запросов PostgreSQL|Нет|


## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|PostgreSQLLogs|Журналы сервера PostgreSQL|Нет|


## <a name="microsoftdesktopvirtualizationapplicationgroups"></a>Microsoft. Десктопвиртуализатион/аппликатионграупс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Контрольная точка|Контрольная точка|Нет|
|Error|Error|Нет|
|Управление|Управление|Нет|


## <a name="microsoftdesktopvirtualizationhostpools"></a>Microsoft. Десктопвиртуализатион/хостпулс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|аженсеалсстатус|аженсеалсстатус|Нет|
|Контрольная точка|Контрольная точка|Нет|
|Подключение|Подключение|Нет|
|Error|Error|Нет|
|HostRegistration|HostRegistration|Нет|
|Управление|Управление|Нет|


## <a name="microsoftdesktopvirtualizationworkspaces"></a>Microsoft. Десктопвиртуализатион/рабочие области

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Контрольная точка|Контрольная точка|Нет|
|Error|Error|Нет|
|Веб-канал|Веб-канал|Нет|
|Управление|Управление|Нет|


## <a name="microsoftdeviceselasticpoolsiothubtenants"></a>Microsoft.Devices/ElasticPools/IotHubTenants

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|C2DCommands|Команды, отправляемые из облака на устройство|Нет|
|C2DTwinOperations|Операции переноса из облака на двойник устройства|Нет|
|Конфигурации|Конфигурации|Нет|
|Соединения|Соединения|Нет|
|Операции переноса с двойника устройства в облако|Операции переноса с двойника устройства в облако|Нет|
|DeviceIdentityOperations|Операции с удостоверениями устройства|Нет|
|девицестреамс|Потоки устройств (Предварительная версия)|Нет|
|DeviceTelemetry|Телеметрия устройства|Нет|
|DirectMethods|Прямые методы|Нет|
|DistributedTracing|Распределенная трассировка (предварительная версия)|Нет|
|FileUploadOperations|Операции отправки файлов|Нет|
|JobsOperations|Операции заданий|Нет|
|Маршруты|Маршруты|Нет|
|TwinQueries|Запросы к двойникам|Нет|


## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|C2DCommands|Команды, отправляемые из облака на устройство|Нет|
|C2DTwinOperations|Операции переноса из облака на двойник устройства|Нет|
|Конфигурации|Конфигурации|Нет|
|Соединения|Соединения|Нет|
|Операции переноса с двойника устройства в облако|Операции переноса с двойника устройства в облако|Нет|
|DeviceIdentityOperations|Операции с удостоверениями устройства|Нет|
|девицестреамс|Потоки устройств (Предварительная версия)|Нет|
|DeviceTelemetry|Телеметрия устройства|Нет|
|DirectMethods|Прямые методы|Нет|
|DistributedTracing|Распределенная трассировка (предварительная версия)|Нет|
|FileUploadOperations|Операции отправки файлов|Нет|
|JobsOperations|Операции заданий|Нет|
|Маршруты|Маршруты|Нет|
|TwinQueries|Запросы к двойникам|Нет|


## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|DeviceOperations|Операции с устройствами|Нет|
|ServiceOperations|Операции служб|Нет|


## <a name="microsoftdigitaltwinsdigitaltwinsinstances"></a>Microsoft. Дигиталтвинс/Дигиталтвинсинстанцес

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|дигиталтвинсоператион|дигиталтвинсоператион|Нет|
|евентраутесоператион|евентраутесоператион|Нет|
|моделсоператион|моделсоператион|Нет|
|Queryoperation класса DataService|Queryoperation класса DataService|Нет|


## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|CassandraRequests|CassandraRequests|Нет|
|контролпланерекуестс|контролпланерекуестс|Нет|
|DataPlaneRequests|DataPlaneRequests|Нет|
|гремлинрекуестс|гремлинрекуестс|Нет|
|MongoRequests|MongoRequests|Нет|
|PartitionKeyRUConsumption|PartitionKeyRUConsumption|Нет|
|PartitionKeyStatistics|PartitionKeyStatistics|Нет|
|QueryRuntimeStatistics|QueryRuntimeStatistics|Нет|


## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|деливерифаилурес|Журналы сбоев доставки|Нет|
|публишфаилурес|Публикация журналов сбоев|Нет|


## <a name="microsofteventgridpartnernamespaces"></a>Microsoft. EventGrid/Партнернамеспацес

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|деливерифаилурес|Журналы сбоев доставки|Нет|
|публишфаилурес|Публикация журналов сбоев|Нет|


## <a name="microsofteventgridpartnertopics"></a>Microsoft. EventGrid/Партнертопикс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|деливерифаилурес|Журналы сбоев доставки|Нет|


## <a name="microsofteventgridsystemtopics"></a>Microsoft. EventGrid/Системтопикс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|деливерифаилурес|Журналы сбоев доставки|Нет|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|деливерифаилурес|Журналы сбоев доставки|Нет|
|публишфаилурес|Публикация журналов сбоев|Нет|


## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ArchiveLogs|Журналы архивации|Нет|
|AutoScaleLogs|Журналы автомасштабирования.|Нет|
|кустомерманажедкэйусерлогс|Журналы управляемых ключей клиентов|Нет|
|евенсубвнетконнектионевент|Журналы подключений для фильтрации виртуальной сети и IP-адресов|Нет|
|кафкакурдинаторлогс|Журналы координатора Kafka|Нет|
|кафкаусереррорлогс|Журналы ошибок пользователя Kafka|Нет|
|OperationalLogs|Журналы операций|Нет|


## <a name="microsoftexperimentationexperimentworkspaces"></a>Microsoft. экспериментирование/Експериментворкспацес

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Запрос|Запрос|Нет|


## <a name="microsofthealthcareapisservices"></a>Microsoft.HealthcareApis/services

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AuditLogs|Журналы аудита|Нет|


## <a name="microsoftinsightsautoscalesettings"></a>microsoft.insights/autoscalesettings

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AutoscaleEvaluations|Оценки автомасштабирования|Нет|
|AutoscaleScaleActions|Действия автоматического увеличения масштаба|Нет|


## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AppAvailabilityResults|Результаты доступности|Нет|
|AppBrowserTimings|Время браузера|Нет|
|AppDependencies|Зависимости|Нет|
|AppEvents|События|нет|
|AppExceptions|Исключения|Нет|
|AppMetrics|Метрики|Нет|
|AppPageViews|Просмотры страниц|Нет|
|AppPerformanceCounters|Счетчики производительности|Нет|
|AppRequests|Requests|Нет|
|AppSystemEvents|Системные события|Нет|
|AppTraces|Трассировки|Нет|


## <a name="microsoftiotspacesgraph"></a>Microsoft.IoTSpaces/Graph

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Аудит|Нет|
|Исходящие|Исходящие|Нет|
|Входящий трафик|Входящий трафик|Нет|
|Операционный|Операционный|Нет|
|Трассировка|Трассировка|Нет|
|UserDefinedFunction|UserDefinedFunction|Нет|


## <a name="microsoftkeyvaultmanagedhsms"></a>Microsoft. keyvault/манажедхсмс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AuditEvent|Событие аудита|Нет|


## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AuditEvent|Журналы аудита|Нет|


## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Команда|Команда|Нет|
|FailedIngestion|Неудачные операции приема|Нет|
|инжестионбатчинг|Пакетная обработка приема|Нет|
|Запрос|Запрос|Нет|
|SucceededIngestion|Успешные операции приема|Нет|
|табледетаилс|Сведения о таблице|Нет|
|таблеусажестатистикс|Статистика использования таблиц|Нет|


## <a name="microsoftlogicintegrationaccounts"></a>Microsoft.Logic/integrationAccounts

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|IntegrationAccountTrackingEvents|Integration Account track events|Нет|


## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|WorkflowRuntime|События диагностики среды выполнения рабочего процесса|Нет|


## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|амлкомпутеклустеревент|амлкомпутеклустеревент|Нет|
|амлкомпутеклустернодивент|амлкомпутеклустернодивент|Нет|
|AmlComputeCpuGpuUtilization|AmlComputeCpuGpuUtilization|Нет|
|амлкомпутежобевент|амлкомпутежобевент|Нет|
|AmlRunStatusChangedEvent|AmlRunStatusChangedEvent|Нет|


## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|кэйделиверирекуестс|Запросы на доставку ключей|Нет|


## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ApplicationGatewayAccessLog|Журнал доступа к шлюзу приложений|Нет|
|ApplicationGatewayFirewallLog|Журнал брандмауэра шлюза приложений|Нет|
|ApplicationGatewayPerformanceLog|Журнал производительности шлюза приложений|Нет|


## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AzureFirewallApplicationRule|Правило приложения службы "Брандмауэр Azure"|Нет|
|азурефиреваллднспрокси|DNS-прокси Azure Firewall|Нет|
|AzureFirewallNetworkRule|Правило сети службы "Брандмауэр Azure"|Нет|


## <a name="microsoftnetworkbastionhosts"></a>Microsoft.Network/bastionHosts;

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|бастионаудитлогс|Журналы аудита бастиона|Нет|


## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|PeeringRouteLog|Журналы таблиц пиринга маршрутов|Нет|


## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|FrontdoorAccessLog|Журнал доступа Frontdoor|Нет|
|FrontdoorWebApplicationFirewallLog|Журнал брандмауэра веб-приложения Frontdoor|Нет|


## <a name="microsoftnetworkloadbalancers"></a>Microsoft.Network/loadBalancers

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|LoadBalancerAlertEvent|События оповещения балансировщика нагрузки|Нет|
|LoadBalancerProbeHealthStatus|Состояние работоспособности балансировщика нагрузки|Нет|


## <a name="microsoftnetworknetworksecuritygroups"></a>Microsoft.Network/networksecuritygroups

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|NetworkSecurityGroupEvent|Событие группы безопасности сети|Нет|
|NetworkSecurityGroupFlowEvent|Событие потока правил группы безопасности сети|Нет|
|NetworkSecurityGroupRuleCounter|Счетчик правил группы безопасности сети|Нет|


## <a name="microsoftnetworkp2svpngateways"></a>Microsoft. Network/p2sVpnGateways

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|GatewayDiagnosticLog|Журналы диагностики шлюзов|Нет|
|IKEDiagnosticLog|Журналы диагностики IKE|Нет|
|P2SDiagnosticLog|Журналы диагностики P2S|Нет|


## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|DDoSMitigationFlowLogs|Журналы потоков принятия решений по защите от атак DDoS|Нет|
|DDoSMitigationReports|Отчеты о защите от атак DDoS|Нет|
|DDoSProtectionNotifications|Уведомления о защите от атак DDoS|Нет|


## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ProbeHealthStatusEvents|Событие вывода результатов проверки работоспособности диспетчера трафика|Нет|


## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|GatewayDiagnosticLog|Журналы диагностики шлюзов|Нет|
|IKEDiagnosticLog|Журналы диагностики IKE|Нет|
|P2SDiagnosticLog|Журналы диагностики P2S|Нет|
|RouteDiagnosticLog|Журналы диагностики маршрутов|Нет|
|TunnelDiagnosticLog|Журналы диагностики туннелей|Нет|


## <a name="microsoftnetworkvirtualnetworks"></a>Microsoft.Network/virtualNetworks

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|VMProtectionAlerts|Оповещения о защите виртуальной машины|Нет|


## <a name="microsoftnetworkvpngateways"></a>Microsoft. Network/Впнгатевайс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|GatewayDiagnosticLog|Журналы диагностики шлюзов|Нет|
|IKEDiagnosticLog|Журналы диагностики IKE|Нет|
|RouteDiagnosticLog|Журналы диагностики маршрутов|Нет|
|TunnelDiagnosticLog|Журналы диагностики туннелей|Нет|


## <a name="microsoftnotificationhubsnamespaces"></a>Справочник по шаблонам пространства имен Microsoft.NotificationHubs.

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|OperationalLogs|Журналы операций|Нет|


## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.OperationalInsights/workspaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Аудит|Журналы аудита|Нет|


## <a name="microsoftpowerbitenants"></a>Microsoft. PowerBI/клиенты

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Подсистема|Подсистема|Нет|


## <a name="microsoftpowerbitenantsworkspaces"></a>Microsoft. PowerBI/клиенты/рабочие области

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Подсистема|Подсистема|Нет|


## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Подсистема|Подсистема|Нет|


## <a name="microsoftprojectbabylonaccounts"></a>Microsoft. Прожектбабилон/учетные записи

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|сканстатусложевент|сканстатус|Нет|


## <a name="microsoftpurviewaccounts"></a>Microsoft. зрения/учетные записи

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|сканстатусложевент|сканстатус|Нет|


## <a name="microsoftrecoveryservicesvaults"></a>Microsoft.RecoveryServices/Vaults

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AddonAzureBackupAlerts|Надстройка Azure Backup данные оповещений|Нет|
|AddonAzureBackupJobs|Azure Backup данных задания надстройки|Нет|
|аддоназуребаккупполици|Данные политики Azure Backup надстроек|Нет|
|AddonAzureBackupProtectedInstance|Надстройка Azure Backup данные защищенного экземпляра|Нет|
|AddonAzureBackupStorage|Надстройка Azure Backup данные хранилища|Нет|
|AzureBackupReport|Данные отчетов службы архивации Azure|Нет|
|AzureSiteRecoveryEvents|События Azure Site Recovery|Нет|
|AzureSiteRecoveryJobs|Задания Azure Site Recovery|Нет|
|AzureSiteRecoveryProtectedDiskDataChurn|Обновление данных защищенного диска Azure Site Recovery|Нет|
|AzureSiteRecoveryRecoveryPoints|Точки восстановления Azure Site Recovery|Нет|
|AzureSiteRecoveryReplicatedItems|Реплицированные элементы Azure Site Recovery|Нет|
|AzureSiteRecoveryReplicationDataUploadRate|Периодичность отправки данных репликации Azure Site Recovery|Нет|
|AzureSiteRecoveryReplicationStats|Статистика репликации Azure Site Recovery|Нет|
|кореазуребаккуп|Основные Azure Backup данные|Нет|


## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|хибридконнектионсевент|События Хибридконнектионс|Нет|
|хибридконнектионслогс|хибридконнектионслогс|Нет|


## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|OperationLogs|Журналы операций|Нет|


## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|OperationalLogs|Журналы операций|Нет|


## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|алллогс|Журналы службы SignalR Azure.|Нет|


## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|девопсоператионсаудит|Журналы аудита операций Devops|Нет|
|ResourceUsageStats|Статистика использования ресурсов|Нет|
|SQLSecurityAuditEvents|Событие аудита безопасности SQL|Нет|


## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.Sql/managedInstances/databases

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|ошибки|ошибки|Нет|
|QueryStoreRuntimeStatistics|Статистика среды выполнения хранилища запросов|Нет|
|QueryStoreWaitStatistics|Статистика времени ожидания хранилища запросов|Нет|
|SQLInsights|Аналитика SQL|Нет|


## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AutomaticTuning|Автоматическая настройка|Нет|
|Blocks|Blocks|Нет|
|DatabaseWaitStatistics|Статистика времени ожидания базы данных|Нет|
|Взаимоблокировки|Взаимоблокировки|Нет|
|девопсоператионсаудит|Журналы аудита операций Devops|Нет|
|DmsWorkers|Рабочие роли DMS|Нет|
|ошибки|ошибки|Нет|
|ExecRequests|Выполняющиеся запросы|Нет|
|QueryStoreRuntimeStatistics|Статистика среды выполнения хранилища запросов|Нет|
|QueryStoreWaitStatistics|Статистика времени ожидания хранилища запросов|Нет|
|RequestSteps|Действия, из которых состоит запрос|Нет|
|SQLInsights|Аналитика SQL|Нет|
|SqlRequests|Запросы SQL|Нет|
|SQLSecurityAuditEvents|Событие аудита безопасности SQL|Нет|
|Время ожидания|Время ожидания|Нет|
|Ожидания|Ожидания|Нет|


## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|StorageDelete|StorageDelete|Да|
|StorageRead|StorageRead|Да|
|StorageWrite|StorageWrite|Да|


## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|StorageDelete|StorageDelete|Да|
|StorageRead|StorageRead|Да|
|StorageWrite|StorageWrite|Да|


## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|StorageDelete|StorageDelete|Да|
|StorageRead|StorageRead|Да|
|StorageWrite|StorageWrite|Да|


## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|StorageDelete|StorageDelete|Да|
|StorageRead|StorageRead|Да|
|StorageWrite|StorageWrite|Да|


## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Разработка|Разработка|Нет|
|Выполнение|Выполнение|Нет|


## <a name="microsoftsynapseworkspaces"></a>Microsoft.Synapse/workspaces

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|буилтинсклрексендед|Завершено запросов встроенного пула SQL|Нет|
|гатевайапирекуестс|Запросы API шлюза синапсе|Нет|
|SQLSecurityAuditEvents|Событие аудита безопасности SQL|Нет|
|SynapseRbacOperations|Операции синапсе RBAC|Нет|


## <a name="microsoftsynapseworkspacesbigdatapools"></a>Microsoft. синапсе/workspaces/Бигдатапулс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|бигдатапулаппсендед|Приложения пула больших данных завершены|Нет|


## <a name="microsoftsynapseworkspacessqlpools"></a>Microsoft. синапсе/workspaces/Склпулс

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|DmsWorkers|Рабочие роли DMS|Нет|
|ExecRequests|Выполняющиеся запросы|Нет|
|RequestSteps|Действия, из которых состоит запрос|Нет|
|SqlRequests|Запросы SQL|Нет|
|SQLSecurityAuditEvents|Событие аудита безопасности SQL|Нет|
|Ожидания|Ожидания|Нет|


## <a name="microsofttimeseriesinsightsenvironments"></a>Microsoft.TimeSeriesInsights/environments

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Входящий трафик|Входящий трафик|Нет|
|Управление|Управление|Нет|


## <a name="microsofttimeseriesinsightsenvironmentseventsources"></a>Microsoft.TimeSeriesInsights/environments/eventsources

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|Входящий трафик|Входящий трафик|Нет|
|Управление|Управление|Нет|


## <a name="microsoftwebhostingenvironments"></a>microsoft.web/hostingenvironments

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AppServiceEnvironmentPlatformLogs|Журналы платформы Среда службы приложений|Нет|


## <a name="microsoftwebsites"></a>microsoft.web/sites

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AppServiceAntivirusScanAuditLogs|Сообщить о журналах антивирусного аудита|Нет|
|AppServiceAppLogs|Журналы приложения службы приложений|Нет|
|AppServiceAuditLogs|Доступ к журналам аудита|Нет|
|AppServiceConsoleLogs|Журналы консоли службы приложений|Нет|
|AppServiceFileAuditLogs|Журналы аудита изменения содержимого сайта|Нет|
|AppServiceHTTPLogs|Журналы HTTP|Нет|
|AppServiceIPSecAuditLogs|Журналы аудита IPSecurity|Нет|
|AppServicePlatformLogs|Журналы платформы службы приложений|Нет|
|FunctionAppLogs|Журналы приложений функций|Нет|


## <a name="microsoftwebsitesslots"></a>microsoft.web/sites/slots

|Категория|Отображаемое имя категории|Затраты на экспорт|
|---|---|---|
|AppServiceAntivirusScanAuditLogs|Сообщить о журналах антивирусного аудита|Нет|
|AppServiceAppLogs|Журналы приложения службы приложений|Нет|
|AppServiceAuditLogs|Доступ к журналам аудита|Нет|
|AppServiceConsoleLogs|Журналы консоли службы приложений|Нет|
|AppServiceFileAuditLogs|Журналы аудита изменения содержимого сайта|Нет|
|AppServiceHTTPLogs|Журналы HTTP|Нет|
|AppServiceIPSecAuditLogs|Журналы аудита IPSecurity|Нет|
|AppServicePlatformLogs|Журналы платформы службы приложений|Нет|
|FunctionAppLogs|Журналы приложений функций|Нет|



## <a name="next-steps"></a>Next Steps

* [Дополнительные сведения о журналах ресурсов](../essentials/platform-logs-overview.md)
* [Потоковая передача журналов ресурсов в **концентраторы событий**](./resource-logs.md#send-to-azure-event-hubs)
* [Изменение параметров диагностики журнала ресурсов с помощью Azure Monitor REST API](/rest/api/monitor/diagnosticsettings)
* [Сбор журналов и метрик для служб Azure для использования в Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)

