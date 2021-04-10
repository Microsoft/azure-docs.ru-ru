---
title: Добавочное копирование нескольких таблиц с помощью PowerShell
description: В этом кратком руководстве показано, как создать фабрику данных Azure с конвейером, который загружает разностные данные из нескольких таблиц базы данных SQL Server в Базу данных SQL Azure.
ms.author: yexu
author: dearandyxu
ms.reviewer: douglasl, jburchel
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 02/18/2021
ms.openlocfilehash: 231f4033b90b152a5893cff93584d9c9bb794321
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104783189"
---
# <a name="incrementally-load-data-from-multiple-tables-in-sql-server-to-azure-sql-database-using-powershell"></a>Добавочная загрузка данных из нескольких таблиц в SQL Server в Базу данных SQL Azure с использованием PowerShell

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этом кратком руководстве показано, как создать фабрику данных Azure с конвейером, который загружает разностные данные из нескольких таблиц базы данных SQL Server в Базу данных SQL Azure.    

В этом руководстве вы выполните следующие шаги:

> [!div class="checklist"]
> * подготовите исходное и конечное хранилища данных;
> * Создали фабрику данных.
> * Создайте локальную среду выполнения интеграции.
> * Установка среды выполнения интеграции. 
> * Создали связанные службы. 
> * Создали наборы данных источника, приемника и предела.
> * создадите и запустите конвейер, а также начнете его мониторинг;
> * Просмотрите результаты операции.
> * Добавление или обновление данных в исходных таблицах.
> * Повторный запуск конвейера и выполнение его мониторинга.
> * Просмотр окончательных результатов.

## <a name="overview"></a>Обзор
Ниже приведены важные действия для создания этого решения. 

1. **Выберите столбец для предела.**

    Выберите один столбец из каждой таблицы в исходном хранилище данных, который можно использовать для идентификации новых или обновленных записей при каждом запуске. Как правило, данные в этом выбранном столбце (например, последнее_время_изменения или идентификатор) продолжают увеличиваться по мере создания или обновления строк. В качестве предела используется максимальное значение в этом столбце.

2. **Подготовьте хранилище данных для хранения значений предела.**

    В этом руководстве вы сохраните значение предела в базе данных SQL.

3. **Создайте конвейер, следуя инструкциям ниже**.
    
    а. Создайте действие ForEach, которое выполняет итерацию по списку имен исходной таблицы. Этот список передается в конвейер в качестве параметра. В каждой исходной таблице этот параметр вызывает следующие действия для загрузки разностных данных для этой таблицы.

    b. Создайте два действия поиска. Используйте первое действие поиска для получения последнего значения предела, а второе для получения нового значения предела. Эти значения передаются в действие копирования.

    c. Создайте действие копирования, копирующее строки из исходного хранилища данных со значениями столбцов предела, которые выше значений старого предела и меньше значений нового. Затем оно копирует разностные данные из исходного хранилища данных в хранилище BLOB-объектов Azure в качестве нового файла.

    d. Создайте действие хранимой процедуры, которое обновляет значение предела для конвейера при последующем выполнении. 

    Ниже приведена общая схема решения. 

    ![Пошаговая загрузка данных](media/tutorial-incremental-copy-multiple-tables-powershell/high-level-solution-diagram.png)


Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* **SQL Server.** В этом учебнике используйте базу данных SQL Server в качестве исходного хранилища данных. 
* **База данных SQL Azure**. Базу данных в службе "База данных SQL Azure" следует использовать в качестве принимающего хранилища данных. Если у вас нет базы данных SQL, создайте ее, следуя указаниям из руководства [Создание отдельной базы данных в Базе данных SQL Azure](../azure-sql/database/single-database-create-quickstart.md). 

### <a name="create-source-tables-in-your-sql-server-database"></a>Создание исходных таблиц в базе данных SQL Server

1. Откройте [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) или [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) и подключитесь к базе данных SQL Server.

2. В **Обозревателе сервера (SSMS)** или в **Области подключения (Azure Data Studio)** , щелкните правой кнопкой мыши на базу данных и выберите **Новый запрос**.

3. Выполните следующую команду SQL в базе данных, чтобы создать таблицы с именами `customer_table` и `project_table`.

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
        
    INSERT INTO customer_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'John','9/1/2017 12:56:00 AM'),
    (2, 'Mike','9/2/2017 5:23:00 AM'),
    (3, 'Alice','9/3/2017 2:36:00 AM'),
    (4, 'Andy','9/4/2017 3:21:00 AM'),
    (5, 'Anny','9/5/2017 8:06:00 AM');
    
    INSERT INTO project_table
    (Project, Creationtime)
    VALUES
    ('project1','1/1/2015 0:00:00 AM'),
    ('project2','2/2/2016 1:23:00 AM'),
    ('project3','3/4/2017 5:16:00 AM');
    ```

### <a name="create-destination-tables-in-your-azure-sql-database"></a>Создание целевых таблиц в базе данных SQL Azure

1. Откройте [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) или [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) и подключитесь к базе данных SQL Server.

2. В **Обозревателе сервера (SSMS)** или в **Области подключения (Azure Data Studio)** , щелкните правой кнопкой мыши на базу данных и выберите **Новый запрос**.

3. Выполните следующую команду SQL в базе данных, чтобы создать таблицы с именами `customer_table` и `project_table`.  

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
    ```

### <a name="create-another-table-in-azure-sql-database-to-store-the-high-watermark-value"></a>Создание дополнительной таблицы в Базе данных SQL Azure для хранения значения верхнего предела

1. Выполните указанную ниже команду SQL для базы данных, чтобы создать таблицу с именем `watermarktable` для хранения значения предела. 
    
    ```sql
    create table watermarktable
    (
    
        TableName varchar(255),
        WatermarkValue datetime,
    );
    ```
2. Вставьте исходные значения предела для обеих исходных таблиц в таблицу значений предела.

    ```sql

    INSERT INTO watermarktable
    VALUES 
    ('customer_table','1/1/2010 12:00:00 AM'),
    ('project_table','1/1/2010 12:00:00 AM');
    
    ```

### <a name="create-a-stored-procedure-in-the-azure-sql-database"></a>Создание хранимой процедуры в Базе данных SQL Azure 

Выполните указанную ниже команду, чтобы создать хранимую процедуру в своей базе данных. Эта хранимая процедура обновляет значение предела после каждого запуска конвейера. 

```sql
CREATE PROCEDURE usp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

UPDATE watermarktable
SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName

END

```

### <a name="create-data-types-and-additional-stored-procedures-in-azure-sql-database"></a>Создание типов данных и дополнительных хранимых процедур в Базе данных SQL Azure

Выполните следующий запрос, чтобы создать две хранимые процедуры и два типа данных в своей базе данных. Они используются для объединения данных из исходных в целевые таблицы. 

Чтобы упростить начало работы, мы непосредственно используем эти хранимые процедуры, передающие разностные данные в табличной переменной, а затем объединяем их в целевое хранилище. Учтите, что для хранения в табличной переменной не ожидается большое число строк разностных данных (более 100).  

Если нужно объединить большое количество строк разностных данных в целевом хранилище, мы рекомендуем с помощью действия копирования сначала скопировать разностные данные во временную "промежуточную" таблицу в целевом хранилище, а затем создать собственную хранимую процедуру, не используя табличную переменную для их объединения из "промежуточной" таблицы в "итоговую". 


```sql
CREATE TYPE DataTypeforCustomerTable AS TABLE(
    PersonID int,
    Name varchar(255),
    LastModifytime datetime
);

GO

CREATE PROCEDURE usp_upsert_customer_table @customer_table DataTypeforCustomerTable READONLY
AS

BEGIN
  MERGE customer_table AS target
  USING @customer_table AS source
  ON (target.PersonID = source.PersonID)
  WHEN MATCHED THEN
      UPDATE SET Name = source.Name,LastModifytime = source.LastModifytime
  WHEN NOT MATCHED THEN
      INSERT (PersonID, Name, LastModifytime)
      VALUES (source.PersonID, source.Name, source.LastModifytime);
END

GO

CREATE TYPE DataTypeforProjectTable AS TABLE(
    Project varchar(255),
    Creationtime datetime
);

GO

CREATE PROCEDURE usp_upsert_project_table @project_table DataTypeforProjectTable READONLY
AS

BEGIN
  MERGE project_table AS target
  USING @project_table AS source
  ON (target.Project = source.Project)
  WHEN MATCHED THEN
      UPDATE SET Creationtime = source.Creationtime
  WHEN NOT MATCHED THEN
      INSERT (Project, Creationtime)
      VALUES (source.Project, source.Creationtime);
END
```

