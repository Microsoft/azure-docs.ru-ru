---
title: Передача больших наборов данных в методы автономного Azure Data Lake Storage 1-го поколения
description: Используйте службу импорта и экспорта для копирования данных из хранилища BLOB-объектов Azure в Azure Data Lake Storage 1-го поколения
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 940b7ac90f85e0254d59459b70ccc15312cd69f4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98700845"
---
# <a name="use-the-azure-importexport-service-for-offline-copy-of-data-to-data-lake-storage-gen1"></a>Используйте службу импорта и экспорта Azure для автономной копии данных в Data Lake Storage 1-го поколения

В этой статье вы узнаете, как скопировать огромные наборы данных (>200 ГБ) в Data Lake Storage 1-го поколения с помощью методов автономного копирования, таких как [Служба импорта и экспорта Azure](../import-export/storage-import-export-service.md). В частности, в качестве примера в этой статье используется файл размером 339 420 860 416 байт, т. е. 319 ГБ на диске. Давайте назовем этот файл 319GB.tsv.

Служба импорта и экспорта Azure позволяет безопасно переносить большие объемы данных в хранилище BLOB-объектов Azure, отправляя жесткие диски в центр обработки данных Azure.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы убедитесь, что у вас есть такие компоненты.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись хранения Azure.**
* **Учетная запись Azure Data Lake Storage 1-го поколения**. За инструкциями по созданию учетной записи обращайтесь к статье [Начало работы с Azure Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md).

## <a name="prepare-the-data"></a>Подготовка данных

