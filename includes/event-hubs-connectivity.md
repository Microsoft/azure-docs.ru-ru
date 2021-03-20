---
title: включить файл
description: включить файл
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 01/21/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 44afd8ea4ef2ab06ec31b7528e9776faebc3b4dc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98689942"
---
### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>Какие порты нужно открыть в брандмауэре? 
Для отправки и получения событий можно использовать следующие протоколы с концентраторами событий Azure:

- Расширенный протокол управления очередью сообщений 1,0 (AMQP)
- Протокол HTTP 1,1 с TLS (HTTPS)
- Apache Kafka

В следующей таблице приведены исходящие порты, которые необходимо открыть для использования этих протоколов в целях взаимодействия с концентраторами событий Azure. 

| Протокол | порты; | Сведения | 
| -------- | ----- | ------- | 
| AMQP | 5671 и 5672 | См. [Руководство по использованию протокола AMQP](../articles/service-bus-messaging/service-bus-amqp-protocol-guide.md). | 
| HTTPS | 443 | Этот порт используется для HTTP/REST API и для сокетов AMQP-over. |
| Kafka | 9093 | См. [Использование Центров событий Azure из приложений Apache Kafka](../articles/event-hubs/event-hubs-for-kafka-ecosystem-overview.md).

Порт HTTPS необходим для исходящей связи, если AMQP используется через порт 5671, так как некоторые операции управления, выполняемые клиентскими пакетами SDK, и получение маркеров от Azure Active Directory (при использовании) выполняются по протоколу HTTPS. 

Официальные пакеты SDK для Azure обычно используют протокол AMQP для отправки и получения событий из концентраторов событий. Параметр протокола AMQP-over-WebSockets выполняется через порт TCP 443, как и API HTTP, но в противном случае он функционально идентичен обычному AMQP. Этот параметр имеет более высокую начальную задержку при подключении из-за дополнительных циклов подтверждения соединения и немного больше ресурсов, чем компромисс для совместного использования HTTPS-порта. Если выбран этот режим, для обмена данными достаточно TCP-порта 443. Следующие параметры позволяют выбрать режим AMQP или AMQP WebSockets.

| Язык | Параметр   |
| -------- | ----- |
| .NET     | Свойство [евенсубконнектионоптионс. TransportType](/dotnet/api/azure.messaging.eventhubs.eventhubconnectionoptions.transporttype) с [евенсубстранспорттипе. амкпткп](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) или [евенсубстранспорттипе. AmqpWebSockets](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) |
| Java     | [com. Microsoft. Azure. eventhubs. евентпроцессорклиентбуилдер. transporttype](/java/api/com.azure.messaging.eventhubs.eventprocessorclientbuilder.transporttype) с [амкптранспорттипе. AMQP](/java/api/com.azure.core.amqp.amqptransporttype) или [AmqpTransportType.AMQP_WEB_SOCKETS](/java/api/com.azure.core.amqp.amqptransporttype) |
| Узел  | [Евенсубконсумерклиентоптионс](/javascript/api/@azure/event-hubs/eventhubconsumerclientoptions) имеет `webSocketOptions` свойство. |
| Python | [EventHubConsumerClient.transport_type](/python/api/azure-eventhub/azure.eventhub.eventhubconsumerclient) с [TransportType. AMQP](/python/api/azure-eventhub/azure.eventhub.transporttype) или [TransportType. амкповервебсоккет](/python/api/azure-eventhub/azure.eventhub.transporttype) |

### <a name="what-ip-addresses-do-i-need-to-allow"></a>Какие IP-адреса нужно разрешить?
При работе с Azure иногда необходимо разрешить определенные диапазоны IP-адресов или URL-адреса в корпоративном брандмауэре или прокси, чтобы получить доступ ко всем используемым службам Azure или. Убедитесь, что трафик разрешен для IP-адресов, используемых концентраторами событий. IP-адреса, используемые концентраторами событий Azure: см. в разделе [диапазоны IP-адресов Azure и теги службы — общедоступное облако](https://www.microsoft.com/download/details.aspx?id=56519).

Кроме того, убедитесь, что IP-адрес пространства имен разрешен. Чтобы найти IP-адреса, которые должны быть разрешены для подключений, выполните следующие действия.

1. В командной строке выполните следующую команду: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Запишите IP-адрес, возвращенный в `Non-authoritative answer`. 

При использовании **избыточности зоны** для пространства имен необходимо выполнить несколько дополнительных действий. 

1. Сначала следует запустить nslookup в пространстве имен.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Запишите имя в разделе **не заслуживающий доверия ответ**, который имеет один из следующих форматов: 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Выполните команду nslookup для каждого из них с суффиксами s1, s2 и s3, чтобы получить IP-адреса всех трех экземпляров, работающих в трех зонах доступности. 

    > [!NOTE]
    > IP-адрес, возвращенный `nslookup` командой, не является статическим IP-адресом. Однако он остается постоянным до тех пор, пока базовое развертывание не будет удалено или перемещено в другой кластер.

### <a name="what-client-ips-are-sending-events-to-or-receiving-events-from-my-namespace"></a>Какие IP-адреса клиента отправляют события или получают события из пространства имен?
Сначала включите [фильтрацию IP-адресов](../articles/event-hubs/event-hubs-ip-filtering.md) в пространстве имен. 

Затем включите журналы диагностики для [событий подключения виртуальной сети концентраторов событий](../articles/event-hubs/event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema) , следуя инструкциям в разделе [Включение журналов диагностики](../articles/event-hubs/event-hubs-diagnostic-logs.md#enable-diagnostic-logs). Вы увидите IP-адрес, для которого запрещено подключение.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

> [!IMPORTANT]
> Журналы виртуальной сети создаются только в том случае, если пространство имен разрешает доступ с **конкретных IP-адресов** (правил IP-фильтра). Если вы не хотите ограничивать доступ к пространству имен с помощью этих функций и по-прежнему хотите получать журналы виртуальной сети для мониторинга IP-адресов клиентов, подключающихся к пространству имен концентраторов событий, можно использовать следующее решение: включить фильтрацию IP-адресов и добавить общий IPv4-диапазон (1.0.0.0/1-255.0.0.0/1). Концентраторы событий не поддерживают диапазоны адресов IPv6. 

> [!NOTE]
> В настоящее время невозможно определить исходный IP-адрес отдельного сообщения или события. 