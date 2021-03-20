---
title: Обновление моделей Машинное обучение Azure Studio (классическая модель) с помощью фабрики данных Azure
description: Описывает создание прогнозирующих конвейеров с помощью фабрики данных Azure и Машинное обучение Azure Studio (классическая модель).
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: ef89ecef34a7c5afb94547181f449b0fc393e67c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100377573"
---
# <a name="update-azure-machine-learning-studio-classic-models-by-using-update-resource-activity"></a>Обновление моделей Машинное обучение Azure Studio (классическая модель) с помощью действия "обновить ресурс"

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Эта статья дополняет основную статью фабрики данных Azure — Машинное обучение Azure Studio (классическая модель): [Создание прогнозирующих конвейеров с помощью машинное обучение Azure Studio (классическая модель) и фабрики данных Azure](transform-data-using-machine-learning.md). Перед прочтением этой статьи ознакомьтесь с основной статьей, если вы еще этого не сделали.

## <a name="overview"></a>Обзор
В рамках процесса моделей эксплуатацию Машинное обучение Azure Studio (классические модели) модель обучена и сохранена. После этого она используется для создания прогнозной веб-службы. Созданная веб-служба может впоследствии использоваться на веб-сайтах, панелях мониторинга и в мобильных приложениях.

Модели, создаваемые с помощью Машинное обучение Azure Studio (классическая модель), обычно не являются статическими. Если появляются новые данные или у пользователя API есть свои данные, модель нужно переобучить. 

Переобучение может требоваться достаточно часто. С действием пакетного выполнения и действиями обновления ресурсов можно эксплуатацию повторное обучение модели Машинное обучение Azure Studio (классическая модель) и обновление прогнозной веб-службы с помощью фабрики данных.

На следующем рисунке показа связь между обучением и прогнозными веб-службами.

![Веб-службы](./media/update-machine-learning-models/web-services.png)

## <a name="azure-machine-learning-studio-classic-update-resource-activity"></a>Действие "обновить ресурс" Машинное обучение Azure Studio (классическая модель)

В следующем фрагменте кода JSON определяется действие выполнения пакета Машинное обучение Azure Studio (классическая модель).

```json
{
    "name": "amlUpdateResource",
    "type": "AzureMLUpdateResource",
    "description": "description",
    "linkedServiceName": {
        "type": "LinkedServiceReference",
        "referenceName": "updatableScoringEndpoint2"
    },
    "typeProperties": {
        "trainedModelName": "ModelName",
        "trainedModelLinkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "StorageLinkedService"
                },
        "trainedModelFilePath": "ilearner file path"
    }
}
```

| Свойство.                      | Описание                              | Обязательно |
| :---------------------------- | :--------------------------------------- | :------- |
| name                          | Имя действия в конвейере.     | Да      |
| description                   | Описание действия.  | Нет       |
| type                          | Для действия "обновить ресурс" Машинное обучение Azure Studio (классическая модель) тип действия —  **AzureMLUpdateResource**. | Да      |
| linkedServiceName             | Связанная служба Машинное обучение Azure Studio (классическая), содержащая свойство updateResourceEndpoint. | Да      |
| trainedModelName              | Имя модуля модели обучения для обновления в эксперименте веб-службы. | Да      |
| trainedModelLinkedServiceName | Имя связанной службы хранилища Azure, содержащей файл iLearner, который был отправлен действием обновления. | Да      |
| trainedModelFilePath          | Относительный путь к файлу в trainedModelLinkedService для представления файла iLearner, который отправлен действием обновления. | Да      |

## <a name="end-to-end-workflow"></a>Комплексный рабочий процесс

Процесс переобучения модели и обновления прогнозных веб-служб включает в себя такие шаги:

- Вызов **веб-службы обучения** с помощью **действия выполнения пакета**. Вызов веб-службы обучения аналогичен вызову прогнозной веб-службы, описанной в статье [Создание прогнозных конвейеров с помощью машинное обучение Azure Studio (классическая модель) и действие выполнения пакета фабрики данных](transform-data-using-machine-learning.md). Выходные данные обучающей веб-службы — это файл iLearner, который можно использовать для обновления прогнозной веб-службы.
- Вызов **обновления конечной точки ресурса****прогнозной веб-службы** с помощью **действия обновления ресурса**, чтобы обновить веб-службу и добавить новую обученную модель.

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Связанная служба Машинное обучение Azure Studio (классическая модель)

Для работы описанного выше рабочего процесса необходимо создать две Машинное обучение Azure Studio (классические) связанные службы:

1. Связанная служба Машинное обучение Azure Studio (классическая модель) к веб-службе для обучения, эта связанная служба используется действием пакетной обработки так же, как описано в разделе [Создание прогнозных конвейеров с помощью машинное обучение Azure Studio (классическая модель) и действие выполнения пакета фабрики данных](transform-data-using-machine-learning.md). Различие — это выходная часть обучающей веб-службы — файл iLearner, который затем используется действием "обновить ресурс" для обновления прогнозной веб-службы.
2. Связанная служба Машинное обучение Azure Studio (классическая) к конечной точке обновления ресурса прогнозной веб-службы. Эта связанная служба используется действием обновления ресурса для обновления прогнозной веб-службы с помощью файла iLearner, возвращенного на предыдущем шаге.

