---
title: Копирование данных из Sybase с помощью фабрики данных Azure
description: Узнайте, как копировать данные из Sybase в поддерживаемые хранилища данных в качестве приемников с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/10/2020
ms.author: jingwang
ms.openlocfilehash: 2ef63eded5403c1cf5faddec71ed3503c3ae2138
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100384815"
---
# <a name="copy-data-from-sybase-using-azure-data-factory"></a>Копирование данных из Sybase с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-onprem-sybase-connector.md)
> * [Текущая версия](connector-sybase.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных Sybase. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Sybase поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Вы можете скопировать данные из базы данных Sybase в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель Sybase поддерживает:

- SAP Sybase SQL Anywhere (ASA) **версии 16 и выше**.
- копирование данных с использованием **базовой** проверки подлинности или проверки подлинности **Windows**.

Sybase IQ и ASE не поддерживаются. Вместо этого можно использовать универсальный соединитель ODBC с драйвером Sybase.

## <a name="prerequisites"></a>Предварительные условия

Для использования этого соединителя Sybase вам нужно:

- Настроить локальную среду выполнения интеграции. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).
- Установить [поставщик данных для Sybase iAnywhere.Data.SQLAnywhere](https://go.microsoft.com/fwlink/?linkid=324846) версии 16 или более поздней на машине со средой выполнения интеграции.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения объектов фабрики данных, относящихся к соединителю Sybase.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Sybase поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **Sybase**. | Да |
| server | Имя сервера Sybase. |Да |
| База данных | Имя базы данных Sybase. |Да |
| authenticationType | Тип проверки подлинности, используемый для подключения к базе данных Sybase.<br/>Допустимые значения: **Basic** и **Windows**. |Да |
| username | Укажите имя пользователя для подключения к базе данных Sybase. |Да |
| password | Введите пароль для учетной записи пользователя, указанной для выбранного имени пользователя. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Требуется локальная среда IR, как упоминалось в разделе [Предварительные требования](#prerequisites). |Да |

**Пример**.

```json
{
    "name": "SybaseLinkedService",
    "properties": {
        "type": "Sybase",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "authenticationType": "Basic",
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Sybase.

Чтобы скопировать данные из Sybase, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **сибасетабле** . | Да |
| tableName | Имя таблицы в базе данных Sybase. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "SybaseDataset",
    "properties": {
        "type": "SybaseTable",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Sybase linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Если вы ранее использовали типизированный набор данных `RelationalTable`, он пока поддерживается и не требует изменений, но мы рекомендуем при любом удобном случае перейти на новую версию.

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых Sybase в качестве источника.

### <a name="sybase-as-source"></a>Sybase в качестве источника

Чтобы скопировать данные из Sybase, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **сибасесаурце** . | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM MyTable"`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromSybase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Sybase input dataset name>",
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
                "type": "SybaseSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Если вы ранее использовали типизированный источник `RelationalSource`, он пока поддерживается и не требует изменений, но мы рекомендуем в дальнейшем использовать более новую версию.

## <a name="data-type-mapping-for-sybase"></a>Сопоставление типов для Sybase

При копировании данных из Sybase используются сопоставления типов данных Sybase с промежуточными типами данных фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

Sybase поддерживает типы T-SQL. Таблицы сопоставлений SQL типов к типам промежуточных данных Фабрики данных Azure см. в разделе [Data type mapping for Azure SQL Database](connector-azure-sql-database.md#data-type-mapping-for-azure-sql-database) (Отображение типа данных для Базы данных Azure SQL) статьи "Copy data to or from Azure SQL Database by using Azure Data Factory" (Копирование данных в базу данных Azure SQL или из нее с использованием Фабрики данных Azure).

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).



## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
