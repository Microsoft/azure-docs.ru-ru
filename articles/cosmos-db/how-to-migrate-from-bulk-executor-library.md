---
title: Миграция из библиотеки NET выполнителя в пакетную поддержку в Azure Cosmos DB пакете SDK для .NET v3
description: Узнайте, как перенести приложение из использования библиотеки пакетного исполнителя в пакетную поддержку в Azure Cosmos DB SDK v3.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/24/2020
ms.author: maquaran
ms.custom: devx-track-dotnet
ms.openlocfilehash: 24d6b475964e4bf7745495e9c41d0e89bb76f7e9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341301"
---
# <a name="migrate-from-the-bulk-executor-library-to-the-bulk-support-in-azure-cosmos-db-net-v3-sdk"></a>Миграция из библиотеки NET выполнителя в пакетную поддержку в Azure Cosmos DB пакете SDK для .NET v3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описываются действия, необходимые для переноса кода существующего приложения, использующего [библиотеку .NET NET выполнителя](bulk-executor-dot-net.md) , в функцию [пакетной поддержки](tutorial-sql-api-dotnet-bulk-import.md) в последней версии пакета SDK для .NET.

## <a name="enable-bulk-support"></a>Включить поддержку групповой поддержки

Включите поддержку групповой поддержки для `CosmosClient` экземпляра с помощью конфигурации [алловбулкексекутион](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.allowbulkexecution) :

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="Initialization":::

## <a name="create-tasks-for-each-operation"></a>Создание задач для каждой операции

Поддержка пакетной поддержки в пакете SDK для .NET осуществляется с помощью [параллельной библиотеки задач](/dotnet/standard/parallel-programming/task-parallel-library-tpl) и операций группирования, выполняемых одновременно. 

В пакете SDK нет единственного метода, который будет принимать список документов или операций в качестве входного параметра, но вместо этого необходимо создать задачу для каждой операции, которую нужно выполнить в пакетном режиме, а затем просто дождаться их завершения.

Например, если начальным входом является список элементов, в котором каждый элемент имеет следующую схему:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="Model":::

Если требуется выполнить групповое импорт (аналогично использованию BulkExecutor. Булкимпортасинк), необходимо иметь одновременные вызовы `CreateItemAsync` . Пример:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkImport":::

Если требуется выполнить групповое *Обновление* (аналогично использованию [BulkExecutor. булкупдатеасинк](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkupdateasync)), необходимо иметь одновременные вызовы `ReplaceItemAsync` метода после обновления значения элемента. Пример:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkUpdate":::

Если вы хотите выполнить групповое *Удаление* (аналогично использованию [BulkExecutor. булкделетеасинк](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkdeleteasync)), необходимо иметь одновременные вызовы `DeleteItemAsync` с `id` ключом раздела и для каждого элемента. Пример:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkDelete":::

## <a name="capture-task-result-state"></a>Сохранить состояние результата задачи

В предыдущих примерах кода мы создали параллельный список задач и назвали `CaptureOperationResponse` метод для каждой из этих задач. Этот метод является расширением, позволяющим нам поддерживать *похожую схему ответа* как BulkExecutor, записывая все ошибки и отслеживая [Использование единиц запросов](request-units.md).

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="CaptureOperationResult":::

Где `OperationResponse` объявлен как:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="OperationResult":::

## <a name="execute-operations-concurrently"></a>Выполнять операции одновременно

Для трассировки области всего списка задач мы используем этот вспомогательный класс:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="BulkOperationsHelper":::

`ExecuteAsync`Метод будет ожидать завершения всех операций, и вы сможете использовать его следующим образом:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="WhenAll":::

## <a name="capture-statistics"></a>Статистика записи

Предыдущий код ожидает завершения всех операций и вычисляет необходимую статистику. Эти статистические данные похожи на [булкимпортреспонсе](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkimport.bulkimportresponse)библиотеки небольшого выполнителя.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration/Program.cs" ID="ResponseType":::

`BulkOperationResponse`Содержит:

1. Общее время, затраченное на обработку списка операций с помощью группового поддержки.
1. Количество успешных операций.
1. Общее количество потребленных единиц запросов.
1. При наличии сбоев отображается список кортежей, содержащих исключение, и связанный элемент для ведения журнала и идентификации.

## <a name="retry-configuration"></a>Настройка повторных попыток

Библиотека полного Исполнительного исполнителя представила [рекомендации](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account) по установке и для RetryOptions в для `MaxRetryWaitTimeInSeconds` `MaxRetryAttemptsOnThrottledRequests` [](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.retryoptions) `0` делегирования управления библиотеке.

Для пакетной поддержки в пакете SDK для .NET нет скрытого поведения. Параметры повтора можно настроить непосредственно с помощью [космосклиентоптионс. максретряттемптсонрателимитедрекуестс](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretryattemptsonratelimitedrequests) и [космосклиентоптионс. MaxRetryWaitTimeOnRateLimitedRequests](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretrywaittimeonratelimitedrequests).

> [!NOTE]
> В тех случаях, когда подготовленные единицы запроса значительно ниже, чем ожидалось, в зависимости от объема данных, может потребоваться задать для них высокие значения. Эта операция требует больше времени, но она имеет более высокую шансы на полное успешное завершение из-за более высоких повторных попыток.

## <a name="performance-improvements"></a>Улучшения производительности

Как и в случае с другими операциями с пакетом SDK для .NET, использование интерфейсов API потока приводит к повышению производительности и позволяет избежать ненужной сериализации. 

Использование API-интерфейсов потока возможно только в том случае, если характер используемых данных соответствует потоку байтов (например, потокам файлов). В таких случаях использование `CreateItemStreamAsync` `ReplaceItemStreamAsync` методов, или `DeleteItemStreamAsync` и работы с `ResponseMessage` (а не `ItemResponse` ) увеличивает пропускную способность, которую можно достичь.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о выпусках пакета SDK для .NET см. в статье [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) .
* Получите полный [Исходный код миграции](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration) с сайта GitHub.
* [Дополнительные примеры для выполнения инструкций на GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkSupport)