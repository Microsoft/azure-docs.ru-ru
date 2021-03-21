---
title: Преобразование данных с помощью действия Spark
description: Сведения о преобразовании данных путем запуска программ Spark из конвейера фабрики данных Azure с помощью действия Spark.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 05/08/2020
ms.openlocfilehash: e5c50d2cbd16ad2808dab485ad2b2870d6f3d350
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392363"
---
# <a name="transform-data-using-spark-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия Spark в фабрике данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-spark.md)
> * [Текущая версия](transform-data-using-spark.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие Spark в [конвейере](concepts-pipelines-activities.md) фабрики данных выполняет программу Spark в [вашем](compute-linked-services.md#azure-hdinsight-linked-service) кластере или в кластере [по требованию](compute-linked-services.md#azure-hdinsight-on-demand-linked-service) HDInsight. Данная статья основана на материалах статьи о [действиях преобразования данных](transform-data.md) , в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования. Если вы используете связанную службу Spark по требованию, фабрика данных автоматически создает кластер Spark для обработки данных, когда это необходимо, а затем удаляет кластер после завершения обработки. 


## <a name="spark-activity-properties"></a>Свойства действия Spark
Ниже приведен пример определения JSON действия Spark.    

```json
{
    "name": "Spark Activity",
    "description": "Description",
    "type": "HDInsightSpark",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "sparkJobLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "rootPath": "adfspark",
        "entryFilePath": "test.py",
        "sparkConfig": {
            "ConfigItem1": "Value"
        },
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
        ]
    }
}
```

В следующей таблице приведено описание свойств, используемых в определении JSON.

| Свойство              | Описание                              | Обязательно |
| --------------------- | ---------------------------------------- | -------- |
| name                  | Имя действия в конвейере.    | Да      |
| description           | Описание действия.  | Нет       |
| type                  | Для действия Spark используется тип действия HDInsightSpark. | Да      |
| linkedServiceName     | Имя связанной службы Spark HDInsight, в которой выполняется программа Spark. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| SparkJobLinkedService | Связанная служба службы хранилища Azure, в которой хранятся файл задания Spark, зависимости и журналы. Здесь поддерживаются только связанные службы **[Хранилище BLOB-объектов Azure](./connector-azure-blob-storage.md)** и **[ADLS 2-го поколения](./connector-azure-data-lake-storage.md)** . Если значение этого свойства не указано, используется хранилище, связанное с кластером HDInsight. Значением этого свойства может быть только связанная служба хранилища Azure. | Нет       |
| rootPath              | Контейнер BLOB-объектов Azure и папка, содержащая файл Spark. В имени файла учитывается регистр знаков. Подробнее о структуре этой папки см. в разделе о структуре папок (следующий раздел). | Да      |
| entryFilePath         | Относительный путь к корневой папке пакета и кода Spark. Файл записи должен быть файлом Python или JAR-файлом. | Да      |
| className             | Основной класс Java или Spark приложения.      | Нет       |
| аргументы             | Список аргументов командной строки для программы Spark. | Нет       |
| proxyUser             | Учетная запись пользователя для олицетворения, используемая для выполнения программы Spark. | Нет       |
| sparkConfig           | Укажите значения для свойств конфигурации Spark, перечисленных в разделе [Конфигурация Spark — свойства приложения](https://spark.apache.org/docs/latest/configuration.html#available-properties). | Нет       |
| getDebugInfo          | Указывает, когда файлы журнала Spark копируются в службу хранилища Azure, используемое кластером HDInsight или определенное sparkJobLinkedService. Допустимые значения: None (никогда), Always (всегда) или Failure (в случае сбоя). Значение по умолчанию: Нет. | Нет       |

## <a name="folder-structure"></a>Структура папок
Задания Spark обеспечивают большую гибкость, чем задания Pig и Hive. Для заданий Spark можно указать несколько зависимостей, например пакеты JAR (размещаются в CLASSPATH Java), файлы Python (размещаются в PYTHONPATH) и другие файлы.

Создайте следующую структуру папок в хранилище BLOB-объектов Azure, на которое ссылается связанная служба HDInsight. Затем передайте зависимые файлы в соответствующие вложенные папки в корневой папке, определенной значением **entryFilePath**. Например, передайте файлы Python во вложенную папку pyFiles, а JAR-файлы — во вложенную папку jars, расположенный в корневой папке. Во время выполнения служба фабрики данных ожидает в хранилище BLOB-объектов Azure следующую структуру папок.     

| путь                  | Описание                              | Обязательно | Тип   |
| --------------------- | ---------------------------------------- | -------- | ------ |
| `.` (root)            | Путь к корневому каталогу задания Spark в хранилище связанной службы. | Да      | Папка |
| &lt;Определяется пользователем&gt; | Путь к файлу записи задания Spark. | Да      | Файл   |
| ./jars                | Все файлы в этой папке передаются и помещаются в папку CLASSPATH Java для кластера. | Нет       | Папка |
| ./pyFiles             | Все файлы в этой папке передаются и помещаются в папку PYTHONPATH для кластера. | Нет       | Папка |
| ./files               | Все файлы в этой папке передаются и помещаются в рабочий каталог исполнителя. | Нет       | Папка |
| ./archives            | Все файлы в этой папке не сжаты. | Нет       | Папка |
| ./logs                | Папка, в которой содержатся журналы из кластера Spark. | Нет       | Папка |

Ниже приведен пример хранилища, содержащего два файла заданий Spark в хранилище BLOB-объектов Azure, на которое ссылается связанная служба HDInsight.

```
SparkJob1
    main.jar
    files
        input1.txt
        input2.txt
    jars
        package1.jar
        package2.jar
    logs

SparkJob2
    main.py
    pyFiles
        scrip1.py
        script2.py
    logs
```
## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами: 

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)
