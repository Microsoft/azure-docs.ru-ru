---
title: Копирование данных в Oracle и обратно с помощью фабрики данных Azure
description: Узнайте, как копировать данные из поддерживаемых исходных хранилищ в базу данных Oracle или из Oracle в поддерживаемые хранилища приемников с помощью фабрики данных.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: jingwang
ms.openlocfilehash: 9e6be88af13d5dd7ddceba32ec08cab54ca5e3a0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587293"
---
# <a name="copy-data-from-and-to-oracle-by-using-azure-data-factory"></a>Копирование данных из Oracle и обратно с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-onprem-oracle-connector.md)
> * [Текущая версия](connector-oracle.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных Oracle и в нее. Она основана на [обзоре действия копирования](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Oracle поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Вы можете скопировать данные из базы данных Oracle в любое хранилище данных, поддерживаемое в качестве приемника. Также можно скопировать данные из любого поддерживаемого хранилища данных в базу данных Oracle. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель Oracle поддерживает:

- Следующие версии базы данных Oracle:
    - Oracle 19c R1 (19,1) и выше
    - Oracle 18C R1 (18,1) и выше
    - Oracle 12c R1 (12,1) и выше
    - Oracle 11g R1 (11,1) и выше
    - Oracle 10G R1 (10,1) и более поздних версий
    - Oracle 9i R2 (9,2) и более поздние версии
    - Oracle 8i R3 (8.1.7) и более поздние версии
    - Служба Oracle Database Cloud Ексадата
- Параллельное копирование из источника Oracle. Дополнительные сведения см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) .

> [!Note]
> Прокси-сервер Oracle не поддерживается.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)] 

Среда выполнения интеграции содержит встроенный драйвер Oracle. Поэтому вам не нужно вручную устанавливать драйвер при копировании данных из базы данных Oracle и в нее.

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю Oracle.

## <a name="linked-service-properties"></a>Свойства связанной службы

