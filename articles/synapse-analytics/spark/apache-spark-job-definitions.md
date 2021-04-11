---
title: Руководство по Создание определения задания Apache Spark в Synapse Studio
description: Учебник. Использование Azure Synapse Analytics для создания определений заданий Spark и их отправки в пул Apache Spark для Azure Synapse Analytics.
author: Jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 10/16/2020
ms.openlocfilehash: 15b67c969cb0464256caed58a2e7388eb7a76b9c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105608776"
---
# <a name="tutorial-create-apache-spark-job-definition-in-synapse-studio"></a>Руководство по Создание определения задания Apache Spark в Synapse Studio

В этом учебнике показано, как использовать Azure Synapse Studio для создания определений заданий Apache Spark и их отправки в бессерверный пул Apache Spark.

В рамках этого руководства рассматриваются следующие задачи:
> [!div class="checklist"]
>
> - Создание определения задания Apache Spark для PySpark (Python)
> - Создание определения задания Apache Spark для Spark (Scala)
> - Создание определения задания Apache Spark для .NET Spark (C# или F#)
> - Создание определения задания путем импорта JSON-файла
> - Экспорт файла определения задания Apache Spark в локальную среду
> - Отправка определения задания Apache Spark в виде пакетного задания
> - Добавление определения задания Apache Spark в конвейер


## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к изучению этого руководства, убедитесь, что выполнены следующие требования.

* Рабочая область Azure Synapse Analytics. См. руководство по [созданию рабочей области Azure Synapse Analytics](../../machine-learning/how-to-manage-workspace.md).
* Бессерверный пул Apache Spark.
* Учетная запись хранения ADLS 2-го поколения. Вам потребуется роль **Участник для данных BLOB-объектов хранилища** для файловой системы ADLS 2-го поколения, с которой вы хотите работать. Если у вас нет этой роли, добавьте разрешение вручную.
* Если вы не хотите использовать хранилище рабочей области по умолчанию, свяжите необходимую учетную запись хранения ADLS 2-го поколения с Synapse Studio. 


## <a name="create-an-apache-spark-job-definition-for-pyspark-python"></a>Создание определения задания Apache Spark для PySpark (Python)

В этом разделе рассказывается о том, как создать определение задания Apache Spark для PySpark (Python).

1. Откройте [Azure Synapse Studio](https://web.azuresynapse.net/).

2. Вы можете перейти на страницу [примеров файлов для создания определений заданий Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Python) и скачать **примеры файлов для python.zip**, а затем распаковать сжатый пакет и извлечь файлы **wordcount.py** и **shakespeare.txt**. 

     ![примеры файлов](./media/apache-spark-job-definitions/sample-files.png)

3. Последовательно выберите **Данные** -> **Связанные** -> **Azure Data Lake Storage 2-го поколения** и отправьте файлы **wordcount.py** и **shakespeare.txt** в файловую систему ADLS 2-го поколения. 

     ![Отправка файла Python](./media/apache-spark-job-definitions/upload-python-file.png)

4. Выберите центр **Разработка**, щелкните значок "+" и выберите пункт **Spark job definition** (Определение задания Spark), чтобы создать определение задания Spark. 

     ![Создание нового определения для Python](./media/apache-spark-job-definitions/create-new-definition.png)

5. В главном окне определения задания Apache Spark из раскрывающегося списка языков выберите **PySpark (Python)** .

     ![выбор python](./media/apache-spark-job-definitions/select-python.png)

6. Введите сведения об определении задания Apache Spark. 

     |  Свойство   | Описание   |  
     | ----- | ----- |  
     |Имя определения задания| Введите имя определения задания Apache Spark. Это имя можно изменить в любое время до публикации. <br> Образец. `job definition sample`|
     |Основной файл определения| Основной файл, используемый для задания. Выберите PY-файл в хранилище. Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**. <br> Образец. `abfss://…/path/to/wordcount.py`|
     |Аргументы командной строки| Дополнительные аргументы для задания. <br> Образец. `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Примечание. Два аргумента для образца определения задания разделены пробелами.*|
     |Файлы ссылок| Дополнительные файлы, используемые для ссылки в основном файле определения. Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**. |
     |Пул Spark| Задание будет отправлено в выбранный пул Apache Spark.|
     |Версия Spark| Версия Apache Spark, которая используется в пуле Apache Spark.|
     |Исполнители| Количество исполнителей, которые будут предоставлены для задания в определенном пуле Apache Spark.|
     |Размер исполнителя| Количество ядер и памяти, которые будут использоваться для исполнителей, предоставленных для задания в определенном пуле Apache Spark.|  
     |Размер драйвера| Количество ядер и объем памяти, которые будут использоваться для драйвера, предоставленного для задания в указанном пуле Apache Spark.|

     ![Выбор значения для определения задания Spark для Python](./media/apache-spark-job-definitions/create-py-definition.png)

7. Выберите **Опубликовать**, чтобы сохранить определение задания Apache Spark.

     ![Публикация определения PY](./media/apache-spark-job-definitions/publish-py-definition.png)

## <a name="create-an-apache-spark-job-definition-for-apache-sparkscala"></a>Создание определения задания Apache Spark для Apache Spark (Scala)

В этом разделе рассказывается о том, как создать определение задания Apache Spark для Apache Spark (Scala).

 1. Откройте [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Вы можете перейти на страницу [примеров файлов для создания определений заданий Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Scala) и скачать **примеры файлов для scala.zip**, а затем распаковать сжатый пакет и извлечь файлы **wordcount.jar** и **shakespeare.txt**. 
 
     ![примеры файлов scala](./media/apache-spark-job-definitions/sample-files-scala.png)

 3. Последовательно выберите **Данные** -> **Связанные** -> **Azure Data Lake Storage 2-го поколения** и отправьте файлы **wordcount.jar** и **shakespeare.txt** в файловую систему ADLS 2-го поколения.
 
     ![Подготовка структуры Scala](./media/apache-spark-job-definitions/prepare-scala-structure.png)

 4. Выберите центр **Разработка**, щелкните значок "+" и выберите пункт **Spark job definition** (Определение задания Spark), чтобы создать определение задания Spark. (Пример изображения аналогичен примеру для шага 4 **Создание определения задания Apache Spark для PySpark (Python)** .)

 5. В главном окне определения задания Apache Spark из раскрывающегося списка языков выберите **Spark (Scala)** .

     ![выбор scala](./media/apache-spark-job-definitions/select-scala.png)

 6. Введите сведения об определении задания Apache Spark. Пример сведений можно скопировать.

     |  Свойство   | Описание   |  
     | ----- | ----- |  
     |Имя определения задания| Введите имя определения задания Apache Spark. Это имя можно изменить в любое время до публикации. <br> Образец. `scala`|
     |Основной файл определения| Основной файл, используемый для задания. Выберите JAR-файл в хранилище. Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**. <br> Образец. `abfss://…/path/to/wordcount.jar`|
     |Имя главного класса| Полный идентификатор или основной класс, который находится в основном файле определения. <br> Образец. `WordCount`|
     |Аргументы командной строки| Дополнительные аргументы для задания. <br> Образец. `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Примечание. Два аргумента для образца определения задания разделены пробелами.* |
     |Файлы ссылок| Дополнительные файлы, используемые для ссылки в основном файле определения. Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**.|
     |Пул Spark| Задание будет отправлено в выбранный пул Apache Spark.|
     |Версия Spark| Версия Apache Spark, которая используется в пуле Apache Spark.|
     |Исполнители| Количество исполнителей, которые будут предоставлены для задания в определенном пуле Apache Spark.|  
     |Размер исполнителя| Количество ядер и памяти, которые будут использоваться для исполнителей, предоставленных для задания в определенном пуле Apache Spark.|
     |Размер драйвера| Количество ядер и объем памяти, которые будут использоваться для драйвера, предоставленного для задания в указанном пуле Apache Spark.|

     ![Выбор значения для определения задания Spark для Scala](./media/apache-spark-job-definitions/create-scala-definition.png)

 7. Выберите **Опубликовать**, чтобы сохранить определение задания Apache Spark.

      ![Публикация определения Scala](./media/apache-spark-job-definitions/publish-scala-definition.png)

## <a name="create-an-apache-spark-job-definition-for-net-sparkcf"></a>Создание определения задания Apache Spark для .NET Spark (C# или F#)

В этом разделе рассказывается о том, как создать определение задания Apache Spark для .NET Spark (C# или F#).
 1. Откройте [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Вы можете перейти на страницу [примеров файлов для создания определений заданий Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/DotNET) и скачать **примеры файлов для dotnet.zip**, а затем распаковать сжатый пакет и извлечь файлы **wordcount.zip** и **shakespeare.txt**. 

     ![пример DotNet](./media/apache-spark-job-definitions/sample-dotnet.png)

 3. Последовательно выберите **Данные** -> **Связанные** -> **Azure Data Lake Storage 2-го поколения** и отправьте файлы **wordcount.zip** и **shakespeare.txt** в файловую систему ADLS 2-го поколения.
 
     ![Подготовка структуры .NET](./media/apache-spark-job-definitions/prepare-dotnet-structure.png)

 4. Выберите центр **Разработка**, щелкните значок "+" и выберите пункт **Spark job definition** (Определение задания Spark), чтобы создать определение задания Spark. (Пример изображения аналогичен примеру для шага 4 **Создание определения задания Apache Spark для PySpark (Python)** .)

 5. В главном окне определения задания Apache Spark из раскрывающегося списка языков выберите **.NET Spark (C#/F#)** .

     ![выбор DotNet](./media/apache-spark-job-definitions/select-dotnet.png)

 6. Введите сведения об определении задания Apache Spark. Пример сведений можно скопировать.
    
     |  Свойство   | Описание   |  
     | ----- | ----- |  
     |Имя определения задания| Введите имя определения задания Apache Spark. Это имя можно изменить в любое время до публикации. <br> Образец. `dotnet`|
     |Основной файл определения| Основной файл, используемый для задания. Выберите из хранилища ZIP-файл, содержащий приложение .NET для Apache Spark (то есть основной исполняемый файл, библиотеки DLL с пользовательскими функциями и другие необходимые файлы). Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**. <br> Образец. `abfss://…/path/to/wordcount.zip`|
     |Основной исполняемый файл| Основной исполняемый файл — это основной ZIP-файл определения. <br> Образец. `WordCount`|
     |Аргументы командной строки| Дополнительные аргументы для задания. <br> Образец. `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Примечание. Два аргумента для образца определения задания разделены пробелами.* |
     |Файлы ссылок| Дополнительные файлы, необходимые рабочим узлам для запуска приложения .NET для Apache Spark, не включенные в основной ZIP-файл определения (то есть зависимые JAR-файлы, дополнительные библиотеки DLL с пользовательскими функциями и другие файлы конфигурации). Для отправки файла в учетную запись хранения можно выбрать **Отправить файл**.|
     |Пул Spark| Задание будет отправлено в выбранный пул Apache Spark.|
     |Версия Spark| Версия Apache Spark, которая используется в пуле Apache Spark.|
     |Исполнители| Количество исполнителей, которые будут предоставлены для задания в определенном пуле Apache Spark.|  
     |Размер исполнителя| Количество ядер и памяти, которые будут использоваться для исполнителей, предоставленных для задания в определенном пуле Apache Spark.|
     |Размер драйвера| Количество ядер и объем памяти, которые будут использоваться для драйвера, предоставленного для задания в указанном пуле Apache Spark.|

     ![Выбор значения для определения задания Spark для .NET](./media/apache-spark-job-definitions/create-dotnet-definition.png)

 7. Выберите **Опубликовать**, чтобы сохранить определение задания Apache Spark.

      ![Публикация определения .NET](./media/apache-spark-job-definitions/publish-dotnet-definition.png)

## <a name="create-apache-spark-job-definition-by-importing-a-json-file"></a>Создание определения задания Apache Spark путем импорта JSON-файла

 Вы можете импортировать существующий локальный файл JSON в рабочую область Azure Synapse с помощью меню **Действия** (...) в обозревателе определений заданий Apache Spark, чтобы создать новое определение задания Apache Spark.

 ![Создание определения для импорта](./media/apache-spark-job-definitions/create-import-definition.png)

 
 Определение задания Spark полностью совместимо с API Livy. Вы можете добавить параметры для других свойств Livy [(документация по Livy, раздел о REST API (apache.org))](https://livy.incubator.apache.org/docs/latest/rest-api.html) в локальный JSON-файл. Можно также указать параметры, связанные с конфигурацией Spark, в соответствующем свойстве, как показано ниже. Затем можно импортировать JSON-файл обратно, чтобы создать новое определение для пакетного задания Apache Spark. Пример JSON для импорта определения Spark:
 
```Scala
   {
  "targetBigDataPool": {
    "referenceName": "socdemolarge",
    "type": "BigDataPoolReference"
  },
  "requiredSparkVersion": "2.3",
  "language": "scala",
  "jobProperties": {
    "name": "robinSparkDefinitiontest",
    "file": "adl://socdemo-c14.azuredatalakestore.net/users/robinyao/wordcount.jar",
    "className": "WordCount",
    "args": [
      "adl://socdemo-c14.azuredatalakestore.net/users/robinyao/shakespeare.txt"
    ],
    "jars": [],
    "files": [],
    "conf": {
      "spark.dynamicAllocation.enabled": "false",
      "spark.dynamicAllocation.minExecutors": "2",
      "spark.dynamicAllocation.maxExecutors": "2"
    },
    "numExecutors": 2,
    "executorCores": 8,
    "executorMemory": "24g",
    "driverCores": 8,
    "driverMemory": "24g"
  }
}

```

![Другие свойства Livy](./media/apache-spark-job-definitions/other-livy-properties.png)

## <a name="export-an-existing-apache-spark-job-definition-file"></a>Экспорт существующего файла определения задания Apache Spark

 Можно экспортировать существующие файлы определений заданий Apache Spark в локальную среду с помощью меню **Действия** (...) в проводнике. При необходимости вы можете обновить такой JSON-файл, добавив в него свойства Livy, и импортировать его обратно для создания нового определения задания.

 ![Создание определения для экспорта](./media/apache-spark-job-definitions/create-export-definition.png)

 ![Создание определения для экспорта 2](./media/apache-spark-job-definitions/create-export-definition-2.png)

## <a name="submit-an-apache-spark-job-definition-as-a-batch-job"></a>Отправка определения задания Apache Spark в виде пакетного задания

Созданное определение задания Apache Spark можно отправить в пул Apache Spark. Убедитесь, что у вас есть роль **Участник для данных BLOB-объектов хранилища** для файловой системы ADLS 2-го поколения, с которой вы хотите работать. Если у вас нет этой роли, добавьте разрешение вручную.

### <a name="scenario-1-submit-apache-spark-job-definition"></a>Сценарий 1. Отправка определения заданий Apache Spark
 1. Откройте окно определения задания Apache Spark, выбрав его.

      ![Открытие определения задания Spark для отправки ](./media/apache-spark-job-definitions/open-spark-definition.png)

 2. Нажмите кнопку **Отправить**, чтобы отправить проект в выбранный пул Apache Spark. Вы можете щелкнуть вкладку **Spark monitoring URL** (URL-адрес мониторинга Spark), чтобы просмотреть LogQuery приложения Apache Spark.

    ![Нажатие кнопки "Отправить" для отправки определения задания Spark](./media/apache-spark-job-definitions/submit-spark-definition.png)

    ![Диалоговое окно Spark Submission (Отправка в Spark)](./media/apache-spark-job-definitions/submit-definition-result.png)

### <a name="scenario-2-view-apache-spark-job-running-progress"></a>Сценарий 2. Просмотр хода выполнения задания Apache Spark

 1. Щелкните **Монитор** и выберите параметр **Apache Spark applications** (Приложения Apache Spark). В списке будет отправленное приложение Apache Spark.

     ![Просмотр приложения Spark](./media/apache-spark-job-definitions/view-spark-application.png)

 2. Затем выберите приложение Apache Spark, которое отображается в окне задания **SparkJobDefinition**. В этом окне можно просмотреть ход выполнения задания.
     
     ![Просмотр LogQuery приложения Spark](./media/apache-spark-job-definitions/view-job-log-query.png)

### <a name="scenario-3-check-output-file"></a>Сценарий 3. Проверка выходного файла

 1. Последовательно выберите **Данные** -> **Связанные** -> **Azure Data Lake Storage 2-го поколения** (hozhaobdbj), откройте созданную ранее папку **результатов**, в которой можно просмотреть, созданы ли выходные данные.

     ![Просмотр выходного файла](./media/apache-spark-job-definitions/view-output-file.png)

## <a name="add-an-apache-spark-job-definition-into-pipeline"></a>Добавление определения задания Apache Spark в конвейер

В этом сценарии определение задания Apache Spark добавляется в конвейер.

 1. Откройте имеющееся определение задания Apache Spark.

 2. Щелкните значок в правом верхнем углу определения задания Apache Spark, выберите **Existing Pipeline** (Имеющийся конвейер) или **New pipeline** (Новый конвейер). Дополнительные сведения см. на странице Pipeline (Конвейер).

     ![Добавление в конвейер 1](./media/apache-spark-job-definitions/add-to-pipeline01.png)

     ![Добавление в конвейер 2](./media/apache-spark-job-definitions/add-to-pipeline02.png)


## <a name="next-steps"></a>Дальнейшие действия

Далее вы можете использовать Azure Synapse Studio для создания наборов данных Power BI и управления такими данными. Дополнительные сведения см. в статье [Краткое руководство. Связывание рабочей области Power BI с рабочей областью Synapse](../quickstart-power-bi.md). 

