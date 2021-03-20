---
title: Использование хранилища BLOB-объектов в качестве хранилища контрольных точек в Azure Stack Hub
description: В этой статье описывается, как использовать хранилище BLOB-объектов в качестве хранилища контрольных точек в концентраторах событий в концентраторе Azure Stack.
ms.topic: how-to
ms.date: 12/09/2020
ms.openlocfilehash: 07d7cf844480a9a88468c17cecc7ca38cca5d176
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97007829"
---
# <a name="use-blob-storage-as-checkpoint-store---event-hubs-on-azure-stack-hub"></a>Использование хранилища BLOB-объектов в качестве хранилища контрольных точек — концентраторов событий в концентраторе Azure Stack
Если вы используете хранилище BLOB-объектов Azure в качестве хранилища контрольных точек в среде, которая поддерживает другую версию пакета SDK для большого двоичного объекта хранилища, чем те, которые обычно доступны в Azure, необходимо использовать код, чтобы изменить версию API службы хранилища до определенной версии, поддерживаемой этой средой. Например, если вы используете [концентраторы событий в Azure Stack Hub версии 2002](/azure-stack/user/event-hubs-overview), самая высокая доступная версия для службы хранилища — версия 2017-11-09. В этом случае необходимо использовать код для настройки API службы хранилища до версии 2017-11-09. Пример назначения конкретной версии API хранилища см. в следующих примерах на сайте GitHub: 

- [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/)
- [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java). 
- [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsWithApiSpecificStorage.js) или  [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript/src/receiveEventsWithApiSpecificStorage.ts) 
- Python — [синхронный](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob/samples/receive_events_using_checkpoint_store_storage_api_version.py), [асинхронный](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/receive_events_using_checkpoint_store_storage_api_version_async.py)

Если вы запускаете приемник концентраторов событий, который использует хранилище BLOB-объектов в качестве хранилища контрольных точек, не нацеливание на версию, которую поддерживает Azure Stack концентратора, вы получите следующее сообщение об ошибке:

```
The value for one of the HTTP headers is not in the correct format
```


## <a name="sample-error-message-in-python"></a>Пример сообщения об ошибке в Python
Для Python ошибка `azure.core.exceptions.HttpResponseError` передается обработчику ошибок `on_error(partition_context, error)` `EventHubConsumerClient.receive()` . Но метод `receive()` не вызывает исключение. `print(error)` выводит следующие сведения об исключении:

```bash
The value for one of the HTTP headers is not in the correct format.

RequestId:f048aee8-a90c-08ba-4ce1-e69dba759297
Time:2020-03-17T22:04:13.3559296Z
ErrorCode:InvalidHeaderValue
Error:None
HeaderName:x-ms-version
HeaderValue:2019-07-07
```

Средство ведения журнала регистрирует два предупреждения, подобные следующим:

```bash
WARNING:azure.eventhub.extensions.checkpointstoreblobaio._blobstoragecsaio: 
An exception occurred during list_ownership for namespace '<namespace-name>.eventhub.<region>.azurestack.corp.microsoft.com' eventhub 'python-eh-test' consumer group '$Default'. 

Exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07')

WARNING:azure.eventhub.aio._eventprocessor.event_processor:EventProcessor instance '26d84102-45b2-48a9-b7f4-da8916f68214' of eventhub 'python-eh-test' consumer group '$Default'. An error occurred while load-balancing and claiming ownership. 

The exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07'). Retrying after 71.45254944090853 seconds
```



## <a name="next-steps"></a>Дальнейшие действия

Сведения о секционировании и создании контрольных точек см. в следующей статье: [Балансировка нагрузки секций между несколькими экземплярами приложения](event-processor-balance-partition-load.md)
