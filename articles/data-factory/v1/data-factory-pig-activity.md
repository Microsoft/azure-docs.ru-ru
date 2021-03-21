---
title: Преобразование данных с помощью действия Pig в фабрике данных Azure
description: Узнайте, как можно использовать действие Pig в фабрике данных Azure версии 1 для запуска скриптов Pig в собственном кластере HDInsight по требованию или в вашем компьютере.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 2f59734b5452b5a06b49583954f8851e84e8f84d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387178"
---
# <a name="transform-data-using-pig-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия Pig в фабрике данных Azure
> [!div class="op_single_selector" title1="Действия преобразования"]
> * [Действие Hive](data-factory-hive-activity.md) 
> * [Действие Pig](data-factory-pig-activity.md)
> * [Действие MapReduce](data-factory-map-reduce.md)
> * [Действие потоковой передачи Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Действие Spark](data-factory-spark.md)
> * [Действие выполнения пакета в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-batch-execution-activity.md)
> * [Действие обновления ресурса в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-update-resource-activity.md)
> * [Действие хранимой процедуры](data-factory-stored-proc-activity.md)
> * [Действие U-SQL в Data Lake Analytics](data-factory-usql-activity.md)
> * [Настраиваемое действие .NET](data-factory-use-custom-activities.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. руководство по [преобразованию данных с помощью действия Pig в службе "Фабрика данных"](../transform-data-using-hadoop-pig.md).


Действие Pig HDInsight в [конвейере](data-factory-create-pipelines.md) фабрики данных выполняет запросы Pig к [вашему собственному](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) кластеру HDInsight или кластеру HDInsight [по запросу](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) под управлением Windows или Linux. Данная статья основана на материалах статьи о [действиях преобразования данных](data-factory-data-transformation-activities.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования.

> [!NOTE] 
> Если вы не знакомы с фабрикой данных Azure, сначала ознакомьтесь со статьей [Введение в фабрику данных Azure](data-factory-introduction.md) и руководством [Создание первого конвейера для преобразования данных с помощью кластера Hadoop](data-factory-build-your-first-pipeline.md). 

## <a name="syntax"></a>Синтаксис

```JSON
{
  "name": "HiveActivitySamplePipeline",
  "properties": {
    "activities": [
      {
        "name": "Pig Activity",
        "description": "description",
        "type": "HDInsightPig",
        "inputs": [
          {
            "name": "input tables"
          }
        ],
        "outputs": [
          {
            "name": "output tables"
          }
        ],
        "linkedServiceName": "MyHDInsightLinkedService",
        "typeProperties": {
          "script": "Pig script",
          "scriptPath": "<pathtothePigscriptfileinAzureblobstorage>",
          "defines": {
            "param1": "param1Value"
          }
        },
        "scheduler": {
          "frequency": "Day",
          "interval": 1
        }
      }
    ]
  }
}
```

## <a name="syntax-details"></a>Сведения о синтаксисе

| Свойство | Описание | Обязательно |
| --- | --- | --- |
| name |Имя действия. |Да |
| description |Текст, описывающий, для чего используется действие |Нет |
| type |HDinsightPig |Да |
| Ввод данных |Входные данные, используемые действием Pig. |Нет |
| outputs |Выходные данные, создаваемые действием Pig. |Да |
| linkedServiceName |Ссылка на кластер HDInsight, зарегистрированный в качестве связанной службы в фабрике данных. |Да |
| скрипт |Указывается встроенный сценарий Pig. |нет |
| scriptPath |Путь к файлу сценария Pig в хранилище BLOB-объектов Azure. Можно использовать либо свойство script, либо свойство scriptPath, но не оба сразу. В имени файла учитывается регистр знаков. |Нет |
| defines |Параметры в виде пары "ключ — значение", ссылки на которые указываются в сценарии Pig. |Нет |

## <a name="example"></a>Пример
Рассмотрим пример с анализом игровых журналов. Предположим, вы хотите определить время, которое игроки проводят за игрой, выпущенной вашей компанией.

Ниже приведен журнал игры в формате CSV-файла. Он содержит следующие поля: ProfileID, SessionStart, Duration, SrcIPAddress и GameType.

```
1809,2014-05-04 12:04:25.3470000,14,221.117.223.75,CaptureFlag
1703,2014-05-04 06:05:06.0090000,16,12.49.178.247,KingHill
1703,2014-05-04 10:21:57.3290000,10,199.118.18.179,CaptureFlag
1809,2014-05-04 05:24:22.2100000,23,192.84.66.141,KingHill
.....
```

**Сценарий Pig** для обработки этих данных выглядит так:

```
PigSampleIn = LOAD 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/samplein/' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);

GroupProfile = Group PigSampleIn all;

PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);

Store PigSampleOut into 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/sampleoutpig/' USING PigStorage (',');
```

Чтобы выполнить его в конвейере фабрики данных, выполните следующие действия:

1. Создайте связанную службу для регистрации [собственного вычислительного кластера HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) или настройте [вычислительный кластер HDInsight по запросу](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Давайте назовем эту связанную службу **HDInsightLinkedService**.
2. Создайте [связанную службу](data-factory-azure-blob-connector.md) для настройки подключения к хранилищу BLOB-объектов Azure, в котором хранятся данные. Назовем эту связанную службу **StorageLinkedService**.
3. Создайте [наборы данных](data-factory-create-datasets.md) , указывающие на входные и выходные данные. Назовем входной набор данных **PigSampleIn**, а выходной — **PigSampleOut**.
4. Скопируйте запрос Pig в файл, настроенный хранилищем BLOB-объектов Azure на шаге 2. Если хранилище BLOB-объектов Azure, размещающее данные, отличается от хранилища, размещающего файл запроса, то создайте отдельную связанную службу хранилища Azure. Добавьте ссылку на эту связанную службу в конфигурации действия. Используйте **scriptPath** , чтобы указать путь к файлу скрипта Pig и **scriptLinkedService**. 
   
   > [!NOTE]
   > Также можно добавить сценарий Pig непосредственно в определение действия, используя свойство **script** . Однако мы не рекомендуем это делать, так как в этом случае потребуется экранировать все специальные знаки в сценарии, что может вызвать проблемы при отладке. Мы рекомендуем следовать инструкциям, описанным в шаге 4.
   >
   >
5. Создайте конвейер с действием HDInsightPig. Это действие обрабатывает входные данные, запуская сценарий Pig в кластере HDInsight.

    ```JSON
    {
      "name": "PigActivitySamplePipeline",
      "properties": {
        "activities": [
          {
            "name": "PigActivitySample",
            "type": "HDInsightPig",
            "inputs": [
              {
                "name": "PigSampleIn"
              }
            ],
            "outputs": [
              {
                "name": "PigSampleOut"
              }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
              "scriptPath": "adfwalkthrough\\scripts\\enrichlogs.pig",
              "scriptLinkedService": "StorageLinkedService"
            },
            "scheduler": {
              "frequency": "Day",
              "interval": 1
            }
          }
        ]
      }
    }
    ```
6. Разверните конвейер. Дополнительные сведения см. в разделе [Создание конвейеров](data-factory-create-pipelines.md). 
7. Отслеживайте состояние конвейера, используя функции мониторинга и управления фабрикой данных. Подробные сведения см. в статье [Мониторинг конвейеров фабрики данных и управление ими](data-factory-monitor-manage-pipelines.md).

## <a name="specifying-parameters-for-a-pig-script"></a>Указание параметров для сценария Pig
Рассмотрим следующий пример: журналы игры ежедневно добавляются в хранилище BLOB-объектов Azure и хранятся в папках, разбитых по дате и времени. Вам необходимо параметризовать сценарий Pig так, чтобы адрес входной папки передавался во время выполнения динамически, а выходной набор данных также сегментировался по дате и времени.

Чтобы использовать параметризованный сценарии Pig, выполните описанные ниже действия.

* Задайте параметры в разделе **defines**.

    ```JSON
    {
      "name": "PigActivitySamplePipeline",
      "properties": {
        "activities": [
          {
            "name": "PigActivitySample",
            "type": "HDInsightPig",
            "inputs": [
              {
                "name": "PigSampleIn"
              }
            ],
            "outputs": [
              {
                "name": "PigSampleOut"
              }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
              "scriptPath": "adfwalkthrough\\scripts\\samplepig.hql",
              "scriptLinkedService": "StorageLinkedService",
              "defines": {
                "Input": "$$Text.Format('wasb: //adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0: yyyy}/monthno={0:MM}/dayno={0: dd}/',SliceStart)",
                "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
              }
            },
            "scheduler": {
              "frequency": "Day",
              "interval": 1
            }
          }
        ]
      }
    }
    ```
* В сценарии Pig сошлитесь на параметры с помощью **$parameterName**, как показано в следующем примере.

    ```
    PigSampleIn = LOAD '$Input' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);
    GroupProfile = Group PigSampleIn all;
    PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);
    Store PigSampleOut into '$Output' USING PigStorage (','); 
    ```

## <a name="see-also"></a>См. также:
* [Действие Hive](data-factory-hive-activity.md)
* [Действие MapReduce](data-factory-map-reduce.md)
* [Действие потоковой передачи Hadoop](data-factory-hadoop-streaming-activity.md)
* [Вызов программ Spark](data-factory-spark.md)
* [Вызов сценариев R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)
