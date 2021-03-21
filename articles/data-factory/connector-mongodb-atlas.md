---
title: Копирование данных из MongoDB Atlas
description: Узнайте, как копировать данные из MongoDB Atlas в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 09/28/2020
ms.openlocfilehash: b2f77e4bd8df66084937da3dd203ebb71d9a3511
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100368801"
---
# <a name="copy-data-from-mongodb-atlas-using-azure-data-factory"></a>Копирование данных из MongoDB Atlas с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных MongoDB Atlas. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Данные из базы данных MongoDB Atlas можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель MongoDB Atlas поддерживает **версии до 4,2**.

## <a name="prerequisites"></a>Предварительные условия

Если для копирования используется Azure Integration Runtime, убедитесь, что вы добавили Azure Integration Runtime IP- [адресов](azure-integration-runtime-ip-addresses.md) действующего региона в список доступа к MongoDB Atlas.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю MongoDB Atlas.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы MongoDB Atlas поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type |Для свойства Type необходимо задать значение **монгодбатлас** . |Да |
| connectionString |Укажите строку подключения MongoDB Atlas, например `mongodb+srv://<username>:<password>@<clustername>.<randomString>.<hostName>/<dbname>?<otherProperties>` . <br/><br /> Строку подключения также можно разместить в Azure Key Vault. Дополнительные сведения см. в разделе [хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md) . |Да |
| База данных | Имя базы данных, к которой нужно получить доступ. | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |Нет |

**Пример**.

```json
{
    "name": "MongoDbAtlasLinkedService",
    "properties": {
        "type": "MongoDbAtlas",
        "typeProperties": {
            "connectionString": "mongodb+srv://<username>:<password>@<clustername>.<randomString>.<hostName>/<dbname>?<otherProperties>",
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

Полный список разделов и свойств, используемых для определения наборов данных, приведен в статье [Наборы данных и связанные службы в фабрике данных Azure](concepts-datasets-linked-services.md). Для набора данных MongoDB Atlas поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **монгодбатласколлектион** . | Да |
| collectionName |Имя коллекции в базе данных MongoDB Atlas. |Да |

**Пример**.

```json
{
    "name": "MongoDbAtlasDataset",
    "properties": {
        "type": "MongoDbAtlasCollection",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<MongoDB Atlas linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником MongoDB Atlas.

### <a name="mongodb-atlas-as-source"></a>MongoDB Atlas в качестве источника

В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **монгодбатлассаурце** . | Да |
| фильтр | Задает фильтр выбора с помощью операторов запросов. Чтобы получить все документы в коллекции, не указывайте этот параметр или передайте пустой документ ({}). | Нет |
| cursorMethods.project | Определяет, какие поля в документах для проекции необходимо получить. Чтобы получить все поля в соответствующих документах, не указывайте этот параметр. | Нет |
| cursorMethods.sort | Определяет, в каком порядке запрос будет возвращать соответствующие документы. См. [cursor.sort()](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort). | Нет |
| cursorMethods.limit | Определяет максимальное количество документов, возвращаемых сервером. См. [cursor.limit()](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit).  | Нет |
| cursorMethods.skip | Указывает число документов для пропуска и from, с которых MongoDB Atlas начинает возвратить результаты. См. [cursor.skip()](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip). | Нет |
| batchSize | Указывает число документов, возвращаемых в каждом пакете ответа из экземпляра MongoDB Atlas. В большинстве случаев изменение размера пакета не влияет на пользователя или приложение. В Cosmos DB размер пакета не может превышать 40 МБ — это сумма размеров всех документов в этом пакете, поэтому при работе с большими документами их количество нужно уменьшать. | Нет<br/>(значение по умолчанию — **100**) |

>[!TIP]
>ADF поддерживает прием документа BSON в **строгом режиме**. Убедитесь в том, что запрос фильтра находится в строгом режиме, а не в режиме оболочки. Более подробное описание см. в [руководстве MongoDB](https://docs.mongodb.com/manual/reference/mongodb-extended-json/index.html).

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromMongoDbAtlas",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<MongoDB Atlas input dataset name>",
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
                "type": "MongoDbAtlasSource",
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

Этот соединитель MongoDB Atlas можно использовать для экспорта документов JSON "как есть" из коллекции MongoDB Atlas в различные файловые хранилища или в Azure Cosmos DB. Для такого независимого от схемы копирования пропустите раздел "структура" (также называемый *схема*) в наборе данных и сопоставление схемы в действии копирования.

## <a name="schema-mapping"></a>Сопоставление схем

Сведения о копировании данных из MongoDB Atlas в табличный приемник см. в разделе [сопоставление схем](copy-activity-schema-and-type-mapping.md#schema-mapping).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
