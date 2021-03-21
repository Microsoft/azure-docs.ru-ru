---
title: Перемещение данных из источника HTTP в Azure
description: Узнайте о перемещении данных из локального или облачного источника HTTP с помощью фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/22/2018
robots: noindex
ms.openlocfilehash: ce29b5a112d70575a721b0b527947fd95868da80
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382928"
---
# <a name="move-data-from-an-http-source-by-using-azure-data-factory"></a>Перемещение данных из источника HTTP с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-http-connector.md)
> * [Версия 2 (текущая)](../connector-http.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью [Копирование данных из источника HTTP с помощью фабрики данных Azure](../connector-http.md).

В этой статье описано, как с помощью действия копирования в фабрике данных Azure переместить данные из локальной или облачной конечной точки HTTP в поддерживаемый приемник данных. Эта статья основана на статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью копирования. В статье также приведен список хранилищ, поддерживаемых действием копирования в качестве источников и приемников данных.

В настоящее время фабрика данных Azure поддерживает только перемещение данных из источника HTTP в другие хранилища. Перемещение данных из других хранилищ в источник HTTP не поддерживается.

## <a name="supported-scenarios-and-authentication-types"></a>Поддерживаемые сценарии и типы аутентификации

Вы можете использовать этот соединитель HTTP, чтобы получить данные *из локальной и облачной конечных точек HTTP/S* с помощью методов HTTP **GET** или **POST**. Поддерживаемые типы аутентификации: **Anonymous**, **Basic**, **Digest**, **Windows** и **ClientCertificate**. Обратите внимание, как отличаются между собой этот соединитель и [соединитель веб-таблиц](data-factory-web-table-connector.md). Соединитель веб-таблиц извлекает содержимое таблицы со страницы HTML.

При копировании данных из локальной конечной точки HTTP вы должны установить шлюз управления данными в локальной среде или на виртуальной машине Azure. См. дополнительные сведения о шлюзе управления данными и пошаговые инструкции по его настройке в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md).

## <a name="get-started"></a>Начало работы

Вы можете создать конвейер с действием копирования, которое перемещает данные из источника HTTP с помощью разных инструментов и интерфейсов API:

- Проще всего создать конвейер с помощью мастера копирования данных. Краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных см. в статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md).

- Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **шаблон Azure Resource Manager**, **API .NET** или **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в статье [Руководство. Копирование данных из хранилища BLOB-объектов Azure в базу данных SQL с помощью фабрики данных](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). Примеры JSON для копирования данных из источника HTTP в хранилище BLOB-объектов Azure см. в разделе [Примеры JSON](#json-examples).

## <a name="linked-service-properties"></a>Свойства связанной службы

В приведенной ниже таблице описываются элементы JSON, которые относятся к связанной службе HTTP:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type | Свойству **Тип** необходимо задать значение **Http**. | Да |
| url | Базовый URL-адрес веб-сервера. | Да |
| authenticationType | Указывает тип проверки подлинности. Допустимые значения **Anonymous**, **Basic**, **Digest**, **Windows** и **ClientCertificate**. <br><br> С дополнительными свойствами и примерами JSON этих типов проверки подлинности ознакомьтесь в последующих разделах этой статьи. | Да |
| enableServerCertificateValidation | Указывает, следует ли включить проверку сертификата TLS или SSL на сервере, если источником является веб-сервер HTTPS. Если ваш сервер HTTPS использует самозаверенный сертификат, установите значение **false**. | Нет<br /> (значение по умолчанию — **true**) |
| gatewayName | Имя экземпляра шлюза управления данными для подключения к локальному источнику HTTP. | Да, если вы копируете данные из локального источника HTTP |
| encryptedCredential | Зашифрованные учетные данные для доступа к конечной точке HTTP. Значение создается автоматически при настройке сведений для проверки подлинности в мастере копирования или в диалоговом окне **ClickOnce**. | Нет<br /> (применимо, только когда копирование данных выполняется с локального HTTP-сервера) |

Дополнительные сведения о настройке учетных данных для локального источника данных соединителя HTTP см. в статье [Перемещение данных между локальными источниками и облаком используя шлюз управления данными](data-factory-move-data-between-onprem-and-cloud.md).

### <a name="using-basic-digest-or-windows-authentication"></a>Использование типов проверки подлинности Basic, Digest или Windows

Задайте параметру **authenticationType** значение **Basic**, **Digest** или **Windows**. В дополнение к общим свойствам соединителя HTTP, описанных в предыдущих разделах, установите следующие свойства:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| userName | Имя пользователя для доступа к конечной точке HTTP. | Да |
| password | Пароль пользователя (**username**). | Да |

**Пример. использование обычной, дайджест-или проверки подлинности Windows**

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "basic",
            "url" : "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

### <a name="using-clientcertificate-authentication"></a>Использование типа проверки подлинности ClientCertificate

Чтобы использовать базовую проверку подлинности установите **authenticationType** на **ClientCertificate**. В дополнение к общим свойствам соединителя HTTP, описанных в предыдущих разделах, установите следующие свойства:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| embeddedCertData | Содержимое двоичных данных PFX-файла с кодировкой Base64. | Укажите либо **ембеддедцертдата** , либо **certThumbprint** |
| certThumbprint | Отпечаток сертификата, установленного в хранилище сертификатов на компьютере шлюза. Применимо, только когда копирование данных выполняется из локального источника HTTP. | Укажите либо **ембеддедцертдата** , либо **certThumbprint** |
| password | Пароль, связанный с сертификатом. | Нет |

Используя **certThumbprint** для проверки подлинности и установив сертификат в личном хранилище локального компьютера, предоставьте разрешение на чтение для службы шлюза:

1. Откройте консоль управления (MMC). Добавьте оснастку **Сертификаты**, которая связана с **локальным компьютером**.
2. Разверните узел **Сертификаты**  >  **Личные**, а затем выберите **Сертификаты**.
3. Щелкните правой кнопкой мыши сертификат из личного хранилища, а затем выберите **все задачи**  > **Управление закрытыми ключами**.
3. На вкладке **Безопасность** добавьте учетную запись пользователя, под которой запущена служба узла шлюза управления данными с доступом на чтение к сертификату.  

**Пример: использование сертификата клиента**

Эта связанная служба связывает фабрику данных с локальным веб-сервером HTTP. Она использует сертификат клиента, установленный на компьютере со шлюзом управления данными.

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"
        }
    }
}
```

**Пример: использование сертификата клиента в файле**

Эта связанная служба связывает фабрику данных с локальным веб-сервером HTTP. Она использует файл сертификата клиента на компьютере, где установлен шлюз управления данными.

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "Base64-encoded cert data",
            "password": "password of cert"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Некоторые разделы файла JSON в наборе данных, такие как структура, доступность и политика, одинаковы для всех типов наборов данных (например, база данных SQL Azure, хранилище BLOB-объектов Azure и таблица хранилища Azure).

Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md).

Раздел **typeProperties** отличается для каждого типа набора данных. Раздел **typeProperties** содержит сведения о расположении данных в хранилище. Раздел **typeProperties** набора данных типа **Http** содержит перечисленные ниже свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство **type** для набора данных должно иметь значение **Http**. | Да |
| relativeUrl | Относительный URL-адрес ресурса, который содержит данные. Если путь не задан, используется только URL-адрес, указанный в определении связанной службы. <br><br> Для создания динамического URL-адреса можно использовать [функции фабрики данных и системные переменные](data-factory-functions-variables.md). Пример: **relativeUrl**: **$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)**. | Нет |
| requestMethod | Метод HTTP. Допустимые значения: **GET** и **POST**. | Нет <br />(значение по умолчанию — **GET**) |
| additionalHeaders | Дополнительные заголовки HTTP-запроса. | Нет |
| requestBody | Текст HTTP-запроса. | Нет |
| format | Если вы хотите *извлечь данные из конечной точки HTTP "как есть"*  — без анализа, пропустите параметр **format**. <br><br> Поддерживаемые форматы для выполнения анализа содержимого ответа HTTP в процессе выполнения операции копирования: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** и **ParquetFormat**. Дополнительные сведения см. в разделах о [Текстовый формат](data-factory-supported-file-and-compression-formats.md#text-format), [Формат JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Формат Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Формат ORC](data-factory-supported-file-and-compression-formats.md#orc-format) и [Формат Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). |Нет |
| compression | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Optimal** и **Fastest**. Узнайте больше о [форматах файлов и сжатия данных в фабрике данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). |Нет |

**Пример. использование метода GET (по умолчанию)**

```json
{
  "name": "HttpSourceDataInput",
    "properties": {
    "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
          "relativeUrl": "XXX/test.xml",
          "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

**Пример. использование метода POST**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
       "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

Полный список разделов и свойств, доступных для определения действий, см. в статье, посвященной [конвейерам и действиям в Фабрике данных Azure](data-factory-create-pipelines.md). 

Свойства, доступные в разделе действия **typeProperties**, зависят от типа действия. Для действия копирования свойства различаются в зависимости от типов источников и приемников.

В настоящее время, если источник в действии копирования относится к типу **HttpSource**, то поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
| -------- | ----------- | -------- |
| httpRequestTimeout | Время ожидания (значение **Временной диапазон**) ответа для HTTP-запроса. Это интервал времени для получения ответа, а не считывания данных ответа. | Нет<br />(значение по умолчанию: **00:01:40**) |

## <a name="supported-file-and-compression-formats"></a>Поддерживаемые форматы файлов и сжатия

Дополнительные сведения см. в статье [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

## <a name="json-examples"></a>Примеры JSON

Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Это пример процесса копирования данных из источника HTTP в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать *непосредственно* из любых источников в любой [поддерживаемый](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемник используя действие копирования в фабрике данных Azure.

**Пример: копирование данных из источника HTTP в хранилище BLOB-объектов Azure**

Решение фабрики данных для этого примера содержит следующие сущности фабрики данных.

*   Связанная служба типа [HTTP](#linked-service-properties).
*   Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
*   Входной [набор данных](data-factory-create-datasets.md) типа [Http](#dataset-properties).
*   Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
*   [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [HttpSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные из источника HTTP каждый час копируются в большой двоичный объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

### <a name="http-linked-service"></a>Связанная служба HTTP

В этом примере используется связанная служба HTTP с анонимной проверкой подлинности. Сведения о возможных типах аутентификации см. в разделе [Связанная служба HTTP](#linked-service-properties).

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Связанная служба хранилища Azure

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
    }
  }
}
```

### <a name="http-input-dataset"></a>Входной набор данных HTTP

Если параметру **external** присвоить значение **true**, фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой службе.

```json
{
  "name": "HttpSourceDataInput",
    "properties": {
    "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)",
        "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
    },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}

```

### <a name="azure-blob-output-dataset"></a>Выходной набор данных BLOB-объекта Azure

Данные записываются в новый большой двоичный объект каждый час (**Частота**: **час**, **интервал**: **1**).

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/Movies"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

### <a name="pipeline-that-uses-a-copy-activity"></a>Конвейер, использующий действие копирования

Конвейер содержит действие копирования, которое использует входные и выходные наборы данных. Действие копирования выполняется по расписанию каждый час. В определении конвейера JSON для **source** установлено значение типа **HttpSource**, а для **sink** — значение типа **BlobSink**.

Список свойств, поддерживаемых **HttpSource**, см. [HttpSource](#copy-activity-properties).

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with a copy activity",
    "activities":[  
      {
        "name": "HttpSourceToAzureBlob",
        "description": "Copy from an HTTP source to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "HttpSourceDataInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "HttpSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```

> [!NOTE]
> Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в статье [Сопоставление столбцов набора данных в фабрике данных Azure](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Производительность и настройка

Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действия копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
