---
title: Преобразование данных с помощью JAR-файла
description: Узнайте, как обрабатывать или преобразовывать данные, запустив в конвейере фабрики данных Azure JAR-файл.
ms.service: data-factory
ms.topic: conceptual
ms.author: abnarain
author: nabhishek
ms.date: 02/10/2021
ms.openlocfilehash: ccfe8fbf330e1c7f6f415b64a1f18d93a084a0ba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100374020"
---
# <a name="transform-data-by-running-a-jar-activity-in-azure-databricks"></a>Преобразование данных с помощью выполнения действий Jar в Azure Databricks

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие Jar в Azure Databricks в [конвейере фабрики данных](concepts-pipelines-activities.md) позволяет запустить файл Spark Jar в кластере Azure Databricks. Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования. Azure Databricks — это управляемая платформа для запуска Apache Spark.

Уделите 11 минут вашего времени, чтобы просмотреть следующее видео с кратким обзором и демонстрацией этой функции:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Execute-Jars-and-Python-scripts-on-Azure-Databricks-using-Data-Factory/player]

## <a name="databricks-jar-activity-definition"></a>Определение действия Jar в Databricks

Ниже приведен пример определения JSON действия JAR для модуля данных.

```json
{
    "name": "SparkJarActivity",
    "type": "DatabricksSparkJar",
    "linkedServiceName": {
        "referenceName": "AzureDatabricks",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mainClassName": "org.apache.spark.examples.SparkPi",
        "parameters": [ "10" ],
        "libraries": [
            {
                "jar": "dbfs:/docs/sparkpi.jar"
            }
        ]
    }
}

```

## <a name="databricks-jar-activity-properties"></a>Свойства действия Jar в Databricks

В следующей таблице приведено описание свойств, используемых в определении JSON.

|Свойство|Описание|Обязательно|
|:--|---|:-:|
|name|Имя действия в конвейере.|Да|
|description|Описание действия.|Нет|
|type|Тип действия Jar в Databricks — DatabricksSparkJar.|Да|
|linkedServiceName|Имя связанной службы Databricks, в которой выполняется действие Jar. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).|Да|
|mainClassName|Полное имя класса, содержащего метод main, который будет выполнен. Этот класс должен содержаться в файле JAR, предоставляемом в виде библиотеки. JAR-файл может содержать несколько классов. Каждый из классов может содержать метод Main.|Да|
|parameters|Параметры, которые будут переданы в метод main. Это свойство представляет собой массив строк.|Нет|
|libraries|Список библиотек, которые должны быть установлены на кластере, на котором будет выполнено задание. Массив объектов <строка, объект>|Да (по крайней мере один метод, содержащий mainClassName)|

> [!NOTE]
> **Известная ошибка** . при использовании одного и того же [интерактивного кластера](compute-linked-services.md#example---using-existing-interactive-cluster-in-databricks) для выполнения операций JAR-файла с параллельными модулями данных (без перезапуска кластера) существует известная ошибка в модулях данных, где в параметрах первого действия будут использоваться следующие действия. Поэтому в последующие задания передаются неверные параметры. Для устранения этой проблемы используйте вместо этого [кластер заданий](compute-linked-services.md#example---using-new-job-cluster-in-databricks) .

## <a name="supported-libraries-for-databricks-activities"></a>Поддерживаемые библиотеки для действий Databricks

В приведенном выше определении действия "кирпичы данных" вы указали следующие типы библиотек: `jar` , `egg` , `maven` , `pypi` , `cran` .

```json
{
    "libraries": [
        {
            "jar": "dbfs:/mnt/libraries/library.jar"
        },
        {
            "egg": "dbfs:/mnt/libraries/library.egg"
        },
        {
            "maven": {
                "coordinates": "org.jsoup:jsoup:1.7.2",
                "exclusions": [ "slf4j:slf4j" ]
            }
        },
        {
            "pypi": {
                "package": "simplejson",
                "repo": "http://my-pypi-mirror.com"
            }
        },
        {
            "cran": {
                "package": "ada",
                "repo": "https://cran.us.r-project.org"
            }
        }
    ]
}

```

Дополнительные сведения см. в [документации по модулям](/azure/databricks/dev-tools/api/latest/libraries#managedlibrarieslibrary) данных для типов библиотек.

## <a name="how-to-upload-a-library-in-databricks"></a>Отправка библиотеки в Databricks

### <a name="you-can-use-the-workspace-ui"></a>Вы можете использовать пользовательский интерфейс рабочей области:

1. [Использование пользовательского интерфейса рабочей области "кирпичи"](/azure/databricks/libraries/#create-a-library)

2. Чтобы получить путь dBFS библиотеки, добавленной с помощью пользовательского интерфейса, можно использовать интерфейс [командной строки для модуля](/azure/databricks/dev-tools/cli/#install-the-cli)данных.

   Обычно библиотеки Jar, добавленные с помощью пользовательского интерфейса, хранятся в каталоге dbfs:/FileStore/jars. Вы можете получить список всех библиотек, выполнив следующую команду в интерфейсе командной строки: *databricks fs ls dbfs:/FileStore/job-jars*.

### <a name="or-you-can-use-the-databricks-cli"></a>Также можно использовать интерфейс командной строки для модуля:

1. Следуйте инструкциям [по копированию библиотеки с помощью интерфейса командной строки для модуля](/azure/databricks/dev-tools/cli/#copy-a-file-to-dbfs) обработки

2. Использование интерфейса командной строки кирпичей [(этапы установки)](/azure/databricks/dev-tools/cli/#install-the-cli)

   Например, чтобы скопировать JAR-файл в dBFS: `dbfs cp SparkPi-assembly-0.1.jar dbfs:/docs/sparkpi.jar`

## <a name="next-steps"></a>Дальнейшие действия

Чтобы ознакомиться с 11-минутным введением и демонстрацией этой функции, просмотрите [видео](https://channel9.msdn.com/Shows/Azure-Friday/Execute-Jars-and-Python-scripts-on-Azure-Databricks-using-Data-Factory/player).
