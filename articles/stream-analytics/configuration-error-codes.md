---
title: Коды ошибок конфигурации — Azure Stream Analytics
description: Устранение неполадок с Azure Stream Analytics, используя коды ошибок конфигурации.
ms.author: sidram
author: sidramadoss
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 6137ceeb0d86b1531adab910175ddbc4722ef858
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019522"
---
# <a name="azure-stream-analytics-configuration-error-codes"></a>Коды ошибок данных в Azure Stream Analytics

Журналы действий и журналы ресурсов можно использовать для отладки и устранения ошибок при выполнении заданий в Azure Stream Analytics. В этой статье приводится описание всех кодов конфигурации. Ошибки конфигурации связаны с конфигурацией задания, а также с конфигурациями входных и выходных данных.

## <a name="eventhubunauthorizedaccess"></a>EventHubUnauthorizedAccess

* **Причина.** В концентраторе событий возникла ошибка — *Несанкционированный доступ*.

## <a name="eventhubreceiverepochconflict"></a>EventHubReceiverEpochConflict

* **Причина.** Существует более одного приемника концентратора событий с разными значениями эпохи.
* **Рекомендация**. Убедитесь, что обозреватель *Service Bus Explorer* или приложение *EventProcessorHost* не подключены во время выполнения задания Stream Analytics.

## <a name="eventhubreceiverquotaexceeded"></a>EventHubReceiverQuotaExceeded

* **Причина.** Stream Analytics не удается подключиться к разделу, так как достигнуто максимально допустимое количество получателей на секцию в группе потребителей.
* **Рекомендация**. Убедитесь, что другие задания Stream Analytics или обозреватель Service Bus Explorer не используют одну группу потребителей.

## <a name="eventhuboutputthrottled"></a>EventHubOutputThrottled

* **Причина.** При записи данных в концентратор событий произошла ошибка из-за регулирования полосы пропускания.
* **Рекомендация**. Если это происходит постоянно, обновите пропускную способность.

## <a name="eventhuboutputinvalidconnectionconfig"></a>EventHubOutputInvalidConnectionConfig

* **Причина.** Указана неправильная конфигурация подключения.
* **Рекомендация**. Исправьте конфигурацию и перезапустите задание.

## <a name="eventhuboutputinvalidhostname"></a>EventHubOutputInvalidHostname

* **Причина.** Узел концентратора событий недоступен.
* **Рекомендация**. Убедитесь, что имя узла указано правильно.

## <a name="eventhuboutputunexpectedpartitioncount"></a>EventHubOutputUnexpectedPartitionCount

* **Причина.** Отправитель EventHub обнаружил непредвиденное число секций EventHub.
* **Рекомендация**. Перезапустите задание Stream Analytics, если счетчик секций EventHub изменился.

## <a name="cosmosdbpartitionkeynotfound"></a>CosmosDBPartitionKeyNotFound

* **Причина.** Stream Analytics не удалось найти ключ секции для определенной коллекции Cosmos DB в базе данных.
* **Рекомендация**. Убедитесь, что для коллекции указан допустимый ключ секции в Cosmos DB.

## <a name="cosmosdbinvalidpartitionkeycolumn"></a>CosmosDBInvalidPartitionKeyColumn

* **Причина.** Возникает, когда ключ секции не является ни конечным объектом, ни объектом на верхнем уровне.

## <a name="cosmosdbinvalididcolumn"></a>CosmosDBInvalidIdColumn

* **Причина.** Выходные данные запроса не могут содержать столбец \[id], если как свойство первичного ключа выбран другой столбец.

## <a name="cosmosdbdatabasenotfound"></a>CosmosDBDatabaseNotFound

* **Причина.** Stream Analytics не удается найти базу данных CosmosDB.

## <a name="cosmosdbcollectionnotfound"></a>CosmosDBCollectionNotFound

* **Причина.** Stream Analytics не удается найти определенную коллекцию Cosmos DB в базе данных.

## <a name="cosmosdboutputwritethrottling"></a>CosmosDBOutputWriteThrottling

* **Причина.** Произошла ошибка при записи данных из-за регулирования полосы пропускания Cosmos DB.
* **Рекомендация**. Обновите уровень производительности коллекции и настройте производительность базы данных.

## <a name="sqldatabaseconnectionstringerror"></a>SQLDatabaseConnectionStringError

* **Причина.** В задании Stream Analytics произошла ошибка проверки подлинности.
* **Рекомендация**. Убедитесь, что строка подключения к базе данных SQL указана правильно.

## <a name="sqldatabasemanagedidentityauthenticationerror"></a>SQLDatabaseManagedIdentityAuthenticationError

* **Причина.** В задании Stream Analytics произошла ошибка проверки подлинности. 
* **Рекомендация**. Убедитесь, что имя учетной записи настроено правильно, а управляемое удостоверение задания имеет доступ к базе данных SQL.

## <a name="sqldatabaseoutputnotableerror"></a>SQLDatabaseOutputNoTableError

* **Причина.** Stream Analytics не удается найти сведения о схеме для конкретной таблицы.

## <a name="sqldwoutputinvalidserviceedition"></a>SQLDWOutputInvalidServiceEdition

* **Причина.** База данных SQL не поддерживается.
* **Рекомендация**: используйте выделенный пул SQL.

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с входящими подключениями](stream-analytics-troubleshoot-input.md)
* [Устранение неполадок с выходными данными в Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Устранение неполадок с запросами в Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Устранение неполадок в Azure Stream Analytics с помощью журналов ресурсов](stream-analytics-job-diagnostic-logs.md)
* [Ошибки данных в Azure Stream Analytics](data-errors.md)
