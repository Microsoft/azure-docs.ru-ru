---
title: Использование служебной шины Azure на платформе .NET с протоколом AMQP 1.0 | Документация Майкрософт
description: В этой статье описывается, как использовать служебную шину Azure из приложения .NET с помощью AMQP (Улучшенный протокол очереди обмена сообщениями).
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 20800363327aefda073cd484dc737b28e60466a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98632856"
---
# <a name="use-service-bus-from-net-with-amqp-10"></a>Использование служебной шины на платформе .NET с протоколом AMQP 1.0

Поддержка AMQP 1.0 доступна в пакете служебной шины версии 2.1 или более поздней. Чтобы гарантировать использование последней версии, скачайте части служебной шины с [NuGet][NuGet].

> [!NOTE]
> Для служебной шины можно использовать либо Расширенный протокол управления очередью сообщений (AMQP), либо протокол обмена сообщениями служебной шины (SBMP) с библиотекой .NET. AMQP — это протокол по умолчанию, используемый библиотекой .NET. Рекомендуется использовать протокол AMQP (по умолчанию), а не переопределять его. 

## <a name="configure-net-applications-to-use-amqp-10"></a>Настройка приложений .NET для использования протокола AMQP 1.0

По умолчанию клиентская библиотека .NET служебной шины взаимодействует со службой служебной шины с помощью протокола AMQP. Можно также явно указать AMQP в качестве типа транспорта, как показано в следующем разделе. 

