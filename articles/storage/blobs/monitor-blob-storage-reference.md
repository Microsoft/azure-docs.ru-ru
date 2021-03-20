---
title: Справочник по данным мониторинга хранилища BLOB-объектов Azure | Документация Майкрософт
description: Справочник по журналам и метрикам для мониторинга данных из хранилища BLOB-объектов Azure.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 10/02/2020
ms.author: normesta
ms.subservice: logs
ms.custom: subject-monitoring
ms.openlocfilehash: b37586f66106e33b2a8dad034a6e7c131484be73
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100571618"
---
# <a name="azure-blob-storage-monitoring-data-reference"></a>Справочник по данным мониторинга хранилища BLOB-объектов Azure

Сведения о сборе и анализе данных мониторинга для службы хранилища Azure см. на странице [Мониторинг службы хранилища Azure](monitor-blob-storage.md).

## <a name="metrics"></a>Метрики

В следующих таблицах перечислены метрики платформы, собранные для службы хранилища Azure: 

### <a name="capacity-metrics"></a>Метрики емкости

Значения метрик емкости обновляются ежедневно (до 24 часов). Интервал времени определяет интервал, за который представлены значения метрик. Поддерживаемый интервал времени для всех метрик емкости — один час (PT1H).

Служба хранилища Azure предоставляет следующие метрики емкости в Azure Monitor.

#### <a name="account-level"></a>На уровне учетной записи

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="blob-storage"></a>Хранилище BLOB-объектов

В этой таблице показаны [метрики хранилища BLOB-объектов](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsblobservices).

