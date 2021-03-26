---
title: Копирование данных в пакетном режиме с помощью PowerShell
description: Использование Фабрики данных Azure c действием копирования для копирования данных из исходного хранилища данных в целевое хранилище в пакетном режиме.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 02/18/2021
ms.openlocfilehash: fc539ababf4cb240fbe78de0d87b1f127807f604
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101740440"
---
# <a name="copy-multiple-tables-in-bulk-by-using-azure-data-factory-using-powershell"></a>Копирование нескольких таблиц в пакетном режиме с помощью Фабрики данных Azure и PowerShell

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этом руководстве показано, как **скопировать несколько таблиц из Базы данных SQL Azure в Azure Synapse Analytics**. Этот подход можно применить и в других сценариях. Например, копирование таблиц из SQL Server или Oracle в Базу данных SQL Azure, хранилище данных или большой двоичный объект Azure, копирование различных путей из большого двоичного объекта в таблицы Базы данных SQL Azure.

В целом это руководство включает следующие шаги:

> [!div class="checklist"]
> * Создали фабрику данных.
> * Создание экземпляров Базы данных SQL Azure, Azure Synapse Analytics и связанных служб хранилища Azure.
> * Создание Базы данных SQL Azure и наборов данных Azure Synapse Analytics.
> * Создание конвейера для поиска таблиц, которые нужно скопировать, и конвейера для выполнения операции копирования. 
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

В этом руководстве используется Azure PowerShell. Сведения об использовании других средств или пакетов SDK для создания фабрики данных см. в [этом кратком руководстве](quickstart-create-data-factory-dot-net.md). 

## <a name="end-to-end-workflow"></a>Комплексный рабочий процесс
В этом сценарии есть несколько таблиц базы данных SQL Azure, которые необходимо скопировать в Azure Synapse Analytics. Вот логическая последовательность действий рабочего процесса в конвейерах:

![Рабочий процесс](media/tutorial-bulk-copy/tutorial-copy-multiple-tables.png)

* Первый конвейер ищет список таблиц, который необходимо скопировать в хранилище данных-приемник.  В качестве альтернативы можно создать таблицу метаданных, в которой перечислены все таблицы, которые нужно скопировать в хранилище данных-приемник. Затем конвейер активирует другой конвейер, который выполняет итерацию по каждой таблице базы данных и выполняет операцию копирования данных.
* Второй конвейер выполняет фактическое копирование. Он принимает список таблиц в качестве параметра. Чтобы добиться лучшей производительности, для каждой таблицы в списке скопируйте определенную таблицу из Базы данных SQL Azure в соответствующую таблицу Azure Synapse Analytics, используя [промежуточное копирование с помощью хранилища BLOB-объектов и PolyBase](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-synapse-analytics). В этом примере первый конвейер передает список таблиц в качестве значения параметра. 

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Azure PowerShell**. Следуйте инструкциям по [установке и настройке Azure PowerShell](/powershell/azure/install-Az-ps).
* **Учетная запись хранения Azure.** Учетная запись хранения Azure используется в качестве промежуточного хранилища больших двоичных объектов в операции массового копирования. 
* **База данных SQL Azure**. Эта база данных содержит исходные данные. 
* **Azure Synapse Analytics**. Это хранилище данных содержит данные, копируемые из базы данных SQL. 

### <a name="prepare-sql-database-and-azure-synapse-analytics"></a>Подготовка Базы данных SQL Azure и Azure Synapse Analytics

**Подготовка исходной базы данных SQL Azure:**

Создайте базу данных в службе "База данных SQL", используя пример данных Adventure Works LT, представленный в статье [Создание базы данных в службе "База данных SQL Azure"](../azure-sql/database/single-database-create-quickstart.md). В этом учебнике все таблицы из этого примера базы данных копируются в Azure Synapse Analytics.

**Подготовка приемника в Azure Synapse Analytics**.

1. Если у вас нет рабочей области Azure Synapse Analytics, создайте ее по инструкциям из статьи [Начало работы с Azure Synapse Analytics](..\synapse-analytics\get-started.md).

2. Создание соответствующих схем таблиц в Azure Synapse Analytics. Фабрика данных Azure используется для миграции и копирования данных на более поздних этапах.

## <a name="azure-services-to-access-sql-server"></a>Доступ служб Azure к серверу SQL Server

В Базе данных SQL и Azure Synapse Analytics предоставьте службам Azure доступ к серверу SQL Server. Убедитесь, что параметр **Разрешить доступ к службам Azure** **включен** для вашего сервера. Этот параметр позволяет службе Фабрики данных читать данные из Базы данных SQL Azure и записывать их в Azure Synapse Analytics. Чтобы проверить и при необходимости включить этот параметр, сделайте следующее.