Текущий выпуск содержит некоторые функции интерфейса API, не поддерживаемые при использовании AMQP. Эти неподдерживаемые функции перечислены в разделе [Различия в поведении](#behavioral-differences). Кроме того, при использовании AMQP отличается значение некоторых дополнительных параметров конфигурации.

### <a name="configuration-using-appconfig"></a>Настройка с помощью файла App.config

Рекомендуется, чтобы приложения использовали для хранения настроек файл конфигурации App.config. Для приложений служебной шины строку подключения служебной шины можно хранить в файле App.config. Ниже приводится пример файла App.config:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp" />
    </appSettings>
</configuration>
```

Значение параметра `Microsoft.ServiceBus.ConnectionString` — это строка подключения служебной шины, используемая для настройки подключения к шине. Используется следующий формат:

`Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp`

Где `namespace` и `SAS key` предоставляются на [портале Azure][Azure portal] при создании пространства имен служебной шины. Дополнительные сведения см. в статье [Создание пространства имен служебной шины с помощью портала Azure][Create a Service Bus namespace using the Azure portal].

Если вы используете AMQP, добавьте строку подключения с `;TransportType=Amqp`. Эта запись сообщает клиентской библиотеке о подключении к служебной шине по протоколу AMQP 1.0.

### <a name="amqp-over-websockets"></a>AMQP через WebSocket
Чтобы использовать AMQP через WebSockets, задайте `TransportType` в строке подключения значение `AmqpWebSockets` . Например, `Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=AmqpWebSockets`. 

Если вы используете библиотеку .NET Microsoft. Azure. ServiceBus, установите для параметра [сервицебусконнектион. TransportType](/dotnet/api/microsoft.azure.servicebus.servicebusconnection.transporttype) значение Амкпвебсоккетс типа [TransportType enum](/dotnet/api/microsoft.azure.servicebus.transporttype).

Если вы используете библиотеку .NET Azure. Messaging. ServiceBus, установите для параметра [сервицебусклиент. TransportType](/dotnet/api/azure.messaging.servicebus.servicebusclient.transporttype) значение Амкпвебсоккетс типа [сервицебустранспорттипе enum](/dotnet/api/azure.messaging.servicebus.servicebustransporttype).


## <a name="message-serialization"></a>Сериализация сообщений

Если вы используете протокол по умолчанию, в клиентской библиотеке .NET по умолчанию используется тип [DataContractSerializer][DataContractSerializer] для сериализации экземпляра [BrokeredMessage][BrokeredMessage]. Это делает возможным транспорт между клиентской библиотекой и службой служебной шины. При использовании транспортного режима AMQP клиентская библиотека применяет систему типов AMQP для сериализации [сообщение в брокере][BrokeredMessage] в сообщение AMQP. Сериализация позволяет получать и распознавать сообщение принимающим приложением, выполняемым на разных платформах. Например, это может быть приложение Java, которое использует JMS API для доступа к служебной шине.

Создавая экземпляр [BrokeredMessage][BrokeredMessage], вы можете передать объект .NET конструктору в качестве параметра (как текст сообщения). Для объектов, которые могут быть сопоставлены с простыми типами AMQP, текст сообщения сериализуется в типы данных AMQP. Если объект не удается непосредственно сопоставить с простым типом AMQP (когда пользовательский тип определяется приложением), объект сериализуется с использованием [DataContractSerializer][DataContractSerializer], а сериализованные байты отправляются в сообщении данных AMQP.

Для упрощения взаимодействия с клиентами, отличными от .NET, используйте только те типы .NET, которые могут быть сериализованы непосредственно в типы AMQP (как текст сообщения). В таблице ниже приведены эти типы, а также соответствующие им типы AMQP.

| Тип объекта текста .NET | Сопоставленный тип AMQP | Тип раздела текста AMQP |
| --- | --- | --- |
| bool |Логическое |Значение AMQP |
| byte |ubyte |Значение AMQP |
| ushort |ushort |Значение AMQP |
| uint |uint |Значение AMQP |
| ulong |ulong |Значение AMQP |
| sbyte |byte |Значение AMQP |
| short |short |Значение AMQP |
| INT |INT |Значение AMQP |
| long |long |Значение AMQP |
| FLOAT |FLOAT |Значение AMQP |
| double |double |Значение AMQP |
| Decimal |decimal128 |Значение AMQP |
| char |char |Значение AMQP |
| Дата и время |TIMESTAMP |Значение AMQP |
| Guid |uuid |Значение AMQP |
| byte[] |binary |Значение AMQP |
| строка |строка |Значение AMQP |
| System.Collections.IList |list |Значение AMQP. В коллекции могут содержаться только элементы, которые определены в этой таблице. |
| System.Array |array |Значение AMQP. В коллекции могут содержаться только элементы, которые определены в этой таблице. |
| System.Collections.IDictionary |карта |Значение AMQP. В коллекции могут содержаться только элементы, которые определены в этой таблице. Примечание: поддерживаются только строковые ключи. |
| URI |Описанный тип string (см. таблицу ниже) |Значение AMQP |
| DateTimeOffset |Описанный тип long (см. таблицу ниже) |Значение AMQP |
| TimeSpan |Описанный тип long (см. таблицу ниже) |Значение AMQP |
| Stream |binary |Данные AMQP (может быть несколько). Разделы данных содержат необработанные байты из объекта Stream. |
| Другой объект |binary |Данные AMQP (может быть несколько). Содержит сериализованные двоичные данные объекта, который использует DataContractSerializer или предоставленный приложением сериализатор. |

| Тип .NET | Сопоставленный описанный тип AMQP | Примечания |
| --- | --- | --- |
| URI |`<type name=”uri” class=restricted source=”string”> <descriptor name=”com.microsoft:uri” /></type>` |Uri.AbsoluteUri |
| DateTimeOffset |`<type name=”datetime-offset” class=restricted source=”long”> <descriptor name=”com.microsoft:datetime-offset” /></type>` |DateTimeOffset.UtcTicks |
| TimeSpan |`<type name=”timespan” class=restricted source=”long”> <descriptor name=”com.microsoft:timespan” /></type>` |TimeSpan.Ticks |

## <a name="behavioral-differences"></a>Различия в поведении

Существуют некоторые различия в поведении API .NET служебной шины при использовании AMQP в сравнении с протоколом по умолчанию.

* Свойство [OperationTimeout][OperationTimeout] игнорируется.
* `MessageReceiver.Receive(TimeSpan.Zero)` реализуется в виде `MessageReceiver.Receive(TimeSpan.FromSeconds(10))`.
* Завершать сообщения с помощью маркеров блокировки могут только получатели сообщений, первоначально получившие эти сообщения.

## <a name="control-amqp-protocol-settings"></a>Параметры управления протоколом AMQP

[Интерфейсы API .NET](/dotnet/api/) предоставляют несколько параметров для управления поведением протокола AMQP:

* **[MessageReceiver. PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagereceiver.prefetchcount#Microsoft_ServiceBus_Messaging_MessageReceiver_PrefetchCount)**: управляет начальным кредитом, примененным к ссылке. Значение по умолчанию равно 0.
* **[MessagingFactorySettings.AmqpTransportSettings.MaxFrameSize](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.maxframesize#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_MaxFrameSize)** определяет максимальный размер кадра AMQP, доступный во время согласования при открытии подключения. Значение по умолчанию: 65 536 байт.
* **[MessagingFactorySettings.AmqpTransportSettings.Batчфлушинтервал](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.batchflushinterval#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_BatchFlushInterval)**: если передачи являются пакетными, это значение определяет максимальную задержку отправки расстановки. По умолчанию наследуется отправителями и получателями. Отдельные отправители или получатели могут переопределить значение по умолчанию, составляющее 20 миллисекунд.
* **[MessagingFactorySettings. AmqpTransportSettings. усесслстреамсекурити](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.usesslstreamsecurity#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_UseSslStreamSecurity)**: определяет, устанавливаются ли подключения AMQP через TLS-соединение. Значение по умолчанию — **true**.

## <a name="next-steps"></a>Дальнейшие действия

Хотите узнать больше? Перейдите по следующим ссылкам:

* [Протокол AMQP служебной шины — обзор]
* [Руководство по использованию протокола AMQP 1.0]

[Create a Service Bus namespace using the Azure portal]: service-bus-create-namespace-portal.md
[DataContractSerializer]: /dotnet/api/system.runtime.serialization.datacontractserializer
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Microsoft.ServiceBus.Messaging.MessagingFactory.AcceptMessageSession]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory.acceptmessagesession#Microsoft_ServiceBus_Messaging_MessagingFactory_AcceptMessageSession
[OperationTimeout]: /dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
[NuGet]: https://nuget.org/packages/WindowsAzure.ServiceBus/
[Azure portal]: https://portal.azure.com
[Протокол AMQP служебной шины — обзор]: service-bus-amqp-overview.md
[Руководство по использованию протокола AMQP 1.0]: service-bus-amqp-protocol-guide.md