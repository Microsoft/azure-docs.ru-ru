---
title: Отказоустойчивость действия копирования в Фабрике данных Azure
description: Узнайте, как обеспечить отказоустойчивость для действия копирования в Фабрике данных Azure (ADF) путем пропуска несовместимых данных.
author: dearandyxu
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: yexu
ms.openlocfilehash: 0fe1470661c006399ea176af1112d271524b2a1f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100390969"
---
#  <a name="fault-tolerance-of-copy-activity-in-azure-data-factory"></a>Отказоустойчивость действия копирования в Фабрике данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-copy-activity-fault-tolerance.md)
> * [Текущая версия](copy-activity-fault-tolerance.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

При копировании данных из исходного в целевое хранилище действие копирования в Фабрике данных Azure обеспечивает определенный уровень отказоустойчивости для предотвращения сбоев в ходе перемещения данных. Допустим, вы копируете миллионы строк из исходного в целевое хранилище, при этом в целевой базе данных создан первичный ключ, а в исходной базе данных первичные ключи не определены. При копировании повторяющихся строк произойдет сбой из-за нарушения первичного ключа в целевой базе данных. На данном этапе действие копирования предлагает два способа решения таких проблем: 
- Можно прервать действие копирования при возникновении какой-либо ошибки. 
- Можно продолжить копирование оставшихся элементов, включив отказоустойчивость, чтобы пропускать несовместимые данные. Например, можно пропустить повторяющуюся строку в этом случае. Кроме того, можно зарегистрировать пропущенные данные в журнале сеанса, включив его в рамках действия копирования. Дополнительные сведения см. в [журнале сеансов в действии копирования](copy-activity-log.md) .

## <a name="copying-binary-files"></a>Копирование двоичных файлов 

Служба ADF поддерживает следующие сценарии отказоустойчивости при копировании двоичных файлов. Вы можете прервать действие копирования или продолжить копирование остальных элементов в следующих сценариях:

1. Файлы, которые копируются службой ADF, в это же время удаляются другими приложениями.
2. Некоторые папки или файлы не позволяют службе ADF получить доступ, так как списки ACL для этих файлов или папок требуют более высокого уровня разрешений, чем у подключения, настроенного в ADF.
3. Один или несколько файлов не проверяются на согласованность между исходным и целевым хранилищами, если в ADF включена проверка согласованности данных.

### <a name="configuration"></a>Конфигурация 
При копировании двоичных файлов между хранилищами можно включить отказоустойчивость, как показано ниже. 

```json
"typeProperties": { 
    "source": { 
        "type": "BinarySource", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreReadSettings", 
            "recursive": true 
            } 
    }, 
    "sink": { 
        "type": "BinarySink", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreWriteSettings" 
        } 
    }, 
    "skipErrorFile": { 
        "fileMissing": true, 
        "fileForbidden": true, 
        "dataInconsistency": true,
        "invalidFileName": true        
    }, 
    "validateDataConsistency": true, 
    "logSettings": {
        "enableCopyActivityLog": true,
        "copyActivityLogSettings": {            
            "logLevel": "Warning",
            "enableReliableLogging": false
        },
        "logLocationSettings": {
            "linkedServiceName": {
               "referenceName": "ADLSGen2",
               "type": "LinkedServiceReference"
            },
            "path": "sessionlog/"
        }
    }
} 
```
Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | -------- 
skipErrorFile | Группа свойств для указания типов сбоев, которые необходимо пропустить во время перемещения данных. | | нет
fileMissing | Одна из пар "ключ-значение" в контейнере свойств skipErrorFile, определяющая необходимость пропуска файлов, которые удаляются другими приложениями во время копировании службой ADF. <br/> – True: необходимо скопировать остальные файлы, пропуская файлы, удаляемые другими приложениями. <br/> – False: необходимо прервать действие копирования в случае удаления каких-либо файлов из исходного хранилища в ходе перемещения данных. <br/>Обратите внимание, что по умолчанию это свойство имеет значение True. | True (по умолчанию) <br/>False | нет
fileForbidden | Одна из пар "ключ-значение" в контейнере свойств skipErrorFile, определяющая необходимость пропуска определенных файлов, если списки ACL для этих файлов или папок требуют более высокого уровня разрешений, чем у подключения, настроенного в ADF. <br/> – True: необходимо скопировать остальные файлы, пропуская эти. <br/> – False: необходимо прервать действие копирования при возникновении проблемы с разрешениями для папок или файлов. | True <br/>False (по умолчанию) | нет
dataInconsistency | Одна из пар "ключ-значение" в контейнере свойств skipErrorFile, определяющая необходимость пропуска несогласованных данных между исходным и целевым хранилищами. <br/> – True: необходимо скопировать остальные данные, пропуская несогласованные. <br/> – False: необходимо прервать действие копирования при обнаружении несогласованных данных. <br/>Обратите внимание, что это свойство действительно, только если свойство validateDataConsistency имеет значение True. | True <br/>False (по умолчанию) | нет
инвалидфиленаме | Одна из пар "ключ-значение" в контейнере свойств Скиперрорфиле для определения необходимости пропуска определенных файлов, если имена файлов недопустимы для целевого хранилища. <br/> — True: необходимо скопировать оставшуюся версию, пропуская файлы, имеющие недопустимые имена файлов. <br/> -False: вы хотите прервать действие копирования, если у файлов есть недопустимые имена файлов. <br/>Имейте в виду, что это свойство работает при копировании двоичных файлов из любого хранилища в ADLS 2-го поколения или копирования двоичных файлов из AWS S3 в любое хранилище хранилища. | True <br/>False (по умолчанию) | нет
логсеттингс  | Группа свойств, которые можно указать, если вы хотите зарегистрировать имена пропущенных объектов в журнале. | &nbsp; | нет
linkedServiceName | Связанная служба [хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) для хранения файлов журнала сеанса. | Имя связанной службы `AzureBlobStorage` или `AzureBlobFS`, которая ссылается на экземпляр хранилища, используемый для хранения файла журнала. | нет
path | Путь к файлам журнала. | Укажите путь, используемый для хранения файлов журнала. Если путь не указан, служба создаст контейнер самостоятельно. | Нет

> [!NOTE]
> Ниже приведены предварительные требования к включению отказоустойчивости в действии копирования при копировании двоичных файлов.
> Для пропуска определенных файлов при их удалении из исходного хранилища:
> - Исходный набор данных и набор данных приемника должны иметь двоичный формат, и тип сжатия указывать нельзя. 
> - Поддерживаемые типы хранилища данных: хранилище BLOB-объектов Azure, Azure Data Lake Storage 1-го поколения, Azure Data Lake Storage 2-го поколения, хранилище файлов Azure, файловая система, FTP, SFTP, Amazon S3, Google Cloud Storage и HDFS.
> - Только при указании нескольких файлов в исходном наборе данных, который может быть папкой, подстановочным знаком или списком файлов, действие копирования может пропустить определенные файлы ошибок. Если один файл указан в исходном наборе данных для копирования в назначение, действие копирования завершится ошибкой, если произошла какая-либо ошибка.
>
> Для пропуска определенных файлов, если их доступ запрещен из исходного хранилища:
> - Исходный набор данных и набор данных приемника должны иметь двоичный формат, и тип сжатия указывать нельзя. 
> - Поддерживаемые типы хранилища данных: хранилище BLOB-объектов Azure, Azure Data Lake Storage 1-го поколения, Azure Data Lake Storage 2-го поколения, хранилище файлов Azure, SFTP, Amazon S3 и HDFS.
> - Только при указании нескольких файлов в исходном наборе данных, который может быть папкой, подстановочным знаком или списком файлов, действие копирования может пропустить определенные файлы ошибок. Если один файл указан в исходном наборе данных для копирования в назначение, действие копирования завершится ошибкой, если произошла какая-либо ошибка.
>
> Для пропуска определенных файлов при проверке на несогласованность исходного и конечного хранилища:
> - Дополнительные сведения [см. в](./copy-activity-data-consistency.md)документации по согласованности данных.

### <a name="monitoring"></a>Наблюдение 

#### <a name="output-from-copy-activity"></a>Выходные данные действия копирования
С помощью выходных данных каждого выполнения действия копирования можно получить количество считанных, записанных и пропущенных файлов. 

```json
"output": {
            "dataRead": 695,
            "dataWritten": 186,
            "filesRead": 3,  
            "filesWritten": 1, 
            "filesSkipped": 2, 
            "throughput": 297,
            "logFilePath": "myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "dataConsistencyVerification": 
           { 
                "VerificationResult": "Verified", 
                "InconsistentData": "Skipped" 
           } 
        }

```

#### <a name="session-log-from-copy-activity"></a>Журнал сеанса действия копирования

Если вы настроили регистрацию имен пропущенных файлов в журнале, файл журнала можно найти по этому пути:`https://[your-blob-account].blob.core.windows.net/[path-if-configured]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv` 

Файлы журнала должны иметь формат CSV. Ниже приведена схема файла журнала.

Столбец | Описание 
-------- | -----------  
Timestamp | Метка времени пропуска файла службой ADF.
Level | Уровень ведения журнала для этого элемента. Для элемента, обозначающего пропуск файла, будет использоваться уровень Warning.
OperationName | Рабочее поведение действия копирования ADF для каждого файла. Для пропуска файла будет использоваться значение FileSkip.
OperationItem | Имена пропускаемых файлов.
Message | Дополнительные сведения о причине пропуска файла.

Ниже приведен пример файла журнала. 
```
Timestamp,Level,OperationName,OperationItem,Message 
2020-03-24 05:35:41.0209942,Warning,FileSkip,"bigfile.csv","File is skipped after read 322961408 bytes: ErrorCode=UserErrorSourceBlobNotExist,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=The required Blob is missing. ContainerName: https://transferserviceonebox.blob.core.windows.net/skipfaultyfile, path: bigfile.csv.,Source=Microsoft.DataTransfer.ClientLibrary,'." 
2020-03-24 05:38:41.2595989,Warning,FileSkip,"3_nopermission.txt","File is skipped after read 0 bytes: ErrorCode=AdlsGen2OperationFailed,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=ADLS Gen2 operation failed for: Operation returned an invalid status code 'Forbidden'. Account: 'adlsgen2perfsource'. FileSystem: 'skipfaultyfilesforbidden'. Path: '3_nopermission.txt'. ErrorCode: 'AuthorizationPermissionMismatch'. Message: 'This request is not authorized to perform this operation using this permission.'. RequestId: '35089f5d-101f-008c-489e-01cce4000000'..,Source=Microsoft.DataTransfer.ClientLibrary,''Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Operation returned an invalid status code 'Forbidden',Source=,''Type=Microsoft.Azure.Storage.Data.Models.ErrorSchemaException,Message='Type=Microsoft.Azure.Storage.Data.Models.ErrorSchemaException,Message=Operation returned an invalid status code 'Forbidden',Source=Microsoft.DataTransfer.ClientLibrary,',Source=Microsoft.DataTransfer.ClientLibrary,'." 
```
Из журнала выше можно узнать, что файл bigfile.csv был пропущен из-за того, что другое приложение удалило его во время копирования службой ADF. А файл 3_nopermission.txt был пропущен, так как службе ADF не удалось получить к нему доступ из-за проблемы с разрешениями.


## <a name="copying-tabular-data"></a>Копирование табличных данных 

### <a name="supported-scenarios"></a>Поддерживаемые сценарии
Действие копирования поддерживает три сценария для обработки несовместимых табличных данных: обнаружение, пропуск и запись в журнал.

- **Несовместимость между типом данных источника и собственным типом приемника**. 

    Пример: выполняется копирование данных из CSV-файла в хранилище BLOB-объектов в базу данных SQL с использованием определения схемы, которое содержит три столбца типа INT. Строки CSV-файла, которые содержат числовые данные, например 123,456,789, успешно копируются в хранилище приемника. А строки, которые содержат нечисловые значения, например 123,456, abc, определяются как несовместимые и пропускаются.

- **Несоответствие числа столбцов между источником и приемником**.

    Пример: выполняется копирование данных из CSV-файла в хранилище BLOB-объектов в базу данных SQL с использованием определения схемы, которое содержит шесть столбцов. Строки CSV-файла, которые содержат шесть столбцов, успешно копируются в хранилище приемника. Строки CSV-файла, которые содержат больше шести столбцов, определяются как несовместимые и пропускаются.

- **Нарушение первичного ключа при записи в SQL Server, Базу данных SQL Azure или Azure Cosmos DB**.

    Пример: выполняется копирование данных из SQL Server в базу данных SQL. Определен первичный ключ в базе данных SQL приемников, но такой первичный ключ не определен на исходном сервере SQL Server. Повторяющиеся строки, которые существуют в источнике, не удается скопировать в приемник. Действие копирования копирует только первую строку исходных данных в приемник. Последующие строки в источнике, которые содержат повторяющиеся значения первичного ключа, определяются как несовместимые и пропускаются.

>[!NOTE]
>- Чтобы загрузить данные в Azure синапсе Analytics с помощью Polybase, настройте параметры отказоустойчивого отказоустойчивости Polybase, указав политики отклонения через "[polyBaseSettings](connector-azure-sql-data-warehouse.md#azure-sql-data-warehouse-as-sink)" в действии копирования. По-прежнему можно включить перенаправление несовместимых строк PolyBase в BLOB-объект или ADLS в обычном режиме, как показано ниже.
>- Эта функция не применяется, когда действие копирования настроено на вызов [Amazon Redshift Unload](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift).
>- Эта функция не применяется, когда действие копирования настроено на вызов [хранимой процедуры их приемника SQL](./connector-azure-sql-database.md#invoke-a-stored-procedure-from-a-sql-sink).

### <a name="configuration"></a>Конфигурация
В следующем примере приводится определение JSON для настройки пропуска несовместимых строк в действии копирования.

```json
"typeProperties": { 
    "source": { 
        "type": "AzureSqlSource" 
    }, 
    "sink": { 
        "type": "AzureSqlSink" 
    }, 
    "enableSkipIncompatibleRow": true, 
    "logSettings": {
        "enableCopyActivityLog": true,
        "copyActivityLogSettings": {            
            "logLevel": "Warning",
            "enableReliableLogging": false
        },
        "logLocationSettings": {
            "linkedServiceName": {
               "referenceName": "ADLSGen2",
               "type": "LinkedServiceReference"
            },
            "path": "sessionlog/"
        }
    } 
}, 
```

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | -------- 
enableSkipIncompatibleRow | Указывает, следует ли пропустить несовместимые строки во время копирования или нет. | True<br/>False (по умолчанию) | Нет
логсеттингс | Группа свойств, которые можно указать, если вы хотите записать несовместимые строки в журнал. | &nbsp; | нет
linkedServiceName | Связанная [служба хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) для хранения журнала, содержащего пропущенные строки. | Имя связанной службы `AzureBlobStorage` или `AzureBlobFS`, которая ссылается на экземпляр хранилища, используемый для хранения файла журнала. | нет
path | Путь к файлам журнала, содержащего пропущенные строки. | Укажите путь к журналу, в котором будут храниться несовместимые данные. Если путь не указан, служба создаст контейнер самостоятельно. | нет

### <a name="monitor-skipped-rows"></a>Мониторинг пропущенных строк
Когда действие копирования будет завершено, вы увидите число пропущенных строк в выходных данных действия копирования:

```json
"output": {
            "dataRead": 95,
            "dataWritten": 186,
            "rowsCopied": 9,
            "rowsSkipped": 2,
            "copyDuration": 16,
            "throughput": 0.01,
            "logFilePath": "myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "errors": []
        },

```

Если вы настроили регистрацию несовместимых строк в журнале, файл журнала можно найти по этому пути: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv`. 

Файлы журнала должны иметь формат CSV. Ниже приведена схема файла журнала.

Столбец | Описание 
-------- | -----------  
Timestamp | Метка времени пропуска несовместимых строк службой ADF.
Level | Уровень ведения журнала для этого элемента. Для элемента, обозначающего пропуск строк, будет использоваться уровень Warning.
OperationName | Рабочее поведение действия копирования ADF для каждой строки. Для пропуска несовместимой строки будет использоваться значение TabularRowSkip.
OperationItem | Пропущенные данные из исходного хранилища данных.
Message | Дополнительные сведения о причине несовместимости конкретной строки.


Ниже приведен пример содержимого файла журнала.

```
Timestamp, Level, OperationName, OperationItem, Message
2020-02-26 06:22:32.2586581, Warning, TabularRowSkip, """data1"", ""data2"", ""data3""," "Column 'Prop_2' contains an invalid value 'data3'. Cannot convert 'data3' to type 'DateTime'." 
2020-02-26 06:22:33.2586351, Warning, TabularRowSkip, """data4"", ""data5"", ""data6"",", "Violation of PRIMARY KEY constraint 'PK_tblintstrdatetimewithpk'. Cannot insert duplicate key in object 'dbo.tblintstrdatetimewithpk'. The duplicate key value is (data4)." 
```

Из приведенного выше примера файла журнала можно увидеть, что одна строка, "data1, data2, data3", была пропущена из-за проблемы с преобразованием типов при копировании из исходного в целевое хранилище. Другая строка, "data4, data5, data6", была пропущена из-за проблемы с нарушением первичного ключа в целевом хранилище. 


## <a name="copying-tabular-data-legacy"></a>Копирование табличных данных (устаревший способ)

Ниже описан устаревший способ обеспечения отказоустойчивости, который применяется только при копировании табличных данных. При создании нового конвейера или действия вместо него рекомендуется начать с [этого](#copying-tabular-data).

### <a name="configuration"></a>Конфигурация
В следующем примере приводится определение JSON для настройки пропуска несовместимых строк в действии копирования.

```json
"typeProperties": {
    "source": {
        "type": "BlobSource"
    },
    "sink": {
        "type": "SqlSink",
    },
    "enableSkipIncompatibleRow": true,
    "redirectIncompatibleRowSettings": {
         "linkedServiceName": {
              "referenceName": "<Azure Storage or Data Lake Store linked service>",
              "type": "LinkedServiceReference"
            },
            "path": "redirectcontainer/erroroutput"
     }
}
```

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | -------- 
enableSkipIncompatibleRow | Указывает, следует ли пропустить несовместимые строки во время копирования или нет. | True<br/>False (по умолчанию) | нет
redirectIncompatibleRowSettings | Группа свойств, которые можно указать, если вы хотите записать несовместимые строки в журнал. | &nbsp; | нет
linkedServiceName | Связанная [служба хранилища Azure](connector-azure-blob-storage.md#linked-service-properties) или [Azure Data Lake Store](connector-azure-data-lake-store.md#linked-service-properties), в которой будет храниться журнал, содержащий пропущенные строки. | Имя связанной службы `AzureStorage` или `AzureDataLakeStore`, которая ссылается на экземпляр хранилища, используемый для хранения файла журнала. | нет
path | Путь к файлу журнала, который содержит пропущенные строки. | Укажите путь к журналу, в котором будут храниться несовместимые данные. Если путь не указан, служба создаст контейнер самостоятельно. | нет

### <a name="monitor-skipped-rows"></a>Мониторинг пропущенных строк
Когда действие копирования будет завершено, вы увидите число пропущенных строк в выходных данных действия копирования:

```json
"output": {
            "dataRead": 95,
            "dataWritten": 186,
            "rowsCopied": 9,
            "rowsSkipped": 2,
            "copyDuration": 16,
            "throughput": 0.01,
            "redirectRowPath": "https://myblobstorage.blob.core.windows.net//myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "errors": []
        },

```
При настройке записи несовместимых строк файл журнала можно найти по этому пути: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/[copy-activity-run-id]/[auto-generated-GUID].csv`. 

Файлы журнала должны быть представлены только в формате CSV. Пропущенные исходные данные при необходимости регистрируются с использованием запятой в качестве разделителя столбцов. Добавим к исходным данным источника в файле журнала два дополнительных столбца — ErrorCode и ErrorMessage. В них содержатся сведения о первопричине несовместимости. ErrorCode и ErrorMessage указываются в двойных кавычках. 

Ниже приведен пример содержимого файла журнала:

```
data1, data2, data3, "UserErrorInvalidDataValue", "Column 'Prop_2' contains an invalid value 'data3'. Cannot convert 'data3' to type 'DateTime'."
data4, data5, data6, "2627", "Violation of PRIMARY KEY constraint 'PK_tblintstrdatetimewithpk'. Cannot insert duplicate key in object 'dbo.tblintstrdatetimewithpk'. The duplicate key value is (data4)."
```

## <a name="next-steps"></a>Дальнейшие действия
См. другие статьи о действии копирования:

- [Действие копирования в фабрике данных Azure](copy-activity-overview.md)
- [Руководство по настройке производительности действия копирования](copy-activity-performance.md)