### <a name="azure-powershell"></a>Azure PowerShell

Чтобы установить новые модули Azure PowerShell, выполните инструкции из статьи [Установка и настройка Azure PowerShell](/powershell/azure/azurerm/install-azurerm-ps).

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. Определите переменную для имени группы ресурсов, которую в дальнейшем можно будет использовать в командах PowerShell. Скопируйте текст следующей команды в PowerShell, укажите имя [группы ресурсов Azure](../azure-resource-manager/management/overview.md) в двойных кавычках, а затем выполните команду. Например, `"adfrg"`.

    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Если группа ресурсов уже имеется, вы можете не перезаписывать ее. Назначьте переменной `$resourceGroupName` другое значение и еще раз выполните команду.

2. Определите переменную для расположения фабрики данных. 

    ```powershell
    $location = "East US"
    ```
3. Чтобы создать группу ресурсов Azure, выполните следующую команду: 

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ``` 
    Если группа ресурсов уже имеется, вы можете не перезаписывать ее. Назначьте переменной `$resourceGroupName` другое значение и еще раз выполните команду.

4. Определите переменную для имени фабрики данных. 

    > [!IMPORTANT]
    >  Измените имя фабрики данных, чтобы сделать его глобально уникальным. Например, ADFIncMultiCopyTutorialFactorySP1127. 

    ```powershell
    $dataFactoryName = "ADFIncMultiCopyTutorialFactory";
    ```
5. Чтобы создать фабрику данных, выполните командлет **Set-AzDataFactoryV2**. 
    
    ```powershell
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных должно быть глобально уникальным. Если появляется следующая ошибка, измените имя и повторите попытку.

    ```powershell
    Set-AzDataFactoryV2 : HTTP Status Code: Conflict
    Error Code: DataFactoryNameInUse
    Error Message: The specified resource name 'ADFIncMultiCopyTutorialFactory' is already in use. Resource names must be globally unique.
    ```

* Чтобы создать экземпляры фабрики данных, нужно назначить учетной записи пользователя, используемой для входа в Azure, роль участника, владельца либо администратора подписки Azure.

* Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, База данных SQL, Управляемый экземпляр SQL и т. д.) и вычислительные ресурсы (Azure HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.

[!INCLUDE [data-factory-create-install-integration-runtime](../../includes/data-factory-create-install-integration-runtime.md)]

## <a name="create-linked-services"></a>Создание связанных служб

Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В рамках этого раздела вы создадите связанные службы для базы данных SQL Server и базы данных в службе "База данных SQL Azure". 

### <a name="create-the-sql-server-linked-service"></a>Создание связанной службы SQL Server

На этом шаге вы свяжете базу данных SQL Server с фабрикой данных.

1. Создайте JSON-файл с именем **SqlServerLinkedService.json** в папке C:\ADFTutorials\IncCopyMultiTableTutorial (создайте локальные папки, если они не существуют) со следующим содержимым. Выберите правильный раздел в зависимости от типа проверки подлинности, используемого для подключения к SQL Server.  

    > [!IMPORTANT]
    > Выберите правильный раздел в зависимости от типа проверки подлинности, используемого для подключения к SQL Server.

    Если используется проверка подлинности SQL, скопируйте следующее определение JSON:

    ```json
    {  
        "name":"SqlServerLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=False;data source=<servername>;initial catalog=<database name>;user id=<username>;Password=<password>"
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    } 
   ```    
    Если используется проверка подлинности Windows, скопируйте следующее определение JSON:

    ```json
    {  
        "name":"SqlServerLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=True;data source=<servername>;initial catalog=<database name>",
                "userName":"<username> or <domain>\\<username>",
                "password":{  
                    "type":"SecureString",
                    "value":"<password>"
                }
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
    ```
    > [!IMPORTANT]
    > - Выберите правильный раздел в зависимости от типа проверки подлинности, используемого для подключения к SQL Server.
    > - Замените &lt;integration runtime name> именем своей среды выполнения интеграции.
    > - Перед сохранением файла замените &lt;servername>, &lt;databasename>, &lt;username> и &lt;password> значениями имени сервера, базы данных, пользователя и пароля для базы данных SQL Server.
    > - Если в имени учетной записи пользователя или имени сервера необходимо использовать символ косой черты (`\`), добавьте escape-символ (`\`). Например, `mydomain\\myuser`.

2. В PowerShell запустите следующий командлет, чтобы перейти к папке C:\ADFTutorials\IncCopyMultiTableTutorial.

    ```powershell
    Set-Location 'C:\ADFTutorials\IncCopyMultiTableTutorial'
    ```

3. Выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу AzureStorageLinkedService. В указанном ниже примере вы передадите значения для параметров *ResourceGroupName* и *DataFactoryName*. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerLinkedService" -File ".\SqlServerLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : SqlServerLinkedService
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerLinkedService
    ```

### <a name="create-the-sql-database-linked-service"></a>Создание связанной службы Базы данных SQL

1. Создайте JSON-файл с именем **AzureSQLDatabaseLinkedService.json** в папке C:\ADFTutorials\IncCopyMultiTableTutorial и добавьте в него следующее содержимое. (Если папка ADF отсутствует, создайте ее.) Вместо значений &lt;servername&gt;, &lt;database name&gt;, &lt;user name&gt; и &lt;password&gt; укажите имя сервера SQL Server, имя базы данных, имя пользователя и пароль, прежде чем сохранить файл. 

    ```json
    {  
        "name":"AzureSQLDatabaseLinkedService",
        "properties":{  
            "annotations":[  
    
            ],
            "type":"AzureSqlDatabase",
            "typeProperties":{  
                "connectionString":"integrated security=False;encrypt=True;connection timeout=30;data source=<servername>.database.windows.net;initial catalog=<database name>;user id=<user name>;Password=<password>;"
            }
        }
    }
    ```
2. В PowerShell выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу AzureSQLDatabaseLinkedService. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>Создание наборов данных

На этом шаге вы создадите наборы данных для представления источника данных, назначение данных и место для хранения предела.

### <a name="create-a-source-dataset"></a>Создание исходного набора данных

1. Создайте файл JSON с именем **SourceDataset.json** в той же папке со следующим содержимым: 

    ```json
    {  
        "name":"SourceDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"SqlServerLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  
    
            ],
            "type":"SqlServerTable",
            "schema":[  
    
            ]
        }
    }
   
    ```

    Действие копирования в конвейере использует SQL-запрос для загрузки данных, вместо того чтобы загружать всю таблицу.

2. Выполните командлет **Set-AzDataFactoryV2Dataset**, чтобы создать набор данных SourceDataset.
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    Вот пример выходных данных командлета:
    
    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-sink-dataset"></a>Создание набора данных приемника

1. Создайте файл JSON с именем **SinkDataset.json** в той же папке со следующим содержимым. Элемент tableName динамически задается конвейером в среде выполнения. Действие ForEach в конвейере выполняет итерацию по списку имен таблиц и передает имя таблицы для этого набора данных в каждой итерации. 

    ```json
    {  
        "name":"SinkDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"AzureSQLDatabaseLinkedService",
                "type":"LinkedServiceReference"
            },
            "parameters":{  
                "SinkTableName":{  
                    "type":"String"
                }
            },
            "annotations":[  
    
            ],
            "type":"AzureSqlTable",
            "typeProperties":{  
                "tableName":{  
                    "value":"@dataset().SinkTableName",
                    "type":"Expression"
                }
            }
        }
    }
    ```

2. Выполните командлет **Set-AzDataFactoryV2Dataset**, чтобы создать набор данных SinkDataset.
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    Вот пример выходных данных командлета:
    
    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-a-watermark"></a>Создание набора данных для предела

На этом шаге вы создадите набор данных для хранения значения верхнего предела. 

1. Создайте файл JSON с именем **WatermarkDataset.json** в той же папке со следующим содержимым: 

    ```json
    {
        "name": " WatermarkDataset ",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "watermarktable"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }    
    ```
2. Выполните командлет **Set-AzDataFactoryV2Dataset**, чтобы создать набор данных WatermarkDataset.
    
    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "WatermarkDataset" -File ".\WatermarkDataset.json"
    ```

    Вот пример выходных данных командлета:
    
    ```json
    DatasetName       : WatermarkDataset
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset    
    ```

## <a name="create-a-pipeline"></a>Создание конвейера

Этот конвейер принимает список имен таблиц в качестве параметра. Действие **ForEach** выполняет итерацию по списку имен таблиц, а затем выполняет следующие операции: 

1. Использует **действие поиска**, чтобы получить старое значение предела (начальное значение или значение, используемое в последней итерации).

2. Использует **действие поиска**, чтобы получить новое значение предела (максимальное значение в столбце предела в исходной таблице).

3. Использует **действие копирования**, чтобы скопировать данные между двумя значениями пределов из исходной в целевую базу данных.

4. Использует **действие хранимой процедуры**, чтобы обновить старое значение предела для использования на первом шаге следующей итерации. 

### <a name="create-the-pipeline"></a>Создание конвейера

1. Создайте JSON-файл с именем **IncrementalCopyPipeline.json** в той же папке со следующим содержимым: 

    ```json
    {  
        "name":"IncrementalCopyPipeline",
        "properties":{  
            "activities":[  
                {  
                    "name":"IterateSQLTables",
                    "type":"ForEach",
                    "dependsOn":[  
    
                    ],
                    "userProperties":[  
    
                    ],
                    "typeProperties":{  
                        "items":{  
                            "value":"@pipeline().parameters.tableList",
                            "type":"Expression"
                        },
                        "isSequential":false,
                        "activities":[  
                            {  
                                "name":"LookupOldWaterMarkActivity",
                                "type":"Lookup",
                                "dependsOn":[  
    
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"AzureSqlSource",
                                        "sqlReaderQuery":{  
                                            "value":"select * from watermarktable where TableName  =  '@{item().TABLE_NAME}'",
                                            "type":"Expression"
                                        }
                                    },
                                    "dataset":{  
                                        "referenceName":"WatermarkDataset",
                                        "type":"DatasetReference"
                                    }
                                }
                            },
                            {  
                                "name":"LookupNewWaterMarkActivity",
                                "type":"Lookup",
                                "dependsOn":[  
    
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"SqlServerSource",
                                        "sqlReaderQuery":{  
                                            "value":"select MAX(@{item().WaterMark_Column}) as NewWatermarkvalue from @{item().TABLE_NAME}",
                                            "type":"Expression"
                                        }
                                    },
                                    "dataset":{  
                                        "referenceName":"SourceDataset",
                                        "type":"DatasetReference"
                                    },
                                    "firstRowOnly":true
                                }
                            },
                            {  
                                "name":"IncrementalCopyActivity",
                                "type":"Copy",
                                "dependsOn":[  
                                    {  
                                        "activity":"LookupOldWaterMarkActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    },
                                    {  
                                        "activity":"LookupNewWaterMarkActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    }
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "source":{  
                                        "type":"SqlServerSource",
                                        "sqlReaderQuery":{  
                                            "value":"select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and @{item().WaterMark_Column} <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'",
                                            "type":"Expression"
                                        }
                                    },
                                    "sink":{  
                                        "type":"AzureSqlSink",
                                        "sqlWriterStoredProcedureName":{  
                                            "value":"@{item().StoredProcedureNameForMergeOperation}",
                                            "type":"Expression"
                                        },
                                        "sqlWriterTableType":{  
                                            "value":"@{item().TableType}",
                                            "type":"Expression"
                                        },
                                        "storedProcedureTableTypeParameterName":{  
                                            "value":"@{item().TABLE_NAME}",
                                            "type":"Expression"
                                        },
                                        "disableMetricsCollection":false
                                    },
                                    "enableStaging":false
                                },
                                "inputs":[  
                                    {  
                                        "referenceName":"SourceDataset",
                                        "type":"DatasetReference"
                                    }
                                ],
                                "outputs":[  
                                    {  
                                        "referenceName":"SinkDataset",
                                        "type":"DatasetReference",
                                        "parameters":{  
                                            "SinkTableName":{  
                                                "value":"@{item().TABLE_NAME}",
                                                "type":"Expression"
                                            }
                                        }
                                    }
                                ]
                            },
                            {  
                                "name":"StoredProceduretoWriteWatermarkActivity",
                                "type":"SqlServerStoredProcedure",
                                "dependsOn":[  
                                    {  
                                        "activity":"IncrementalCopyActivity",
                                        "dependencyConditions":[  
                                            "Succeeded"
                                        ]
                                    }
                                ],
                                "policy":{  
                                    "timeout":"7.00:00:00",
                                    "retry":0,
                                    "retryIntervalInSeconds":30,
                                    "secureOutput":false,
                                    "secureInput":false
                                },
                                "userProperties":[  
    
                                ],
                                "typeProperties":{  
                                    "storedProcedureName":"[dbo].[usp_write_watermark]",
                                    "storedProcedureParameters":{  
                                        "LastModifiedtime":{  
                                            "value":{  
                                                "value":"@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}",
                                                "type":"Expression"
                                            },
                                            "type":"DateTime"
                                        },
                                        "TableName":{  
                                            "value":{  
                                                "value":"@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}",
                                                "type":"Expression"
                                            },
                                            "type":"String"
                                        }
                                    }
                                },
                                "linkedServiceName":{  
                                    "referenceName":"AzureSQLDatabaseLinkedService",
                                    "type":"LinkedServiceReference"
                                }
                            }
                        ]
                    }
                }
            ],
            "parameters":{  
                "tableList":{  
                    "type":"array"
                }
            },
            "annotations":[  
    
            ]
        }
    }
    ```
2. Выполните командлет **Set-AzDataFactoryV2Pipeline**, чтобы создать конвейер IncrementalCopyPipeline.
    
   ```powershell
   Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ``` 

   Пример выходных данных: 

   ```console
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
   ```
 
## <a name="run-the-pipeline"></a>Запуск конвейера

1. Создайте файл параметров с именем **Parameters.json** в той же папке со следующим содержимым:

    ```json
    {
        "tableList": 
        [
            {
                "TABLE_NAME": "customer_table",
                "WaterMark_Column": "LastModifytime",
                "TableType": "DataTypeforCustomerTable",
                "StoredProcedureNameForMergeOperation": "usp_upsert_customer_table"
            },
            {
                "TABLE_NAME": "project_table",
                "WaterMark_Column": "Creationtime",
                "TableType": "DataTypeforProjectTable",
                "StoredProcedureNameForMergeOperation": "usp_upsert_project_table"
            }
        ]
    }
    ```
2. Запустите конвейер IncrementalCopyPipeline, выполнив командлет **Invoke-AzDataFactoryV2Pipeline**. Замените заполнители собственными именами группы ресурсов и фабрики данных.

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"        
    ``` 

## <a name="monitor-the-pipeline"></a>Мониторинг конвейера

1. Войдите на [портал Azure](https://portal.azure.com).

2. Выберите **Все службы**, выполните поиск по фразе *Фабрики данных* и выберите **Фабрики данных**. 

3. Найдите и выберите в списке свою фабрику данных, чтобы открыть страницу **Фабрика данных**. 

4. На странице **Фабрика данных** выберите **Создание и мониторинг**, чтобы запустить Фабрику данных Azure на отдельной вкладке.

5. На странице **Начало работы** выберите **Мониторинг** слева. 
![Снимок экрана: страница "Начало работы" Фабрики данных Azure](media/doc-common-process/get-started-page-monitor-button.png)    

6. Вы можете увидеть все запуски конвейеров и их состояние. Обратите внимание, что в следующем примере состояние выполнения конвейера имеет значение **Успешно**. Чтобы проверить параметры, переданные в конвейер, щелкните ссылку в столбце **Параметры**. Если произошла ошибка, вы увидите ссылку в столбце **Ошибка**.

    ![Снимок экрана: выполнение конвейеров для фабрики данных, включая ваш конвейер.](media/tutorial-incremental-copy-multiple-tables-powershell/monitor-pipeline-runs-4.png)    
7. Если щелкнуть ссылку в столбце **Действия**, вы увидите все выполняемые действия в конвейере. 

8. Выберите **All Pipeline Runs** (Все запуски конвейера), чтобы вернуться к представлению **Pipeline Runs** (Запуски конвейера). 

## <a name="review-the-results"></a>Просмотр результатов

В SQL Server Management Studio выполните следующие запросы в целевой базе данных SQL. Так вы проверите, что данные скопированы из исходных таблиц в целевые. 

**Запрос** 
```sql
select * from customer_table
```

**Выходные данные**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           Alice   2017-09-03 02:36:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

**Запрос**

```sql
select * from project_table
```

**Выходные данные**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
```

**Запрос**

```sql
select * from watermarktable
```

**Выходные данные**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-05 08:06:00.000
project_table   2017-03-04 05:16:00.000
```

Обратите внимание, что значения предела для обеих таблиц обновились. 

## <a name="add-more-data-to-the-source-tables"></a>Добавление данных в исходные таблицы

Выполните следующий запрос в исходной базе данных SQL Server, чтобы обновить имеющуюся строку в таблице customer_table. Вставьте новую строку в таблицу project_table. 

```sql
UPDATE customer_table
SET [LastModifytime] = '2017-09-08T00:00:00Z', [name]='NewName' where [PersonID] = 3

INSERT INTO project_table
(Project, Creationtime)
VALUES
('NewProject','10/1/2017 0:00:00 AM');
``` 

## <a name="rerun-the-pipeline"></a>Повторный запуск конвейера

1. Теперь снова запустите конвейер. Для этого выполните следующую команду PowerShell:

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupname -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"
    ```
2. Чтобы отслеживать выполнение конвейера, следуйте инструкциям из раздела [Мониторинг конвейера](#monitor-the-pipeline). Когда конвейер находится в состоянии **Выполняется**, отображается ссылка на другое действие в столбце **Действия**, с помощью которой можно отменить выполнение конвейера. 

3. Выберите **Обновить**, чтобы обновить список, пока конвейер не будет выполнен. 

4. (Необязательно.) Щелкните ссылку **View Activity Runs** (Просмотреть выполнения действий) в области **Действия**, чтобы просмотреть все выполнения действий, связанные с этим запуском конвейера. 

## <a name="review-the-final-results"></a>Просмотр окончательных результатов

В SQL Server Management Studio выполните следующие запросы в целевой базе данных SQL Azure. Так вы проверите, что обновленные и новые данные скопированы из исходных таблиц в целевые. 

**Запрос** 
```sql
select * from customer_table
```

**Выходные данные**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           NewName 2017-09-08 00:00:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

Обратите внимание на новые значения в столбцах **Name** и **LastModifytime** для строки под номером 3 в столбце **PersonID**. 

**Запрос**

```sql
select * from project_table
```

**Выходные данные**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
NewProject  2017-10-01 00:00:00.000
```

Обратите внимание, что запись **NewProject** добавлена в таблицу project_table. 

**Запрос**

```sql
select * from watermarktable
```

**Выходные данные**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-08 00:00:00.000
project_table   2017-10-01 00:00:00.000
```

Обратите внимание, что значения предела для обеих таблиц обновились.

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы выполнили следующие шаги: 

> [!div class="checklist"]
> * подготовите исходное и конечное хранилища данных;
> * Создали фабрику данных.
> * Создание локальной среды выполнения интеграции (IR).
> * Установка среды выполнения интеграции.
> * Создали связанные службы. 
> * Создали наборы данных источника, приемника и предела.
> * создадите и запустите конвейер, а также начнете его мониторинг;
> * Просмотрите результаты операции.
> * Добавление или обновление данных в исходных таблицах.
> * Повторный запуск конвейера и выполнение его мониторинга.
> * Просмотр окончательных результатов.

Перейдите к следующему руководству, чтобы узнать о преобразовании данных с помощью кластера Spark в Azure:

> [!div class="nextstepaction"]
>[Добавочная загрузка данных из базы данных SQL Azure в хранилище BLOB-объектов Azure с использованием сведений об отслеживании изменений](tutorial-incremental-copy-change-tracking-feature-powershell.md)
