---
title: Создание конвейеров прогнозных данных с помощью фабрики данных Azure
description: Описывает создание прогнозных конвейеров с помощью фабрики данных Azure и Машинное обучение Azure Studio (классическая модель).
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: 2773ab2a7caa1eb4d198495a3ebe4ef0c14a5a32
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104785518"
---
# <a name="create-predictive-pipelines-using-azure-machine-learning-studio-classic-and-azure-data-factory"></a>Создание прогнозирующих конвейеров с помощью Машинное обучение Azure Studio (классическая модель) и фабрики данных Azure

> [!div class="op_single_selector" title1="Действия преобразования"]
> * [Действие Hive](data-factory-hive-activity.md)
> * [Действие Pig](data-factory-pig-activity.md)
> * [Действие MapReduce](data-factory-map-reduce.md)
> * [Действие потоковой передачи Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Действие Spark](data-factory-spark.md)
> * [Действие выполнения пакета в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-batch-execution-activity.md)
> * [Действие обновления ресурса в Студии машинного обучения Azure (классическая)](data-factory-azure-ml-update-resource-activity.md)
> * [Действие хранимой процедуры](data-factory-stored-proc-activity.md)
> * [Действие U-SQL в Data Lake Analytics](data-factory-usql-activity.md)
> * [Настраиваемое действие .NET](data-factory-use-custom-activities.md)

## <a name="introduction"></a>Введение
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [преобразовании данных с помощью машинного обучения в службе "Фабрика данных"](../transform-data-using-machine-learning.md).

### <a name="azure-machine-learning-studio-classic"></a>Машинное обучение Azure Studio (классическая модель)
[Машинное обучение Azure Studio (классическая модель)](https://azure.microsoft.com/documentation/services/machine-learning/) позволяет создавать, тестировать и развертывать решения прогнозной аналитики. Этот процесс можно обобщить до трех этапов.

1. **Создайте обучающий эксперимент**. Этот шаг выполняется с помощью Машинное обучение Azure Studio (классическая модель). Studio (классическая модель) — это среда для совместной работы визуальной разработки, используемая для обучения и тестирования модели прогнозной аналитики с помощью обучающих данных.
2. **Преобразуйте его в прогнозный эксперимент**. Когда модель будет обучена с помощью существующих данных и готова для оценки новых данных, следует подготовить и оптимизировать эксперимент для оценки.
3. **Развертывание эксперимента в виде веб-службы**. Оценивающий эксперимент можно опубликовать в виде веб-службы Azure. С помощью конечной точки этой веб-службы вы можете отправлять данные в свою модель и получать из нее результаты прогнозов.

### <a name="azure-data-factory"></a>Фабрика данных Azure
Фабрика данных — это облачная служба интеграции данных, которая организует и автоматизирует **Перемещение** и **Преобразование** данных. Вы можете создавать решения для интеграции данных с помощью фабрики данных Azure, которая получает, преобразует и обрабатывает данные из различных хранилищ, а также публикует обработанные данные в хранилищах данных.

Служба фабрики данных позволяет создавать конвейеры данных для перемещения и преобразования данных, а затем запускать конвейеры по указанному расписанию (ежечасно, ежедневно, еженедельно и т. д.). Служба также предоставляет широкие возможности визуализации для отображения журнала преобразований данных и зависимостей между конвейерами данных, а также для мониторинга всех конвейеров данных в едином представлении с целью оперативного выявления проблем и настройки оповещений.

См. статьи [Общие сведения о службе фабрики данных Azure](data-factory-introduction.md) и [Создание первого конвейера](data-factory-build-your-first-pipeline.md), чтобы быстро приступить к работе со службой фабрики данных Azure.

### <a name="data-factory-and-machine-learning-studio-classic-together"></a>Фабрика данных и Машинное обучение Studio (классическая модель) вместе
Фабрика данных Azure позволяет легко создавать конвейеры, использующие опубликованную веб-службу [машинное обучение Azure Studio (классическая модель)][azure-machine-learning] для прогнозной аналитики. С помощью **действия выполнения пакета** в конвейере фабрики данных Azure можно вызвать веб-службу Studio (классическая) для выполнения прогнозов по данным в пакетной службе. Дополнительные сведения см. в разделе вызов веб-службы Машинное обучение Azure Studio (классическая модель) с помощью действия выполнения пакета.

Со временем прогнозные модели в экспериментах по оценке (классическая модель) необходимо переучить с помощью новых входных наборов данных. Вы можете переучить модель Studio (классической) из конвейера фабрики данных, выполнив следующие действия.

1. Опубликуйте обучающий (не прогнозный) эксперимент в виде веб-службы. Этот шаг выполняется в среде Studio (классическая модель), так как в предыдущем сценарии можно предоставить прогнозный эксперимент в качестве веб-службы.
2. Используйте действие выполнения пакета Studio (классическая модель), чтобы вызвать веб-службу для обучающего эксперимента. По сути, можно использовать действие выполнения пакета Studio (классическая модель) для вызова веб-службы для обучения и оценки.

После завершения повторного обучения обновите веб-службу оценки (прогнозный эксперимент, представленный в виде веб-службы) с использованием новой обученной модели с помощью **действия ресурса обновления машинное обучение Azure Studio (классическая модель)**. Дополнительные сведения см. в статье [Updating Azure Machine Learning models using Update Resource Activity](data-factory-azure-ml-update-resource-activity.md) (Обновление моделей машинного обучения Azure с помощью действия "Обновить ресурс").

## <a name="invoking-a-web-service-using-batch-execution-activity"></a>Вызов веб-службы с помощью действия выполнения пакета
Фабрика данных Azure используется для координации перемещения и обработки данных, а затем выполнения пакетного выполнения с помощью Studio (классическая модель). Для этого выполните действия верхнего уровня:

1. Создайте связанную службу Машинное обучение Azure Studio (классическая модель). Вам потребуются следующие значения:

   1. **Универсальный код ресурса (URI) запроса** для API пакетного выполнения. Чтобы найти URI запроса, перейдите по ссылке **Выполнение пакета** на странице веб-служб.
   2. **Ключ API** для опубликованной (классической) веб-службы Studio. Чтобы найти ключ API, щелкните опубликованную веб-службу.
   3. Используйте действие **AzureMLBatchExecution** .

      ![Панель мониторинга Машинное обучение Studio (классическая модель)](./media/data-factory-azure-ml-batch-execution-activity/AzureMLDashboard.png)

      ![Универсальный код ресурса (URI) пакета](./media/data-factory-azure-ml-batch-execution-activity/batch-uri.png)

### <a name="scenario-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>Сценарий: эксперименты с вводом-выводом веб-службы, ссылающейся на данные в хранилище больших двоичных объектов Azure
В этом сценарии веб-служба Studio (классическая модель) выполняет прогнозы, используя данные из файла в хранилище BLOB-объектов Azure, и сохраняет результаты прогноза в хранилище больших двоичных объектов. Следующий код JSON определяет конвейер фабрики данных действием AzureMLBatchExecution. У действия имеется входной (**DecisionTreeInputBlob**) и выходной (**DecisionTreeResultBlob**) наборы данных. Набор **DecisionTreeInputBlob** передается в веб-службу в качестве входных данных с помощью свойства JSON **webServiceInput**, а **DecisionTreeResultBlob** — в качестве выходных данных с помощью свойства JSON **webServiceOutputs**.

> [!IMPORTANT]
> Если веб-служба принимает несколько входных наборов данных, используйте свойство **webServiceInputs** вместо **webServiceInput**. В соответствующем разделе [Веб-службе требуется несколько входных данных](#web-service-requires-multiple-inputs) приведен пример использования свойства webServiceInputs.
>
> Наборы данных, на которые ссылаются свойства **webServiceInput** / **webServiceInputs** и **webServiceOutputs** (в **typeProperties**), также должны включаться в **входные** и **выходные данные** действия.
>
> В эксперименте Studio (классическая модель), входные и выходные порты веб-службы и глобальные параметры имеют имена по умолчанию ("INPUT1", "INPUT2"), которые можно настраивать. Имена, используемые для параметров webServiceInputs, webServiceOutputs и globalParameters, должны точно совпадать с именами в экспериментах. Вы можете просмотреть пример полезных данных запроса на странице справки по выполнению пакета для классической конечной точки Studio, чтобы проверить ожидаемое сопоставление.
>
>

```json
{
  "name": "PredictivePipeline",
  "properties": {
    "description": "use AzureML model",
    "activities": [
      {
        "name": "MLActivity",
        "type": "AzureMLBatchExecution",
        "description": "prediction analysis on batch input",
        "inputs": [
          {
            "name": "DecisionTreeInputBlob"
          }
        ],
        "outputs": [
          {
            "name": "DecisionTreeResultBlob"
          }
        ],
        "linkedServiceName": "MyAzureMLLinkedService",
        "typeProperties":
        {
            "webServiceInput": "DecisionTreeInputBlob",
            "webServiceOutputs": {
                "output1": "DecisionTreeResultBlob"
            }
        },
        "policy": {
          "concurrency": 3,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        }
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```

> [!NOTE]
> Только входные и выходные данные действия AzureMLBatchExecution могут передаваться как параметры веб-службы. Например, в приведенном выше фрагменте JSON DecisionTreeInputBlob —входные данные действия AzureMLBatchExecution, которые передаются в качестве входных данных в веб-службу через параметр webServiceInput.
>
>

### <a name="example"></a>Пример
В этом примере для размещения входных и выходных данных используется служба хранилища Azure.

Советуем, прежде чем приступить к этому примеру, ознакомиться со статьей [Учебник. Создание первого конвейера для обработки данных с помощью кластера Hadoop][adf-build-1st-pipeline]. C помощью редактора фабрики данных можно создать артефакты фабрики данных (связанные службы, наборы данных и конвейер) для этого примера.

1. Создайте **связанную службу** для **службы хранилища Azure**. Если входные и выходные файлы находятся в разных учетных записях хранения, то потребуются две связанные службы. Ниже приведен пример JSON:

   ```json
   {
     "name": "StorageLinkedService",
     "properties": {
       "type": "AzureStorage",
       "typeProperties": {
         "connectionString": "DefaultEndpointsProtocol=https;AccountName= [acctName];AccountKey=[acctKey]"
       }
     }
   }
   ```

2. Создайте **входной****набор данных** фабрики данных Azure. В отличие от некоторых других наборов данных фабрики данных, эти наборы должны содержать значения **folderPath** и **fileName**. Можно использовать секционирование для выполнения каждого пакета (каждого среза данных) для обработки или создания уникальных входных и выходных файлов. Возможно, потребуется включить некоторые вышестоящие действия для преобразования входных данных в формат CSV-файла и его размещения в учетной записи хранения для каждого среза. В этом случае не будут включены параметры **external** и **externalData**, как показано в примере ниже, а DecisionTreeInputBlob будет выходным набором данных другого действия.

   ```json
   {
     "name": "DecisionTreeInputBlob",
     "properties": {
       "type": "AzureBlob",
       "linkedServiceName": "StorageLinkedService",
       "typeProperties": {
         "folderPath": "azuremltesting/input",
         "fileName": "in.csv",
         "format": {
           "type": "TextFormat",
           "columnDelimiter": ","
         }
       },
       "external": true,
       "availability": {
         "frequency": "Day",
         "interval": 1
       },
       "policy": {
         "externalData": {
           "retryInterval": "00:01:00",
           "retryTimeout": "00:10:00",
           "maximumRetry": 3
         }
       }
     }
   }
   ```

   Входной CSV-файл должен содержать строку заголовков столбцов. Если вы используете **действие копирования** для создания или перемещения CSV-файла в хранилище BLOB-объектов, следует установить для свойства **blobWriterAddHeader** приемника значение **true**. Пример:

   ```json
   sink:
   {
     "type": "BlobSink",
     "blobWriterAddHeader": true
     }
   ```

   если CSV-файл не имеет строку заголовков, может появиться следующая ошибка: **Ошибка в действии: ошибка при чтении строки. Непредвиденный токен: StartObject. Путь '', строка 1, позиция 1**.

3. Создайте **выходной****набор данных** фабрики данных Azure. В этом примере используется секционирование для создания уникального выходного пути при выполнении каждого из срезов. Без этого действие перезапишет файл.

   ```json
   {
     "name": "DecisionTreeResultBlob",
     "properties": {
       "type": "AzureBlob",
       "linkedServiceName": "StorageLinkedService",
       "typeProperties": {
         "folderPath": "azuremltesting/scored/{folderpart}/",
         "fileName": "{filepart}result.csv",
         "partitionedBy": [
           {
             "name": "folderpart",
             "value": {
               "type": "DateTime",
               "date": "SliceStart",
               "format": "yyyyMMdd"
             }
           },
           {
             "name": "filepart",
             "value": {
               "type": "DateTime",
               "date": "SliceStart",
               "format": "HHmmss"
             }
           }
         ],
         "format": {
           "type": "TextFormat",
           "columnDelimiter": ","
         }
       },
       "availability": {
         "frequency": "Day",
         "interval": 15
       }
     }
   }
   ```

4. Создайте **связанную службу** типа **AzureMLLinkedService**, предоставляющую ключ API и URL-адрес выполнения пакета для модели.

   ```json
   {
     "name": "MyAzureMLLinkedService",
     "properties": {
       "type": "AzureML",
       "typeProperties": {
         "mlEndpoint": "https://[batch execution endpoint]/jobs",
         "apiKey": "[apikey]"
       }
     }
   }
   ```

5. Наконец, создайте конвейер, содержащий действие **AzureMLBatchExecution** . Конвейер выполняет следующие действия:

   1. Получает расположение входного файла из входных наборов данных.
   2. Вызывает API пакетного выполнения пакета студии (классическая модель)
   3. Копирует выходные данные пакетного выполнения в заданный большой двоичный объект в выходном наборе данных.

      > [!NOTE]
      > У действия AzureMLBatchExecution может быть ноль или более входных наборов данных и один или несколько выходных наборов данных.
      >
      >

      ```json
      {
        "name": "PredictivePipeline",
        "properties": {
          "description": "use AzureML model",
          "activities": [
            {
              "name": "MLActivity",
              "type": "AzureMLBatchExecution",
              "description": "prediction analysis on batch input",
              "inputs": [
                {
                  "name": "DecisionTreeInputBlob"
                }
                ],
              "outputs": [
                {
                  "name": "DecisionTreeResultBlob"
                }
                ],
              "linkedServiceName": "MyAzureMLLinkedService",
              "typeProperties":
                {
                "webServiceInput": "DecisionTreeInputBlob",
                "webServiceOutputs": {
                  "output1": "DecisionTreeResultBlob"
                }
                },
              "policy": {
                "concurrency": 3,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "02:00:00"
              }
            }
          ],
          "start": "2016-02-13T00:00:00Z",
          "end": "2016-02-14T00:00:00Z"
        }
      }
      ```

      **Начальные** и **конечные** даты и время должны иметь [Формат ISO](https://en.wikipedia.org/wiki/ISO_8601). Например, 2014-10-14T16:32:41Z. Значение времени окончания (**end**) является необязательным. Если значение свойства **End** не указано, оно вычисляется как "**Начало + 48 часов".** Чтобы запустить конвейер в течение неопределенного срока, укажите значение **9999-09-09** в качестве значения свойства **end**. Дополнительные сведения о свойствах JSON см. в [справочнике по скриптам JSON](/previous-versions/azure/dn835050(v=azure.100)).

      > [!NOTE]
      > Указывать входные данные для действия AzureMLBatchExecution необязательно.
      >
      >

### <a name="scenario-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>Сценарий: эксперименты со использованием модулей чтения и записи для обращения к данным в различных хранилищах
Другим распространенным сценарием при создании экспериментов Studio (классическая модель) является использование модулей чтения и записи. Модуль чтения используется для загрузки данных в эксперимент, а модуль записи — для сохранения данных экспериментов. Дополнительные сведения о модулях [чтения](/azure/machine-learning/studio-module-reference/import-data) и [записи](/azure/machine-learning/studio-module-reference/export-data) см. в соответствующих разделах в библиотеке MSDN.

При использовании модулей чтения и записи рекомендуется указывать параметр веб-службы для каждого их свойства. Эти параметры веб-службы позволяют настроить значения во время выполнения. Например, можно создать эксперимент с модулем чтения, который использует базу данных SQL Azure: XXX.database.windows.net. После развертывания веб-службы необходимо разрешить потребителям веб-службы указать другой логический SQL Server с именем YYY.database.windows.net. Вы можете использовать параметр веб-службы, чтобы разрешить настройку этого значения.

> [!NOTE]
> Входные и выходные данные веб-службы отличаются от параметров веб-службы. В первом сценарии вы узнали, как можно указать входные и выходные данные для веб-службы Studio (классическая модель). В этом сценарии вы передаете параметры для веб-службы, соответствующие свойствам модулей чтения и записи.
>
>

Рассмотрим сценарий использования параметров веб-службы. У вас есть развернутая веб-служба Studio (классическая), которая использует модуль чтения для чтения данных из одного из источников данных, поддерживаемых Studio (классическая модель) (например, база данных SQL Azure). После пакетного выполнения результаты записываются с помощью модуля записи (база данных SQL Azure).  В экспериментах не определены входные и выходные данные веб-службы. В этом случае мы советуем настроить соответствующие параметры веб-службы для модулей чтения и записи. Это позволит настроить их при использовании действия AzureMLBatchExecution. Параметры веб-службы указываются в разделе **globalParameters** действия JSON, как показано ниже.

```json
"typeProperties": {
    "globalParameters": {
        "Param 1": "Value 1",
        "Param 2": "Value 2"
    }
}
```

Вы также можете использовать [функции фабрики данных](data-factory-functions-variables.md) при передаче значений для параметров веб-службы, как показано в следующем примере:

```json
"typeProperties": {
    "globalParameters": {
       "Database query": "$$Text.Format('SELECT * FROM myTable WHERE timeColumn = \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(WindowStart, 0))"
    }
}
```

> [!NOTE]
> В параметрах веб-службы учитывается регистр, поэтому убедитесь, что имена, которые указываются в действии JSON, соответствуют именам, предоставляемым веб-службой.
>
>

### <a name="using-a-reader-module-to-read-data-from-multiple-files-in-azure-blob"></a>Использование модуля чтения для чтения данных из нескольких файлов в большом двоичном объекте Azure
Конвейеры больших данных с такими действиями, как Pig и Hive, могут формировать один или несколько выходных файлов без расширений. Например, если указать внешнюю таблицу Hive, данные для нее могут храниться в хранилище больших двоичных объектов с следующим именем: 000000_0. В эксперименте можно применять модуль чтения для чтения нескольких файлов и использовать их для прогнозирования.

При использовании модуля чтения в эксперименте Studio (классическая модель) вы можете указать в качестве входных данных большой двоичный объект Azure. Файлы в хранилище BLOB-объектов Azure могут представлять собой выходные файлы (например, 000000_0), созданные с помощью сценария Pig и Hive, который выполняется в HDInsight. Модуль чтения позволяет читать файлы (без расширений). Для этого необходимо настроить параметр **Path to container, directory/blob** (Путь к контейнеру, каталогу или большому двоичному объекту). Параметр **Path to container** (Путь к контейнеру) указывает на контейнер, а **directory/blob** (Путь к каталогу или большому двоичному объекту) — на папку, содержащую файлы, как показано на рисунке ниже. Звездочка (\*) **указывает на то, что все файлы в контейнере или папке (например, data/aggregateddata/year=2014/month-6/\*)** будут считываться в рамках эксперимента.

![Свойства большого двоичного объекта Azure](./media/data-factory-create-predictive-pipelines/azure-blob-properties.png)

### <a name="example"></a>Пример
#### <a name="pipeline-with-azuremlbatchexecution-activity-with-web-service-parameters"></a>Конвейер с действием AzureMLBatchExecution с параметрами веб-службы

```JSON
{
  "name": "MLWithSqlReaderSqlWriter",
  "properties": {
    "description": "Azure Machine Learning Studio (classic) model with sql azure reader/writer",
    "activities": [
      {
        "name": "MLSqlReaderSqlWriterActivity",
        "type": "AzureMLBatchExecution",
        "description": "test",
        "inputs": [
          {
            "name": "MLSqlInput"
          }
        ],
        "outputs": [
          {
            "name": "MLSqlOutput"
          }
        ],
        "linkedServiceName": "MLSqlReaderSqlWriterDecisionTreeModel",
        "typeProperties":
        {
            "webServiceInput": "MLSqlInput",
            "webServiceOutputs": {
                "output1": "MLSqlOutput"
            }
              "globalParameters": {
                "Database server name": "<myserver>.database.windows.net",
                "Database name": "<database>",
                "Server user account name": "<user name>",
                "Server user account password": "<password>"
              }
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        },
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```

В приведенном выше примере JSON:

* Развернутая веб-служба Studio (классическая) использует модуль чтения и записи для чтения и записи данных из базы данных SQL Azure и обратно. Эта веб-служба предоставляет следующие параметры: Database server name, Database name, Server user account name и Server user account password.
* **Начальные** и **конечные** даты и время должны иметь [Формат ISO](https://en.wikipedia.org/wiki/ISO_8601). Например, 2014-10-14T16:32:41Z. Значение времени окончания (**end**) является необязательным. Если значение свойства **End** не указано, оно вычисляется как "**Начало + 48 часов".** Чтобы запустить конвейер в течение неопределенного срока, укажите значение **9999-09-09** в качестве значения свойства **end**. Дополнительные сведения о свойствах JSON см. в [справочнике по скриптам JSON](/previous-versions/azure/dn835050(v=azure.100)).

### <a name="other-scenarios"></a>Другие сценарии
#### <a name="web-service-requires-multiple-inputs"></a>Веб-службе требуется несколько входных данных
Если веб-служба принимает несколько входных наборов данных, используйте свойство **webServiceInputs** вместо **webServiceInput**. Наборы данных, на которые ссылается свойство **webServiceInputs**, также должны быть включены в раздел **inputs** действия.

В эксперименте Машинное обучение Azure Studio (классическая модель), входные и выходные порты веб-службы и глобальные параметры имеют имена по умолчанию ("INPUT1", "INPUT2"), которые можно настраивать. Имена, используемые для параметров webServiceInputs, webServiceOutputs и globalParameters, должны точно совпадать с именами в экспериментах. Вы можете просмотреть пример полезных данных запроса на странице справки по выполнению пакета для классической конечной точки Studio, чтобы проверить ожидаемое сопоставление.

```JSON
{
    "name": "PredictivePipeline",
    "properties": {
        "description": "use AzureML model",
        "activities": [{
            "name": "MLActivity",
            "type": "AzureMLBatchExecution",
            "description": "prediction analysis on batch input",
            "inputs": [{
                "name": "inputDataset1"
            }, {
                "name": "inputDataset2"
            }],
            "outputs": [{
                "name": "outputDataset"
            }],
            "linkedServiceName": "MyAzureMLLinkedService",
            "typeProperties": {
                "webServiceInputs": {
                    "input1": "inputDataset1",
                    "input2": "inputDataset2"
                },
                "webServiceOutputs": {
                    "output1": "outputDataset"
                }
            },
            "policy": {
                "concurrency": 3,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "02:00:00"
            }
        }],
        "start": "2016-02-13T00:00:00Z",
        "end": "2016-02-14T00:00:00Z"
    }
}
```

#### <a name="web-service-does-not-require-an-input"></a>Веб-служба не требует входных данных
Веб-службы пакетного выполнения Машинное обучение Azure Studio (классическая модель) можно использовать для запуска любых рабочих процессов, например скриптов R или Python, которые могут не требовать никаких входных данных. Кроме того, эксперимент можно настроить с помощью модуля чтения, не предоставляющего никаких глобальных параметров. В этом случае настройка действия AzureMLBatchExecution выполняется следующим образом:

```JSON
{
    "name": "scoring service",
    "type": "AzureMLBatchExecution",
    "outputs": [
        {
            "name": "myBlob"
        }
    ],
    "typeProperties": {
        "webServiceOutputs": {
            "output1": "myBlob"
        }
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

#### <a name="web-service-does-not-require-an-inputoutput"></a>Веб-служба не требует входных или выходных данных
В веб-службе пакетного выполнения Машинное обучение Azure Studio (классическая) могут отсутствовать настроенные выходные данные веб-службы. В данном примере не настроены ни входные или выходные параметры, ни глобальные параметры. Выходные данные в действии настроены, но не назначены в качестве выходных данных в свойстве webServiceOutput.

```JSON
{
    "name": "retraining",
    "type": "AzureMLBatchExecution",
    "outputs": [
        {
            "name": "placeholderOutputDataset"
        }
    ],
    "typeProperties": {
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

#### <a name="web-service-uses-readers-and-writers-and-the-activity-runs-only-when-other-activities-have-succeeded"></a>Веб-служба использует модули чтения и записи, а действие выполняется только при успешном завершении других действий
Модули чтения и записи веб-службы Машинное обучение Azure Studio (классическая модель) могут быть настроены для работы с GlobalParameters или без них. Однако вызовы службы можно встроить в конвейер с использованием зависимостей наборов данных для вызова службы только при успешном завершении обработки вышестоящих данных. При таком подходе после окончания пакетного выполнения можно запустить другое действие. В этом случае зависимости можно представить, используя входные и выходные данные действия, но не называя их входными или выходными данными веб-службы.

```JSON
{
    "name": "retraining",
    "type": "AzureMLBatchExecution",
    "inputs": [
        {
            "name": "upstreamData1"
        },
        {
            "name": "upstreamData2"
        }
    ],
    "outputs": [
        {
            "name": "downstreamData"
        }
    ],
    "typeProperties": {
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

**Общие выводы** :

* Если в конечной точке эксперимента используются входные данные веб-службы, они представляются набором данных больших двоичных объектов и включаются во входные данные действия, а также в свойство webServiceInput. В противном случае свойство webServiceInput опускается.
* Если в конечной точке эксперимента используются выходные данные веб-службы, они представляются наборами данных больших двоичных объектов и включаются в выходные данные действия, а также в свойство webServiceOutputs. Выходные данные действия и свойство webServiceOutputs сопоставляются с именем выходных данных в эксперименте. В противном случае свойство webServiceOutputs опускается.
* Если конечная точка эксперимента раскрывает глобальные параметры, они даются в свойстве globalParameters действия в виде пар "ключ — значение". В противном случае свойство globalParameters опускается. В ключах учитывается регистр. [функции фабрики данных Azure](data-factory-functions-variables.md) .
* Дополнительные наборы данных могут включаться в свойства входных и выходных данных действия без включения соответствующих ссылок в раздел typeProperties свойства. Они управляют выполнением с помощью зависимостей срезов. В остальном действие AzureMLBatchExecution их игнорирует.


## <a name="updating-models-using-update-resource-activity"></a>Обновление моделей с помощью действия обновления ресурса
После завершения повторного обучения обновите веб-службу оценки (прогнозный эксперимент, представленный в виде веб-службы) с использованием новой обученной модели с помощью **действия ресурса обновления машинное обучение Azure Studio (классическая модель)**. Дополнительные сведения см. в статье [Updating Azure Machine Learning models using Update Resource Activity](data-factory-azure-ml-update-resource-activity.md) (Обновление моделей машинного обучения Azure с помощью действия "Обновить ресурс").

### <a name="reader-and-writer-modules"></a>Модули чтения и записи
Распространенный сценарий использования параметров веб-службы заключается в использовании модулей чтения и записи SQL Azure. Модуль чтения используется для загрузки данных в эксперимент из служб управления данными, находящихся за пределами среды Studio (классическая модель). Модуль записи предназначен для сохранения данных из экспериментов в службах управления данными за пределами среды Studio (классическая модель).

Сведения о модулях [чтения](/azure/machine-learning/studio-module-reference/import-data) и [записи](/azure/machine-learning/studio-module-reference/export-data) больших двоичных объектов Azure и SQL Azure см. в соответствующих разделах в библиотеке MSDN. В примере из предыдущего раздела используется модуль чтения и модуль записи больших двоичных объектов Azure. В этом разделе рассматривается использование модуля чтения и модуля записи SQL Azure.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы
**Вопрос.** Имеется несколько файлов, сформированных моими конвейерами больших данных. Можно использовать действие AzureMLBatchExecution для работы с этими файлами?

**Ответ.** Да. Дополнительные сведения см. в статье **Использование модуля чтения для чтения данных из нескольких файлов в большом двоичном объекте Azure**.

## <a name="azure-machine-learning-studio-classic-batch-scoring-activity"></a>Действие пакетной оценки Машинное обучение Azure Studio (классическая модель)
Если вы используете действие **AzureMLBatchScoring** для интеграции с машинное обучение Azure Studio (классической), рекомендуется использовать последнюю действие **AzureMLBatchExecution** .

Действие AzureMLBatchExecution введено в 2015 году в августовском выпуске пакета SDK для Azure и Azure PowerShell.

Если вы планируете использовать действие AzureMLBatchScoring дальше, продолжите чтение этого раздела.

### <a name="azure-machine-learning-studio-classic-batch-scoring-activity-using-azure-storage-for-inputoutput"></a>Действие пакетной оценки Машинное обучение Azure Studio (классическая модель) с использованием хранилища Azure для ввода-вывода

```JSON
{
  "name": "PredictivePipeline",
  "properties": {
    "description": "use AzureML model",
    "activities": [
      {
        "name": "MLActivity",
        "type": "AzureMLBatchScoring",
        "description": "prediction analysis on batch input",
        "inputs": [
          {
            "name": "ScoringInputBlob"
          }
        ],
        "outputs": [
          {
            "name": "ScoringResultBlob"
          }
        ],
        "linkedServiceName": "MyAzureMLLinkedService",
        "policy": {
          "concurrency": 3,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        }
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```

### <a name="web-service-parameters"></a>Параметры веб-службы
Чтобы указать значения для параметров веб-службы, добавьте раздел **typeProperties** в раздел **AzureMLBatchScoringActivty** в конвейере JSON, как показано в примере ниже.

```JSON
"typeProperties": {
    "webServiceParameters": {
        "Param 1": "Value 1",
        "Param 2": "Value 2"
    }
}
```
Вы также можете использовать [функции фабрики данных](data-factory-functions-variables.md) при передаче значений для параметров веб-службы, как показано в следующем примере:

```JSON
"typeProperties": {
    "webServiceParameters": {
       "Database query": "$$Text.Format('SELECT * FROM myTable WHERE timeColumn = \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(WindowStart, 0))"
    }
}
```

> [!NOTE]
> В параметрах веб-службы учитывается регистр, поэтому убедитесь, что имена, которые указываются в действии JSON, соответствуют именам, предоставляемым веб-службой.
>
>

## <a name="see-also"></a>См. также:
* [Запись блога Azure: «Приступая к работе с фабрикой данных Azure и Машинным обучением Azure»](https://azure.microsoft.com/blog/getting-started-with-azure-data-factory-and-azure-machine-learning-4/)

[adf-build-1st-pipeline]: data-factory-build-your-first-pipeline.md

[azure-machine-learning]: https://azure.microsoft.com/services/machine-learning/