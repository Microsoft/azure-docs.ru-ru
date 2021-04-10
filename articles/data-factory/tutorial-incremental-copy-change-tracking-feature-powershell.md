---
title: Добавочное копирование данных с помощью решения "Отслеживание изменений" и PowerShell
description: В этом руководстве вы создадите конвейер Фабрики данных Azure, который пошагово копирует разностные данные из нескольких таблиц в базе данных SQL Server в Базу данных SQL Azure.
ms.author: yexu
author: dearandyxu
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019, devx-track-azurepowershell
ms.date: 02/18/2021
ms.openlocfilehash: a31f8ce227175e65f7119c25dcc575dc6fafdcd4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101727759"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-change-tracking-information-using-powershell"></a>Добавочная загрузка данных из Базы данных SQL Azure в хранилище BLOB-объектов Azure с использованием сведений об отслеживания изменений и PowerShell

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Из этого руководства вы узнаете, как создать фабрику данных Azure с конвейером, который загружает в хранилище BLOB-объектов Azure разностные данные на основе сведений об **отслеживании изменений** из базы данных-источника в службе "База данных SQL Azure".  

В этом руководстве вы выполните следующие шаги:

> [!div class="checklist"]
> * подготовите исходное хранилище данных;
> * Создали фабрику данных.
> * Создали связанные службы.
> * создадите источник, приемник и наборы данных отслеживания изменений;
> * создадите, запустите и начнете мониторинг конвейера полного копирования;
> * добавите или обновите данные в исходной таблице;
> * создаете, запустите и начнете мониторинг конвейера добавочного копирования.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Обзор
В решениях для интеграции данных после начальной загрузки данных широко используется добавочная загрузка. В некоторых случаях измененные за определенный период времени данные в исходном хранилище данных можно легко разделить на такие срезы, как LastModifyTime или CreationTime. В некоторых случаях определить разностные данные, полученные в ходе последней обработки данных, нет возможности. Для этого используется технология "Отслеживание изменений", поддерживаемая такими хранилищами данных, как База данных Azure SQL и SQL Server.  В этом руководстве описано, как использовать службу "Фабрика данных Azure" с технологией "Отслеживание изменений" (SQL) для добавочной загрузки разностных данных из Базы данных SQL Azure в хранилище BLOB-объектов Azure.  Дополнительные сведения о технологии "Отслеживание изменений" (SQL) см. в статье [Об отслеживании изменений (SQL Server)](/sql/relational-databases/track-changes/about-change-tracking-sql-server).

## <a name="end-to-end-workflow"></a>Комплексный рабочий процесс
Ниже описан стандартный рабочий процесс по добавочной загрузке данных с помощью технологии "Отслеживание изменений".

> [!NOTE]
> И База данных SQL Azure, и SQL Server поддерживают технологию "Отслеживание изменений". В этом руководстве в качестве исходного хранилища данных используется база данных Azure SQL. Можете также использовать экземпляр SQL Server.

1. **Начальная загрузка данных журнала** (выполняется однократно):
    1. Включите технологию "Отслеживание изменений" в базе данных-источнике, хранящейся в Базе данных SQL Azure.
    2. Получите начальное значение SYS_CHANGE_VERSION в базе данных в качестве базовых показателей для записи измененных данных.
    3. Загрузите все данные из базы данных-источника в Хранилище BLOB-объектов Azure.
2. **Добавочная загрузка разностных данных по расписанию** (выполняется периодически после начальной загрузки данных):
    1. Получите старое и новое значения SYS_CHANGE_VERSION.
    2. Загрузите разностные данные, присоединив первичные ключи измененных строк (для двух значений SYS_CHANGE_VERSION) из таблицы **sys.change_tracking_tables**, с данными в **исходной таблице**, а затем переместите разностные данные в целевое хранилище.
    3. Обновите SYS_CHANGE_VERSION для следующей загрузки разностных данных.

## <a name="high-level-solution"></a>Общее решение
В этом руководстве описано, как создать два конвейера, которые выполняют следующие две операции:  

