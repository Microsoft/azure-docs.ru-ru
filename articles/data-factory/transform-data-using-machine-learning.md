---
title: Создание конвейеров прогнозных данных
description: Узнайте, как создать прогнозный конвейер с помощью Машинное обучение Azure Studio (классическая модель) — действия выполнения пакета в фабрике данных Azure.
author: nabhishek
ms.author: abnarain
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/16/2020
ms.openlocfilehash: 37a31891c3c1d812b396548036c4b59cc6523c2d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375669"
---
# <a name="create-a-predictive-pipeline-using-azure-machine-learning-studio-classic-and-azure-data-factory"></a>Создание прогнозного конвейера с помощью Машинное обучение Azure Studio (классическая модель) и фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-azure-ml-batch-execution-activity.md)
> * [Текущая версия](transform-data-using-machine-learning.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Машинное обучение Azure Studio (классическая модель)](https://azure.microsoft.com/documentation/services/machine-learning/) позволяет создавать, тестировать и развертывать решения прогнозной аналитики. Этот процесс можно обобщить до трех этапов.

1. **Создайте обучающий эксперимент**. Этот шаг выполняется с помощью Машинное обучение Azure Studio (классическая модель). Машинное обучение Azure Studio (классическая модель) — это среда для совместной работы визуальной разработки, используемая для обучения и тестирования модели прогнозной аналитики с помощью обучающих данных.
2. **Преобразуйте его в прогнозный эксперимент**. Когда модель будет обучена с помощью существующих данных и готова для оценки новых данных, следует подготовить и оптимизировать эксперимент для оценки.
3. **Развертывание эксперимента в виде веб-службы**. Оценивающий эксперимент можно опубликовать в виде веб-службы Azure. С помощью конечной точки этой веб-службы вы можете отправлять данные в свою модель и получать из нее результаты прогнозов.

### <a name="data-factory-and-azure-machine-learning-studio-classic-together"></a>Фабрика данных и Машинное обучение Azure Studio (классическая модель) вместе
Фабрика данных Azure позволяет легко создавать конвейеры, использующие опубликованную веб-службу [машинное обучение Azure Studio (классическая модель)](https://azure.microsoft.com/documentation/services/machine-learning) для прогнозной аналитики. С помощью **действия выполнения пакета** в конвейере фабрики данных Azure можно вызвать веб-службу машинное обучение Azure Studio (классическая), чтобы сделать прогнозы для данных в пакетной службе.

Со временем прогнозные модели в экспериментах оценки Машинное обучение Azure Studio (классическая модель) необходимо переучить с помощью новых входных наборов данных. Чтобы повторно обучить модель из конвейера Фабрики данных, выполните следующие действия:

1. Опубликуйте обучающий (не прогнозный) эксперимент в виде веб-службы. Этот шаг выполняется в Машинное обучение Azure Studio (классическая модель), так как в предыдущем сценарии можно предоставить прогнозный эксперимент в качестве веб-службы.
2. Используйте действие выполнения пакета Машинное обучение Azure Studio (классическая модель), чтобы вызвать веб-службу для обучающего эксперимента. По сути, вы можете использовать действие пакетного выполнения Машинное обучение Azure Studio (классическая модель) для вызова веб-службы обучения и оценки.

После завершения повторного обучения обновите веб-службу оценки (прогнозный эксперимент, представленный в виде веб-службы) с использованием новой обученной модели с помощью **действия ресурса обновления машинное обучение Azure Studio (классическая модель)**. Дополнительные сведения см. в статье [Updating Azure Machine Learning models using Update Resource Activity](update-machine-learning-models.md) (Обновление моделей машинного обучения Azure с помощью действия "Обновить ресурс").

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Связанная служба Машинное обучение Azure Studio (классическая модель)

Создайте связанную службу **машинное обучение Azure Studio (классическая)** , чтобы связать веб-службу машинное обучение Azure Studio (классическая) с фабрикой данных Azure. Связанная служба используется действием выполнения пакета Машинное обучение Azure Studio (классическая модель) и [действием обновления ресурса](update-machine-learning-models.md).

```JSON
{
    "type" : "linkedServices",
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "URL to Azure ML Predictive Web Service",
            "apiKey": {
                "type": "SecureString",
                "value": "api key"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

В статье [Вычисление поддерживаемых сред с помощью фабрики данных Azure](compute-linked-services.md) описаны свойства в определении JSON.

Машинное обучение Azure Studio (классическая модель) поддерживает классические веб-службы и новые веб-службы для прогнозного эксперимента. В фабрике данных можно выбрать один из них для использования. Чтобы получить сведения, необходимые для создания связанной службы Машинное обучение Azure Studio (классическая модель), перейдите на страницу https://services.azureml.net , где перечислены все ваши (новые) веб-службы и классические веб-службы. Щелкните веб-службу, к которой требуется получить доступ, а затем щелкните страницу **Использование**. Скопируйте **первичный ключ** для свойства **apiKey** и **запросы пакетной службы** для свойства **mlEndpoint**.

![Веб-службы Машинное обучение Azure Studio (классическая модель)](./media/transform-data-using-machine-learning/web-services.png)

## <a name="azure-machine-learning-studio-classic-batch-execution-activity"></a>Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)

В следующем фрагменте кода JSON определяется действие выполнения пакета Машинное обучение Azure Studio (классическая модель). Определение действия содержит ссылку на созданную ранее связанную службу Машинное обучение Azure Studio (классическая модель).

```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "<web service input name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path1"
            },
            "<web service input name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path2"
            }
        },
        "webServiceOutputs": {
            "<web service output name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path3"
            },
            "<web service output name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path4"
            }
        },
        "globalParameters": {
            "<Parameter 1 Name>": "<parameter value>",
            "<parameter 2 name>": "<parameter 2 value>"
        }
    }
}
```

| Свойство.          | Описание                              | Обязательно |
| :---------------- | :--------------------------------------- | :------- |
| name              | Имя действия в конвейере.     | Да      |
| description       | Описание действия.  | Нет       |
| type              | Для Data Lake Analytics действия U-SQL тип действия — **AzureMLBatchExecution**. | Да      |
| linkedServiceName | Связанные службы для связанной службы Машинное обучение Azure Studio (классическая модель). Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| webServiceInputs  | Пары "ключ — значение" — сопоставление имен входных данных веб-службы Машинное обучение Azure Studio (классическая модель). Ключ должен соответствовать входным параметрам, определенным в опубликованной веб-службе Машинное обучение Azure Studio (классическая модель). Значение — это пара свойств связанных служб хранилища Azure и FilePath, указывающих на расположения входных больших двоичных объектов. | Нет       |
| webServiceOutputs | Пары "ключ — значение" — сопоставление имен выходных данных веб-службы Машинное обучение Azure Studio (классическая модель). Ключ должен соответствовать выходным параметрам, определенным в опубликованной веб-службе Машинное обучение Azure Studio (классическая модель). Значением является пара свойств связанных служб хранилища Azure и FilePath, указывающих на расположения выходных больших двоичных объектов. | Нет       |
| globalParameters  | Пары "ключ — значение", передаваемые в конечную точку службы пакетного выполнения Машинное обучение Azure Studio (классическая модель). Ключи должны соответствовать именам параметров веб-службы, определенных в опубликованной веб-службе Машинное обучение Azure Studio (классическая модель). Значения передаются в свойстве GlobalParameters запроса выполнения пакета Машинное обучение Azure Studio (классическая модель). | Нет       |

### <a name="scenario-1-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>Сценарий 1. Эксперименты с вводом-выводом веб-службы, ссылающейся на данные в хранилище BLOB-объектов

В этом сценарии веб-служба Машинное обучение Azure Studio (классическая) делает прогнозы, используя данные из файла в хранилище BLOB-объектов Azure, и сохраняет результаты прогноза в хранилище больших двоичных объектов. Следующий код JSON определяет конвейер фабрики данных действием AzureMLBatchExecution. Входные и выходные данные в хранилище BLOB-объектов Azure указываются с помощью пары LinkedName и FilePath. В образце связанной службы входных и выходных данных можно использовать различные связанные службы для каждого из входных и выходных файлов фабрики, чтобы иметь возможность выбрать нужные файлы и отправить их в веб-службу Машинное обучение Azure Studio (классическая модель).

> [!IMPORTANT]
> В эксперименте Машинное обучение Azure Studio (классическая модель), входные и выходные порты веб-службы и глобальные параметры имеют имена по умолчанию ("INPUT1", "INPUT2"), которые можно настраивать. Имена, используемые для параметров webServiceInputs, webServiceOutputs и globalParameters, должны точно совпадать с именами в экспериментах. Вы можете просмотреть пример полезных данных запроса на странице справки по выполнению пакетов для конечной точки Машинное обучение Azure Studio (классической), чтобы проверить ожидаемое сопоставление.


```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "input1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in1.csv"
            },
            "input2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in2.csv"
            }
        },
        "webServiceOutputs": {
            "outputName1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out1.csv"
            },
            "outputName2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out2.csv"
            }
        }
    }
}
```
### <a name="scenario-2-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>Сценарий 2. Эксперименты с использованием модулей чтения и записи для обращения к данным в различных хранилищах
Другим распространенным сценарием при создании экспериментов Машинное обучение Azure Studio (классическая модель) является использование модулей импорта данных и выходных данных. Модуль импорта используется для загрузки данных в эксперимент, а модуль вывода — для сохранения данных экспериментов. Дополнительные сведения о модулях импорта и вывода данных см. в [этом](/azure/machine-learning/studio-module-reference/import-data) и в [этом](/azure/machine-learning/studio-module-reference/export-data) разделах библиотеки MSDN.

При использовании модулей импорта и вывода данных рекомендуется указывать параметр веб-службы для каждого их свойства. Эти параметры веб-службы позволяют настроить значения во время выполнения. Например, можно создать эксперимент с модулем импорта, который использует базу данных SQL Azure: XXX.database.windows.net. После развертывания веб-службы необходимо разрешить потребителям веб-службы указать другой логический сервер SQL Server с именем `YYY.database.windows.net` . Вы можете использовать параметр веб-службы, чтобы разрешить настройку этого значения.

> [!NOTE]
> Входные и выходные данные веб-службы отличаются от параметров веб-службы. В первом сценарии вы узнали, как можно указать входные и выходные данные для веб-службы Машинное обучение Azure Studio (классическая модель). В этом сценарии вы передаете параметры для веб-службы, соответствующие свойствам модулей импорта и вывода данных.

Рассмотрим сценарий использования параметров веб-службы. У вас есть развернутая веб-служба Машинное обучение Azure Studio (классическая), которая использует модуль чтения для чтения данных из одного из источников данных, поддерживаемых Машинное обучение Azure Studio (классическая модель) (например, база данных SQL Azure). После пакетного выполнения результаты записываются с помощью модуля записи (база данных SQL Azure).  В экспериментах не определены входные и выходные данные веб-службы. В этом случае мы советуем настроить соответствующие параметры веб-службы для модулей чтения и записи. Это позволит настроить их при использовании действия AzureMLBatchExecution. Параметры веб-службы указываются в разделе **globalParameters** действия JSON, как показано ниже.

```JSON
"typeProperties": {
    "globalParameters": {
        "Database server name": "<myserver>.database.windows.net",
        "Database name": "<database>",
        "Server user account name": "<user name>",
        "Server user account password": "<password>"
    }
}
```

> [!NOTE]
> В параметрах веб-службы учитывается регистр, поэтому убедитесь, что имена, которые указываются в действии JSON, соответствуют именам, предоставляемым веб-службой.

После завершения повторного обучения обновите веб-службу оценки (прогнозный эксперимент, представленный в виде веб-службы) с использованием новой обученной модели с помощью **действия ресурса обновления машинное обучение Azure Studio (классическая модель)**. Дополнительные сведения см. в статье [Updating Azure Machine Learning models using Update Resource Activity](update-machine-learning-models.md) (Обновление моделей машинного обучения Azure с помощью действия "Обновить ресурс").

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами:

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)