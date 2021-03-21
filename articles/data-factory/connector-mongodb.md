---
title: Копирование данных из MongoDB
description: Узнайте, как копировать данные из MongoDB на поддерживаемые приемники хранилища данных с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 01/08/2021
ms.openlocfilehash: eae2d2adfe2cfdd7e47f2ace9ede9253003aa5bf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100368784"
---
# <a name="copy-data-from-mongodb-using-azure-data-factory"></a>Копирование данных из MongoDB с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных MongoDB. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

>[!IMPORTANT]
>ADF выпустил новую версию соединителя MongoDB, которая обеспечивает более эффективную поддержку MongoDB. При использовании предыдущей соединителя MongoDB в решении, которое поддерживается как есть, для обеспечения обратной совместимости см. статью [Соединитель MongoDB (для прежних версий)](connector-mongodb-legacy.md).


## <a name="supported-capabilities"></a>Поддерживаемые возможности

Данные из базы данных MongoDB можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель MongoDB поддерживает **версии до 4,2**.


## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]


## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к соединителю MongoDB.


## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы MongoDB поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type |Для свойства Type необходимо задать значение **MongoDbV2** . |Да |
| connectionString |Укажите строку подключения MongoDB, например `mongodb://[username:password@]host[:port][/[database][?options]]`. Дополнительные сведения см. в [руководстве MongoDB по строке подключения](https://docs.mongodb.com/manual/reference/connection-string/). <br/><br /> Строку подключения также можно разместить в Azure Key Vault. Дополнительные сведения см. в разделе [хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md) . |Да |
| База данных | Имя базы данных, к которой нужно получить доступ. | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |Нет |

**Пример**.

```json
{
    "name": "MongoDBLinkedService",
    "properties": {
        "type": "MongoDbV2",
        "typeProperties": {
            "connectionString": "mongodb://[username:password@]host[:port][/[database][?options]]",
            "database": "myDatabase"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, используемых для определения наборов данных, приведен в статье [Наборы данных и связанные службы в фабрике данных Azure](concepts-datasets-linked-services.md). Набор данных MongoDB поддерживает следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **MongoDbV2Collection** . | Да |
| collectionName |Имя коллекции в базе данных MongoDB |Да |

**Пример**.

```json
{
    "name": "MongoDbDataset",
    "properties": {
        "type": "MongoDbV2Collection",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<MongoDB linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```


## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником MongoDB.

### <a name="mongodb-as-source"></a>MongoDB в качестве источника

В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **MongoDbV2Source** . | Да |
| фильтр | Задает фильтр выбора с помощью операторов запросов. Чтобы получить все документы в коллекции, не указывайте этот параметр или передайте пустой документ ({}). | Нет |
| cursorMethods.project | Определяет, какие поля в документах для проекции необходимо получить. Чтобы получить все поля в соответствующих документах, не указывайте этот параметр. | Нет |
| cursorMethods.sort | Определяет, в каком порядке запрос будет возвращать соответствующие документы. См. [cursor.sort()](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort). | Нет |
| cursorMethods.limit | Определяет максимальное количество документов, возвращаемых сервером. См. [cursor.limit()](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit).  | Нет |
| cursorMethods.skip | Определяет, сколько документов нужно пропустить, прежде чем MongoDB начнет выдавать результаты. См. [cursor.skip()](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip). | Нет |
| batchSize | Определяет, сколько документов должно быть выдано в каждом пакете ответа от экземпляра MongoDB. В большинстве случаев изменение размера пакета не влияет на пользователя или приложение. Cosmos DB ограничения для каждого пакета не могут превышать 40 МБ, то есть сумма batchSize количества документов, поэтому уменьшите это значение, если размер документа велик. | Нет<br/>(значение по умолчанию — **100**) |

>[!TIP]
>ADF поддерживает прием документа BSON в **строгом режиме**. Убедитесь в том, что запрос фильтра находится в строгом режиме, а не в режиме оболочки. Более подробное описание см. в [руководстве MongoDB](https://docs.mongodb.com/manual/reference/mongodb-extended-json/index.html).

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromMongoDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<MongoDB input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "MongoDbV2Source",
                "filter": "{datetimeData: {$gte: ISODate(\"2018-12-11T00:00:00.000Z\"),$lt: ISODate(\"2018-12-12T00:00:00.000Z\")}, _id: ObjectId(\"5acd7c3d0000000000000000\") }",
                "cursorMethods": {
                    "project": "{ _id : 1, name : 1, age: 1, datetimeData: 1 }",
                    "sort": "{ age : 1 }",
                    "skip": 3,
                    "limit": 3
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```


## <a name="export-json-documents-as-is"></a>Экспорт документов JSON без изменений

Соединитель MongoDB можно использовать для экспорта документов JSON из коллекции MongoDB в различные файловые хранилища или Azure Cosmos DB без изменений. Для такого независимого от схемы копирования пропустите раздел "структура" (также называемый *схема*) в наборе данных и сопоставление схемы в действии копирования.


## <a name="schema-mapping"></a>Сопоставление схем

Инструкции по копированию данных из MongoDB в табличный приемник см. в разделе о [сопоставлении схем](copy-activity-schema-and-type-mapping.md#schema-mapping).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
