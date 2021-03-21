---
title: Перемещение данных в таблицу Azure или из нее
description: Узнайте, как переместить данные в таблицу SQL Azure и из нее с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: cc8e0272dc690ed541883ae943c118cbbe2f1854
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392125"
---
# <a name="move-data-to-and-from-azure-table-using-azure-data-factory"></a>Перемещение данных в таблицу SQL Azure и из нее с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-azure-table-connector.md)
> * [Версия 2 (текущая)](../connector-azure-table-storage.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе Хранилища таблиц Azure в службе "Фабрика данных Azure" версии 2](../connector-azure-table-storage.md).

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure перемещать данные в хранилище таблиц Azure и из него. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования. 

Данные можно скопировать из любого поддерживаемого в качестве источника хранилища данных в хранилище таблиц Azure или из хранилища таблиц Azure в любое поддерживаемое в качестве приемника хранилище данных. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, который перемещает данные в хранилище таблиц Azure или из него, с помощью различных инструментов и интерфейсов API.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) . 

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги: 

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. 
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. 

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных в хранилище таблиц Azure и из него, приведены в разделе [Примеры JSON](#json-examples) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к хранилищу таблиц Azure. 

## <a name="linked-service-properties"></a>Свойства связанной службы
Для связи хранилища BLOB-объектов Azure с фабрикой данных Azure можно использовать два типа связанных служб: **AzureStorage** и **AzureStorageSas**. Связанная служба хранилища Azure предоставляет фабрике данных глобальный доступ к хранилищу Azure, а связанная служба SAS хранилища Azure — ограниченный или привязанный ко времени доступ к хранилищу Azure. Других различий между этими связанными службами нет. Выберите связанную службу, соответствующую вашим задачам. Более подробно эти службы описаны в следующих разделах.

[!INCLUDE [data-factory-azure-storage-linked-services](../../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел typeProperties во всех типах наборов данных разный. В нем содержатся сведения о расположении данных в хранилище данных. Раздел **typeProperties** набора данных типа **AzureTable** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы в экземпляре базы данных таблиц Azure, на которое ссылается связанная служба. |Да. Если tableName указывается без azureTableSourceQuery, все записи из таблицы копируются в целевое расположение. Если azureTableSourceQuery указывается, записи из таблицы, удовлетворяющие запросу, копируются в целевое расположение. |

### <a name="schema-by-data-factory"></a>Схема фабрики данных
Для хранилищ данных без схемы, таких как таблица Azure, служба фабрики данных определяет схему одним из следующих способов.

1. Если указать структуру данных с помощью свойства **structure** в определении набора данных, служба фабрики данных считает эту структуру схемой. В этом случае, если строка не содержит значение столбца, ему присваивается значение NULL.
2. Если структура данных не указана с помощью свойства **structure** в определении набора данных, то фабрика данных определяет схему, используя первую строку данных. В этом случае, если первая строка не содержит полную схему, после операции копирования некоторые столбцы будут отсутствовать.

Таким образом для источников данных без схемы рекомендуется указывать структуру данных с помощью свойства **structure** .

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства, например имя, описание, входные и выходные таблицы, политики и т. д., доступны для всех типов действий.

С другой стороны, свойства, доступные в разделе typeProperties действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

**AzureTableSource** в разделе typeProperties поддерживает следующие свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| AzureTableSourceQuery |Используйте пользовательский запрос для чтения данных. |Строка запроса таблицы Azure. Примеры приведены в следующем разделе. |Нет. Если tableName указывается без azureTableSourceQuery, все записи из таблицы копируются в целевое расположение. Если azureTableSourceQuery указывается, записи из таблицы, удовлетворяющие запросу, копируются в целевое расположение. |
| azureTableSourceIgnoreTableNotFound |Указывает, игнорируются ли исключения таблицы. |TRUE<br/>FALSE |Нет |

### <a name="azuretablesourcequery-examples"></a>Примеры azureTableSourceQuery
Если столбец таблицы Azure имеет тип строки:

```JSON
azureTableSourceQuery": "$$Text.Format('PartitionKey ge \\'{0:yyyyMMddHH00_0000}\\' and PartitionKey le \\'{0:yyyyMMddHH00_9999}\\'', SliceStart)"
```

Если столбец таблицы Azure имеет тип даты и времени:

```JSON
"azureTableSourceQuery": "$$Text.Format('DeploymentEndTime gt datetime\\'{0:yyyy-MM-ddTHH:mm:ssZ}\\' and DeploymentEndTime le datetime\\'{1:yyyy-MM-ddTHH:mm:ssZ}\\'', SliceStart, SliceEnd)"
```

**AzureTableSink** поддерживает указанные ниже свойства в разделе typeProperties.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |Значение ключа раздела по умолчанию, которое может использоваться приемником. |Строковое значение. |Нет |
| azureTablePartitionKeyName |Укажите имя столбца, значения которого используются в качестве ключей секций. Если не указано, в качестве ключа раздела используется AzureTableDefaultPartitionKeyValue. |Имя столбца. |Нет |
| azureTableRowKeyName |Укажите имя столбца, значения которого используются в качестве ключа строки. Если имя не указано, используйте для каждой строки идентификатор GUID. |Имя столбца. |Нет |
| azureTableInsertType |Режим для вставки данных в таблицу Azure.<br/><br/>Это свойство контролирует, будут ли заменены или объединены значения в существующих строках в выходной таблице с совпадающими ключами секций и строк. <br/><br/>Чтобы узнать о действии этих параметров (merge и replace), ознакомьтесь со статьями [Insert or Merge Entity](/rest/api/storageservices/Insert-Or-Merge-Entity) (Вставка или слияние сущностей) и [Insert or Replace Entity](/rest/api/storageservices/Insert-Or-Replace-Entity) (Вставка или замена сущности). <br/><br>  Этот параметр применяется на уровне строки, а не таблицы, и ни один из параметров не приводит к удалению строк в выходной таблице, которые отсутствуют во входных данных. |merge (по умолчанию)<br/>replace |Нет |
| writeBatchSize |Вставка данных в таблицу Azure при достижении writeBatchSize или writeBatchTimeout. |Целое число (количество строк) |Нет (значение по умолчанию — 10 000). |
| writeBatchTimeout |Вставка данных в таблицу Azure при достижении writeBatchSize или writeBatchTimeout |Интервал времени<br/><br/>Пример: 00:20:00 (20 минут). |Нет (по умолчанию используется время ожидания, стандартное для клиента хранения — 90 секунд) |

### <a name="azuretablepartitionkeyname"></a>azureTablePartitionKeyName
Чтобы вы могли использовать целевой столбец как azureTablePartitionKeyName, необходимо сопоставить исходный столбец с целевым столбцом с помощью свойства JSON translator.

В следующем примере исходный столбец DivisionID сопоставляется с целевым столбцом: DivisionID.  

```JSON
"translator": {
    "type": "TabularTranslator",
    "columnMappings": "DivisionID: DivisionID, FirstName: FirstName, LastName: LastName"
}
```
DivisionID указывается в качестве ключа секции.

```JSON
"sink": {
    "type": "AzureTableSink",
    "azureTablePartitionKeyName": "DivisionID",
    "writeBatchSize": 100,
    "writeBatchTimeout": "01:00:00"
}
```
## <a name="json-examples"></a>Примеры JSON
Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В них показано, как копировать данные в хранилище таблиц Azure и хранилище BLOB-объектов Azure и обратно. Однако данные можно скопировать данные **непосредственно** из любых источников на любой из поддерживаемых приемников. Дополнительные сведения см. в разделе "Поддерживаемые хранилища данных и форматы" статьи [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

## <a name="example-copy-data-from-azure-table-to-azure-blob"></a>Пример. Копирование данных из таблицы Azure в большой двоичный объект Azure
В примере ниже используется следующее:

1. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) (используется для табличного & большого двоичного объекта).
2. Входной [набор данных](data-factory-create-datasets.md) типа [AzureTable](#dataset-properties).
3. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются AzureTableSource и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В примере ниже данные из стандартной секции таблицы Azure копируются в большой двоичный объект каждый час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба хранилища Azure:**

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
Фабрика данных Azure поддерживает два типа связанных служб хранилища Azure: **AzureStorage** и **AzureStorageSas**. Для первой необходимо указать строку подключения, содержащую ключ учетной записи, а для второй — URI подписанного URL-адреса. Дополнительные сведения см. в разделе [Связанные службы](#linked-service-properties).  

**Входной набор данных таблицы Azure**

В примере предполагается, что вы создали таблицу MyTable в таблице Azure.

Если параметру external присвоить значение true, то фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой фабрике данных.

```JSON
{
  "name": "AzureTableInput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
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

**Выходной набор данных большого двоичного объекта Azure:**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```JSON
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

**Действие копирования в конвейере с источником AzureTableSource и приемником BlobSink**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **AzureTableSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный в свойстве **AzureTableSourceQuery** , каждый час выбирает данные для копирования из стандартного раздела.

```JSON
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureTabletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureTableInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureTableSource",
                        "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
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

## <a name="example-copy-data-from-azure-blob-to-azure-table"></a>Пример. Копирование данных из большого двоичного объекта Azure в таблицу Azure
В примере ниже используется следующее:

1. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) (используется для таблиц и больших двоичных объектов).
2. Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
3. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureTable](#dataset-properties).
4. [Конвейер](data-factory-create-pipelines.md) с действием копирования, которое использует [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) и [AzureTableSink](#copy-activity-properties).

В этом примере данные временного ряда каждый час копируются из большого двоичного объекта Azure в таблицу Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба хранилища Azure (для таблиц и больших двоичных объектов Azure)**:

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

Фабрика данных Azure поддерживает два типа связанных служб хранилища Azure: **AzureStorage** и **AzureStorageSas**. Для первой необходимо указать строку подключения, содержащую ключ учетной записи, а для второй — URI подписанного URL-адреса. Дополнительные сведения см. в разделе [Связанные службы](#linked-service-properties).

**Входной набор данных большого двоичного объекта Azure:**

Данные берутся из нового BLOB-объекта каждый час (frequency: hour, interval: 1). Путь к папке с BLOB-объектом и имя файла вычисляются динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц и день начала, а в имени файла — час начала. параметр "External": "true" информирует службу фабрики данных о том, что набор данных является внешним по отношению к фабрике данных и не создается действием в фабрике данных.

```JSON
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

**Выходной набор данных таблицы Azure**

В примере данные копируются в таблицу с именем MyTable в таблице Azure. Создайте таблицу Azure с количеством столбцов, которое должно быть в CSV-файле большого двоичного объекта. Новые строки добавляются в таблицу каждый час.

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
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

**Действие копирования в конвейере с источником BlobSource и приемником AzureTableSink**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **BlobSource**, а для типа **sink** — значение **AzureTableSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoTable",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureTableOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "AzureTableSink",
            "writeBatchSize": 100,
            "writeBatchTimeout": "01:00:00"
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
## <a name="type-mapping-for-azure-table"></a>Сопоставление типов для таблиц Azure
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , действие копирования выполняет автоматическое преобразование типов из исходных типов в типы приемников с помощью следующего пошагового подхода.

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

При перемещении данных в таблицу Azure или из нее выполняется преобразование типов OData таблиц Azure в тип .NET (и наоборот) на основе указанных ниже [сопоставлений, определенных службой таблиц Azure](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model).

| Тип данных OData | Тип .NET | Сведения |
| --- | --- | --- |
| Edm.Binary |byte[] |Массив байтов размером до 64 КБ. |
| Edm.Boolean |bool |Значение типа Boolean. |
| Edm.DateTime |Дата и время |64-битное значение времени, выраженное в формате UTC. Допустимый диапазон времени начинается в 00:00 1 января 1601 года н. э. Англиканское летоисчисление, часовой пояс — UTC. Заканчивается диапазон 31 декабря 9999 года. |
| Edm.Double |double |64-битное значение с плавающей запятой. |
| Edm.Guid |Guid |128-битный идентификатор GUID. |
| Edm.Int32 |Int32 |32-битное целое число. |
| Edm.Int64 |Int64 |64-битное целое число. |
| Edm.String |Строка |Значение в кодировке UTF-16. Размер строкового значения не должен превышать 64 КБ. |

### <a name="type-conversion-sample"></a>Пример преобразования типов
В следующем примере данные копируются из BLOB-объекта Azure в таблицу Azure с преобразованием типов.

Предположим, набор данных большого двоичного объекта имеет формат CSV и содержит три столбца. Один из них — это столбец даты и времени, в котором данные указаны в пользовательском формате (используются французские сокращения дней недели).

Определите исходный набор данных большого двоичного объекта и укажите определения типов столбцов, как показано ниже.

```JSON
{
    "name": " AzureBlobInput",
    "properties":
    {
        "structure":
        [
            { "name": "userid", "type": "Int64"},
            { "name": "name", "type": "String"},
            { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
        ],
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
            "fileName":"myfile.csv",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "external": true,
        "availability":
        {
            "frequency": "Hour",
            "interval": 1,
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
Учитывая сопоставление типа OData таблицы Azure с типом .NET, таблицу в таблице Azure нужно определить посредством приведенной ниже схемы.

**Схема таблицы Azure**

| Имя столбца | Тип |
| --- | --- |
| userid |Edm.Int64 |
| name |Edm.String |
| lastlogindate |Edm.DateTime |

Далее нужно определить набор данных таблицы Azure, как показано ниже. Не нужно указывать раздел Structure с информацией о типе, так как сведения о типе уже указаны в базовом хранилище данных.

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
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

В этом случае при перемещении данных из большого двоичного объекта в таблицу Azure фабрика данных автоматически преобразует типы (включая тип поля с датой и временем в пользовательском формате), используя язык и региональные параметры fr-fr.

> [!NOTE]
> Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md). В ней описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.