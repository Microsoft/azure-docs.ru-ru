---
title: Резервное копирование виртуальных машин Azure с помощью REST API
description: Из этой статьи вы узнаете, как настраивать, инициировать и администрировать операции резервного копирования виртуальной машины Azure с помощью REST API.
ms.topic: conceptual
ms.date: 08/03/2018
ms.assetid: b80b3a41-87bf-49ca-8ef2-68e43c04c1a3
ms.openlocfilehash: 9ba22c51c7a6c26a232ed20aec21fc83d2c54b37
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92171459"
---
# <a name="back-up-an-azure-vm-using-azure-backup-via-rest-api"></a>Резервное копирование виртуальных машин Azure с помощью службы Azure Backup и REST API

В этой статье описано, как управлять резервными копиями виртуальных машин Azure с помощью службы Azure Backup и REST API. Настройте защиту в первый раз для ранее незащищенной виртуальной машины Azure, активируйте резервное копирование по запросу для защищенной ВИРТУАЛЬНОЙ машины Azure и измените свойства резервной копии виртуальной машины с помощью REST API, как описано здесь.

См. дополнительные сведения о создании [хранилищ](backup-azure-arm-userestapi-createorupdatevault.md) и [политик](backup-azure-arm-userestapi-createorupdatepolicy.md) с помощью REST API.

Предположим, вы хотите защитить виртуальную машину testVM в группе ресурсов testRG в хранилище Служб восстановления testVault, расположенном в группе ресурсов testVaultRG, с помощью политики по умолчанию (с именем DefaultPolicy).

## <a name="configure-backup-for-an-unprotected-azure-vm-using-rest-api"></a>Настройка резервного копирования для незащищенных виртуальных машин Azure с помощью REST API

### <a name="discover-unprotected-azure-vms"></a>Обнаружение незащищенных виртуальных машин Azure

Во-первых, для хранилища нужно обеспечить возможность идентификации виртуальной машины Azure. Это можно сделать с помощью [операции обновления](/rest/api/backup/protectioncontainers/refresh). Это асинхронная операция *POST*  , которая гарантирует, что хранилище получает последний список всех незащищенных виртуальных машин в текущей подписке и кэширует их. После кэширования виртуальной машины Службы восстановления получат доступ к виртуальной машине для включения защиты.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01
```

Для URI POST используются параметры `{subscriptionId}`, `{vaultName}`, `{vaultresourceGroupName}`, `{fabricName}`. Значение параметра `{fabricName}` — Azure. В соответствии с нашим примером `{vaultName}` это «testvault задано» и `{vaultresourceGroupName}` «тестваултрг». Так как все необходимые параметры задаются в универсальном коде ресурса (URI), нет необходимости в отдельном тексте запроса.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01
```

#### <a name="responses-to-refresh-operation"></a>Ответы на операцию обновления

Операция обновления — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Она возвращает два ответа: 202 (принято), когда создается другая операция, и 200 (ОК), когда эта операция завершается.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|204 No Content (содержимое отсутствует)     |         |  ОК (без содержимого, которое возвращаются)      |
|202 — принято     |         |     Принято    |

##### <a name="example-responses-to-refresh-operation"></a>Примеры ответов для операции обновления

После отправления запроса *POST* возвращается ответ 202 (принято).

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
X-Content-Type-Options: nosniff
x-ms-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-client-request-id: 4910609f-bb9b-4c23-8527-eb6fa2d3253f; 4910609f-bb9b-4c23-8527-eb6fa2d3253f
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-routing-request-id: SOUTHINDIA:20180521T105701Z:43cf550d-e463-421c-8922-37e4766db27d
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:57:00 GMT
Location: https://management.azure.com/subscriptions//00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Отследите итоговую операцию, используя заголовок location с помощью простой команды *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
```

После обнаружения всех виртуальных машин Azure команда GET возвращает номер ответа 204 (содержимое отсутствует). Хранилище теперь сможет обнаруживать все виртуальные машины в пределах подписки.

```http
HTTP/1.1 204 NoContent
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-client-request-id: 25bb6345-f9fc-4406-be1a-dc6db0eefafe; 25bb6345-f9fc-4406-be1a-dc6db0eefafe
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14997
x-ms-correlation-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-routing-request-id: SOUTHINDIA:20180521T105825Z:cf6cd73b-9189-4942-a61d-878fcf76b1c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:58:25 GMT
X-Powered-By: ASP.NET
```

### <a name="selecting-the-relevant-azure-vm"></a>Выбор нужной виртуальной машины Azure

 Вы можете убедиться, что кэширование выполнено, [отобразив список всех защищаемых элементов](/rest/api/backup/backupprotectableitems/list) в пределах подписки. Затем в ответе найдите нужную виртуальную машину. [Ответ этой операции](#example-responses-to-get-operation) также содержит сведения о том, как службы восстановления ИДЕНТИФИЦИРУют виртуальную машину.  Ознакомившись с этой процедурой, вы можете пропустить этот шаг и перейти непосредственно к [включению защиты](#enabling-protection-for-the-azure-vm).

Это можно сделать с помощью операции *GET*.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter=backupManagementType eq 'AzureIaasVM'
```

