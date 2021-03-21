---
title: Действие Delete в Фабрике данных Azure
description: Сведения о том, как удалять файлы из различных хранилищ файлов с помощью действия Delete в Фабрике данных Azure.
author: dearandyxu
ms.author: yexu
ms.service: data-factory
ms.topic: conceptual
ms.date: 08/12/2020
ms.openlocfilehash: 3021d29f472dbbf43ae53981287b1f4676e8f932
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392720"
---
# <a name="delete-activity-in-azure-data-factory"></a>Действие Delete в Фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


Действие удаление в фабрике данных Azure можно использовать для удаления файлов или папок из локальных хранилищ хранилищ или облачных хранилищ хранилища. Это действие используется для очистки или архивации ненужных файлов.

> [!WARNING]
> Удаленные файлы или папки нельзя восстановить (если только в хранилище не включено обратимое удаление). Учитывайте это при использовании действия Delete для удаления файлов или папок.

## <a name="best-practices"></a>Рекомендации

Здесь приведено несколько рекомендаций по использованию действия Delete.

-   Создавайте резервные копии файлов перед их удалением с помощью действия Delete на случай, если вам потребуется восстановить эти файлы в будущем.

-   Убедитесь, что Фабрика данных имеет права на запись, чтобы удалять папки или файлы из хранилища.

-   Убедитесь, что вы не удаляете файлы, которые тем временем записываются. 

-   Если вы хотите удалить файлы или папки из локальной системы, убедитесь, что вы используете локальную среду выполнения интеграции с версией более 3,14.

## <a name="supported-data-stores"></a>Поддерживаемые хранилища данных

-   [Хранилище BLOB-объектов Azure](connector-azure-blob-storage.md)
-   [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md)
-   [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md)
-   [Хранилище файлов Azure](connector-azure-file-storage.md)
-   [Файловая система](connector-file-system.md)
-   [FTP](connector-ftp.md)
-   [SFTP](connector-sftp.md)
-   [Amazon S3](connector-amazon-simple-storage-service.md)
-   [Google Cloud Storage](connector-google-cloud-storage.md)
-   [HDFS](connector-hdfs.md)

## <a name="syntax"></a>Синтаксис

```json
{
    "name": "DeleteActivity",
    "type": "Delete",
    "typeProperties": {
        "dataset": {
            "referenceName": "<dataset name>",
            "type": "DatasetReference"
        },
        "storeSettings": {
            "type": "<source type>",
            "recursive": true/false,
            "maxConcurrentConnections": <number>
        },
        "enableLogging": true/false,
        "logStorageSettings": {
            "linkedServiceName": {
                "referenceName": "<name of linked service>",
                "type": "LinkedServiceReference"
            },
            "path": "<path to save log file>"
        }
    }
}
```

## <a name="type-properties"></a>Свойства типа

