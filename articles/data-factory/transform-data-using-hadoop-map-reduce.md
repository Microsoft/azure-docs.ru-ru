---
title: Преобразование данных с помощью действия MapReduce для Hadoop
description: Узнайте, как обрабатывать данные путем выполнения программ Hadoop MapReduce в кластере Azure HDInsight из фабрики данных Azure.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 05/08/2020
ms.openlocfilehash: f03906586d6226c92cfa69e1a139d4c876cbf723
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375890"
---
# <a name="transform-data-using-hadoop-mapreduce-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия Hadoop MapReduce в фабрике данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-map-reduce.md)
> * [Текущая версия](transform-data-using-hadoop-map-reduce.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие MapReduce HDInsight в [конвейере](concepts-pipelines-activities.md) Фабрики данных вызывает программу MapReduce для [вашего](compute-linked-services.md#azure-hdinsight-linked-service) кластера HDInsight или кластера HDInsight [по запросу](compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования.

Если вы не знакомы с Фабрикой данных Azure, сначала ознакомьтесь со статьей [Введение в фабрику данных Azure](introduction.md) и руководством [Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-powershell.md) перед чтением этой статьи.

Дополнительные сведения о выполнении скриптов Pig и Hive в кластере HDInsight из конвейера с помощью действий Pig и Hive в HDInsight см. в статьях о действии [Pig](transform-data-using-hadoop-pig.md) и [Hive](transform-data-using-hadoop-hive.md).

## <a name="syntax"></a>Синтаксис

```json
{
    "name": "Map Reduce Activity",
    "description": "Description",
    "type": "HDInsightMapReduce",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "className": "org.myorg.SampleClass",
        "jarLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "jarFilePath": "MyAzureStorage/jars/sample.jar",
        "getDebugInfo": "Failure",
        "arguments": [
            "-SampleHadoopJobArgument1"
        ],
        "defines": {
            "param1": "param1Value"
        }
    }
}
```

## <a name="syntax-details"></a>Сведения о синтаксисе

| Свойство          | Описание                              | Обязательно |
| ----------------- | ---------------------------------------- | -------- |
| name              | Имя действия.                     | Да      |
| description       | Текст, описывающий, для чего используется действие | Нет       |
| type              | Для действия MapReduce Activity используется тип действия HDinsightMapReduce. | Да      |
| linkedServiceName | Ссылка на кластер HDInsight, зарегистрированный в качестве связанной службы в фабрике данных. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| className         | Имя класса для выполнения.         | Да      |
| jarLinkedService  | Ссылки на связанные службы хранилища Azure, используемые для хранения файлов Jar. Здесь поддерживаются только связанные службы **[Хранилище BLOB-объектов Azure](./connector-azure-blob-storage.md)** и **[ADLS 2-го поколения](./connector-azure-data-lake-storage.md)** . Если не указать эту связанную службу, будет использоваться связанная служба хранилища Azure, определенная в связанной службе HDInsight. | Нет       |
| jarFilePath       | Укажите путь к файлам Jar, которые хранятся в службе хранилища Azure, на который ссылается jarLinkedService. В имени файла учитывается регистр знаков. | Да      |
| jarlibs           | Массив строк пути к файлам библиотеки Jar, на которые ссылается задание, хранящееся в службе хранилища Azure, которая, в свою очередь, определена в свойстве jarLinkedService. В имени файла учитывается регистр знаков. | Нет       |
| getDebugInfo      | Указывает, когда файлы журнала копируются в службу хранилища Azure, используемую кластером HDInsight или определенную jarLinkedService. Допустимые значения: None (никогда), Always (всегда) или Failure (в случае сбоя). Значение по умолчанию: Нет. | Нет       |
| аргументы         | Указывает массив аргументов для задания Hadoop. Аргументы передаются в качестве аргументов командной строки в каждую задачу. | Нет       |
| defines           | Параметры в виде пары "ключ — значение", ссылки на которые указываются в скрипте Hive. | Нет       |



## <a name="example"></a>Пример
С помощью действия MapReduce можно выполнить JAR-файл MapReduce в кластере HDInsight. В приведенном ниже образце определения конвейера JSON действие HDInsight настроено на запуск JAR-файла Mahout.

```json
{
    "name": "MapReduce Activity for Mahout",
    "description": "Custom MapReduce to generate Mahout result",
    "type": "HDInsightMapReduce",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
        "jarLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
        "arguments": [
            "-s",
            "SIMILARITY_LOGLIKELIHOOD",
            "--input",
            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input",
            "--output",
            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/",
            "--maxSimilaritiesPerItem",
            "500",
            "--tempDir",
            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"
        ]
    }
}
```
Вы можете указать необходимые аргументы для программы MapReduce в разделе **arguments**. Во время выполнения вы увидите несколько дополнительных аргументов (например, mapreduce.job.tags) платформы MapReduce. Чтобы отличать свои аргументы от аргументов MapReduce, вы можете использовать параметр и значение в качестве аргументов, как показано в следующем примере (-s, --input, --output и т. д. — параметры, за которыми сразу следуют их значения).

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами:

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)