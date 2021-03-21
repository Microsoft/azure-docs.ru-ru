---
title: Журнал сеанса в действии копирования
description: Узнайте, как включить журнал сеанса в действии копирования в фабрике данных Azure.
author: dearandyxu
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: yexu
ms.openlocfilehash: 7cb00d62556babbd8e43e2fac2faa815a63943ed
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100385274"
---
#  <a name="session-log-in-copy-activity"></a>Журнал сеанса в действии копирования

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Вы можете записать в журнал скопированные имена файлов в действии копирования, что может помочь в дальнейшей проверке того, что данные не копируются из источника в конечное хранилище, а также согласованы между исходным и целевым хранилищами путем просмотра скопированных файлов в журналах сеансов действий копирования.  

При включении параметра отказоустойчивости в действии копирования для пропуска неисправных данных также могут регистрироваться пропущенные файлы и пропущенные строки.  [В действии копирования](copy-activity-fault-tolerance.md)можно получить дополнительные сведения о отказоустойчивости. 

## <a name="configuration"></a>Конфигурация
В следующем примере представлено определение JSON для включения журнала сеанса в действии копирования: 

```json
"typeProperties": {
    "source": {
        "type": "BinarySource",
        "storeSettings": {
            "type": "AzureDataLakeStoreReadSettings",
            "recursive": true
        },
        "formatSettings": {
            "type": "BinaryReadSettings"
        }
    },
    "sink": {
        "type": "BinarySink",
        "storeSettings": {
            "type": "AzureBlobFSWriteSettings"
        }
    },                    
    "skipErrorFile": {
        "fileForbidden": true,
        "dataInconsistency": true
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

Свойство. | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | -------- 
енаблекопяктивитилог | Если задано значение true, вы сможете записывать в журнал скопированные файлы, пропущенные файлы или пропущенные строки.  | True<br/>False (по умолчанию) | Нет
LogLevel | "Info" будет записывать в журнал все скопированные файлы, пропущенные файлы и пропущенные строки. "Предупреждение" будет записывать пропущенные файлы и пропущенные строки.  | Info<br/>Предупреждение (по умолчанию) | Нет
енаблерелиаблелоггинг | Если это так, действие копирования в надежном режиме будет очищать журналы сразу после копирования каждого файла в место назначения.  При копировании огромных объемов файлов с включенным режимом надежного ведения журнала в действии копирования следует рассчитывать на то, что пропускная способность копирования будет затронута, поскольку для каждого копирования файлов требуются операции двойной записи. Один запрос относится к целевому хранилищу, а другой — к хранилищу хранилища журналов.  Действие копирования в режиме наилучших усилий приведет к сбросу журналов с пакетом записей в течение определенного периода времени, где пропускная способность копирования будет значительно ниже. В этом режиме не гарантируется полнота и своевременность ведения журнала, так как существует несколько возможных причин, когда последний пакет событий журнала не был записан в файл журнала при сбое действия копирования. В данный момент вы увидите, что несколько файлов, скопированных в место назначения, не регистрируются.  | True<br/>False (по умолчанию) | Нет
логлокатионсеттингс | Группа свойств, которые можно использовать для указания места хранения журналов сеанса. | | Нет
linkedServiceName | Связанная служба [хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) для хранения файлов журнала сеанса. | Имя связанной службы `AzureBlobStorage` или `AzureBlobFS`, которая ссылается на экземпляр хранилища, используемый для хранения файла журнала. | Нет
path | Путь к файлам журнала. | Укажите путь, по которому следует хранить файлы журнала. Если путь не указан, служба создаст контейнер самостоятельно. | нет


## <a name="monitoring"></a>Наблюдение

### <a name="output-from-copy-activity"></a>Выходные данные действия копирования
После выполнения действия копирования можно увидеть путь к файлам журнала из выходных данных каждого запуска действия копирования. Файлы журнала можно найти по пути: `https://[your-blob-account].blob.core.windows.net/[logFilePath]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv` .  Файлы журнала должны иметь формат CSV. 

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

> [!NOTE]
> Если `enableCopyActivityLog` свойство имеет значение `Enabled` , имена файлов журнала формируются системой.

