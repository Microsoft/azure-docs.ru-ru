---
title: Перемещение данных в SQL Server и из них
description: Узнайте, как с помощью фабрики данных Azure перемещать данные в базу данных SQL Server и обратно на локальных компьютерах и виртуальных машинах Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
robots: noindex
ms.openlocfilehash: fbd1e1d652db3bbd91344ea828278d057baeb060
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100368818"
---
# <a name="move-data-to-and-from-sql-server-using-azure-data-factory"></a>Перемещение данных в SQL Server с помощью фабрики данных Azure и обратно

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-sqlserver-connector.md)
> * [Версия 2 (текущая)](../connector-sql-server.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе SQL Server в службе "Фабрика данных Azure" версии 2](../connector-sql-server.md).

В этой статье объясняется, как с помощью действия копирования в фабрике данных Azure перемещать данные в базу данных SQL Server или из нее. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="supported-scenarios"></a>Поддерживаемые сценарии
Данные можно скопировать **из базы данных SQL Server** в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Данные можно скопировать **в базу данных SQL Server** из следующих хранилищ данных:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="supported-sql-server-versions"></a>Поддерживаемые версии SQL Server
Этот соединитель SQL Server поддерживает копирование данных в экземпляры следующих версий, размещенные в локальной среде или в Azure IaaS, и обратно с использованием проверки подлинности SQL и Windows: SQL Server 2016, SQL Server 2014, SQL Server 2012, SQL Server 2008 R2, SQL Server 2008, SQL Server 2005.

## <a name="enabling-connectivity"></a>Включение соединения
Где бы ни размещалась система SQL Server — локально или на виртуальных машинах Azure IaaS (инфраструктура как услуга), — основные понятия и действия, необходимые для соединения с ней, одинаковы. В обоих случаях для подключения необходимо использовать шлюз управления данными.

В статье [Перемещение данных между локальными и облачными ресурсами](data-factory-move-data-between-onprem-and-cloud.md) приведены сведения о шлюзе управления данными и пошаговые инструкции по его настройке. Настройка экземпляра шлюза необходима для подключения к SQL Server.

