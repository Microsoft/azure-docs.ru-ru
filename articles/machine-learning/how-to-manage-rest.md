---
title: Использование RESTFUL для управления ресурсами машинного обучения
titleSuffix: Azure Machine Learning
description: Использование интерфейсов API для создания, запуска и удаления ресурсов Машинное обучение Azure, таких как Рабочая область или регистрация моделей.
author: lobrien
ms.author: laobri
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 01/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: b61050db29ff960b7923f2211b2f09649608b37e
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520325"
---
# <a name="create-run-and-delete-azure-ml-resources-using-rest"></a>Создание, запуск и удаление ресурсов машинного обучения Azure с помощью функции "ОСТАВШАЯся"



Существует несколько способов управления ресурсами машинного обучения Azure. Можно использовать [портал](https://portal.azure.com/), [интерфейс командной строки](/cli/azure)или [пакет SDK для Python](/python/api/overview/azure/ml/intro). Также можно выбрать REST API. REST API использует глаголы HTTP для создания, извлечения, обновления и удаления ресурсов. REST API работает с любым языком или инструментом, который может выполнять HTTP-запросы. Простая структура RESTFUL часто делает ее хорошим выбором в средах создания сценариев и для автоматизации Млопс. 

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Получение маркера авторизации
> * Создание запроса на языке ОСТАВШЕЙся в правильном формате с помощью проверки подлинности субъекта-службы
> * Использование запросов GET для получения сведений о иерархических ресурсах машинного обучения Azure
> * Использование запросов размещения и POST для создания и изменения ресурсов
> * Использование запросов на удаление для очистки ресурсов 
> * Использование авторизации на основе ключей для оценки развернутых моделей

## <a name="prerequisites"></a>Предварительные условия

- **Подписка Azure** , для которой у вас есть права администратора. Если у вас нет такой подписки, попробуйте использовать [бесплатную или платную личную подписку](https://aka.ms/AMLFree) .
- [Рабочая область машинного обучения Azure](./how-to-manage-workspace.md)
- Запросы на администрирование для других пользователей используют проверку подлинности субъекта-службы. Выполните действия, описанные в разделе [Настройка проверки подлинности для машинное обучение Azure ресурсов и рабочих процессов](./how-to-setup-authentication.md#service-principal-authentication) , чтобы создать субъект-службу в рабочей области.
- Служебная программа для **фигур** . Эта **программа доступна** в [подсистеме Windows для Linux](/windows/wsl/install-win10) или любого дистрибутива UNIX. В PowerShell **фигурный** псевдоним для **Invoke-WebRequest** и `curl -d "key=val" -X POST uri` становится `Invoke-WebRequest -Body "key=val" -Method POST -Uri uri` . 

## <a name="retrieve-a-service-principal-authentication-token"></a>Получение маркера проверки подлинности субъекта-службы

Запросы на администрирование для всех пользователей проходят проверку подлинности с помощью неявного потока OAuth2. Этот поток проверки подлинности использует маркер, предоставленный субъектом-службой вашей подписки. Чтобы получить этот маркер, вам потребуется:

- Идентификатор клиента (определяющий организацию, к которой принадлежит подписка);
- Идентификатор клиента (который будет связан с созданным токеном)
- Секрет клиента (который следует защитить)

Эти значения должны быть предоставлены из ответа на создание субъекта-службы. Эти значения обсуждаются в разделе [Настройка проверки подлинности для машинное обучение Azureных ресурсов и рабочих процессов](./how-to-setup-authentication.md#service-principal-authentication). Если вы используете подписку компании, возможно, у вас нет разрешения на создание субъекта-службы. В этом случае следует использовать [бесплатную или платную личную подписку](https://aka.ms/AMLFree).

Получение токена:

1. Откройте окно терминала.
1. Введите следующий код в командной строке
1. Подставьте собственные значения для `{your-tenant-id}` , `{your-client-id}` и `{your-client-secret}` . В этой статье строки, заключенные в фигурные скобки, являются переменными, которые нужно заменить собственными соответствующими значениями.
1. Выполните команду следующую команду.

```bash
curl -X POST https://login.microsoftonline.com/{your-tenant-id}/oauth2/token \
-d "grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.azure.com%2F&client_id={your-client-id}&client_secret={your-client-secret}" \
```

Ответ должен предоставить маркер доступа в течение одного часа:

```json
{
    "token_type": "Bearer",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1578523094",
    "not_before": "1578519194",
    "resource": "https://management.azure.com/",
    "access_token": "your-access-token"
}
```

Запишите маркер, так как он будет использоваться для проверки подлинности всех последующих административных запросов. Это можно сделать, задав заголовок Authorization во всех запросах:

```bash
curl -h "Authorization:Bearer {your-access-token}" ...more args...
```

Обратите внимание, что значение начинается со строки "Bearer", включающей в себя один пробел перед добавлением маркера.

## <a name="get-a-list-of-resource-groups-associated-with-your-subscription"></a>Получение списка групп ресурсов, связанных с подпиской

Чтобы получить список групп ресурсов, связанных с подпиской, выполните:

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups?api-version=2019-11-01 -H "Authorization:Bearer {your-access-token}"
```

В Azure публикуются многие API-интерфейсы RESTFUL. Каждый поставщик услуг обновляет свой API с учетом их собственной ритмичности, но делает это без нарушения работы существующих программ. Поставщик услуг использует `api-version` аргумент для обеспечения совместимости. `api-version`Аргумент отличается от службы к службе. Например, для службы Машинное обучение Текущая версия API — `2019-11-01` . Для учетных записей хранения это `2019-06-01` . Для хранилищ ключей это `2019-09-01` . Все вызовы RESTFUL должны задать `api-version` для аргумента ожидаемое значение. Вы можете полагаться на синтаксис и семантику указанной версии, даже если API продолжит развиваться. Если отправить запрос поставщику без `api-version` аргумента, ответ будет содержать доступный для человека список поддерживаемых значений. 

Приведенный выше вызов приведет к сжатию ответа JSON в форме: 

```json
{
    "value": [
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG1",
            "name": "RG1",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "westus2",
            "properties": {
                "provisioningState": "Succeeded"
            }
        },
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG2",
            "name": "RG2",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "eastus",
            "properties": {
                "provisioningState": "Succeeded"
            }
        }
    ]
}
```


## <a name="drill-down-into-workspaces-and-their-resources"></a>Детализация рабочих областей и их ресурсов

Чтобы получить набор рабочих областей в группе ресурсов, выполните следующую команду, подставив `{your-subscription-id}` , `{your-resource-group}` и `{your-access-token}` : 

```
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Опять же, вы получите список JSON, на этот раз содержащий список, каждый элемент которого содержит сведения о рабочей области:

```json
{
    "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/DeepLearningResourceGroup/providers/Microsoft.MachineLearningServices/workspaces/my-workspace",
    "name": "my-workspace",
    "type": "Microsoft.MachineLearningServices/workspaces",
    "location": "centralus",
    "tags": {},
    "etag": null,
    "properties": {
        "friendlyName": "",
        "description": "",
        "creationTime": "2020-01-03T19:56:09.7588299+00:00",
        "storageAccount": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.storage/storageaccounts/myworkspace0275623111",
        "containerRegistry": null,
        "keyVault": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.keyvault/vaults/myworkspace2525649324",
        "applicationInsights": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.insights/components/myworkspace2053523719",
        "hbiWorkspace": false,
        "workspaceId": "cba12345-abab-abab-abab-ababab123456",
        "subscriptionState": null,
        "subscriptionStatusChangeTimeStampUtc": null,
        "discoveryUrl": "https://centralus.experiments.azureml.net/discovery"
    },
    "identity": {
        "type": "SystemAssigned",
        "principalId": "abcdef1-abab-1234-1234-abababab123456",
        "tenantId": "1fedcba-abab-1234-1234-abababab123456"
    },
    "sku": {
        "name": "Basic",
        "tier": "Basic"
    }
}
```

Для работы с ресурсами в рабочей области необходимо переключиться с общего сервера **Management.Azure.com** на REST API сервер, относящийся к расположению рабочей области. Запишите значение `discoveryUrl` ключа в приведенном выше ответе JSON. Если вы получили этот URL-адрес, вы получите ответ примерно следующего вида:

```json
{
  "api": "https://centralus.api.azureml.ms",
  "catalog": "https://catalog.cortanaanalytics.com",
  "experimentation": "https://centralus.experiments.azureml.net",
  "gallery": "https://gallery.cortanaintelligence.com/project",
  "history": "https://centralus.experiments.azureml.net",
  "hyperdrive": "https://centralus.experiments.azureml.net",
  "labeling": "https://centralus.experiments.azureml.net",
  "modelmanagement": "https://centralus.modelmanagement.azureml.net",
  "pipelines": "https://centralus.aether.ms",
  "studiocoreservices": "https://centralus.studioservice.azureml.com"
}
```

Значение `api` ответа — это URL-адрес сервера, который будет использоваться для дополнительных запросов. Например, чтобы перечислить эксперименты, отправьте следующую команду. Замените на `regional-api-server` значение `api` ответа (например, `centralus.api.azureml.ms` ). Также замените `your-subscription-id` , `your-resource-group` , `your-workspace-name` и `your-access-token` как обычно:

```bash
curl https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Аналогично, чтобы получить зарегистрированные модели в рабочей области, отправьте:

```bash
curl https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Обратите внимание, что для перечисления экспериментов путь начинается с `history/v1.0` , когда для вывода списка моделей путь начинается с `modelmanagement/v1.0` . REST API разделена на несколько операционных групп, каждый из которых имеет отдельный путь. 

|Область|Путь|
|-|-|
|Artifacts|/рест/апи/азуремл|
|Хранилища данных|/азуре/мачине-леарнинг/хов-то-акцесс-дата|
|Настройка гиперпараметров|устройство/v 1.0/|
|Модели|моделманажемент/v 1.0/|
|Журнал выполнения|выполнение/v 1.0/и журнал/версия 1.0/|

Вы можете исследовать REST API, используя общий шаблон:

|Компонент URL-адреса|Например, .|
|-|-|
| https://| |
| региональные API — сервер/ | centralus.api.azureml.ms/ |
| Operations-Path/ | Журнал/версия 1.0/ |
| Subscriptions/{ваша-Subscription-ID}/ | Subscriptions/abcde123-abab-abab-1234-0123456789abc/ |
| resourceGroups/{ваша-Resource-Group}/ | resourceGroups/MyResourceGroup/ |
| Providers/Operation-provider/ | Providers/Microsoft. Мачинелеарнингсервицес/ |
| Поставщик-ресурс-путь/ | рабочие области/Млворкспаце/MyWorkspace/Фирстексперимент/запуски/1/ |
| Operations — Endpoint/ | артефакты/метаданные/ |


## <a name="create-and-modify-resources-using-put-and-post-requests"></a>Создание и изменение ресурсов с помощью запросов размещения и POST

В дополнение к извлечению ресурсов с помощью команды GET, REST API поддерживает создание всех ресурсов, необходимых для обучения, развертывания и мониторинга решений машинного обучения. 

Для обучения и выполнения моделей ML требуются ресурсы вычислений. Вы можете вывести список ресурсов рабочей области с помощью: 

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Для создания или перезаписи именованного ресурса используется запрос на размещение. В следующем примере в дополнение к знакомым подстановкам `your-subscription-id` , `your-resource-group` , `your-workspace-name` и `your-access-token` , замените `your-compute-name` и значениями для `location` , `vmSize` , `vmPriority` ,, `scaleSettings` `adminUserName` и `adminUserPassword` . В соответствии с указаниями, указанными в справочнике по [ссылке вычислительная среда машинного обучения-CREATE или Update SDK](/rest/api/azureml/workspacesandcomputes/machinelearningcompute/createorupdate), следующая команда создает выделенный Standard_D1 с одним узлом (Базовый вычислительный ресурс ЦП), который будет масштабироваться через 30 минут:

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes/{your-compute-name}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{your-azure-location}",
    "properties": {
        "computeType": "AmlCompute",
        "properties": {
            "vmSize": "Standard_D1",
            "vmPriority": "Dedicated",
            "scaleSettings": {
                "maxNodeCount": 1,
                "minNodeCount": 0,
                "nodeIdleTimeBeforeScaleDown": "PT30M"
            }
        }
    },
    "userAccountCredentials": {
        "adminUserName": "{adminUserName}",
        "adminUserPassword": "{adminUserPassword}"
    }
}'
```

> [!Note]
> В терминалах Windows может потребоваться экранирование символов двойной кавычки при отправке данных JSON. То есть текст, например, `"location"` становится `\"location\"` . 

Успешный запрос получит `201 Created` ответ, но обратите внимание, что этот ответ просто означает, что процесс подготовки начался. Чтобы подтвердить успешность выполнения, необходимо опросить (или использовать портал).

### <a name="create-an-experimental-run"></a>Создание экспериментального запуска

Чтобы запустить выполнение в эксперименте, требуется папка ZIP, содержащая сценарий обучения и связанные файлы, а также файл JSON определения запуска. Папка ZIP должна содержать файл записи Python в корневом каталоге. В качестве примера заархивировать тривиальные программы Python, такие как следующий, в папку с именем **train.zip**.

```python
# hello.py
# Entry file for run
print("Hello, REST!")
```

Сохраните следующий фрагмент кода как **definition.jsв**. Подтвердите, что значение "script" соответствует имени файла Python, который вы только что заархивированы. Подтвердите, что значение "Target" соответствует имени доступного ресурса вычислений. 

```json
{
    "Configuration":{  
       "Script":"hello.py",
       "Arguments":[  
          "234"
       ],
       "SourceDirectoryDataStore":null,
       "Framework":"Python",
       "Communicator":"None",
       "Target":"cpu-compute",
       "MaxRunDurationSeconds":1200,
       "NodeCount":1,
       "Environment":{  
          "Python":{  
             "InterpreterPath":"python",
             "UserManagedDependencies":false,
             "CondaDependencies":{  
                "name":"project_environment",
                "dependencies":[  
                   "python=3.6.2",
                   {  
                      "pip":[  
                         "azureml-defaults"
                      ]
                   }
                ]
             }
          },
          "Docker":{  
             "BaseImage":"mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
          }
      },
       "History":{  
          "OutputCollection":true
       }
    }
}
```

Опубликовать эти файлы на сервере с помощью `multipart/form-data` содержимого:

```bash
curl https://{regional-api-server}/execution/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-name}/startrun?api-version=2019-11-01 \
  -X POST \
  -H "Content-Type: multipart/form-data" \
  -H "Authorization:Bearer {your-access-token}" \
  -F projectZipFile=@train.zip \
  -F runDefinitionFile=@runDefinition.json
```

Успешный запрос POST приведет к формированию `200 OK` состояния с текстом ответа, содержащим идентификатор созданного запуска:

```json
{
  "runId": "my-first-experiment_1579642222877"
}
```

Вы можете отслеживать выполнение с помощью шаблона развитых, который теперь должен быть знаком:

```bash
curl 'https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-names}/runs/{your-run-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}'
```

### <a name="delete-resources-you-no-longer-need"></a>Удаление ресурсов, которые больше не нужны

Некоторые, но не все ресурсы поддерживают команду DELETE. Проверьте [ссылку на API](/rest/api/azureml/) перед фиксацией в REST API для удаления вариантов использования. Чтобы удалить модель, например, можно использовать:

```bash
curl
  -X DELETE \
'https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models/{your-model-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' 
```

## <a name="use-rest-to-score-a-deployed-model"></a>Использование ОСТАВШЕЙся для оценки развернутой модели

Хотя можно развернуть модель, чтобы она выполняла проверку подлинности с субъектом-службой, большинство клиентских развертываний используют проверку подлинности на основе ключей. Соответствующий ключ можно найти на странице развертывания на вкладке " **конечные точки** " в студии. В том же расположении будет показан URI оценки конечной точки. Входные данные модели должны быть смоделированы как массив JSON с именем `data` :

```bash
curl 'https://{scoring-uri}' \
 -H 'Authorization:Bearer {your-key}' \
 -H 'Content-Type: application/json' \
  -d '{ "data" : [ {model-specific-data-structure} ] }
```

## <a name="create-a-workspace-using-rest"></a>Создание рабочей области с помощью функции "ОСТАВШАЯся" 

Каждая Рабочая область машинного обучения Azure зависит от четырех других ресурсов Azure: реестра контейнеров с включенным администрированием, хранилищем ключей, ресурсом Application Insights и учетной записью хранения. Невозможно создать рабочую область, пока не будут созданы эти ресурсы. Сведения о создании каждого ресурса см. в справочнике по REST API.

Чтобы создать рабочую область, установите вызов, аналогичный приведенному ниже, в `management.azure.com` . Хотя этот вызов требует установки большого количества переменных, он структурно идентичен другим вызовам, обсуждаемым в этой статье. 

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.MachineLearningServices/workspaces/{your-new-workspace-name}?api-version=2019-11-01' \
  -H 'Authorization: Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{desired-region}",
    "properties": {
        "friendlyName" : "{your-workspace-friendly-name}",
        "description" : "{your-workspace-description}",
        "containerRegistry" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.ContainerRegistry/registries/{your-registry-name}",
        keyVault" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.Keyvault/vaults/{your-keyvault-name}",
        "applicationInsights" : "subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.insights/components/{your-application-insights-name}",
        "storageAccount" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.Storage/storageAccounts/{your-storage-account-name}"
    },
    "identity" : {
        "type" : "systemAssigned"
    }
}'
```

`202 Accepted`В возвращенных заголовках должен быть получен ответ и `Location` URI. Этот универсальный код ресурса (URI) можно получить для получения сведений о развертывании, включая полезные сведения об отладке, если возникла проблема с одним из зависимых ресурсов (например, если вы забыли включить административный доступ в реестре контейнеров). 

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="resource-provider-errors"></a>Ошибки поставщика ресурсов

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Перемещение рабочей области

> [!WARNING]
> Перемещение рабочей области Машинного обучения Azure в другую подписку или перемещение главной подписки на новый клиент не поддерживается. Это может привести к ошибкам.

### <a name="deleting-the-azure-container-registry"></a>Удаление Реестра контейнеров Azure

Для некоторых операций в рабочей области Машинного обучения Azure используется реестр контейнеров Azure (ACR) Он автоматически создает экземпляр ACR при необходимости.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с полным [справочником по AzureML REST API](/rest/api/azureml/).
- Узнайте, как использовать конструктор для [прогнозирования цены автомобилей с помощью конструктора](./tutorial-designer-automobile-price-train-score.md).
- Изучите [машинное обучение Azure с записными книжками Jupyter](..//machine-learning/samples-notebooks.md).
