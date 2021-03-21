---
title: Преобразование данных
description: Преобразуйте данные или обработайте данные в фабрике данных Azure с помощью Hadoop, Машинное обучение Azure Studio (классическая модель) или Azure Data Lake Analytics.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 07/31/2018
ms.openlocfilehash: 0a1eb593e9f9f15f88aefb2fe06706153a4b74a4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100361406"
---
# <a name="transform-data-in-azure-data-factory"></a>Преобразование данных в фабрике данных Azure

> [!div class="op_single_selector"]
> * [Поток данных для сопоставления](data-flow-create.md)
> * [Hive](transform-data-using-hadoop-hive.md)  
> * [Pig](transform-data-using-hadoop-pig.md)  
> * [MapReduce](transform-data-using-hadoop-map-reduce.md)  
> * [Потоковая передача HDInsight](transform-data-using-hadoop-streaming.md)
> * [HDInsight Spark](transform-data-using-spark.md)
> * [Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md) 
> * [Хранимая процедура](transform-data-using-stored-procedure.md)
> * [Аналитика озера данных U-SQL](transform-data-using-data-lake-analytics.md)
> * [Записная книжка кирпичей](transform-data-databricks-notebook.md)
> * [Databricks Jar](transform-data-databricks-jar.md)
> * [Databricks Python](transform-data-databricks-python.md)
> * [Пользовательские действия .NET](transform-data-using-dotnet-custom-activity.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="overview"></a>Обзор
В этой статье описываются действия по преобразованию данных в фабрике данных Azure, которые можно использовать для преобразования и обработки необработанных данных в прогнозы и аналитические сведения в масштабе. Действие преобразования выполняется в вычислительной среде, например Azure Databricks или Azure HDInsight. Статья содержит ссылки на статьи с подробными сведениями о каждом действии преобразования.

Фабрика данных поддерживает указанные ниже действия преобразования, которые вы можете добавлять в [конвейеры](concepts-pipelines-activities.md) как по отдельности, так и в цепочке с другим действием.

## <a name="transform-natively-in-azure-data-factory-with-data-flows"></a>Встроенное преобразование в фабрике данных Azure с помощью потоков данных

### <a name="mapping-data-flows"></a>Сопоставление потоков данных

Сопоставление потоков данных — это визуально спроектированные преобразования данных в фабрике данных Azure. Потоки данных позволяют инженерам данных разрабатывать логику преобразования графических данных без написания кода. Результирующие потоки данных выполняются в виде действий в конвейерах фабрики данных Azure, использующих масштабируемые кластеры Spark. Действия потока данных могут быть реализованы с помощью существующих возможностей планирования фабрики данных, управления, потоков и мониторинга. Дополнительные сведения см. в разделе [сопоставление потоков данных](concepts-data-flow-overview.md).

### <a name="data-wrangling"></a>Структурирование данных

Power Query в фабрике данных Azure обеспечивает облачное масштабирование данных структурирование, что позволяет итеративно выполнять подготовку данных без кода в масштабе облака. Структурирование данных интегрируется с [Power Query Online](/power-query/) и делает функции Power Query M доступными для структурирование данных в облачном масштабировании с помощью выполнения Spark. Дополнительные сведения см. [в статье Data структурирование in ADF](wrangling-overview.md).

## <a name="external-transformations"></a>Внешние преобразования

При необходимости можно вручную преобразовать код и управлять внешней средой вычислений самостоятельно.

### <a name="hdinsight-hive-activity"></a>Действие Hive HDInsight
Действие Hive HDInsight в конвейере фабрики данных выполняет запросы Hive к вашему собственному кластеру HDInsight или кластеру HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье [Преобразование данных с помощью действия Hadoop Hive в фабрике данных Azure](transform-data-using-hadoop-hive.md). 

### <a name="hdinsight-pig-activity"></a>Действие Pig HDInsight
Действие Pig HDInsight в конвейере фабрики данных выполняет запросы Pig к вашему собственному кластеру HDInsight или кластеру HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье [Преобразование данных с помощью действия Hadoop Pig в фабрике данных Azure](transform-data-using-hadoop-pig.md). 

### <a name="hdinsight-mapreduce-activity"></a>Действие MapReduce HDInsight
Действие MapReduce HDInsight в конвейере фабрики данных выполняет программы MapReduce для вашего собственного кластера HDInsight или кластера HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в статье [Преобразование данных с помощью действия MapReduce в фабрике данных Azure](transform-data-using-hadoop-map-reduce.md).

### <a name="hdinsight-streaming-activity"></a>Действие потоковой передачи HDInsight
Действие потоковой передачи HDInsight в конвейере фабрики данных выполняет программы потоковой передачи Hadoop для вашего собственного кластера HDInsight или кластера HDInsight по запросу под управлением Windows или Linux. Дополнительные сведения об этом действии см. в разделе [Потоковая активность Hadoop](transform-data-using-hadoop-streaming.md).

### <a name="hdinsight-spark-activity"></a>Действие HDInsight Spark
Действие HDInsight Spark в конвейере фабрики данных выполняет программы Spark в вашем кластере HDInsight. Дополнительные сведения см. в разделе [Вызов программ Spark из фабрики данных](transform-data-using-spark.md). 

### <a name="azure-machine-learning-studio-classic-activities"></a>Действия Машинное обучение Azure Studio (классические)
Фабрика данных Azure позволяет легко создавать конвейеры, использующие опубликованную веб-службу Машинное обучение Azure Studio (классическая модель) для прогнозной аналитики. С помощью [действия выполнения пакета](transform-data-using-machine-learning.md) в конвейере фабрики данных Azure можно вызвать веб-службу Studio (классическая) для выполнения прогнозов по данным в пакетной службе.

Со временем прогнозные модели в экспериментах по оценке (классическая модель) необходимо переучить с помощью новых входных наборов данных. После завершения повторного обучения необходимо обновить веб-службу оценки с помощью переученной модели машинного обучения. Чтобы обновить веб-службу с помощью заново обученной модели, можно использовать [действие обновления ресурса](update-machine-learning-models.md).  

Дополнительные сведения об этих действиях Studio (классическая модель) см. в статье [Использование действий машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md) . 

### <a name="stored-procedure-activity"></a>Действие хранимой процедуры
Вы можете использовать действие SQL Server хранимой процедуры в конвейере фабрики данных для вызова хранимой процедуры в одном из следующих хранилищ данных: база данных SQL Azure, Azure синапсе Analytics, SQL Server базу данных на предприятии или на виртуальной машине Azure. Дополнительные сведения см. в статье [Преобразование данных с помощью действия хранимой процедуры SQL Server в фабрике данных Azure](transform-data-using-stored-procedure.md).  

### <a name="data-lake-analytics-u-sql-activity"></a>Действие U-SQL Data Lake Analytics
Действие U-SQL Data Lake Analytics запускает скрипт U-SQL для кластера Azure Data Lake Analytics. Дополнительные сведения см. в статье [Преобразование данных с помощью сценариев U-SQL в Azure Data Lake Analytics](transform-data-using-data-lake-analytics.md). 

### <a name="databricks-notebook-activity"></a>Действие Notebook в Databricks

Действие Azure Databricks записной книжке в конвейере фабрики данных запускает записную книжку "кирпичы" в рабочей области Azure Databricks. Azure Databricks — это управляемая платформа для запуска Apache Spark. См. раздел [Преобразование данных с помощью записной книжки Databricks](transform-data-databricks-notebook.md).

### <a name="databricks-jar-activity"></a>Действие JAR в Databricks

Действие Jar в Azure Databricks в конвейере Фабрики данных позволяет запустить файл Spark Jar в кластере Azure Databricks. Azure Databricks — это управляемая платформа для запуска Apache Spark. См. раздел [Преобразование данных с помощью выполнения действий Jar в Azure Databricks](transform-data-databricks-jar.md).

### <a name="databricks-python-activity"></a>Действие Python в Databricks

Действие Python в Azure Databricks в конвейере Фабрики данных позволяет запустить файл Python в кластере Azure Databricks. Azure Databricks — это управляемая платформа для запуска Apache Spark. См. раздел [Преобразование данных с помощью выполнения действий Python в Azure Databricks](transform-data-databricks-python.md).

### <a name="custom-activity"></a>Настраиваемое действие
Если вам нужно преобразовать данные способом, который не поддерживается фабрикой данных Azure, то можно создать настраиваемое действие с собственной логикой обработки данных и использовать это действие в конвейере. Можно настроить запуск настраиваемого действия .NET с помощью пакетной службы Azure или кластера HDInsight. Дополнительные сведения см. в разделе [Использование настраиваемых действий в конвейере фабрики данных Azure](transform-data-using-dotnet-custom-activity.md). 

Можно создать настраиваемое действие для выполнения сценариев R в кластере HDInsight, где установлена среда R. Ознакомьтесь с примером в репозитории GitHub [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample) (Запуск сценария R с помощью фабрики данных Azure). 

### <a name="compute-environments"></a>Вычислительные среды
Вы создаете связанную службу для среды вычислений, а затем используете эту службу при определении действия преобразования. Фабрика данных поддерживает вычислительные среды двух типов. 

- **По требованию**: в этом случае вычислительная среда полностью управляется фабрикой данных. Среда автоматически создается службой фабрики данных перед отправкой задания для обработки данных и удаляется после его выполнения. Вы можете настраивать и изменять для вычислительной среды "по требованию" детализированные параметры выполнения задания, управления кластером и действий начальной загрузки. 
- **Собственная**— в этом случае вы регистрируете собственную вычислительную среду (например, кластер HDInsight) и используете ее в качестве связанной службы в фабрике данных. Вы будете управлять средой вычислений, а служба фабрики данных — использовать ее для выполнения действий. 

В статье [Связанные службы вычислений](compute-linked-services.md) описывается, какие службы вычислений поддерживает фабрика данных. 

## <a name="next-steps"></a>Дальнейшие действия
Пример использования действия преобразования см. в руководстве [Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-powershell.md).
