---
title: Копирование данных из Google Cloud Storage с помощью фабрики данных Azure
description: Узнайте, как копировать данные из Google Cloud Storage в поддерживаемые хранилища данных, выступающие в роли приемников, с помощью Фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: jingwang
ms.openlocfilehash: b8e518ba23c877ee80197ad94a6bc01b23b044a3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588999"
---
# <a name="copy-data-from-google-cloud-storage-by-using-azure-data-factory"></a>Копирование данных из Google Cloud Storage с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано копирование данных из облачного хранилища Google (GCS). Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Соединитель облачного хранилища Google поддерживается для выполнения следующих действий.

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)
- [Действие удаления](delete-activity.md)

В частности, этот соединитель облачного хранилища Google поддерживает копирование файлов в виде или анализ файлов с помощью [поддерживаемых форматов файлов и кодеков сжатия](supported-file-formats-and-compression-codecs.md). Она использует преимущества совместимости с S3, совместимой с режимом "за пределом".

## <a name="prerequisites"></a>Предварительные требования

Для учетной записи хранения Google Cloud требуется следующая настройка:

1. Включите возможность взаимодействия для своей учетной записи облачного хранилища Google.
2. Задайте проект по умолчанию, содержащий данные, которые необходимо скопировать из целевого сегмента GC.
3. Создайте учетную запись службы и определите правильные уровни разрешений с помощью Cloud IAM на обеспечить. 
4. Создание ключей доступа для этой учетной записи службы.

![Извлеките ключ доступа для облачного хранилища Google.](media/connector-google-cloud-storage/google-storage-cloud-settings.png)

## <a name="required-permissions"></a>Необходимые разрешения

Чтобы скопировать данные из Google Cloud Storage, убедитесь, что у вас были предоставлены следующие разрешения для операций с объектами: ` storage.objects.get` и ` storage.objects.list` .

Если вы используете пользовательский интерфейс фабрики данных для создания, ` storage.buckets.list` то для таких операций, как проверка подключения к связанной службе и просмотр из корня, требуются дополнительные разрешения. Если вы не хотите предоставлять это разрешение, можно выбрать параметры "проверить подключение к пути к файлу" или "Обзор по указанному пути" в пользовательском интерфейсе.

