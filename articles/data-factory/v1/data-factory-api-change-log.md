---
title: Фабрика данных — журнал изменений API .NET
description: Описание критических изменений, дополнений к компонентам, исправлений ошибок и т. д. в конкретной версии API .NET для фабрики данных Azure.
author: dcstwh
ms.author: weetok
ms.reviewer: jonburchel
ms.service: data-factory
ms.topic: conceptual
robots: noindex
ms.date: 01/22/2018
ms.openlocfilehash: 2d292bbc820574d4360be51e93588f8f908f468d
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104785603"
---
# <a name="azure-data-factory---net-api-change-log"></a>Фабрика данных Azure — журнал изменений в .NET API
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. 

В этой статье содержится информация об изменениях в определенной версии пакета SDK для фабрики данных Azure. Последнюю версию пакета NuGet для фабрики данных Azure можно найти [здесь](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactories)

## <a name="version-4110"></a>Версия 4.11.0
Добавление функций.

* Добавлены следующие типы связанных служб:
  * [OnPremisesMongoDbLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisesmongodblinkedservice)
  * [AmazonRedshiftLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.amazonredshiftlinkedservice)
  * [AwsAccessKeyLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.awsaccesskeylinkedservice)
* Добавлены следующие типы наборов данных:
  * [MongoDbCollectionDataset](/dotnet/api/microsoft.azure.management.datafactories.models.mongodbcollectiondataset)
  * [AmazonS3Dataset](/dotnet/api/microsoft.azure.management.datafactories.models.amazons3dataset)
* Добавлены следующие типы источников копий:
  * [MongoDbSource](/dotnet/api/microsoft.azure.management.datafactories.models.mongodbsource)