1. **Начальная загрузка** — вы создадите конвейер с действием копирования, которое копирует все данные из исходного хранилища данных (База данных Azure SQL) в целевое хранилище данных (хранилище BLOB-объектов Azure).

    ![Полная загрузка данных](media/tutorial-incremental-copy-change-tracking-feature-powershell/full-load-flow-diagram.png)
1.  **Добавочная загрузка** — вы создадите конвейер со следующими действиями для периодического выполнения:
    1. Создайте **два действия поиска**, чтобы получить старое и новое значения SYS_CHANGE_VERSION из базы данных SQL Azure и передать их в действие копирования.
    2. Создайте **одно действие копирования**, чтобы скопировать вставленные, обновленные или удаленные данные, связанные с двумя значениями SYS_CHANGE_VERSION, из Базы данных SQL Azure в хранилище BLOB-объектов.
    3. Создайте **одно действие хранимой процедуры**, чтобы обновить значение SYS_CHANGE_VERSION для запуска следующего конвейера.

    ![Схема процесса добавочной загрузки](media/tutorial-incremental-copy-change-tracking-feature-powershell/incremental-load-flow-diagram.png)


Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Установите Azure PowerShell. Чтобы установить модули Azure PowerShell, выполните инструкции из статьи [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps).
* **База данных SQL Azure**. Используйте базу данных как **исходное** хранилище данных. Если у вас нет базы данных в службе "База данных SQL Azure", вы можете создать ее, выполнив инструкции из статьи [Создание отдельной базы данных в Базе данных SQL Azure](../azure-sql/database/single-database-create-quickstart.md).
* **Учетная запись хранения Azure.** В этом руководстве в качестве **приемника** будет использоваться хранилище BLOB-объектов. Если у вас нет учетной записи хранения Azure, ознакомьтесь с разделом [Создание учетной записи хранения](../storage/common/storage-account-create.md). Создайте контейнер с именем **adftutorial**. 

### <a name="create-a-data-source-table-in-your-database"></a>Создание таблицы источника данных в базе данных

1. Запустите **SQL Server Management Studio** и подключитесь к Базе данных SQL.
2. В **обозревателе сервера** щелкните правой кнопкой мыши **базу данных** и выберите **Создать запрос**.
3. Выполните указанную ниже команду SQL для базы данных, чтобы создать таблицу с именем `data_source_table` в качестве исходного хранилища данных.  

    ```sql
    create table data_source_table
    (
        PersonID int NOT NULL,
        Name varchar(255),
        Age int
        PRIMARY KEY (PersonID)
    );

    INSERT INTO data_source_table
        (PersonID, Name, Age)
    VALUES
        (1, 'aaaa', 21),
        (2, 'bbbb', 24),
        (3, 'cccc', 20),
        (4, 'dddd', 26),
        (5, 'eeee', 22);

    ```
