---
title: Восстановление виртуальных машин Azure с помощью REST API
description: Из этой статьи вы узнаете, как управлять операциями восстановления резервной копии виртуальных машин Azure с помощью REST API.
ms.topic: conceptual
ms.date: 09/12/2018
ms.assetid: b8487516-7ac5-4435-9680-674d9ecf5642
ms.openlocfilehash: da6b4cd6134f0cd1fd3d6e04e814bbf8aec9b07d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102452158"
---
# <a name="restore-azure-virtual-machines-using-rest-api"></a>Восстановление виртуальных машин Azure с помощью REST API

После завершения резервного копирования виртуальной машины Azure, использующей Azure Backup, можно восстановить все виртуальные машины Azure или диски или файлы из одной и той же резервной копии. В этой статье описываются способы восстановления виртуальной машины Azure или дисков с помощью REST API.

Для любой операции восстановления необходимо сначала определить соответствующую точку восстановления.

## <a name="select-recovery-point"></a>Выбор точки восстановления

Доступные точки восстановления архивируемого элемента можно перечислить при помощи [перечисления точек восстановления REST API](/rest/api/backup/recoverypoints/list). Это простая операция *Get* со всеми соответствующими значениями.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints?api-version=2019-05-13
```

`{containerName}` и `{protectedItemName}` созданы [здесь](backup-azure-arm-userestapi-backupazurevms.md#example-responses-to-get-operation). Значение параметра `{fabricName}` — Azure.

URI *GET* имеет все необходимые параметры. Нет необходимости в дополнительном тексте запроса.

### <a name="responses"></a>Ответы

|Имя  |Type  |Описание  |
|---------|---------|---------|
|200 ОК     |   [RecoveryPointResourceList](/rest/api/backup/recoverypoints/list#recoverypointresourcelist)      |       ОК  |

#### <a name="example-response"></a>Пример ответа

После отправки универсального кода ресурса (URI) *GET* возвращается ответ 200 (OК).

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: 03453538-2f8d-46de-8374-143ccdf60f40
x-ms-client-request-id: c48f4436-ce3f-42da-b537-12710d4d1a24; c48f4436-ce3f-42da-b537-12710d4d1a24
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14998
x-ms-correlation-request-id: 03453538-2f8d-46de-8374-143ccdf60f40
x-ms-routing-request-id: SOUTHINDIA:20180604T071851Z:03453538-2f8d-46de-8374-143ccdf60f40
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 07:18:51 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/VM;testRG;testVM/recoveryPoints/20982486783671",
      "name": "20982486783671",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints",
      "properties": {
        "objectType": "IaasVMRecoveryPoint",
        "recoveryPointType": "AppConsistent",
        "recoveryPointTime": "2018-06-04T06:06:00.5121087Z",
        "recoveryPointAdditionalInfo": "",
        "sourceVMStorageType": "NormalStorage",
        "isSourceVMEncrypted": false,
        "isInstantIlrSessionActive": false,
        "recoveryPointTierDetails": [
          {
            "type": 1,
            "status": 1
          },
          {
            "type": 2,
            "status": 1
          }
        ],
        "isManagedVirtualMachine": true,
        "virtualMachineSize": "Standard_A1_v2",
        "originalStorageAccountOption": false
      }
    },
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/VM;testRG;testVM/recoveryPoints/23358112038108",
      "name": "23358112038108",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints",
      "properties": {
        "objectType": "IaasVMRecoveryPoint",
        "recoveryPointType": "CrashConsistent",
        "recoveryPointTime": "2018-06-03T06:20:56.3043878Z",
        "recoveryPointAdditionalInfo": "",
        "sourceVMStorageType": "NormalStorage",
        "isSourceVMEncrypted": false,
        "isInstantIlrSessionActive": false,
        "recoveryPointTierDetails": [
          {
            "type": 1,
            "status": 1
          },
          {
            "type": 2,
            "status": 1
          }
        ],
        "isManagedVirtualMachine": true,
        "virtualMachineSize": "Standard_A1_v2",
        "originalStorageAccountOption": false
      }
    },
......
```

Точка восстановления определяется с помощью ответа `{name}` выше.

## <a name="restore-operations"></a>Операции восстановления

