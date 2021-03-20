---
title: Проверка производительности и масштабирования с помощью Azure Cosmos DB
description: Узнайте, как тестировать масштабирование и производительность с помощью Azure Cosmos DB. Затем вы сможете оценить возможности Azure Cosmos DB для сценариев с высокопроизводительными приложениями.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/23/2019
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: cb0dd8eb879c5511da3c79a3dee6a715fb1275e4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97358679"
---
# <a name="performance-and-scale-testing-with-azure-cosmos-db"></a>Проверка производительности и масштабирования с помощью Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Проверка производительности и масштабирования — ключевой этап в разработке приложения. Для многих приложений уровень баз данных оказывает значительное влияние на общую производительность и масштабируемость. Поэтому он имеет огромное значение при проверке производительности. Система [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) разработана с расчетом на эластичное масштабирование и предсказуемую производительность. Поэтому она отлично подходит для приложений, требующих высокую производительность для уровня базы данных. 

Эта статья предназначена для специалистов, разрабатывающих наборы тестов производительности для рабочих нагрузок Cosmos DB или оценивающих Cosmos DB для сценариев с высокопроизводительными приложениями. Эта статья уделяет основное внимание изолированной проверке производительности базы данных, но также включает рекомендации для приложений в рабочей среде.

Ознакомившись с данной статьей, вы сможете ответить на следующие вопросы. 

* Где найти образец клиентского приложения .NET для тестирования производительности Azure Cosmos DB? 
* Как добиться высокого уровня пропускной способности с помощью Azure Cosmos DB из клиентского приложения?

Чтобы начать работу с кодом, скачайте проект со страницы [примера для тестирования производительности Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark). 

> [!NOTE]
> Цель этого приложения — продемонстрировать, как обеспечить максимальную производительность Azure Cosmos DB при использовании небольшого числа клиентских компьютеров. Цель примера — не допустить всплесков нагрузки на пропускную способность Azure Cosmos DB (которую можно масштабировать без ограничений).

Если вам нужны параметры конфигурации на стороне клиента для повышения производительности Azure Cosmos DB, ознакомьтесь со статьей [Советы по повышению производительности для Azure Cosmos DB](performance-tips.md).

## <a name="run-the-performance-testing-application"></a>Запуск приложения для тестирования производительности
Самый быстрый способ начать работу — скомпилировать и запустить пример .NET, как описано в следующих шагах. Вы также можете просмотреть исходный код и реализовать аналогичные конфигурации в клиентских приложениях.

**Шаг 1.** Скачайте проект из [Azure Cosmos DB примера теста производительности](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark)или разветвления репозитория GitHub.

**Шаг 2.** Измените параметры для EndpointUrl, Authorizationkey согласно инструкциям, CollectionThroughput и DocumentTemplate (необязательно) в App.config.

> [!NOTE]
> Перед подготовкой коллекций с высокой пропускной способностью просмотрите [страницу цен](https://azure.microsoft.com/pricing/details/cosmos-db/), чтобы рассчитать затраты для каждой коллекции. В Azure Cosmos DB использование места на диске и пропускной способности тарифицируются независимо друг от друга на почасовой основе, Вы можете сэкономить затраты, удалив или снижая пропускную способность контейнеров Azure Cosmos после тестирования.
> 
> 

**Шаг 3**. Скомпилируйте и запустите консольное приложение из командной строки. Вы должны увидеть следующий результат:

```bash
C:\Users\cosmosdb\Desktop\Benchmark>DocumentDBBenchmark.exe
Summary:
---------------------------------------------------------------------
Endpoint: https://arramacquerymetrics.documents.azure.com:443/
Collection : db.data at 100000 request units per second
Document Template*: Player.json
Degree of parallelism*: -1
---------------------------------------------------------------------
DocumentDBBenchmark starting...
Found collection data with 100000 RU/s
Starting Inserts with 100 tasks
Inserted 4503 docs @ 4491 writes/s, 47070 RU/s (122B max monthly 1KB reads)
Inserted 17910 docs @ 8862 writes/s, 92878 RU/s (241B max monthly 1KB reads)
Inserted 32339 docs @ 10531 writes/s, 110366 RU/s (286B max monthly 1KB reads)
Inserted 47848 docs @ 11675 writes/s, 122357 RU/s (317B max monthly 1KB reads)
Inserted 58857 docs @ 11545 writes/s, 120992 RU/s (314B max monthly 1KB reads)
Inserted 69547 docs @ 11378 writes/s, 119237 RU/s (309B max monthly 1KB reads)
Inserted 80687 docs @ 11345 writes/s, 118896 RU/s (308B max monthly 1KB reads)
Inserted 91455 docs @ 11272 writes/s, 118131 RU/s (306B max monthly 1KB reads)
Inserted 102129 docs @ 11208 writes/s, 117461 RU/s (304B max monthly 1KB reads)
Inserted 112444 docs @ 11120 writes/s, 116538 RU/s (302B max monthly 1KB reads)
Inserted 122927 docs @ 11063 writes/s, 115936 RU/s (301B max monthly 1KB reads)
Inserted 133157 docs @ 10993 writes/s, 115208 RU/s (299B max monthly 1KB reads)
Inserted 144078 docs @ 10988 writes/s, 115159 RU/s (298B max monthly 1KB reads)
Inserted 155415 docs @ 11013 writes/s, 115415 RU/s (299B max monthly 1KB reads)
Inserted 166126 docs @ 10992 writes/s, 115198 RU/s (299B max monthly 1KB reads)
Inserted 173051 docs @ 10739 writes/s, 112544 RU/s (292B max monthly 1KB reads)
Inserted 180169 docs @ 10527 writes/s, 110324 RU/s (286B max monthly 1KB reads)
Inserted 192469 docs @ 10616 writes/s, 111256 RU/s (288B max monthly 1KB reads)
Inserted 199107 docs @ 10406 writes/s, 109054 RU/s (283B max monthly 1KB reads)
Inserted 200000 docs @ 9930 writes/s, 104065 RU/s (270B max monthly 1KB reads)

Summary:
---------------------------------------------------------------------
Inserted 200000 docs @ 9928 writes/s, 104063 RU/s (270B max monthly 1KB reads)
---------------------------------------------------------------------
DocumentDBBenchmark completed successfully.
Press any key to exit...
```

**Шаг 4 (при необходимости)**. Пропускная способность (единицы запроса в секунду), определенная с помощью инструмента, должна быть не меньше подготовленной пропускной способности коллекции или набора коллекций. Если это не так, увеличение степени параллелизма (DegreeOfParallelism) с небольшим шагом приращения может помочь в достижении предельного значения. Если пропускная способность клиентского приложения достигла пика, запустите несколько экземпляров приложения на дополнительных клиентских компьютерах. Если вам нужна помощь с этим шагом, отправьте запрос в службу поддержки из [портал Azure](https://portal.azure.com).

После запуска приложения вы можете опробовать различные [политики индексирования](index-policy.md) и [уровни согласованности](consistency-levels.md), чтобы понять, как они влияют на пропускную способность и задержки. Вы также можете просмотреть исходный код и реализовывать аналогичные настройки в ваших тестовых наборах или приложениях в рабочей среде.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье мы рассмотрели способы тестирования производительности и масштабируемости с Azure Cosmos DB с помощью консольного приложения .NET. Дополнительные сведения см. в следующих статьях:

* [Пример для тестирования производительности Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark)
* [Советы по улучшению производительности Azure Cosmos DB](performance-tips.md)
* [Секционирование, ключи секции и масштабирование в Azure Cosmos DB](partitioning-overview.md)