### <a name="the-schema-of-the-log-file"></a>Схема файла журнала

Ниже приведена схема файла журнала.

Столбец | Описание 
-------- | -----------  
Отметка времени | Метка времени, когда ADF считывает, записывает или пропускает объект.
Level | Уровень ведения журнала для этого элемента. Может быть "warning" или "info".
OperationName | Рабочее поведение действия копирования ADF для каждого объекта. Это может быть "Филереад", "Филеврите", "Филескип" или "Табуларровскип".
OperationItem | Имена файлов или пропущенные строки.
Сообщение | Дополнительные сведения о том, что файл был считан из исходного хранилища или записан в целевое хранилище. Это также может быть причиной пропуска файла или строк.

Ниже приведен пример файла журнала. 
```
Timestamp, Level, OperationName, OperationItem, Message
2020-10-19 08:39:13.6688152,Info,FileRead,"sample1.csv","Start to read file: {""Path"":""sample1.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:39:56.3190846, Warning, FileSkip, "sample1.csv", "File is skipped after read 548000000 bytes: ErrorCode=DataConsistencySourceDataChanged,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Source file 'sample1.csv' is changed by other clients during the copy activity run.,Source=,'." 
2020-10-19 08:40:13.6688152,Info,FileRead,"sample2.csv","Start to read file: {""Path"":""sample2.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:40:13.9003981,Info,FileWrite,"sample2.csv","Start to write file from source file: sample2.csv."
2020-10-19 08:45:17.6508407,Info,FileRead,"sample2.csv","Complete reading file successfully. "
2020-10-19 08:45:28.7390083,Info,FileWrite,"sample2.csv","Complete writing file from source file: sample2.csv. File is successfully copied."
```
Из файла журнала, приведенного выше, можно увидеть, что файл sample1.csv был пропущен, поскольку его не удалось проверить на согласованность в исходном и целевом хранилищах. Дополнительные сведения о том, почему файл sample1.csv стал несогласованным, указывают на то, что он был изменен другими приложениями при выполнении действия копирования ADF. Вы также можете увидеть, sample2.csv успешно скопированы из источника в конечное хранилище.

Для дальнейшего анализа файлов журнала можно использовать несколько ядер анализа.  Ниже приведено несколько примеров использования SQL-запроса для анализа файла журнала путем импорта файла журнала CSV в базу данных SQL, где имя таблицы может быть Сессионлогдемо.  

-   Отправьте мне список скопированных файлов. 
```sql
select OperationItem from SessionLogDemo where Message like '%File is successfully copied%'
```

-   Отправьте мне список файлов, скопированный в определенный диапазон времени. 
```sql
select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%'
```

-   Дать мне определенный файл с скопированным временем и метаданными. 
```sql
select * from SessionLogDemo where OperationItem='<file name>'
```

-   Дайте мне список файлов с их метаданными, скопированными в пределах диапазона времени. 
```sql
select * from SessionLogDemo where OperationName='FileRead' and Message like 'Start to read%' and OperationItem in (select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%')
```

-   Выдать мне список пропущенных файлов. 
```sql
select OperationItem from SessionLogDemo where OperationName='FileSkip'
```

-   Дайте мне причину, по которой определенный файл пропущен. 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip'
```

-   Дать мне список пропущенных файлов по одной из причин: "файл BLOB не существует". 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip' and Message like '%UserErrorSourceBlobNotExist%'
```

-   Дайте мне имя файла, для которого требуется наибольшее время копирования.
```sql
select top 1 OperationItem, CopyDuration=DATEDIFF(SECOND, min(TIMESTAMP), max(TIMESTAMP)) from SessionLogDemo group by OperationItem order by CopyDuration desc
```


## <a name="next-steps"></a>Дальнейшие действия
См. другие статьи о действиях копирования:

- [Действие копирования в фабрике данных Azure](copy-activity-overview.md)
- [Отказоустойчивость действия копирования в фабрике данных Azure](copy-activity-fault-tolerance.md)
- [Согласованность данных действия копирования](copy-activity-data-consistency.md)
