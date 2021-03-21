---
title: Копирование данных в базу данных SQL Azure и из нее
description: Узнайте, как копировать данные в базу данных SQL Azure и из нее с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 738b875a273faddd20a67be0f6feb90825f66c9f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100370518"
---
# <a name="copy-data-to-and-from-azure-sql-database-using-azure-data-factory"></a>Копирование данных в базу данных SQL Azure и из нее с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-azure-sql-connector.md)
> * [Версия 2 (текущая)](../connector-azure-sql-database.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе Базы данных SQL Azure в службе "Фабрика данных Azure" версии 2](../connector-azure-sql-database.md).

В этой статье рассказывается, как с помощью действия копирования в Фабрике данных Azure перемещать данные в базу данных SQL Azure и обратно. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии
Данные можно скопировать **из базы данных SQL Azure** в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Данные можно скопировать **в базу данных SQL Azure** из следующих хранилищ данных:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="supported-authentication-type"></a>Поддерживаемый тип проверки подлинности
Соединитель Базы данных SQL Azure поддерживает обычную проверку подлинности.

## <a name="getting-started"></a>Начало работы
Можно создать конвейер с действием копирования, которое перемещает данные из базы данных SQL Azure или в нее с помощью различных инструментов и API-интерфейсов.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров.
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, если вы копируете данные из хранилища BLOB-объектов Azure в базу данных SQL Azure, создайте две связанные службы, связывающие учетную запись хранения Azure и базу данных SQL Azure с фабрикой данных. Сведения о свойствах связанной службы, относящихся к базе данных SQL Azure, см. в разделе [Свойства связанной службы](#linked-service-properties).
3. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом на последнем этапе, создайте набор данных, чтобы указать контейнер BLOB-объектов и папку, содержащую входные данные. Кроме того, вы создаете другой набор данных, чтобы указать таблицу SQL в базе данных SQL Azure, которая содержит данные, скопированные из хранилища BLOB-объектов. Сведения о свойствах набора данных, относящихся к Azure Data Lake Store, см. в разделе [Свойства набора данных](#dataset-properties).
4. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше BlobSource используется как источник, а SqlSink — как приемник для действия копирования. Аналогично, при копировании из Базы данных SQL Azure в хранилище BLOB-объектов Azure в действии копирования используются SqlSource и BlobSink. Сведения о свойствах действия копирования, относящихся к базе данных SQL Azure, см. в разделе [Свойства действия копирования](#copy-activity-properties). Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе об источнике данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных из базы данных SQL Azure, см. в разделе [Примеры определений JSON](#json-examples-for-copying-data-to-and-from-sql-database) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для базы данных SQL Azure.

## <a name="linked-service-properties"></a>Свойства связанной службы
Связанная служба SQL Azure связывает базу данных SQL Azure с фабрикой данных. В таблице ниже приведено описание элементов JSON, которые относятся к связанной службе SQL Azure.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Для свойства Type необходимо задать значение **AzureSqlDatabase** . |Да |
| connectionString |В свойстве connectionString указываются сведения, необходимые для подключения к экземпляру Базы данных SQL Azure. Поддерживается только обычная проверка подлинности. |Да |

> [!IMPORTANT]
> [Брандмауэр базы данных SQL Azure](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure) необходимо настроить, чтобы [разрешить службам Azure получать доступ к серверу](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure). Кроме того, при копировании данных в Базу данных SQL Azure из внешнего по отношению к Azure источника, включая локальные источники данных, с помощью шлюза фабрики данных необходимо настроить соответствующий диапазон IP-адресов для компьютера, который отправляет данные в Базу данных SQL Azure.

## <a name="dataset-properties"></a>Свойства набора данных
Чтобы указать набор данных для представления входных или выходных данных в базе данных SQL Azure, задайте для свойства Type набора данных значение: **AzureSqlTable**. Для свойства **linkedServiceName** набора данных задайте имя связанной службы Azure SQL.

Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел typeProperties во всех типах наборов данных разный. В нем содержатся сведения о расположении данных в хранилище данных. Раздел **typeProperties** набора данных с типом **AzureSqlTable** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы или представления в экземпляре Базы данных SQL Azure, на которое ссылается связанная служба. |Да |

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

> [!NOTE]
> Действие копирования принимает только один набор входных данных и возвращает только один набор выходных.

В свою очередь свойства, доступные в разделе **typeProperties** действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

При перемещении данных из базы данных SQL Azure задайте тип источника **SqlSource** в действии копирования. Аналогично, если вы перемещаете данные в базу данных SQL Azure, задайте тип приемника **SqlSink** в действии копирования. Этот раздел содержит список свойств, поддерживаемых типами SqlSource и SqlSink.

### <a name="sqlsource"></a>SqlSource
Для действия копирования, если источник относится к типу **SqlSource**, в разделе **typeProperties** доступны указанные ниже свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| sqlReaderQuery |Используйте пользовательский запрос для чтения данных. |Строка запроса SQL. Например, `select * from MyTable`. |нет |
| sqlReaderStoredProcedureName |Имя хранимой процедуры, которая считывает данные из исходной таблицы. |Имя хранимой процедуры. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры. |Пары имен и значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |

Если для SqlSource указано **sqlReaderQuery**, то действие копирования выполняет этот запрос для источника базы данных SQL Azure для получения данных. Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры).

Если не указать sqlReaderQuery или sqlReaderStoredProcedureName, то для построения запроса (`select column1, column2 from mytable`) к базе данных SQL Azure будут использованы столбцы, определенные в разделе структуры набора данных JSON. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.

> [!NOTE]
> При использовании **sqlReaderStoredProcedureName** по-прежнему необходимо указать значение свойства **tableName** в наборе данных JSON. Хотя какие-либо проверки этой таблицы отсутствуют.
>
>

### <a name="sqlsource-example"></a>Пример SqlSource

```JSON
"source": {
    "type": "SqlSource",
    "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
    "storedProcedureParameters": {
        "stringData": { "value": "str3" },
        "identifier": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
    }
}
```

**Определение хранимой процедуры:**

```SQL
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sqlsink"></a>SqlSink
**SqlSink** поддерживает указанные ниже свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| writeBatchTimeout |Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия. |Интервал времени<br/><br/> Пример "00:30:00" (30 минут). |Нет |
| writeBatchSize |Вставляет данные в таблицу SQL, когда размер буфера достигает значения writeBatchSize. |Целое число (количество строк) |Нет (значение по умолчанию — 10 000). |
| sqlWriterCleanupScript |Укажите запрос на выполнение действия копирования, позволяющий убедиться в том, что данные конкретного среза очищены. Дополнительные сведения см. в разделе [Повторяющаяся операция копирования](#repeatable-copy). |Инструкция запроса. |Нет |
| sliceIdentifierColumnName |Укажите имя столбца, в которое действие копирования добавляет автоматически созданный идентификатор среза. Этот идентификатор используется для очистки данных конкретного среза при повторном запуске. Дополнительные сведения см. в разделе [Повторяющаяся операция копирования](#repeatable-copy). |Имя столбца с типом данных binary(32). |Нет |
| sqlWriterStoredProcedureName |Имя хранимой процедуры, в которой определяется, как применить исходные данные в целевой таблице. Например, можно определить выполнение операций upsert или преобразований с помощью вашей собственной бизнес-логики. <br/><br/>Обратите внимание, что эта хранимая процедура будет **вызываться для каждого пакета**. Чтобы однократно выполнить операцию, в которой не используются исходные данные, например удаление или усечение, примените свойство `sqlWriterCleanupScript`. |Имя хранимой процедуры. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры. |Пары имен и значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |
| sqlWriterTableType |Укажите имя типа таблицы для использования в хранимой процедуре. Действие копирования делает перемещаемые данные доступными во временной таблице этого типа. Код хранимой процедуры затем можно использовать для объединения копируемых и существующих данных. |Имя типа таблицы. |Нет |

#### <a name="sqlsink-example"></a>Пример SqlSink

```JSON
"sink": {
    "type": "SqlSink",
    "writeBatchSize": 1000000,
    "writeBatchTimeout": "00:05:00",
    "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
    "sqlWriterTableType": "CopyTestTableType",
    "storedProcedureParameters": {
        "identifier": { "value": "1", "type": "Int" },
        "stringData": { "value": "str1" },
        "decimalData": { "value": "1", "type": "Decimal" }
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-sql-database"></a>Примеры JSON для копирования данных в базу данных SQL и обратно
Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В них показано, как копировать данные в Базу данных SQL Azure и хранилище BLOB-объектов Azure и обратно. Тем не менее данные можно копировать **непосредственно** из любых источников в любой указанный [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемник. Это делается с помощью действия копирования в фабрике данных Azure.

### <a name="example-copy-data-from-azure-sql-database-to-azure-blob"></a>Пример. Копирование данных из базы данных SQL Azure в BLOB-объект Azure
Образец определяет следующие сущности фабрики данных.

1. Связанная служба типа [AzureSqlDatabase](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [AzureSqlTable](#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [SqlSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные временных рядов копируются (каждый час, каждый день и т. д.) из таблицы в базе данных SQL Azure в большой двоичный объект каждый час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба базы данных SQL Azure:**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
Список свойств, поддерживаемых этой связанной службой, приведен в разделе "Связанная служба SQL Azure".

**Связанная служба хранилища BLOB-объектов Azure**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Список свойств, поддерживаемых этой связанной службой, приведен в разделе [Большой двоичный объект Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service).


**Входной набор данных SQL Azure**

В примере предполагается, что вы создали таблицу MyTable в SQL Azure и она содержит столбец с именем «timestampcolumn» для данных временных рядов.

Если параметру External присвоить значение true, служба фабрики данных Azure будет иметь внешний набор данных, который является внешним по отношению к фабрике данных и не будет создан действием в фабрике данных.

```JSON
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

Список свойств, поддерживаемых этим типом набора данных, приведен в разделе "Свойства типа «Набор данных Azure SQL»".

**Выходной набор данных большого двоичного объекта Azure:**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
Список свойств, поддерживаемых этим типом набора данных, приведен в разделе [Свойства типа «Набор данных большого двоичного объекта Azure»](data-factory-azure-blob-connector.md#dataset-properties) .

**Действие копирования в конвейере с базой данных SQL в качестве источника и BLOB-объектом в качестве приемника:**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **SqlSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный для свойства **SqlReaderQuery** , выбирает для копирования данные за последний час.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "AzureSQLtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSQLInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
В этом примере для свойства SqlSource указано **sqlReaderQuery** . Действие копирования выполняет этот запрос для источника Базы данных SQL Azure для получения данных. Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры).

Если не указать sqlReaderQuery или sqlReaderStoredProcedureName, то для построения запроса к базе данных SQL Azure будут использованы столбцы, определенные в разделе структуры набора данных JSON. Например, `select column1, column2 from mytable`. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.

Список свойств, поддерживаемых SqlSource и BlobSink, см. в разделах [SqlSource](#sqlsource) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

### <a name="example-copy-data-from-azure-blob-to-azure-sql-database"></a>Пример. Копирование данных из BLOB-объекта Azure в базу данных SQL Azure
Образец определяет следующие сущности фабрики данных.

1. Связанная служба типа [AzureSqlDatabase](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureSqlTable](#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) и [SqlSink](#copy-activity-properties).

В этом примере данные временных рядов (ежечасно, ежедневно и т. д.) копируются из большого двоичного объекта Azure в таблицу в базе данных SQL Azure каждый час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба SQL Azure:**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
Список свойств, поддерживаемых этой связанной службой, приведен в разделе "Связанная служба SQL Azure".

**Связанная служба хранилища BLOB-объектов Azure**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Список свойств, поддерживаемых этой связанной службой, приведен в разделе [Большой двоичный объект Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service).


**Входной набор данных большого двоичного объекта Azure:**

Данные берутся из нового BLOB-объекта каждый час (frequency: hour, interval: 1). Путь к папке с BLOB-объектом и имя файла вычисляются динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц и день начала, а в имени файла — час начала. параметр "External": "true" информирует службу фабрики данных о том, что эта таблица является внешней по отношению к фабрике данных и не создается действием в фабрике данных.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
Список свойств, поддерживаемых этим типом набора данных, приведен в разделе [Свойства типа «Набор данных большого двоичного объекта Azure»](data-factory-azure-blob-connector.md#dataset-properties) .

**Выходной набор данных Базы данных SQL Azure:**

В примере данные копируются в таблицу с именем MyTable в SQL Azure. Создайте таблицу в базе данных SQL Azure с тем количеством столбцов, которое должно быть в CSV-файле большого двоичного объекта. Новые строки добавляются в таблицу каждый час.

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
Список свойств, поддерживаемых этим типом набора данных, приведен в разделе "Свойства типа «Набор данных Azure SQL»".

**Действие копирования в конвейере с BLOB-объектом в качестве источника и базой данных SQL в качестве приемника:**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **BlobSource**, а для типа **sink** — значение **SqlSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource",
            "blobColumnSeparators": ","
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
Список свойств, поддерживаемых SqlSink и BlobSource, см. в разделах [SqlSink](#sqlsink) и [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties).

## <a name="identity-columns-in-the-target-database"></a>Столбцы идентификаторов в целевой базе данных
В этом разделе приведен пример копирования данных из исходной таблицы без столбца идентификаторов в целевую таблицу со столбцом идентификаторов.

**Исходная таблица:**

```SQL
create table dbo.SourceTbl
(
    name varchar(100),
    age int
)
```
**Целевая таблица:**

```SQL
create table dbo.TargetTbl
(
    identifier int identity(1,1),
    name varchar(100),
    age int
)
```
Обратите внимание, что в целевой таблице есть столбец идентификаторов.

**Определение JSON исходного набора данных**

```JSON
{
    "name": "SampleSource",
    "properties": {
        "type": " SqlServerTable",
        "linkedServiceName": "TestIdentitySQL",
        "typeProperties": {
            "tableName": "SourceTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```
**Определение JSON целевого набора данных**

```JSON
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "type": "AzureSqlTable",
        "linkedServiceName": "TestIdentitySQLSource",
        "typeProperties": {
            "tableName": "TargetTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }
}
```

Обратите внимание, что у исходной и целевой таблиц разные схемы (в целевой есть дополнительный столбец с идентификаторами). В этом сценарии необходимо указать свойство **Structure** в определении целевого набора данных, которое не включает столбец идентификаторов.

## <a name="invoke-stored-procedure-from-sql-sink"></a>Вызов хранимой процедуры из приемника SQL
Пример вызова хранимой процедуры из приемника SQL в действии копирования в конвейере приведен в статье [Invoke stored procedure from copy activity in Azure Data Factory](data-factory-invoke-stored-procedure-from-copy-activity.md) (Вызов хранимой процедуры из действия копирования в фабрике данных Azure).

## <a name="type-mapping-for-azure-sql-database"></a>Сопоставление типов для Базы данных SQL Azure
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , действие копирования выполняет автоматическое преобразование типов из исходных типов в типы приемников, используя следующий 2-этапный подход:

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

Когда данные перемещаются в базу данных SQL Azure и обратно, для преобразования типа SQL в тип .NET и наоборот используются следующие сопоставления. Сопоставление аналогично сопоставлению типов данных SQL Server для ADO.NET.

| Тип ядра СУБД SQL Server | Тип платформы .NET Framework |
| --- | --- |
| BIGINT |Int64 |
| binary |Byte[] |
| bit |Логическое |
| char |String, Char[] |
| Дата |Дата и время |
| Datetime |Дата и время |
| datetime2 |Дата и время |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| Изображение |Byte[] |
| INT |Int32 |
| money |Decimal |
| nchar |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| nvarchar |String, Char[] |
| real |Один |
| rowversion |Byte[] |
| smalldatetime |Дата и время |
| smallint |Int16 |
| smallmoney |Decimal |
| sql_variant |Object * |
| текст |String, Char[] |
| time |TimeSpan |
| TIMESTAMP |Byte[] |
| tinyint |Byte |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| xml |Xml |

## <a name="map-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника
Дополнительные сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="repeatable-copy"></a>Повторяющаяся операция копирования
При копировании данных в базу данных SQL Server действие копирования по умолчанию добавляет данные в таблицу приемника. Чтобы вместо этого выполнить UPSERT, см. статью [повторяемая запись в SqlSink](data-factory-repeatable-copy.md#repeatable-write-to-sqlsink) .

При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных политику повтора, чтобы при сбое срез выполнялся повторно. При повторном выполнении среза в любом случае необходимо убедиться в том, что считываются те же данные, независимо от того, сколько раз выполняется срез. См. раздел [повторяемые операции чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.