---
title: Доступность и согласованность в Центрах событий Azure | Документация Майкрософт
description: Узнайте, как обеспечить максимальную доступность и согласованность в Центрах событий Azure с помощью секций.
ms.topic: article
ms.date: 03/15/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 6cd446cf86c22b851bae9cb9d8535a8e5234e08b
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104722537"
---
# <a name="availability-and-consistency-in-event-hubs"></a>Доступность и согласованность в Центрах событий
В этой статье содержатся сведения о доступности и согласованности, поддерживаемые концентраторами событий Azure. 

## <a name="availability"></a>Доступность
Концентраторы событий Azure распределяют риск разрушительных сбоев отдельных компьютеров или даже полных стоек в кластерах, охватывающих несколько доменов сбоя в центре обработки данных. Он реализует прозрачное обнаружение сбоев и механизмы отработки отказа таким способом, чтобы служба продолжала работать в пределах проверенных уровней обслуживания и обычно без заметного перерыва при возникновении таких сбоев. 

Если пространство имен концентраторов событий было создано с включенными [зонами доступности](../availability-zones/az-overview.md) , риск сбоя будет распределяться по трем физически разделенным средствам, а служба имеет достаточно резервных копий, чтобы мгновенно справиться с полной, катастрофической потерей всей ситуации. Дополнительные сведения см. в статье Служба [концентраторов событий Azure: географическое аварийное восстановление](event-hubs-geo-dr.md).

Когда клиентское приложение отправляет события в концентратор событий, не указывая секцию, события автоматически распределяются между секциями в концентраторе событий. Если Секция по какой-либо причине недоступна, события распределяются между остальными секциями. Такое поведение обеспечивает наивысший показатель времени непрерывной работы. Для вариантов использования, требующих максимального времени, рекомендуется использовать эту модель вместо отправки событий в определенную секцию. 

## <a name="consistency"></a>Согласованность
В некоторых сценариях важным может быть упорядочение событий. Например, вам может потребоваться, чтобы серверная система обрабатывала команду обновления перед командой удаления. В этом сценарии клиентское приложение отправляет события в определенную секцию, чтобы их порядок сохранялся. Когда приложение-потребитель использует эти события из раздела, они считываются по порядку. 

Применяя такую конфигурацию, учитывайте, что в случае недоступности определенной секции, в которую выполняется отправка, вы получите сообщение об ошибке. В качестве точки сравнения, если у вас нет сходства с одной секцией, служба концентраторов событий отправляет событие в следующий доступный раздел.

Таким образом, если высокий уровень доступности наиболее важен, не следует ориентироваться на конкретную секцию (с помощью идентификатора секции или ключа). Использование идентификатора секции или ключа понижает уровень доступности концентратора событий до уровня раздела. В этом сценарии вы выбираете явный вариант доступности (без идентификатора или ключа секции) и согласованности (закрепление событий в определенной секции). Подробные сведения о секциях в концентраторах событий см. в разделе [partitions](event-hubs-features.md#partitions).

## <a name="appendix"></a>Приложение

### <a name="send-events-without-specifying-a-partition"></a>Отправка событий без указания секции
Рекомендуется отправлять события в концентратор событий, не устанавливая сведения о секции, чтобы служба концентраторов событий не сбалансировать нагрузку между секциями. Ознакомьтесь со следующими краткими руководствами по началу работы с различными языками программирования. 

- [Отправка событий с помощью .NET](event-hubs-dotnet-standard-getstarted-send.md)
- [Отправка событий с помощью Java](event-hubs-java-get-started-send.md)
- [Отправка событий с помощью JavaScript](event-hubs-python-get-started-send.md)
- [Отправка событий с помощью Python](event-hubs-python-get-started-send.md)


### <a name="send-events-to-a-specific-partition"></a>Отправка событий в определенную секцию
В этом разделе вы узнаете, как отправить события в определенную секцию с помощью различных языков программирования. 

### <a name="net"></a>[.NET](#tab/dotnet)
Чтобы отправить события в определенную секцию, создайте пакет с помощью метода [евенсубпродуцерклиент. креатебатчасинк](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.createbatchasync#Azure_Messaging_EventHubs_Producer_EventHubProducerClient_CreateBatchAsync_Azure_Messaging_EventHubs_Producer_CreateBatchOptions_System_Threading_CancellationToken_) , указав либо либо `PartitionId` `PartitionKey` в [креатебатчоптионс](/dotnet/api/azure.messaging.eventhubs.producer.createbatchoptions?view=azure-dotnet). Следующий код отправляет пакет событий в определенную секцию, указывая ключ секции. Концентраторы событий гарантируют, что все события, совместно использующие значение ключа секции, хранятся вместе и доставляются в порядке поступления.

```csharp
var batchOptions = new CreateBatchOptions { PartitionKey = "cities" };
using var eventBatch = await producer.CreateBatchAsync(batchOptions);
```

Можно также использовать метод [евенсубпродуцерклиент. SendAsync](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.sendasync#Azure_Messaging_EventHubs_Producer_EventHubProducerClient_SendAsync_System_Collections_Generic_IEnumerable_Azure_Messaging_EventHubs_EventData__Azure_Messaging_EventHubs_Producer_SendEventOptions_System_Threading_CancellationToken_) , указав в [сендевентоптионс](/dotnet/api/azure.messaging.eventhubs.producer.sendeventoptions)значение **PartitionId** или **PartitionKey** .

```csharp
var sendEventOptions  = new SendEventOptions { PartitionKey = "cities" };
// create the events array
producer.SendAsync(events, sendOptions)
```


### <a name="java"></a>[Java](#tab/java)
Чтобы отправить события в определенную секцию, создайте пакет с помощью метода [createBatch](/java/api/com.azure.messaging.eventhubs.eventhubproducerclient.createbatch) , указав **идентификатор секции** или **ключ секции** в [креатебатчоптионс](/java/api/com.azure.messaging.eventhubs.models.createbatchoptions). Следующий код отправляет пакет событий в определенную секцию, указывая ключ секции. 

```java
CreateBatchOptions batchOptions = new CreateBatchOptions();
batchOptions.setPartitionKey("cities");
```

Можно также использовать метод [евенсубпродуцерклиент. Send](/java/api/com.azure.messaging.eventhubs.eventhubproducerclient.send#com_azure_messaging_eventhubs_EventHubProducerClient_send_java_lang_Iterable_com_azure_messaging_eventhubs_EventData__com_azure_messaging_eventhubs_models_SendOptions_) , указав **идентификатор секции** или **ключ секции** в [сендоптионс](/java/api/com.azure.messaging.eventhubs.models.sendoptions).

```java
List<EventData> events = Arrays.asList(new EventData("Melbourne"), new EventData("London"), new EventData("New York"));
SendOptions sendOptions = new SendOptions();
sendOptions.setPartitionKey("cities");
producer.send(events, sendOptions);
```


### <a name="python"></a>[Python](#tab/python) 
Чтобы отправить события в определенную секцию, при создании пакета с помощью [`EventHubProducerClient.create_batch`](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#create-batch---kwargs-) метода укажите `partition_id` или `partition_key` . Затем используйте метод, [`EventHubProducerClient.send_batch`](/python/api/azure-eventhub/azure.eventhub.aio.eventhubproducerclient#send-batch-event-data-batch--typing-union-azure-eventhub--common-eventdatabatch--typing-list-azure-eventhub-) чтобы отправить пакет в секцию концентратора событий. 

```python
event_data_batch = await producer.create_batch(partition_key='cities')
```

Можно также использовать метод [EventHubProducerClient.send_batch](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#send-batch-event-data-batch----kwargs-) , указав значения для `partition_id` `partition_key` параметров или.

```python
producer.send_batch(event_data_batch, partition_key="cities")
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)
Чтобы отправить события в определенную секцию, [Создайте пакет](/javascript/api/@azure/event-hubs/eventhubproducerclient#createBatch_CreateBatchOptions_) с помощью объекта [евенсубпродуцерклиент. креатебатчоптионс](/javascript/api/@azure/event-hubs/eventhubproducerclient#createBatch_CreateBatchOptions_) , указав `partitionId` или `partitionKey` . Затем отправьте пакет в концентратор событий с помощью метода [евенсубпродуцерклиент. SendBatch](/javascript/api/@azure/event-hubs/eventhubproducerclient#sendBatch_EventDataBatch__OperationOptions_) . 

См. следующий пример.

```javascript
const batchOptions = { partitionKey = "cities"; };
const batch = await producer.createBatch(batchOptions);
```

Можно также использовать метод [евенсубпродуцерклиент. sendBatch](/javascript/api/@azure/event-hubs/eventhubproducerclient#sendBatch_EventData____SendBatchOptions_) , указав **идентификатор секции** или **ключ секции** в [сендбатчоптионс](/javascript/api/@azure/event-hubs/sendbatchoptions).

```javascript
const sendBatchOptions = { partitionKey = "cities"; };
// prepare events
producer.sendBatch(events, sendBatchOptions);
```

---



## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о Центрах событий см. в следующих источниках:

- [Обзор службы Центров событий](./event-hubs-about.md)
- [Термины, касающиеся Концентраторов событий](event-hubs-features.md)