| Метрика | Описание |
| ------------------- | ----------------- |
| BlobCapacity | Объем хранилища BLOB-объектов, используемый в учетной записи хранения. <br/><br/> Единица измерения: Байты <br/> Тип агрегирования: Среднее <br/> Пример значения. 1024 <br/> Размеры: **BlobType** и **BlobTier** ([определение](#metrics-dimensions)) |
| BlobCount    | Количество больших двоичных объектов, хранящихся в учетной записи хранения. <br/><br/> Единица измерения: Count <br/> Тип агрегирования: Среднее <br/> Пример значения. 1024 <br/> Размеры: **BlobType** и **BlobTier** ([определение](#metrics-dimensions)) |
| блобпровисионедсизе | Объем хранилища, подготовленного в учетной записи хранения. Эта метрика применима только к учетным записям хранения класса Premium. <br/><br/> Единица: байт <br/> Тип агрегирования: Среднее |
| ContainerCount    | Число контейнеров в учетной записи хранения. <br/><br/> Единица измерения: Count <br/> Тип агрегирования: Среднее <br/> Пример значения. 1024 |
| IndexCapacity     | Объем хранилища, используемый для индекса ADLS 2-го поколения (иерархического). <br/><br/> Единица измерения: Байты <br/> Тип агрегирования: Среднее <br/> Пример значения. 1024 |

### <a name="transaction-metrics"></a>Метрики транзакций

Метрики транзакций создаются при каждом запросе к учетной записи хранения, отправляемом из службы хранилища Azure в Azure Monitor. Если учетная запись хранения неактивна, данные на основе метрик транзакций за этот период будут отсутствовать. Все метрики транзакций доступны на уровне службы учетной записи и хранилища BLOB-объектов. Интервал времени определяет интервал, за который представлены значения метрик. Поддерживаемые интервалы времени для всех метрик транзакций — PT1H и PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Измерения метрик

Служба хранилища Azure поддерживает следующие измерения метрик в Azure Monitor.

### <a name="dimensions-available-to-all-storage-services"></a>Измерения, доступные для всех служб хранилища

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

### <a name="dimensions-specific-to-blob-storage"></a>Измерения, относящиеся к хранилищу BLOB-объектов

| Название измерения | Описание |
| ------------------- | ----------------- |
| **BlobType** | Тип большого двоичного объекта только для метрик больших двоичных объектов. Поддерживаемые значения: **BlockBlob**, **PageBlob** и **Azure Data Lake Storage**. Добавочные BLOB-объекты включаются в **BlockBlob**. |
| **BlobTier** | Служба хранилища Azure предоставляет различные уровни доступа, которые позволяют хранить данные больших двоичных объектов наиболее экономичным способом. Дополнительные сведения об уровнях большого двоичного объекта в службе хранилища Azure см. на [этой странице](../blobs/storage-blob-storage-tiers.md). Поддерживаются следующие значения: <br/> <li>**Горячий**: горячий уровень доступа</li> <li>**Холодный**: холодный уровень доступа</li> <li>**Архивирование**. Архивный уровень доступа</li> <li>**Премиум**: уровень "Премиум" для блочного BLOB-объекта</li> <li>**P4/P6/P10/P15/P20/P30/P40/P50/P60**: типы уровней для страничного BLOB-объекта категории "Премиум"</li> <li>**Стандартный**. Тип уровня для страничного BLOB-объекта (цен. категория "Стандартный")</li> <li>**Необусловленный**: тип уровня для учетной записи хранения общего назначения версии 1</li> |

Для метрик с поддержкой измерений необходимо указать значение измерения, чтобы увидеть соответствующие значения метрик. Например, если вы хотите просмотреть значения **Transactions** для поиска успешных ответов, необходимо выполнить фильтрацию по измерению **ResponseType** со значением **Success**. Если взглянуть на значение **BlobCount** для блочного BLOB-объекта, необходимо отфильтровать измерение **BlobType** с **BlockBlob**.

## <a name="resource-logs-preview"></a>Журналы ресурсов (предварительная версия)

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей, таблиц, учетных записей хранения категории "Премиум" общего назначения версии 1 и учетных записей хранения общего назначения версии 2. Классические учетные записи хранения не поддерживаются.

В следующей таблице перечислены свойства журналов ресурсов службы хранилища Azure при их сборе в журналах Azure Monitor или службе хранилища Azure. Свойства содержат сведения об операции, службе и типе авторизации, которые использовались для выполнения операции.

### <a name="fields-that-describe-the-operation"></a>Поля со сведениями об операции

```json
{
    "time": "2019-02-28T19:10:21.2123117Z",
    "resourceId": "/subscriptions/12345678-2222-3333-4444-555555555555/resourceGroups/mytestrp/providers/Microsoft.Storage/storageAccounts/testaccount1/blobServices/default",
    "category": "StorageWrite",
    "operationName": "PutBlob",
    "operationVersion": "2017-04-17",
    "schemaVersion": "1.0",
    "statusCode": 201,
    "statusText": "Success",
    "durationMs": 5,
    "callerIpAddress": "192.168.0.1:11111",
    "correlationId": "ad881411-201e-004e-1c99-cfd67d000000",
    "location": "uswestcentral",
    "uri": "http://mystorageaccount.blob.core.windows.net/cont1/blobname?timeout=10"
}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Поля, описывающие аутентификацию операции

```json
{
    "identity": {
        "authorization": [
            {
                "action": "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read",
                "principals": [
                    {
                        "id": "fde5ba15-4355-4223-b811-cccccccccccc",
                        "type": "User"
                    }
                ],
                "roleAssignmentId": "ecf75cb8-491c-4a25-ad6e-aaaaaaaaaaaa",
                "roleDefinitionId": "b7e6dc6d-f1e8-4753-8033-ffffffffffff"
            }
        ],
        "requester": {
            "appId": "691458b9-1327-4635-9f55-bbbbbbbbbbbb",
            "audience": "https://storage.azure.com/",
            "objectId": "fde5ba15-4355-4223-b811-cccccccccccc",
            "tenantId": "72f988bf-86f1-41af-91ab-dddddddddddd",
            "tokenIssuer": "https://sts.windows.net/72f988bf-86f1-41af-91ab-eeeeeeeeeeee/"
           },
        "type": "OAuth"
    },
}

```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Поля со сведениями о службе

```json
{
    "properties": {
        "accountName": "testaccount1",
        "requestUrl": "https://testaccount1.blob.core.windows.net:443/upload?restype=container&comp=list&prefix=&delimiter=%2F&marker=&maxresults=30&include=metadata&_=1551405598426",
        "userAgentHeader": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/17.17134",
        "referrerHeader": "blob:https://ms.portal.azure.com/6f50025f-3b88-488d-b29e-3c592a31ddc9",
        "clientRequestId": "",
        "etag": "",
        "serverLatencyMs": 63,
        "serviceType": "blob",
        "operationCount": 0,
        "requestHeaderSize": 2658,
        "requestBodySize": 0,
        "responseHeaderSize": 295,
        "responseBodySize": 2018,
        "contentLengthHeader": 0,
        "requestMd5": "",
        "serverMd5": "",
        "lastModifiedTime": "",
        "conditionsUsed": "",
        "smbTreeConnectID" : "0x3",
        "smbPersistentHandleID" : "0x6003f",
        "smbVolatileHandleID" : "0xFFFFFFFF00000065",
        "smbMessageID" : "0x3b165",
        "smbCreditsConsumed" : "0x3",
        "smbCommandDetail" : "0x2000 bytes at offset 0xf2000",
        "smbFileId" : " 0x9223442405598953",
        "smbSessionID" : "0x8530280128000049",
        "smbCommandMajor" : "0x6",
        "smbCommandMinor" : "DirectoryCloseAndDelete"
    }

}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>См. также раздел

- Описание мониторинга службы хранилища Azure см. на [этой странице](monitor-blob-storage.md).
- Подробные сведения о мониторинге ресурсов Azure см. в статье [Мониторинг ресурсов Azure с помощью Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