Полный список ролей хранилища Google Cloud и соответствующих разрешений см. в статье [роли IAM для облачного хранилища](https://cloud.google.com/storage/docs/access-control/iam-roles) на сайте Google Cloud.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)] 

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей Фабрики данных, относящихся к Google Cloud Storage.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанных служб Google Cloud Storage поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства **Type** необходимо задать значение **гуглеклаудстораже**. | Да |
| accessKeyId | Идентификатор секретного ключа доступа. Сведения о том, как найти ключ доступа и секрет, приводятся в разделе [Предварительные требования](#prerequisites). |Да |
| secretAccessKey | Сам секретный ключ доступа. Пометьте это поле как **SecureString** , чтобы безопасно хранить его в фабрике данных, или [сослаться на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| serviceUrl | Укажите пользовательскую конечную точку GCS в качестве `https://storage.googleapis.com` . | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду выполнения интеграции (если хранилище данных находится в частной сети). Если это свойство не указано, служба использует среду выполнения интеграции Azure по умолчанию. |Нет |

Приведем пример:

```json
{
    "name": "GoogleCloudStorageLinkedService",
    "properties": {
        "type": "GoogleCloudStorage",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                "type": "SecureString",
                "value": "<secret access key>"
            },
            "serviceUrl": "https://storage.googleapis.com"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для облачного хранилища Google в `location` параметрах набора данных на основе формата:

| Свойство.   | Описание                                                  | Обязательно |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | Свойство **Type** в `location` наборе данных должно иметь значение **гуглеклаудсторажелокатион**. | Да      |
| bucketName | Имя контейнера GCS.                                          | Да      |
| folderPath | Путь к папке в заданном контейнере. Если вы хотите использовать подстановочный знак для фильтрации папки, пропустите этот параметр и укажите его в параметрах источника действия. | Нет       |
| fileName   | Имя файла в заданном контейнере и пути к папке. Если вы хотите использовать подстановочный знак для фильтрации файлов, пропустите этот параметр и укажите значение в параметрах источника действия. | Нет       |

**Пример**.

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Google Cloud Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "GoogleCloudStorageLocation",
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

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Google Cloud Storage.

### <a name="google-cloud-storage-as-a-source-type"></a>Хранилище Google Cloud в качестве типа источника

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для облачного хранилища Google в `storeSettings` параметрах в источнике копирования на основе формата:

| Свойство.                 | Описание                                                  | Обязательно                                                    |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| type                     | Свойство **Type** в разделе `storeSettings` должно иметь значение **гуглеклаудсторажереадсеттингс**. | Да                                                         |
| ***Поиск файлов для копирования*** |  |  |
| ВАРИАНТ 1. Статический путь<br> | Копирование из заданного контейнера или папки/пути к файлу, которые указаны в наборе данных. Если вы хотите скопировать все файлы из контейнера или папки, дополнительно укажите `wildcardFileName` в качестве `*` . |  |
| Вариант 2. Префикс GCS<br>- prefix | Префикс для имени ключа GC в заданном контейнере, настроенном в наборе данных для фильтрации исходных файлов GC. Будут выбраны ключи GC, имена которых начинаются с `bucket_in_dataset/this_prefix` . Он использует фильтр на стороне службы GC, обеспечивающий лучшую производительность по сравнению с фильтром с подстановочными знаками. | нет |
| ВАРИАНТ 3. Подстановочный знак<br>- wildcardFolderPath | Путь к папке с подстановочными знаками в заданном контейнере, настроенном в наборе данных для фильтрации исходных папок. <br>Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу). Используйте `^` для экранирования, если имя папки содержит подстановочный знак или escape-символ внутри. <br>Дополнительные примеры приведены в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). | нет                                            |
| ВАРИАНТ 3. Подстановочный знак<br>- wildcardFileName | Имя файла с подстановочными знаками в заданном контейнере и пути к папке (или пути к папке с подстановочными знаками) для фильтрации исходных файлов. <br>Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу). Используйте `^` для экранирования, если имя файла содержит подстановочный знак или escape-символ внутри.  Дополнительные примеры приведены в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). | Да |
| ВАРИАНТ 3. Список файлов<br>- fileListPath | Указывает, что нужно скопировать заданный набор файлов. Укажите текстовый файл, содержащий список файлов, которые необходимо скопировать, по одному файлу в строке, который является относительным путем к пути, настроенному в наборе данных.<br/>Если вы используете этот параметр, не указывайте имя файла в наборе данных. Ознакомьтесь с дополнительными примерами в разделе [Примеры списков файлов](#file-list-examples). |Нет |
| ***Дополнительные параметры*** |  | |
| recursive | Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. Обратите внимание, что если параметру **recursive** присвоено значение **true** и приемник является хранилищем на основе файлов, пустая папка или подпапка не копируется и не создается в приемнике. <br>Допустимые значения: **true** (по умолчанию) и **false**.<br>Это свойство не применяется при настройке `fileListPath`. |Нет |
| делетефилесафтеркомплетион | Указывает, будут ли удалены двоичные файлы из исходного хранилища после успешного перемещения в целевое хранилище. Файл удаляется для каждого файла, поэтому при сбое действия копирования вы увидите, что некоторые файлы уже скопированы в место назначения и удалены из источника, а остальные остаются в исходном хранилище. <br/>Это свойство допустимо только в сценарии копирования двоичных файлов. Значение по умолчанию: false. |Нет |
| modifiedDatetimeStart    | Файлы фильтруются с учетом атрибута: Последнее изменение. <br>Файлы будут выбраны, если время их последнего изменения находится в диапазоне времени `modifiedDatetimeStart` и `modifiedDatetimeEnd`. Время представлено часовым поясом UTC в формате "2018-12-01T05:00:00Z". <br> Свойства могут иметь **значение NULL**, что означает, что фильтр атрибутов файла не будет применяться к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно **null**, то будут выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.<br/>Это свойство не применяется при настройке `fileListPath`. | Нет                                            |
| modifiedDatetimeEnd      | То же, что и выше.                                               | Нет                                                          |
| енаблепартитиондисковери | Для секционированных файлов укажите, следует ли анализировать секции из пути к файлу и добавить их в качестве дополнительных исходных столбцов.<br/>Допустимые значения: **false** (по умолчанию) и **true**. | Нет                                            |
| партитионрутпас | Если обнаружение секций включено, укажите абсолютный корневой путь для чтения секционированных папок в виде столбцов данных.<br/><br/>Если значение не указано, по умолчанию<br/>— При использовании пути к файлу в наборе данных или списке файлов в источнике корневой путь к разделу — это путь, настроенный в наборе данных.<br/>— При использовании фильтра папки с подстановочными знаками корневой путь к разделу является вложенным путем перед первым подстановочным знаком.<br/><br/>Например, предположим, что вы настроили путь в наборе данных как «root/папка/year = 2020/месяц = 08/Day = 27»:<br/>— Если указать корневой путь к разделу "root/папка/year = 2020", действие копирования создаст еще два столбца `month` и `day` со значением "08" и "27" соответственно в дополнение к столбцам внутри файлов.<br/>— Если корневой путь секции не указан, лишний столбец не будет создан. | Нет                                            |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет                                                          |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromGoogleCloudStorage",
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
                    "type": "GoogleCloudStorageReadSettings",
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

В этом разделе описывается результирующее поведение при использовании пути к списку файлов в источнике действия копирования.

Предположим, что у вас есть следующая исходная структура папок и вы хотите скопировать файлы полужирным шрифтом:

| Пример исходной структуры                                      | Содержимое файла FileListToCopy.txt                             | Конфигурация фабрики данных                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Метаданные<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | Файл1.csv<br>Вложенная_папка1/Файл3.csv<br>Вложенная_папка1/Файл5.csv | **В наборе данных:**<br>- Контейнер: `bucket`<br>- Путь к папке: `FolderA`<br><br>**В источнике действия копирования:**<br>— Путь к списку файлов: `bucket/Metadata/FileListToCopy.txt` <br><br>Путь к списку файлов указывает на текстовый файл в том же хранилище данных, который содержит список файлов, которые необходимо скопировать, по одному файлу в строке с относительным путем к пути, заданному в наборе данных. |

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Свойства действия GetMetadata

Чтобы получить сведения о свойствах, проверьте [действие with Metadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Свойства действия удаления

Чтобы получить сведения о свойствах, установите флажок [действие удаления](delete-activity.md).

## <a name="legacy-models"></a>Устаревшие модели

Если вы используете соединитель Amazon S3 для копирования данных из Google Cloud Storage, он по-прежнему поддерживается в целях обратной совместимости. Мы рекомендуем использовать новую модель, упомянутую ранее. Пользовательский интерфейс создания фабрики данных переключился на создание новой модели.

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, которые действие копирования в фабрике данных Azure поддерживает в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats).
