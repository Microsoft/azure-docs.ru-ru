---
title: Настройка журналов диагностики Центров событий Azure | Документация Майкрософт
description: Узнайте, как настроить журналы действий и журналы диагностики для Центров событий в Azure.
ms.topic: article
ms.date: 02/25/2021
ms.openlocfilehash: 5067a2962693ee1c1955aa90e61b43358495585a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104602610"
---
# <a name="set-up-diagnostic-logs-for-an-azure-event-hub"></a>Настройка журналов диагностики для центра событий Azure

Для Центров событий Azure можно просмотреть журналы двух типов.

* **[Журналы действий](../azure-monitor/essentials/platform-logs-overview.md)** . Эти журналы содержат сведения об операциях, выполняемых с заданием. Данные журналы всегда включены. Чтобы просмотреть записи журнала действий, выберите **Журнал действий** в левой области для пространства имен концентратора событий на портале Azure. Пример: "Create or Update Namespace" (Создание или обновление пространства имен), "Create or Update Event Hub" (Создание или обновление концентратора событий).

    ![Журнал действий для пространства имен Центров событий](./media/event-hubs-diagnostic-logs/activity-log.png)
* **[Журналы диагностики](../azure-monitor/essentials/platform-logs-overview.md)** . Журналы диагностики предоставляют более подробные сведения об операциях и действиях, выполняемых в пространстве имен с помощью API или через клиентов управления, используя языковой пакет SDK. 
    
    В следующем разделе показано, как включить журналы диагностики для пространства имен Центров событий.

## <a name="enable-diagnostic-logs"></a>Включение журналов диагностики
По умолчанию журналы диагностики отключены. Чтобы включить журналы диагностики, сделайте следующее:

