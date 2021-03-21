---
title: Концентраторы событий Azure — исключения (прежние версии)
description: В этой статье приводится список исключений обмена сообщениями Центров событий и предлагаемые действия.
ms.topic: article
ms.date: 02/10/2021
ms.openlocfilehash: a76c98ec7d6d1f3370ed8787bf10d1d16a7baaa5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100390901"
---
# <a name="event-hubs-messaging-exceptions---net-legacy"></a>Исключения обмена сообщениями концентраторов событий — .NET (устаревший)
В этом разделе перечислены исключения .NET, созданные платформа .NET Framework API. 

> [!IMPORTANT]
> Некоторые исключения, перечисленные в статье, относятся только к устаревшей библиотеке концентраторов событий .NET. Например: Microsoft. ServiceBus. * Exceptions.
> 
> Сведения о Евенсубсексцептион, вызванном новой библиотекой .NET, см. в разделе [евенсубсексцептион-.NET](exceptions-dotnet.md) .

## <a name="exception-categories"></a>Категории исключений

Интерфейсы API .NET концентраторов событий создают исключения, которые можно разделить на следующие категории, а также связанные действия, которые можно предпринять, чтобы попытаться исправить их:

 - Ошибка в коде пользователя: 
 
   - [System. ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1&preserve-view=true)
   - [System. InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1&preserve-view=true)
   - [System.OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1&preserve-view=true)
   - [System. Runtime. Serialization. SerializationException](/dotnet/api/system.runtime.serialization.serializationexception?view=netcore-3.1&preserve-view=true)
   
   Общее действие: попытайтесь исправить код, прежде чем продолжать.
 
 - Ошибка настройки или конфигурации: 
 
   - [Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception)
   - [Microsoft.Azure.EventHubs.MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception)
   - [System.UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1&preserve-view=true)
   
   Общее действие: Проверьте конфигурацию и при необходимости измените ее.
   
 - Временные исключения: 
 
   - [Microsoft. ServiceBus. Messaging. MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception)
   - [Microsoft.ServiceBus.Messaging.ServerBusyException](#serverbusyexception)
   - [Microsoft.Azure.EventHubs.ServerBusyException](#serverbusyexception)
   - [Microsoft. ServiceBus. Messaging. MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)
   
   Общее действие. Повторите операцию или уведомите пользователей.
 
 - Другие исключения: 
 
   - [System. Transactions. TransactionException](/dotnet/api/system.transactions.transactionexception?view=netcore-3.1&preserve-view=true)
   - [System.TimeoutException](#timeoutexception)
   - [Microsoft. ServiceBus. Messaging. MessageLockLostException](/dotnet/api/microsoft.servicebus.messaging.messagelocklostexception)
   - [Microsoft. ServiceBus. Messaging. SessionLockLostException](/dotnet/api/microsoft.servicebus.messaging.sessionlocklostexception)
   
   Общее действие: зависит от типа исключения; см. таблицу в следующем разделе. 

## <a name="exception-types"></a>Типы исключений
В следующей таблице перечислены типы исключений обмена сообщениями и их причины, а также рекомендуемые действия, которые можно выполнять.

| Тип исключения | Описание, причина, примеры | Рекомендуемое действие | Примечание к автоматическому или немедленному повтору |
| -------------- | -------------------------- | ---------------- | --------------------------------- |
| [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true) |Сервер не ответил на запрошенную операцию в течение указанного времени, который управляется [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings). Возможно, сервер выполнил запрошенную операцию. Это исключение может произойти из-за сетевой или другой задержки инфраструктуры. |Проверьте состояние системы для обеспечения согласованности и при необходимости повторите попытку.<br /> Ознакомьтесь с разделом [TimeoutException](#timeoutexception). | Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1&preserve-view=true) |Запрошенная пользовательская операция не разрешена в пределах сервера или службы. Подробные сведения см. в сообщении об исключении. Например, операция [Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) создаст это исключение, если сообщение было получено в режиме [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode). | Проверьте код и обратитесь к документации. Убедитесь, что запрошенная операция допустима. | Повторная попытка не поможет. |
| [OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1&preserve-view=true) | Предпринята попытка вызвать операцию с объектом, который уже закрыт, прерван или удален. Иногда это означает, что внешняя транзакция уже удалена. | Проверьте код и убедитесь, что он не вызывает операции с удаленным объектом. | Повторная попытка не поможет. |
| [UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1&preserve-view=true) | Объекту [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) не удалось получить маркер, токен является недопустимым, или маркер не содержит утверждений, необходимых для выполнения операции. | Убедитесь, что поставщик маркеров создан с использованием правильных значений. Проверьте конфигурацию службы контроля доступа. | Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1&preserve-view=true)<br /> [ArgumentNullException](/dotnet/api/system.argumentnullexception?view=netcore-3.1&preserve-view=true)<br />[ArgumentOutOfRangeException](/dotnet/api/system.argumentoutofrangeexception?view=netcore-3.1&preserve-view=true) | Для этого метода предоставлен один или несколько недопустимых аргументов. Универсальный код ресурса (URI), переданный в [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory), содержит сегменты пути. В [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) передана недопустимая схема универсального кода ресурса (URI). Значение свойства превышает 32 КБ. | Проверьте вызывающий код и убедитесь, что аргументы заданы правильно. | Повторная попытка не поможет. |
| [Microsoft.ServiceBus.Messaging MessagingEntityNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception) <br /><br/> [Microsoft.Azure.EventHubs MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception) | Сущность, связанная с операцией, не существует или была удалена. | Убедитесь, что сущность существует. | Повторная попытка не поможет. |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) | Клиент не может установить соединение с концентратором событий. |Убедитесь, что имя узла указано правильно и узел доступен. | Повторная попытка может помочь при наличии периодических сбоев подключения. |
| [ServerBusyException Microsoft. ServiceBus. Messaging](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) <br /> <br/>[Microsoft.Azure.EventHubs ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) | В настоящее время служба не может обработать запрос. | Клиент может некоторое время подождать, после чего следует повторить операцию. <br /> Ознакомьтесь с разделом [ServerBusyException](#serverbusyexception). | Клиент может повторить операцию через определенный промежуток времени. Если в результате повтора возникает другое исключение, проверьте поведение повтора этого исключения. |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) | Универсальное исключение обмена сообщениями, которое может быть вызвано в следующих случаях: предпринята попытка создать [QueueClient](/dotnet/api/microsoft.servicebus.messaging.queueclient) с использованием имени или пути, которые относятся к другому типу сущности (например, разделу). Предпринята попытка отправить сообщение размером свыше 1 МБ. Во время обработки запроса возникла ошибка сервера или службы. Подробные сведения см. в сообщении об исключении. Как правило, это временное исключение. | Проверьте код и убедитесь, что для текста сообщения применяются только сериализуемые объекты (или используйте настраиваемый сериализатор). Обратитесь к документации для получения сведений о поддерживаемых типах значений свойств и используйте только поддерживаемые типы. Проверьте свойство [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception) . Если оно имеет значение **true**, можно повторить операцию. | Алгоритм поведения не определен и может не помочь. |
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) | Предпринята попытка создать сущность с именем, которое уже используется другой сущностью в этом пространстве имен службы. | Удалите существующую сущность или выберите другое имя для сущности, которую нужно создать. | Повторная попытка не поможет. |
| [QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) | Сущность обмена сообщениями достигла максимально допустимого размера. Это исключение может произойти, если уже было открыто максимальное число получателей (равное 5) на группу потребителей. | Создайте в сущности пространство, получая сообщения из сущности или ее подочередей. <br /> Ознакомьтесь с разделом [QuotaExceededException](#quotaexceededexception). | Повторная попытка может помочь, если сообщения были удалены. |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.servicebus.messaging.messagingentitydisabledexception) | Выполнен запрос на операцию среды выполнения с отключенной сущностью. |Активируйте сущность. | Повторная попытка может помочь, если сущность была активирована. |
| [Microsoft.ServiceBus.Messaging MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) <br /><br/> [Microsoft.Azure.EventHubs MessageSizeExceededException](/dotnet/api/microsoft.azure.eventhubs.messagesizeexceededexception) | Полезная нагрузка сообщения превышает ограничение в 1 МБ. Ограничение в 1 МБ предназначено для всего сообщения, которое может включать системные свойства и любые дополнительные издержки .NET. | Сократите размер полезных данных сообщения, а затем повторите операцию. |Повторная попытка не поможет. |

## <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) указывает на превышение квоты для конкретного объекта.

Это исключение может произойти, если уже было открыто максимальное число получателей (5) на группу потребителей.

### <a name="event-hubs"></a>Центры событий
Центры событий имеют ограничение в 20 групп потребителей на один концентратор событий. При попытке создать больше групп появляется исключение [QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception). 

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true) указывает, что запущенная пользователем операция занимает больше времени, чем время ожидания операции. 

Для Центров событий время ожидания указывается как часть строки подключения или с помощью [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.servicebus.servicebusconnectionstringbuilder). Само сообщение об ошибке может отличаться, но оно всегда содержит значение времени ожидания, указанное для текущей операции. 

Время ожидания должно выполняться во время или между операциями обслуживания, например обновлениями службы концентраторов событий (или) обновлениями ОС на ресурсах, где выполняется служба. Во время обновления операционной системы сущности перемещаются, и узлы обновляются или перезапускаются, что может привести к превышению времени ожидания. Сведения о соглашении об уровне обслуживания (SLA) для службы концентраторов событий Azure см. в статье соглашение об уровне обслуживания [для концентраторов событий](https://azure.microsoft.com/support/legal/sla/event-hubs/). 


### <a name="common-causes"></a>Основные причины
Существует две основные причины этой ошибки: неправильная конфигурация или временная ошибка службы.

- **Неправильная конфигурация.** Время ожидания операции может быть слишком коротким для состояния операции. Время ожидания операции в пакете SDK клиента по умолчанию составляет 60 секунд. Проверьте, не задано ли в вашем коде слишком маленькое значение. Состояние использования сети и ЦП может повлиять на время, затрачиваемое на выполнение определенной операции, поэтому время ожидания операции не должно быть небольшим.
- **Временная ошибка службы**. Иногда служба "Центры событий" может испытывать задержки при обработке запросов, например в периоды интенсивной загрузки сети. В таких случаях можно настроить повторную попытку выполнения операции через некоторое время до ее успешного завершения. Если же операцию по-прежнему не удается выполнить после нескольких попыток, посетите [сайт состояния служб Azure](https://azure.microsoft.com/status/), чтобы получить сведения об известных простоях служб.

## <a name="serverbusyexception"></a>ServerBusyException

[Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) или [Microsoft.Azure.EventHubs.ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) указывают на то, что сервер перегружен. Для этого исключения есть два соответствующих кода ошибки.

### <a name="error-code-50002"></a>Код ошибки 50002
Эта ошибка может возникать по одной из двух причин:

- Нагрузка равномерно распределяется по всем секциям в концентраторе событий, а одна секция — к локальному ограничению единицы пропускной способности.
    
    **Решение**. возможно, это поможет при пересмотре стратегии распределения секций или попытке [EventHubClient. Send (евентдатависаутпартитионкэй)](/dotnet/api/microsoft.servicebus.messaging.eventhubclient) .

- Пространство имен концентраторов событий не имеет достаточного количества единиц пропускной способности (вы можете проверить экран **метрик** в окне пространства имен концентраторов событий в [портал Azure](https://portal.azure.com) для подтверждения). На портале отображается агрегированная (1 минута) информация, но мы измеряем пропускную способность в реальном времени, так что это лишь оценка.

    **Решение**. увеличение количества единиц пропускной способности в пространстве имен может помочь. 

    Единицы пропускной способности можно настроить на странице " **масштаб** " или на странице " **Обзор** " на странице **пространства имен концентраторов событий** в портал Azure. Или можно использовать [Автоматическое](event-hubs-auto-inflate.md)расширение, которое автоматически масштабируется по мере увеличения количества единиц пропускной способности в соответствии с потребностями использования.

    Единицы пропускной способности (единиц пропускной способности) применяются ко всем концентраторам событий в пространстве имен концентраторов событий. Это означает, что единицы пропускной способности приобретаются на уровне пространства имен и совместно используются всеми Центрами событий в этом пространстве имен. Каждая единица пропускной способности предоставляет следующие возможности для пространства имен:

    - до 1 МБ в секунду входящих событий (событий, отправленных в концентратор событий), но не более 1000 входящих событий, операций управления или контролирующих вызовов API в секунду;
    - До 2 МБ в секунду исходящих событий (событий, полученных от концентратора событий), но не более 4096 исходящих событий.
    - до 84 ГБ хранилища событий (достаточно для срока хранения 24 часа, используемого по умолчанию).
    
    На странице **Обзор** в разделе **Показывать метрики** перейдите на вкладку **пропускная способность** . Выберите диаграмму, чтобы открыть ее в окне большего размера с интервалом в 1 минуту на оси x. Взгляните на пиковые значения и разделите их на 60, чтобы получить входящие байты/секунды или исходящие байты в секунду. Используйте аналогичный подход для вычисления количества запросов в секунду в пиковое время на вкладке **запросы** . 

    Если вы видите значения, превышающие число единиц пропускной способности * (1 МБ в секунду для входящих или 1000 запросов на входящий трафик/с, 2 МБ в секунду для исходящего трафика), увеличьте число единиц пропускной способности, используя **масштаб** (в левой меню) пространства имен концентраторов событий, чтобы вручную увеличить или использовать функцию [автоматического](event-hubs-auto-inflate.md) расширения концентраторов событий. Обратите внимание, что автоматическое расширение может увеличиться до 20 единиц ПРОПУСКНОЙ способности. Чтобы подать ему ровно 40 единиц пропускной способности, отправьте [запрос в службу поддержки](../azure-portal/supportability/how-to-create-azure-support-request.md).

### <a name="error-code-50008"></a>Код ошибки 50008

Эта ошибка не должна возникать часто. Она происходит, когда контейнеру, выполняющему код для пространства имен, не хватает ресурсов ЦП — всего за пару секунд до запуска подсистемы балансировки нагрузки Центров событий.

**Решение**. Ограничьте вызовы метода GetRuntimeInformation. Концентраторы событий Azure поддерживают до 50 вызовов в секунду на группу потребителей в Жетрунтимеинфо в секунду. При достижении предела может появиться исключение, аналогичное следующему:

```
ExceptionId: 00000000000-00000-0000-a48a-9c908fbe84f6-ServerBusyException: The request was terminated because the namespace 75248:aaa-default-eventhub-ns-prodb2b is being throttled. Error code : 50008. Please wait 10 seconds and try again.
```


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Центрах событий см. в следующих источниках:

* [Общие сведения о Центрах событий](./event-hubs-about.md)
* [Создание концентратора событий](event-hubs-create.md)
* [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.md)