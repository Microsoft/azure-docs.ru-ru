---
title: Копирование данных в Azure Databricks Дельта Lake
description: Узнайте, как копировать данные в Azure Databricks Delta Lake и обратно с помощью действия копирования в конвейере фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/24/2020
ms.openlocfilehash: bdf71276d59dec9a19e29ae7f49cb92a0512c05a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100364245"
---
# <a name="copy-data-to-and-from-azure-databricks-delta-lake-by-using-azure-data-factory"></a>Копирование данных в Azure Databricks Delta Lake с помощью фабрики данных Azure и обратно

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные в Azure Databricks Delta Lake и обратно. Эта статья посвящена [действию копирования в фабрике данных Azure](copy-activity-overview.md) , в котором представлен общий обзор действий копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот Azure Databricks-соединитель дельты Lake поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с [поддерживаемой таблицей источника или приемника](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Как правило, фабрика данных Azure поддерживает разностную версию Lake со следующими возможностями в соответствии с различными потребностями.

- Действие копирования поддерживает Azure Databricks Дельта Lake Connector для копирования данных из любого поддерживаемого исходного хранилища данных в Azure Databricks разностной таблицы и из разностной таблицы в любое поддерживаемое хранилище данных, которое является приемником. Он использует кластер модуля данных для выполнения перемещения данных, см. сведения в [разделе Предварительные требования](#prerequisites).
- [Поток данных сопоставления](concepts-data-flow-overview.md) поддерживает общий [разностный формат](format-delta.md) в службе хранилища Azure в качестве источника и приемника для чтения и записи разностных файлов для ETL без кода и выполняется на управляемых Azure Integration Runtime.
- [Действия модуля](transform-data-databricks-notebook.md) обработки данных поддерживают координацию рабочей нагрузки ETL или машинного обучения на основе кода в разностной версии Lake.

## <a name="prerequisites"></a>Предварительные условия

Чтобы использовать этот Azure Databricks разностного соединителя, необходимо настроить кластер в Azure Databricks.

- Чтобы скопировать данные в разностную копию, действие копирования вызывает Azure Databricks кластер для чтения данных из службы хранилища Azure, которая является исходным источником или промежуточной областью, где фабрика данных впервые записывает исходные данные через встроенную промежуточную копию. Узнайте больше от [дельты Lake в качестве приемника](#delta-lake-as-sink).
- Аналогично, чтобы скопировать данные из разностной копии, действие копирования вызывает Azure Databricks кластер для записи данных в службу хранилища Azure, которая является исходным приемником или промежуточной областью, из которой фабрика данных продолжит записывать данные в окончательный приемник через встроенную промежуточную копию. Узнайте больше от [дельты Lake в качестве источника](#delta-lake-as-source).

Для кластера блоков данных необходим доступ к большому двоичному объекту Azure или учетной записи Azure Data Lake Storage 2-го поколения — контейнеру хранилища или файловой системе, используемой для источника, приемника, промежуточного хранения и контейнера или файловой системы, в которой необходимо записать разностные таблицы.

- Чтобы использовать **Azure Data Lake Storage 2-го поколения**, можно настроить ключ доступа **субъекта-службы** или **учетной записи хранения** в кластере кирпичей в составе конфигурации Apache Spark. Выполните инструкции из раздела [доступ напрямую с субъектом-службой](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-with-service-principal-and-oauth-20) или [напрямую с помощью ключа доступа к учетной записи хранения](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-using-the-storage-account-access-key).

- Чтобы использовать **хранилище BLOB-объектов Azure**, можно настроить **ключ доступа к учетной записи хранения** или **токен SAS** в кластере кирпичей в составе конфигурации Apache Spark. Выполните действия, описанные в статье [доступ к хранилищу BLOB-объектов Azure с помощью API RDD](/azure/databricks/data/data-sources/azure/azure-storage#access-azure-blob-storage-using-the-rdd-api).

При выполнении действия копирования, если настроенный кластер был завершен, фабрика данных автоматически запускает ее. Если вы создаете конвейер с помощью пользовательского интерфейса создания фабрики данных, для таких операций, как предварительная версия данных, необходим динамический кластер, фабрика данных не будет запускать кластер от вашего имени.

#### <a name="specify-the-cluster-configuration"></a>Укажите конфигурацию кластера

1. В раскрывающемся списке **режим кластера** выберите **стандартный**.

2. В раскрывающемся списке **версия Databricks Runtime** выберите версию среды выполнения для модуля сбора.

3. Включите [автоматическую оптимизацию](/azure/databricks/delta/optimizations/auto-optimize) , добавив следующие свойства в [конфигурацию Spark](/azure/databricks/clusters/configure#spark-config):

   ```
   spark.databricks.delta.optimizeWrite.enabled true
   spark.databricks.delta.autoCompact.enabled true
   ```

4. Настройте кластер в зависимости от потребностей в интеграции и масштабировании.

Сведения о конфигурации кластера см. в разделе [Настройка кластеров](/azure/databricks/clusters/configure).

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, определяющих сущности фабрики данных, относящиеся к Azure Databricks разностного соединителя.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Azure Databricks Delta Lake поддерживаются следующие свойства.

| Свойство.    | Описание                                                  | Обязательно |
| :---------- | :----------------------------------------------------------- | :------- |
| type        | Для свойства Type необходимо задать значение **азуредатабрикксделталаке**. | Да      |
| домен      | Укажите URL-адрес рабочей области Azure Databricks, например `https://adb-xxxxxxxxx.xx.azuredatabricks.net` . |          |
| clusterId   | Укажите идентификатор кластера для существующего кластера. Это должен быть уже созданный интерактивный кластер. <br>Идентификатор интерактивного кластера можно найти, выбрав "Рабочая область Databricks -> Кластеры -> Имя интерактивного кластера -> Конфигурация -> Теги". [Подробнее](/azure/databricks/clusters/configure#cluster-tags). |          |
| accessToken | Чтобы фабрика данных прошла аутентификацию в Azure Databricks, необходим маркер доступа. Маркер доступа должен быть создан в рабочей области Databricks. Более подробные инструкции по поиску маркера доступа можно найти [здесь](/azure/databricks/dev-tools/api/latest/authentication#generate-token). |          |
| connectVia  | [Среда выполнения интеграции](concepts-integration-runtime.md) , используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду выполнения интеграции (если хранилище данных находится в частной сети). Если не указано, используется среда выполнения интеграции Azure по умолчанию. | Нет       |

**Пример**.

```json
{
    "name": "AzureDatabricksDeltaLakeLinkedService",
    "properties": {
        "type": "AzureDatabricksDeltaLake",
        "typeProperties": {
            "domain": "https://adb-xxxxxxxxx.xx.azuredatabricks.net",
            "clusterId": "<cluster id>",
            "accessToken": {
                "type": "SecureString", 
                "value": "<access token>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). 

Для Azure Databricks Delta Lake DataSet поддерживаются следующие свойства.

| Свойство.  | Описание                                                  | Обязательно                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | Свойство Type набора данных должно иметь значение **азуредатабрикксделталакедатасет**. | Да                         |
| База данных | Имя базы данных. |Нет для источника, да для приемника  |
| table | Имя разностной таблицы. |Нет для источника, да для приемника  |

**Пример**.

```json
{
    "name": "AzureDatabricksDeltaLakeDataset",
    "properties": {
        "type": "AzureDatabricksDeltaLakeDataset",
        "typeProperties": {
            "database": "<database name>",
            "table": "<delta table name>"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "linkedServiceName": {
            "referenceName": "<name of linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых Azure Databricks разностным и приемником.

### <a name="delta-lake-as-source"></a>Дельта Lake в качестве источника

Чтобы скопировать данные из Azure Databricks Delta Lake, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство.                     | Описание                                                  | Обязательно |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | Свойство Type источника действия копирования должно иметь значение **азуредатабрикксделталакесаурце**. | Да      |
| query          | Укажите SQL запрос для чтения данных. Для элемента управления командировкой по времени используйте следующий шаблон:<br>- `SELECT * FROM events TIMESTAMP AS OF timestamp_expression`<br>- `SELECT * FROM events VERSION AS OF version` | Нет       |
| exportSettings | Дополнительные параметры, используемые для получения данных из разностной таблицы. | Нет       |
| ***В разделе `exportSettings` :*** |  |  |
| тип | Тип команды экспорта, установленный в **азуредатабрикксделталакикспорткомманд**. | Да |
| dateFormat | Форматирование типа Date до String с форматом даты. Пользовательские форматы дат соответствуют форматам по [шаблону даты и времени](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Если не указано, используется значение по умолчанию `yyyy-MM-dd` . | Нет |
| тиместампформат | Форматирование типа timestamp в String с использованием формата метки времени. Пользовательские форматы дат соответствуют форматам по [шаблону даты и времени](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Если не указано, используется значение по умолчанию `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]` . | Нет |

#### <a name="direct-copy-from-delta-lake"></a>Непосредственная копия из разностного Lake

Если хранилище данных приемника и формат соответствуют критериям, описанным в этом разделе, можно использовать действие копирования для непосредственного копирования из Azure Databricks разностной таблицы в приемник. Фабрика данных проверяет параметры и не выполняет действие копирования, если не выполняются следующие условия.

- **Связанная служба приемника** — это [хранилище BLOB-объектов Azure](connector-azure-blob-storage.md) или [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md). Учетные данные учетной записи должны быть предварительно настроены в конфигурации кластера Azure Databricks. Дополнительные сведения см. в статье [Предварительные требования](#prerequisites).

- **Формат данных приемника** — **Parquet**, **текст с разделителями** или **Avro** со следующими конфигурациями и указывает на папку, а не на файл.

    - Для формата **Parquet** кодек сжатия — **None**, **привязки** или **gzip**.
    - Для **текстового формата с разделителями** :
        - `rowDelimiter` любой отдельный символ.
        - `compression` может иметь **значение None**, **bzip2**, **gzip**.
        - `encodingName` UTF-7 не поддерживается.
    - Для формата **Avro** кодек сжатия — **None**, **Deflate** или **привязки**.

- В источнике действия копирования `additionalColumns` не указано.
- При копировании данных в текст с разделителями в приемнике действия копирования `fileExtension` необходимо указать CSV-файл.
- В сопоставлении действия копирования тип преобразование не включено.

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
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
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-delta-lake"></a>Промежуточное копирование из Delta Lake

Если хранилище данных приемника или формат не соответствуют критериям прямого копирования, как упоминалось в последнем разделе, включите встроенную промежуточную копию, используя временный экземпляр службы хранилища Azure. Промежуточное копирование также обеспечивает лучшую пропускную способность. Фабрика данных экспортирует данные из Azure Databricks Delta Lake в промежуточное хранилище, затем копирует данные в приемник и, наконец, очищает временные данные из промежуточного хранилища. Дополнительные сведения о копировании данных с помощью промежуточного хранения см. в разделе [поэтапное копирование](copy-activity-performance-features.md#staged-copy) .

Чтобы использовать эту функцию, создайте [связанную службу хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [связанную службу Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) , которая ссылается на учетную запись хранения в качестве промежуточного промежуточного хранения. Затем укажите `enableStaging` Свойства и `stagingSettings` в действии копирования.

>[!NOTE]
>Учетные данные промежуточного хранения должны быть предварительно настроены в Azure Databricks конфигурации кластера. Дополнительные сведения см. в статье [Предварительные требования](#prerequisites).

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
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
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

### <a name="delta-lake-as-sink"></a>Дельта Lake в качестве приемника

Чтобы скопировать данные в Azure Databricks Delta Lake, в разделе **приемника** действия копирования поддерживаются следующие свойства.

| Свойство.      | Описание                                                  | Обязательно |
| :------------ | :----------------------------------------------------------- | :------- |
| type          | Свойство Type приемника действия копирования, для которого задано значение **азуредатабрикксделталакесинк**. | Да      |
| preCopyScript | Укажите SQL-запрос для выполнения действия копирования, прежде чем записывать данные в разностную таблицу кирпичей данных в каждом запуске. Это свойство можно использовать для очистки предварительно загруженных данных или для добавления усечения таблицы или инструкции чистильщика. | Нет       |
| importSettings | Дополнительные параметры, используемые для записи данных в разностную таблицу. | Нет |
| ***В разделе `importSettings` :*** |                                                              |  |
| тип | Тип команды импорта, установленный в **азуредатабрикксделталакеимпорткомманд**. | Да |
| dateFormat | Форматирование строки до типа Date с использованием формата даты. Пользовательские форматы дат соответствуют форматам по [шаблону даты и времени](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Если не указано, используется значение по умолчанию `yyyy-MM-dd` . | Нет |
| тиместампформат | Форматирование строки до типа timestamp с использованием формата метки времени. Пользовательские форматы дат соответствуют форматам по [шаблону даты и времени](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Если не указано, используется значение по умолчанию `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]` . | Нет |

#### <a name="direct-copy-to-delta-lake"></a>Прямое копирование в разностную версию Lake

Если исходное хранилище данных и формат соответствуют критериям, описанным в этом разделе, можно использовать действие копирования для непосредственного копирования из источника в Azure Databricks Delta Lake. Фабрика данных Azure проверяет параметры и завершает выполнение действия копирования, если не выполняются следующие условия.

- **Связанная служба источника** — [хранилище BLOB-объектов Azure](connector-azure-blob-storage.md) или [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md). Учетные данные учетной записи должны быть предварительно настроены в конфигурации кластера Azure Databricks. Дополнительные сведения см. в статье [Предварительные требования](#prerequisites).

- **Исходный формат данных** — **Parquet**, текст с **разделителями** или **Avro** со следующими конфигурациями и указывает на папку, а не на файл.

    - Для формата **Parquet** кодек сжатия — **None**, **привязки** или **gzip**.
    - Для **текстового формата с разделителями** :
        - `rowDelimiter` значение по умолчанию или любой отдельный символ.
        - `compression` может иметь **значение None**, **bzip2**, **gzip**.
        - `encodingName` UTF-7 не поддерживается.
    - Для формата **Avro** кодек сжатия — **None**, **Deflate** или **привязки**.

- В источнике действия копирования: 

    - `wildcardFileName` содержит только подстановочный знак `*` , но не `?` `wildcardFolderName` указан и.
    - Параметры `prefix`, `modifiedDateTimeStart`, `modifiedDateTimeEnd` и `enablePartitionDiscovery` не указываются.
    - `additionalColumns` не задан.

- В сопоставлении действия копирования тип преобразование не включено.

**Пример**.

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            }
        }
    }
]
```

#### <a name="staged-copy-to-delta-lake"></a>Промежуточное копирование в разностную версию Lake

Если исходное хранилище данных или формат не соответствуют критериям прямого копирования, как упоминалось в последнем разделе, включите встроенную промежуточную копию, используя временный экземпляр службы хранилища Azure. Промежуточное копирование также обеспечивает лучшую пропускную способность. Фабрика данных автоматически преобразует данные в соответствии с требованиями к формату данных в промежуточное хранилище, а затем загружает данные в разностную среду Lake. Наконец, он очищает временные данные из хранилища. Дополнительные сведения о копировании данных с помощью промежуточного хранения см. в разделе [поэтапное копирование](copy-activity-performance-features.md#staged-copy) .

Чтобы использовать эту функцию, создайте [связанную службу хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [связанную службу Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) , которая ссылается на учетную запись хранения в качестве промежуточного промежуточного хранения. Затем укажите `enableStaging` Свойства и `stagingSettings` в действии копирования.

>[!NOTE]
>Учетные данные промежуточного хранения должны быть предварительно настроены в Azure Databricks конфигурации кластера. Дополнительные сведения см. в статье [Предварительные требования](#prerequisites).

**Пример**.

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

## <a name="monitoring"></a>Мониторинг

Фабрика данных Azure предоставляет те же [возможности мониторинга действий копирования](copy-activity-monitoring.md) , что и другие соединители. Кроме того, так как загрузка данных из или в разностную версию Lake выполняется в кластере Azure Databricks, можно [Просмотреть подробные журналы кластера](/azure/databricks/clusters/clusters-manage#--view-cluster-logs) и [наблюдать за производительностью](/azure/databricks/clusters/clusters-manage#--monitor-performance).

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Дополнительные сведения о свойствах см. в разделе [действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия

Список хранилищ данных, поддерживаемых в качестве источников и приемников в рамках действия копирования в фабрике данных, см. в разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).
