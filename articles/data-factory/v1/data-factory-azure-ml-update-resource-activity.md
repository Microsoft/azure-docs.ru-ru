---
title: Обновление моделей Машинное обучение с помощью фабрики данных Azure
description: Описывает создание прогнозирующих конвейеров с помощью фабрики данных Azure версии 1 и Машинное обучение Azure Studio (классическая модель).
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: 2d9156bfcc11817a647c33053a2d04c653c0a706
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104785501"
---
# <a name="updating-azure-machine-learning-studio-classic-models-using-update-resource-activity"></a>Обновление моделей Машинное обучение Azure Studio (классическая модель) с помощью действия "обновить ресурс"

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


> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью об [обновлении моделей машинного обучения с помощью службы "Фабрика данных"](../update-machine-learning-models.md).

Эта статья дополняет основную статью фабрики данных Azure — Машинное обучение Azure Studio (классическая модель): [Создание прогнозирующих конвейеров с помощью машинное обучение Azure Studio (классическая модель) и фабрики данных Azure](data-factory-azure-ml-batch-execution-activity.md). Перед прочтением этой статьи ознакомьтесь с основной статьей, если вы еще этого не сделали. 

## <a name="overview"></a>Обзор
Со временем прогнозные модели в экспериментах оценки Машинное обучение Azure Studio (классическая модель) необходимо переучить с помощью новых входных наборов данных. Когда повторное обучение будет завершено, вам потребуется обновить веб-службу оценки на основании обновленной модели машинного обучения. Типичные действия по включению повторного обучения и обновления моделей Studio (классическая модель) с помощью веб-служб:

