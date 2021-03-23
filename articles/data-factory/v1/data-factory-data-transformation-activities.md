---
title: 'Преобразование данных: обработка & преобразование данных '
description: Узнайте, как преобразовать данные или обработать данные в фабрике данных Azure с помощью Hadoop, Машинное обучение Azure Studio (классическая модель) или Azure Data Lake Analytics.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 422acbaab097182b800e3bad35b0121284db9cd8
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782934"
---
# <a name="transform-data-in-azure-data-factory-version-1"></a>Преобразование данных в фабрике данных Azure версии 1
> [!div class="op_single_selector"]
> * [Hive](data-factory-hive-activity.md)  
> * [Pig](data-factory-pig-activity.md)  
> * [MapReduce](data-factory-map-reduce.md)  
> * [Потоковая передача Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Машинное обучение Azure Studio (классическая модель)](data-factory-azure-ml-batch-execution-activity.md) 
> * [Хранимая процедура](data-factory-stored-proc-activity.md)
> * [Аналитика озера данных U-SQL](data-factory-usql-activity.md)
> * [Пользовательские действия .NET](data-factory-use-custom-activities.md)

## <a name="overview"></a>Обзор
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [действиях по преобразованию данных в службе "Фабрика данных"](../transform-data.md).

В этой статье объясняются действия преобразования данных в фабрике данных Azure, с помощью которых можно обрабатывать необработанные данные и преобразовывать их в прогнозы и аналитику. Действие преобразования выполняется в вычислительной среде, например в кластере Azure HDInsight или пакетной службе Azure. Статья содержит ссылки на статьи с подробными сведениями о каждом действии преобразования.

Фабрика данных поддерживает указанные ниже действия преобразования, которые вы можете добавлять в [конвейеры](data-factory-create-pipelines.md) как по отдельности, так и в цепочке с другим действием.

> [!NOTE]
> Пошаговое руководство см. в статье [Учебник. Создание первого конвейера для обработки данных с помощью кластера Hadoop](data-factory-build-your-first-pipeline.md).  
> 
> 

## <a name="hdinsight-hive-activity"></a>Действие Hive HDInsight
Действие Hive HDInsight в конвейере фабрики данных выполняет запросы Hive к вашему собственному кластеру HDInsight или кластеру HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье о [действиях Hive](data-factory-hive-activity.md) . 

## <a name="hdinsight-pig-activity"></a>Действие Pig HDInsight
Действие Pig HDInsight в конвейере фабрики данных выполняет запросы Pig к вашему собственному кластеру HDInsight или кластеру HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье о [действии Pig](data-factory-pig-activity.md) . 

## <a name="hdinsight-mapreduce-activity"></a>Действие MapReduce HDInsight
Действие MapReduce HDInsight в конвейере фабрики данных выполняет программы MapReduce для вашего собственного кластера HDInsight или кластера HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье о [действии MapReduce](data-factory-map-reduce.md) .

## <a name="hdinsight-streaming-activity"></a>Действие потоковой передачи HDInsight
Действие потоковой передачи HDInsight в конвейере фабрики данных выполняет программы потоковой передачи Hadoop для вашего собственного кластера HDInsight или кластера HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в разделе [Потоковая активность Hadoop](data-factory-hadoop-streaming-activity.md).

## <a name="hdinsight-spark-activity"></a>Действие HDInsight Spark
Действие HDInsight Spark в конвейере фабрики данных выполняет программы Spark в вашем кластере HDInsight. Дополнительные сведения см. в разделе [Вызов программ Spark из фабрики данных](data-factory-spark.md). 

