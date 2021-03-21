---
title: Копирование данных из PostgreSQL с помощью фабрики данных Azure
description: Узнайте, как копировать данные из PostgreSQL на поддерживаемые приемники хранилища данных с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/19/2020
ms.author: jingwang
ms.openlocfilehash: c0d8783c4b07c8c87e57ccd9a5d65983825347c9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100384101"
---
# <a name="copy-data-from-postgresql-by-using-azure-data-factory"></a>Копирование данных из PostgreSQL с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-onprem-postgresql-connector.md)
> * [Текущая версия](connector-postgresql.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных PostgreSQL. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель PostgreSQL поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из базы данных PostgreSQL можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель PostgreSQL поддерживает PostgreSQL **версии 7.4 и более поздних**.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

Начиная с версии 3.7 служба Integration Runtime предоставляет встроенный драйвер PostgreSQL, поэтому вам не потребуется устанавливать драйвер вручную.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к соединителю PostgreSQL.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы PostgreSQL поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **PostgreSql** | Да |
| connectionString | Строка подключения к базе данных Azure для PostgreSQL через интерфейс ODBC. <br/>Вы можете также поместить пароль в Azure Key Vault и извлечь конфигурацию `password` из строки подключения. Ознакомьтесь с приведенными ниже примерами и подробными сведениями в статье [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |нет |

Типичная строка подключения — `Server=<server>;Database=<database>;Port=<port>;UID=<username>;Password=<Password>`. Дополнительные свойства, которые вы можете установить в вашем случае:

| Свойство | Описание | Параметры | Обязательно |
|:--- |:--- |:--- |:--- |
| EncryptionMethod (EM)| Метод, используемый драйвером для шифрования данных, отправленных между драйвером и сервером базы данных. Пример: `EncryptionMethod=<0/1/6>;`.| 0 (без шифрования) **(по умолчанию)** -1 (SSL) или 6 (RequestSSL) | Нет |
| ValidateServerCertificate (VSC) | Определяет, проверяет ли драйвер сертификат, отправленный сервером базы данных, когда включено шифрование SSL (метод шифрования = 1). Пример: `ValidateServerCertificate=<0/1>;`.| 0 (отключено) **(по умолчанию)** -1 (включено) | Нет |

**Пример**.

```json
{
    "name": "PostgreSqlLinkedService",
    "properties": {
        "type": "PostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>;Database=<database>;Port=<port>;UID=<username>;Password=<Password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример: хранение пароля в Azure Key Vault**

```json
{
    "name": "PostgreSqlLinkedService",
    "properties": {
        "type": "PostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>;Database=<database>;Port=<port>;UID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Ранее используемая связанная служба PostgreSQL со приведенными ниже полезными данными по-прежнему поддерживается. Но мы рекомендуем в дальнейшем использовать более новую версию.

**Предыдущие полезные данные:**

```json
{
    "name": "PostgreSqlLinkedService",
    "properties": {
        "type": "PostgreSql",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных PostgreSQL.

Чтобы скопировать данные из PostgreSQL, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **постгресклтабле** . | Да |
| схема | Имя схемы. |Нет (если свойство query указано в источнике действия)  |
| table | Имя таблицы. |Нет (если свойство query указано в источнике действия)  |
| tableName | Имя таблицы со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новых рабочих нагрузок используйте `schema` и `table`. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "PostgreSQLDataset",
    "properties":
    {
        "type": "PostgreSqlTable",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<PostgreSQL linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Если вы использовали `RelationalTable` типизированный набор данных, он по-прежнему поддерживается "как есть", хотя вы можете использовать новый объект, который будет использоваться в дальнейшем.

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником PostgreSQL.

### <a name="postgresql-as-source"></a>PostgreSQL в качестве источника

Чтобы скопировать данные из PostgreSQL, в разделе **источник** действия копирования поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **постгресклсаурце** . | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"query": "SELECT * FROM \"MySchema\".\"MyTable\""`. | Нет (если для набора данных задано свойство tableName) |

> [!NOTE]
> В именах схем и таблиц учитывается регистр. Заключите имя в `""` (двойные кавычки) в запросе.

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromPostgreSQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<PostgreSQL input dataset name>",
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
                "type": "PostgreSqlSource",
                "query": "SELECT * FROM \"MySchema\".\"MyTable\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Если вы ранее использовали типизированный источник `RelationalSource`, он пока поддерживается и не требует изменений, но мы рекомендуем в дальнейшем использовать более новую версию.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