Для второй связанной службы Машинное обучение Azure Studio (классическая модель) конфигурация отличается, если веб-служба Машинное обучение Azure Studio (классическая) является классической веб-службой или новой веб-службой. Различия отдельно рассматриваются в следующих разделах.

## <a name="web-service-is-new-azure-resource-manager-web-service"></a>Веб-служба — новая веб-служба Azure Resource Manager

Если веб-служба является новым типом веб-службы и предоставляет конечную точку Azure Resource Manager, то необходимо добавить вторую конечную точку **не по умолчанию**. Конечная точка **updateResourceEndpoint** в связанной службе имеет такой формат:

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview
```

При запросе веб-службы на [портале веб-служб машинное обучение Azure Studio (классическая модель)](https://services.azureml.net/)можно получить значения для владельцев мест в URL-адресе.

Новому типу конечной точки ресурса для обновления необходима проверка подлинности субъекта-службы. Чтобы использовать проверку подлинности субъекта-службы, зарегистрируйте сущность приложения в Azure Active Directory (Azure AD) и предоставьте ей роль **участника** или **владельца** подписки или группы ресурсов, которой принадлежит веб-служба. Ознакомьтесь со статьей [Создание приложения Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../active-directory/develop/howto-create-service-principal-portal.md). Запишите следующие значения, которые используются для определения связанной службы:

- Идентификатор приложения
- Ключ приложения
- Tenant ID

Ниже приведен пример определения связанной службы:

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000  000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": {
                "type": "SecureString",
                "value": "APIKeyOfEndpoint1"
            },
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "servicePrincipalKey"
            },
            "tenant": "mycompany.com"
        }
    }
}
```

Ниже приведен сценарий с более подробными сведениями. В нем приведен пример повторного обучения и обновления моделей Машинное обучение Azure Studio (классическая модель) из конвейера фабрики данных Azure.


## <a name="sample-retraining-and-updating-an-azure-machine-learning-studio-classic-model"></a>Пример: повторное обучение и обновление модели Машинное обучение Azure Studio (классическая модель)

В этом разделе приведен пример конвейера, который использует **действие выполнения пакета машинное обучение Azure Studio (классическая модель)** для повторного обучения модели. Конвейер также использует действие " **обновить ресурс" машинное обучение Azure Studio (классическая модель)** для обновления модели в веб-службе оценки. Здесь также приведены фрагменты JSON для всех связанных служб, наборов данных и конвейера, которые используются в примере.

### <a name="azure-blob-storage-linked-service"></a>Связанная служба хранилища BLOB-объектов Azure
Служба хранилища Azure содержит следующие данные:

* Данные для обучения. Входные данные для веб-службы обучения Машинное обучение Azure Studio (классическая модель).
* Файл iLearner. Выходные данные веб-службы обучения Машинное обучение Azure Studio (классическая модель). Этот файл также служит в качестве входных данных для действия "Обновить ресурс".

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

### <a name="linked-service-for-azure-machine-learning-studio-classic-training-endpoint"></a>Связанная служба для конечной точки обучения Машинное обучение Azure Studio (классическая модель)
В следующем фрагменте кода JSON определяется связанная служба Машинное обучение Azure Studio (классическая), которая указывает на конечную точку по умолчанию для веб-службы обучения.

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

### <a name="linked-service-for-azure-machine-learning-studio-classic-updatable-scoring-endpoint"></a>Связанная служба для обновляемой конечной точки оценки Машинное обучение Azure Studio (классическая модель):
В следующем фрагменте кода JSON определяется связанная служба Машинное обучение Azure Studio (классическая), которая указывает на обновляемую конечную точку веб-службы оценки.

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

### <a name="pipeline"></a>Конвейер
Конвейер содержит два действия: **AzureMLBatchExecution** и **AzureMLUpdateResource**. Действие выполнения пакета принимает входные данные для обучения и создает выходной файл iLearner. Действие ресурса обновления принимает этот файл iLearner и использует его для обновления прогнозной веб-службы.

```JSON
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "amlBEGetilearner",
                "description": "Use AML BES to get the ileaner file from training web service",
                "type": "AzureMLBatchExecution",
                "linkedServiceName": {
                    "referenceName": "trainingEndpoint",
                    "type": "LinkedServiceReference"
                },
                "typeProperties": {
                    "webServiceInputs": {
                        "input1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        },
                        "input2": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        }
                    },
                    "webServiceOutputs": {
                        "output1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/output"
                        }
                    }
                }
            },
            {
                "name": "amlUpdateResource",
                "type": "AzureMLUpdateResource",
                "description": "Use AML Update Resource to update the predict web service",
                "linkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "updatableScoringEndpoint2"
                },
                "typeProperties": {
                    "trainedModelName": "ADFV2Sample Model [trained model]",
                    "trainedModelLinkedServiceName": {
                        "type": "LinkedServiceReference",
                        "referenceName": "StorageLinkedService"
                    },
                    "trainedModelFilePath": "azuremltesting/output/newModelForArm.ilearner"
                },
                "dependsOn": [
                    {
                        "activity": "amlbeGetilearner",
                        "dependencyConditions": [ "Succeeded" ]
                    }
                ]
            }
        ]
    }
}
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
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)