| Свойство | Описание | Обязательно |
| --- | --- | --- |
| набор данных | Предоставляет ссылку на набор данных, чтобы определить, какие файлы или папки должны быть удалены. | Да |
| recursive | Указывает, откуда файлы удаляются рекурсивно: из вложенных папок или только из указанной папки.  | Нет. Значение по умолчанию — `false`. |
| maxConcurrentConnections | Количество одновременных подключений к хранилищу с целью удаления папок или файлов.   |  Нет. Значение по умолчанию — `1`. |
| enablelogging | Указывает, нужно ли записывать имена удаленных папок или файлов. Если значение равно true, необходимо дополнительно предоставить учетную запись хранения для сохранения файла журнала, чтобы можно было в нем отслеживать поведение действия Delete. | Нет |
| logStorageSettings | Это свойство применимо, только если для параметра enablelogging установлено значение true.<br/><br/>Группа свойств хранилища, в котором будет храниться файл журнала, содержащий имена файлов и папок, которые были удалены действием Delete. | Нет |
| linkedServiceName | Это свойство применимо, только если для параметра enablelogging установлено значение true.<br/><br/>Связанная служба [хранилища Azure](connector-azure-blob-storage.md#linked-service-properties), [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md#linked-service-properties)или [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) для хранения файла журнала, содержащего имя папки или файла, которые были удалены действием удаления. Следует иметь в виду, что для удаления файлов необходимо настроить тот же тип Integration Runtime из того, который использовался действием DELETE. | Нет |
| path | Это свойство применимо, только если для параметра enablelogging установлено значение true.<br/><br/>Путь, по которому хранится файл журнала в учетной записи хранения. Если путь не указан, служба создаст контейнер самостоятельно. | нет |

## <a name="monitoring"></a>Наблюдение

Отслеживать результаты действия Delete можно в двух местах: 
-   выходные данные действия Delete;
-   файл журнала.

### <a name="sample-output-of-the-delete-activity"></a>Пример выходных данных действия Delete

```json
{ 
  "datasetName": "AmazonS3",
  "type": "AmazonS3Object",
  "prefix": "test",
  "bucketName": "adf",
  "recursive": true,
  "isWildcardUsed": false,
  "maxConcurrentConnections": 2,  
  "filesDeleted": 4,
  "logPath": "https://sample.blob.core.windows.net/mycontainer/5c698705-a6e2-40bf-911e-e0a927de3f07",
  "effectiveIntegrationRuntime": "MyAzureIR (West Central US)",
  "executionDuration": 650
}
```

### <a name="sample-log-file-of-the-delete-activity"></a>Пример файла журнала действия Delete

| Имя | Category | Состояние | Ошибка |
|:--- |:--- |:--- |:--- |
| test1/yyy.jsв | Файл | Удаленная |  |
| test2/hello789.txt | Файл | Удаленная |  |
| test2/test3/hello000.txt | Файл | Удаленная |  |
| test2/test3/zzz.jsвкл. | Файл | Удаленная |  |

## <a name="examples-of-using-the-delete-activity"></a>Примеры использования действия Delete

### <a name="delete-specific-folders-or-files"></a>Удаление конкретных файлов или папок

Хранилище состоит из следующей структуры папок:

Root/<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8.txt

Теперь вы используете действие Delete для удаления папок или файлов с помощью комбинации различных значений свойств действия Delete и набора данных.

| folderPath | fileName | recursive | Выходные данные |
|:--- |:--- |:--- |:--- |
| Root/ Folder_A_2 | NULL | Неверно | Root/<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>4.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>5.csv</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8.txt |
| Root/ Folder_A_2 | NULL | Верно | Root/<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;<strike>Folder_A_2/</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>4.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>5.csv</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>Folder_B_1/</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>6.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>7.csv</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>Folder_B_2/</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>8.txt</strike> |
| Root/ Folder_A_2 | *.txt | Неверно | Root/<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>4.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8.txt |
| Root/ Folder_A_2 | *.txt | Верно | Root/<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.txt<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;Folder_A_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>4.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_1/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>6.txt</strike><br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Folder_B_2/<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strike>8.txt</strike> |

### <a name="periodically-clean-up-the-time-partitioned-folder-or-files"></a>Периодическое очищение разделенных по времени папок или файлов

Вы можете создать конвейер для периодической очистки разделенных по времени папок или файлов.  Например, со структурой папок, похожей на `/mycontainer/2018/12/14/*.csv`.  Вы можете использовать системную переменную ADF триггера планировщика, чтобы определить папки и файлы, которые должны быть удалены при каждом запуске конвейера. 

#### <a name="sample-pipeline"></a>Пример конвейера

```json
{
    "name":"cleanup_time_partitioned_folder",
    "properties":{
        "activities":[
            {
                "name":"DeleteOneFolder",
                "type":"Delete",
                "dependsOn":[

                ],
                "policy":{
                    "timeout":"7.00:00:00",
                    "retry":0,
                    "retryIntervalInSeconds":30,
                    "secureOutput":false,
                    "secureInput":false
                },
                "userProperties":[

                ],
                "typeProperties":{
                    "dataset":{
                        "referenceName":"PartitionedFolder",
                        "type":"DatasetReference",
                        "parameters":{
                            "TriggerTime":{
                                "value":"@formatDateTime(pipeline().parameters.TriggerTime, 'yyyy/MM/dd')",
                                "type":"Expression"
                            }
                        }
                    },
                    "logStorageSettings":{
                        "linkedServiceName":{
                            "referenceName":"BloblinkedService",
                            "type":"LinkedServiceReference"
                        },
                        "path":"mycontainer/log"
                    },
                    "enableLogging":true,
                    "storeSettings":{
                        "type":"AzureBlobStorageReadSettings",
                        "recursive":true
                    }
                }
            }
        ],
        "parameters":{
            "TriggerTime":{
                "type":"string"
            }
        },
        "annotations":[

        ]
    }
}
```

#### <a name="sample-dataset"></a>Пример набора данных

```json
{
    "name":"PartitionedFolder",
    "properties":{
        "linkedServiceName":{
            "referenceName":"BloblinkedService",
            "type":"LinkedServiceReference"
        },
        "parameters":{
            "TriggerTime":{
                "type":"string"
            }
        },
        "annotations":[

        ],
        "type":"Binary",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "folderPath":{
                    "value":"@dataset().TriggerTime",
                    "type":"Expression"
                },
                "container":{
                    "value":"mycontainer",
                    "type":"Expression"
                }
            }
        }
    }
}
```

#### <a name="sample-trigger"></a>Пример триггера

```json
{
    "name": "DailyTrigger",
    "properties": {
        "runtimeState": "Started",
        "pipelines": [
            {
                "pipelineReference": {
                    "referenceName": "cleanup_time_partitioned_folder",
                    "type": "PipelineReference"
                },
                "parameters": {
                    "TriggerTime": "@trigger().scheduledTime"
                }
            }
        ],
        "type": "ScheduleTrigger",
        "typeProperties": {
            "recurrence": {
                "frequency": "Day",
                "interval": 1,
                "startTime": "2018-12-13T00:00:00.000Z",
                "timeZone": "UTC",
                "schedule": {
                    "minutes": [
                        59
                    ],
                    "hours": [
                        23
                    ]
                }
            }
        }
    }
}
```

### <a name="clean-up-the-expired-files-that-were-last-modified-before-201811"></a>Очистите файлы с истекшим сроком хранения, которые подвергались модификации еще до 1 января 2018 г.

Вы можете создать конвейер для очистки старых или устаревших файлов, используя фильтр атрибутов файлов: "LastModified" в наборе данных.  

#### <a name="sample-pipeline"></a>Пример конвейера

```json
{
    "name":"CleanupExpiredFiles",
    "properties":{
        "activities":[
            {
                "name":"DeleteFilebyLastModified",
                "type":"Delete",
                "dependsOn":[

                ],
                "policy":{
                    "timeout":"7.00:00:00",
                    "retry":0,
                    "retryIntervalInSeconds":30,
                    "secureOutput":false,
                    "secureInput":false
                },
                "userProperties":[

                ],
                "typeProperties":{
                    "dataset":{
                        "referenceName":"BlobFilesLastModifiedBefore201811",
                        "type":"DatasetReference"
                    },
                    "logStorageSettings":{
                        "linkedServiceName":{
                            "referenceName":"BloblinkedService",
                            "type":"LinkedServiceReference"
                        },
                        "path":"mycontainer/log"
                    },
                    "enableLogging":true,
                    "storeSettings":{
                        "type":"AzureBlobStorageReadSettings",
                        "recursive":true,
                        "modifiedDatetimeEnd":"2018-01-01T00:00:00.000Z"
                    }
                }
            }
        ],
        "annotations":[

        ]
    }
}
```

#### <a name="sample-dataset"></a>Пример набора данных

```json
{
    "name":"BlobFilesLastModifiedBefore201811",
    "properties":{
        "linkedServiceName":{
            "referenceName":"BloblinkedService",
            "type":"LinkedServiceReference"
        },
        "annotations":[

        ],
        "type":"Binary",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "fileName":"*",
                "folderPath":"mydirectory",
                "container":"mycontainer"
            }
        }
    }
}
```

### <a name="move-files-by-chaining-the-copy-activity-and-the-delete-activity"></a>Перемещение файлов с помощью связывания действия копирования и действия Delete

Вы можете переместить файл, используя действие копирования, чтобы скопировать файл, а затем действие удаления для удаления файла в конвейере.  Если вы хотите переместить несколько файлов, можно использовать действие GetMetadata + действие Filter + действие ForEach + действие копирования + действие Delete, как показано в следующем примере.

> [!NOTE]
> Нужно быть осторожным, если вы хотите переместить всю папку, определив набор данных, содержащий только путь к папке, а затем используя действие копирования и действие Delete для указания ссылки на тот же набор данных, представляющий папку. Дело в том, что новые файлы НЕ должны поступать в папку между операциями копирования и удаления.  Если же в папку поступили новые файлы сразу после того, как действие копирования завершило копирование файлов, а действие Delete еще не начало работу, вероятно, что действие Delete удалит новые файлы, которые еще НЕ успели скопироваться в назначенное место. 

#### <a name="sample-pipeline"></a>Пример конвейера

```json
{
    "name":"MoveFiles",
    "properties":{
        "activities":[
            {
                "name":"GetFileList",
                "type":"GetMetadata",
                "dependsOn":[

                ],
                "policy":{
                    "timeout":"7.00:00:00",
                    "retry":0,
                    "retryIntervalInSeconds":30,
                    "secureOutput":false,
                    "secureInput":false
                },
                "userProperties":[

                ],
                "typeProperties":{
                    "dataset":{
                        "referenceName":"OneSourceFolder",
                        "type":"DatasetReference",
                        "parameters":{
                            "Container":{
                                "value":"@pipeline().parameters.SourceStore_Location",
                                "type":"Expression"
                            },
                            "Directory":{
                                "value":"@pipeline().parameters.SourceStore_Directory",
                                "type":"Expression"
                            }
                        }
                    },
                    "fieldList":[
                        "childItems"
                    ],
                    "storeSettings":{
                        "type":"AzureBlobStorageReadSettings",
                        "recursive":true
                    },
                    "formatSettings":{
                        "type":"BinaryReadSettings"
                    }
                }
            },
            {
                "name":"FilterFiles",
                "type":"Filter",
                "dependsOn":[
                    {
                        "activity":"GetFileList",
                        "dependencyConditions":[
                            "Succeeded"
                        ]
                    }
                ],
                "userProperties":[

                ],
                "typeProperties":{
                    "items":{
                        "value":"@activity('GetFileList').output.childItems",
                        "type":"Expression"
                    },
                    "condition":{
                        "value":"@equals(item().type, 'File')",
                        "type":"Expression"
                    }
                }
            },
            {
                "name":"ForEachFile",
                "type":"ForEach",
                "dependsOn":[
                    {
                        "activity":"FilterFiles",
                        "dependencyConditions":[
                            "Succeeded"
                        ]
                    }
                ],
                "userProperties":[

                ],
                "typeProperties":{
                    "items":{
                        "value":"@activity('FilterFiles').output.value",
                        "type":"Expression"
                    },
                    "batchCount":20,
                    "activities":[
                        {
                            "name":"CopyAFile",
                            "type":"Copy",
                            "dependsOn":[

                            ],
                            "policy":{
                                "timeout":"7.00:00:00",
                                "retry":0,
                                "retryIntervalInSeconds":30,
                                "secureOutput":false,
                                "secureInput":false
                            },
                            "userProperties":[

                            ],
                            "typeProperties":{
                                "source":{
                                    "type":"BinarySource",
                                    "storeSettings":{
                                        "type":"AzureBlobStorageReadSettings",
                                        "recursive":false,
                                        "deleteFilesAfterCompletion":false
                                    },
                                    "formatSettings":{
                                        "type":"BinaryReadSettings"
                                    },
                                    "recursive":false
                                },
                                "sink":{
                                    "type":"BinarySink",
                                    "storeSettings":{
                                        "type":"AzureBlobStorageWriteSettings"
                                    }
                                },
                                "enableStaging":false,
                                "dataIntegrationUnits":0
                            },
                            "inputs":[
                                {
                                    "referenceName":"OneSourceFile",
                                    "type":"DatasetReference",
                                    "parameters":{
                                        "Container":{
                                            "value":"@pipeline().parameters.SourceStore_Location",
                                            "type":"Expression"
                                        },
                                        "Directory":{
                                            "value":"@pipeline().parameters.SourceStore_Directory",
                                            "type":"Expression"
                                        },
                                        "filename":{
                                            "value":"@item().name",
                                            "type":"Expression"
                                        }
                                    }
                                }
                            ],
                            "outputs":[
                                {
                                    "referenceName":"OneDestinationFile",
                                    "type":"DatasetReference",
                                    "parameters":{
                                        "Container":{
                                            "value":"@pipeline().parameters.DestinationStore_Location",
                                            "type":"Expression"
                                        },
                                        "Directory":{
                                            "value":"@pipeline().parameters.DestinationStore_Directory",
                                            "type":"Expression"
                                        },
                                        "filename":{
                                            "value":"@item().name",
                                            "type":"Expression"
                                        }
                                    }
                                }
                            ]
                        },
                        {
                            "name":"DeleteAFile",
                            "type":"Delete",
                            "dependsOn":[
                                {
                                    "activity":"CopyAFile",
                                    "dependencyConditions":[
                                        "Succeeded"
                                    ]
                                }
                            ],
                            "policy":{
                                "timeout":"7.00:00:00",
                                "retry":0,
                                "retryIntervalInSeconds":30,
                                "secureOutput":false,
                                "secureInput":false
                            },
                            "userProperties":[

                            ],
                            "typeProperties":{
                                "dataset":{
                                    "referenceName":"OneSourceFile",
                                    "type":"DatasetReference",
                                    "parameters":{
                                        "Container":{
                                            "value":"@pipeline().parameters.SourceStore_Location",
                                            "type":"Expression"
                                        },
                                        "Directory":{
                                            "value":"@pipeline().parameters.SourceStore_Directory",
                                            "type":"Expression"
                                        },
                                        "filename":{
                                            "value":"@item().name",
                                            "type":"Expression"
                                        }
                                    }
                                },
                                "logStorageSettings":{
                                    "linkedServiceName":{
                                        "referenceName":"BloblinkedService",
                                        "type":"LinkedServiceReference"
                                    },
                                    "path":"container/log"
                                },
                                "enableLogging":true,
                                "storeSettings":{
                                    "type":"AzureBlobStorageReadSettings",
                                    "recursive":true
                                }
                            }
                        }
                    ]
                }
            }
        ],
        "parameters":{
            "SourceStore_Location":{
                "type":"String"
            },
            "SourceStore_Directory":{
                "type":"String"
            },
            "DestinationStore_Location":{
                "type":"String"
            },
            "DestinationStore_Directory":{
                "type":"String"
            }
        },
        "annotations":[

        ]
    }
}
```

#### <a name="sample-datasets"></a>Типовые наборы данных

Набор данных, используемый действием GetMetadata для перечисления списка файлов.

```json
{
    "name":"OneSourceFolder",
    "properties":{
        "linkedServiceName":{
            "referenceName":"AzureStorageLinkedService",
            "type":"LinkedServiceReference"
        },
        "parameters":{
            "Container":{
                "type":"String"
            },
            "Directory":{
                "type":"String"
            }
        },
        "annotations":[

        ],
        "type":"Binary",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "folderPath":{
                    "value":"@{dataset().Directory}",
                    "type":"Expression"
                },
                "container":{
                    "value":"@{dataset().Container}",
                    "type":"Expression"
                }
            }
        }
    }
}
```

Набор данных для источника данных, используемый действием копирования и действием Delete.

```json
{
    "name":"OneSourceFile",
    "properties":{
        "linkedServiceName":{
            "referenceName":"AzureStorageLinkedService",
            "type":"LinkedServiceReference"
        },
        "parameters":{
            "Container":{
                "type":"String"
            },
            "Directory":{
                "type":"String"
            },
            "filename":{
                "type":"string"
            }
        },
        "annotations":[

        ],
        "type":"Binary",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "fileName":{
                    "value":"@dataset().filename",
                    "type":"Expression"
                },
                "folderPath":{
                    "value":"@{dataset().Directory}",
                    "type":"Expression"
                },
                "container":{
                    "value":"@{dataset().Container}",
                    "type":"Expression"
                }
            }
        }
    }
}
```

Набор данных для назначения данных, используемый действием копирования.

```json
{
    "name":"OneDestinationFile",
    "properties":{
        "linkedServiceName":{
            "referenceName":"AzureStorageLinkedService",
            "type":"LinkedServiceReference"
        },
        "parameters":{
            "Container":{
                "type":"String"
            },
            "Directory":{
                "type":"String"
            },
            "filename":{
                "type":"string"
            }
        },
        "annotations":[

        ],
        "type":"Binary",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "fileName":{
                    "value":"@dataset().filename",
                    "type":"Expression"
                },
                "folderPath":{
                    "value":"@{dataset().Directory}",
                    "type":"Expression"
                },
                "container":{
                    "value":"@{dataset().Container}",
                    "type":"Expression"
                }
            }
        }
    }
}
```

Кроме того, можно получить шаблон для перемещения файлов [отсюда](solution-template-move-files.md).

## <a name="known-limitation"></a>Известные ограничения

-   Действие удаления не поддерживает удаление списка папок, описанных с помощью подстановочных знаков.

-   При использовании фильтра атрибутов файлов в действии Delete: Модифиеддатетиместарт и Модифиеддатетиминд чтобы выбрать файлы для удаления, обязательно установите "Вилдкардфиленаме": "*" в действии Delete.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о перемещении файлов в фабрике данных Azure.

-   [Инструмент копирования данных в фабрике данных Azure](copy-data-tool.md)
