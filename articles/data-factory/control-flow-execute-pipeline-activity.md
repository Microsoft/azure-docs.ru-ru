---
title: Выполнение действия конвейера в фабрике данных Azure
description: Узнайте, как использовать действие выполнения конвейера для вызова одного конвейера фабрики данных из другого.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: a24bd6988a9bc8f15723ebf931f2751a09c42e26
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104783750"
---
# <a name="execute-pipeline-activity-in-azure-data-factory"></a>Действие выполнения конвейера в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие выполнения конвейера позволяет конвейеру фабрики данных вызвать другой конвейер.



## <a name="syntax"></a>Синтаксис

```json
{
    "name": "MyPipeline",
    "properties": {
        "activities": [
            {
                "name": "ExecutePipelineActivity",
                "type": "ExecutePipeline",
                "typeProperties": {
                    "parameters": {                        
                        "mySourceDatasetFolderPath": {
                            "value": "@pipeline().parameters.mySourceDatasetFolderPath",
                            "type": "Expression"
                        }
                    },
                    "pipeline": {
                        "referenceName": "<InvokedPipelineName>",
                        "type": "PipelineReference"
                    },
                    "waitOnCompletion": true
                 }
            }
        ],
        "parameters": [
            {
                "mySourceDatasetFolderPath": {
                    "type": "String"
                }
            }
        ]
    }
}
```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
name | Имя действия выполнения конвейера. | Строка | Да
type | Должно иметь значение **ExecutePipeline**. | Строка | Да
pipeline | Ссылка на зависимый конвейер, вызывающий этот конвейер. Объект ссылки конвейера имеет два свойства: **referenceName** и **type**. Свойство referenceName указывает имя конвейера в ссылке. Для свойства type необходимо задать значение PipelineReference. | PipelineReference | Да
parameters | Параметры для передачи в вызванный конвейер | Объект JSON, сопоставляющий имена параметров со значениями аргументов | Нет
waitOnCompletion | Определяет, будет ли при выполнении действия ожидаться завершение выполнения зависимого конвейера. Значение по умолчанию — false. | Логическое | Нет

## <a name="sample"></a>Образец
В этом сценарии есть два конвейера:

- **Главный конвейер.** Этот конвейер содержит одно действие выполнения конвейера, вызывающее вызванный конвейер. Главный конвейер принимает два параметра: `masterSourceBlobContainer`, `masterSinkBlobContainer`.
- **Вызванный конвейер**. Этот конвейер содержит одно действие копирования, копирующее данные из источника больших двоичных объектов Azure в приемник больших двоичных объектов Azure. Вызванный конвейер принимает два параметра: `sourceBlobContainer`, `sinkBlobContainer`.

### <a name="master-pipeline-definition"></a>Определение главного конвейера

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ExecutePipeline",
        "typeProperties": {
          "pipeline": {
            "referenceName": "invokedPipeline",
            "type": "PipelineReference"
          },
          "parameters": {
            "sourceBlobContainer": {
              "value": "@pipeline().parameters.masterSourceBlobContainer",
              "type": "Expression"
            },
            "sinkBlobContainer": {
              "value": "@pipeline().parameters.masterSinkBlobContainer",
              "type": "Expression"
            }
          },
          "waitOnCompletion": true
        },
        "name": "MyExecutePipelineActivity"
      }
    ],
    "parameters": {
      "masterSourceBlobContainer": {
        "type": "String"
      },
      "masterSinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

### <a name="invoked-pipeline-definition"></a>Определение вызванного конвейера

```json
{
  "name": "invokedPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "name": "CopyBlobtoBlob",
        "inputs": [
          {
            "referenceName": "SourceBlobDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sinkBlobDataset",
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

**Связанная служба**

```json
{
    "name": "BlobStorageLinkedService",
    "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=*****;AccountKey=*****"
    }
  }
}
```

**Исходный набор данных**
```json
{
    "name": "SourceBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sourceBlobContainer",
        "type": "Expression"
      },
      "fileName": "salesforce.txt"
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

**Целевой набор данных**
```json
{
    "name": "sinkBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sinkBlobContainer",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

### <a name="running-the-pipeline"></a>Запуск конвейера

Чтобы запустить главный конвейер в этом примере, для параметров masterSourceBlobContainer и masterSinkBlobContainer нужно передать следующие значения: 

```json
{
  "masterSourceBlobContainer": "executetest",
  "masterSinkBlobContainer": "executesink"
}
```

Главный конвейер переадресовывает эти значения в вызванный конвейер, как показано в следующем примере: 

```json
{
    "type": "ExecutePipeline",
    "typeProperties": {
      "pipeline": {
        "referenceName": "invokedPipeline",
        "type": "PipelineReference"
      },
      "parameters": {
        "sourceBlobContainer": {
          "value": "@pipeline().parameters.masterSourceBlobContainer",
          "type": "Expression"
        },
        "sinkBlobContainer": {
          "value": "@pipeline().parameters.masterSinkBlobContainer",
          "type": "Expression"
        }
      },

      ....
}

```
## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с другими действиями потока управления, которые поддерживаются фабрикой данных: 

- [Действие For Each](control-flow-for-each-activity.md)
- [Действие получения метаданных](control-flow-get-metadata-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Веб-действие](control-flow-web-activity.md)