1. Щелкните **Все службы** слева и выберите **Серверы SQL**.
2. Выберите сервер и щелкните **Брандмауэр** в разделе **Параметры**.
3. На странице **Параметры брандмауэра** щелкните **ВКЛ** для параметра **Разрешить доступ к службам Azure**.

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. Запустите **PowerShell**. Не закрывайте Azure PowerShell, пока выполняются описанные в учебнике инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды.

    Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.
        
    ```powershell
    Connect-AzAccount
    ```
    Чтобы просмотреть все подписки для этой учетной записи, выполните следующую команду:

    ```powershell
    Get-AzSubscription
    ```
    Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать. Замените значение **SubscriptionId** на идентификатор подписки Azure:

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"
    ```
2. Выполните командлет **Set-AzDataFactoryV2**, чтобы создать фабрику данных. Перед выполнением команды замените заполнители собственными значениями. 

    ```powershell
    $resourceGroupName = "<your resource group to create the factory>"
    $dataFactoryName = "<specify the name of data factory to create. It must be globally unique.>"
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
    ```

    Обратите внимание на следующие моменты.

    * Имя фабрики данных Azure должно быть глобально уникальным. Если появляется следующая ошибка, измените имя и повторите попытку.

        ```
        The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
        ```

    * Чтобы создавать экземпляры фабрики данных, вы должны быть участником или администратором подписки Azure.
    * Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.

## <a name="create-linked-services"></a>Создание связанных служб

В этом руководстве создаются три связанные службы для источника, приемника и промежуточного большого двоичного объекта соответственно, включая подключения к хранилищам данных:

### <a name="create-the-source-azure-sql-database-linked-service"></a>Создание исходной связанной службы Базы данных SQL Azure

1. Создайте файл JSON с именем **AzureSqlDatabaseLinkedService.json** в папке **C:\ADFv2TutorialBulkCopy** с приведенным ниже содержимым. (Создайте папку ADFv2TutorialBulkCopy, если она еще не существует.)

    > [!IMPORTANT]
    > Перед сохранением файла замените &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; и &lt;password&gt; значениями своей базы данных SQL Azure.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. В **Azure PowerShell** переключитесь в папку **ADFv2TutorialBulkCopy**.

3. Выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу. **AzureSqlDatabaseLinkedService**. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureSqlDatabaseLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

### <a name="create-the-sink-azure-synapse-analytics-linked-service"></a>Создание приемника связанной службы Azure Synapse Analytics

1. Создайте файл JSON с именем **AzureSqlDWLinkedService.json** в папке **C:\ADFv2TutorialBulkCopy** со следующим содержимым:

    > [!IMPORTANT]
    > Перед сохранением файла замените &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; и &lt;password&gt; значениями своей базы данных SQL Azure.

    ```json
    {
        "name": "AzureSqlDWLinkedService",
        "properties": {
            "type": "AzureSqlDW",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. Чтобы создать связанную службу **AzureSqlDWLinkedService**, выполните командлет **Set-AzDataFactoryV2LinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWLinkedService" -File ".\AzureSqlDWLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureSqlDWLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDWLinkedService
    ```

### <a name="create-the-staging-azure-storage-linked-service"></a>Создание промежуточной связанной службы хранилища Azure

В этом руководстве хранилища BLOB-объектов Azure используются в качестве области промежуточного хранения, чтобы включить PolyBase для повышения производительности копирования.

1. Создайте файл JSON с именем **AzureStorageLinkedService.json** в папке **C:\ADFv2TutorialBulkCopy** со следующим содержимым:

    > [!IMPORTANT]
    > Перед сохранением файла замените значения &lt;accountname&gt; и &lt;accountkey&gt; на имя вашей учетной записи хранения Azure и ее ключ.

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
            }
        }
    }
    ```

2. Чтобы создать связанную службу **AzureStorageLinkedService**, выполните командлет **Set-AzDataFactoryV2LinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

## <a name="create-datasets"></a>Создание наборов данных

В этом руководстве создаются наборы данных источника и приемника, в которых указывается место хранения данных:

### <a name="create-a-dataset-for-source-sql-database"></a>Создание набора данных для исходной базы данных SQL

1. Создайте файл JSON с именем **AzureSqlDatabaseDataset.json** в папке **C:\ADFv2TutorialBulkCopy** со следующим содержимым. TableName является фиктивным,так как позже в действии копирования будет использован SQL-запрос для извлечения данных.

    ```json
    {
        "name": "AzureSqlDatabaseDataset",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDatabaseLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": "dummy"
            }
        }
    }
    ```

2. Чтобы создать набор данных **AzureSqlDatabaseDataset**, выполните командлет **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseDataset" -File ".\AzureSqlDatabaseDataset.json"
    ```

    Пример выходных данных:

    ```console
    DatasetName       : AzureSqlDatabaseDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-sink-azure-synapse-analytics"></a>Создание набора данных для приемника Azure Synapse Analytics

1. Создайте файл JSON с именем **AzureSqlDWDataset.json** в папке **C:\ADFv2TutorialBulkCopy** со приведенным ниже содержимым. tableName задается в качестве параметра, позже действие копирования, которое ссылается на этот набор данных, передает фактическое значение в набор данных.

    ```json
    {
        "name": "AzureSqlDWDataset",
        "properties": {
            "type": "AzureSqlDWTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDWLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": {
                    "value": "@{dataset().DWTableName}",
                    "type": "Expression"
                }
            },
            "parameters":{
                "DWTableName":{
                    "type":"String"
                }
            }
        }
    }
    ```

2. Чтобы создать набор данных **AzureSqlDWDataset**, выполните командлет **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWDataset" -File ".\AzureSqlDWDataset.json"
    ```

    Пример выходных данных:

    ```console
    DatasetName       : AzureSqlDWDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDwTableDataset
    ```

## <a name="create-pipelines"></a>Создание конвейеров

В этом руководстве создается два конвейера:

### <a name="create-the-pipeline-iterateandcopysqltables"></a>Создание конвейера IterateAndCopySQLTables

Этот конвейер принимает список таблиц в качестве параметра. Для каждой таблицы в списке он копирует данные из таблицы в Базе данных SQL Azure в Azure Synapse Analytics с помощью промежуточного копирования и PolyBase.

1. Создайте файл JSON с именем **IterateAndCopySQLTables.json** в папке **C:\ADFv2TutorialBulkCopy** со следующим содержимым:

    ```json
    {
        "name": "IterateAndCopySQLTables",
        "properties": {
            "activities": [
                {
                    "name": "IterateSQLTables",
                    "type": "ForEach",
                    "typeProperties": {
                        "isSequential": "false",
                        "items": {
                            "value": "@pipeline().parameters.tableList",
                            "type": "Expression"
                        },
                        "activities": [
                            {
                                "name": "CopyData",
                                "description": "Copy data from Azure SQL Database to Azure Synapse Analytics",
                                "type": "Copy",
                                "inputs": [
                                    {
                                        "referenceName": "AzureSqlDatabaseDataset",
                                        "type": "DatasetReference"
                                    }
                                ],
                                "outputs": [
                                    {
                                        "referenceName": "AzureSqlDWDataset",
                                        "type": "DatasetReference",
                                        "parameters": {
                                            "DWTableName": "[@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                        }
                                    }
                                ],
                                "typeProperties": {
                                    "source": {
                                        "type": "SqlSource",
                                        "sqlReaderQuery": "SELECT * FROM [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                    },
                                    "sink": {
                                        "type": "SqlDWSink",
                                        "preCopyScript": "TRUNCATE TABLE [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]",
                                        "allowPolyBase": true
                                    },
                                    "enableStaging": true,
                                    "stagingSettings": {
                                        "linkedServiceName": {
                                            "referenceName": "AzureStorageLinkedService",
                                            "type": "LinkedServiceReference"
                                        }
                                    }
                                }
                            }
                        ]
                    }
                }
            ],
            "parameters": {
                "tableList": {
                    "type": "Object"
                }
            }
        }
    }
    ```

2. Чтобы создать конвейер **IterateAndCopySQLTables**, выполните командлет **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IterateAndCopySQLTables" -File ".\IterateAndCopySQLTables.json"
    ```

    Пример выходных данных:

    ```console
    PipelineName      : IterateAndCopySQLTables
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-the-pipeline-gettablelistandtriggercopydata"></a>Создание конвейера GetTableListAndTriggerCopyData

Этот конвейер выполняет два действия:

* Ищет системную таблицу базы данных SQL Azure, чтобы получить список таблиц для копирования.
* Активирует конвейер IterateAndCopySQLTables для выполнения копирования данных.

1. Создайте файл JSON с именем **GetTableListAndTriggerCopyData.json** в папке **C:\ADFv2TutorialBulkCopy** со следующим содержимым:

    ```json
    {
        "name":"GetTableListAndTriggerCopyData",
        "properties":{
            "activities":[
                { 
                    "name": "LookupTableList",
                    "description": "Retrieve the table list from Azure SQL dataabse",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE = 'BASE TABLE' and TABLE_SCHEMA = 'SalesLT' and TABLE_NAME <> 'ProductModel'"
                        },
                        "dataset": {
                            "referenceName": "AzureSqlDatabaseDataset",
                            "type": "DatasetReference"
                        },
                        "firstRowOnly": false
                    }
                },
                {
                    "name": "TriggerCopy",
                    "type": "ExecutePipeline",
                    "typeProperties": {
                        "parameters": {
                            "tableList": {
                                "value": "@activity('LookupTableList').output.value",
                                "type": "Expression"
                            }
                        },
                        "pipeline": {
                            "referenceName": "IterateAndCopySQLTables",
                            "type": "PipelineReference"
                        },
                        "waitOnCompletion": true
                    },
                    "dependsOn": [
                        {
                            "activity": "LookupTableList",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ]
                }
            ]
        }
    }
    ```

2. Чтобы создать конвейер **GetTableListAndTriggerCopyData**, выполните командлет **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "GetTableListAndTriggerCopyData" -File ".\GetTableListAndTriggerCopyData.json"
    ```

    Пример выходных данных:

    ```console
    PipelineName      : GetTableListAndTriggerCopyData
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {LookupTableList, TriggerCopy}
    Parameters        :
    ```

## <a name="start-and-monitor-pipeline-run"></a>Запуск и мониторинг выполнения конвейера

1. Запустите выполнение главного конвейера GetTableListAndTriggerCopyData и запишите идентификатор выполнения конвейера для будущего мониторинга. В рамках конвейера запускается выполнение конвейера IterateAndCopySQLTables, как указано в действии ExecutePipeline.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'GetTableListAndTriggerCopyData'
    ```

2.  Выполните следующий скрипт, чтобы постоянно проверять состояние выполнения конвейера **GetTableListAndTriggerCopyData** и вывести окончательный результат выполнения конвейера и действия выполнения.

    ```powershell
    while ($True) {
        $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ($run.Status -ne 'InProgress') {
                Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
                Write-Host "Pipeline run details:" -foregroundcolor "Yellow"
                $run
                break
            }
            Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
        }

        Start-Sleep -Seconds 15
    }

    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result
    ```

    Вот результат примера выполнения:

    ```console
    Pipeline run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    RunId             : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    LastUpdated       : 9/18/2017 4:08:15 PM
    Parameters        : {}
    RunStart          : 9/18/2017 4:06:44 PM
    RunEnd            : 9/18/2017 4:08:15 PM
    DurationInMs      : 90637
    Status            : Succeeded
    Message           : 

    Activity run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : LookupTableList
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {source, dataset, firstRowOnly}
    Output            : {count, value, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:06:46 PM
    ActivityRunEnd    : 9/18/2017 4:07:09 PM
    DurationInMs      : 22995
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : TriggerCopy
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {pipeline, parameters, waitOnCompletion}
    Output            : {pipelineRunId}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:07:11 PM
    ActivityRunEnd    : 9/18/2017 4:08:14 PM
    DurationInMs      : 62581
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    ```

3. Вы можете получить идентификатор выполнения конвейера **IterateAndCopySQLTables** и просмотреть подробный результат выполнения действия, как показано ниже.

    ```powershell
    Write-Host "Pipeline 'IterateAndCopySQLTables' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "TriggerCopy"}).Output.ToString()
    ```

    Вот результат примера выполнения:

    ```json
    {
        "pipelineRunId": "7514d165-14bf-41fb-b5fb-789bea6c9e58"
    }
    ```

    ```powershell
    $result2 = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId <copy above run ID> -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result2
    ```

3. Подключитесь к приемнику Azure Synapse Analytics и подтвердите, что данные из базы данных SQL Azure скопированы надлежащим образом.

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы выполнили следующие шаги: 

> [!div class="checklist"]
> * Создали фабрику данных.
> * Создание экземпляров Базы данных SQL Azure, Azure Synapse Analytics и связанных служб хранилища Azure.
> * Создание Базы данных SQL Azure и наборов данных Azure Synapse Analytics.
> * Создание конвейера для поиска таблиц, которые нужно скопировать, и конвейера для выполнения операции копирования. 
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

Перейдите к следующему руководству, чтобы узнать о копировании данных по шагам из источника в место назначения:
> [!div class="nextstepaction"]
>[Пошаговая загрузка данных из базы данных SQL Azure в хранилище BLOB-объектов Azure](tutorial-incremental-copy-powershell.md)