4. Включите функцию **Отслеживание изменений** для базы данных и исходной таблицы (data_source_table), выполнив следующий SQL-запрос:

    > [!NOTE]
    > - Замените &lt;your database name&gt; именем базы данных, которая содержит таблицу data_source_table.
    > - Измененные данные в текущем примере хранятся в течение двух дней. Если загружать измененные данные каждые три дня или реже, некоторые измененные данные не будут включены.  В таком случае вам нужно увеличить значение CHANGE_RETENTION. В противном случае нужно следить за тем, чтобы период для загрузки измененных данных не превышал два дня. Дополнительные сведения см. в разделе [Включение отслеживания изменений для базы данных](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server#enable-change-tracking-for-a-database).

    ```sql
    ALTER DATABASE <your database name>
    SET CHANGE_TRACKING = ON  
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  

    ALTER TABLE data_source_table
    ENABLE CHANGE_TRACKING  
    WITH (TRACK_COLUMNS_UPDATED = ON)
    ```
5. Создайте таблицу и сохраните стандартное значение ChangeTracking_version, выполнив следующий запрос:

    ```sql
    create table table_store_ChangeTracking_version
    (
        TableName varchar(255),
        SYS_CHANGE_VERSION BIGINT,
    );

    DECLARE @ChangeTracking_version BIGINT
    SET @ChangeTracking_version = CHANGE_TRACKING_CURRENT_VERSION();  

    INSERT INTO table_store_ChangeTracking_version
    VALUES ('data_source_table', @ChangeTracking_version)
    ```

    > [!NOTE]
    > Если после включения отслеживания изменений для базы данных SQL данные не меняются, значение ChangeTracking_version будет равно 0.
6. Выполните следующий запрос, чтобы создать хранимую процедуру в базе данных. Конвейер вызывает эту хранимую процедуру для обновления значения ChangeTracking_version в таблице, созданной на предыдущем шаге.

    ```sql
    CREATE PROCEDURE Update_ChangeTracking_Version @CurrentTrackingVersion BIGINT, @TableName varchar(50)
    AS

    BEGIN

    UPDATE table_store_ChangeTracking_version
    SET [SYS_CHANGE_VERSION] = @CurrentTrackingVersion
    WHERE [TableName] = @TableName

    END    
    ```

### <a name="azure-powershell"></a>Azure PowerShell
Чтобы установить модули Azure PowerShell, выполните инструкции из статьи [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="create-a-data-factory"></a>Создание фабрики данных
1. Определите переменную для имени группы ресурсов, которую в дальнейшем можно будет использовать в командах PowerShell. Скопируйте текст следующей команды в PowerShell, укажите имя [группы ресурсов Azure](../azure-resource-manager/management/overview.md) в двойных кавычках, а затем выполните команду. Например: `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$resourceGroupName` другое значение и еще раз выполните команду.
2. Определите переменную для расположения фабрики данных:

    ```powershell
    $location = "East US"
    ```
3. Чтобы создать группу ресурсов Azure, выполните следующую команду:

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ```
    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$resourceGroupName` другое значение и еще раз выполните команду.
3. Определите переменную для имени фабрики данных.

    > [!IMPORTANT]
    >  Измените имя фабрики данных, чтобы оно было глобально уникальным.  

    ```powershell
    $dataFactoryName = "IncCopyChgTrackingDF";
    ```
5. Чтобы создать фабрику данных, выполните командлет **Set-AzDataFactoryV2**.

    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. Если появляется следующая ошибка, измените имя и повторите попытку.

    ```
    The specified Data Factory name 'ADFIncCopyChangeTrackingTestFactory' is already in use. Data Factory names must be globally unique.
    ```
* Чтобы создать экземпляры фабрики данных, нужно назначить учетной записи пользователя, используемой для входа в Azure, роль **участника**, **владельца** либо **администратора** подписки Azure.
* Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.


## <a name="create-linked-services"></a>Создание связанных служб
Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом разделе объясняется, как создать связанные службы для учетной записи хранения Azure и Базы данных SQL Azure.

### <a name="create-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure
На этом шаге вы свяжете учетную запись хранения Azure с фабрикой данных.

1. Создайте JSON-файл с именем **AzureStorageLinkedService.json** в папке **C:\ADFTutorials\IncCopyChangeTrackingTutorial** и добавьте в него следующее (если эта папка отсутствует, создайте ее). Перед сохранением файла замените значения `<accountName>` и `<accountKey>` на имя вашей учетной записи хранения Azure и ее ключ.

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
2. В **Azure PowerShell** перейдите к папке **C:\ADFTutorials\IncCopyChangeTrackingTutorial**.
3. Выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу. **AzureStorageLinkedService**. В указанном ниже примере вы передадите значения для параметров **ResourceGroupName** и **DataFactoryName**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

### <a name="create-azure-sql-database-linked-service"></a>Создание связанной службы Базы данных SQL Azure
На этом шаге вы свяжете базу данных с фабрикой данных.

1. Создайте JSON-файл с именем **AzureSQLDatabaseLinkedService.json** в папке **C:\ADFTutorials\IncCopyChangeTrackingTutorial** и добавьте в него следующее. Прежде чем сохранить файл, вместо значений **&lt;server&gt;, &lt;database name&gt;, &lt;user id&gt; и &lt;password&gt;** укажите имя сервера, имя базы данных, идентификатор пользователя и пароль.

    ```json
    {
        "name": "AzureSQLDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server = tcp:<server>.database.windows.net,1433;Initial Catalog=<database name>; Persist Security Info=False; User ID=<user name>; Password=<password>; MultipleActiveResultSets = False; Encrypt = True; TrustServerCertificate = False; Connection Timeout = 30;"
            }
        }
    }
    ```
2. В **Azure PowerShell** выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу: **AzureSQLDatabaseLinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>Создание наборов данных
На этом шаге вы создадите наборы данных, которые представляют данные источника и приемника, а также передадите в хранилище значение SYS_CHANGE_VERSION.

### <a name="create-a-source-dataset"></a>Создание исходного набора данных
На этом шаге вы создадите набор данных для представления исходных данных.

1. Создайте файл JSON с именем SourceDataset.json в той же папке со следующим содержимым:

    ```json
    {
        "name": "SourceDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "data_source_table"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }   
    ```

2.  Выполните командлет Set-AzDataFactoryV2Dataset, чтобы создать набор данных: SourceDataset.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    Вот пример выходных данных командлета:

    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-sink-dataset"></a>Создание набора данных приемника
На этом шаге вы создадите набор данных для представления данных, которые копируются из исходного хранилища данных.

1. Создайте файл JSON с именем SinkDataset.json в той же папке со следующим содержимым:

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/incchgtracking",
                "fileName": "@CONCAT('Incremental-', pipeline().RunId, '.txt')",
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
    ```

    Контейнер adftutorial создается в хранилище BLOB-объектов Azure как необходимый компонент. Создайте контейнер (если его еще нет) или присвойте ему имя имеющегося контейнера. В этом руководстве имя выходного файла создается динамически с помощью выражения @CONCAT('Incremental-', pipeline().RunId, '.txt').
2.  Выполните командлет Set-AzDataFactoryV2Dataset, чтобы создать набор данных: SinkDataset.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    Вот пример выходных данных командлета:

    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

### <a name="create-a-change-tracking-dataset"></a>Создание набора данных для отслеживания изменений
На этом шаге вы создадите набор данных для хранения значения ChangeTracking_version.  

1. Создайте в той же папке JSON-файл с именем ChangeTrackingDataset.json со следующим содержимым:

    ```json
    {
        "name": " ChangeTrackingDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "table_store_ChangeTracking_version"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
    ```

    Таблица table_store_ChangeTracking_version создается как необходимый компонент.
2.  Выполните командлет Set-AzDataFactoryV2Dataset, чтобы создать набор данных: ChangeTrackingDataset

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "ChangeTrackingDataset" -File ".\ChangeTrackingDataset.json"
    ```

    Вот пример выходных данных командлета:

    ```json
    DatasetName       : ChangeTrackingDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

## <a name="create-a-pipeline-for-the-full-copy"></a>Создание конвейера для полного копирования данных
На этом этапе вы создадите конвейер с действием копирования, которое копирует все данные из исходного хранилища данных (База данных Azure SQL) в целевое хранилище данных (хранилище BLOB-объектов Azure).

1. Создайте файл JSON FullCopyPipeline.json в той же папке со следующим содержимым.

    ```json
    {
        "name": "FullCopyPipeline",
        "properties": {
            "activities": [{
                "name": "FullCopyActivity",
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "SqlSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },

                "inputs": [{
                    "referenceName": "SourceDataset",
                    "type": "DatasetReference"
                }],
                "outputs": [{
                    "referenceName": "SinkDataset",
                    "type": "DatasetReference"
                }]
            }]
        }
    }
    ```
2. Выполните командлет Set-AzDataFactoryV2Pipeline, чтобы создать конвейер: FullCopyPipeline.

   ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "FullCopyPipeline" -File ".\FullCopyPipeline.json"
   ```

   Пример выходных данных:

   ```console
    PipelineName      : FullCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {FullCopyActivity}
    Parameters        :
   ```

### <a name="run-the-full-copy-pipeline"></a>Запуск конвейера полного копирования
Запустите конвейер **FullCopyPipeline**, выполнив командлет **Invoke-AzDataFactoryV2Pipeline**.

```powershell
Invoke-AzDataFactoryV2Pipeline -PipelineName "FullCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName        
```

### <a name="monitor-the-full-copy-pipeline"></a>Мониторинг конвейера полного копирования

1. Войдите на [портал Azure](https://portal.azure.com).
2. Щелкните **Все службы**, выполните поиск по ключевому слову `data factories` и выберите **Фабрики данных**.

    ![Меню "Фабрики данных"](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-data-factories-menu-1.png)
3. Найдите и выберите в списке свою **фабрику данных**, чтобы открыть страницу "Фабрика данных".

    ![Поиск фабрики данных](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-search-data-factory-2.png)
4. На странице фабрики данных, щелкните плитку **Мониторинг и управление**.

    ![Плитка Monitor & Manage (Мониторинг и управление)](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-monitor-manage-tile-3.png)    
5. На отдельной вкладке будет запущено **приложение интеграции данных**. Вы можете увидеть все **запуски конвейеров**  и их состояние. Обратите внимание, что в следующем примере состояние выполнения конвейера имеет значение **Успешно**. Чтобы проверить параметры, переданные в конвейер, перейдите по ссылке в столбце **Параметры**. Если произошла ошибка, вы увидите ссылку в столбце **Ошибка**. Щелкните ссылку в столбце **Действия**.

    ![Снимок экрана: выполнение конвейера для фабрики данных.](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-pipeline-runs-4.png)    
6. Если щелкнуть ссылку в столбце **Действия**, вы увидите следующую страницу. На ней отображаются все **выполняемые действия** в конвейере.

    ![Снимок экрана: выполнения действий для фабрики данных с активированной ссылкой "Конвейеры".](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-activity-runs-5.png)
7. Чтобы вернуться к представлению **Запуски конвейера**, щелкните ссылку **Конвейеры**, как показано на рисунке.


### <a name="review-the-results"></a>Просмотр результатов
Вы увидите файл с именем `incremental-<GUID>.txt` в папке `incchgtracking` контейнера `adftutorial`.

![Выходной файл, полученный в результате полного копирования](media/tutorial-incremental-copy-change-tracking-feature-powershell/full-copy-output-file.png)

Файл должен включать данные из базы данных:

```
1,aaaa,21
2,bbbb,24
3,cccc,20
4,dddd,26
5,eeee,22
```

## <a name="add-more-data-to-the-source-table"></a>Добавление данных в исходную таблицу

Выполните следующий запрос к базе данных, чтобы добавить и обновить строку.

```sql
INSERT INTO data_source_table
(PersonID, Name, Age)
VALUES
(6, 'new','50');


UPDATE data_source_table
SET [Age] = '10', [name]='update' where [PersonID] = 1

```

## <a name="create-a-pipeline-for-the-delta-copy"></a>Создание конвейера для копирования разностных данных
В этом шаге вы создадите конвейер со следующими действиями для периодического выполнения: **Действия поиска** получают старое и новое значения SYS_CHANGE_VERSION из базы данных SQL Azure и передают их в действие копирования. **Действие копирования** копирует вставленные, обновленные или удаленные данные, связанные с двумя значениями SYS_CHANGE_VERSION, из Базы данных SQL Azure в хранилище BLOB-объектов. **Действие хранимой процедуры** обновляет значение SYS_CHANGE_VERSION для запуска следующего конвейера.

1. Создайте файл JSON IncrementalCopyPipeline.json в той же папке со следующим содержимым.

    ```json
    {
        "name": "IncrementalCopyPipeline",
        "properties": {
            "activities": [
                {
                    "name": "LookupLastChangeTrackingVersionActivity",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select * from table_store_ChangeTracking_version"
                        },
                        "dataset": {
                            "referenceName": "ChangeTrackingDataset",
                            "type": "DatasetReference"
                        }
                    }
                },
                {
                    "name": "LookupCurrentChangeTrackingVersionActivity",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "SELECT CHANGE_TRACKING_CURRENT_VERSION() as CurrentChangeTrackingVersion"
                        },
                        "dataset": {
                            "referenceName": "SourceDataset",
                            "type": "DatasetReference"
                        }
                    }
                },
                {
                    "name": "IncrementalCopyActivity",
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select data_source_table.PersonID,data_source_table.Name,data_source_table.Age, CT.SYS_CHANGE_VERSION, SYS_CHANGE_OPERATION from data_source_table RIGHT OUTER JOIN CHANGETABLE(CHANGES data_source_table, @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.SYS_CHANGE_VERSION}) as CT on data_source_table.PersonID = CT.PersonID where CT.SYS_CHANGE_VERSION <= @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    },
                    "dependsOn": [
                        {
                            "activity": "LookupLastChangeTrackingVersionActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        },
                        {
                            "activity": "LookupCurrentChangeTrackingVersionActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ],
                    "inputs": [
                        {
                            "referenceName": "SourceDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "SinkDataset",
                            "type": "DatasetReference"
                        }
                    ]
                },
                {
                    "name": "StoredProceduretoUpdateChangeTrackingActivity",
                    "type": "SqlServerStoredProcedure",
                    "typeProperties": {
                        "storedProcedureName": "Update_ChangeTracking_Version",
                        "storedProcedureParameters": {
                            "CurrentTrackingVersion": {
                                "value": "@{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}",
                                "type": "INT64"
                            },
                            "TableName": {
                                "value": "@{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.TableName}",
                                "type": "String"
                            }
                        }
                    },
                    "linkedServiceName": {
                        "referenceName": "AzureSQLDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "dependsOn": [
                        {
                            "activity": "IncrementalCopyActivity",
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

2. Выполните командлет Set-AzDataFactoryV2Pipeline, чтобы создать конвейер: FullCopyPipeline.

   ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ```

   Пример выходных данных:

   ```console
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {LookupLastChangeTrackingVersionActivity, LookupCurrentChangeTrackingVersionActivity, IncrementalCopyActivity, StoredProceduretoUpdateChangeTrackingActivity}
    Parameters        :
   ```

### <a name="run-the-incremental-copy-pipeline"></a>Запуск конвейера добавочного копирования
Запустите конвейер **IncrementalCopyPipeline**, выполнив командлет **Invoke-AzDataFactoryV2Pipeline**.

```powershell
Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName     
```


### <a name="monitor-the-incremental-copy-pipeline"></a>Мониторинг конвейера добавочного копирования
1. В **приложении интеграции данных** обновите представление **запусков конвейера**. Вы должны увидеть в списке конвейер IncrementalCopyPipeline. Щелкните ссылку в столбце **Действия**.  

    ![Снимок экрана: выполнение конвейеров для фабрики данных, включая ваш конвейер.](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-pipeline-runs-6.png)    
2. Если щелкнуть ссылку в столбце **Действия**, вы увидите следующую страницу. На ней отображаются все **выполняемые действия** в конвейере.

    ![Снимок экрана: выполнения конвейера для фабрики данных, несколько из которых отмечены как успешные.](media/tutorial-incremental-copy-change-tracking-feature-powershell/monitor-activity-runs-7.png)
3. Чтобы вернуться к представлению **Запуски конвейера**, щелкните ссылку **Конвейеры**, как показано на рисунке.

### <a name="review-the-results"></a>Просмотр результатов
Вы увидите второй файл в папке `incchgtracking` контейнера `adftutorial`.

![Выходной файл, полученный в результате добавочного копирования](media/tutorial-incremental-copy-change-tracking-feature-powershell/incremental-copy-output-file.png)

Файл должен включать только разностные данные из базы данных. Запись с `U` — это обновленная строка в базе данных, а `I` — добавленная строка.

```
1,update,10,2,U
6,new,50,1,I
```
Первые три столбца — это измененные данные из таблицы data_source_table. Последние два столбцы — это метаданные из таблицы sys.change_tracking_tables. Четвертый столбец — это версия SYS_CHANGE_VERSION для каждой изменившейся строки. Пятый столбец — это операция  U = update, I = insert.  Дополнительные сведения об отслеживании изменений см. в описании функции [CHANGETABLE](/sql/relational-databases/system-functions/changetable-transact-sql).

```
==================================================================
PersonID Name    Age    SYS_CHANGE_VERSION    SYS_CHANGE_OPERATION
==================================================================
1        update  10            2                                 U
6        new     50            1                                 I
```


## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве рассказывается о копировании новых и измененных файлов на основе параметра LastModifiedDate:

> [!div class="nextstepaction"]
>[Copy new files by lastmodifieddate](tutorial-incremental-copy-lastmodified-copy-data-tool.md) (Копирование новых файлов с использованием параметра LastModifiedDate)