URI *GET* имеет все необходимые параметры. Для этой операции текст запроса также не требуется.

#### <a name="responses-to-get-operation"></a>Ответы на операцию получения

|Имя  |Type  |Описание  |
|---------|---------|---------|
|200 ОК     | [WorkloadProtectableItemResourceList](/rest/api/backup/backupprotectableitems/list#workloadprotectableitemresourcelist)        |       ОК |

#### <a name="example-responses-to-get-operation"></a>Примеры ответов для получения операции

После отправки запроса *GET* возвращается ответ 200 (OK).

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-client-request-id: 40c8601a-c217-4c68-87da-01db8dac93dd; 40c8601a-c217-4c68-87da-01db8dac93dd
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14979
x-ms-correlation-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-routing-request-id: SOUTHINDIA:20180521T071408Z:7c2cf56a-e6be-4345-96df-c27ed849fe36
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:14:08 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainerv2;testRG;testVM/protectableItems/vm;iaasvmcontainerv2;testRG;testVM",
      "name": "iaasvmcontainerv2;testRG;testVM",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
      "properties": {
        "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "virtualMachineVersion": "Compute",
        "resourceGroup": "testRG",
        "backupManagementType": "AzureIaasVM",
        "protectableItemType": "Microsoft.Compute/virtualMachines",
        "friendlyName": "testVM",
        "protectionState": "NotProtected"
      }
    },……………..

```

> [!TIP]
> Число значений в ответе *GET* ограничено 200 на страницу. Используйте поле nextLink, чтобы получить URL-адрес для следующего набора ответов.

Ответ содержит список всех незащищенных виртуальных машин Azure, где каждое значение `{value}` содержит все сведения, необходимые для настройки резервного копирования Службами восстановления Azure. При настройке резервного копирования запишите значения полей `{name}` и `{virtualMachineId}` в разделе `{properties}`. Создайте из этих значений полей две переменные, как показано ниже.

- containerName = "iaasvmcontainer;"+`{name}`
- protectedItemName = "vm;"+ `{name}`
- `{virtualMachineId}` будет использоваться позже в [тексте запроса](#example-request-body).

В нашем примере указанные выше значения будут такими:

- containerName = "iaasvmcontainer;iaasvmcontainerv2;testRG;testVM"
- protectedItemName = "vm;iaasvmcontainerv2;testRG;testVM"

### <a name="enabling-protection-for-the-azure-vm"></a>Включение защиты для виртуальной машины Azure

После кэширования и идентификации соответствующей виртуальной машины выберите политику защиты. Дополнительные сведения о существующих политиках в хранилище см. в [списке API политик](/rest/api/backup/backuppolicies/list). Ссылаясь на имя политики, выберите [соответствующую политику](/rest/api/backup/protectionpolicies/get). См. дополнительные сведения о [создании политик](backup-azure-arm-userestapi-createorupdatepolicy.md). В примере ниже выбрано "DefaultPolicy".

Включение защиты — это асинхронная операция *PUT*, которая создает защищенный элемент.

```http
https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Значения параметров `{containerName}` и `{protectedItemName}` соответствуют указанным выше. Значение параметра `{fabricName}` — Azure. В нашем примере мы получим следующее:

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="create-the-request-body"></a>Создание текста запроса

Чтобы создать защищенный элемент, используйте компоненты текста запроса.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|properties     | AzureIaaSVMProtectedItem        |Свойства ресурса ProtectedItem         |

