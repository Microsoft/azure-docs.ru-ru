---
title: Перемещение данных с FTP-сервера с использованием фабрики данных Azure
description: Узнайте, как перемещать данные с FTP-сервера с использованием фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/02/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: a8a8ca44fbdb7610f85bc53c23d502d2efb01c8b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363956"
---
# <a name="move-data-from-an-ftp-server-by-using-azure-data-factory"></a>Перемещение данных с FTP-сервера с использованием фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-ftp-connector.md)
> * [Версия 2 (текущая)](../connector-ftp.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статье о [соединителе FTP в службе "Фабрика данных Azure" версии 2](../connector-ftp.md).

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure перемещать данные с FTP-сервера. Это продолжение статьи о [действиях перемещения данных](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью действия копирования.

Данные с FTP-сервера можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Сейчас фабрика данных поддерживает только перемещение данных с FTP-сервера в другие хранилища данных, но не наоборот. Она поддерживает локальные и облачные FTP-серверы.

> [!NOTE]
> Действие копирования не удаляет исходный файл после его успешного копирования в место назначения. Если необходимо удалить исходный файл после успешного копирования, создайте настраиваемое действие для удаления файла и используйте это действие в конвейере.

## <a name="enable-connectivity"></a>Включение соединения
При перемещении данных с **локального** FTP-сервера в облачное хранилище данных (например, хранилище BLOB-объектов Azure) установите и используйте шлюз управления данными. Шлюз управления данными — это агент клиента, установленный на локальном компьютере и позволяющий облачным службам подключаться к локальному ресурсу. Подробные сведения см. в разделе [Шлюз управления данными](data-factory-data-management-gateway.md). Пошаговые инструкции по настройке и использованию шлюза см. в разделе [Перемещение данных между локальными источниками и облаком](data-factory-move-data-between-onprem-and-cloud.md). Шлюз используется для подключения к FTP-серверу, даже если сервер выполняется на виртуальной машине Azure типа "инфраструктура как услуга" (IaaS).

Шлюз можно установить на тот же локальный компьютер или виртуальную машину Azure IaaS, что и FTP-сервер. Тем не менее рекомендуется установить шлюз на отдельный компьютер или виртуальную машину IaaS, чтобы избежать состязания за ресурсы, а также для повышения производительности. При установке шлюза на отдельный компьютер у компьютера должен быть доступ к FTP-серверу.

## <a name="get-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, который перемещает данные с исходного FTP-сервера, с помощью разных средств или интерфейсов API.

Проще всего создать конвейер с помощью **мастера копирования фабрики данных**. Краткое пошаговое руководство представлено в статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md).

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или интерфейса API для создания конвейера, который перемещает данные из исходного хранилища данных в приемник, необходимо выполнить указанные ниже действия.

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования.
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании средств или интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Пример определений JSON для сущностей фабрики данных, которые используются для копирования данных из хранилища данных FTP, доступен в разделе [Пример JSON. Копирование данных с FTP-сервера в большой двоичный объект Azure](#json-example-copy-data-from-ftp-server-to-azure-blob) этой статьи.

> [!NOTE]
> Подробные сведения о поддерживаемых форматах файлов и сжатия см. в разделе [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к FTP.

## <a name="linked-service-properties"></a>Свойства связанной службы
В приведенной ниже таблице описываются элементы JSON, которые относятся к связанной службе FTP.

| Свойство. | Описание | Обязательно | По умолчанию |
| --- | --- | --- | --- |
| тип |Задайте значение FtpServer. |Да |&nbsp; |
| host |Укажите имя или IP-адрес FTP-сервера. |Да |&nbsp; |
| authenticationType |Укажите тип проверки подлинности. |Да |Обычная, анонимная |
| username |Укажите пользователя, имеющего доступ к FTP-серверу. |нет |&nbsp; |
| password |Укажите пароль для пользователя (username). |Нет |&nbsp; |
| encryptedCredential |Укажите зашифрованные учетные данные для доступа к FTP-серверу. |Нет |&nbsp; |
| gatewayName |Укажите имя шлюза управления данными для подключения к локальному FTP-серверу. |Нет |&nbsp; |
| порт |Укажите порт, прослушиваемый FTP-сервером. |Нет |21 |
| enableSsl |Укажите, какой канал следует использовать (FTP через SSL или TLS). |Нет |true |
| enableServerCertificateValidation |Уточните, следует ли включать проверку сертификатов TLS и SSL на сервере при использовании FTP через канал SSL или TLS. |Нет |true |

>[!NOTE]
>Соединитель FTP поддерживает доступ к FTP-серверу без шифрования или явного шифрования SSL/TLS. оно не поддерживает неявное шифрование SSL/TLS.

### <a name="use-anonymous-authentication"></a>Использование анонимной проверки подлинности

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "authenticationType": "Anonymous",
              "host": "myftpserver.com"
        }
    }
}
```

### <a name="use-username-and-password-in-plain-text-for-basic-authentication"></a>Использование имени пользователя и пароля в виде обычного текста для обычной проверки подлинности

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
    "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```

