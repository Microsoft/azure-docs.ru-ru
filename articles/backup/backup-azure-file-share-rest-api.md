---
title: Резервное копирование файловых ресурсов Azure с помощью REST API
description: Узнайте, как использовать REST API для резервного копирования файловых ресурсов Azure в хранилище служб восстановления.
ms.topic: conceptual
ms.date: 02/16/2020
ms.openlocfilehash: 8d2d8ed88da133986540a293185c8e37000ab87b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88824871"
---
# <a name="backup-azure-file-share-using-azure-backup-via-rest-api"></a>Резервное копирование файлового ресурса Azure с помощью Azure Backup через API-интерфейс.

В этой статье описывается, как создать резервную копию файлового ресурса Azure с помощью Azure Backup с помощью REST API.

В этой статье предполагается, что вы уже создали хранилище служб восстановления и политику для настройки резервного копирования для общей папки. Если вы этого не сделали, см. руководства по созданию [хранилища](./backup-azure-arm-userestapi-createorupdatevault.md) и [созданию](./backup-azure-arm-userestapi-createorupdatepolicy.md) политик REST API для создания новых хранилищ и политик.

В этой статье мы будем использовать следующие ресурсы:

- **RecoveryServicesVault**: *азурефилесваулт*

- **Политика:** *schedule1*

- **Группа ресурсов**: *azurefiles*

- **Учетная запись хранения**: *testvault2*

- **Общая папка**: *тестшаре*

## <a name="configure-backup-for-an-unprotected-azure-file-share-using-rest-api"></a>Настройка резервного копирования для незащищенного файлового ресурса Azure с помощью REST API

### <a name="discover-storage-accounts-with-unprotected-azure-file-shares"></a>Обнаружение учетных записей хранения с незащищенными файловыми ресурсами Azure

Хранилищу необходимо найти все учетные записи хранения Azure в подписке с общими папками, которые можно архивировать в хранилище служб восстановления. Это можно сделать с помощью [операции обновления](/rest/api/backup/protectioncontainers/refresh). Это асинхронная операция *POST* , которая гарантирует, что хранилище получает последний список всех незащищенных файловых ресурсов Azure в текущей подписке и кэширует их. После кэширования общей папки службы восстановления могут получить доступ к общей папке и защитить ее.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01&$filter={$filter}
```

URI Post имеет `{subscriptionId}` Параметры, `{vaultName}` , `{vaultresourceGroupName}` и `{fabricName}` . В нашем примере значения для различных параметров будут выглядеть следующим образом:

- `{fabricName}` является *Azure*

- `{vaultName}`*азурефилесваулт*

- `{vaultresourceGroupName}`*azurefiles*

- $filter = Баккупманажементтипе EQ ' AzureStorage '

Так как все необходимые параметры указаны в универсальном коде ресурса (URI), нет необходимости в отдельном тексте запроса.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

#### <a name="responses-to-the-refresh-operation"></a>Ответы на операцию обновления

Операция обновления — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Он возвращает два ответа: 202 (принято) при создании другой операции и 200 (ОК) после завершения этой операции.

##### <a name="example-responses-to-the-refresh-operation"></a>Примеры ответов на операцию обновления

После отправления запроса *POST* возвращается ответ 202 (принято).

```http
HTTP/1.1 202 Accepted
'Pragma': 'no-cache'
'Expires': '-1'
'Location': ‘https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/ResourceGroups
/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/
cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01’
'Retry-After': '60'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-client-request-id': ‘3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61’
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-reads': '11996'
'x-ms-correlation-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-routing-request-id': CENTRALUSEUAP:20200203T091326Z:6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'Date': 'Mon, 03 Feb 2020 09:13:25 GMT'
```

Отследите итоговую операцию, используя заголовок location с помощью простой команды *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01
```

После обнаружения всех учетных записей хранения Azure команда GET возвращает ответ 200 (без содержимого). Теперь хранилище может обнаружить любую учетную запись хранения с общими папками, которые можно архивировать в рамках подписки.

```http
HTTP/1.1 200 NoContent
Cache-Control  : no-cache
Pragma   : no-cache
X-Content-Type-Options  : nosniff
x-ms-request-id    : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security  : max-age=31536000; includeSubDomains
X-Powered-By    : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11933
x-ms-correlation-request-id   : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105304Z:d9bdb266-8349-4dbd-9688-de52f07648b2
Date   : Mon, 27 Jan 2020 10:53:04 GMT
```