1. Создайте эксперимент в [машинное обучение Azure Studio (классическая модель)](https://studio.azureml.net).
2. Если вы удовлетворены моделью, используйте Машинное обучение Azure Studio (классическая модель), чтобы опубликовать веб-службы как для **обучающего** , так и для оценки,**прогнозного эксперимента**.

В следующей таблице описаны веб-службы, используемые в данном примере.  Дополнительные сведения см. в разделе [программное переобучение моделей машинное обучение Azure Studio (классическая модель)](../../machine-learning/classic/retrain-machine-learning-model.md) .

- **Веб-служба обучения**: получает данные для обучения и создает обученные модели. Результат повторного обучения — файл формата ILEARNER в хранилище BLOB-объектов Azure. При публикации обучающего эксперимента в виде веб-службы автоматически создается **конечная точка по умолчанию** . Вы можете создать несколько конечных точек, но в этом примере используется только конечная точка по умолчанию.
- **Веб-служба оценки**: получает данные без метки и осуществляет прогнозирование. Выходные данные прогноза могут иметь различные формы, например CSV-файл или строки в базе данных SQL Azure, в зависимости от конфигурации эксперимента. При публикации прогнозного эксперимента в виде веб-службы автоматически создается конечная точка по умолчанию. 

На следующем рисунке показана связь между конечными точками обучения и оценки в Машинное обучение Azure Studio (классическая модель).

![Веб-службы](./media/data-factory-azure-ml-batch-execution-activity/web-services.png)

**Веб-службу обучения** можно вызвать с помощью **действия выполнения пакета машинное обучение Azure Studio (классическая модель)**. Вызов веб-службы обучения осуществляется так же, как и вызов веб-службы Машинное обучение Azure Studio (классическая модель) (оценка веб-службы) для оценки данных. В предыдущих разделах подробно описано, как вызвать веб-службу Машинное обучение Azure Studio (классическая модель) из конвейера фабрики данных Azure. 

Вы можете вызвать **веб-службу оценки** с помощью **действия "обновить ресурс" машинное обучение Azure Studio (классическая модель)** , чтобы обновить веб-службу с использованием новой обученной модели. Ниже приведены примеры определений связанной службы. 

## <a name="scoring-web-service-is-a-classic-web-service"></a>Веб-служба оценки — классическая веб-служба
Если веб-служба оценки является **классической веб-службой**, создайте вторую, **не используемую по умолчанию и обновляемую конечную точку** , используя портал Azure. Инструкции см. в статье [Создание конечных точек](../../machine-learning/classic/create-endpoint.md). Создав обновляемую конечную точку не по умолчанию, выполните следующие действия:

* Щелкните **Выполнение пакета**, чтобы получить значение URI свойства JSON **mlEndpoint**.
* Затем щелкните **Обновить ресурс**, чтобы получить значение URI свойства JSON **updateResourceEndpoint**. Ключ API указывается на странице конечной точки (в правом нижнем углу).

![обновляемая конечная точка](./media/data-factory-azure-ml-batch-execution-activity/updatable-endpoint.png)

В следующем примере представлен вариант определения JSON для связанной службы AzureML. Связанная служба использует для аутентификации ключ apiKey.  

```json
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--scoring experiment--/jobs",
            "apiKey": "endpoint2Key",
            "updateResourceEndpoint": "https://management.azureml.net/workspaces/xxx/webservices/--scoring experiment--/endpoints/endpoint2"
        }
    }
}
```

## <a name="scoring-web-service-is-azure-resource-manager-web-service"></a>Веб-служба оценки — веб-служба Azure Resource Manager 
Если веб-служба является новым типом веб-службы и предоставляет конечную точку Azure Resource Manager, то необходимо добавить вторую конечную точку **не по умолчанию**. Конечная точка **updateResourceEndpoint** в связанной службе имеет такой формат: 

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview. 
```

При запросе веб-службы на [портале веб-служб машинное обучение Azure Studio (классическая модель)](https://services.azureml.net/)можно получить значения для владельцев мест в URL-адресе. Для нового типа конечной точки обновления ресурса требуется маркер AAD (Azure Active Directory). Укажите **servicePrincipalId** и **сервицепринЦипалкэй** в связанной службе Studio (классическая модель). Ознакомьтесь со статьей [Создание приложения Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../../active-directory/develop/howto-create-service-principal-portal.md). Ниже приведен пример определения связанной службы AzureML. 

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": "xxxxxxxxxxxx",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": "xxxxx",
            "tenant": "mycompany.com"
        }
    }
}
```

Ниже приведен сценарий с более подробными сведениями. В нем приведен пример повторного обучения и обновления моделей Studio (классической модели) из конвейера фабрики данных Azure.

## <a name="scenario-retraining-and-updating-a-studio-classic-model"></a>Сценарий: повторное обучение и обновление модели Studio (классическая модель)
В этом разделе приведен пример конвейера, который использует **действие выполнения пакета машинное обучение Azure Studio (классическая модель)** для повторного обучения модели. Конвейер также использует действие " **обновить ресурс" машинное обучение Azure Studio (классическая модель)** для обновления модели в веб-службе оценки. Здесь также приведены фрагменты JSON для всех связанных служб, наборов данных и конвейера, которые используются в примере.

Пример конвейера представлен на схеме. Как видите, действие выполнения пакета Studio (классическая модель) принимает входные данные для обучения и создает выходные данные для обучения (файл iLearner). Действие "обновить ресурс" в студии (классическая модель) принимает эти обучающие данные и обновляет ее в конечной точке веб-службы оценки. Действие «Обновить ресурс» не создает никаких выходных данных. Набор данных placeholderBlob является фиктивным набором данных, который необходим фабрике данных Azure для запуска конвейера.

![схема конвейера](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

### <a name="azure-blob-storage-linked-service"></a>Связанная служба хранилища BLOB-объектов Azure
Служба хранилища Azure содержит следующие данные:

* Данные для обучения. Входные данные для веб-службы обучения студии (классическая модель).  
* Файл iLearner. Выходные данные веб-службы обучения Studio (классическая модель). Этот файл также служит в качестве входных данных для действия "Обновить ресурс".  

Вот пример определения JSON для связанной службы:

```JSON
{
    "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=name;AccountKey=key"
        }
    }
}
```

### <a name="training-input-dataset"></a>Входной набор данных для обучения:
Следующий набор данных представляет входные данные для обучения для веб-службы обучения Studio (классической). Действие выполнения пакета Studio (классическая модель) принимает этот набор данных в качестве входных данных.

```JSON
{
    "name": "trainingData",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "labeledexamples",
            "fileName": "labeledexamples.arff",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
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

### <a name="training-output-dataset"></a>Выходной набор данных для обучения:
Следующий набор данных представляет выходной файл iLearner из веб-службы обучения Машинное обучение Azure Studio (классической). Действие выполнения пакета Машинное обучение Azure Studio (классическая модель) создает этот набор данных. Этот набор данных также является входными данными для действия "обновить ресурс Машинное обучение Azure Studio (классическая модель)".

```JSON
{
    "name": "trainedModelBlob",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "trainingoutput",
            "fileName": "model.ilearner",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
            "interval": 1
        }
    }
}
```

### <a name="linked-service-for-studio-classic-training-endpoint"></a>Связанная служба для конечной точки обучения Studio (классическая модель)
В следующем фрагменте кода JSON определяется связанная служба Studio (классическая), которая указывает на конечную точку по умолчанию для веб-службы обучения.

```JSON
{    
    "name": "trainingEndpoint",
      "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--training experiment--/jobs",
              "apiKey": "myKey"
        }
      }
}
```

В **машинное обучение Azure Studio (классическая модель)** выполните следующие действия, чтобы получить значения для **mlEndpoint** и **apiKey**:

1. В меню слева щелкните **ВЕБ-СЛУЖБЫ** .
2. В списке веб-служб выберите **веб-службу обучения** .
3. Рядом с полем **Ключ API** нажмите кнопку копирования. Вставьте ключ из буфера обмена в редактор JSON фабрики данных.
4. В **машинное обучение Azure Studio (классическая модель)** щелкните ссылку **выполнение пакета** .
5. Скопируйте **URI запроса** из раздела **Запрос** и вставьте его в редактор JSON фабрики данных.   

### <a name="linked-service-for-studio-classic-updatable-scoring-endpoint"></a>Связанная служба для обновляемой конечной точки Studio (классическая модель):
В следующем фрагменте кода JSON определяется связанная служба Studio (классическая модель), которая указывает на конечную точку, которая не является обновляемой по умолчанию для веб-службы оценки.  

```JSON
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/00000000eb0abe4d6bbb1d7886062747d7/services/00000000026734a5889e02fbb1f65cefd/jobs?api-version=2.0",
            "apiKey": "sooooooooooh3WvG1hBfKS2BNNcfwSO7hhY6dY98noLfOdqQydYDIXyf2KoIaN3JpALu/AKtflHWMOCuicm/Q==",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "fe200044-c008-4008-a005-94000000731",
            "servicePrincipalKey": "zWa0000000000Tp6FjtZOspK/WMA2tQ08c8U+gZRBlw=",
            "tenant": "mycompany.com"
        }
    }
}
```

### <a name="placeholder-output-dataset"></a>Заполнитель выходного набора данных
Действие "обновить ресурс" в студии (классическая модель) не создает никаких выходных данных. Однако фабрике данных Azure требуется выходной набор данных для планирования конвейера. Поэтому в этом примере используется фиктивный набор данных (заполнитель набора данных).  

```JSON
{
    "name": "placeholderBlob",
    "properties": {
        "availability": {
            "frequency": "Week",
            "interval": 1
        },
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "any",
            "format": {
                "type": "TextFormat"
            }
        }
    }
}
```

### <a name="pipeline"></a>Конвейер
Конвейер содержит два действия: **AzureMLBatchExecution** и **AzureMLUpdateResource**. Действие выполнения пакета Машинное обучение Azure Studio (классическая модель) принимает обучающие данные в качестве входных данных и создает файл iLearner в качестве выходных данных. Это действие обращается к веб-службе обучения (обучающему эксперименту, опубликованному в виде веб-службы), передает ей данные для обучения и получает файл ilearner. Набор данных placeholderBlob является фиктивным набором данных, который необходим фабрике данных Azure для запуска конвейера.

![схема конвейера](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }              
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "Training Exp for ADF ML [trained model]",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "outputs": [
                    {
                        "name": "placeholderBlob"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00Z",
           "end": "2016-02-14T00:00:00Z"
    }
}
```