---
title: Сопоставление столбцов набора данных в фабрике данных Azure
description: Узнайте, как сопоставить исходные столбцы с целевыми столбцами.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
robots: noindex
ms.openlocfilehash: fef2c6f120ae25e6aa1846d4971ff707da9bab92
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100371130"
---
# <a name="map-source-dataset-columns-to-destination-dataset-columns"></a>Сопоставление столбцов исходного набора данных со столбцами целевого набора данных
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. 

Сопоставление столбцов можно использовать для указания того, как столбцы, указанные в "структуре" исходной таблицы, сопоставляются со столбцами, заданными в "структуре" таблицы приемника. Свойство **ColumnMapping** доступно в разделе **typeProperties** действия копирования.

Сопоставление столбцов применимо в следующих сценариях:

* Все столбцы в разделе structure набора данных, используемого в качестве источника, сопоставляются со всеми столбцами в разделе structure набора данных, используемого в качестве приемника.
* Подмножество столбцов в разделе structure набора данных, используемого в качестве источника, сопоставляется со всеми столбцами в разделе structure набора данных, используемого в качестве приемника.

Ниже приведены неправильные условия, которые приводят к порождению исключения.

* Меньше или больше столбцов в "структуре" таблицы приемника, чем указано в сопоставлении.
* Повторяющееся сопоставление.
* В результате запроса SQL нет имени столбца, который указан в сопоставлении.

> [!NOTE]
> Приведенные ниже примеры предназначены для SQL Azure и большого двоичного объекта Azure, но подходят для любого хранилища данных, поддерживающего прямоугольные наборы данных. В этих примерах можно настроить набор данных и определения связанных служб, чтобы указать данные в соответствующем источнике данных.

## <a name="sample-1--column-mapping-from-azure-sql-to-azure-blob"></a>Пример 1. Сопоставление столбцов из SQL Azure с большим двоичным объектом Azure
В этом примере входная таблица имеет структуру и указывает на таблицу SQL в базе данных SQL Azure.

```json
{
    "name": "AzureSQLInput",
    "properties": {
        "structure": 
         [
           { "name": "userid"},
           { "name": "name"},
           { "name": "group"}
         ],
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Выходная таблица имеет структуру, и она указывает на большой двоичный объект в хранилище больших двоичных объектов Azure.

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
         "structure": 
          [
                { "name": "myuserid"},
                { "name": "myname" },
                { "name": "mygroup"}
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
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Ниже приведен фрагмент JSON, который определяет действие копирования в конвейере. Столбцы из источника сопоставляются со столбцами в приемнике (**ColumnMappings**) с помощью свойства **Translator**.

```json
{
    "name": "CopyActivity",
    "description": "description", 
    "type": "Copy",
    "inputs":  [ { "name": "AzureSQLInput"  } ],
    "outputs":  [ { "name": "AzureBlobOutput" } ],
    "typeProperties":    {
        "source":
        {
            "type": "SqlSource"
        },
        "sink":
        {
            "type": "BlobSink"
        },
        "translator": 
        {
            "type": "TabularTranslator",
            "ColumnMappings": "UserId: MyUserId, Group: MyGroup, Name: MyName"
        }
    },
   "scheduler": {
          "frequency": "Hour",
          "interval": 1
        }
}
```
**Поток сопоставления столбцов:**

![Процесс сопоставления столбцов](./media/data-factory-map-columns/column-mapping-flow.png)

## <a name="sample-2--column-mapping-with-sql-query-from-azure-sql-to-azure-blob"></a>Пример 2. Сопоставление столбцов из Azure SQL с большим двоичным объектом Azure с помощью SQL-запроса
В этом примере SQL-запрос используется для извлечения данных из SQL Azure вместо простого указания имени таблицы и имен столбцов в разделе Structure. 

```json
{
    "name": "CopyActivity",
    "description": "description", 
    "type": "CopyActivity",
    "inputs":  [ { "name": " AzureSQLInput"  } ],
    "outputs":  [ { "name": " AzureBlobOutput" } ],
    "typeProperties":
    {
        "source":
        {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartDateTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
        },
        "sink":
        {
            "type": "BlobSink"
        },
        "Translator": 
        {
            "type": "TabularTranslator",
            "ColumnMappings": "UserId: MyUserId, Group: MyGroup,Name: MyName"
        }
    },
    "scheduler": {
          "frequency": "Hour",
          "interval": 1
        }
}
```
В этом случае результаты запроса сначала сопоставляются со столбцами, указанными в поле «структура» источника. Далее столбцы из источника "структура" сопоставляются со столбцами в "структуре" приемника с правилами, заданными в columnMappings.  Предположим, что запрос возвращает 5 столбцов, два больше столбцов, чем указано в «структуре» источника.

**Процесс сопоставления столбцов**

![Процесс сопоставления столбцов 2](./media/data-factory-map-columns/column-mapping-flow-2.png)

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с руководством по использованию действия копирования в следующей статье: 

- [Копирование данных из хранилища BLOB-объектов Azure в базу данных SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