### <a name="get-list-of-storage-accounts-with-file-shares-that-can-be-backed-up-with-recovery-services-vault"></a>Возвращает список учетных записей хранения с общими папками, которые можно архивировать с помощью хранилища служб восстановления.

Чтобы убедиться в том, что выполняется кэширование, перечислите все учетные записи хранения в подписке с общими папками, для которых можно создать резервную копию в хранилище служб восстановления. Затем найдите нужную учетную запись хранения в ответе. Это делается с помощью операции [Get протектаблеконтаинерс](/rest/api/backup/protectablecontainers/list) .

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectableContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

URI *GET* имеет все необходимые параметры. Для этой операции текст запроса также не требуется.

Пример текста ответа:

```json
{

  "value": [

    {

      "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers
 /Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/

protectableContainers/StorageContainer;Storage;AzureFiles;testvault2",

      "name": "StorageContainer;Storage;AzureFiles;testvault2",

      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectableContainers",

      "properties": {

        "friendlyName": "testvault2",

        "backupManagementType": "AzureStorage",

        "protectableContainerType": "StorageContainer",

        "healthStatus": "Healthy",

        "containerId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/
 AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2"

      }

    }

  ]

}
```

Так как мы можем разместить в тексте ответа учетную запись хранения *testvault2* с понятным именем, операция обновления, выполненная выше, была успешной. Теперь хранилище служб восстановления может успешно обнаружить учетные записи хранения с незащищенными файлами в одной подписке.

### <a name="register-storage-account-with-recovery-services-vault"></a>Регистрация учетной записи хранения в хранилище служб восстановления

Этот шаг необходим, только если вы не зарегистрировали учетную запись хранения в хранилище ранее. Хранилище можно зарегистрировать с помощью [операции протектионконтаинерс-Register](/rest/api/backup/protectioncontainers/register).

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}?api-version=2016-12-01
```

Задайте переменные для URI следующим образом:

- {resourceGroupName} — *azurefiles*
- {fabricName} — *Azure*
- {vaultName} — *азурефилесваулт*
- {containerName} — это атрибут Name в тексте ответа операции GET Протектаблеконтаинерс.
   В нашем примере это *StorageContainer; Объема AzureFiles; testvault2*

>[!NOTE]
> Всегда Возьмите атрибут имени ответа и заполните его в этом запросе. Не следует жестко кодировать или создавать формат имени контейнера. Если вы создаете или жестко назначите его, вызов API завершится ошибкой, если формат имени контейнера изменяется в будущем.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2?api-version=2016-12-01
```

Текст запроса создания выглядит следующим образом:

```json
{

 "properties": {


  "containerType": "StorageContainer",


  "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",


  "resourceGroup": "AzureFiles",


  "friendlyName": "testvault2",


  "backupManagementType": "AzureStorage"


 }
```