## <a name="azure-machine-learning-studio-classic-activities"></a>Действия Машинное обучение Azure Studio (классические)
Фабрика данных Azure позволяет легко создавать конвейеры, использующие опубликованную веб-службу Машинное обучение Azure Studio (классическая модель) для прогнозной аналитики. С помощью [действия выполнения пакета](data-factory-azure-ml-batch-execution-activity.md#invoking-a-web-service-using-batch-execution-activity) в конвейере фабрики данных Azure можно вызвать веб-службу Studio (классическая) для выполнения прогнозов по данным в пакетной службе.

Со временем прогнозные модели в экспериментах по оценке (классическая модель) необходимо переучить с помощью новых входных наборов данных. После завершения повторного обучения необходимо обновить веб-службу оценки с помощью переученной модели машинного обучения. [Действие обновить ресурс](data-factory-azure-ml-batch-execution-activity.md#updating-models-using-update-resource-activity) можно использовать для обновления веб-службы с использованием новой обученной модели.  

Дополнительные сведения об этих действиях Studio (классическая модель) см. в статье [Использование действий машинное обучение Azure Studio (классическая модель)](data-factory-azure-ml-batch-execution-activity.md) . 

## <a name="stored-procedure-activity"></a>Действие хранимой процедуры
Вы можете использовать действие SQL Server хранимой процедуры в конвейере фабрики данных для вызова хранимой процедуры в одном из следующих хранилищ данных: база данных SQL Azure, Azure синапсе Analytics, SQL Server базу данных на предприятии или на виртуальной машине Azure. Дополнительные сведения см. в статье о [действии хранимой процедуры](data-factory-stored-proc-activity.md) .  

## <a name="data-lake-analytics-u-sql-activity"></a>Действие U-SQL Data Lake Analytics
Действие U-SQL Data Lake Analytics запускает сценарий U-SQL для кластера Azure Data Lake Analytics. Дополнительные сведения см. в статье о [действиях U-SQL в аналитике данных](data-factory-usql-activity.md) . 

## <a name="net-custom-activity"></a>Настраиваемое действие .NET
Если вам нужно преобразовать данные способом, который не поддерживается фабрикой данных Azure, то можно создать настраиваемое действие с собственной логикой обработки данных и использовать это действие в конвейере. Можно настроить запуск настраиваемого действия .NET с помощью пакетной службы Azure или кластера HDInsight. Дополнительные сведения см. в разделе [Использование настраиваемых действий в конвейере фабрики данных Azure](data-factory-use-custom-activities.md). 

Можно создать настраиваемое действие для выполнения сценариев R в кластере HDInsight, где установлена среда R. Ознакомьтесь с примером в репозитории GitHub [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample) (Запуск сценария R с помощью фабрики данных Azure). 

## <a name="compute-environments"></a>Вычислительные среды
Вы создаете связанную службу для среды вычислений, а затем используете эту службу при определении действия преобразования. Фабрика данных поддерживает вычислительные среды двух типов. 

1. **По требованию**: в этом случае вычислительная среда полностью управляется фабрикой данных. Среда автоматически создается службой фабрики данных перед отправкой задания для обработки данных и удаляется после его выполнения. Вы можете настраивать и изменять для вычислительной среды "по требованию" детализированные параметры выполнения задания, управления кластером и действий начальной загрузки. 
2. **Собственная**— в этом случае вы регистрируете собственную вычислительную среду (например, кластер HDInsight) и используете ее в качестве связанной службы в фабрике данных. Вы будете управлять средой вычислений, а служба фабрики данных — использовать ее для выполнения действий. 

В статье [Связанные службы вычислений](data-factory-compute-linked-services.md) описывается, какие службы вычислений поддерживает фабрика данных. 

## <a name="summary"></a>Сводка
Фабрика данных Azure поддерживает приведенные ниже действия преобразования данных и вычислительных сред для них. Действия преобразования можно добавлять в конвейеры как по отдельности, так и в цепочке с другим действием.

| Действия по преобразованию данных | Вычислительная среда |
|:--- |:--- |
| [Hive](data-factory-hive-activity.md) |HDInsight [Hadoop] |
| [Pig](data-factory-pig-activity.md) |HDInsight [Hadoop] |
| [MapReduce](data-factory-map-reduce.md) |HDInsight [Hadoop] |
| [Потоковая передача Hadoop](data-factory-hadoop-streaming-activity.md) |HDInsight [Hadoop] |
| [Действия Машинного обучения Azure Studio (классическая модель): машинного обучения Azure и фабрики данных Azure](data-factory-azure-ml-batch-execution-activity.md) |Azure |
| [Хранимая процедура](data-factory-stored-proc-activity.md) |Azure SQL, Azure Synapse Analytics или SQL Server |
| [Аналитика озера данных U-SQL](data-factory-usql-activity.md) |Аналитика озера данных Azure |
| [DotNet](data-factory-use-custom-activities.md) |HDInsight [Hadoop] или пакетная служба Azure |

