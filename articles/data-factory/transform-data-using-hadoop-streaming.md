---
title: Преобразование данных с помощью действия потоковой передачи Hadoop
description: Эта статья содержит сведения о том, как с помощью действия потоковой передачи Hadoop в фабрике данных Azure преобразовывать данные, выполняя программы потоковой передачи Hadoop в кластере Hadoop.
author: nabhishek
ms.author: abnarain
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/08/2020
ms.openlocfilehash: e2a9bc9d664ba15da3cdefa5cf28519ab703d6ce
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100361441"
---
# <a name="transform-data-using-hadoop-streaming-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия потоковой передачи Hadoop в фабрике данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-hadoop-streaming-activity.md)
> * [Текущая версия](transform-data-using-hadoop-streaming.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие потоковой передачи HDInsight в [конвейере](concepts-pipelines-activities.md) фабрики данных выполняет программы потоковой передачи Hadoop для [вашего](compute-linked-services.md#azure-hdinsight-linked-service) кластера HDInsight или кластера HDInsight [по запросу](compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования.

Если вы не знакомы с фабрикой данных Azure, сначала ознакомьтесь со статьей [Введение в фабрику данных Azure](introduction.md) и руководством [Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-powershell.md). 

## <a name="json-sample"></a>Образец JSON
```json
{
    "name": "Streaming Activity",
    "description": "Description",
    "type": "HDInsightStreaming",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mapper": "MyMapper.exe",
        "reducer": "MyReducer.exe",
        "combiner": "MyCombiner.exe",
        "fileLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "filePaths": [
            "<containername>/example/apps/MyMapper.exe",
            "<containername>/example/apps/MyReducer.exe",
            "<containername>/example/apps/MyCombiner.exe"
        ],
        "input": "wasb://<containername>@<accountname>.blob.core.windows.net/example/input/MapperInput.txt",
        "output": "wasb://<containername>@<accountname>.blob.core.windows.net/example/output/ReducerOutput.txt",
        "commandEnvironment": [
            "CmdEnvVarName=CmdEnvVarValue"
        ],
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
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
| type              | Для действия потоковой передачи Hadoop используется тип действия HDInsightStreaming. | Да      |
| linkedServiceName | Ссылка на кластер HDInsight, зарегистрированный в качестве связанной службы в фабрике данных. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| mapper            | Указывает имя исполняемого файла средства сопоставления. | Да      |
| reducer           | Указывает имя исполняемого файла средства приведения. | Да      |
| combiner          | Указывает имя исполняемого файла средства объединения. | Нет       |
| fileLinkedService | Ссылки на связанные службы хранилища Azure, используемые для хранения программ средств сопоставления, объединения и приведения, которые следует выполнить. Здесь поддерживаются только связанные службы **[Хранилище BLOB-объектов Azure](./connector-azure-blob-storage.md)** и **[ADLS 2-го поколения](./connector-azure-data-lake-storage.md)** . Если не указать эту связанную службу, будет использоваться связанная служба хранилища Azure, определенная в связанной службе HDInsight. | Нет       |
| filePath          | Предоставляет массив путей к программам средств сопоставления, объединения и приведения, хранящийся в службе хранилища Azure, на которую ссылается свойство fileLinkedService. Путь учитывает регистр. | Да      |
| input             | Указывает путь WASB к входному файлу для средства сопоставления. | Да      |
| output            | Указывает путь WASB к выходному файлу для средства приведения. | Да      |
| getDebugInfo      | Указывает, когда файлы журнала копируются в службу хранилища Azure, используемую кластером HDInsight или определенную scriptLinkedService. Допустимые значения: None (никогда), Always (всегда) или Failure (в случае сбоя). Значение по умолчанию: Нет. | Нет       |
| аргументы         | Указывает массив аргументов для задания Hadoop. Аргументы передаются в качестве аргументов командной строки в каждую задачу. | Нет       |
| defines           | Параметры в виде пары "ключ — значение", ссылки на которые указываются в скрипте Hive. | Нет       | 

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами: 

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)