Полный список определений текста запроса и другие сведения см. в разделе [протектионконтаинерс-Register](/rest/api/backup/protectioncontainers/register#azurestoragecontainer).

Это асинхронная операция, которая возвращает два ответа: "202 Accepted", когда операция принята, и "200 ОК" после завершения операции.  Чтобы отвести состояние операции, используйте заголовок Location для получения последнего состояния операции.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/operationresults/1a3c8ee7-e0e5-43ed-b8b3-73cc992b6db9?api-version=2016-12-01
```

Пример текста ответа после завершения операции:

```json
{
    "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/
protectionContainers/StorageContainer;Storage;AzureFiles;testvault2",
    "name": "StorageContainer;Storage;AzureFiles;testvault2",
    "properties": {
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
        "protectedItemCount": 0,
        "friendlyName": "testvault2",
        "backupManagementType": "AzureStorage",
        "registrationStatus": "Registered",
        "healthStatus": "Healthy",
        "containerType": "StorageContainer",
        "protectableObjectType": "StorageContainer"
    }
}
```

Можно проверить успешность регистрации по значению параметра *регистратионстатус* в тексте ответа. В нашем случае он показывает состояние, зарегистрированное для *testvault2*, поэтому операция регистрации прошла успешно.

### <a name="inquire-all-unprotected-files-shares-under-a-storage-account"></a>Запрос всех незащищенных файлов, общих для учетной записи хранения

Сведения о защищенных элементах в учетной записи хранения можно запросить с помощью операции " [контейнеры защиты — запрос](/rest/api/backup/protectioncontainers/inquire) ". Это асинхронная операция, и результаты должны быть отслеживаниются с помощью заголовка Location.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/inquire?api-version=2016-12-01
```

Задайте переменные для указанного выше URI следующим образом:

- {vaultName} — *азурефилесваулт*
- {fabricName} — *Azure*
- {containerName} — см. атрибут Name в тексте ответа операции GET Протектаблеконтаинерс. В нашем примере это *StorageContainer; Объема AzureFiles; testvault2*

```http
https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/inquire?api-version=2016-12-01
```

После успешного выполнения запроса возвращается код состояния "ОК".

```http
Cache-Control : no-cache
Pragma   : no-cache
X-Content-Type-Options: nosniff
x-ms-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security: max-age=31536000; includeSubDomains
Server  : Microsoft-IIS/10.0
X-Powered-B : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11932
x-ms-correlation-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-routing-request-id   : CENTRALUSEUAP:20200127T105305Z:68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
Date  : Mon, 27 Jan 2020 10:53:05 GMT
```

### <a name="select-the-file-share-you-want-to-back-up"></a>Выберите общую папку для резервного копирования

Вы можете получить список всех защищаемых элементов в подписке и найти нужную общую папку для резервного копирования с помощью операции [Get баккуппротектаблеитемс](/rest/api/backup/backupprotectableitems/list) .

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter={$filter}
```

Создайте URI следующим образом:

- {vaultName} — *азурефилесваулт*
- {$filter}- *баккупманажементтипе EQ ' AzureStorage '*

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupProtectableItems?$filter=backupManagementType eq 'AzureStorage'&api-version=2016-12-01
```

Пример ответа:

```json
Status Code:200

{
    "value": [
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afaccount1/protectableItems/azurefileshare;azurefiles1",
            "name": "azurefileshare;azurefiles1",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afaccount1",
                "parentContainerFriendlyName": "afaccount1",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "azurefiles1",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afsaccount/protectableItems/azurefileshare;afsresource",
            "name": "azurefileshare;afsresource",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
                "parentContainerFriendlyName": "afsaccount",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "afsresource",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare",
            "name": "azurefileshare;testshare",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
                "parentContainerFriendlyName": "testvault2",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "testshare",
                "protectionState": "NotProtected"
            }
        }
    ]
}
```

Ответ содержит список всех незащищенных файловых ресурсов и содержит все сведения, необходимые службе восстановления Azure для настройки резервного копирования.

### <a name="enable-backup-for-the-file-share"></a>Включить резервное копирование для общей папки

После определения соответствующей общей папки с понятным именем выберите политику для защиты. Дополнительные сведения о существующих политиках в хранилище см. в разделе [API политики списка](/rest/api/backup/backuppolicies/list). Ссылаясь на имя политики, выберите [соответствующую политику](/rest/api/backup/protectionpolicies/get). См. дополнительные сведения о [создании политик](./backup-azure-arm-userestapi-createorupdatepolicy.md).

Включение защиты — это асинхронная операция *размещения* , которая создает "защищенный элемент".

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Задайте переменные **containername** и **протектедитемнаме** , используя атрибут ID в тексте ответа операции Get баккуппротектаблеитемс.

В нашем примере идентификатор общей папки, которую мы хотим защитить:

```output
"/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare
```

- {containername} — *storagecontainer; хранилище; azurefiles; testvault2*
- {Протектедитемнаме} — *azurefileshare; тестшаре*

Также можно обратиться к атрибуту **Name** контейнера защиты и ответам на защищенные элементы.

>[!NOTE]
>Всегда Возьмите атрибут имени ответа и заполните его в этом запросе. Не следует жестко кодировать или создать формат имени контейнера или защищенного элемента. Если вы создаете или жестко назначите его, вызов API завершится ошибкой, если в будущем изменяется формат имени контейнера или защищенного элемента.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare?api-version=2016-12-01
```

Создайте текст запроса:

Следующий текст запроса определяет свойства, необходимые для создания защищенного элемента.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupPolicies/schedule1"
  }
}
```

**Значение sourceresourceid** — это **парентконтаинерфабриЦид** в ответ на Get баккуппротектаблеитемс.

Пример ответа

Создание защищенного элемента — это асинхронная операция, которая создает другую операцию, которую необходимо отслеживанию. Он возвращает два ответа: 202 (принято) при создании другой операции и 200 (ОК) после завершения этой операции.

После отправки запроса на *Размещение* для создания или обновления защищенного элемента начальный ответ — 202 (принято) с заголовком Location.

```http
HTTP/1.1 202 Accepted
Cache-Control  : no-cache
Pragma  : no-cache
Location : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
Retry-Afte  : 60
Azure-AsyncOperation  : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
X-Content-Type-Options : nosniff
x-ms-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-client-request-id: 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security : max-age=31536000; includeSubDomains
X-Powered-By  : ASP.NET
x-ms-ratelimit-remaining-subscription-writes: 1198
x-ms-correlation-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105412Z:b55527fa-f473-4f09-b169-9cc3a7a39065
Date : Mon, 27 Jan 2020 10:54:12 GMT
```

Затем отследите итоговую операцию с помощью заголовка Location или Azure-AsyncOperation заголовка с помощью команды  *Get* .

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
```

Завершенная операция возвращает код 200 (ОК) с содержимым защищенного элемента в тексте ответа.

Образец текста ответа:

```json
{
    "id": "c3a52d1d-0853-4211-8141-477c65740264",
    "name": "c3a52d1d-0853-4211-8141-477c65740264",
    "status": "Succeeded",
    "startTime": "2020-02-03T18:10:48.296012Z",
    "endTime": "2020-02-03T18:10:48.296012Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b"
    }
}
```

Это подтверждает, что защита для общей папки включена, и первая резервная копия будет активирована в соответствии с расписанием политики.

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Активация резервной копии по запросу для файлового ресурса

После настройки общего файлового ресурса Azure резервные копии выполняются в соответствии с расписанием политики. Вы можете дождаться первого запланированного резервного копирования или активировать эту операцию по требованию в любое время.

Активация резервного копирования по запросу — это операция POST.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

{containerName} и {Протектедитемнаме} созданы ранее при включении резервного копирования. В нашем примере мы получим следующее:

```http
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/backup?api-version=2017-07-01
```

### <a name="create-request-body"></a>Создание текста запроса

Чтобы активировать резервное копирование по запросу, используйте компоненты текста запроса.

| Имя       | Type                       | Описание                       |
| ---------- | -------------------------- | --------------------------------- |
| Свойства | азурефилешаребаккупрекуес | Свойства Баккупрекуестресаурце |

Полный список определений в тексте запроса и другие сведения см. в [документации по активации резервного копирования для защищенных элементов для REST API](/rest/api/backup/backups/trigger#request-body).

Пример текста запроса

```json
{

  "properties":{

   "objectType":"AzureFileShareBackupRequest",
    "recoveryPointExpiryTimeInUTC":"2020-03-07T18:29:59.000Z"
}

}
```

### <a name="responses-to-the-on-demand-backup-operation"></a>Ответы на операцию резервного копирования по запросу

Активация резервного копирования по запросу — это [асинхронная операция](../azure-resource-manager/management/async-operations.md). Это означает, что такая операция создает другую операцию, которая должна отслеживаться отдельно.

Он возвращает два ответа: 202 (принято) при создании другой операции и 200 (ОК) после завершения этой операции.

### <a name="example-responses-to-the-on-demand-backup-operation"></a>Пример ответов на операцию резервного копирования по запросу

После отправки запроса *POST* для выполнения резервного копирования по запросу будет получен первоначальный ответ 202 (принято) с заголовками location или Azure-AsyncOperation.

```http
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationResults/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'Retry-After': '60'
'Azure-AsyncOperation': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationsStatus/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-client-request-id': 'a644712a-4895-11ea-ba57-0a580af42708, a644712a-4895-11ea-ba57-0a580af42708'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1199'
'x-ms-correlation-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T040339Z:2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'Date': 'Thu, 06 Feb 2020 04:03:38 GMT'
'Content-Length': '0'
```

Затем отследите итоговую операцию с помощью заголовка Location или Azure-AsyncOperation заголовка с помощью команды  *Get* .

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2016-12-01
```

Завершенная операция возвращает код 200 (ОК) с идентификатором результата задания резервного копирования в тексте ответа.

#### <a name="sample-response-body"></a>Образец текста ответа

```json
{
    "id": "dc62d524-427a-4093-968d-e951c0a0726e",
    "name": "dc62d524-427a-4093-968d-e951c0a0726e",
    "status": "Succeeded",
    "startTime": "2020-02-06T11:06:02.1327954Z",
    "endTime": "2020-02-06T11:06:02.1327954Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "39282261-cb52-43f5-9dd0-ffaf66beeaef"
    }
}
```

Так как задание резервного копирования — это длительная операция, оно должно отслеживаться, как описано в [документации по мониторингу заданий для REST API](./backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [восстановить файловые ресурсы Azure с помощью API-интерфейса](restore-azure-file-share-rest-api.md).
