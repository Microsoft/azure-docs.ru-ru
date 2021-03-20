---
title: Служебная шина Azure — исключения обмена сообщениями | Документация Майкрософт
description: В этой статье представлен список исключений обмена сообщениями служебной шины Azure и предлагаемых действий, которые необходимо выполнить при возникновении исключения.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 3b56aff2635593d6cb49adbcf3784ddd5cb4fa39
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99219151"
---
# <a name="service-bus-messaging-exceptions"></a>Исключения обмена сообщениями служебной шины
В этой статье перечислены исключения .NET, созданные платформа .NET Framework API. 

## <a name="exception-categories"></a>Категории исключений
API обмена сообщениями создают исключения, которые можно разделить на следующие категории с указанием связанного действия, предпринимаемого для их устранения. Значение и причины исключения могут различаться в зависимости от типа сущности обмена сообщениями.

1. Ошибка кодирования пользователя ([System. ArgumentException](/dotnet/api/system.argumentexception), [System. InvalidOperationException](/dotnet/api/system.invalidoperationexception), [System. OperationCanceledException](/dotnet/api/system.operationcanceledexception), [System. Runtime. Serialization. SerializationException](/dotnet/api/system.runtime.serialization.serializationexception)). Общее действие: прежде чем продолжить, попытайтесь исправить код.
2. Ошибка установки или конфигурации ([Microsoft. servicebus. Messaging. MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception), [System. UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception). Общее действие: проверьте конфигурацию и измените ее при необходимости.
3. Временные исключения ([Microsoft. servicebus. Messaging. MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception), [Microsoft. servicebus. Messaging. ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception), [Microsoft. servicebus. Messaging. MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)). Общее действие: повторите операцию или уведомите пользователей. `RetryPolicy`Класс в клиентском пакете SDK можно настроить на автоматическую обработку повторных попыток. Дополнительные сведения см. в разделе [руководство по повторным попыткам](/azure/architecture/best-practices/retry-service-specific#service-bus).
4. Другие исключения ([System. Transactions. TransactionException](/dotnet/api/system.transactions.transactionexception), [System. TimeoutException](/dotnet/api/system.timeoutexception), [Microsoft. servicebus. Messaging. MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception), [Microsoft. servicebus. Messaging. SessionLockLostException](/dotnet/api/microsoft.azure.servicebus.sessionlocklostexception)). Общее действие: зависит от типа исключения; см. таблицу в следующем разделе: 

## <a name="exception-types"></a>Типы исключений
В следующей таблице перечислены типы исключений обмена сообщениями и их причины, а также рекомендуемые действия, которые можно выполнять.

| **Тип исключения** | **Описание, причина, примеры** | **Предлагаемое действие** | **Примечание к автоматическому или немедленному повтору** |
| --- | --- | --- | --- |
| [TimeoutException](/dotnet/api/system.timeoutexception) |Сервер не ответил на запрошенную операцию в течение указанного времени, который управляется [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings). Возможно, сервер выполнил запрошенную операцию. Это может произойти из-за задержек сети или других инфраструктур. |Проверьте состояние системы для обеспечения согласованности и при необходимости повторите попытку. Ознакомьтесь с разделом [Исключения времени ожидания](#timeoutexception). |Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [InvalidOperationException](/dotnet/api/system.invalidoperationexception) |Запрошенная пользовательская операция не разрешена в пределах сервера или службы. Подробные сведения см. в сообщении об исключении. Например, [Complete ()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) создает это исключение, если сообщение было получено в режиме [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode) . |Проверьте код и обратитесь к документации. Убедитесь, что запрошенная операция допустима. |Повторная попытка не помогает. |
| [OperationCanceledException](/dotnet/api/system.operationcanceledexception) |Предпринята попытка вызвать операцию с объектом, который уже закрыт, прерван или удален. Иногда это означает, что внешняя транзакция уже удалена. |Проверьте код и убедитесь, что он не вызывает операции с удаленным объектом. |Повторная попытка не помогает. |
| [UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception) |Объекту [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) не удалось получить маркер, токен является недопустимым, или маркер не содержит утверждений, необходимых для выполнения операции. |Убедитесь, что поставщик маркеров создан с использованием правильных значений. Проверьте конфигурацию службы контроля доступа. |Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [ArgumentException](/dotnet/api/system.argumentexception)<br /> [ArgumentNullException](/dotnet/api/system.argumentnullexception)<br />[ArgumentOutOfRangeException](/dotnet/api/system.argumentoutofrangeexception) |Для этого метода предоставлен один или несколько недопустимых аргументов.<br /> Универсальный код ресурса (URI), переданный в [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory), содержит сегменты пути.<br /> В [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) передана недопустимая схема универсального кода ресурса (URI). <br />Значение свойства превышает 32 КБ. |Проверьте вызывающий код и убедитесь, что аргументы заданы правильно. |Повторная попытка не помогает. |
| [MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception) |Сущность, связанная с операцией, не существует или была удалена. |Убедитесь, что сущность существует. |Повторная попытка не помогает. |
| [MessageNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagenotfoundexception) |Предпринята попытка получить сообщение с определенным порядковым номером. Это сообщение не найдено. |Убедитесь, что сообщение еще не получено. Проверьте очередь недоставленных сообщений, чтобы увидеть, не находится ли в ней сообщение. |Повторная попытка не помогает. |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) |Клиент не может установить подключение к служебной шине. |Убедитесь, что имя узла указано правильно и узел доступен. <p>Если код выполняется в среде с брандмауэром или прокси-сервером, убедитесь, что трафик к домену/IP-адресу или порту служебной шины не заблокирован.</p>|Повторная попытка может помочь при наличии периодических сбоев подключения. |
| [ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception) |В данный момент служба не может обработать запрос. |Клиент может некоторое время подождать, после чего следует повторить операцию. |Клиент может повторить операцию через определенный промежуток времени. Если в результате повтора возникает другое исключение, проверьте поведение повтора этого исключения. |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) |Универсальное исключение обмена сообщениями, порождаемое в следующих случаях.<p>Предпринята попытка создать [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient) с использованием имени или пути, которые относятся к другому типу сущности (например, разделу).</p><p>Предпринята попытка отправить сообщение размером свыше 256 КБ. </p>Во время обработки запроса возникла ошибка сервера или службы. Подробные сведения см. в сообщении об исключении. Обычно это временное исключение.</p><p>Запрос был завершен, так как выполняется регулирование сущности. Код ошибки: 50001, 50002, 50008. </p> | Проверьте код и убедитесь, что для текста сообщения применяются только сериализуемые объекты (или используйте настраиваемый сериализатор). <p>Обратитесь к документации для получения сведений о поддерживаемых типах значений свойств и используйте только поддерживаемые типы.</p><p> Проверьте свойство [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception) . Если **это так, можно** повторить операцию. </p>| Если исключение вызвано регулированием, подождите несколько секунд и повторите операцию. Поведение повторных попыток не определено и может не помочь в других сценариях.|
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) |Предпринята попытка создать сущность с именем, которое уже используется другой сущностью в этом пространстве имен службы. |Удалите существующую сущность или выберите другое имя для сущности, которую нужно создать. |Повторная попытка не помогает. |
| [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) |Сущность обмена сообщениями достигла максимально допустимого размера, или превышено максимально возможное количество подключений к пространству имен. |Создайте в сущности пространство, получая сообщения из сущности или ее подочередей. См. [QuotaExceededException](#quotaexceededexception). |Повторная попытка может помочь, если сообщения были удалены. |
| [RuleActionException](/dotnet/api/microsoft.servicebus.messaging.ruleactionexception) |Служебная шина возвращает это исключение, если вы пытаетесь создать недопустимое действие правила. Служебная шина вкладывает это исключение в недоставленное сообщение, если во время обработки действия правила для данного сообщения произошла ошибка. |Проверьте правильность действия правила. |Повторная попытка не помогает. |
| [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) |Служебная шина возвращает это исключение, если вы пытаетесь создать недопустимый фильтр. Служебная шина вкладывает это исключение в недоставленное сообщение, если во время обработки фильтра для данного сообщения произошла ошибка. |Проверьте правильность фильтра. |Повторная попытка не помогает. |
| [SessionCannotBeLockedException](/dotnet/api/microsoft.servicebus.messaging.sessioncannotbelockedexception) |Предпринята попытка принять сеанс с конкретным идентификатором сеанса, но сеанс в данный момент заблокирован другим клиентом. |Убедитесь, что сеанс разблокирован другими клиентами. |Повторная попытка может помочь, если сеанс был освобожден до этого времени. |
| [TransactionSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.transactionsizeexceededexception) |В состав транзакции входит слишком много операций. |Сократите количество операций, входящих в состав этой транзакции. |Повторная попытка не помогает. |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.azure.servicebus.messagingentitydisabledexception) |Выполнен запрос на операцию среды выполнения с отключенной сущностью. |Активируйте сущность. |Повторная попытка может помочь, если сущность была активирована. |
| [NoMatchingSubscriptionException](/dotnet/api/microsoft.servicebus.messaging.nomatchingsubscriptionexception) |Служебная шина возвращает это исключение, если сообщение отправлено в раздел, в котором включена предварительная фильтрация, и ни один из фильтров не соответствует. |Убедитесь, что соответствует по меньшей мере один фильтр. |Повторная попытка не помогает. |
| [MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |Полезные данные сообщения превышают предел в 256 КБ. Предел в 256 КБ является общим размером сообщения с учетом системных свойств и служебных данных .NET. |Сократите размер полезных данных сообщения, а затем повторите операцию. |Повторная попытка не помогает. |
| [TransactionException](/dotnet/api/system.transactions.transactionexception) |Внешняя транзакция (*Transaction.Current*) является недопустимой. Возможно, она завершена или прервана. Дополнительные сведения можно найти во внутреннем исключении. | |Повторная попытка не помогает. |
| [TransactionInDoubtException](/dotnet/api/system.transactions.transactionindoubtexception) |Предпринята попытка выполнить операцию с сомнительной транзакцией либо попытка зафиксировать транзакцию, и транзакция становится сомнительной. |Ваше приложение должно обрабатывать это исключение (как особый случай), так как транзакция уже может быть зафиксирована. |- |

## <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) указывает на превышение квоты для конкретного объекта.

### <a name="queues-and-topics"></a>Очереди и разделы
Для очередей и разделов часто бывает размер очереди. Свойство сообщения об ошибке содержит дополнительные сведения, как показано в следующем примере:

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException
Message: The maximum entity size has been reached or exceeded for Topic: 'xxx-xxx-xxx'. 
    Size of entity in bytes:1073742326, Max entity size in bytes:
1073741824..TrackingId:xxxxxxxxxxxxxxxxxxxxxxxxxx, TimeStamp:3/15/2013 7:50:18 AM
```

Сообщение уведомляет о том, что для раздела превышено ограничение размера, в данном случае 1 ГБ (ограничение по умолчанию). 

### <a name="namespaces"></a>Пространства имен

Для пространств имен исключение [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) может указывать на то, что приложение превысило максимально возможное количество подключений к пространству имен. Пример:

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException: ConnectionsQuotaExceeded for namespace xxx.
<tracking-id-guid>_G12 ---> 
System.ServiceModel.FaultException`1[System.ServiceModel.ExceptionDetail]: 
ConnectionsQuotaExceeded for namespace xxx.
```

### <a name="common-causes"></a>Основные причины
Существует две основные причины этой ошибки: очередь недоставленных сообщений и неработающие получатели сообщений.

1. **[Очередь недоставленных сообщений](service-bus-dead-letter-queues.md)** Модуль чтения не может завершить сообщения, и сообщения возвращаются в очередь или раздел по истечении срока действия блокировки. Это может произойти, если средство чтения встречает исключение, препятствующее вызову [BrokeredMessage. Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.complete). После 10 попыток считывания сообщения оно по умолчанию перемещается в очередь недоставленных сообщений. Это поведение управляется свойством [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) со значением по умолчанию "10". По мере накапливания сообщений в очереди недоставленных сообщений они начинают занимать много места.
   
    Чтобы устранить эту проблему, считайте и завершите сообщения в очереди недоставленных сообщений, как и в любой другой очереди. Вы можете использовать метод [FormatDeadLetterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formatdeadletterpath), который помогает форматировать путь очереди недоставленных сообщений.
2. **Получатель остановлен**. Получатель прекратил прием сообщений из очереди или подписки. Определить эту проблему можно с помощью свойства [QueueDescription.MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails), которое выводит полную статистику по сообщениям. Если свойство [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount) имеет значение High или растет, то сообщения не считываются так быстро, как они записываются.

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](/dotnet/api/system.timeoutexception) указывает, что запущенная пользователем операция занимает больше времени, чем время ожидания операции. 

Следует проверить значение свойства [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit), так как достижение этого ограничения также может породить исключение [TimeoutException](/dotnet/api/system.timeoutexception).

Время ожидания должно выполняться во время или между операциями обслуживания, такими как обновления службы служебной шины (или), в ресурсах, где выполняется служба. Во время обновления операционной системы сущности перемещаются, и узлы обновляются или перезапускаются, что может привести к превышению времени ожидания. Сведения о соглашении об уровне обслуживания (SLA) для службы служебной шины Azure см. в статье соглашение об уровне обслуживания для служебной [шины](https://azure.microsoft.com/support/legal/sla/service-bus/).


### <a name="queues-and-topics"></a>Очереди и разделы
Для очередей и разделов время ожидания указывается в свойстве [MessagingFactorySettings.OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings) (как часть строки подключения) или с помощью [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.azure.servicebus.servicebusconnectionstringbuilder). Само сообщение об ошибке может отличаться, но оно всегда содержит значение времени ожидания, указанное для текущей операции. 

## <a name="messagelocklostexception"></a>MessageLockLostException

### <a name="cause"></a>Причина

**MessageLockLostException** создается при получении сообщения с помощью режима получения [PeekLock](message-transfers-locks-settlement.md#peeklock) , а блокировка, удерживаемая клиентом, истекает на стороне службы.

Срок действия блокировки сообщения может истечь из-за различных причин. 

  * Срок действия таймера блокировки истек до его продления клиентским приложением.
  * Клиентское приложение получило блокировку, сохранило его в постоянном хранилище, а затем перезапускается. После перезапуска клиентское приложение просмотрело сообщения порядковых и попыталось выполнить эти действия.

### <a name="resolution"></a>Решение

В случае **MessageLockLostException** клиентское приложение больше не может обработать сообщение. Клиентское приложение может дополнительно рассмотреть возможность регистрации исключения для анализа, но клиент *должен* ликвидировать сообщение.

Так как срок действия блокировки сообщения истек, он вернется к очереди (или подписке) и может быть обработан следующим клиентским приложением, которое вызывает метод Receive.

Если **MaxDeliveryCount** превышено, сообщение может быть перемещено в **Свойство DeadLetterQueue**.

## <a name="sessionlocklostexception"></a>SessionLockLostException

### <a name="cause"></a>Причина

**SessionLockLostException** вызывается при принятии сеанса и истечения срока действия, удерживаемого клиентом на стороне службы.

Срок действия блокировки сеанса может истечь из-за различных причин. 

  * Срок действия таймера блокировки истек до его продления клиентским приложением.
  * Клиентское приложение получило блокировку, сохранило его в постоянном хранилище, а затем перезапускается. После перезапуска клиентское приложение просмотрело сеансы порядковых и попытался обработать сообщения в этих сеансах.

### <a name="resolution"></a>Решение

В случае **SessionLockLostException** клиентское приложение больше не может обрабатывать сообщения в сеансе. Клиентское приложение может рассмотреть возможность регистрации исключения для анализа, но клиент *должен* ликвидировать сообщение.

Поскольку срок действия блокировки сеанса истек, он вернется к очереди (или подписке) и может быть заблокирован следующим клиентским приложением, которое принимает сеанс. Так как блокировка сеанса удерживается одним клиентским приложением в любой момент времени, обработка в порядке гарантируется.

## <a name="socketexception"></a>SocketException

### <a name="cause"></a>Причина

Исключение **SocketException** в следующих случаях:
   * При неудачной попытке соединения из-за неправильного ответа узла через указанное время (код ошибки TCP 10060).
   * Произошел сбой установленного соединения, так как не удалось ответить подключенному узлу.
   * Произошла ошибка при обработке сообщения, или превышено время ожидания удаленным узлом.
   * Базовая сетевая ошибка ресурса.

### <a name="resolution"></a>Решение

Ошибки **SocketException** указывают на то, что виртуальная машина, на которой размещены приложения, не может преобразовать имя `<mynamespace>.servicebus.windows.net` в соответствующий IP-адрес. 

Проверьте, выполняется ли следующая команда при сопоставлении с IP-адресом.

```Powershell
PS C:\> nslookup <mynamespace>.servicebus.windows.net
```

который должен предоставить выходные данные, как показано ниже

```bash
Name:    <cloudappinstance>.cloudapp.net
Address:  XX.XX.XXX.240
Aliases:  <mynamespace>.servicebus.windows.net
```

Если указанное выше имя **не разрешается** в IP-адрес и псевдоним пространства имен, проверьте, какой сетевой администратор должен исследовать дополнительные сведения. Разрешение имен выполняется через DNS-сервер, как правило, ресурс в клиентской сети. Если разрешение DNS выполняется Azure DNS обратитесь в службу поддержки Azure.

Если разрешение имен **работает правильно**, проверьте, разрешены ли [здесь](service-bus-troubleshooting-guide.md#connectivity-certificate-or-timeout-issues) подключения к служебной шине Azure.


## <a name="messagingexception"></a>MessagingException

### <a name="cause"></a>Причина

**MessagingException** — это универсальное исключение, которое может вызываться по различным причинам. Некоторые из причин перечислены ниже.

   * Предпринята попытка создать **QueueClient** для **раздела** или **подписки**.
   * Размер отправляемого сообщения превышает ограничение для данного уровня. Узнайте больше о [квотах и ограничениях](service-bus-quotas.md)служебной шины.
   * Конкретный запрос плоскости данных (отправка, получение, завершение, прекращение) был прерван из-за регулирования.
   * Временные проблемы, вызванные обновлением службы и перезагрузкой.

> [!NOTE]
> Приведенный выше список исключений не является исчерпывающим.

### <a name="resolution"></a>Решение

Действия по устранению зависят от того, что вызвало исключение **MessagingException** .

   * Для **временных проблем** (где **_для_*параметра IsTrue _ задано значение _*_true_*) или для* проблем регулирования**, повторная попытка операции может разрешить эту операцию. Для этого можно использовать политику повтора по умолчанию в пакете SDK.
   * Для других проблем сведения в исключении указывают на проблемы и способы их устранения, которые можно вывести из одного и того же.

## <a name="next-steps"></a>Дальнейшие действия
Полные справочные материалы по API служебной шины для .NET доступны в [справочнике по API Azure для .NET](/dotnet/api/overview/azure/service-bus).
Советы по устранению неполадок см. в разделе [руководство по устранению неполадок](service-bus-troubleshooting-guide.md)