### <a name="use-port-enablessl-enableservercertificatevalidation"></a>Использование свойств port, enableSsl, enableServerCertificateValidation

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

### <a name="use-encryptedcredential-for-authentication-and-gateway"></a>Использование свойства encryptedCredential для проверки подлинности и шлюза

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "mygateway"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, доступных для определения наборов данных, см. в разделе [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных.

Раздел **typeProperties** отличается для каждого типа набора данных. Он предоставляет сведения, которые относятся к типу набора данных. Раздел **typeProperties** набора данных типа **FileShare** содержит перечисленные ниже свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| folderPath |Подпуть к папке. Используйте escape-символ "\" для специальных символов в строке. Примеры приведены в разделе "Примеры определений связанной службы и набора данных".<br/><br/>Вы можете использовать это свойство вместе с параметром **partitionBy**, чтобы в путях к папкам учитывались дата и время начала и окончания среза. |Да |
| fileName |Укажите имя файла в свойстве **folderPath**, если требуется создать таблицу, ссылающуюся на конкретный файл в папке. Если этому свойству не присвоить значение, таблица будет указывать на все файлы в папке.<br/><br/>Если свойство **fileName** не указано для выходного набора данных, то имя созданного файла будет иметь следующий формат: <br/><br/>`Data.<Guid>.txt` (пример: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Нет |
| fileFilter |Укажите фильтр, который будет использоваться для выбора подмножества файлов в **FolderPath**, а не всех файлов.<br/><br/>Допустимые значения: `*` (несколько знаков) и `?` (один знак).<br/><br/>Пример 1: `"fileFilter": "*.log"`<br/>Пример 2: `"fileFilter": 2014-1-?.txt"`<br/><br/> Свойство **fileFilter** применяется ко входному набору данных FileShare. Распределенная файловая система Hadoop (HDFS) не поддерживает это свойство. |Нет |
| partitionedBy |Используется для того, чтобы указать динамические **путь к папке** и **имя файла** для временного ряда данных. Например, можно указать **путь к папке**, который будет параметризироваться для данных за каждый час. |Нет |
| format | Поддерживаются следующие типы формата: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Свойству **type** в разделе format необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах формат [текста](data-factory-supported-file-and-compression-formats.md#text-format), [Формат JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Формат Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Формат ORC](data-factory-supported-file-and-compression-formats.md#orc-format)и [Формат Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format) . <br><br> Если требуется скопировать файлы между файловыми хранилищами как есть (двоичное копирование), пропустите раздел формата в определениях входного и выходного наборов данных. |Нет |
| compression | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Optimal** и **Fastest**. Узнайте больше о [форматах файлов и сжатия данных в фабрике данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). |нет |
| useBinaryTransfer |Укажите, следует ли использовать режим передачи в двоичном формате. Значение true, если следует использовать двоичный формат (это значение по умолчанию), и false, если следует использовать ASCII. Это свойство можно использовать, только когда тип связанной службы — FtpServer. |Нет |

> [!NOTE]
> Свойства **fileName** и **fileFilter** нельзя использовать одновременно.

### <a name="use-the-partionedby-property"></a>Использование свойства partionedBy
Как сказано выше, для данных временных рядов путь к папке (**folderPath**) и имя файла (**fileName**) можно указывать динамически. Это делается с помощью свойства **partitionedBy**.

Сведения о наборах данных временных рядов, планировании и срезах см. в разделах [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md), [Планирование и исполнение с использованием фабрики данных](data-factory-scheduling-and-execution.md) и [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md).

#### <a name="sample-1"></a>Пример 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
В этом примере {Slice} заменяется значением SliceStart (системная переменная фабрики данных) в формате ГГГГММДДЧЧ. SliceStart указывает время начала среза. Путь к папке отличается для каждого среза. (Пример: wikidatagateway/wikisampledataout/2014100103 или wikidatagateway/wikisampledataout/2014100104.)

#### <a name="sample-2"></a>Пример 2

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
В этом примере год, месяц, день и время SliceStart извлекаются в отдельные переменные, используемые в свойствах **folderPath** и **fileName**.

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, доступных для определения действий, см. в разделе [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Свойства (такие как имя, описание, входные и выходные таблицы, политики и т. д.) доступны для всех типов действий.

Свойства, доступные в разделе **typeProperties** действия, с другой стороны, зависят от типа действия. Для действия копирования свойства типа различаются в зависимости от типов источников и приемников.

Если источник относится к типу **FileSystemSource**, в действии копирования в разделе **typeProperties** доступны указанные ниже свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| recursive |Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. |True, False (по умолчанию) |Нет |

## <a name="json-example-copy-data-from-ftp-server-to-azure-blob"></a>Пример JSON. Копирование данных с FTP-сервера в большой двоичный объект Azure
В этом примере показано, как скопировать данные с FTP-сервера в хранилище BLOB-объектов Azure. Однако данные можно напрямую копировать в любые приемники, указанные в статье [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Для этого применяется действие копирования в фабрике данных.

В следующих примерах приведены примеры определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)или [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md):

* Связанная служба типа [FtpServer](#linked-service-properties)
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Входной [набор данных](data-factory-create-datasets.md) типа [FileShare](#dataset-properties)
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [FileSystemSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

В этом примере данные с FTP-сервера каждый час копируются в большой двоичный объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

### <a name="ftp-linked-service"></a>Связанная служба FTP

В этом примере используется обычная проверка подлинности на основе имени пользователя и пароля, которые передаются в виде обычного текста. Кроме того, можно использовать любой из следующих способов:

* анонимная аутентификация;
* обычная аутентификация и шифрование учетных данных;
* FTP через SSL или TLS (FTPS).

Сведения о возможных типах проверки подлинности см. в разделе [Связанная служба FTP](#linked-service-properties).

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```
### <a name="azure-storage-linked-service"></a>Связанная служба хранения Azure

```JSON
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
### <a name="ftp-input-dataset"></a>Входной набор данных FTP

Этот набор данных ссылается на папку FTP `mysharedfolder` и файл `test.csv`. Конвейер копирует файл в место назначения.

Если параметру **External** присвоить **значение true** , служба фабрики данных будет изменяться, что набор данных является внешним по отношению к фабрике данных, и не будет создан действием в фабрике данных.

```JSON
{
  "name": "FTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "FTPLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv",
      "useBinaryTransfer": true
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

### <a name="azure-blob-output-dataset"></a>Выходной набор данных BLOB-объекта Azure

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используются год, месяц, день и час времени начала.

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/ftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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


### <a name="a-copy-activity-in-a-pipeline-with-file-system-source-and-blob-sink"></a>Действие копирования в конвейере с файловой системой в качестве источника и BLOB-объектом в качестве приемника

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **источника** установлено значение **FileSystemSource**, а для типа **приемника** — значение **BlobSink**.

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-08-24T18:00:00Z",
        "end": "2016-08-24T19:00:00Z"
    }
}
```
> [!NOTE]
> Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:

* Сведения о ключевых факторах, влияющих на производительность перемещения данных (действие копирования) в фабрике данных, и различных способах оптимизации этого процесса см. в [руководстве по настройке производительности действия копирования](data-factory-copy-activity-performance.md).

* Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
