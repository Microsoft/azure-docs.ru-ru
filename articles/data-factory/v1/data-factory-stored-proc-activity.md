---
title: Действие "Хранимая процедура SQL Server"
description: Узнайте, как можно использовать действие SQL Server хранимой процедуры для вызова хранимой процедуры в базе данных SQL Azure или Azure синапсе Analytics из конвейера фабрики данных.
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
author: nabhishek
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 05717352936bed888e108277d0163e43bc5a37af
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100368767"
---
# <a name="sql-server-stored-procedure-activity"></a>Действие "Хранимая процедура SQL Server"
> [!div class="op_single_selector" title1="Действия преобразования"]
> * [Действие Hive](data-factory-hive-activity.md)
> * [Действие Pig](data-factory-pig-activity.md)
> * [Действие MapReduce](data-factory-map-reduce.md)
> * [Действие потоковой передачи Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Действие Spark](data-factory-spark.md)
> * [Действие выполнения пакета в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-batch-execution-activity.md)
> * [Действие обновления ресурса в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-update-resource-activity.md)
> * [Действие хранимой процедуры](data-factory-stored-proc-activity.md)
> * [Действие U-SQL в Data Lake Analytics](data-factory-usql-activity.md)
> * [Настраиваемое действие .NET](data-factory-use-custom-activities.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. руководство по [преобразованию данных с помощью действия хранимой процедуры в службе "Фабрика данных"](../transform-data-using-stored-procedure.md).

## <a name="overview"></a>Обзор
Действия преобразования данных в [конвейере](data-factory-create-pipelines.md) фабрики данных позволяют преобразовать необработанные данные и переработать их в прогнозы и аналитику. Действие хранимой процедуры — это одно из действий преобразования данных, которые поддерживает фабрика данных. Данная статья основана на материалах статьи о [действиях преобразования данных](data-factory-data-transformation-activities.md), в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования в фабрике данных.

C его помощью можно вызвать хранимую процедуру в одном из следующих хранилищ данных вашего предприятия или на виртуальной машине Azure.

- База данных SQL Azure
- Azure Synapse Analytics
- База данных SQL Server. Если вы используете SQL Server, установите шлюз управления данными на том же компьютере, на котором размещена база данных, или на отдельном компьютере, имеющем доступ к базе данных. Шлюз управления данными — это компонент, который обеспечивает безопасное и управляемое подключение локальных источников данных или данных виртуальной машины Azure к облачным службам. Дополнительные сведения см. в статье [Шлюз управления данными](data-factory-data-management-gateway.md).

> [!IMPORTANT]
> При копировании данных в базу данных SQL Azure или SQL Server можно настроить класс **SqlSink** в действии копирования для вызова хранимой процедуры с помощью свойства **sqlWriterStoredProcedureName**. Дополнительные сведения см. в статье [Вызов хранимой процедуры из действия копирования в фабрике данных Azure](data-factory-invoke-stored-procedure-from-copy-activity.md). Дополнительные сведения о свойствах см. в следующих статьях о соединителях: [Перемещение данных в базу данных SQL Azure и из нее с помощью фабрики данных Azure](data-factory-azure-sql-connector.md#copy-activity-properties) и [Перемещение данных в базу данных SQL Server и обратно на локальных компьютерах и виртуальных машинах Azure IaaS с помощью фабрики данных Azure](data-factory-sqlserver-connector.md#copy-activity-properties). Вызов хранимой процедуры при копировании данных в Azure синапсе Analytics с помощью действия копирования не поддерживается. Однако действие хранимой процедуры можно использовать для вызова хранимой процедуры в Azure синапсе Analytics.
>
> При копировании данных из базы данных SQL Azure или SQL Server или Azure синапсе Analytics можно настроить **SqlSource** в действии копирования, чтобы вызвать хранимую процедуру для считывания данных из базы данных-источника с помощью свойства **sqlReaderStoredProcedureName** . Дополнительные сведения см. в следующих статьях о соединителях: [база данных SQL Azure](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties), [Azure синапсе Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties) .

В следующем пошаговом руководстве действие хранимой процедуры в конвейере используется для вызова хранимой процедуры в базе данных SQL Azure.

## <a name="walkthrough"></a>Пошаговое руководство
### <a name="sample-table-and-stored-procedure"></a>Образец таблицы и хранимая процедура
1. Создайте следующую **таблицу** в базе данных SQL Azure с помощью SQL Server Management Studio или другого подходящего инструмента. Столбец datetimestamp содержит дату и время создания соответствующего идентификатора.

    ```SQL
    CREATE TABLE dbo.sampletable
    (
        Id uniqueidentifier,
        datetimestamp nvarchar(127)
    )
    GO

    CREATE CLUSTERED INDEX ClusteredID ON dbo.sampletable(Id);
    GO
    ```
    Id — уникальный идентификатор, а столбец datetimestamp содержит дату и время создания соответствующего идентификатора.
    
    ![Пример данных](./media/data-factory-stored-proc-activity/sample-data.png)

    В этом примере хранимая процедура находится в базе данных SQL Azure. Если хранимая процедура находится в Azure синапсе Analytics и SQL Server базе данных, подход аналогичен. В случае с базой данных SQL Server необходимо установить [шлюз управления данными](data-factory-data-management-gateway.md).
2. Создайте следующую **хранимую процедуру**, вставляющую данные в таблицу **sampletable**.

    ```SQL
    CREATE PROCEDURE usp_sample @DateTime nvarchar(127)
    AS

    BEGIN
        INSERT INTO [sampletable]
        VALUES (newid(), @DateTime)
    END
    ```

   > [!IMPORTANT]
   > **Имя** и **регистр** параметра (в этом примере — DateTime) должны соответствовать имени и регистру параметра, указанного в конвейере или действии JSON. В определении хранимой процедуры убедитесь, что **\@** используется в качестве префикса для параметра.

### <a name="create-a-data-factory"></a>Создание фабрики данных
1. Войдите на [портал Azure](https://portal.azure.com/).
2. Щелкните **Создать** в меню слева, выберите **Аналитика** и щелкните **Фабрика данных**.

    ![Новая фабрика данных 1](media/data-factory-stored-proc-activity/new-data-factory.png)
3. В колонке **Новая фабрика данных** введите **SProcDF** в поле "Имя". Имена фабрики данных Azure являются **глобально уникальными**. Для успешного создания фабрики добавьте свое имя в качестве префикса имени фабрики.

   ![Новая фабрика данных 2](media/data-factory-stored-proc-activity/new-data-factory-blade.png)
4. Выберите свою **подписку Azure**.
5. Для **группы ресурсов** выполните одно из следующих действий.
   1. Щелкните **Создать** и введите имя группы ресурсов.
   2. Щелкните **Использовать существующий** и укажите имеющуюся группу ресурсов.
6. Укажите **расположение** фабрики данных.
7. Выберите **Закрепить на панели мониторинга**, чтобы при следующем входе фабрика данных отобразилась на панели мониторинга.
8. В колонке **Создание фабрики данных** нажмите кнопку **Создать**.
9. Созданная фабрика данных появится на **панели мониторинга** портала Azure. Ее содержимое отобразится на соответствующей странице.

   ![Домашняя страница фабрики данных](media/data-factory-stored-proc-activity/data-factory-home-page.png)

### <a name="create-an-azure-sql-linked-service"></a>Создание связанной службы SQL Azure
После создания фабрики данных вы создадите связанную службу SQL Azure, которая связывает базу данных в базе данных SQL Azure, которая содержит таблицу sampletable и usp_sample хранимую процедуру, в фабрику данных.

1. В колонке **Фабрика данных** щелкните **Создать и развернуть** для **SProcDF**, чтобы запустить редактор фабрики данных.
2. Щелкните **Новое хранилище данных** в командной строке и выберите **База данных Azure SQL**. В редакторе отобразится сценарий JSON для создания связанной службы SQL Azure.

   ![Новое хранилище данных 1](media/data-factory-stored-proc-activity/new-data-store.png)
3. Внесите следующие изменения в сценарий JSON:

   1. Замените `<servername>` именем сервера.
   2. Замените `<databasename>` базой данных, в которой создана таблица и хранимая процедура.
   3. Замените `<username@servername>` учетной записью пользователя, у которой есть доступ к базе данных.
   4. Замените `<password>` паролем к учетной записи пользователя.

      ![Новое хранилище данных 2](media/data-factory-stored-proc-activity/azure-sql-linked-service.png)
4. Чтобы развернуть связанную службу, нажмите кнопку **Развернуть** на панели команд. Экземпляр AzureSqlLinkedService должен отображаться в иерархическом представлении слева.

    ![представление в виде дерева со связанной службой 1](media/data-factory-stored-proc-activity/tree-view.png)

### <a name="create-an-output-dataset"></a>Создание выходного набора данных
Необходимо указать набор выходных данных для действия хранимой процедуры, даже если хранимая процедура не возвращает данные. Это связано с тем, что набор выходных данных управляет расписанием действия (то, как часто запускается действие: ежечасно, ежедневно и т. д.). Выходной набор данных должен использовать **связанную службу** , которая ссылается на базу данных SQL Azure или Azure синапсе Analytics, или на базу данных SQL Server, в которой должна выполняться хранимая процедура. Выходной набор данных может использоваться для передачи результатов хранимой процедуры для дальнейшей обработки с помощью другого действия ([цепочки действий](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) в конвейере. Тем не менее фабрика данных не записывает выходные данные хранимой процедуры в этот набор данных автоматически. Выходные данные записывает сама хранимая процедура в таблицу SQL, на которую указывает выходной набор данных. В некоторых случаях набор выходных данных может быть **фиктивным** (набор данных, указывающий на таблицу, которая не содержит выходные данные хранимой процедуры). Такой набор данных используется, только чтобы задать расписание выполнения действия хранимой процедуры.

1. Нажмите **... Дополнительно** на панели инструментов нажмите кнопку **создать набор данных** и выберите **Azure SQL**. **Новый набор данных** на панели команд и выберите **Azure SQL**.

    ![представление в виде дерева со связанной службой 2](media/data-factory-stored-proc-activity/new-dataset.png)
2. Скопируйте и вставьте следующий скрипт JSON в редактор JSON.

    ```JSON
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "sampletable"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
3. Чтобы развернуть набор данных, нажмите кнопку **Развернуть** на панели команд. Набор данных должен отображаться в иерархическом представлении.

    ![иерархическое представление со связанными службами](media/data-factory-stored-proc-activity/tree-view-2.png)

### <a name="create-a-pipeline-with-sqlserverstoredprocedure-activity"></a>Создание конвейера с помощью действия SqlServerStoredProcedure
Теперь создадим конвейер с помощью действия хранимой процедуры.

Обратите внимание на следующие свойства:

- Свойству **type** присвоено значение **SqlServerStoredProcedure**.
- Параметру **storedProcedureName** в свойствах типа присвоено значение **usp_sample** (имя хранимой процедуры).
- Раздел **storedProcedureParameters** содержит один параметр с именем **DataTime**. Имя и регистр параметра в формате JSON должны совпадать с именем и регистром параметра в определении хранимой процедуры. Если для параметра необходимо передать значение null, используйте синтаксис `"param1": null` (все символы в нижнем регистре).

1. Нажмите **... На панели** команд щелкните **создать конвейер**.
2. Скопируйте и вставьте следующий фрагмент JSON.

    ```JSON
    {
        "name": "SprocActivitySamplePipeline",
        "properties": {
            "activities": [
                {
                    "type": "SqlServerStoredProcedure",
                    "typeProperties": {
                        "storedProcedureName": "usp_sample",
                        "storedProcedureParameters": {
                            "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                        }
                    },
                    "outputs": [
                        {
                            "name": "sprocsampleout"
                        }
                    ],
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                    },
                    "name": "SprocActivitySample"
                }
            ],
            "start": "2017-04-02T00:00:00Z",
            "end": "2017-04-02T05:00:00Z",
            "isPaused": false
        }
    }
    ```
3. Чтобы развернуть конвейер, щелкните **Развернуть** на панели инструментов.

### <a name="monitor-the-pipeline"></a>Мониторинг конвейера
1. Щелкните **X**, чтобы закрыть колонки редактора фабрики данных и вернуться в колонку фабрики данных. Затем щелкните **Схема**.

    ![Плитка диаграммы 1](media/data-factory-stored-proc-activity/data-factory-diagram-tile.png)
2. В **представлении схемы** вы увидите все конвейеры и наборы данных, используемые в этом руководстве.

    ![Плитка диаграммы 2](media/data-factory-stored-proc-activity/data-factory-diagram-view.png)
3. В представлении схемы дважды щелкните набор данных `sprocsampleout`. Отобразятся срезы в состоянии "Готово". Должно отобразиться пять срезов, так как из JSON срез создается каждый час в периоде между временем начала и окончания.

    ![Плитка диаграммы 3](media/data-factory-stored-proc-activity/data-factory-slices.png)
4. Когда срез находится в состоянии **Готово** , выполните запрос к `select * from sampletable` базе данных, чтобы убедиться, что данные были вставлены в таблицу хранимой процедурой.

   ![Выходные данные](./media/data-factory-stored-proc-activity/output.png)

   В разделе [Мониторинг конвейера](data-factory-monitor-manage-pipelines.md) представлены подробные сведения о мониторинге конвейеров фабрики данных Azure.

## <a name="specify-an-input-dataset"></a>Указание набора входных данных
В этом руководстве действие хранимой процедуры не содержит наборы входных данных. При указании набора входных данных действие хранимой процедуры не выполняется, пока не стает доступен срез набора входных данных (в состоянии "Готово"). Набор данных может быть внешним (который не был создан с помощью другого действия в одном конвейере) или внутренним, созданным с помощью вышестоящего действия (действие, которое выполняется перед этим действием). Для действия хранимой процедуры можно указать несколько наборов входных данных. В таком случае действие хранимой процедуры выполняется только в том случае, когда доступны все срезы набора входных данных (в состоянии "Готово"). Входной набор данных не может потребляться в хранимой процедуре в качестве параметра. Он используется только для проверки зависимости перед запуском действия хранимой процедуры.

## <a name="chaining-with-other-activities"></a>Объединение с другими действиями
Если необходимо объединить вышестоящее действие с этим действием, укажите выходные данные вышестоящего действия в качестве входных данных этого действия. При этом действие хранимой процедуры не выполняется, пока не завершится вышестоящее действие и набор выходных данных вышестоящего действия не станет доступен (в состоянии "Готово"). В качестве наборов входных данных действия хранимой процедуры можно указать наборы выходных данных нескольких вышестоящих действий. В таком случае действие хранимой процедуры выполняется только в том случае, когда доступны все срезы набора входных данных.

В следующем примере выходными данными действия копирования является набор OutputDataset, который представляет собой входные данные действия хранимой процедуры. Таким образом, действие хранимой процедуры не выполняется, пока не завершится действие копирования и срез набора данных OutputDataset не станет доступен (в состоянии "Готово"). При указании нескольких наборов входных данных действие хранимой процедуры не выполняется, пока не станет доступен срез набора входных данных (в состоянии "Готово"). Наборы входных данных нельзя использовать непосредственно в качестве параметров действия хранимой процедуры

Дополнительные сведения о цепочках действий см. в разделе [Несколько действий в конвейере](data-factory-create-pipelines.md#multiple-activities-in-a-pipeline).

```json
{
    "name": "ADFTutorialPipeline",
    "properties": {
        "description": "Copy data from a blob to blob",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [ { "name": "InputDataset" } ],
                "outputs": [ { "name": "OutputDataset" } ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst"
                },
                "name": "CopyFromBlobToSQL"
            },
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "SPSproc"
                },
                "inputs": [ { "name": "OutputDataset" } ],
                "outputs": [ { "name": "SQLOutputDataset" } ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "RunStoredProcedure"
            }
        ],
        "start": "2017-04-12T00:00:00Z",
        "end": "2017-04-13T00:00:00Z",
        "isPaused": false,
    }
}
```

Аналогичным образом, чтобы связать действие хранимой процедуры с **нижестоящими действиями** (действия, которые выполняются после завершения действия хранимой процедуры), укажите набор выходных данных действия хранимой процедуры в качестве входных данных нижестоящего действия в конвейере.

> [!IMPORTANT]
> При копировании данных в базу данных SQL Azure или SQL Server можно настроить класс **SqlSink** в действии копирования для вызова хранимой процедуры с помощью свойства **sqlWriterStoredProcedureName**. Дополнительные сведения см. в статье [Вызов хранимой процедуры из действия копирования в фабрике данных Azure](data-factory-invoke-stored-procedure-from-copy-activity.md). Дополнительные сведения о свойствах см. в следующих статьях о соединителях: [Перемещение данных в базу данных SQL Azure и из нее с помощью фабрики данных Azure](data-factory-azure-sql-connector.md#copy-activity-properties) и [Перемещение данных в базу данных SQL Server и обратно на локальных компьютерах и виртуальных машинах Azure IaaS с помощью фабрики данных Azure](data-factory-sqlserver-connector.md#copy-activity-properties).
> 
> При копировании данных из базы данных SQL Azure или SQL Server или Azure синапсе Analytics можно настроить **SqlSource** в действии копирования, чтобы вызвать хранимую процедуру для считывания данных из базы данных-источника с помощью свойства **sqlReaderStoredProcedureName** . Дополнительные сведения см. в следующих статьях о соединителях: [база данных SQL Azure](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties), [Azure синапсе Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties) .

## <a name="json-format"></a>Формат JSON
Ниже приведен формат JSON для определения действия хранимой процедуры:

```JSON
{
    "name": "SQLSPROCActivity",
    "description": "description",
    "type": "SqlServerStoredProcedure",
    "inputs": [ { "name": "inputtable" } ],
    "outputs": [ { "name": "outputtable" } ],
    "typeProperties":
    {
        "storedProcedureName": "<name of the stored procedure>",
        "storedProcedureParameters":
        {
            "param1": "param1Value"
            …
        }
    }
}
```

В следующей таблице описаны эти свойства JSON:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| name | Имя действия. |Да |
| description |Текст, описывающий, для чего используется действие |Нет |
| type | Нужно задать значение **SqlServerStoredProcedure** | Да |
| Ввод данных | Необязательный параметр. Если указать входной набор данных, он должен быть доступен (в состоянии "Готово") для выполнения действия хранимой процедуры. Входной набор данных не может потребляться в хранимой процедуре в качестве параметра. Он используется только для проверки зависимости перед запуском действия хранимой процедуры. |Нет |
| outputs | Для действия хранимой процедуры необходимо указать выходной набор данных. Выходной набор данных указывает **расписание** для действия хранимой процедуры (ежечасно, еженедельно, ежемесячно и т. д.). <br/><br/>Выходной набор данных должен использовать **связанную службу** , которая ссылается на базу данных SQL Azure или Azure синапсе Analytics, или на базу данных SQL Server, в которой должна выполняться хранимая процедура. <br/><br/>Выходной набор данных может использоваться для передачи результатов хранимой процедуры для дальнейшей обработки с помощью другого действия ([цепочки действий](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) в конвейере. Тем не менее фабрика данных не записывает выходные данные хранимой процедуры в этот набор данных автоматически. Выходные данные записывает сама хранимая процедура в таблицу SQL, на которую указывает выходной набор данных. <br/><br/>В некоторых случаях выходной набор данных может быть **фиктивным набором данных**. Он используется, только чтобы задать расписание выполнения действия хранимой процедуры. |Да |
| storedProcedureName |Укажите имя хранимой процедуры в базе данных SQL Azure, Azure синапсе Analytics или SQL Server, представленной связанной службой, которую использует выходная таблица. |Да |
| storedProcedureParameters |Указываемые значения для параметров хранимой процедуры. Если для параметра необходимо передать значение null, используйте синтаксис param1: null (все символы в нижнем регистре). Чтобы научиться использовать это свойство, см. пример ниже. |Нет |

## <a name="passing-a-static-value"></a>Передача статического значения
Теперь рассмотрим добавление в таблицу другого столбца с именем "Scenario", содержащего статическое значение с именем "образец документа".

![Пример данных 2](./media/data-factory-stored-proc-activity/sample-data-2.png)

**Таблица**

```SQL
CREATE TABLE dbo.sampletable2
(
    Id uniqueidentifier,
    datetimestamp nvarchar(127),
    scenario nvarchar(127)
)
GO

CREATE CLUSTERED INDEX ClusteredID ON dbo.sampletable2(Id);
```

**Хранимая процедура:**

```SQL
CREATE PROCEDURE usp_sample2 @DateTime nvarchar(127) , @Scenario nvarchar(127)

AS

BEGIN
    INSERT INTO [sampletable2]
    VALUES (newid(), @DateTime, @Scenario)
END
```

Теперь передайте параметр **сценария** и значение из действия хранимой процедуры. Раздел **typeProperties** в предыдущем примере выглядит, как в следующем фрагменте кода:

```JSON
"typeProperties":
{
    "storedProcedureName": "usp_sample",
    "storedProcedureParameters":
    {
        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)",
        "Scenario": "Document sample"
    }
}
```

**Набор данных фабрики данных:**

```JSON
{
    "name": "sprocsampleout2",
    "properties": {
        "published": false,
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "sampletable2"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Конвейер фабрики данных**

```JSON
{
    "name": "SprocActivitySamplePipeline2",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "usp_sample2",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)",
                        "Scenario": "Document sample"
                    }
                },
                "outputs": [
                    {
                        "name": "sprocsampleout2"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SprocActivitySample"
            }
        ],
        "start": "2016-10-02T00:00:00Z",
        "end": "2016-10-02T05:00:00Z"
    }
}
```
