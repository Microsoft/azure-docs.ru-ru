---
title: Перемещение данных из источников OData
description: Узнайте, как перемещать данные из источников OData с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: fae78459a752d78fe47f189bca67667e917ba561
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100380072"
---
# <a name="move-data-from-an-odata-source-using-azure-data-factory"></a>Перемещение данных из источника OData с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-odata-connector.md)
> * [Версия 2 (текущая)](../connector-odata.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе OData в службе "Фабрика данных Azure" версии 2](../connector-odata.md).


В этой статье описывается, как с помощью действия копирования в фабрике данных Azure перемещать данные из источника OData. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

Данные из источника OData можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, поддерживаемых действием копирования в качестве приемников, см. в таблице [Поддерживаемые хранилища данных](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Сейчас фабрика данных поддерживает только перемещение данных из источника OData в другие хранилища данных, но не наоборот.

## <a name="supported-versions-and-authentication-types"></a>Поддерживаемые версии и типы аутентификации
Этот соединитель OData поддерживает OData версии 3.0 и 4.0 и может копировать данные как из облачных, так и из локальных источников OData. Для копирования из локальных источников необходимо установить шлюз управления данными. Сведения о шлюзе управления данными см. в статье [Перемещение данных между локальными источниками и облаком](data-factory-move-data-between-onprem-and-cloud.md).

Далее приведены поддерживаемые типы аутентификации:

* Чтобы получить доступ к **облачному** каналу OData, можно использовать анонимную, обычную (имя пользователя и пароль) аутентификацию или аутентификацию OAuth на основе Azure Active Directory.
* Чтобы получить доступ к **локальному** каналу OData, можно использовать анонимную, обычную (имя пользователя и пароль) аутентификацию или аутентификацию Windows.

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, который перемещает данные из источника OData, с помощью разных инструментов и интерфейсов API.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования.
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON.  Пример определений JSON для сущностей фабрики данных, которые используются для копирования данных из источника OData, доступен в разделе [Пример JSON. Копирование данных из источника OData в большой двоичный объект Azure](#json-example-copy-data-from-odata-source-to-azure-blob) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к источнику OData.

## <a name="linked-service-properties"></a>Свойства связанной службы
В приведенной далее таблице содержится описание элементов JSON, которые относятся к связанной службе OData.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Для свойства Type необходимо задать значение **OData** . |Да |
| url |URL-адрес службы OData. |Да |
| authenticationType |Тип проверки подлинности, используемый для подключения к источнику OData. <br/><br/> Возможные значения для облачной службы OData: "Анонимно", "Обычная" и "OAuth" (обратите внимание, что сейчас фабрика данных Azure поддерживает только аутентификацию OAuth на основе Azure Active Directory). <br/><br/> Для локальной службы OData возможными значениями являются "Анонимная", "Обычная" и "Windows". |Да |
| username |При использовании обычной проверки подлинности укажите имя пользователя. |Да (только при использовании обычной проверки подлинности) |
| password |Введите пароль для учетной записи пользователя, указанной для выбранного имени пользователя. |Да (только при использовании обычной проверки подлинности) |
| authorizedCredential |Если используется OAuth, в мастере копирования фабрики данных или редакторе нажмите кнопку **Авторизовать** и введите свои учетные данные, после чего значение этого свойства будет создано автоматически. |Да (только при использовании аутентификации OAuth) |
| gatewayName |Имя шлюза, который следует использовать службе фабрики данных для подключения к локальной службе OData. Указывайте, только если вы копируете данные из локального источника OData. |Нет |

### <a name="using-basic-authentication"></a>Использовать обычную проверку подлинности
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
        "typeProperties":
        {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Basic",
            "username": "username",
            "password": "password"
        }
    }
}
```

### <a name="using-anonymous-authentication"></a>Использовать анонимную проверку подлинности
```json
{
    "name": "ODataLinkedService",
    "properties":
    {
        "type": "OData",
        "typeProperties":
        {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

### <a name="using-windows-authentication-accessing-on-premises-odata-source"></a>Использование проверки подлинности Windows при получении доступа к локальному источнику OData
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
        "typeProperties":
        {
            "url": "<endpoint of on-premises OData source e.g. Dynamics CRM>",
            "authenticationType": "Windows",
            "username": "domain\\user",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-oauth-authentication-accessing-cloud-odata-source"></a>Использование аутентификации OAuth при получении доступа к облачному источнику OData
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of cloud OData source e.g. https://<tenant>.crm.dynamics.com/XRMServices/2011/OrganizationData.svc>",
            "authenticationType": "OAuth",
            "authorizedCredential": "<auto generated by clicking the Authorize button on UI>"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел **typeProperties** отличается для каждого типа набора данных и предоставляет сведения о расположении данных в хранилище данных. Раздел typeProperties набора данных типа **ODataResource** (который включает в себя набор данных OData) содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| path |Путь к ресурсу OData |Нет |

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

С другой стороны, свойства, доступные в разделе typeProperties действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

Если источник относится к типу **RelationalSource** (который содержит OData), в разделе typeProperties доступны следующие свойства.

| Свойство. | Описание | Пример | Обязательно |
| --- | --- | --- | --- |
| query |Используйте пользовательский запрос для чтения данных. |"?$select=Name, Description&$top=5" |Нет |

## <a name="type-mapping-for-odata"></a>Сопоставление типов для OData
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , действие копирования выполняет автоматическое преобразование типов из исходных типов в типы приемников с помощью следующего пошагового подхода.

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

При перемещении данных из OData Business Warehouse для преобразования типов OData Business Warehouse в типы .NET используются следующие сопоставления.

| Тип данных OData | Тип .NET |
| --- | --- |
| Edm.Binary |Byte[] |
| Edm.Boolean |Bool |
| Edm.Byte |Byte[] |
| Edm.DateTime |Дата и время |
| Edm.Decimal |Decimal |
| Edm.Double |Double |
| Edm.Single |Один |
| Edm.Guid |Guid |
| Edm.Int16 |Int16 |
| Edm.Int32 |Int32 |
| Edm.Int64 |Int64 |
| Edm.SByte |Int16 |
| Edm.String |Строка |
| Edm.Time |TimeSpan |
| Edm.DateTimeOffset |DateTimeOffset |

> [!Note]
> Сложные типы данных OData, например объекты, не поддерживаются.

## <a name="json-example-copy-data-from-odata-source-to-azure-blob"></a>Пример JSON. Копирование данных из источника OData в большой двоичный объект Azure
В этом примере приводятся образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Вы узнаете, как копировать данные из источника OData в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать в любой из указанных [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемников. Это делается с помощью действия копирования в фабрике данных Azure. Пример содержит следующие сущности фабрики данных.

1. Связанная служба типа [OData](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [ODataResource](#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [RelationalSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В примере данные из запросов к источнику OData каждый час копируются в BLOB-объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба OData**. В этом примере используется анонимный доступ. Сведения о различных типах проверки подлинности, которые можно использовать, см. в разделе [Свойства связанной службы OData](#linked-service-properties).

```json
{
    "name": "ODataLinkedService",
    "properties":
    {
        "type": "OData",
        "typeProperties":
        {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

**Связанная служба хранилища Azure:**

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

**Входной набор данных OData**

Если параметру external присвоить значение true, то фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой фабрике данных.

```json
{
    "name": "ODataDataset",
    "properties":
    {
        "type": "ODataResource",
        "typeProperties":
        {
            "path": "Products"
        },
        "linkedServiceName": "ODataLinkedService",
        "structure": [],
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
        }
    }
}
```

Указывать **path** в определении набора данных необязательно.

**Выходной набор данных большого двоичного объекта Azure:**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```json
{
    "name": "AzureBlobODataDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/odata/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Действие копирования в конвейере с OData в качестве источника и большим двоичным объектом в качестве приемника**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **RelationalSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, заданный для свойства **query** , выбирает самые последние (новейшие) данные из источника OData.

```json
{
    "name": "CopyODataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "?$select=Name, Description&$top=5",
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "ODataDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobODataDataSet"
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
                "name": "ODataToBlob"
            }
        ],
        "start": "2017-02-01T18:00:00Z",
        "end": "2017-02-03T19:00:00Z"
    }
}
```

Указывать **query** в определении конвейера необязательно. **URL-адрес** , который служба фабрики данных использует для извлечения данных, имеет следующий вид: URL-адрес, указанный в связанной службе (обязательно) + путь, указанный в наборе данных (необязательно) + запрос в конвейере (необязательно).

### <a name="type-mapping-for-odata"></a>Сопоставление типов для OData
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md), во время копирования типы источников автоматически преобразовываются в типы приемников. Такое преобразование выполняется в два этапа.

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

При перемещении данных из хранилищ данных OData типы данных OData сопоставляются с типами .NET.

## <a name="map-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника
Дополнительные сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Повторяющиеся операции чтения из реляционных источников
При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных политику повтора, чтобы при сбое срез выполнялся повторно. При повторном выполнении среза в любом случае необходимо убедиться в том, что считываются те же данные, независимо от того, сколько раз выполняется срез. См. раздел [повторяемые операции чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
