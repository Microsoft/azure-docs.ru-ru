---
title: Состояние асинхронных операций
description: Описание действий по отслеживанию асинхронных операций в Azure. Приводятся значения для получения состояния длительной операции.
ms.topic: conceptual
ms.date: 08/21/2020
ms.custom: seodec18
ms.openlocfilehash: e2c5ba137d5277466cf1b382d2b0b1bc02259f00
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88723458"
---
# <a name="track-asynchronous-azure-operations"></a>Отслеживание асинхронных операций Azure

Некоторые операции REST выполняются в Azure асинхронно, поскольку не могут быть быстро завершены. Из этой статьи вы узнаете, как отслеживать состояние асинхронных операций, используя возвращаемые в ответе значения.  

## <a name="status-codes-for-asynchronous-operations"></a>Коды состояния для асинхронных операций

Изначально асинхронная операция возвращает один из следующих кодов состояния HTTP:

* 201 Created (создано);
* 202 Accepted (принято).

После успешного завершения операции возвращаются следующие коды:

* 200 OK;
* 204 No Content (нет содержимого).

Ответы по выполняемой операции описаны в разделе [Документация по REST API](/rest/api/azure/).

После получения кода ответа 201 или 202 вы можете отслеживать состояние операции.

## <a name="url-to-monitor-status"></a>URL-адрес для отслеживания состояния

Существует два разных способа мониторинга состояния асинхронной операции. Правильный подход определяется путем проверки значений заголовков, возвращенных исходным запросом. Сначала найдите:

* `Azure-AsyncOperation` — это URL-адрес для проверки текущего состояния операции. Если операция возвращает это значение, используйте его для наблюдения за состоянием операции.
* `Retry-After` — это количество секунд ожидания перед проверкой состояния асинхронной операции.

Если `Azure-AsyncOperation` не является одним из значений заголовка, найдите:

* `Location` — это URL-адрес для определения момента завершения операции. Это значение следует использовать только в том случае, если Azure-AsyncOperation не возвращается.
* `Retry-After` — это количество секунд ожидания перед проверкой состояния асинхронной операции.

## <a name="azure-asyncoperation-request-and-response"></a>Запрос и ответ с использованием Azure-AsyncOperation

Если у вас есть URL-адрес из `Azure-AsyncOperation` значения заголовка, отправьте запрос GET по этому URL-адресу. Используйте значение из `Retry-After` , чтобы запланировать частоту проверки состояния. Вы получите объект ответа, который указывает состояние операции. При проверке состояния операции с `Location` URL-адресом возвращается другой ответ. Дополнительные сведения о ответе URL-адреса расположения см. в статье [Создание учетной записи хранения (202 с расположением и повторными попытками)](#create-storage-account-202-with-location-and-retry-after).

Свойства ответа могут быть различными, но всегда включать состояние асинхронной операции.

```json
{
    "status": "{status-value}"
}
```

В следующем примере показаны другие значения, которые могут быть возвращены операцией:

```json
{
    "id": "{resource path from GET operation}",
    "name": "{operation-id}",
    "status" : "Succeeded | Failed | Canceled | {resource provider values}",
    "startTime": "2017-01-06T20:56:36.002812+00:00",
    "endTime": "2017-01-06T20:56:56.002812+00:00",
    "percentComplete": {double between 0 and 100 },
    "properties": {
        /* Specific resource provider values for successful operations */
    },
    "error" : {
        "code": "{error code}",  
        "message": "{error description}"
    }
}
```

Объект error возвращается в том случае, если операция находится в состоянии сбоя (Failed) или отмены (Canceled). Все остальные значения являются необязательными. Ответ, который вы получаете, может отличаться от примера.

## <a name="provisioningstate-values"></a>Значения provisioningState

Операции, которые создают, обновляют или удаляют ресурсы (PUT, PATCH, DELETE), обычно возвращают значение `provisioningState`. После завершения операции возвращается одно из следующих трех значений:

* Выполнено
* Сбой
* Отменено

Любое другое значение означает, что операция еще выполняется. Поставщик ресурсов может возвращать настраиваемое значение, указывающее его состояние. Например, значение **Accepted** (Принято) может сигнализировать о том, что запрос успешно получен и выполняется.

## <a name="example-requests-and-responses"></a>Примеры запросов и ответов

### <a name="start-virtual-machine-202-with-azure-asyncoperation"></a>Запуск виртуальной машины (код 202, значение Azure-AsyncOperation)

В этом примере показано, как определить состояние [операции запуска для виртуальных машин](/rest/api/compute/virtualmachines/start). Исходный запрос имеет следующий формат:

```HTTP
POST 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2019-12-01
```

В ответ получен код состояния 202. Среди полученных в заголовке значений есть такое:

```HTTP
Azure-AsyncOperation : https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2019-12-01
```

Чтобы проверить состояние асинхронной операции, отправьте запрос на полученный URL-адрес.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2019-12-01
```

Текст ответа содержит состояние операции:

```json
{
  "startTime": "2017-01-06T18:58:24.7596323+00:00",
  "status": "InProgress",
  "name": "9a062a88-e463-4697-bef2-fe039df73a02"
}
```

### <a name="deploy-resources-201-with-azure-asyncoperation"></a>Развертывание ресурсов (код 201, значение Azure-AsyncOperation)

В этом примере показано, как определить состояние [операции развертывания для развертывания ресурсов](/rest/api/resources/deployments/createorupdate) в Azure. Исходный запрос имеет следующий формат:

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2020-06-01
```

В ответ получен код состояния 201. Текст ответа включает следующие данные:

```json
"provisioningState":"Accepted",
```

Среди полученных в заголовке значений есть такое:

```HTTP
Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2020-06-01
```

Чтобы проверить состояние асинхронной операции, отправьте запрос на полученный URL-адрес.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2020-06-01
```

Текст ответа содержит состояние операции:

```json
{
    "status": "Running"
}
```

После завершения развертывания возвращается ответ с такими данными:

```json
{
    "status": "Succeeded"
}
```

### <a name="create-storage-account-202-with-location-and-retry-after"></a>Создание учетной записи хранения (код 202, значения Location и Retry-After)

В этом примере показано, как определить состояние [операции создания для учетных записей хранения](/rest/api/storagerp/storageaccounts/create). Исходный запрос имеет следующий формат:

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2019-06-01
```

Кроме того, текст запроса содержит свойства учетной записи хранения:

```json
{
    "location": "South Central US",
    "properties": {},
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage"
}
```

В ответ получен код состояния 202. Среди полученных в заголовке значений есть такие:

```HTTP
Location: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2019-06-01
Retry-After: 17
```

Подождав указанное в значении Retry-After количество секунд, проверьте состояние асинхронной операции посредством отправки еще одного запроса на указанный URL-адрес.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2019-06-01
```

Если запрос все еще выполняется, вы получите код состояния 202. Если запрос завершен, вы получите код состояния 200, а в тексте ответа будут указаны свойства созданной учетной записи хранения.

## <a name="next-steps"></a>Дальнейшие действия

* Документацию по каждой операции REST см. в [справочнике по REST API](/rest/api/azure/).
* Сведения о развертывании шаблонов с помощью REST API диспетчер ресурсов см. в статье [развертывание ресурсов с помощью шаблонов диспетчер ресурсов и диспетчер ресурсов REST API](../templates/deploy-rest.md).