Полный список определений в тексте запроса и другие сведения см. в [документации по созданию защищенного элемента для REST API](/rest/api/backup/protecteditems/createorupdate#request-body).

##### <a name="example-request-body"></a>Примеры текста запроса

Следующий текст запроса определяет свойства, необходимые для создания защищенного элемента.

```json
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy"
  }
}
```

`{sourceResourceId}` и `{virtualMachineId}` — это использованные ранее параметры [ответа со списком защищаемых элементов](#example-responses-to-get-operation).

#### <a name="responses-to-create-protected-item-operation"></a>Ответы для операции создания защищенного элемента

Создание защищенного элемента — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Она возвращает два ответа: 202 (принято), когда создается другая операция, и 200 (ОК), когда эта операция завершается.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|200 ОК     |    [ProtectedItemResource](/rest/api/backup/protecteditemoperationresults/get#protecteditemresource)     |  ОК       |
|202 — принято     |         |     Принято    |

##### <a name="example-responses-to-create-protected-item-operation"></a>Примеры ответов для операции создания защищенного элемента

После отправки запроса *PUT* для создания или обновления защищенного элемента будет получен первоначальный ответ 202 (принято) с заголовками location или Azure-AsyncOperation.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-client-request-id: e1f94eef-9b2d-45c4-85b8-151e12b07d03; e1f94eef-9b2d-45c4-85b8-151e12b07d03
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-routing-request-id: SOUTHINDIA:20180521T073907Z:db785be0-bb20-4598-bc9f-70c9428b170b
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:39:06 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Затем отследите итоговую операцию, используя заголовок location или Azure-AsyncOperation с помощью простой команды *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Завершенная операция возвращает код 200 (ОК) с содержимым защищенного элемента в тексте ответа.

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM",
  "name": "VM;testRG;testVM",
  "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems",
  "properties": {
    "friendlyName": "testVM",
    "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "protectionStatus": "Healthy",
    "protectionState": "IRPending",
    "healthStatus": "Passed",
    "lastBackupStatus": "",
    "lastBackupTime": "2001-01-01T00:00:00Z",
    "protectedItemDataId": "17592691116891",
    "extendedInfo": {
      "recoveryPointCount": 0,
      "policyInconsistent": false
    },
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "backupManagementType": "AzureIaasVM",
    "workloadType": "VM",
    "containerName": "iaasvmcontainerv2;testRG;testVM",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy",
    "policyName": "DefaultPolicy"
  }
}
```

Это подтверждает, что для виртуальной машины включена защита, а первая резервная копия будет активирована в соответствии с расписанием политики.

### <a name="excluding-disks-in-azure-vm-backup"></a>Исключение дисков в резервной копии виртуальной машины Azure

Azure Backup также предоставляет способ выборочного резервного копирования подмножества дисков в виртуальной машине Azure. Дополнительные сведения приведены [здесь](selective-disk-backup-restore.md). Если вы хотите выборочно создать резервную копию нескольких дисков во время включения защиты, следующий фрагмент кода должен быть [телом запроса во время включения защиты](#example-request-body).

```json
{
"properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy",
    "extendedProperties":  {
      "diskExclusionProperties":{
          "diskLunList":[0,1],
          "isInclusionList":true
        }
    }
}
}
```

В тексте запроса выше список дисков для резервного копирования предоставляется в разделе Расширенные свойства.

|Свойство.  |Значение  |
|---------|---------|
|дисклунлист     | Список LUN диска представляет собой список *LUN дисков данных*. **Резервное копирование диска операционной системы всегда не требуется**.        |
|исинклусионлист     | Должен иметь **значение true** , чтобы LUN был включен во время резервного копирования. Если значение равно **false**, упомянутые выше LUN будут исключены.         |

Поэтому, если требуется только резервное копирование только диска операционной системы, необходимо исключить _все_ диски данных. Проще всего сказать, что диски данных включать не нужно. Поэтому список LUN диска будет пустым, и **исинклусионлист** будет иметь **значение true**. Аналогичным образом, подумайте о том, что является более простым способом выбора подмножества: несколько дисков следует всегда исключать, или необходимо всегда включать несколько дисков. Выберите список LUN и значение логической переменной соответствующим образом.

## <a name="trigger-an-on-demand-backup-for-a-protected-azure-vm"></a>Активация резервного копирования по запросу для защищенной виртуальной машины Azure

Когда виртуальная машина Azure настроена для архивации, резервное копирование выполняется в соответствии с расписанием политики. Вы можете дождаться первого запланированного резервного копирования или активировать эту операцию по требованию в любое время. Срок хранения для резервных копий по запросу отличается от срока хранения политики резервного копирования. Кроме того, вы можете указать определенные дату и время. Если срок хранения не будет указан, будет использоваться значение по умолчанию — 30 дней со дня активации резервного копирования по запросу.

Активация резервного копирования по запросу — это операция *POST*.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

Значения параметров `{containerName}` и `{protectedItemName}` соответствуют указанным [выше](#responses-to-get-operation). Значение параметра `{fabricName}` — Azure. В нашем примере мы получим следующее:

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM/backup?api-version=2016-12-01
```

### <a name="create-the-request-body-for-on-demand-backup"></a>Создание текста запроса для резервного копирования по запросу

Чтобы активировать резервное копирование по запросу, используйте компоненты текста запроса.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|properties     | [IaaSVMBackupRequest](/rest/api/backup/backups/trigger#iaasvmbackuprequest)        |Свойства BackupRequestResource         |

Полный список определений в тексте запроса и другие сведения см. в [документации по активации резервного копирования для защищенных элементов для REST API](/rest/api/backup/backups/trigger#request-body).

#### <a name="example-request-body-for-on-demand-backup"></a>Пример текста запроса для резервного копирования по запросу

Следующий текст запроса определяет свойства, необходимые для активации резервного копирования для защищенного элемента. Если срок хранения не указан, он будет храниться в течение 30 дней с момента запуска задания резервного копирования.

```json
{
   "properties": {
    "objectType": "IaasVMBackupRequest",
    "recoveryPointExpiryTimeInUTC": "2018-12-01T02:16:20.3156909Z"
  }
}
```

### <a name="responses-for-on-demand-backup"></a>Ответы на резервное копирование по запросу

Активация резервного копирования по запросу — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Она возвращает два ответа: 202 (принято), когда создается другая операция, и 200 (ОК), когда эта операция завершается.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|202 — принято     |         |     Принято    |

#### <a name="example-responses-for-on-demand-backup"></a>Примеры ответов для резервного копирования по запросу

После отправки запроса *POST* для выполнения резервного копирования по запросу будет получен первоначальный ответ 202 (принято) с заголовками location или Azure-AsyncOperation.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-client-request-id: 7df8e874-1d66-4f81-8e91-da2fe054811d; 7df8e874-1d66-4f81-8e91-da2fe054811d
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-routing-request-id: SOUTHINDIA:20180521T083541Z:7885ca75-c7c6-43fb-a38c-c0cc437d8810
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:35:41 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationResults/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Затем отследите итоговую операцию, используя заголовок location или Azure-AsyncOperation с помощью простой команды *GET*.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Завершенная операция возвращает код 200 (ОК) с идентификатором результата задания резервного копирования в тексте ответа.

```json
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-client-request-id: 5a63209d-3708-4e69-a75f-9499f4c8977c; 5a63209d-3708-4e69-a75f-9499f4c8977c
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14995
x-ms-correlation-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-routing-request-id: SOUTHINDIA:20180521T083723Z:a8b13524-2c95-445f-b107-920806f385c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:37:22 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "00000000-0000-0000-0000-000000000000",
  "status": "Succeeded",
  "startTime": "2018-05-21T08:35:40.9488967Z",
  "endTime": "2018-05-21T08:35:40.9488967Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "7ddead57-bcb9-4269-ac31-6a1b57588700"
  }
}
```

Так как задание резервного копирования — это длительная операция, оно должно отслеживаться, как описано в [документации по мониторингу заданий для REST API](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="modify-the-backup-configuration-for-a-protected-azure-vm"></a>Изменение параметров резервного копирования для защищенной виртуальной машины Azure

### <a name="changing-the-policy-of-protection"></a>Изменение политики защиты

Чтобы изменить политику, с помощью которой обеспечивается защита виртуальной машины, можно использовать тот же формат, что при [включении защиты](#enabling-protection-for-the-azure-vm). Просто укажите идентификатор политики в [тексте запроса](#example-request-body) и отправьте запрос. Например, чтобы изменить политику testVM с "DefaultPolicy" на "Продполици", укажите идентификатор "Продполици" в тексте запроса.

```json
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/ProdPolicy"
  }
}
```

Как уже упоминалось, ответ будет в таком же формате, как и при [включении защиты](#responses-to-create-protected-item-operation).

#### <a name="excluding-disks-during-azure-vm-protection"></a>Исключение дисков во время защиты виртуальной машины Azure

Если резервная копия виртуальной машины Azure уже создана, можно указать список дисков для резервного копирования или исключения, изменив политику защиты. Просто Подготовьте запрос в том же формате, что и [исключение дисков во время включения защиты](#excluding-disks-in-azure-vm-backup) .

> [!IMPORTANT]
> Текст запроса выше всегда является окончательной копией дисков данных, которые необходимо исключить или добавить. Это не приводит к *добавлению* в предыдущую конфигурацию. Например, если вы сначала обновляете защиту как "исключить данные с диска 1", а затем повторяйте с параметром "исключить диск данных 2", в последующих резервных копиях будет *исключен только диск данных 2* , а на диск данных 1 будут включены данные. Это всегда окончательный список, который будет включен в последующие резервные копии или исключен из него.

Чтобы получить текущий список дисков, которые исключены или включены, получите защищенные сведения об элементе, как описано [здесь](/rest/api/backup/protecteditems/get). Ответ предоставит список LUN диска данных и указывает, включены ли они или исключены из них.

### <a name="stop-protection-but-retain-existing-data"></a>Снятие защиты с сохранением существующих данных

Чтобы снять защиту с защищенной виртуальной машины и сохранить резервную копию данных, удалите политику из текста запроса и отправьте запрос. После удаления связи с политикой операции резервного копирования больше не будут активироваться и новые точки восстановления не будут создаваться.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": ""
  }
}
```

Ответ будет в том же формате, что и при [активации резервного копирования по запросу](#example-responses-for-on-demand-backup). Итоговое задание должно отслеживаться, как описано в [документации по мониторингу заданий для REST API](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

### <a name="stop-protection-and-delete-data"></a>Снятие защиты и удаление данных

Чтобы снять защиту с защищенной виртуальной машины и удалить резервную копию данных, выполните [операцию удаления](/rest/api/backup/protecteditems/delete).

Остановка защиты и удаление данных осуществляется с помощью операции *DELETE*.

```http
DELETE https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Значения параметров `{containerName}` и `{protectedItemName}` соответствуют указанным [выше](#responses-to-get-operation). Значение параметра `{fabricName}` — Azure. В нашем примере мы получим следующее:

```http
DELETE https://management.azure.com//Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="responses-for-delete-protection"></a>Ответы для защиты при удалении

*DELETE* — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Она возвращает два ответа: 202 (принято), когда создается другая операция, и 204 (содержимое отсутствует), когда эта операция завершается.

|Имя  |Type  |Описание  |
|---------|---------|---------|
|204 NoContent (содержимое отсутствует)     |         |  NoContent       |
|202 — принято     |         |     Принято    |

> [!IMPORTANT]
> Для защиты от случайных сценариев удаления существует [функция обратимого удаления, доступная](use-restapi-update-vault-properties.md#soft-delete-state) для хранилища служб восстановления. Если для состояния обратимого удаления хранилища задано значение включено, то операция удаления не будет немедленно удалять данные. Он будет храниться в течение 14 дней, а затем окончательно очищен. За этот период за 14 дней не будет заряжена стоимость хранилища. Чтобы отменить операцию удаления, ознакомьтесь с [разделом Отмена и удаление](#undo-the-deletion).

### <a name="undo-the-deletion"></a>Отменить удаление

Отмена случайного удаления аналогична созданию элемента резервного копирования. После отмены удаления элемент сохраняется, но последующие резервные копии не запускаются.

Отмена удаления — это операция *размещения* , которая очень похожа на [изменение политики](#changing-the-policy-of-protection) и/или [Включение защиты](#enabling-protection-for-the-azure-vm). Просто укажите намерение отменить удаление с помощью переменной *исрехидрате*  в [тексте запроса](#example-request-body) и отправьте запрос. Например, чтобы отменить удаление для testVM, следует использовать следующий текст запроса.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "protectionState": "ProtectionStopped",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "isRehydrate": true
  }
}
```

Ответ будет в том же формате, что и при [активации резервного копирования по запросу](#example-responses-for-on-demand-backup). Итоговое задание должно отслеживаться, как описано в [документации по мониторингу заданий для REST API](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Дальнейшие действия

[Восстановление данных из резервной копии виртуальной машины Azure](backup-azure-arm-userestapi-restoreazurevms.md)

Дополнительные сведения о REST API Azure Backup с использованием API REST см. в следующих документах:

- [Recovery Services](/rest/api/recoveryservices/) (Службы восстановления)
- [Начало работы с Azure REST API](/rest/api/azure/)