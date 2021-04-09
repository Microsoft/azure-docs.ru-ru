---
title: Копирование данных из DB2 с помощью Фабрики данных Azure
description: Узнайте, как копировать данные из DB2 в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: jingwang
ms.openlocfilehash: 642f12386a7695e026eb0c30016acf6f53fc9e95
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100381126"
---
# <a name="copy-data-from-db2-by-using-azure-data-factory"></a>Копирование данных из DB2 с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-onprem-db2-connector.md)
> * [Текущая версия](connector-db2.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных DB2. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель базы данных DB2 поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Действие поиска](control-flow-lookup-activity.md)

Данные из базы данных DB2 можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель DB2 поддерживает приведенные ниже платформы и версии IBM DB2 с архитектурой распределенной реляционной базы данных (DRDA) SQL Access Manager (SQLAM) версии 9, 10 и 11.  Он использует протокол DDM/DRDA.

* IBM DB2 для z/OS версии 12.1
* IBM DB2 для z/OS версии 11.1
* IBM DB2 для z/OS версии 10.1
* IBM DB2 для i версии 7.3
* IBM DB2 для i версии 7.2
* IBM DB2 для i версии 7.1
* IBM DB2 для LUW версии 11
* IBM DB2 для LUW версии 10.5
* IBM DB2 для LUW версии 10.1

>[!TIP]
>Соединитель DB2 создан на основе поставщика OLE DB для DB2 (Майкрософт). Чтобы устранить неполадки с соединителем DB2, изучите [коды ошибок поставщика данных](/host-integration-server/db2oledbv/data-provider-error-codes#drda-protocol-errors).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

Среда выполнения интеграции предоставляет встроенный драйвер DB2, поэтому при копировании данных из DB2 вам не потребуется устанавливать драйвер вручную.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к соединителю DB2.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы DB2 поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойству type необходимо задать значение **Db2** | Да |
| connectionString | Укажите сведения, необходимые для подключения к экземпляру DB2.<br/> Вы можете также поместить пароль в Azure Key Vault и извлечь конфигурацию `password` из строки подключения. Ознакомьтесь с приведенными ниже примерами и подробными сведениями в статье [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |нет |

Типичный набор свойств для строки подключения.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| server |Имя сервера DB2. Вы можете указать номер порта следом за именем сервера, разделив их двоеточием, например `server:port`.<br>Соединитель DB2 использует протокол DDM/DRDA и порт 50 000 по умолчанию, если он не настроен. Для конкретной базы данных DB2 используемый порт может отличаться в зависимости от версии и настроенных параметров. Например, для DB2 LUW по умолчанию используется порт 50 000, а для AS400 — 446 (или 448, если включен TLS). Сведения о настройке порта см. в следующих документах по версиям DB2: [DB2 z/OS](https://www.ibm.com/support/knowledgecenter/SSEPGG_11.5.0/com.ibm.db2.luw.qb.dbconn.doc/doc/t0008229.html), [DB2 iSeries](https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_74/ddp/rbal1ports.htm) и [DB2 LUW](https://www.ibm.com/support/knowledgecenter/en/SSEKCU_1.1.3.0/com.ibm.psc.doc/install/psc_t_install_typical_db2_port.html). |Да |
| База данных |Имя базы данных DB2. |Да |
| authenticationType |Тип проверки подлинности, используемый для подключения к базе данных DB2.<br/>Допустимое значение: **Базовый** |Да |
| username |Укажите имя пользователя для подключения к базе данных DB2. |Да |
| password |Введите пароль для учетной записи пользователя, указанной для выбранного имени пользователя. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| packageCollection    | Укажите, в каком расположении служба ADF создает необходимые пакеты при запросе к базе данных. Если этот параметр не задан, Фабрика данных использует {username} в качестве значения по умолчанию. | нет |
| certificateCommonName | При использовании протоколов SSL или TLS необходимо ввести значение для параметра "Общее имя сертификата". | нет |

> [!TIP]
> Если вы получите сообщение об ошибке `The package corresponding to an SQL statement execution request was not found. SQLSTATE=51002 SQLCODE=-805`, это означает, что необходимый пакет для пользователя не создан. По умолчанию ADF попытается создать пакет в коллекции с тем же именем, что у пользователя, которое использовалось при подключении к DB2. Укажите свойство коллекции пакетов, чтобы определить, где ADF будет создавать необходимые пакеты при запросе к базе данных.

**Пример**.

```json
{
    "name": "Db2LinkedService",
    "properties": {
        "type": "Db2",
        "typeProperties": {
            "connectionString": "server=<server:port>;database=<database>;authenticationType=Basic;username=<username>;password=<password>;packageCollection=<packagecollection>;certificateCommonName=<certname>;"
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
    "name": "Db2LinkedService",
    "properties": {
        "type": "Db2",
        "typeProperties": {
            "connectionString": "server=<server:port>;database=<database>;authenticationType=Basic;username=<username>;packageCollection=<packagecollection>;certificateCommonName=<certname>;",
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

Если вы раньше использовали связанную службу DB2 с перечисленными ниже полезными данными, такая система пока поддерживается и не требует изменений, но мы рекомендуем при любом удобном случае перейти на новую версию.

**Предыдущие полезные данные:**

```json
{
    "name": "Db2LinkedService",
    "properties": {
        "type": "Db2",
        "typeProperties": {
            "server": "<servername:port>",
            "database": "<dbname>",
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных DB2.

Для копирования данных из DB2 поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type набора данных необходимо задать следующее значение: **Db2Table** | Да |
| схема | Имя схемы. |Нет (если свойство query указано в источнике действия)  |
| table | Имя таблицы. |Нет (если свойство query указано в источнике действия)  |
| tableName | Имя таблицы со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новых рабочих нагрузок используйте `schema` и `table`. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "DB2Dataset",
    "properties":
    {
        "type": "Db2Table",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<DB2 linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Если вы ранее использовали типизированный набор данных `RelationalTable`, он пока поддерживается и не требует изменений, но мы рекомендуем при любом удобном случае перейти на новую версию.

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником DB2.

### <a name="db2-as-source"></a>DB2 в качестве источника

Для копирования данных из DB2 в разделе **source** для действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойству type источника действия копирования необходимо задать значение **Db2Source** | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromDB2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<DB2 input dataset name>",
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
                "type": "Db2Source",
                "query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Если вы ранее использовали типизированный источник `RelationalSource`, он пока поддерживается и не требует изменений, но мы рекомендуем в дальнейшем использовать более новую версию.

## <a name="data-type-mapping-for-db2"></a>Сопоставление типов данных для DB2

При копировании данных из DB2 используются следующие сопоставления из типов данных DB2 с промежуточными типами данных фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип базы данных DB2 | Тип промежуточных данных фабрики данных |
|:--- |:--- |
| BigInt |Int64 |
| Двоичные данные |Byte[] |
| BLOB-объект |Byte[] |
| CHAR |Строка |
| Clob |Строка |
| Дата |Datetime |
| DB2DynArray |Строка |
| DbClob |Строка |
| Decimal |Decimal |
| DecimalFloat |Decimal |
| Double |Double |
| Float |Double |
| GRAPHIC |Строка |
| Целое число |Int32 |
| LongVarBinary |Byte[] |
| LongVarChar |Строка |
| LongVarGraphic |Строка |
| Числовой |Decimal |
| Real |Один |
| SmallInt |Int16 |
| Time |TimeSpan |
| Отметка времени |Дата и время |
| VarBinary |Byte[] |
| VarChar |Строка |
| VarGraphic |Строка |
| Xml |Byte[] |

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.