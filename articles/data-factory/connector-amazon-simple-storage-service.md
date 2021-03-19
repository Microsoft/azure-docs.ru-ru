---
title: Копирование данных из Amazon Simple Storage Service (S3)
description: Узнайте, как копировать данные из Amazon Simple Storage Service (S3) в поддерживаемые хранилища данных приемника с помощью фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/17/2021
ms.openlocfilehash: 03b0cd852f34e115cc5bbc60448e45fcbb680474
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104601233"
---
# <a name="copy-data-from-amazon-simple-storage-service-by-using-azure-data-factory"></a>Копирование данных из Amazon Simple Storage Service с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
>
> * [Версия 1](v1/data-factory-amazon-simple-storage-service-connector.md)
> * [Текущая версия](connector-amazon-simple-storage-service.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Эта статья рассказывает, как копировать данные из Amazon Simple Storage Service (Amazon S3). Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

>[!TIP]
>Дополнительные сведения о сценарии переноса данных из Amazon S3 в службу хранилища Azure см. в статье [Использование фабрики данных Azure для переноса данных из Amazon S3 в службу хранилища Azure](data-migration-guidance-s3-azure-storage.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Соединитель Amazon S3 предназначен для поддержки следующих действий.

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)
- [Действие удаления](delete-activity.md)

В частности, этот соединитель Amazon S3 поддерживает копирование файлов в виде или анализ файлов с помощью [поддерживаемых форматов файлов и кодеков сжатия](supported-file-formats-and-compression-codecs.md). Можно также [сохранить метаданные файла во время копирования](#preserve-metadata-during-copy). Для проверки подлинности запросов в S3 соединитель использует [подпись AWS версии 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html).

>[!TIP]
>Этот соединитель Amazon S3 можно использовать для копирования данных из *любого поставщика хранилища, совместимого с S3*, такого как [Google Cloud Storage](connector-google-cloud-storage.md). Укажите соответствующий URL-адрес службы в конфигурации связанной службы.

## <a name="required-permissions"></a>Необходимые разрешения

Чтобы скопировать данные из Amazon S3, убедитесь, что у вас были предоставлены следующие разрешения для операций с объектами Amazon S3: `s3:GetObject` и `s3:GetObjectVersion` .

Если вы используете пользовательский интерфейс фабрики данных для создания `s3:ListAllMyBuckets` , `s3:ListBucket` / `s3:GetBucketLocation` для таких операций, как проверка подключения к связанной службе и просмотр из корня, требуются дополнительные разрешения. Если вы не хотите предоставлять эти разрешения, можно выбрать параметры "проверить подключение к пути к файлу" или "Обзор по указанному пути" в пользовательском интерфейсе.

Полный список разрешений Amazon S3 см. в разделе [Указание разрешений в политике](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html) на сайте AWS.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)] 

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, характерных для Amazon S3.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Amazon S3 поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства **Type** необходимо задать значение **AmazonS3**. | Да |
| authenticationType | Укажите тип проверки подлинности, используемый для подключения к Amazon S3. Вы можете использовать ключи доступа для учетной записи AWS Identity and Access Management (IAM) или [временных учетных данных безопасности](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html).<br>Допустимые значения: `AccessKey` (по умолчанию) и `TemporarySecurityCredentials` . |Нет |
| accessKeyId | Идентификатор секретного ключа доступа. |Да |
| secretAccessKey | Сам секретный ключ доступа. Пометьте это поле как **SecureString**, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| сессионтокен | Применяется при использовании проверки подлинности с [временными учетными данными безопасности](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) . Узнайте, как [запросить временные учетные данные безопасности](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html#api_getsessiontoken) из AWS.<br>Примечание. AWS временные учетные данные истекает от 15 минут до 36 часов в зависимости от параметров. Убедитесь, что учетные данные действительны при выполнении действия, особенно для рабочей нагрузки. Например, вы можете периодически обновлять его и хранить в Azure Key Vault.<br>Пометьте это поле как **SecureString**, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Нет |
| serviceUrl | Укажите настраиваемую конечную точку S3, если вы копируете данные из совместимого с S3 поставщика хранилища, отличного от официальной службы Amazon S3. Например, для копирования данных из Google Cloud Storage укажите `https://storage.googleapis.com`. | нет |
| форцепасстиле | Указывает, следует ли использовать [уровень доступа](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html#path-style-access) S3, а не виртуальный размещенный в стиле доступ. Допустимые значения: **false** (по умолчанию), **true**.<br>Если вы подключаетесь к поставщику хранилища, совместимому с S3, отличному от официальной службы Amazon S3, и для этого хранилища данных требуется доступ в стиле Path (например, в [облачном хранилище Oracle](https://docs.oracle.com/iaas/Content/Object/Tasks/s3compatibleapi.htm)), задайте для этого свойства значение true. Ознакомьтесь с документацией по каждому хранилищу данных в случае, если необходим доступ в стиле Path. |Нет |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду выполнения интеграции (если хранилище данных находится в частной сети). Если это свойство не указано, служба использует среду выполнения интеграции Azure по умолчанию. |Нет |

>[!TIP]
>Укажите URL-адрес настраиваемой службы S3, если вы копируете данные из хранилища, совместимого с S3, отличного от официальной службы Amazon S3.

**Пример. Использование проверки подлинности с помощью ключа доступа**

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AmazonS3",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                "type": "SecureString",
                "value": "<secret access key>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример. Использование временной проверки подлинности с учетными данными безопасности**

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AmazonS3",
        "typeProperties": {
            "authenticationType": "TemporarySecurityCredentials",
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                "type": "SecureString",
                "value": "<secret access key>"
            },
            "sessionToken": {
                "type": "SecureString",
                "value": "<session token>"
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для Amazon S3 в `location` параметрах набора данных на основе формата:

| Свойство.   | Описание                                                  | Обязательно |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | Свойство **Type** в `location` наборе данных должно иметь значение **AmazonS3Location**. | Да      |
| bucketName | Имя контейнера S3.                                          | Да      |
| folderPath | Путь к папке в заданном контейнере. Если вы хотите использовать подстановочный знак для фильтрации папки, пропустите этот параметр и укажите его в параметрах источника действия. | Нет       |
| fileName   | Имя файла в заданном контейнере и пути к папке. Если вы хотите использовать подстановочный знак для фильтрации файлов, пропустите этот параметр и укажите значение в параметрах источника действия. | Нет       |
| version | Версия объекта S3, если включено управление версиями S3. Если он не указан, будет выбрана последняя версия. |Нет |

**Пример**.

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AmazonS3Location",
                "bucketName": "bucketname",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Amazon S3.

### <a name="amazon-s3-as-a-source-type"></a>Amazon S3 в качестве типа источника

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для Amazon S3 в `storeSettings` параметрах источника копирования на основе формата:

| Свойство.                 | Описание                                                  | Обязательно                                                    |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| type                     | Свойство **Type** в разделе `storeSettings` должно иметь значение **AmazonS3ReadSettings**. | Да                                                         |
| ***Поиск файлов для копирования*** |  |  |
| ВАРИАНТ 1. Статический путь<br> | Копирование из заданного контейнера или папки/пути к файлу, которые указаны в наборе данных. Если вы хотите скопировать все файлы из контейнера или папки, дополнительно укажите `wildcardFileName` в качестве `*` . |  |
| Вариант 2. Префикс S3<br>- prefix | Префикс для имени ключа S3 в заданном контейнере, настроенном в наборе данных для фильтрации исходных файлов S3. Будут выбраны ключи S3, имена которых начинаются с `bucket_in_dataset/this_prefix` . Он использует фильтр на стороне службы S3's, который обеспечивает лучшую производительность, чем фильтр с подстановочными знаками.<br/><br/>При использовании префикса и выборе копирования в файловый приемник с сохранением иерархии Обратите внимание на вложенный путь после последнего "/" в префиксе. Например, у вас есть исходный код  `bucket/folder/subfolder/file.txt` и настраивается префикс в виде `folder/sub` , а затем сохраненный путь к файлу — `subfolder/file.txt` . | нет |
| ВАРИАНТ 3. Подстановочный знак<br>- wildcardFolderPath | Путь к папке с подстановочными знаками в заданном контейнере, настроенном в наборе данных для фильтрации исходных папок. <br>Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу). Используйте `^` для экранирования, если имя папки содержит подстановочный знак или escape-символ внутри. <br>Дополнительные примеры приведены в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). | нет                                            |
| ВАРИАНТ 3. Подстановочный знак<br>- wildcardFileName | Имя файла с подстановочными знаками в заданном контейнере и пути к папке (или пути к папке с подстановочными знаками) для фильтрации исходных файлов. <br>Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу). Используйте `^` для экранирования, если имя файла содержит подстановочный знак или escape-символ внутри.  Дополнительные примеры приведены в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). | Да |
| ВАРИАНТ 4. Список файлов<br>- fileListPath | Указывает, что нужно скопировать заданный набор файлов. Укажите текстовый файл, содержащий список файлов, которые необходимо скопировать, по одному файлу в строке, который является относительным путем к пути, настроенному в наборе данных.<br/>Если вы используете этот параметр, не указывайте имя файла в наборе данных. Ознакомьтесь с дополнительными примерами в разделе [Примеры списков файлов](#file-list-examples). |Нет |
| ***Дополнительные параметры*** |  | |
| recursive | Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. Обратите внимание, что если параметру **recursive** присвоено значение **true** и приемник является хранилищем на основе файлов, пустая папка или подпапка не копируется и не создается в приемнике. <br>Допустимые значения: **true** (по умолчанию) и **false**.<br>Это свойство не применяется при настройке `fileListPath`. |Нет |
| делетефилесафтеркомплетион | Указывает, будут ли удалены двоичные файлы из исходного хранилища после успешного перемещения в целевое хранилище. Файл удаляется для каждого файла, поэтому при сбое действия копирования вы увидите, что некоторые файлы уже скопированы в место назначения и удалены из источника, а остальные остаются в исходном хранилище. <br/>Это свойство допустимо только в сценарии копирования двоичных файлов. Значение по умолчанию: false. |Нет |
| modifiedDatetimeStart    | Файлы фильтруются с учетом атрибута: Последнее изменение. <br>Файлы будут выбраны, если время их последнего изменения находится в диапазоне времени `modifiedDatetimeStart` и `modifiedDatetimeEnd`. Время применяется к часовому поясу UTC в формате "2018-12-01T05:00:00Z". <br> Свойства могут иметь **значение NULL**, что означает, что фильтр атрибутов файла не будет применяться к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.<br/>Это свойство не применяется при настройке `fileListPath`. | Нет                                            |
| modifiedDatetimeEnd      | То же, что и выше.                                               | Нет                                                          |
| енаблепартитиондисковери | Для секционированных файлов укажите, следует ли анализировать секции из пути к файлу и добавить их в качестве дополнительных исходных столбцов.<br/>Допустимые значения: **false** (по умолчанию) и **true**. | Нет                                            |
| партитионрутпас | Если обнаружение секций включено, укажите абсолютный корневой путь для чтения секционированных папок в виде столбцов данных.<br/><br/>Если значение не указано, по умолчанию<br/>— При использовании пути к файлу в наборе данных или списке файлов в источнике корневой путь к разделу — это путь, настроенный в наборе данных.<br/>— При использовании фильтра папки с подстановочными знаками корневой путь к разделу является вложенным путем перед первым подстановочным знаком.<br/>— При использовании префикса корневой путь к разделу является вложенным путем перед последним "/". <br/><br/>Например, предположим, что вы настроили путь в наборе данных как «root/папка/year = 2020/месяц = 08/Day = 27»:<br/>— Если указать корневой путь к разделу "root/папка/year = 2020", действие копирования создаст еще два столбца `month` и `day` со значением "08" и "27" соответственно в дополнение к столбцам внутри файлов.<br/>— Если корневой путь секции не указан, лишний столбец не будет создан. | Нет                                            |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет                                                          |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromAmazonS3",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AmazonS3ReadSettings",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Примеры фильтров папок и файлов

В этом разделе описываются результаты применения фильтров с подстановочными знаками к пути папки и имени файла.

| bucket | ключ | recursive | Структура исходной папки и результат фильтрации (извлекаются файлы, выделенные полужирным шрифтом)|
|:--- |:--- |:--- |:--- |
| bucket | `Folder*/*` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| bucket | `Folder*/*` | Да | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| bucket | `Folder*/*.csv` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| bucket | `Folder*/*.csv` | Да | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |

### <a name="file-list-examples"></a>Примеры списков файлов

В этом разделе описывается результирующее поведение использования пути к списку файлов в источнике действия копирования.

Предположим, что у вас есть следующая исходная структура папок и вы хотите скопировать файлы полужирным шрифтом:

| Пример исходной структуры                                      | Содержимое файла FileListToCopy.txt                             | Конфигурация фабрики данных                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Метаданные<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | Файл1.csv<br>Вложенная_папка1/Файл3.csv<br>Вложенная_папка1/Файл5.csv | **В наборе данных:**<br>- Контейнер: `bucket`<br>- Путь к папке: `FolderA`<br><br>**В источнике действия копирования:**<br>— Путь к списку файлов: `bucket/Metadata/FileListToCopy.txt` <br><br>Путь к списку файлов указывает на текстовый файл в том же хранилище данных, который содержит список файлов, которые необходимо скопировать, по одному файлу в строке с относительным путем к пути, заданному в наборе данных. |

## <a name="preserve-metadata-during-copy"></a>Сохранение метаданных файла во время копирования

При копировании файлов из Amazon S3 в Azure Data Lake Storage 2-го поколения или хранилище BLOB-объектов Azure можно сохранить метаданные файла вместе с данными. Подробнее см. в разделе [Сохранение метаданных](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Свойства действия GetMetadata

Чтобы получить сведения о свойствах, проверьте [действие with Metadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Свойства действия удаления

Чтобы получить сведения о свойствах, установите флажок [действие удаления](delete-activity.md).

## <a name="legacy-models"></a>Устаревшие модели

>[!NOTE]
>Следующие модели по-прежнему поддерживаются в соответствии с обратной совместимостью. Мы рекомендуем использовать новую модель, упомянутую ранее. Пользовательский интерфейс создания фабрики данных переключился на создание новой модели.

### <a name="legacy-dataset-model"></a>Устаревшая модель набора данных

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство **Type** набора данных должно иметь значение **AmazonS3Object**. |Да |
| bucketName | Имя контейнера S3. Фильтр с подстановочными знаками не поддерживается. |Да для действия "Копировать" или "Поиск", "нет" для действия "неметаданные" |
| ключ | Имя или фильтр подстановочного знака для ключа объекта S3 в указанном контейнере. Применяется только в том случае, если свойство **prefix** не задано. <br/><br/>Фильтр с подстановочными знаками поддерживается как для части папки, так и для части имени файла. Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу).<br/>Пример 1. `"key": "rootfolder/subfolder/*.csv"`<br/>Пример 2. `"key": "rootfolder/subfolder/???20180427.txt"`<br/>Дополнительные примеры приведены в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). Используйте `^` для экранирования, если фактическая папка или имя файла содержит подстановочный знак или escape-символ внутри. |нет |
| prefix | Префикс для ключа объекта S3. Выбираются объекты, ключи которых начинаются с этого префикса. Применяется только в том случае, если свойство **ключа** не задано. |Нет |
| version | Версия объекта S3, если включено управление версиями S3. Если версия не указана, будет получена последняя версия. |Нет |
| modifiedDatetimeStart | Файлы фильтруются с учетом атрибута: Последнее изменение. Файлы будут выбраны, если время их последнего изменения находится в диапазоне времени `modifiedDatetimeStart` и `modifiedDatetimeEnd`. Время представлено часовым поясом UTC в формате "2018-12-01T05:00:00Z". <br/><br/> Имейте в виду, что включение этого параметра повлияет на общую производительность перемещения данных при необходимости фильтрации огромных объемов файлов. <br/><br/> Свойства могут иметь **значение NULL**, что означает, что фильтр атрибутов файла не будет применяться к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно null, то будут выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.| Нет |
| modifiedDatetimeEnd | Файлы фильтруются с учетом атрибута: Последнее изменение. Файлы будут выбраны, если время их последнего изменения находится в диапазоне времени `modifiedDatetimeStart` и `modifiedDatetimeEnd`. Время представлено часовым поясом UTC в формате "2018-12-01T05:00:00Z". <br/><br/> Имейте в виду, что включение этого параметра повлияет на общую производительность перемещения данных при необходимости фильтрации огромных объемов файлов. <br/><br/> Свойства могут иметь **значение NULL**, что означает, что фильтр атрибутов файла не будет применяться к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.| Нет |
| format | Если требуется скопировать файлы между файловыми хранилищами "как есть" (двоичное копирование), можно пропустить раздел форматирования в определениях входного и выходного наборов данных.<br/><br/>Если необходимо проанализировать или создать файлы определенного формата, поддерживаются следующие форматы файлов: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Свойству **type** в разделе **format** необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах о [текстовом формате](supported-file-formats-and-compression-codecs-legacy.md#text-format), [формате JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [формате Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [формате Orc](supported-file-formats-and-compression-codecs-legacy.md#orc-format) и [формате Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Нет (только для сценария двоичного копирования) |
| compression | Укажите тип и уровень сжатия данных. Дополнительные сведения см. в разделе [Поддержка сжатия](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**.<br/>Поддерживаемые уровни: **Optimal** и **Fastest**. |Нет |

>[!TIP]
>Чтобы скопировать все файлы в папке, укажите **bucketName** для контейнера и **префикса** для части папки.
>
>Чтобы скопировать один файл с заданным именем, укажите **bucketName** для контейнера и **ключ** для части папки и имя файла.
>
>Чтобы скопировать подмножество файлов в папке, укажите **bucketName** для контейнера и **ключ** для части папки, а также примените фильтр с подстановочными знаками.

**Пример. Использование префикса**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3Object",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "prefix": "testFolder/test",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

**Пример. Использование ключа и версии (необязательно)**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "key": "testFolder/testfile.csv.gz",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-source-model-for-the-copy-activity"></a>Устаревшая исходная модель для действия копирования

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство **Type** источника действия копирования должно иметь значение **FileSystemSource**. |Да |
| recursive | Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. Обратите внимание, что если параметру **recursive** присвоено значение **true** и приемник является хранилищем на основе файлов, пустая папка или подпапка не будет скопирована или создана в приемнике.<br/>Допустимые значения: **true** (по умолчанию) и **false**. | Нет |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromAmazonS3",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Amazon S3 input dataset name>",
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
                "type": "FileSystemSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, которые действие копирования в фабрике данных Azure поддерживает в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats).
