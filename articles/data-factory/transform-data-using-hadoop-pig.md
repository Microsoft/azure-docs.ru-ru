---
title: Преобразование данных с помощью действия Pig для Hadoop
description: Узнайте, как с помощью действия Pig в фабрике данных Azure выполнять запросы Pig к собственному кластеру HDInsight или к кластеру HDInsight по требованию.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 05/08/2020
ms.openlocfilehash: 1d0f5ce9dfbf09023b57accb761b5dd85b0f3599
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375720"
---
# <a name="transform-data-using-hadoop-pig-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия Hadoop Pig в фабрике данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-pig-activity.md)
> * [Текущая версия](transform-data-using-hadoop-pig.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие Pig HDInsight в [конвейере](concepts-pipelines-activities.md) фабрики данных выполняет запросы Pig к [вашему собственному](compute-linked-services.md#azure-hdinsight-linked-service) кластеру HDInsight или кластеру HDInsight [по запросу](compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования.

Если вы не знакомы с фабрикой данных Azure, сначала ознакомьтесь со статьей [Введение в фабрику данных Azure](introduction.md) и руководством [Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-powershell.md). 

## <a name="syntax"></a>Синтаксис

```json
{
    "name": "Pig Activity",
    "description": "description",
    "type": "HDInsightPig",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "MyAzureStorage\\PigScripts\\MyPigSript.pig",
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

| Свойство            | Описание                              | Обязательно |
| ------------------- | ---------------------------------------- | -------- |
| name                | Имя действия.                     | Да      |
| description         | Текст, описывающий, для чего используется действие | Нет       |
| type                | Для действия Hive используется тип действия HDinsightPig. | Да      |
| linkedServiceName   | Ссылка на кластер HDInsight, зарегистрированный в качестве связанной службы в фабрике данных. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| scriptLinkedService | Ссылки на связанные службы хранилища Azure, используемые для хранения скрипта Pig, который следует выполнить. Здесь поддерживаются только связанные службы **[Хранилище BLOB-объектов Azure](./connector-azure-blob-storage.md)** и **[ADLS 2-го поколения](./connector-azure-data-lake-storage.md)** . Если не указать эту связанную службу, будет использоваться связанная служба хранилища Azure, определенная в связанной службе HDInsight. | нет       |
| scriptPath          | Укажите путь к файлу скрипта, который хранится в службе хранилища Azure, на который ссылается scriptLinkedService. В имени файла учитывается регистр знаков. | Нет       |
| getDebugInfo        | Указывает, когда файлы журнала копируются в службу хранилища Azure, используемую кластером HDInsight или определенную scriptLinkedService. Допустимые значения: None (никогда), Always (всегда) или Failure (в случае сбоя). Значение по умолчанию: Нет. | Нет       |
| аргументы           | Указывает массив аргументов для задания Hadoop. Аргументы передаются в качестве аргументов командной строки в каждую задачу. | Нет       |
| defines             | Параметры в виде пары "ключ — значение", ссылки на которые указываются в скрипте Pig. | нет       |

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами: 

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)