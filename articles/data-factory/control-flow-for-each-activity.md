---
title: Действие ForEach в фабрике данных Azure
description: Действие ForEach определяет повторяющийся поток управления в конвейере. Оно используется для выполнения итерации коллекции и выполнения заданных действий.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/23/2019
ms.openlocfilehash: aeabd74117f99c7cac9bde0eda02b9627caf0804
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102177796"
---
# <a name="foreach-activity-in-azure-data-factory"></a>Действие ForEach в фабрике данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие ForEach определяет повторяющийся поток управления в конвейере. Это действие используется для выполнения итерации коллекции и выполняет указанные в цикле действия. Реализация цикла этого действия аналогична структуре цикла Foreach на языках программирования.

## <a name="syntax"></a>Синтаксис
Свойства описаны далее в этой статье. Свойство элементов является коллекцией, а каждый элемент в коллекции определяется с помощью `@item()`, как показано в следующем синтаксисе:  

```json
{  
   "name":"MyForEachActivityName",
   "type":"ForEach",
   "typeProperties":{  
      "isSequential":"true",
        "items": {
            "value": "@pipeline().parameters.mySinkDatasetFolderPathCollection",
            "type": "Expression"
        },
      "activities":[  
         {  
            "name":"MyCopyActivity",
            "type":"Copy",
            "typeProperties":{  
               ...
            },
            "inputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@pipeline().parameters.mySourceDatasetFolderPath"
                  }
               }
            ],
            "outputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@item()"
                  }
               }
            ]
         }
      ]
   }
}

```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
name | Имя действия ForEach. | Строка | Да
type | Должно быть задано значение **ForEach**. | Строка | Да
isSequential | Определяет, как следует выполнять цикл: последовательно или параллельно.  В параллельном режиме может выполняться одновременно не более 20 итераций цикла. Например, если действие ForEach выполняет итерацию действия копирования с 10 разными наборами данных источника и приемника и **isSequential** имеет значение false, все копии будут выполняться одновременно. Значение по умолчанию — False. <br/><br/> Если isSequential имеет значение false, убедитесь в наличии правильной конфигурации для запуска нескольких исполняемых файлов. В противном случае это свойство следует использовать с осторожностью, чтобы исключить конфликты записи. Дополнительные сведения см. в разделе [Параллельное выполнение](#parallel-execution). | Логическое | Нет. Значение по умолчанию — False.
batchCount | Число пакетов, которое должно использоваться для управления количеством параллельного выполнения (в случае, если isSequential имеет значение false). Это верхний предел параллелизма, но действие for-each не всегда будет выполняться по этому номеру | Целое число (максимум 50) | Нет. Значение по умолчанию — 20.
Элементы | Выражение, возвращающее массив JSON для итерации. | Выражение (возвращающее массив JSON) | Да
Действия | Действия для выполнения. | Список действий | Да

## <a name="parallel-execution"></a>Параллельное выполнение
Если **isSequential** имеет значение false, итерация действия выполняется параллельно (с максимум 20 параллельными итерациями). Этот параметр следует использовать с осторожностью. Этот метод подходит, если параллельные итерации выполняют запись в ту же папку, но в разные файлы. Если параллельные итерации выполняют запись в тот же файл, такой подход обычно вызывает ошибку. 

## <a name="iteration-expression-language"></a>Язык выражений итерации
В действии ForEach предоставьте массив для итерации с помощью свойства **items**. Используйте `@item()` для итерации одного перечисления действия ForEach. Например, если **items** представляет собой массив: [1, 2, 3], `@item()` возвращает 1 в первой итерации, 2 во второй итерации и 3 в третьей итерации.

## <a name="iterating-over-a-single-activity"></a>Итерация отдельного действия
**Сценарий:** копирование из того же исходного файла в большом двоичном объекте Azure в несколько конечных файлов в большом двоичном объекте Azure.

### <a name="pipeline-definition"></a>Определение конвейера

```json
{
    "name": "<MyForEachPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyForEachActivity>",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "true",
                    "items": {
                        "value": "@pipeline().parameters.mySinkDatasetFolderPath",
                        "type": "Expression"
                    },
                    "activities": [
                        {
                            "name": "MyCopyActivity",
                            "type": "Copy",
                            "typeProperties": {
                                "source": {
                                    "type": "BlobSource",
                                    "recursive": "false"
                                },
                                "sink": {
                                    "type": "BlobSink",
                                    "copyBehavior": "PreserveHierarchy"
                                }
                            },
                            "inputs": [
                                {
                                    "referenceName": "<MyDataset>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@pipeline().parameters.mySourceDatasetFolderPath"
                                    }
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "MyDataset",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@item()"
                                    }
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "mySourceDatasetFolderPath": {
                "type": "String"
            },
            "mySinkDatasetFolderPath": {
                "type": "String"
            }
        }
    }
}

```

### <a name="blob-dataset-definition"></a>Определение набора данных большого двоичного объекта

```json
{  
   "name":"<MyDataset>",
   "properties":{  
      "type":"AzureBlob",
      "typeProperties":{  
         "folderPath":{  
            "value":"@dataset().MyFolderPath",
            "type":"Expression"
         }
      },
      "linkedServiceName":{  
         "referenceName":"StorageLinkedService",
         "type":"LinkedServiceReference"
      },
      "parameters":{  
         "MyFolderPath":{  
            "type":"String"
         }
      }
   }
}

```

### <a name="run-parameter-values"></a>Значения параметров запуска

```json
{
    "mySourceDatasetFolderPath": "input/",
    "mySinkDatasetFolderPath": [ "outputs/file1", "outputs/file2" ]
}

```

## <a name="iterate-over-multiple-activities"></a>Итерация нескольких действий
Возможно выполнять итерацию нескольких действий (например, копирование и веб-действие) в действии ForEach. В этом сценарии рекомендуется абстрагировать несколько действий в отдельном конвейере. Затем с помощью действия [ExecutePipeline](control-flow-execute-pipeline-activity.md) в конвейере с действием ForEach вызовите отдельный конвейер с несколькими действиями. 


### <a name="syntax"></a>Синтаксис

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "<MyForEachMultipleActivities>"
        "typeProperties": {
          "isSequential": true,
          "items": {
            ...
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "name": "<MyInnerPipeline>"
              "typeProperties": {
                "pipeline": {
                  "referenceName": "<copyHttpPipeline>",
                  "type": "PipelineReference"
                },
                "parameters": {
                  ...
                },
                "waitOnCompletion": true
              }
            }
          ]
        }
      }
    ],
    "parameters": {
      ...
    }
  }
}