Перед использованием службы импорта и экспорта необходимо разделить передаваемый файл данных на **копии, размер которых не превышает 200 ГБ**. Это нужно сделать, так как инструмент для импорта не работает с файлами, размер которых превышает 200 ГБ. В этой статье файл разбивается на фрагменты размером 100 ГБ. Это можно сделать с помощью [Cygwin](https://cygwin.com/install.html). Это средство поддерживает команды Linux. Воспользуйтесь следующей командой:

```console
split -b 100m 319GB.tsv
```

Операция Split создает файлы с именами, указанными ниже.

* *319. tsv-Part-AA*
* *319. tsv-Part-AB*
* *319. tsv. часть-AC*
* *319. tsv-Part-AD*

## <a name="get-disks-ready-with-data"></a>Подготовка дисков с данными

Для подготовки жестких дисков следуйте инструкциям в статье [Использование службы импорта и экспорта Azure для передачи данных в хранилище BLOB-объектов](../import-export/storage-import-export-service.md) в подразделе **Подготовка дисков**. Вот общая последовательность действий:

1. Приобретите жесткий диск, который отвечает требованиям к использованию в службе импорта и экспорта Azure.
2. Определите учетную запись хранения Azure, в которую будут скопированы данные после доставки диска в центр обработки данных Azure.
3. Используйте [инструмент импорта и экспорта Azure](https://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409) — служебную программу командной строки. Ниже приведен пример фрагмента кода, демонстрирующий использование инструмента.

    ```
    WAImportExport PrepImport /sk:<StorageAccountKey> /t: <TargetDriveLetter> /format /encrypt /logdir:e:\myexportimportjob\logdir /j:e:\myexportimportjob\journal1.jrn /id:myexportimportjob /srcdir:F:\demo\ExImContainer /dstdir:importcontainer/vf1/
    ```
    Дополнительные фрагменты кода см. в статье [Использование службы импорта и экспорта Azure для передачи данных в хранилище BLOB-объектов](../import-export/storage-import-export-service.md).
4. Приведенная выше команда создает файл журнала в указанном расположении. С его помощью вы создадите задание импорта из [портала Azure](https://portal.azure.com).

## <a name="create-an-import-job"></a>создание задания импорта;

Теперь можно создать задание импорта, воспользовавшись инструкциями в статье [Использование службы импорта и экспорта Azure для передачи данных в хранилище BLOB-объектов](../import-export/storage-import-export-service.md) в подразделе **Создание задания импорта**. Помимо других сведений укажите для этого задания импорта файл журнала, созданный при подготовке дисков.

## <a name="physically-ship-the-disks"></a>Физическая доставка дисков

Теперь можно выполнить физическую доставку дисков в центр обработки данных Azure. Там данные копируются в большие двоичные объекты службы хранилища Azure, указанные при создании задания импорта. Кроме того, если при создании задания было решено указать сведения об отслеживании позже, теперь можно вернуться к заданию импорта и обновить номер отслеживания.

## <a name="copy-data-from-blobs-to-data-lake-storage-gen1"></a>Копирование данных из больших двоичных объектов в Data Lake Storage 1-го поколения

После того как задание импорта перейдет в состояние "Завершено", можно проверить, доступны ли данные в указанных больших двоичных объектах службы хранилища Azure. Затем можно переместить данные из больших двоичных объектов в Azure Data Lake Storage 1-го поколения, используя различные методы. Сведения о всех доступных вариантах передачи данных см. в разделе [Прием данных в Azure Data Lake Storage 1-го поколения](data-lake-store-data-scenarios.md#ingest-data-into-data-lake-storage-gen1).

В этом разделе указаны определения JSON, с помощью которых можно создать конвейер фабрики данных Azure для копирования данных. Эти определения JSON доступны на [портале Azure](../data-factory/tutorial-copy-data-portal.md), а также в [Visual Studio](../data-factory/tutorial-copy-data-dot-net.md).

### <a name="source-linked-service-azure-storage-blob"></a>Связанная служба источника (большой двоичный объект службы хранилища Azure)

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="target-linked-service-data-lake-storage-gen1"></a>Целевая связанная служба (Data Lake Storage 1-го поколения)

```JSON
{
    "name": "AzureDataLakeStorageGen1LinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "description": "",
        "typeProperties": {
            "authorization": "<Click 'Authorize' to allow this data factory and the activities it runs to access this Data Lake Storage Gen1 account with your access rights>",
            "dataLakeStoreUri": "https://<adlsg1_account_name>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<OAuth session id from the OAuth authorization session. Each session id is unique and may only be used once>"
        }
    }
}
```

### <a name="input-data-set"></a>Входной набор данных

```JSON
{
    "name": "InputDataSet",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "importcontainer/vf1/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

### <a name="output-data-set"></a>Выходной набор данных

```JSON
{
"name": "OutputDataSet",
"properties": {
  "published": false,
  "type": "AzureDataLakeStore",
  "linkedServiceName": "AzureDataLakeStorageGen1LinkedService",
  "typeProperties": {
    "folderPath": "/importeddatafeb8job/"
    },
  "availability": {
    "frequency": "Hour",
    "interval": 1
    }
  }
}
```

### <a name="pipeline-copy-activity"></a>Конвейер (действие копирования)

```JSON
{
    "name": "CopyImportedData",
    "properties": {
        "description": "Pipeline with copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataSet"
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
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity"
            }
        ],
        "start": "2016-02-08T22:00:00Z",
        "end": "2016-02-08T23:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

Дополнительные сведения см. в статье [Копирование данных в Azure Data Lake Storage 1-го поколения и из него с помощью Фабрики данных Azure](../data-factory/connector-azure-data-lake-store.md).

## <a name="reconstruct-the-data-files-in-data-lake-storage-gen1"></a>Восстановление файлов данных в Data Lake Storage 1-го поколения

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Мы начали работу с файлом размером 319 ГБ и разделили его на файлы меньшего размера для передачи с помощью службы импорта и экспорта Azure. Теперь после передачи данных в Azure Data Lake Storage 1-го поколения мы можем воссоздать исходный файл. Для этого вы также можете воспользоваться приведенными ниже командлетами Azure PowerShell.

```PowerShell
# Login to our account
Connect-AzAccount

# List your subscriptions
Get-AzSubscription

# Switch to the subscription you want to work with
Set-AzContext -SubscriptionId
Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"

# Join  the files
Join-AzDataLakeStoreItem -AccountName "<adlsg1_account_name" -Paths "/importeddatafeb8job/319GB.tsv-part-aa","/importeddatafeb8job/319GB.tsv-part-ab", "/importeddatafeb8job/319GB.tsv-part-ac", "/importeddatafeb8job/319GB.tsv-part-ad" -Destination "/importeddatafeb8job/MergedFile.csv"
```

## <a name="next-steps"></a>Дальнейшие действия

* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
