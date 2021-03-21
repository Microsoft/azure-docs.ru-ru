---
title: Перемещение данных с SFTP Server с помощью фабрики данных Azure
description: Узнайте, как перемещать данные с локального или облачного SFTP-сервера с помощью фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/12/2018
robots: noindex
ms.openlocfilehash: b40e9dc83629362da899d2b5ff29ad42e21c4e32
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382656"
---
# <a name="move-data-from-an-sftp-server-using-azure-data-factory"></a>Перемещение данных с SFTP-сервера с использованием фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-sftp-connector.md)
> * [Версия 2 (текущая)](../connector-sftp.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе SFTP в службе "Фабрика данных Azure" версии 2](../connector-sftp.md).

В этой статье описано, как с помощью действия копирования в фабрике данных Azure переместить данные с локального или облачного SFTP-сервера в поддерживаемое хранилище-приемник данных. В этой статье в продолжение темы о [действиях перемещения данных](data-factory-data-movement-activities.md) приведены общие сведения о перемещении данных с помощью действия копирования, а также список поддерживаемых хранилищ данных, используемых в качестве источников и приемников.

Сейчас фабрика данных поддерживает только перемещение данных с SFTP-сервера в другие хранилища данных, но не наоборот. Она поддерживает локальные и облачные SFTP-серверы.

> [!NOTE]
> Действие копирования не удаляет исходный файл после его успешного копирования в место назначения. Если необходимо удалить исходный файл после успешного копирования, создайте настраиваемое действие для удаления файла и используйте это действие в конвейере.

## <a name="supported-scenarios-and-authentication-types"></a>Поддерживаемые сценарии и типы аутентификации
Этот соединитель SFTP можно использовать для копирования данных с **облачных или локальных SFTP-серверов**. Для подключения к серверу SFTP можно использовать **обычную** аутентификацию или аутентификацию **SshPublicKey**.

Для копирования данных с локального SFTP-сервера необходимо установить шлюз управления данными в локальной среде или на виртуальной машине Azure. Дополнительные сведения о шлюзе см. в статье [Шлюз управления данными](data-factory-data-management-gateway.md). Пошаговые инструкции по настройке и использованию шлюза см. в статье [moving data between on-premises locations and cloud](data-factory-move-data-between-onprem-and-cloud.md) (Перемещение данных между локальными и облачными ресурсами).

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, которое перемещает данные с SFTP-сервера с помощью разных инструментов и API-интерфейсов.

- Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

- Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) . Примеры JSON для копирования данных с SFTP-сервера в хранилище BLOB-объектов Azure см. в разделе [Пример JSON. Копирование данных с FTP-сервера в большой двоичный объект Azure](#json-example-copy-data-from-sftp-server-to-azure-blob) этой статьи.

## <a name="linked-service-properties"></a>Свойства связанной службы
В следующей таблице содержится описание элементов JSON, которые относятся к связанной службе FTP.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type | Задайте для свойства type значение `Sftp`. |Да |
| host | Имя или IP-адрес SFTP-сервера. |Да |
| порт |Порт, прослушиваемый SFTP-сервером. По умолчанию используется значение 21. |нет |
| authenticationType |Укажите тип аутентификации. Допустимые значения: **Basic**, **SshPublicKey**. <br><br> Описание свойств и примеры JSON для каждого типа см. ниже в разделах [использование обычной аутентификации](#using-basic-authentication) и [использование аутентификации с открытым ключом SSH](#using-ssh-public-key-authentication) соответственно. |Да |
| skipHostKeyValidation | Указывает, нужно ли пропустить проверку ключа узла. | Нет. По умолчанию имеет значение False. |
| hostKeyFingerprint | Содержит отпечаток ключа узла. | Да, если для `skipHostKeyValidation` указано значение False.  |
| gatewayName |Имя шлюза управления данными для подключения к локальному SFTP-серверу. | Да, если копирование выполняется с локального SFTP-сервера. |
| encryptedCredential | Зашифрованные учетные данные для доступа к SFTP-серверу. Генерируются автоматически, когда вы выбираете обычную аутентификацию (имя пользователя и пароль) или аутентификацию SshPublicKey (имя пользователя и путь к закрытому ключу или содержимое ключа) в мастере копирования или во всплывающем диалоговом окне ClickOnce. | Нет. Применимо, только если копирование выполняется с локального SFTP-сервера. |

### <a name="using-basic-authentication"></a>Использование обычной аутентификации

Чтобы использовать обычную аутентификацию, задайте для `authenticationType` значение `Basic`, а также все следующие свойства в дополнение к универсальным свойствам соединителя SFTP, которые описаны в последнем разделе.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| username | Пользователь, имеющий доступ к SFTP-серверу. |Да |
| password | Пароль пользователя, указанного в свойстве имя пользователя. | Да |

#### <a name="example-basic-authentication"></a>Пример обычной аутентификации
```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>Пример обычной аутентификации с шифрованием учетных данных

```JSON
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
      }
}
```

### <a name="using-ssh-public-key-authentication"></a>Использование аутентификации с открытым ключом SSH

Чтобы использовать аутентификацию с открытым ключом SSH, задайте для `authenticationType` значение `SshPublicKey`, а также все следующие свойства в дополнение к универсальным свойствам соединителя SFTP, которые описаны в последнем разделе.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| username |Пользователь, имеющий доступ к SFTP-серверу. |Да |
| privateKeyPath | Укажите доступный для шлюза абсолютный путь к файлу закрытого ключа. | Должно быть указано одно из свойств: `privateKeyPath` или `privateKeyContent`. <br><br> Применимо, только если копирование выполняется с локального SFTP-сервера. |
| privateKeyContent | Сериализованная строка с содержимым закрытого ключа. Мастер копирования может автоматически считать файл закрытого ключа и извлечь его содержимое. Если вы используете другие средства и (или) пакет SDK, лучше использовать свойство privateKeyPath. | Должно быть указано одно из свойств: `privateKeyPath` или `privateKeyContent`. |
| passPhrase | Укажите пароль или парольную фразу для расшифровки закрытого ключа, если они используются для защиты файлы ключа. | Да, если файл закрытого ключа защищен парольной фразой. |

> [!NOTE]
> Соединитель SFTP поддерживает только ключ OpenSSH RSA/DSA. Содержимое файла ключа должно начинаться с текста "-----BEGIN [RSA/DSA] PRIVATE KEY-----". Если файл закрытого ключа имеет формат PPK, используйте средство Putty для преобразования из формата PPK в OpenSSH.

#### <a name="example-sshpublickey-authentication-using-private-key-filepath"></a>Пример аутентификации с закрытым ключом SSH, для которого указан путь к файлу

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>Пример аутентификации с закрытым ключом SSH, для которого указано содержимое

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных.

Раздел **typeProperties** отличается для каждого типа набора данных. Он предоставляет сведения, которые относятся к типу набора данных. Раздел typeProperties набора данных с типом **FileShare** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| folderPath |Подпуть к папке. Используйте escape-символ "\" для специальных символов в строке. Примеры приведены в разделе "Примеры определений связанной службы и набора данных".<br/><br/>Вы можете использовать это свойство вместе с параметром **partitionBy**, чтобы в путях к папкам учитывались дата и время начала и окончания среза. |Да |
| fileName |Укажите имя файла в свойстве **folderPath**, если требуется создать таблицу, ссылающуюся на конкретный файл в папке. Если этому свойству не присвоить значение, таблица будет указывать на все файлы в папке.<br/><br/>Если свойство fileName не указано для выходного набора данных, то имя созданного файла будет иметь следующий формат: <br/><br/>`Data.<Guid>.txt` (Пример: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Нет |
| fileFilter |Укажите фильтр для выбора подмножества файлов из folderPath. Фильтр дает возможность выбирать только некоторые файлы, а не все.<br/><br/>Допустимые значения: `*` (несколько знаков) и `?` (один знак).<br/><br/>Пример 1: `"fileFilter": "*.log"`<br/>Пример 2: `"fileFilter": 2014-1-?.txt"`<br/><br/> Свойство fileFilter применяется к входному набору данных FileShare. HDFS не поддерживает это свойство. |Нет |
| partitionedBy |Чтобы указать для временного ряда данных динамические путь к папке и имя файла, используйте свойство partitionedBy. Например, путь к папке folderPath каждый час будет другим. |Нет |
| format | Поддерживаются следующие типы формата: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Свойству **type** в разделе format необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах о [текстовом формате](data-factory-supported-file-and-compression-formats.md#text-format), [формате Json](data-factory-supported-file-and-compression-formats.md#json-format), [формате Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [формате Orc](data-factory-supported-file-and-compression-formats.md#orc-format) и [ формате Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Если требуется скопировать файлы между файловыми хранилищами **как есть** (двоичное копирование), можно пропустить раздел форматирования в определениях входного и выходного наборов данных. |Нет |
| compression | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Оптимальный** и **Самый быстрый**. Узнайте больше о [форматах файлов и сжатия данных в фабрике данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). |нет |
| useBinaryTransfer |Укажите, использовать ли режим передачи в двоичном формате. Значение true, если использовать двоичный режим, и false, если ASCII. Значение по умолчанию: True. Это свойство можно использовать, только когда тип связанной службы является FTP-сервер. |Нет |

> [!NOTE]
> Свойства filename и fileFilter нельзя использовать одновременно.

### <a name="using-partionedby-property"></a>Использование свойства partionedBy
Как сказано выше, для временных рядов данных путь к папке и имя файла можно указывать динамически. Это делается с помощью свойства partitionedBy. Вы можете это сделать при помощи макроса фабрики данных и системных переменных SliceStart и SliceEnd, которые определяют логический период имеющегося среза данных.

Чтобы узнать о наборах данных временных рядов, планировании и срезах, ознакомьтесь со статьями [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md), [Планирование и исполнение с использованием фабрики данных](data-factory-scheduling-and-execution.md) и [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md).

#### <a name="sample-1"></a>Пример 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
В этом примере {Slice} заменяется значением SliceStart (системная переменная фабрики данных) в формате ГГГГММДДЧЧ. SliceStart указывает время начала среза. Значение folderPath отличается для каждого среза. Например, wikidatagateway/wikisampledataout/2014100103 или wikidatagateway/wikisampledataout/2014100104.

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
В этом примере год, месяц, день и время SliceStart извлекаются в отдельные переменные, используемые в свойствах folderPath и fileName.

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (такие как имя, описание, входные и выходные таблицы, политики и т. д.) доступны для всех типов действий.

В свою очередь, доступные в разделе typeProperties свойства зависят от определенного типа действия. Для действия копирования свойства типа различаются в зависимости от типов источников и приемников.

[!INCLUDE [data-factory-file-system-source](../../../includes/data-factory-file-system-source.md)]

## <a name="supported-file-and-compression-formats"></a>Поддерживаемые форматы файлов и сжатия
Дополнительные сведения см. в статье [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

## <a name="json-example-copy-data-from-sftp-server-to-azure-blob"></a>Пример JSON. Копирование данных с FTP-сервера в большой двоичный объект Azure
В следующем примере приводятся образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Вы узнаете, как копировать данные из источника SFTP в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать **непосредственно** из любых источников в любой указанный [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемник. Это делается с помощью действия копирования в фабрике данных Azure.

> [!IMPORTANT]
> Этот пример содержит фрагменты кода JSON. Он не включает в себя пошаговые инструкции по созданию фабрики данных. Эти инструкции приведены в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md) .

Образец состоит из следующих сущностей фабрики данных.

* Связанная служба типа [sftp](#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа общая [Папка.](#dataset-properties)
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [FileSystemSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные с SFTP-сервера каждый час копируются в большой двоичный объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба SFTP**

Этот пример кода использует обычную аутентификацию на основе имени пользователя и пароля, которые передаются в виде обычного текста. Кроме того, можно использовать любой из следующих способов:

* обычная аутентификация и шифрование учетных данных;
* Аутентификация с открытым ключом SSH

Сведения о возможных типах аутентификации см. в разделе [Связанная служба FTP](#linked-service-properties).

```JSON

{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "myuser",
            "password": "mypassword",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```
**Связанная служба хранения Azure**

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
**Входной набор данных SFTP**

Этот набор данных ссылается на папку SFTP `mysharedfolder` и файл `test.csv`. Конвейер копирует файл в место назначения.

Если параметру external присвоить значение true, то фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой фабрике данных.

```JSON
{
  "name": "SFTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "SftpLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Выходной набор данных BLOB-объекта Azure**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/sftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **FileSystemSource**, а для типа **sink** — значение **BlobSink**.

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
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
        "start": "2017-02-20T18:00:00Z",
        "end": "2017-02-20T19:00:00Z"
    }
}
```

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:

* [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .
