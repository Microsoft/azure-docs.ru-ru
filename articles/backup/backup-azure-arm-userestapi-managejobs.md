---
title: Управление заданиями резервного копирования с помощью REST API
description: Из этой статьи вы узнаете, как отслеживание заданий резервного копирования и восстановления Azure Backup с помощью REST API и управление ими.
ms.topic: conceptual
ms.date: 08/03/2018
ms.assetid: b234533e-ac51-4482-9452-d97444f98b38
ms.openlocfilehash: ced0e0020fe955734bf6cc767480fbadd6eaffc1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88890286"
---
# <a name="track-backup-and-restore-jobs-using-rest-api"></a>Отслеживание заданий резервного копирования и восстановления с помощью REST API

Azure Backup служба активирует задания, выполняемые в фоновом режиме, в различных сценариях, таких как запуск резервного копирования, операции восстановления, отключение резервного копирования. Эти задания можно отслеживать с помощью идентификаторов.

## <a name="fetch-job-information-from-operations"></a>Получение информации о задании из операций

Операция, такая как активация резервного копирования, будет всегда возвращать jobID. Например: окончательный ответ [триггера REST API операции резервного копирования](backup-azure-arm-userestapi-backupazurevms.md#example-responses-for-on-demand-backup) выглядит следующим образом:

```http
{
  "id": "cd153561-20d3-467a-b911-cc1de47d4763",
  "name": "cd153561-20d3-467a-b911-cc1de47d4763",
  "status": "Succeeded",
  "startTime": "2018-09-12T02:16:56.7399752Z",
  "endTime": "2018-09-12T02:16:56.7399752Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "41f3e94b-ae6b-4a20-b422-65abfcaf03e5"
  }
}
```

Задание резервного копирования виртуальной машины Azure идентифицируется по полю jobId и может отслеживаться, как уже упоминалось [здесь](/rest/api/backup/jobdetails/), с использованием простого запроса *GET*.

## <a name="tracking-the-job"></a>Отслеживание задания

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupJobs/{jobName}?api-version=2019-05-13
```

Имя задания `{jobName}` — JobId, упомянутое выше. Ответ — всегда 200 OК с полем status, указывающим текущее состояние задания. После завершения работы или "Комплетедвисварнингс" в разделе "Екстендединфо" появится больше сведений о задании.

### <a name="response"></a>Ответ

|Имя  |Type  |Описание  |
|---------|---------|---------|
|200 ОК     | [JobResource](/rest/api/backup/jobdetails/get#jobresource)        | ОК        |

#### <a name="example-response"></a>Пример ответа

После отправки универсального кода ресурса (URI) *GET* возвращается ответ 200 (OК).

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: e9702101-9da2-4681-bdf3-a54e17329a56
x-ms-client-request-id: ba4dff71-1655-4c1d-a71f-c9869371b18b; ba4dff71-1655-4c1d-a71f-c9869371b18b
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14989
x-ms-correlation-request-id: e9702101-9da2-4681-bdf3-a54e17329a56
x-ms-routing-request-id: SOUTHINDIA:20180521T102317Z:e9702101-9da2-4681-bdf3-a54e17329a56
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:23:17 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-RecoveryServices-ResourceGroup-centralindia/providers/microsoft.recoveryservices/vaults/abdemovault/backupJobs/7ddead57-bcb9-4269-ac31-6a1b57588700",
  "name": "7ddead57-bcb9-4269-ac31-6a1b57588700",
  "type": "Microsoft.RecoveryServices/vaults/backupJobs",
  "properties": {
    "jobType": "AzureIaaSVMJob",
    "duration": "00:20:23.0896697",
    "actionsInfo": [
      1
    ],
    "virtualMachineVersion": "Compute",
    "extendedInfo": {
      "tasksList": [
        {
          "taskId": "Take Snapshot",
          "duration": "00:00:00",
          "status": "Completed"
        },
        {
          "taskId": "Transfer data to vault",
          "duration": "00:00:00",
          "status": "Completed"
        }
      ],
      "propertyBag": {
        "VM Name": "uttestvmub1",
        "Backup Size": "2332 MB"
      }
    },
    "entityFriendlyName": "uttestvmub1",
    "backupManagementType": "AzureIaasVM",
    "operation": "Backup",
    "status": "Completed",
    "startTime": "2018-05-21T08:35:40.9488967Z",
    "endTime": "2018-05-21T08:56:04.0385664Z",
    "activityId": "7df8e874-1d66-4f81-8e91-da2fe054811d"
  }
}
}

```
