---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 19927fd274cbb7337248fb2ae8cf9d882612d570
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92369252"
---
Фабрика данных Azure поддерживает указанные ниже действия преобразования, которые вы можете добавлять в конвейеры как по отдельности, так и в связи с другим действием.

| Действия по преобразованию данных | Вычислительная среда |
|:--- |:--- |
| [Hive](../articles/data-factory/v1/data-factory-hive-activity.md) |HDInsight [Hadoop] |
| [Pig](../articles/data-factory/v1/data-factory-pig-activity.md) |HDInsight [Hadoop] |
| [MapReduce](../articles/data-factory/v1/data-factory-map-reduce.md) |HDInsight [Hadoop] |
| [Потоковая передача Hadoop](../articles/data-factory/v1/data-factory-hadoop-streaming-activity.md) |HDInsight [Hadoop] |
| [Spark](../articles/data-factory/v1/data-factory-spark.md) | HDInsight [Hadoop] |
| [Действия Машинного обучения Azure Studio (классическая модель): машинного обучения Azure и фабрики данных Azure](../articles/data-factory/v1/data-factory-azure-ml-batch-execution-activity.md) |Azure |
| [Хранимая процедура](../articles/data-factory/v1/data-factory-stored-proc-activity.md) |Azure SQL, Azure Synapse Analytics или SQL Server |
| [Аналитика озера данных U-SQL](../articles/data-factory/v1/data-factory-usql-activity.md) |Аналитика озера данных Azure |
| [DotNet](../articles/data-factory/v1/data-factory-use-custom-activities.md) |HDInsight [Hadoop] или пакетная служба Azure |

> [!NOTE]
> Действие MapReduce можно использовать для запуска программ Spark в кластере HDInsight Spark. Дополнительные сведения см. в разделе [Вызов программ Spark из фабрики данных](../articles/data-factory/v1/data-factory-spark.md).
> Можно создать настраиваемое действие для выполнения сценариев R в кластере HDInsight, где установлена среда R. Ознакомьтесь с примером в репозитории GitHub [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample) (Запуск сценария R с помощью фабрики данных Azure).
> 
> 

