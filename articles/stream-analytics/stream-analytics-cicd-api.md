---
title: Использование интерфейсов API RESTFUL для непрерывной интеграции и воспроизведения Azure Stream Analytics на IoT Edge
description: Сведения о реализации конвейера непрерывной интеграции и развертывании для Azure Stream Analytics с использованием REST API.
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/04/2018
ms.openlocfilehash: 3c3f776ad0996fa0b7422f0fca2d899a35e853d1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98016139"
---
# <a name="implement-cicd-for-stream-analytics-on-iot-edge-using-apis"></a>Реализация CI/CD для Stream Analytics на IoT Edge с использованием API

Вы можете включить непрерывную интеграцию и развертывание для заданий Azure Stream Analytics с помощью REST API. Эта статья содержит примеры и способы использования нужных API-интерфейсов. Интерфейсы REST API не поддерживаются в службе Azure Cloud Shell.

## <a name="call-apis-from-different-environments"></a>Вызов API-интерфейсов из различных сред

Интерфейсы REST API могут вызываться из Windows и Linux. В следующих командах продемонстрирован правильный синтаксис вызова API-интерфейсов. Использование определенных API-интерфейсов будет показано в следующих разделах этой статьи.

### <a name="linux"></a>Linux

Для Linux можно использовать команды `Curl` или `Wget`:

```bash
curl -u { <username:password> }  -H "Content-Type: application/json" -X { <method> } -d "{ <request body> }" { <url> }   
```

```bash
wget -q -O- --{ <method> } -data="<request body>" --header=Content-Type:application/json --auth-no-challenge --http-user="<Admin>" --http-password="<password>" <url>
```
 
### <a name="windows"></a>Windows

Для Windows используйте Powershell: 

```powershell 
$user = "<username>" 
$pass = "<password>" 
$encodedCreds = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $user,$pass))) 
$basicAuthValue = "Basic $encodedCreds" 
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
$headers.Add("Content-Type", 'application/json') 
$headers.Add("Authorization", $basicAuthValue) 
$content = "<request body>" 
$response = Invoke-RestMethod <url> -Method <method> -Body $content -Headers $Headers 
echo $response 
```
 
## <a name="create-an-asa-job-on-edge"></a>Создание задания ASA с помощью Edge 
 
Чтобы создать задание Stream Analytics, вызовите метод PUT с помощью API-интерфейса Stream Analytics.

|Метод|URL-адрес запроса|
|------|-----------|
|PUT|`https://management.azure.com/subscriptions/{\**subscription-id**}/resourcegroups/{**resource-group-name**}/providers/Microsoft.StreamAnalytics/streamingjobs/{**job-name**}?api-version=2017-04-01-preview`|
 
Пример использования команды **curl**:

```curl
curl -u { <username:password> } -H "Content-Type: application/json" -X { <method> } -d "{ <request body> }" https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/Microsoft.StreamAnalytics/streamingjobs/{jobname}?api-version=2017-04-01-preview  
``` 
 
Пример текста запроса в формате JSON:

```json
{ 
  "location": "West US", 
  "tags": { "key": "value", "ms-suppressjobstatusmetrics": "true" }, 
  "sku": {  
      "name": "Standard" 
    }, 
  "properties": { 
    "sku": {  
      "name": "standard" 
    }, 
       "eventsLateArrivalMaxDelayInSeconds": 1, 
       "jobType": "edge", 
    "inputs": [ 
      { 
        "name": "{inputname}", 
        "properties": { 
         "type": "stream", 
          "serialization": { 
            "type": "JSON", 
            "properties": { 
              "fieldDelimiter": ",", 
              "encoding": "UTF8" 
            } 
          }, 
          "datasource": { 
            "type": "GatewayMessageBus", 
            "properties": { 
            } 
          } 
        } 
      } 
    ], 
    "transformation": { 
      "name": "{queryName}", 
      "properties": { 
        "query": "{query}" 
      } 
    }, 
    "package": { 
      "storageAccount" : { 
        "accountName": "{blobstorageaccountname}", 
        "accountKey": "{blobstorageaccountkey}" 
      }, 
      "container": "{blobcontaine}]" 
    }, 
    "outputs": [ 
      { 
        "name": "{outputname}", 
        "properties": { 
          "serialization": { 
            "type": "JSON", 
            "properties": { 
              "fieldDelimiter": ",", 
              "encoding": "UTF8" 
            } 
          }, 
          "datasource": { 
            "type": "GatewayMessageBus", 
            "properties": { 
            } 
          } 
        } 
      } 
    ] 
  } 
} 
```
 
Дополнительные сведения см. в [документации по API-интерфейсам](/rest/api/streamanalytics/).  
 
## <a name="publish-edge-package"></a>Публикация пакета Edge 
 
Чтобы опубликовать задание Stream Analytics в IoT Edge, вызовите метод POST с помощью API-интерфейса Edge Package Publish.

|Метод|URL-адрес запроса|
|------|-----------|
|POST|`https://management.azure.com/subscriptions/{\**subscriptionid**}/resourceGroups/{**resourcegroupname**}/providers/Microsoft.StreamAnalytics/streamingjobs/{**jobname**}/publishedgepackage?api-version=2017-04-01-preview`|

Эта асинхронная операция возвращает состояние 202 до того, как задание будет успешно опубликовано. Заголовок ответа расположения содержит универсальный код ресурса (URI), который используется для получения состояния процесса. Во время выполнения процесса вызов универсального кода ресурса (URI) в заголовке расположения возвращает состояние 202. После того как процесс завершится, универсальный код ресурса (URI) возвращает состояние 200. 

Пример вызова публикации пакета Edge с помощью команды **curl**: 

```bash
curl -d -X POST https://management.azure.com/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.StreamAnalytics/streamingjobs/{jobname}/publishedgepackage?api-version=2017-04-01-preview
```
 
После отправки POST-запроса ожидается пустой текст ответа. Найдите URL-адрес, расположенный в заголовке ответа, и запишите его для дальнейшего использования.
 
Пример URL-адреса из заголовка ответа:

```
https://management.azure.com/subscriptions/{**subscriptionid**}/resourcegroups/{**resourcegroupname**}/providers/Microsoft.StreamAnalytics/StreamingJobs/{**resourcename**}/OperationResults/023a4d68-ffaf-4e16-8414-cb6f2e14fe23?api-version=2017-04-01-preview 
```
Подождите одну-две минуты, прежде чем запускать следующую команду, чтобы выполнить вызов API с помощью URL-адреса, который вы взяли из заголовка ответа. Повторите запрос, если не удалось получить ответ 200.
 
Пример вызова API с возвращаемым URL-адресом с помощью команды **curl**:

```bash
curl -d –X GET https://management.azure.com/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.StreamAnalytics/streamingjobs/{resourcename}/publishedgepackage?api-version=2017-04-01-preview 
```

Ответ включает сведения, которые необходимо добавить в скрипт развертывания Edge. В примерах ниже показаны сведения, которые вам необходимо собрать, а также места их вставки в манифесте развертывания.
 
Пример текста ответа после успешной публикации:

```json
{ 
  edgePackageUrl : null 
  error : null 
  manifest : "{"supportedPlatforms":[{"os":"linux","arch":"amd64","features":[]},{"os":"linux","arch":"arm","features":[]},{"os":"windows","arch":"amd64","features":[]}],"schemaVersion":"2","name":"{jobname}","version":"1.0.0.0","type":"docker","settings":{"image":"{imageurl}","createOptions":null},"endpoints":{"inputs":["\],"outputs":["{outputnames}"]},"twin":{"contentType":"assignments","content":{"properties.desired":{"ASAJobInfo":"{asajobsasurl}","ASAJobResourceId":"{asajobresourceid}","ASAJobEtag":"{etag}","PublishTimeStamp":"{publishtimestamp}"}}}}" 
  status : "Succeeded" 
} 
```

Пример манифеста развертывания: 

```json
{ 
  "modulesContent": { 
    "$edgeAgent": { 
      "properties.desired": { 
        "schemaVersion": "1.0", 
        "runtime": { 
          "type": "docker", 
          "settings": { 
            "minDockerVersion": "v1.25", 
            "loggingOptions": "", 
            "registryCredentials": {} 
          } 
        }, 
        "systemModules": { 
          "edgeAgent": { 
            "type": "docker", 
            "settings": { 
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0", 
              "createOptions": "{}" 
            } 
          }, 
          "edgeHub": { 
            "type": "docker", 
            "status": "running", 
            "restartPolicy": "always", 
            "settings": { 
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0", 
              "createOptions": "{}" 
            } 
          } 
        }, 
        "modules": { 
          "<asajobname>": { 
            "version": "1.0", 
            "type": "docker", 
            "status": "running", 
            "restartPolicy": "always", 
            "settings": { 
              "image": "<settings.image>", 
              "createOptions": "<settings.createOptions>" 
            } 
            "version": "<version>", 
             "env": { 
              "PlanId": { 
               "value": "stream-analytics-on-iot-edge" 
          } 
        } 
      } 
    }, 
    "$edgeHub": { 
      "properties.desired": { 
        "schemaVersion": "1.0", 
        "routes": { 
            "route": "FROM /* INTO $upstream" 
        }, 
        "storeAndForwardConfiguration": { 
          "timeToLiveSecs": 7200 
        } 
      } 
    }, 
    "<asajobname>": { 
      "properties.desired": {<twin.content.properties.desired>} 
    } 
  } 
} 
```

После того как вы настроите манифест развертывания, ознакомьтесь со статьей о [развертывании модулей Azure IoT Edge с помощью Azure CLI](../iot-edge/how-to-deploy-modules-cli.md), чтобы приступить к развертыванию.


## <a name="next-steps"></a>Дальнейшие действия 
 
* [Azure Stream Analytics в IoT Edge](stream-analytics-edge.md)
* [Deploy Azure Stream Analytics as an IoT Edge module - preview](../iot-edge/tutorial-deploy-stream-analytics.md) (Развертывание Azure Stream Analytics в качестве модуля IoT Edge (предварительная версия))
* [Разработка заданий Edge Stream Analytics с помощью средств Visual Studio](stream-analytics-tools-for-visual-studio-edge-jobs.md)