Вы можете установить шлюз на тот же локальный компьютер или экземпляр облачной виртуальной машины, на котором установлена система SQL Server, однако для более эффективной работы мы рекомендуем устанавливать их на разные компьютеры. Размещение шлюза и SQL Server на разных компьютерах уменьшает конкуренцию за ресурсы.

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, которое перемещает данные в SQL Server базу данных или из нее с помощью различных инструментов и интерфейсов API.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров.
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, при копировании данных из базы данных SQL Server в хранилище BLOB-объектов Azure создайте две связанные службы, чтобы связать базу данных SQL Server и учетную запись хранения Azure с фабрикой данных. Сведения о свойствах связанной службы, относящихся к базе данных SQL Server, см. в разделе [Свойства связанной службы](#linked-service-properties).
3. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом в последнем шаге, создайте набор данных, чтобы указать таблицу SQL в базе данных SQL Server, содержащую входные данные. И создайте другой набор данных, чтобы указать контейнер BLOB-объектов и папку, содержащую данные, скопированные из базы данных SQL Server. Сведения о свойствах набора данных, относящихся к базе данных SQL Server, см. в разделе [Свойства набора данных](#dataset-properties).
4. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше SqlSource используется как источник, а BlobSink — как приемник для действия копирования. Аналогично, при копировании из хранилища BLOB-объектов Azure в базу данных SQL Server в действии копирования используются BlobSource и SqlSink. Сведения о свойствах действия копирования, относящихся к базе данных SQL Server, см. в разделе [Свойства действия копирования](#copy-activity-properties). Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе об источнике данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных в SQL Server базу данных или из нее, см. в разделе [примеры JSON](#json-examples-for-copying-data-from-and-to-sql-server) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для базы данных SQL Server.

## <a name="linked-service-properties"></a>Свойства связанной службы
Создайте связанную службу типа **OnPremisesSqlServer** , чтобы связать базу данных SQL Server с фабрикой данных. В следующей таблице содержится описание элементов JSON, которые относятся к связанной службе SQL Server.

В следующей таблице содержится описание элементов JSON, которые относятся к связанной службе SQL Server.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Свойству type необходимо присвоить значение **OnPremisesSqlServer**. |Да |
| connectionString |Укажите сведения о параметре connectionString, необходимые для подключения к базе данных SQL Server с помощью проверки подлинности SQL или Windows. |Да |
| gatewayName |Имя шлюза, который служба фабрики данных должна использовать для подключения к базе данных SQL Server. |Да |
| username |При использовании проверки подлинности Windows укажите имя пользователя. Например, **domainname\\username**. |Нет |
| password |Введите пароль для учетной записи пользователя, указанной для выбранного имени пользователя. |Нет |

Вы можете зашифровать учетные данные с помощью командлета **New-аздатафакторенкриптвалуе** и использовать их в строке подключения, как показано в следующем примере (свойство **EncryptedCredential** ):

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```

### <a name="samples"></a>Примеры
**JSON для использования проверки подлинности SQL**

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties":
    {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
**JSON для использования проверки подлинности Windows**

Шлюз Управление данными будет олицетворять указанную учетную запись пользователя для подключения к базе данных SQL Server.

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties":
    {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
В примерах использовался набор данных типа **SqlServerTable** для представления таблицы в базе данных SQL Server.

Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы JSON набора данных, такие как структура, доступность и политика, одинаковы для всех типов наборов данных (SQL Server, большой двоичный объект Azure, таблица Azure и т. д.).

Раздел typeProperties во всех типах наборов данных разный. В нем содержатся сведения о расположении данных в хранилище данных. Раздел **typeProperties** для набора данных с типом **SqlServerTable** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы или представления в экземпляре базы данных SQL Server, на который ссылается связанная служба. |Да |

## <a name="copy-activity-properties"></a>Свойства действия копирования
При перемещении данных из базы данных SQL Server в действии копирования задается тип источника **SqlSource**. Аналогично, при перемещении данных в базу данных SQL Server в действии копирования задается тип источника **SqlSink**. Этот раздел содержит список свойств, поддерживаемых типами SqlSource и SqlSink.

Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (такие как имя, описание, входные и выходные таблицы, политики и т. д.) доступны для всех типов действий.

> [!NOTE]
> Действие копирования принимает только один набор входных данных и возвращает только один набор выходных.

В свою очередь свойства, доступные в разделе typeProperties действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

### <a name="sqlsource"></a>SqlSource
Когда источник в действии копирования относится к типу **SqlSource**, в разделе **typeProperties** доступны указанные ниже свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| sqlReaderQuery |Используйте пользовательский запрос для чтения данных. |Строка запроса SQL. Например, select * from MyTable. Может ссылаться на несколько таблиц из базы данных, на которую ссылается входной набор данных. Если не указано другое, выполняется инструкция SQL select from MyTable. |нет |
| sqlReaderStoredProcedureName |Имя хранимой процедуры, которая считывает данные из исходной таблицы. |Имя хранимой процедуры. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры. |Пары имен и значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |

Если для SqlSource указано **sqlReaderQuery** , то действие копирования выполняет этот запрос для источника базы данных SQL Server с целью получения данных.

Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры).

Если не указать sqlReaderQuery или sqlReaderStoredProcedureName, то для построения запроса select к базе данных SQL Server будут использованы столбцы, определенные в разделе структуры. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.

> [!NOTE]
> При использовании **sqlReaderStoredProcedureName** по-прежнему необходимо указать значение свойства **tableName** в наборе данных JSON. Хотя какие-либо проверки этой таблицы отсутствуют.

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


## <a name="json-examples-for-copying-data-from-and-to-sql-server"></a>Примеры JSON для копирования данных в SQL Server и обратно
Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В следующих примерах показано, как копировать данные в базу данных SQL Server и хранилище BLOB-объектов Azure и обратно. Тем не менее данные можно копировать **непосредственно** из любых источников в любой указанный [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемник. Это делается с помощью действия копирования в фабрике данных Azure.

## <a name="example-copy-data-from-sql-server-to-azure-blob"></a>Пример. Копирование данных из базы данных SQL Server в хранилище BLOB-объектов Azure
В примере ниже используется следующее:

1. Связанная служба типа [OnPremisesSqlServer](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [SqlServerTable](#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [SqlSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные временного ряда каждый час копируются из таблицы SQL Server в большой двоичный объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

Сначала настройте шлюз управления данными. Инструкции приведены в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md) .

**Связанная служба SQL Server**
```json
{
  "Name": "SqlServerLinkedService",
  "properties": {
    "type": "OnPremisesSqlServer",
    "typeProperties": {
      "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
      "gatewayName": "<gatewayname>"
    }
  }
}
```
**Связанная служба хранилища BLOB-объектов Azure**

```json
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
**Входной набор данных SQL Server**

В примере предполагается, что вы создали таблицу MyTable в SQL Server и она содержит столбец с именем «timestampcolumn» для данных временных рядов. Можно выполнить запрос к нескольким таблицам в одной базе данных, используя один набор данных, но для typeProperty tableName набора данных нужно указать одну таблицу.

Если параметру external присвоить значение true, фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой службе.

```json
{
  "name": "SqlServerInput",
  "properties": {
    "type": "SqlServerTable",
    "linkedServiceName": "SqlServerLinkedService",
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
**Выходной набор данных BLOB-объекта Azure**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует эти входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **SqlSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный для свойства **SqlReaderQuery** , выбирает для копирования данные за последний час.

```json
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2016-06-01T18:00:00",
    "end":"2016-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "SqlServertoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": " SqlServerInput"
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
В этом примере для свойства SqlSource указано **sqlReaderQuery** . Действие копирования выполняет этот запрос к источнику базы данных SQL Server с целью получения данных. Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры). Свойство sqlReaderQuery может ссылаться на несколько таблиц из базы данных, на которую ссылается входной набор данных. Он не ограничивается таблицей, заданной свойством typeProperty в качестве параметра tableName набора данных.

Если не указать sqlReaderQuery или sqlReaderStoredProcedureName, то для построения запроса select к базе данных SQL Server будут использованы столбцы, определенные в разделе структуры. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.

Список свойств, поддерживаемых SqlSource и BlobSink, см. в разделах [SqlSource](#sqlsource) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

## <a name="example-copy-data-from-azure-blob-to-sql-server"></a>Пример. Копирование данных из BLOB-объекта Azure в базу данных SQL Server
В примере ниже используется следующее:

1. Связанная служба типа [OnPremisesSqlServer](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [SqlServerTable](data-factory-sqlserver-connector.md#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) и SqlSink.

В этом примере данные временного ряда каждый час копируются из большого двоичного объекта Azure в таблицу SQL Server. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба SQL Server**

```json
{
  "Name": "SqlServerLinkedService",
  "properties": {
    "type": "OnPremisesSqlServer",
    "typeProperties": {
      "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
      "gatewayName": "<gatewayname>"
    }
  }
}
```
**Связанная служба хранилища BLOB-объектов Azure**

```json
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
**Входной набор данных большого двоичного объекта Azure**

Данные берутся из нового BLOB-объекта каждый час (frequency: hour, interval: 1). Путь к папке с BLOB-объектом и имя файла вычисляются динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц и день начала, а в имени файла — час начала. параметр "External": "true" информирует службу фабрики данных о том, что набор данных является внешним по отношению к фабрике данных и не создается действием в фабрике данных.

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
**Выходной набор данных SQL Server**

В этом примере данные копируются в таблицу с именем MyTable в SQL Server. Создайте в базе данных SQL Server таблицу с тем же количеством столбцов, которое должно быть в CSV-файле большого двоичного объекта. Новые строки добавляются в таблицу каждый час.

```json
{
  "name": "SqlServerOutput",
  "properties": {
    "type": "SqlServerTable",
    "linkedServiceName": "SqlServerLinkedService",
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
**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует эти входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **BlobSource**, а для типа **sink** — значение **SqlSink**.

```json
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
            "name": " SqlServerOutput "
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

## <a name="troubleshooting-connection-issues"></a>Устранение неполадок с подключением
1. Настройте SQL Server для приема удаленных подключений. Запустите **SQL Server Management Studio**, щелкните правой кнопкой мыши свой **сервер** и выберите пункт **Свойства**. Выберите в списке пункт **Подключения** и установите флажок **Allow remote connections to the server** (Разрешить удаленные подключения к этому серверу).

    ![Включение удаленных подключений](./media/data-factory-sqlserver-connector/AllowRemoteConnections.png)

    Подробные инструкции см. в статье [Настройка параметра конфигурации сервера remote access](/sql/database-engine/configure-windows/configure-the-remote-access-server-configuration-option).
2. Запустите **диспетчер конфигурации SQL Server**. Разверните узел **Сетевая конфигурация SQL Server** для нужного экземпляра и выберите пункт **Protocols for MSSQLSERVER** (Протоколы для MSSQLSERVER). Вы должны увидеть протоколы на панели справа. Включите TCP/TP, щелкнув правой кнопкой мыши имя протокола **TCP/IP** и выбрав пункт **Включить**.

    ![Включение TCP/IP](./media/data-factory-sqlserver-connector/EnableTCPProptocol.png)

    Подробные сведения и альтернативные способы включения протокола TCP/IP см. в статье [Включение или отключение сетевого протокола сервера](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol).
3. В этом же окне дважды щелкните **TCP/IP**, чтобы открыть окно **TCP/IP Properties** (Свойства TCP/IP).
4. Перейдите на вкладку **IP-адреса** . Прокрутите вниз, чтобы увидеть раздел **IPAll** . Запишите **TCP-порт**(по умолчанию — **1433**).
5. Создайте на компьютере **правило брандмауэра Windows** , чтобы разрешить входящий трафик через этот порт.
6. **Проверьте подключение**. Например, \<machine\>\<domain\>.corp\<company\>.com,1433.

   > [!IMPORTANT]
   > 
   > Дополнительные сведения см. в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md).
   > 
   > Советы по устранению неполадок, связанных со шлюзом или подключением, см. в разделе [Устранение неполадок в работе шлюза](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).


## <a name="identity-columns-in-the-target-database"></a>Столбцы идентификаторов в целевой базе данных
В этом разделе приведен пример копирования данных из исходной таблицы без столбца идентификаторов в целевую таблицу со столбцом идентификаторов.

**Исходная таблица:**

```sql
create table dbo.SourceTbl
(
    name varchar(100),
    age int
)
```
**Целевая таблица:**

```sql
create table dbo.TargetTbl
(
    identifier int identity(1,1),
    name varchar(100),
    age int
)
```

Обратите внимание, что в целевой таблице есть столбец идентификаторов.

**Определение JSON исходного набора данных**

```json
{
    "name": "SampleSource",
    "properties": {
        "published": false,
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

```json
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "published": false,
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

## <a name="type-mapping-for-sql-server"></a>Сопоставление типов SQL Server
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , действие копирования выполняет автоматическое преобразование типов из исходных типов в типы приемников, используя следующий 2-этапный подход:

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

Когда данные перемещаются в базу данных SQL Server и обратно, для преобразования типа SQL в тип .NET и наоборот используются следующие сопоставления.

Сопоставление аналогично сопоставлению типов данных SQL Server для ADO.NET.

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

## <a name="mapping-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника
Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="repeatable-copy"></a>Повторяющаяся операция копирования
При копировании данных в базу данных SQL Server действие копирования по умолчанию добавляет данные в таблицу приемника. Чтобы вместо этого выполнить операцию UPSERT, изучите раздел [Повторяемая запись в SqlSink](data-factory-repeatable-copy.md#repeatable-write-to-sqlsink).

При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных политику повтора, чтобы при сбое срез выполнялся повторно. При повторном выполнении среза в любом случае необходимо убедиться в том, что считываются те же данные, независимо от того, сколько раз выполняется срез. См. раздел [повторяемые операции чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.