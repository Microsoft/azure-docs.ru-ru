---
title: Перемещение данных из SAP HANA с помощью фабрики данных Azure
description: Узнайте, как перемещать данные из SAP HANA с использованием фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
robots: noindex
ms.openlocfilehash: 9c192a5f84f36fbdea0dd1d280928aa40186d4ee
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100386991"
---
# <a name="move-data-from-sap-hana-using-azure-data-factory"></a>Перемещение данных из SAP HANA с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-sap-hana-connector.md)
> * [Версия 2 (текущая)](../connector-sap-hana.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе SAP HANA в службе "Фабрика данных Azure" версии 2](../connector-sap-business-warehouse.md).

В этой статье рассказывается, как с помощью действия копирования в фабрике данных Azure перемещать данные из локального экземпляра SAP HANA. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

Вы можете скопировать данные из локального хранилища данных SAP HANA в любой поддерживаемый приемник данных. Список хранилищ данных, поддерживаемых действием копирования в качестве приемников, см. в таблице [Поддерживаемые хранилища данных](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Сейчас фабрика данных поддерживает перемещение данных из SAP HANA в другие хранилища данных, но не наоборот.

## <a name="supported-versions-and-installation"></a>Поддерживаемые версии и установка
Этот соединитель поддерживает все версии базы данных SAP HANA. Он поддерживает копирование данных из информационных моделей HANA (например, представлений Analytic и Calculation) и таблиц со строками и столбцами с помощью SQL-запросов.

Чтобы обеспечить возможность подключения к экземпляру SAP HANA, установите следующие компоненты.
- **Шлюз управления данными**. Служба фабрики данных поддерживает подключение к локальным хранилищам данным (включая SAP HANA) с помощью компонента, который называется шлюзом управления данными. В статье [Перемещение данных между локальными и облачными ресурсами](data-factory-move-data-between-onprem-and-cloud.md) приведены сведения о шлюзе управления данными и пошаговые инструкции по его настройке. Шлюз является обязательным, даже если SAP HANA размещается на виртуальной машине IaaS Azure. Шлюз можно установить на той же ВМ, на которой размещается хранилище данных, или на другой ВМ. Важно, чтобы шлюз мог подключиться к базе данных.
- **Драйвер ODBC для SAP HANA** на компьютере шлюза. Драйвер ODBC для SAP HANA можно скачать на странице [SAP Software Download Center](https://support.sap.com/swdc) (Центр загрузки программного обеспечения SAP). Выполните поиск по ключевой фразе **SAP HANA CLIENT for Windows**. 

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, которое перемещает данные из локального хранилища данных SAP HANA, с помощью разных средств и API-интерфейсов. 

- Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных. 
- Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) . 

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. 
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. 

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON.  Пример определения JSON для сущностей фабрики данных, которые используются для копирования данных из локального хранилища данных SAP HANA, вы найдете в разделе [Пример JSON. Копирование данных из SAP HANA в большой двоичный объект Azure](#json-example-copy-data-from-sap-hana-to-azure-blob) далее в этой статье. 

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для хранилища данных SAP HANA.

## <a name="linked-service-properties"></a>Свойства связанной службы
В следующей таблице содержится описание элементов JSON, которые относятся к связанной службе SAP HANA.

Свойство. | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
server | Имя сервера, на котором размещен экземпляр SAP HANA. Если ваш сервер использует настроенный порт, укажите `server:port`. | строка | Да
authenticationType | Тип проверки подлинности. | . Basic или Windows. | Да 
username | Имя пользователя, имеющего доступ к серверу SAP. | строка | Да
password | Пароль для пользователя | строка | Да
gatewayName | Имя шлюза, который следует использовать службе фабрики данных для подключения к локальному экземпляру SAP HANA. | строка | Да
encryptedCredential | Строка зашифрованных учетных данных. | Строка | Нет

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел **typeProperties** отличается для каждого типа набора данных и предоставляет сведения о расположении данных в хранилище данных. Для набора данных SAP HANA типа **RelationalTable** не поддерживаются какие-либо свойства типа. 


## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (такие как имя, описание, входные и выходные таблицы, политики и т. д.) доступны для всех типов действий.

В свою очередь свойства, доступные в разделе **typeProperties** действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

Если источник действия копирования относится к типу **RelationalSource** (к которому относится SAP HANA), в разделе typeProperties доступны следующие свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| query | Указывает SQL-запрос для чтения данных из экземпляра SAP HANA. | SQL-запрос. | Да |

## <a name="json-example-copy-data-from-sap-hana-to-azure-blob"></a>Пример JSON. Копирование данных из SAP HANA в большой двоичный объект Azure
В следующем примере приводятся образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В этом примере показано, как скопировать данные из локального экземпляра SAP HANA в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать **непосредственно** в любой из указанных [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемников. Это делается с помощью действия копирования в фабрике данных Azure.  

> [!IMPORTANT]
> Этот пример содержит фрагменты кода JSON. Он не включает в себя пошаговые инструкции по созданию фабрики данных. Эти инструкции приведены в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md) .

Образец состоит из следующих сущностей фабрики данных.

1. Связанная служба типа [SapHana](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [RelationalTable](#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [RelationalSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

Пример копирует данные из экземпляра SAP HANA в большой двоичный объект Azure раз в час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

Сначала настройте шлюз управления данными. Инструкции приведены в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md) .

### <a name="sap-hana-linked-service"></a>Связанная служба SAP HANA
Это связанная служба связывает экземпляр SAP HANA с фабрикой данных. Свойству type присваивается значение **SapHana**. Раздел typeProperties содержит сведения о подключении для экземпляра SAP HANA.

```json
{
    "name": "SapHanaLinkedService",
    "properties":
    {
        "type": "SapHana",
        "typeProperties":
        {
            "server": "<server name>",
            "authenticationType": "<Basic, or Windows>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}

```

### <a name="azure-storage-linked-service"></a>Связанная служба хранения Azure
Связанная служба связывает учетную запись хранения Azure с фабрикой данных. Для свойства type задано значение **AzureStorage** Раздел typeProperties содержит сведения о подключении для учетной записи хранения Azure.

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

### <a name="sap-hana-input-dataset"></a>Входной набор данных SAP HANA

Этот набор данных определяет набор данных SAP HANA. Для набора данных фабрики данных задается тип **RelationalTable**. В настоящее время какие-либо свойства типа для набора данных SAP HANA не указываются пользователем. Запрос в определении действия копирования указывает, какие данные следует считывать из экземпляра SAP HANA. 

Если для свойства external задать значение true, то фабрика данных воспримет эту таблицу как внешнюю, которая создана не в результате какого-либо действия в фабрике данных.

Свойства frequency и interval определяют расписание. В этом случае данные считываются из экземпляра SAP HANA раз в час. 

```json
{
    "name": "SapHanaDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapHanaLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

### <a name="azure-blob-output-dataset"></a>Выходной набор данных BLOB-объекта Azure
Этот набор данных определяет выходной набор данных больших двоичных объектов Azure. Для свойства type задано значение AzureBlob. В разделе typeProperties указывается, где сохраняются данные, скопированные из экземпляра SAP HANA. Данные записываются в новый большой двоичный объект раз в час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```json
{
    "name": "AzureBlobDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/saphana/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
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
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```


### <a name="pipeline-with-copy-activity"></a>Конвейер с действием копирования

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **RelationalSource** (для источника SAP HANA), а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный для свойства **query** , выбирает для копирования данные за последний час.

```json
{
    "name": "CopySapHanaToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "<SQL Query for HANA>"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "SapHanaDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SapHanaToBlob"
            }
        ],
        "start": "2017-03-01T18:00:00Z",
        "end": "2017-03-01T19:00:00Z"
    }
}
```


### <a name="type-mapping-for-sap-hana"></a>Сопоставление типов для SAP HANA
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md), во время копирования типы источников автоматически преобразовываются в типы приемников. Такое преобразование выполняется в два этапа:

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

При перемещении данных из SAP HANA для преобразования типов SAP HANA в типы .NET используются следующие сопоставления.

Тип SAP HANA | Тип данных на основе .NET
------------- | ---------------
TINYINT | Byte
SMALLINT | Int16
INT | Int32
bigint | Int64
real | Single
DOUBLE | Single
DECIMAL | Decimal
BOOLEAN | Byte
VARCHAR | Строка
NVARCHAR | Строка
CLOB | Byte[]
ALPHANUM | Строка
BLOB | Byte[]
DATE | Дата/время
TIME | TimeSpan
timestamp | Дата/время
SECONDDATE | Дата/время

## <a name="known-limitations"></a>Известные ограничения
Существует несколько известных ограничений, действующих при копировании данных из SAP HANA.

- Строки NVARCHAR усекаются до 4000 знаков Юникода.
- SMALLDECIMAL не поддерживается.
- VARBINARY не поддерживается.
- Допустимый диапазон дат: 30.12.1899–31.12.9999.

## <a name="map-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника
Дополнительные сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Повторяющиеся операции чтения из реляционных источников
При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных политику повтора, чтобы при сбое срез выполнялся повторно. При повторном выполнении среза в любом случае необходимо убедиться в том, что считываются те же данные, независимо от того, сколько раз выполняется срез. Ознакомьтесь с разделом [Повторяющиеся операции чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
