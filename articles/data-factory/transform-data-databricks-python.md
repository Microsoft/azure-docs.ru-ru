---
title: Преобразование данных с помощью кирпичей Python
description: Узнайте, как обрабатывать или преобразовывать данные, запустив действие Python в конвейере фабрики данных Azure.
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/15/2018
author: dcstwh
ms.author: weetok
ms.custom: devx-track-python
ms.openlocfilehash: 49dfe11ceb01471e3b5afadd30259dcd63e7b82a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373952"
---
# <a name="transform-data-by-running-a-python-activity-in-azure-databricks"></a>Преобразование данных с помощью выполнения действий Python в Azure Databricks
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие Python в Azure Databricks в [конвейере фабрики данных](concepts-pipelines-activities.md) позволяет запустить файл Python в кластере Azure Databricks. Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования. Azure Databricks — это управляемая платформа для запуска Apache Spark.

Уделите 11 минут вашего времени, чтобы просмотреть следующее видео с кратким обзором и демонстрацией этой функции:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Execute-Jars-and-Python-scripts-on-Azure-Databricks-using-Data-Factory/player]

## <a name="databricks-python-activity-definition"></a>Определение действия Python в Databricks

Пример определения JSON для действия Python в Databricks:

```json
{
    "activity": {
        "name": "MyActivity",
        "description": "MyActivity description",
        "type": "DatabricksSparkPython",
        "linkedServiceName": {
            "referenceName": "MyDatabricksLinkedservice",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "pythonFile": "dbfs:/docs/pi.py",
            "parameters": [
                "10"
            ],
            "libraries": [
                {
                    "pypi": {
                        "package": "tensorflow"
                    }
                }
            ]
        }
    }
}
```

## <a name="databricks-python-activity-properties"></a>Свойства действия Python в Databricks

В следующей таблице приведено описание свойств, используемых в определении JSON.

|Свойство|Описание|Обязательно|
|---|---|---|
|name|Имя действия в конвейере.|Да|
|description|Описание действия.|Нет|
|type|Тип действия Python в Databricks — DatabricksSparkPython.|Да|
|linkedServiceName|Имя связанной службы Databricks, в которой выполняется действие Python. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).|Да|
|pythonFile|URI файла Python, который будет выполнен. Поддерживаются только пути DBFS.|Да|
|parameters|Параметры командной строки, которые будут переданы в файл Python. Массив строк.|Нет|
|libraries|Список библиотек, которые должны быть установлены на кластере, на котором будет выполнено задание. Массив объектов <строка, объект>|Нет|

## <a name="supported-libraries-for-databricks-activities"></a>Поддерживаемые библиотеки для действий Databricks

В приведенном выше определении действия Databricks необходимо указать следующие типы библиотек: *jar*, *egg*, *maven*, *pypi*, *cran*.

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

Дополнительные сведения см. в [документации Databricks](/azure/databricks/dev-tools/api/latest/libraries#managedlibrarieslibrary) по типам библиотек.

## <a name="how-to-upload-a-library-in-databricks"></a>Отправка библиотеки в Databricks

### <a name="you-can-use-the-workspace-ui"></a>Вы можете использовать пользовательский интерфейс рабочей области:

1. [Использование пользовательского интерфейса рабочей области "кирпичи"](/azure/databricks/libraries/#create-a-library)

2. Чтобы получить путь dBFS библиотеки, добавленной с помощью пользовательского интерфейса, можно использовать интерфейс [командной строки для модуля](/azure/databricks/dev-tools/cli/#install-the-cli)данных.

   Обычно библиотеки Jar, добавленные с помощью пользовательского интерфейса, хранятся в каталоге dbfs:/FileStore/jars. Вы можете получить список всех библиотек, выполнив следующую команду в интерфейсе командной строки: *databricks fs ls dbfs:/FileStore/job-jars*.

### <a name="or-you-can-use-the-databricks-cli"></a>Также можно использовать интерфейс командной строки для модуля:

1. Следуйте инструкциям [по копированию библиотеки с помощью интерфейса командной строки для модуля](/azure/databricks/dev-tools/cli/#copy-a-file-to-dbfs) обработки

2. Использование интерфейса командной строки кирпичей [(этапы установки)](/azure/databricks/dev-tools/cli/#install-the-cli)

   Например, чтобы скопировать JAR-файл в dBFS: `dbfs cp SparkPi-assembly-0.1.jar dbfs:/docs/sparkpi.jar`
