---
title: Преобразование данных с помощью записной книжки кирпичей
description: Узнайте, как обрабатывать или преобразовывать данные, запустив в фабрике данных Azure записную книжку.
ms.service: data-factory
author: nabhishek
ms.author: abnarain
ms.topic: conceptual
ms.date: 03/15/2018
ms.openlocfilehash: fea572c2e75f62b5e7e7b4634e37da348bdcdaf1
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102183494"
---
# <a name="transform-data-by-running-a-databricks-notebook"></a>Преобразование данных с помощью записной книжки Databricks
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие Azure Databricks записной книжке в [конвейере фабрики данных](concepts-pipelines-activities.md) запускает записную книжку "кирпичы" в рабочей области Azure Databricks. Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования. Azure Databricks — это управляемая платформа для запуска Apache Spark.

## <a name="databricks-notebook-activity-definition"></a>Определение действия Databricks Notebook

Пример определения JSON для действия Databricks Notebook:

```json
{
    "activity": {
        "name": "MyActivity",
        "description": "MyActivity description",
        "type": "DatabricksNotebook",
        "linkedServiceName": {
            "referenceName": "MyDatabricksLinkedservice",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "notebookPath": "/Users/user@example.com/ScalaExampleNotebook",
            "baseParameters": {
                "inputpath": "input/folder1/",
                "outputpath": "output/"
            },
            "libraries": [
                {
                "jar": "dbfs:/docs/library.jar"
                }
            ]
        }
    }
}
```

## <a name="databricks-notebook-activity-properties"></a>Свойства действия Databricks Notebook

В следующей таблице приведено описание свойств, используемых в определении JSON.

|Свойство|Описание|Обязательно|
|---|---|---|
|name|Имя действия в конвейере.|Да|
|description|Описание действия.|Нет|
|type|Тип действия Databricks Notebook — DatabricksNotebook.|Да|
|linkedServiceName|Имя связанной службы Databricks, в которой запускается записная книжка Databricks. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).|Да|
|notebookPath|Абсолютный путь записной книжки, которая будет запущена в рабочей области Databricks. Этот путь должен начинаться с косой черты.|Да|
|baseParameters|Массив пар "ключ-значение". Для каждого выполнения действия можно использовать базовые параметры. Если записная книжка принимает параметр, который не был указан, используется значение по умолчанию из записной книжки. Дополнительные сведения о параметрах Databricks Notebook см. [здесь](https://docs.databricks.com/api/latest/jobs.html#jobsparampair).|Нет|
|libraries|Список библиотек, которые должны быть установлены на кластере, на котором будет выполнено задание. Это может быть массив \<string, object> .|Нет|

## <a name="supported-libraries-for-databricks-activities"></a>Поддерживаемые библиотеки для действий Databricks

В приведенном выше определении действия "кирпичы данных" указываются следующие типы библиотек: *JAR*, *Egg*, *WHL*, *Maven*, *PyPI*, *Cran*.

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
            "whl": "dbfs:/mnt/libraries/mlflow-0.0.1.dev0-py2-none-any.whl"
        },
        {
            "whl": "dbfs:/mnt/libraries/wheel-libraries.wheelhouse.zip"
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

## <a name="passing-parameters-between-notebooks-and-data-factory"></a>Передача параметров между записными книжками и фабрикой данных

Параметры фабрики данных можно передать в записные книжки с помощью свойства *басепараметерс* в действии модуля данных.

В некоторых случаях может потребоваться передача определенных значений из записной книжки обратно в фабрику данных, которую можно использовать для потока управления (условные проверки) в фабрике данных или для использования нисходящими действиями (ограничение размера — 2 МБ).

1. В записной книжке вы можете вызвать [дбутилс. Notebook. Exit ("ReturnValue")](/azure/databricks/notebooks/notebook-workflows#notebook-workflows-exit) , и в фабрике данных будет возвращен соответствующий результат "ReturnValue".

2. Выходные данные в фабрике данных можно использовать с помощью выражения, такого как `@{activity('databricks notebook activity name').output.runOutput}` . 

   > [!IMPORTANT]
   > При передаче объекта JSON значения можно получить, добавив имена свойств. Например, `@{activity('databricks notebook activity name').output.runOutput.PropertyName}`.

## <a name="how-to-upload-a-library-in-databricks"></a>Отправка библиотеки в Databricks

### <a name="you-can-use-the-workspace-ui"></a>Вы можете использовать пользовательский интерфейс рабочей области:

1. [Использование пользовательского интерфейса рабочей области "кирпичи"](/azure/databricks/libraries/#create-a-library)

2. Чтобы получить путь dBFS библиотеки, добавленной с помощью пользовательского интерфейса, можно использовать интерфейс [командной строки для модуля](/azure/databricks/dev-tools/cli/#install-the-cli)данных.

   Обычно библиотеки Jar, добавленные с помощью пользовательского интерфейса, хранятся в каталоге dbfs:/FileStore/jars. Вы можете получить список всех библиотек, выполнив следующую команду в интерфейсе командной строки: *databricks fs ls dbfs:/FileStore/job-jars*.

### <a name="or-you-can-use-the-databricks-cli"></a>Также можно использовать интерфейс командной строки для модуля:

1. Следуйте инструкциям [по копированию библиотеки с помощью интерфейса командной строки для модуля](/azure/databricks/dev-tools/cli/#copy-a-file-to-dbfs) обработки

2. Использование интерфейса командной строки кирпичей [(этапы установки)](/azure/databricks/dev-tools/cli/#install-the-cli)

   Например, чтобы скопировать JAR-файл в dBFS: `dbfs cp SparkPi-assembly-0.1.jar dbfs:/docs/sparkpi.jar`
