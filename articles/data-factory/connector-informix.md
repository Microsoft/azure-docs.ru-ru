---
title: Копирование данных из IBM Informix и обратно с помощью фабрики данных Azure
description: Узнайте, как копировать данные из IBM Informix и обратно с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: jingwang
ms.openlocfilehash: af907704862e80a2747ac064b98242a1d9d7edb3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588874"
---
# <a name="copy-data-from-and-to-ibm-informix-using-azure-data-factory"></a>Копирование данных из IBM Informix и обратно с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные из хранилища данных IBM Informix. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Informix поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из источника Informix можно скопировать в любое поддерживаемое хранилище данных в качестве приемника или скопировать из любого поддерживаемого исходного хранилища данных в приемник Informix. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).


## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать этот соединитель Informix, необходимо выполнить следующие действия:

- Настроить локальную среду выполнения интеграции. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).
- Установите драйвер ODBC Informix для хранилища данных на компьютере Integration Runtime. Сведения об установке и настройке драйвера см. в статье [руководство по драйверам ODBC для Informix](https://www.ibm.com/support/knowledgecenter/SSGU8G_11.70.0/com.ibm.odbc.doc/odbc.htm) в центре базы знаний IBM для получения дополнительных сведений или обратитесь в службу поддержки IBM за инструкциями по установке драйверов.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю Informix.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Informix поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **Informix** . | Да |
| connectionString | Строка подключения ODBC за исключением части учетных данных. Можно указать строку подключения или использовать DSN системы (имя источника данных), который вы настроили на Integration Runtime компьютере (необходимо по-прежнему указать часть учетных данных в связанной службе). <br> Можно также добавить пароль в Azure Key Vault и извлечь `password` конфигурацию из строки подключения. Дополнительные сведения см. в разделе [хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md) .| Да |
| authenticationType | Тип проверки подлинности, используемый для подключения к хранилищу данных Informix.<br/>Допустимые значения: **базовый** и **Анонимный**. | Да |
| userName | При использовании обычной проверки подлинности укажите имя пользователя. | Нет |
| password | Введите пароль для учетной записи пользователя, указанной для выбранного имени пользователя. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | нет |
| учетные данные | Учетные данные в строке подключения, используемые для получения доступа и указанные в формате "драйвер-определенное свойство-значение". Пометьте это поле в качестве SecureString. | Нет |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Требуется локальная среда IR, как упоминалось в разделе [Предварительные требования](#prerequisites). |Да |

**Пример**.

```json
{
    "name": "InformixLinkedService",
    "properties": {
        "type": "Informix",
        "typeProperties": {
            "connectionString": "<Informix connection string or DSN>",
            "authenticationType": "Basic",
            "userName": "<username>",
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Informix.

Чтобы скопировать данные из Informix, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **информикстабле** . | Да |
| tableName | Имя таблицы в Informix. | Нет для источника (если свойство query указано в источнике действия).<br/>Да для приемника. |

**Пример**

```json
{
    "name": "InformixDataset",
    "properties": {
        "type": "InformixTable",
        "linkedServiceName": {
            "referenceName": "<Informix linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Informix.

### <a name="informix-as-source"></a>Informix в качестве источника

Чтобы скопировать данные из Informix, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **информикссаурце** . | Да |
| query | Используйте пользовательский запрос для чтения данных. Например: `"SELECT * FROM MyTable"`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromInformix",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Informix input dataset name>",
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
                "type": "InformixSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="informix-as-sink"></a>Informix в качестве приемника

Чтобы скопировать данные в Informix, в разделе **приемника** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение **информикссинк** . | Да |
| writeBatchTimeout |Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия.<br/>Допустимые значения: промежуток времени. Пример "00:30:00" (30 минут). |Нет |
| writeBatchSize |Вставляет данные в таблицу SQL, когда размер буфера достигает значения writeBatchSize.<br/>Допустимые значения: целое число (количество строк). |Нет (по умолчанию — 0 (автоматическое обнаружение)) |
| preCopyScript |Перед записью данных в хранилище данных при каждом запуске указывайте SQL-запрос для выполнения операции копирования. Это свойство можно использовать для очистки предварительно загруженных данных. |Нет |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyToInformix",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Informix output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "InformixSink"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