## <a name="version-4100"></a>Версия 4.10.0
* В раздел TextFormat добавлены следующие необязательные свойства:
  * [SkipLineCount](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
  * [FirstRowAsHeader](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
  * [TreatEmptyAsNull](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
* Добавлены следующие типы связанных служб:
  * [OnPremisesCassandraLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisescassandralinkedservice)
  * [SalesforceLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.salesforcelinkedservice)
* Добавлены следующие типы наборов данных:
  * [OnPremisesCassandraTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisescassandratabledataset)
* Добавлены следующие типы источников копий:
  * [CassandraSource](/dotnet/api/microsoft.azure.management.datafactories.models.cassandrasource)
* Свойство [WebServiceInputs](/dotnet/api/microsoft.azure.management.datafactories.models.azuremlbatchexecutionactivity) добавлено в действие AzureMLBatchExecutionActivity.
  * Активирована многократная передача входных данных веб-службы в эксперимент машинного обучения Azure.

## <a name="version-491"></a>Версия 4.9.1
### <a name="bug-fix"></a>Исправление ошибок
* Отменена проверка подлинности на основе веб-API для [WebLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.weblinkedservice).

## <a name="version-490"></a>Версия 4.9.0
### <a name="feature-additions"></a>Добавление функций
* Свойства [EnableStaging](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity) и [StagingSettings](/dotnet/api/microsoft.azure.management.datafactories.models.stagingsettings) добавлены в действие CopyActivity. Сведения об этой функции см. в разделе [Промежуточное копирование](data-factory-copy-activity-performance.md#staged-copy).

### <a name="bug-fix"></a>Исправление ошибок
* Добавлена перегрузка метода [ActivityWindowOperationExtensions.List](/dotnet/api/microsoft.azure.management.datafactories.activitywindowoperationsextensions), который принимает экземпляр [ActivityWindowsByActivityListParameters](/dotnet/api/microsoft.azure.management.datafactories.models.activitywindowsbyactivitylistparameters).
* Свойства [WriteBatchSize](/dotnet/api/microsoft.azure.management.datafactories.models.copysink) и [WriteBatchTimeout](/dotnet/api/microsoft.azure.management.datafactories.models.copysink) отмечены как необязательные в CopySink.

## <a name="version-480"></a>Версия 4.8.0
### <a name="feature-additions"></a>Добавление функций
* В действие копирования добавлены следующие дополнительные свойства, которые предназначены для настройки производительности копирования:
  * [ParallelCopies](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity)
  * [CloudDataMovementUnits](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity)

## <a name="version-470"></a>Версия 4.7.0
### <a name="feature-additions"></a>Добавление функций
* Добавлен новый тип StorageFormat — [OrcFormat](/dotnet/api/microsoft.azure.management.datafactories.models.orcformat) — для копирования файлов в формате столбцов с оптимизацией по строкам (ORC).
* Свойства [AllowPolyBase](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsink) и PolyBaseSettings добавлены в SqlDWSink.
  * Позволяет использовать Polybase для копирования данных в Azure синапсе Analytics.

## <a name="version-461"></a>Версия 4.6.1
### <a name="bug-fixes"></a>Исправления ошибок
* Исправляет HTTP-запрос для получения списка действий Windows.
  * Удаляет имя группы ресурсов и имя фабрики данных из полезных данных запроса.

## <a name="version-460"></a>Версия 4.6.0
### <a name="feature-additions"></a>Добавление функций
* В раздел [PipelineProperties](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)добавлены следующие свойства:
  * [PipelineMode](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
  * [ExpirationTime](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
  * [Наборы данных](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
* В раздел [PipelineRuntimeInfo](/dotnet/api/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo)добавлены следующие свойства:
  * [PipelineState](/dotnet/api/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo)
* Добавлен новый тип [StorageFormat](/dotnet/api/microsoft.azure.management.datafactories.models.storageformat) — [JsonFormat](/dotnet/api/microsoft.azure.management.datafactories.models.jsonformat) — для определения наборов данных, данные которых представлены в формате JSON.

## <a name="version-450"></a>Версия 4.5.0
### <a name="feature-additions"></a>Добавление функций
* Добавлен [список операций для окна действий](/dotnet/api/microsoft.azure.management.datafactories.activitywindowoperationsextensions).
  * Добавлены методы для получения окон действий с фильтрам на основе типов сущностей (т. е. фабрик данных, наборов данных, конвейеров и действий).
* Добавлены следующие типы связанных служб:
  * [ODataLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.odatalinkedservice), [WebLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.weblinkedservice).
* Добавлены следующие типы наборов данных:
  * [ODataResourceDataset](/dotnet/api/microsoft.azure.management.datafactories.models.odataresourcedataset), [WebTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.webtabledataset).
* Добавлены следующие типы источников копий:     
  * [WebSource](/dotnet/api/microsoft.azure.management.datafactories.models.websource)

## <a name="version-440"></a>Версия 4.4.0
### <a name="feature-additions"></a>Добавление функций
* Для действий копирования в качестве источника и приемника данных добавлен следующий тип связанной службы:
  * [AzureStorageSasLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azurestoragesaslinkedservice). Основные сведения и примеры см. в разделе [Связанная служба SAS хранилища Azure](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service).

## <a name="version-430"></a>Версия 4.3.0
### <a name="feature-additions"></a>Добавление функций
* Для действий копирования в качестве источника данных добавлен следующий тип связанной службы:
  * [HdfsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.hdfslinkedservice). Основные сведения и примеры см. в статье [Перемещение данных из локальной системы HDFS с помощью фабрики данных Azure](data-factory-hdfs-connector.md).
  * [OnPremisesOdbcLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisesodbclinkedservice). Основные сведения и примеры см. в статье [Перемещение данных из хранилищ данных ODBC с помощью фабрики данных Azure](data-factory-odbc-connector.md).

## <a name="version-420"></a>Версия 4.2.0
### <a name="feature-additions"></a>Добавление функций
* Был добавлен следующий новый тип действия: [AzureMLUpdateResourceActivity](/dotnet/api/microsoft.azure.management.datafactories.models.azuremlupdateresourceactivity). Подробные сведения об этом действии см. в разделе [Обновление моделей машинного обучения Azure с помощью действия "Обновить ресурс"](data-factory-azure-ml-batch-execution-activity.md).
* Новое необязательное свойство [updateResourceEndpoint](/dotnet/api/microsoft.azure.management.datafactories.models.azuremllinkedservice) добавлено в [класс AzureMLLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuremllinkedservice).
* Свойства [LongRunningOperationInitialTimeout](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient) и [LongRunningOperationRetryTimeout](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient) добавлены в класс [DataFactoryManagementClient](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient).
* Разрешена настройка времени ожидания для клиентских вызовов в службу фабрики данных.

## <a name="version-410"></a>Версия 4.1.0
### <a name="feature-additions"></a>Добавление функций
* Добавлены следующие типы связанных служб:
  * [AzureDataLakeStoreLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice)
  * [AzureDataLakeAnalyticsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice)
* Добавлены следующие типы действий:
  * [DataLakeAnalyticsUSQLActivity](/dotnet/api/microsoft.azure.management.datafactories.models.datalakeanalyticsusqlactivity)
* Добавлены следующие типы наборов данных:
  * [AzureDataLakeStoreDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoredataset)
* Добавлены следующие типы источника и приемника для копирования:
  * [AzureDataLakeStoreSource](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoresource)
  * [AzureDataLakeStoreSink](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoresink)

## <a name="version-401"></a>Версия 4.0.1
### <a name="breaking-changes"></a>Критические изменения
Следующие классы были переименованы. Новые имена — это первоначальные имена классов до выхода версии 4.0.0.

| Имя в выпуске 4.0.0 | Имя в выпуске 4.0.1 |
|:--- |:--- |
| AzureSqlDataWarehouseDataset |[AzureSqlDataWarehouseTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuresqldatawarehousetabledataset) |
| AzureSqlDataset |[AzureSqlTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuresqltabledataset) |
| AzureDataset |[AzureTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuretabledataset) |
| OracleDataset |[OracleTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.oracletabledataset) |
| RelationalDataset |[RelationalTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.relationaltabledataset) |
| SqlServerDataset |[SqlServerTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.sqlservertabledataset) |

## <a name="version-400"></a>Версия 4.0.0
### <a name="breaking-changes"></a>Критические изменения
* Следующие классы и интерфейсы были переименованы.

| Старое имя | Новое имя |
|:--- |:--- |
| ITableOperations |[IDatasetOperations](/dotnet/api/microsoft.azure.management.datafactories.idatasetoperations) |
| Таблица |[Набор данных](/dotnet/api/microsoft.azure.management.datafactories.models.dataset) |
| TableProperties |[DatasetProperties](/dotnet/api/microsoft.azure.management.datafactories.models.datasetproperties) |
| TableTypeProprerties |[DatasetTypeProperties](/dotnet/api/microsoft.azure.management.datafactories.models.datasettypeproperties) |
| TableCreateOrUpdateParameters |[DatasetCreateOrUpdateParameters](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdateparameters) |
| TableCreateOrUpdateResponse |[DatasetCreateOrUpdateResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdateresponse) |
| TableGetResponse |[DatasetGetResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetgetresponse) |
| TableListResponse |[DatasetListResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetlistresponse) |
| CreateOrUpdateWithRawJsonContentParameters |[DatasetCreateOrUpdateWithRawJsonContentParameters](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdatewithrawjsoncontentparameters) |

* Методы **List** теперь возвращают результаты с разбивкой на страницы. Если ответ содержит непустое свойство **NextLink** , клиентскому приложению необходимо продолжить извлечение следующей страницы до тех пор, пока не будут возвращены все страницы.  Пример:

    ```csharp
    PipelineListResponse response = client.Pipelines.List("ResourceGroupName", "DataFactoryName");
    var pipelines = new List<Pipeline>(response.Pipelines);

    string nextLink = response.NextLink;
    while (!string.IsNullOrEmpty(nextLink))
    {
        PipelineListResponse nextResponse = client.Pipelines.ListNext(nextLink);
        pipelines.AddRange(nextResponse.Pipelines);

        nextLink = nextResponse.NextLink;
    }
    ```
* **List** возвращает только сводную информацию о конвейере, а не полную информацию. Например, действия в сводной информации о конвейере содержат только имя и тип.

### <a name="feature-additions"></a>Добавление функций
* Класс [SqlDWSink](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsink) поддерживает два новых свойства, **SliceIdentifierColumnName** и **SqlWriterCleanupScript**, для поддержки идемпотентными копирования в Azure Azure синапсе Analytics. Дополнительные сведения об этих свойствах см. в статье [Azure синапсе Analytics](data-factory-azure-sql-data-warehouse-connector.md) .
* Теперь мы поддерживаем выполнение хранимой процедуры в базе данных SQL Azure и источниках аналитики Azure синапсе в рамках действия копирования. Классы [SqlSource](/dotnet/api/microsoft.azure.management.datafactories.models.sqlsource) и [SqlDWSource](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsource) имеют следующие свойства: **SqlReaderStoredProcedureName** и **StoredProcedureParameters**. Дополнительные сведения об этих свойствах см. в статьях [база данных SQL Azure](data-factory-azure-sql-connector.md#sqlsource) и [Azure синапсе Analytics](data-factory-azure-sql-data-warehouse-connector.md#sqldwsource) в Azure.com.