Связанная служба Oracle поддерживает следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **Oracle**. | Да |
| connectionString | Указывает сведения, необходимые для подключения к экземпляру базы данных Oracle. <br/>Можно также добавить пароль в Azure Key Vault и извлечь `password` конфигурацию из строки подключения. Ознакомьтесь со следующими примерами и [Храните учетные данные в Azure Key Vault](store-credentials-in-key-vault.md) с дополнительными сведениями. <br><br>**Поддерживаемые типы подключений**: вы можете использовать **ИД безопасности Oracle** или **имя службы Oracle** для идентификации базы данных.<br>— Если вы используете идентификатор безопасности, используйте этот код для подключения: `Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;`<br>— Если вы используете имя службы, используйте этот код: `Host=<host>;Port=<port>;ServiceName=<servicename>;User Id=<username>;Password=<password>;`<br>Дополнительные параметры подключения Oracle Native позволяют добавить запись в [TNSNAMES. Файл ORA](http://www.orafaq.com/wiki/Tnsnames.ora) на сервере Oracle и в связанной службе ADF Oracle, выберите использовать имя службы Oracle тип соединения и настройте соответствующее имя службы. | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная Azure Integration Runtime. |Нет |

>[!TIP]
>При возникновении ошибки "ORA-01025: УПИ параметр вне диапазона", а версия Oracle — 8i, добавьте `WireProtocolMode=1` в строку подключения. Затем повторите попытку.

При наличии нескольких экземпляров Oracle для сценария отработки отказа можно создать связанную службу Oracle и заполнить основной узел, порт, имя пользователя, пароль и т. д., а также добавить новое значение "**Дополнительные свойства подключения**" с именем свойства как `AlternateServers` и значением "не `(HostName=<secondary host>:PortNumber=<secondary port>:ServiceName=<secondary service name>)` пропускать скобки" и обращать внимание на двоеточия ( `:` ) в качестве разделителя. Например, следующее значение альтернативных серверов определяет два альтернативных сервера баз данных для отработки отказа подключения: `(HostName=AccountingOracleServer:PortNumber=1521:SID=Accounting,HostName=255.201.11.24:PortNumber=1522:ServiceName=ABackup.NA.MyCompany)` .

Дополнительные свойства подключения, которые можно задать в строке подключения, в любом случае:

| Свойство. | Описание | Допустимые значения |
|:--- |:--- |:--- |
| ArraySize |Число байтов, которое соединитель может получить в одном круговом пути сети. Например, `ArraySize=‭10485760‬` .<br/><br/>Большие значения увеличивают пропускную способность, уменьшая число попыток выборки данных по сети. Меньшие значения увеличивают время отклика, так как меньше задержки ожидает передачи данных с сервера. | Целое число от 1 до 4294967296 (4 ГБ). Значение по умолчанию — `60000`. Значение 1 не определяет число байтов, а указывает на выделение пространства только для одной строки данных. |

Существует два варианта, чтобы включить шифрование для подключения Oracle:

-   Чтобы использовать **Шифрование Triple-DES (3DES) и AES (AES)** на стороне сервера Oracle, перейдите в раздел Oracle Advanced Security (OAS) и настройте параметры шифрования. Дополнительные сведения см. в этой [документации Oracle](https://docs.oracle.com/cd/E11882_01/network.112/e40393/asointro.htm#i1008759). Соединитель платформы Oracle Application Development Framework (ADF) автоматически согласовывает метод шифрования для использования того, который вы настроили в OAS при установлении соединения с Oracle.

-   Использование **TLS**:

    1.  Получите сведения о сертификате TLS/SSL. Получите сведения о сертификате TLS/SSL, закодированные в Distinguished Encoding Rules (DER), и сохраните выходные данные (-----Begin Certificate... End Certificate -----) как текстовый файл.

        ```
        openssl x509 -inform DER -in [Full Path to the DER Certificate including the name of the DER Certificate] -text
        ```

        **Пример:** Извлеките сведения о сертификате из Дерцерт. cer и сохраните выходные данные в cert.txt.

        ```
        openssl x509 -inform DER -in DERcert.cer -text
        Output:
        -----BEGIN CERTIFICATE-----
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXX
        -----END CERTIFICATE-----
        ```
    
    2.  Создайте `keystore` или `truststore` . Следующая команда создает `truststore` файл с паролем в формате PKCS-12 или без него.

        ```
        openssl pkcs12 -in [Path to the file created in the previous step] -out [Path and name of TrustStore] -passout pass:[Keystore PWD] -nokeys -export
        ```

        **Пример:** Создайте файл PKCS12 с `truststore` именем митрустсторефиле и паролем.

        ```
        openssl pkcs12 -in cert.txt -out MyTrustStoreFile -passout pass:ThePWD -nokeys -export  
        ```

    3.  Поместите `truststore` файл на автономный IR-компьютер. Например, поместите файл по адресу К:\митрустсторефиле..
    4.  В фабрике данных Azure настройте строку подключения Oracle с помощью `EncryptionMethod=1` и соответствующего `TrustStore` / `TrustStorePassword` значения. Например, `Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;EncryptionMethod=1;TrustStore=C:\\MyTrustStoreFile;TrustStorePassword=<trust_store_password>`.

**Пример**.

```json
{
    "name": "OracleLinkedService",
    "properties": {
        "type": "Oracle",
        "typeProperties": {
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;"
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
    "name": "OracleLinkedService",
    "properties": {
        "type": "Oracle",
        "typeProperties": {
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;",
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
## <a name="dataset-properties"></a>Свойства набора данных

В этом разделе содержится список свойств, поддерживаемых набором данных Oracle. Полный список разделов и свойств, доступных для определения наборов данных, см. в разделе [Наборы данных в фабрике данных Azure](concepts-datasets-linked-services.md). 

Чтобы скопировать данные из и в Oracle, присвойте свойству Type набора данных значение `OracleTable` . Поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение `OracleTable` . | Да |
| схема | Имя схемы. |"Нет" для источника, "Да" для приемника  |
| table | Имя таблицы или представления. |"Нет" для источника, "Да" для приемника  |
| tableName | Имя таблицы или представления со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новой рабочей нагрузки используйте `schema` и `table`. | "Нет" для источника, "Да" для приемника |

**Пример**.

```json
{
    "name": "OracleDataset",
    "properties":
    {
        "type": "OracleTable",
        "schema": [],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        },
        "linkedServiceName": {
            "referenceName": "<Oracle linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Этот раздел содержит список свойств, поддерживаемых источником и приемником Oracle. Полный список разделов и свойств, доступных для определения действий, см. в разделе [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). 

### <a name="oracle-as-source"></a>Oracle в качестве источника

>[!TIP]
>Чтобы эффективно загружать данные из Oracle с помощью секционирования данных, Узнайте больше от [параллельного копирования из Oracle](#parallel-copy-from-oracle).

Чтобы скопировать данные из Oracle, задайте тип источника в действии копирования `OracleSource` . В разделе **source** действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение `OracleSource` . | Да |
| oracleReaderQuery | Используйте пользовательский SQL-запрос для чтения данных. Например, `"SELECT * FROM MyTable"`.<br>При включении секционированной нагрузки необходимо привязать все соответствующие встроенные параметры секции в запросе. Примеры см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) . | Нет |
| partitionOptions | Задает параметры секционирования данных, используемые для загрузки данных из Oracle. <br>Допустимые значения: **None** (по умолчанию), **фисикалпартитионсофтабле** и **динамикранже**.<br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из базы данных Oracle управляется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр Partition не имеет значение `None` . | Нет |
| партитионнамес | Список физических разделов, которые необходимо скопировать. <br>Применяется, если параметр секционирования имеет значение `PhysicalPartitionsOfTable`. Если для получения исходных данных используется запрос, подключите `?AdfTabularPartitionName` в предложении WHERE. Пример см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) . | нет |
| partitionColumnName | Укажите имя исходного столбца **целочисленного типа**, который будет использоваться для секционирования по диапазонам при параллельном копировании. Если этот параметр не указан, первичный ключ таблицы определяется автоопределением и используется в качестве столбца секционирования. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, присоединитесь к  `?AdfRangePartitionColumnName` предложению WHERE. Пример см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) . | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionUpbound` в предложении WHERE. Пример см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) . | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionLowbound` в предложении WHERE. Пример см. в разделе [Параллельное копирование из Oracle](#parallel-copy-from-oracle) . | Нет |

**Пример. копирование данных с помощью простого запроса без секции**

```json
"activities":[
    {
        "name": "CopyFromOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Oracle input dataset name>",
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
                "type": "OracleSource",
                "oracleReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="oracle-as-sink"></a>Oracle в качестве приемника

Чтобы скопировать данные в Oracle, задайте для типа приемника в действии копирования значение `OracleSink` . В разделе **sink** действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение `OracleSink` . | Да |
| writeBatchSize | Вставляет данные в таблицу SQL по достижении размера буфера `writeBatchSize` .<br/>Допустимые значения: целое число (количество строк). |Нет (значение по умолчанию — 10 000) |
| writeBatchTimeout | Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия.<br/>Допустимые значения: временной диапазон. Например, 00:30:00 (30 минут). | Нет |
| preCopyScript | Укажите SQL-запрос для выполнения действия копирования, прежде чем записывать данные в Oracle при каждом запуске. Это свойство можно использовать для очистки предварительно загруженных данных. | Нет |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyToOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Oracle output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "OracleSink"
            }
        }
    }
]
```

## <a name="parallel-copy-from-oracle"></a>Параллельная копия из Oracle

Соединитель Oracle фабрики данных предоставляет встроенное секционирование данных для параллельного копирования данных из Oracle. Параметры секционирования данных можно найти на вкладке **источник** действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-oracle/connector-oracle-partition-options.png)

При включении секционированной копии фабрика данных выполняет параллельные запросы к источнику Oracle для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если установлено значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из базы данных Oracle.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из базы данных Oracle. Ниже приведены рекомендуемые конфигурации для разных сценариев. Если копирование данных выполняется в файловое хранилище данных, мы рекомендуем сохранять данные в папку несколькими файлами (указывая только имя папки), так как производительность в таком случае будет выше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы с физическими секциями.          | **Параметр секции**: физические разделы таблицы. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает физические секции и копирует данные по секциям. |
| Полная загрузка из большой таблицы без физических секций с целочисленным столбцом для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Столбец секционирования**: укажите столбец для секционирования данных. Если не указано, используется первичный ключевой столбец. |
| Загрузка большого объема данных с помощью пользовательского запроса с физическими секциями. | **Параметр секции**: физические разделы таблицы.<br>**Запрос**: `SELECT * FROM <TABLENAME> PARTITION("?AdfTabularPartitionName") WHERE <your_additional_where_clause>`.<br>**Имя секции**: укажите имена разделов, из которых следует копировать данные. Если не указано, фабрика данных автоматически обнаруживает физические секции в таблице, указанной в наборе данных Oracle.<br><br>Во время выполнения фабрика данных заменяет `?AdfTabularPartitionName` фактическое имя секции и отправляется в Oracle. |
| Загрузка большого объема данных с помощью пользовательского запроса без физических секций, а с целочисленным столбцом для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных. Секционирование можно выполнять по столбцу с целочисленным типом данных.<br>**Верхняя граница секции** и **Нижняя граница секции**: укажите, нужно ли выполнять фильтрацию по столбцу секционирования, чтобы получать данные только между нижним и верхним диапазонами.<br><br>Во время выполнения фабрика данных заменяет `?AdfRangePartitionColumnName` , `?AdfRangePartitionUpbound` и `?AdfRangePartitionLowbound` фактическим именем столбца и диапазонами значений для каждой секции и отправляется в Oracle. <br>Например, если для столбца секции "ID" задана Нижняя граница, равная 1, а верхняя граница — 80, а для параллельного копирования — значение 4, фабрика данных извлекает данные на 4 секции. Для них будут применены следующие диапазоны значений идентификаторов: [1, 20], [21, 40], [41, 60] и [61, 80]. |

> [!TIP]
> При копировании данных из несекционированной таблицы можно использовать параметр секции "динамический диапазон" для секционирования по целочисленному столбцу. Если исходные данные не имеют такого типа столбца, можно использовать функцию [ORA_HASH]( https://docs.oracle.com/database/121/SQLRF/functions136.htm) в исходном запросе, чтобы создать столбец и использовать его в качестве столбца секционирования.

**Пример: запрос с физической секцией**

```json
"source": {
    "type": "OracleSource",
    "query": "SELECT * FROM <TABLENAME> PARTITION(\"?AdfTabularPartitionName\") WHERE <your_additional_where_clause>",
    "partitionOption": "PhysicalPartitionsOfTable",
    "partitionSettings": {
        "partitionNames": [
            "<partitionA_name>",
            "<partitionB_name>"
        ]
    }
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "OracleSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="data-type-mapping-for-oracle"></a>Сопоставление типов данных для Oracle

При копировании данных из и в Oracle применяются следующие сопоставления. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных Oracle | Тип промежуточных данных фабрики данных |
|:--- |:--- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(поддерживается только в Oracle 10g и более поздних версий) |
| CHAR |Строка |
| CLOB |Строка |
| DATE |Дата/время |
| FLOAT |десятичное число, строка (если точность больше 28) |
| INTEGER |десятичное число, строка (если точность больше 28) |
| LONG |Строка |
| LONG RAW |Byte[] |
| NCHAR |Строка |
| NCLOB |Строка |
| NUMBER (p,s) |Decimal, строка (если p > 28) |
| ЧИСЛО без точности и масштаба |Double |
| NVARCHAR2 |Строка |
| RAW |Byte[] |
| ROWID |Строка |
| timestamp |Дата/время |
| TIMESTAMP WITH LOCAL TIME ZONE |Строка |
| TIMESTAMP WITH TIME ZONE |Строка |
| UNSIGNED INTEGER |Число |
| VARCHAR2 |Строка |
| XML |Строка |

> [!NOTE]
> Типы данных INTERVAL YEAR TO MONTH и INTERVAL DAY TO SECOND не поддерживаются.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных.