Выбрав [соответствующую точку восстановления](#select-recovery-point), перейдите к активации операции восстановления.

***Все операции восстановления в элементе Backup выполняются с помощью того же API-интерфейса *POST* . Только изменения текста запроса с помощью сценариев восстановления.***

> [!IMPORTANT]
> [Здесь](./backup-azure-arm-restore-vms.md#restore-options)приводятся все сведения о различных параметрах восстановления и их зависимостях. Прежде чем приступить к активации этих операций, ознакомьтесь с разработкой.

Активация операций восстановления является запросом *POST* . Дополнительные сведения об API см. в [REST API "триггер Restore"](/rest/api/backup/restores/trigger).

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints/{recoveryPointId}/restore?api-version=2019-05-13
```

`{containerName}` и `{protectedItemName}` созданы [здесь](backup-azure-arm-userestapi-backupazurevms.md#example-responses-to-get-operation). `{fabricName}` — это Azure, а `{recoveryPointId}` — это поле `{name}` точки восстановления, которое упоминалось [выше](#example-response).

После получения точки восстановления необходимо создать текст запроса для соответствующего сценария восстановления. В следующих разделах приводится краткое описание текста запроса для каждого сценария.

- [Восстановление дисков](#restore-disks)
- [Замена дисков](#replace-disks-in-a-backed-up-virtual-machine)
- [Восстановление в качестве новой виртуальной машины](#restore-as-another-virtual-machine)

### <a name="restore-response"></a>Ответ на восстановление

Активация любой операции восстановления является [асинхронной операцией](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Она возвращает два ответа: 202 (принято), когда создается другая операция, и 200 (ОК), когда эта операция завершается.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|202 — принято     |         |     Принято    |

#### <a name="example-responses"></a>Примеры ответов

После отправки URI *POST* для активации восстановления дисков будет получен первоначальный ответ 202 (принято) с заголовком расположения или асинхронным заголовком Azure.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: 893fe372-8d6c-4c56-b589-45a95eeef95f
x-ms-client-request-id: a15ce064-25bd-4ac6-87e5-e3bc6ec65c0b; a15ce064-25bd-4ac6-87e5-e3bc6ec65c0b
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1198
x-ms-correlation-request-id: 893fe372-8d6c-4c56-b589-45a95eeef95f
x-ms-routing-request-id: SOUTHINDIA:20180604T130003Z:893fe372-8d6c-4c56-b589-45a95eeef95f
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 13:00:03 GMT
Location: https://management.azure.com/subscriptions//subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Затем отследите итоговую операцию, используя заголовок location или Azure-AsyncOperation с помощью простой команды *GET*.

```http
GET https://management.azure.com/subscriptions//subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
```

После завершения операции она возвращает код 200 (ОК) с идентификатором результата задания восстановления в тексте ответа.

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: ea2a8011-eb83-4a4b-9ed2-e694070a966a
x-ms-client-request-id: a7f3a144-ed80-41ee-bffe-ae6a90c35a2f; a7f3a144-ed80-41ee-bffe-ae6a90c35a2f
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14973
x-ms-correlation-request-id: ea2a8011-eb83-4a4b-9ed2-e694070a966a
x-ms-routing-request-id: SOUTHINDIA:20180604T130917Z:ea2a8011-eb83-4a4b-9ed2-e694070a966a
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 13:09:17 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "781a0f18-e250-4d73-b059-5e9ffed4069e",
  "name": "781a0f18-e250-4d73-b059-5e9ffed4069e",
  "status": "Succeeded",
  "startTime": "2018-06-04T13:00:03.8068176Z",
  "endTime": "2018-06-04T13:00:03.8068176Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "3021262a-fb3a-4538-9b37-e3e97a386093"
  }
}
```

Так как задание восстановления является длительной операцией, его следует отслеживать, как описано в статье [мониторинг заданий с помощью REST API документа](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

### <a name="restore-disks"></a>Восстановление дисков

Если необходимо настроить создание виртуальной машины на основе данных резервной копии, можно просто восстановить диски в выбранной учетной записи хранения и создать виртуальную машину на основе этих дисков в соответствии с их требованиями. Учетная запись хранения должна находиться в том же регионе, что и хранилище служб восстановления, и не должна быть избыточной зоной. Диски, а также конфигурация виртуальной машины, сохраненной в резервной копии ("vmconfig.jsна"), будут храниться в указанной учетной записи хранения. Как упоминалось [выше](#restore-operations), соответствующий текст запроса для восстановления дисков приведен ниже.

#### <a name="create-request-body"></a>Создание текста запроса

Чтобы инициировать восстановление диска из резервной копии виртуальной машины Azure, выполните компоненты текста запроса.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|properties     | [IaaSVMRestoreRequest](/rest/api/backup/restores/trigger#iaasvmrestorerequest)        |    RestoreRequestResourceProperties     |

Полный список определений в тексте запроса и другие сведения см. в [документации REST API о запуске восстановления](/rest/api/backup/restores/trigger#request-body).

##### <a name="example-request"></a>Пример запроса

Следующий текст запроса определяет свойства, необходимые для запуска восстановления диска.

```json
{
  "properties": {
    "objectType": "IaasVMRestoreRequest",
    "recoveryPointId": "20982486783671",
    "recoveryType": "RestoreDisks",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",
    "region": "westus",
    "createNewCloudService": false,
    "originalStorageAccountOption": false,
    "encryptionDetails": {
      "encryptionEnabled": false
    }
  }
}
```

### <a name="restore-disks-selectively"></a>Выборочное восстановление дисков

Если вы [выборочно создаете резервные копии дисков](backup-azure-arm-userestapi-backupazurevms.md#excluding-disks-in-azure-vm-backup), список резервных дисков, доступных для архивации, приведен в [сводке по точкам восстановления](#select-recovery-point) и [подробному ответу](/rest/api/backup/recoverypoints/get). Вы также можете выборочно восстановить диски и дополнительные [сведения.](selective-disk-backup-restore.md#selective-disk-restore) Чтобы выборочно восстановить диск из списка резервных копий дисков, найдите LUN диска в ответе точки восстановления и добавьте свойство **ресторедисклунлист** в [текст запроса выше](#example-request) , как показано ниже.

```json
{
    "properties": {
        "objectType": "IaasVMRestoreRequest",
        "recoveryPointId": "20982486783671",
        "recoveryType": "RestoreDisks",
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",
        "region": "westus",
        "createNewCloudService": false,
        "originalStorageAccountOption": false,
        "encryptionDetails": {
          "encryptionEnabled": false
        },
        "restoreDiskLunList" : [0]
    }
}

```

После того как вы отправите ответ, как описано [выше](#responses), и полное задание выполнено, диски и Конфигурация виртуальной машины ("VMConfig.jsна") будут присутствовать в указанной учетной записи хранения.

### <a name="replace-disks-in-a-backed-up-virtual-machine"></a>Замена дисков в резервной виртуальной машине

При восстановлении дисков из точки восстановления диски заменяют текущие диски резервной копии виртуальной машины дисками из точки восстановления. Как упоминалось [выше](#restore-operations), соответствующий текст запроса для замены дисков приведен ниже.

#### <a name="create-request-body"></a>Создание текста запроса

Чтобы активировать замену дисков из резервной копии виртуальной машины Azure, ниже приведены компоненты текста запроса.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|properties     | [IaaSVMRestoreRequest](/rest/api/backup/restores/trigger#iaasvmrestorerequest)        |    RestoreRequestResourceProperties     |

Полный список определений в тексте запроса и другие сведения см. в [документации REST API о запуске восстановления](/rest/api/backup/restores/trigger#request-body).

#### <a name="example-request"></a>Пример запроса

Следующий текст запроса определяет свойства, необходимые для запуска восстановления диска.

```json
{
    "properties": {
        "objectType": "IaasVMRestoreRequest",
        "recoveryPointId": "20982486783671",
        "recoveryType": "OriginalLocation",
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",  
        "region": "westus",
        "createNewCloudService": false,
        "originalStorageAccountOption": false,
        "affinityGroup": "",
        "diskEncryptionSetId": null,
        "subnetId": null,
        "targetDomainNameId": null,
        "targetResourceGroupId": null,
        "targetVirtualMachineId": null,
        "virtualNetworkId": null
     }
}

```

### <a name="restore-as-another-virtual-machine"></a>Восстановление в качестве другой виртуальной машины

Как упоминалось [выше](#restore-operations), в следующем тексте запроса определяются свойства, необходимые для запуска восстановления виртуальной машины.

```json
{
  "properties": {
          "objectType":  "IaasVMRestoreRequest",
          "recoveryPointId": "348916168024334",
          "recoveryType": "AlternateLocation",
          "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
          "targetVirtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Compute/virtualmachines/targetVM",
          "targetResourceGroupId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG",
          "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testingAccount",
          "virtualNetworkId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Network/virtualNetworks/testNet",
          "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Network/virtualNetworks/testNet/subnets/default",
          "region": "westus",
          "createNewCloudService": false,
          "originalStorageAccountOption": false,
          "encryptionDetails": {
            "encryptionEnabled": false
     }
 }
```

Для восстановления дисков ответы должны обрабатываться так же, как [описано выше](#responses).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о REST API Azure Backup с использованием API REST см. в следующих документах:

- [Recovery Services](/rest/api/recoveryservices/) (Службы восстановления)
- [Начало работы с Azure REST API](/rest/api/azure/)