1.  На [портале Azure](https://portal.azure.com) перейдите к пространству имен Центров событий. 
2. Выберите **Параметры диагностики** в разделе **Мониторинг** на левой панели, а затем выберите **+ Add diagnostic setting** (+ Добавить параметр диагностики). 

    ![Страница "Параметры диагностики" (добавление параметра диагностики)](./media/event-hubs-diagnostic-logs/diagnostic-settings-page.png)
4. В разделе **Сведения о категории** выберите **типы журналов диагностики**, которые необходимо включить. Сведения об этих категориях см. далее в этой статье. 
5. В разделе **Сведения о месте назначения** задайте требуемый целевой объект архива (назначение), например учетную запись хранения, концентратор событий или рабочую область Log Analytics.

    ![Страница добавления параметров диагностики](./media/event-hubs-diagnostic-logs/aDD-diagnostic-settings-page.png)
6.  Выберите **Сохранить** на панели инструментов, чтобы сохранить параметры диагностики.

    Новые параметры вступят в силу в течение 10 минут. После этого журналы появятся в настроенной цели для архивирования на панели **Журналы диагностики**.

    Дополнительные сведения о настройке системы диагностики доступны в [обзоре журналов диагностики Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="diagnostic-logs-categories"></a>Категории журналов диагностики

Центры событий собирают данные журналов диагностики для следующих категорий:

| Категория | Описание | 
| -------- | ----------- | 
| Журналы архивации | Собирает сведения об операциях [Event Hubs Capture](event-hubs-capture-overview.md) (Сбор в Центрах событий), в частности журналы, связанные с ошибками сбора. |
| Журналы операций | Собирают данные всех операций управления, которые выполняются в пространстве имен Центров событий Azure. Операции с данными не фиксируются из-за большого объема операций с данными, проводимых в концентраторах событий Azure. |
| Журналы автомасштабирования | Собирают данные операций автоматического расширения, выполняемых в пространстве имен Центров событий. |
| Журналы координатора Kafka | Собирают данные операций координатора Kafka, связанные с Центрами событий. |
| Журналы ошибок пользователя Kafka | Собирают данные операций об API Kafka, которые вызываются в Центрах событий. |
| Событие подключения виртуальной сети (VNet) Центров событий | Собирают сведения об IP-адресах и виртуальных сетях, отправляющих трафик в Центры событий. |
| Журналы пользователя с управляемым клиентом ключом | Собирают данные операций, связанных с ключом, управляемым клиентом. |


Все журналы хранятся в формате JSON (нотация объектов JavaScript). Каждая запись содержит строковые поля, использующие формат, описанный в следующих разделах.

## <a name="archive-logs-schema"></a>Схема архивных журналов

Строки JSON архивных журналов содержат элементы, перечисленные в следующей таблице.

Имя | Описание
------- | -------
`TaskName` | Описание задачи, завершившейся сбоем.
`ActivityId` | Внутренний идентификатор, используемый для отслеживания
`trackingId` | Внутренний идентификатор, используемый для отслеживания
`resourceId` | Идентификатор ресурса Azure Resource Manager
`eventHub` | Полное имя концентратора событий (включает в себя имя пространства имен).
`partitionId` | Секция концентратора событий, в которую записываются данные.
`archiveStep` | Возможные значения: ArchiveFlushWriter, DestinationInit.
`startTime` | Время начала сбоя
`failures` | Количество произошедших сбоев.
`durationInSeconds` | Продолжительность сбоя
`message` | Сообщение об ошибке
`category` | ArchiveLogs

Ниже приведен пример строки JSON журнала архивирования.

```json
{
   "TaskName": "EventHubArchiveUserError",
   "ActivityId": "000000000-0000-0000-0000-0000000000000",
   "trackingId": "0000000-0000-0000-0000-00000000000000000",
   "resourceId": "/SUBSCRIPTIONS/000000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs Namespace Name>",
   "eventHub": "<Event Hub full name>",
   "partitionId": "1",
   "archiveStep": "ArchiveFlushWriter",
   "startTime": "9/22/2016 5:11:21 AM",
   "failures": 3,
   "durationInSeconds": 360,
   "message": "Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.\r\n   at Microsoft.WindowsAzure.Storage.Shared.Protocol.HttpResponseParsers.ProcessExpectedStatusCodeNoException[T](HttpStatusCode expectedStatusCode, HttpStatusCode actualStatusCode, T retVal, StorageCommandBase`1 cmd, Exception ex)\r\n   at Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob.<PutBlockImpl>b__3e(RESTCommand`1 cmd, HttpWebResponse resp, Exception ex, OperationContext ctx)\r\n   at Microsoft.WindowsAzure.Storage.Core.Executor.Executor.EndGetResponse[T](IAsyncResult getResponseResult)\r\n   --- End of inner exception stack trace ---\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.StorageAsyncResult`1.End()\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.AsyncExtensions.<>c__DisplayClass4.<CreateCallbackVoid>b__3(IAsyncResult ar)\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.",
   "category": "ArchiveLogs"
}
```

## <a name="operational-logs-schema"></a>Схема операционных журналов

Строки JSON операционного журнала содержат элементы, перечисленные в приведенной ниже таблице.

Имя | Описание
------- | -------
`ActivityId` | Внутренний идентификатор, используемый для отслеживания. |
`EventName` | Имя операции. Список значений для этого элемента см. в разделе [имена событий](#event-names) . |
`resourceId` | Идентификатор ресурса Azure Resource Manager |
`SubscriptionId` | Идентификатор подписки |
`EventTimeString` | Время операции |
`EventProperties` |Свойства операции. Этот элемент предоставляет дополнительные сведения о событии, как показано в следующем примере. |
`Status` | Состояние операции. Значение может быть либо **успешно** , либо **неудачно**.  |
`Caller` | Объект, вызвавший операцию (портал Azure или клиент управления) |
`Category` | OperationalLogs |

Ниже приведен пример строки JSON операционного журнала.

```json
Example:
{
   "ActivityId": "00000000-0000-0000-0000-00000000000000",
   "EventName": "Create EventHub",
   "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs namespace name>",
   "SubscriptionId": "000000000-0000-0000-0000-000000000000",
   "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
   "EventProperties": "{\"SubscriptionId\":\"0000000000-0000-0000-0000-000000000000\",\"Namespace\":\"<Namespace Name>\",\"Via\":\"https://<Namespace Name>.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
   "Status": "Succeeded",
   "Caller": "ServiceBus Client",
   "category": "OperationalLogs"
}
```

### <a name="event-names"></a>Имена событий
Имя события заполняется типом операции + тип ресурса из следующих перечислений. Например, `Create Queue`, `Retrieve Event Hu` или `Delete Rule`. 

| Operation type (Тип операции) | Тип ресурса | 
| -------------- | ------------- | 
| <ul><li>Создать</li><li>Update</li><li>DELETE</li><li>Получение</li><li>Неизвестно</li></ul> | <ul><li>Пространство имен</li><li>Очередь</li><li>Раздел</li><li>Подписка</li><li>концентратор событий.</li><li>евенсубсубскриптион</li><li>NotificationHub</li><li>нотификатионхубтиер</li><li>шаредакцессполици</li><li>усажекредит</li><li>намеспацепнскредентиалс</li>Правило</li>ConsumerGroup</li> |

## <a name="autoscale-logs-schema"></a>Схема журналов автомасштабирования
JSON журнала автомасштабирования включает элементы, перечисленные в следующей таблице:

| Имя | Описание |
| ---- | ----------- | 
| `TrackingId` | Внутренний идентификатор, используемый для трассировки. |
| `ResourceId` | Идентификатор ресурса Azure Resource Manager. |
| `Message` | Информационное сообщение, содержащее сведения о действии автоматического расширения. Сообщение содержит предыдущее и текущее значение единицы пропускной способности для данного пространства имен, а также сведения о том, что активировало расширения единицы пропускной способности. |

Ниже приведен пример события автомасштабирования. 

```json
{
    "TrackingId": "fb1b3676-bb2d-4b17-85b7-be1c7aa1967e",
    "Message": "Scaled-up EventHub TUs (UpdateStartTimeUTC: 5/13/2020 7:48:36 AM, PreviousValue: 1, UpdatedThroughputUnitValue: 2, AutoScaleReason: 'IncomingMessagesPerSecond reached 2170')",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name"
}
```

## <a name="kafka-coordinator-logs-schema"></a>Схема журналов координатора Kafka
JSON журнала координатора Kafka включает элементы, перечисленные в следующей таблице:

| Имя | Описание |
| ---- | ----------- | 
| `RequestId` | Идентификатор запроса, используемый для трассировки. |
| `ResourceId` | Идентификатор ресурса Azure Resource Manager |
| `Operation` | Имя операции, выполняемой во время координации групп. |
| `ClientId` | Идентификатор клиента |
| `NamespaceName` | Имя пространства имен | 
| `SubscriptionId` | идентификатор подписки Azure; |
| `Message` | Информационное сообщение или предупреждение, содержащее сведения о действиях, выполненных во время координации групп. |

### <a name="example"></a>Пример

```json
{
    "RequestId": "FE01001A89E30B020000000304620E2A_KafkaExampleConsumer#0",
    "Operation": "Join.Start",
    "ClientId": "KafkaExampleConsumer#0",
    "Message": "Start join group for new member namespace-name:c:$default:I:KafkaExampleConsumer#0-cc40856f7f3c4607915a571efe994e82, current group size: 0, API version: 2, session timeout: 10000ms, rebalance timeout: 300000ms.",
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "KafkaCoordinatorLogs"
}
```

## <a name="kafka-user-error-logs-schema"></a>Схема журналов ошибок пользователя Kafka
JSON журнала ошибок пользователя Kafka содержит элементы, перечисленные в следующей таблице:

| Имя | Описание |
| ---- | ----------- |
| `TrackingId` | ИД отслеживания, который используется для трассировки. |
| `NamespaceName` | Имя пространства имен |
| `Eventhub` | Имя концентратора событий |
| `PartitionId` | Partition ID |
| `GroupId` | Идентификатор группы. |
| `ClientId` | Идентификатор клиента |
| `ResourceId` | Идентификатор ресурса Azure Resource Manager. |
| `Message` | Информационное сообщение, содержащее сведения об ошибке. |

## <a name="event-hubs-virtual-network-connection-event-schema"></a>Схема событий подключения виртуальной сети Центров событий
JSON события подключения виртуальной сети (VNet) Центров событий содержит элементы, перечисленные в следующей таблице:

| Имя | Описание |
| ---  | ----------- | 
| `SubscriptionId` | идентификатор подписки Azure; |
| `NamespaceName` | Имя пространства имен |
| `IPAddress` | IP-адрес клиента, подключающегося к службе "Центры событий". |
| `Action` | Действие, выполняемое службой "Центры событий" при оценке запросов на подключение. Поддерживаемые действия: **Accept Connection** (Принять подключение) и **Deny Connection** (Отклонить подключение). |
| `Reason` | Предоставляет причину выполнения действия. |
| `Count` | Количество повторений для данного действия. |
| `ResourceId` | Идентификатор ресурса Azure Resource Manager. |

Журналы виртуальной сети создаются только в том случае, если пространство имен разрешает доступ из **выбранных сетей** или из **конкретных IP-адресов** (правил IP-фильтра). Если вы не хотите ограничивать доступ к пространству имен с помощью этих функций и по-прежнему хотите получать журналы виртуальной сети для мониторинга IP-адресов клиентов, подключающихся к пространству имен концентраторов событий, можно использовать приведенное ниже решение. [Включите фильтрацию IP-](event-hubs-ip-filtering.md)адресов и добавьте общий IPv4-диапазон (1.0.0.0/1-255.0.0.0/1). IP-фильтрация концентраторов событий не поддерживает диапазоны IPv6. Обратите внимание, что адреса частной конечной точки могут отображаться в формате IPv6 в журнале. 

### <a name="example"></a>Пример

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

## <a name="customer-managed-key-user-logs"></a>Журналы пользователя с управляемым клиентом ключом
JSON журнала пользователя с управляемым клиентом ключом включает элементы, перечисленные в следующей таблице:

| Имя | Описание |
| ---- | ----------- | 
| `Category` | Тип категории для сообщения. Это одно из следующих значений: **error** (ошибка) и **info** (информация). |
| `ResourceId` | Внутренний идентификатор ресурса, включающий идентификатор подписки Azure и имя пространства имен. |
| `KeyVault` | Имя ресурса Key Vault. |
| `Key` | Имя ключа Key Vault. |
| `Version` | Версия ключа Key Vault. |
| `Operation` | Имя операции, выполняемой для обслуживания запросов. |
| `Code` | Код состояния |
| `Message` | Сообщение, содержащее сведения об ошибке, или информационное сообщение. |



## <a name="next-steps"></a>Дальнейшие действия
- [Что такое Центры событий Azure?](./event-hubs-about.md)
- [Примеры Центров событий](sdks.md)
- Приступая к работе с Центрами событий
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