```

### <a name="example"></a>Пример
**Сценарий:** итерация по внутреннему конвейеру в действии ForEach с действием выполнения конвейера. Внутренний конвейер копируется с параметризованными определениями схемы.

#### <a name="master-pipeline-definition"></a>Определение главного конвейера

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "MyForEachActivity",
        "typeProperties": {
          "isSequential": true,
          "items": {
            "value": "@pipeline().parameters.inputtables",
            "type": "Expression"
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "typeProperties": {
                "pipeline": {
                  "referenceName": "InnerCopyPipeline",
                  "type": "PipelineReference"
                },
                "parameters": {
                  "sourceTableName": {
                    "value": "@item().SourceTable",
                    "type": "Expression"
                  },
                  "sourceTableStructure": {
                    "value": "@item().SourceTableStructure",
                    "type": "Expression"
                  },
                  "sinkTableName": {
                    "value": "@item().DestTable",
                    "type": "Expression"
                  },
                  "sinkTableStructure": {
                    "value": "@item().DestTableStructure",
                    "type": "Expression"
                  }
                },
                "waitOnCompletion": true
              },
              "name": "ExecuteCopyPipeline"
            }
          ]
        }
      }
    ],
    "parameters": {
      "inputtables": {
        "type": "Array"
      }
    }
  }
}

```

#### <a name="inner-pipeline-definition"></a>Определение внутреннего конвейера

```json
{
  "name": "InnerCopyPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            }
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "name": "CopyActivity",
        "inputs": [
          {
            "referenceName": "sqlSourceDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sourceTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sourceTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sqlSinkDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sinkTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sinkTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceTableName": {
        "type": "String"
      },
      "sourceTableStructure": {
        "type": "String"
      },
      "sinkTableName": {
        "type": "String"
      },
      "sinkTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="source-dataset-definition"></a>Определение исходного набора данных

```json
{
  "name": "sqlSourceDataset",
  "properties": {
    "type": "SqlServerTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "sqlserverLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="sink-dataset-definition"></a>Определение набора данных приемника

```json
{
  "name": "sqlSinkDataSet",
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "azureSqlLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="master-pipeline-parameters"></a>Параметры главного конвейера
```json
{
    "inputtables": [
        {
            "SourceTable": "department",
            "SourceTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ],
            "DestTable": "department2",
            "DestTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ]
        }
    ]
    
}
```

## <a name="aggregating-outputs"></a>Агрегирование данных вывода

Чтобы объединить выходные данные действия __foreach__ , используйте _переменные_ и действие " _Добавить переменную_ ".

Сначала объявите в конвейере  _переменную_`array`. После этого вызывайте действие _Добавления переменной_ внутри каждого цикла __foreach__. Впоследствии вы можете получить агрегат из вашего массива.

## <a name="limitations-and-workarounds"></a>Ограничения и обходные решения

Ниже описаны некоторые ограничения для действия ForEach и рекомендуемые обходные решения.

| Ограничение | Обходной путь |
|---|---|
| Не вкладывайте цикл ForEach в другой цикл ForEach (или в цикл Until). | Разработайте двухуровневый конвейер, где внешний конвейер с внешним циклом ForEach выполняет итерацию по внутреннему конвейеру с вложенным циклом. |
| Количество операций параллельной обработки `batchCount` для действия ForEach не может превышать 50, максимальное число элементов — 100 000. | Разработайте двухуровневый конвейер, где внешний конвейер с действием ForEach выполняет итерацию по внутреннему конвейеру. |
| SetVariable нельзя использовать внутри действия ForEach, которое выполняется параллельно, так как переменные являются глобальными для всего конвейера, они не входят в область ForEach или любое другое действие. | Рассмотрите возможность использования последовательного ForEach или Execute конвейер внутри ForEach (переменная или параметр, обрабатываемые в дочернем конвейере).|
| | |

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с другими действиями потока управления, которые поддерживаются фабрикой данных: 

- [Действие выполнения конвейера](control-flow-execute-pipeline-activity.md)
- [Действие получения метаданных](control-flow-get-metadata-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Веб-действие](control-flow-web-activity.md)
