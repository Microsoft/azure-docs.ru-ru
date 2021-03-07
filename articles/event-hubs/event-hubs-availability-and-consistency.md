---
title: Доступность и согласованность в Центрах событий Azure | Документация Майкрософт
description: Узнайте, как обеспечить максимальную доступность и согласованность в Центрах событий Azure с помощью секций.
ms.topic: article
ms.date: 01/25/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 325cc80daba2a44dedbd5e09ac4858ae2815c1cd
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102425929"
---
# <a name="availability-and-consistency-in-event-hubs"></a>Доступность и согласованность в Центрах событий
В этой статье содержатся сведения о доступности и согласованности, поддерживаемые концентраторами событий Azure. 

## <a name="availability"></a>Доступность
Концентраторы событий Azure распределяют риск разрушительных сбоев отдельных компьютеров или даже полных стоек в кластерах, охватывающих несколько доменов сбоя в центре обработки данных. Он реализует прозрачное обнаружение сбоев и механизмы отработки отказа таким способом, чтобы служба продолжала работать в пределах проверенных уровней обслуживания и обычно без заметного перерыва при возникновении таких сбоев. 

Если пространство имен концентраторов событий было создано с включенными [зонами доступности](../availability-zones/az-overview.md) , риск сбоя будет распределяться по трем физически разделенным средствам, а служба имеет достаточно резервных копий, чтобы мгновенно справиться с полной, катастрофической потерей всей ситуации. Дополнительные сведения см. в статье Служба [концентраторов событий Azure: географическое аварийное восстановление](event-hubs-geo-dr.md).

Когда клиентское приложение отправляет события в концентратор событий, не указывая секцию, события автоматически распределяются между секциями в концентраторе событий. Если Секция по какой-либо причине недоступна, события распределяются между остальными секциями. Такое поведение обеспечивает наивысший показатель времени непрерывной работы. Для вариантов использования, требующих максимального времени, рекомендуется использовать эту модель вместо отправки событий в определенную секцию. 

### <a name="availability-considerations-when-using-a-partition-id-or-key"></a>Рекомендации по доступности при использовании идентификатора или ключа секции
Использование идентификатора секции или ключа секции является необязательным. Тщательно продумайте, следует ли использовать один из них. Если при публикации события не указать идентификатор или ключ секции, концентраторы событий распределяют нагрузку между секциями. При использовании идентификатора или ключа секции для этих секций требуется доступность на одном узле, а сбои могут возникать с течением времени. Например, может потребоваться перезагрузка или исправление для вычислений узлов. Таким образом, если задать идентификатор или ключ секции и эта Секция становится недоступной по какой-либо причине, попытка доступа к данным в этом разделе завершится ошибкой. Если высокая доступность важна, не указывайте идентификатор или ключ секции. В этом случае события отправляются в секции с помощью внутреннего алгоритма балансировки нагрузки. В этом сценарии вы выбираете явный вариант доступности (без идентификатора или ключа секции) и согласованности (закрепление событий в определенной секции). Использование идентификатора секции или ключа понижает уровень доступности концентратора событий до уровня раздела. 

### <a name="availability-considerations-when-handling-delays-in-processing-events"></a>Рекомендации по доступности при обработке задержек при обработке событий
Еще один вопрос заключается в том, что потребительское приложение обрабатывает задержки при обработке событий. В некоторых случаях приложение-потребитель может лучше удалить данные и повторить попытку, а не пытаться обрабатываться, что может привести к дальнейшим задержкам обработки. Например, с тикером котировок лучше подождать, пока появятся актуальные данные, но в чате реального времени или в сценарии VOIP лучше быстрее получить данные, даже если они не полные.

Учитывая эти рекомендации по доступности, в этих сценариях приложение-потребитель может выбрать одну из следующих стратегий обработки ошибок:

- Прерывать (прерывать чтение из концентратора событий до устранения проблем)
- удаление (удалите ненужные сообщения);
- повторная попытка (обработайте сообщение повторно по своему усмотрению);


## <a name="consistency"></a>Согласованность
В некоторых сценариях важным может быть упорядочение событий. Например, вам может потребоваться, чтобы серверная система обрабатывала команду обновления перед командой удаления. В этом сценарии клиентское приложение отправляет события в определенную секцию, чтобы их порядок сохранялся. Когда приложение-потребитель использует эти события из раздела, они считываются по порядку. 

Применяя такую конфигурацию, учитывайте, что в случае недоступности определенной секции, в которую выполняется отправка, вы получите сообщение об ошибке. В качестве точки сравнения, если у вас нет сходства с одной секцией, служба концентраторов событий отправляет событие в следующий доступный раздел.


## <a name="appendix"></a>Приложение

### <a name="send-events-to-a-specific-partition"></a>Отправка событий в определенную секцию
В этом разделе показано, как отправить события в определенную секцию с помощью C#, Java, Python и JavaScript. 

### <a name="net"></a>[.NET](#tab/dotnet)
Полный пример кода, показывающий, как отправить пакет событий в концентратор событий (без задания идентификатора секции или ключа), см. в статье [Отправка событий в концентраторы событий Azure и получение событий с помощью .NET (Azure. Messaging. EventHubs)](event-hubs-dotnet-standard-getstarted-send.md).

Чтобы отправить события в определенную секцию, создайте пакет с помощью метода [евенсубпродуцерклиент. креатебатчасинк](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.createbatchasync#Azure_Messaging_EventHubs_Producer_EventHubProducerClient_CreateBatchAsync_Azure_Messaging_EventHubs_Producer_CreateBatchOptions_System_Threading_CancellationToken_) , указав либо либо `PartitionId` `PartitionKey` в [креатебатчоптионс](//dotnet/api/azure.messaging.eventhubs.producer.createbatchoptions). Следующий код отправляет пакет событий в определенную секцию, указывая ключ секции. 

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
Полный пример кода, демонстрирующий, как отправить пакет событий в концентратор событий (без задания идентификатора секции или ключа), см. в статье [Использование Java для отправки событий в концентраторы событий Azure (Azure-Messaging-eventhubs) и получения событий из](event-hubs-java-get-started-send.md)них.

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
Полный пример кода, показывающий, как отправить пакет событий в концентратор событий (без задания идентификатора секции или ключа), см. в статье [Отправка событий в концентраторы событий или получение событий из него с помощью Python (Azure-eventhub)](event-hubs-python-get-started-send.md).

Чтобы отправить события в определенную секцию, при создании пакета с помощью [`EventHubProducerClient.create_batch`](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#create-batch---kwargs-) метода укажите `partition_id` или `partition_key` . Затем используйте метод, [`EventHubProducerClient.send_batch`](/python/api/azure-eventhub/azure.eventhub.aio.eventhubproducerclient#send-batch-event-data-batch--typing-union-azure-eventhub--common-eventdatabatch--typing-list-azure-eventhub-) чтобы отправить пакет в секцию концентратора событий. 

```python
event_data_batch = await producer.create_batch(partition_key='cities')
```

Можно также использовать метод [EventHubProducerClient.send_batch](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#send-batch-event-data-batch----kwargs-) , указав значения для `partition_id` `partition_key` параметров или.

```python
producer.send_batch(event_data_batch, partition_key="cities")
```


### <a name="javascript"></a>[JavaScript](#tab/javascript)
Полный пример кода, показывающий, как отправить пакет событий в концентратор событий (без задания идентификатора секции или ключа), см. в разделе [Отправка событий в концентраторы событий или получение событий с помощью JavaScript (концентраторы событий и Azure)](event-hubs-node-get-started-send.md).

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

* [Обзор службы Центров событий](./event-hubs-about.md)
* [Создание концентратора событий](event-hubs-create.md)
