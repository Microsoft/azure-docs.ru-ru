---
title: Модель извлечения данных канала изменений
description: Сведения об использовании модели извлечения Azure Cosmos DB для чтения веб-канала изменений и о различиях между моделью извлечения и обработчиком веб-канала изменений
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 03/10/2021
ms.reviewer: sngun
ms.openlocfilehash: 9279dddc92629b17a2a73f3a41fe261d322d677e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103463686"
---
# <a name="change-feed-pull-model-in-azure-cosmos-db"></a>Модель извлечения канала изменений в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

С помощью модели извлечения веб-канала изменений вы можете обрабатывать данные веб-канала изменений Azure Cosmos DB в удобном для себя темпе. Как и в случае с уже изученным [обработчиком веб-канала изменений](change-feed-processor.md), модель извлечения веб-канала изменений можно применить для параллельной обработки изменений в нескольких потребителях веб-канала изменений.

> [!NOTE]
> Модель извлечения веб-канала изменений в настоящее время предоставляется только в [пакете SDK .NET для Azure Cosmos DB и в режиме предварительной версии](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.17.0-preview). Для других версий пакета SDK пока недоступна даже предварительная версия.

## <a name="comparing-with-change-feed-processor"></a>Сравнение с обработчиком канала изменений

Во многих сценариях веб-канал изменений можно обрабатывать как с помощью [обработчика веб-канала изменений](change-feed-processor.md), так и модели извлечения. Маркеры продолжения в модели извлечения и контейнер аренды в обработчике веб-канала изменений выполняют роли "закладок", то есть обозначают последний обработанный элемент (или пакет элементов) в веб-канале изменений.

Но вы не сможете преобразовать маркер продолжения в контейнер аренды (или наоборот).

> [!NOTE]
> В большинстве случаев, когда требуется чтение из веб-канала изменений, простейшим вариантом является использование [обработчика канала изменений](change-feed-processor.md).

Мы рекомендуем применять модель извлечения в следующих ситуациях:

- Чтение изменений из определенного ключа секции
- Управление скоростью, с которой клиент получает изменения для обработки
- Выполнение однократного чтения существующих данных в веб-канале изменений (например, для переноса данных)

Ниже перечислены некоторые ключевые различия между обработчиком веб-канала изменений и моделью извлечения.

|Компонент  | Обработчик канала изменений| Модель извлечения |
| --- | --- | --- |
| Отслеживание текущей точки обработки в веб-канале изменений | Аренда (хранится в контейнере Azure Cosmos DB) | Маркер продолжения (сохраненный в памяти или вручную) |
| Возможность воспроизведения прошлых изменений | Да, с моделью отправки | Да, с моделью извлечения|
| Опрос предстоящих изменений | Автоматическая проверка наличия изменений по указанному пользователем `WithPollInterval` | Вручную |
| Поведение при отсутствии новых изменений | Автоматическое ожидание `WithPollInterval` и повторная проверка | Необходимо перехватить исключение и вручную выполнить повторную проверку |
| Обработка изменений от целого контейнера | Да, с автоматической параллелизацией по нескольким потокам или компьютерам для одного контейнера| Да, с параллелизацией вручную через FeedToken |
| Обработка изменений по одному ключу секции | Не поддерживается | Да|
| Уровень поддержки | Общедоступная версия | Preview (Предварительный просмотр) |

> [!NOTE]
> В отличие от чтения с помощью обработчика веб-канала изменений, необходимо явно отреагировать на случаи, когда новые изменения отсутствуют. 

## <a name="consuming-an-entire-containers-changes"></a>Обработка изменений для всего контейнера

Вы можете создать `FeedIterator` для обработки веб-канала изменений с помощью модели извлечения. При первоначальном создании `FeedIterator` необходимо указать обязательное `ChangeFeedStartFrom` значение, которое состоит из начальной и исходной позиций для чтения изменений, а также требуемого значения `FeedRange` . `FeedRange`Представляет собой диапазон значений ключа секции и указывает элементы, которые будут считываться из веб-канала изменений с использованием этого конкретного `FeedIterator` .

При необходимости можно задать `ChangeFeedRequestOptions` значение `PageSizeHint` . `PageSizeHint`— Это максимальное число элементов, которые будут возвращены на одной странице.

Для `FeedIterator` возможны два варианта. Помимо приведенных ниже примеров, которые возвращают объекты сущностей, для получения ответа можно использовать поддержку `Stream`. Потоки позволяют считывать данные без предварительной десериализации, что экономит ресурсы клиента.

Следующий пример получает `FeedIterator`, который возвращает объекты сущности, роль которых здесь выполняет объект `User`:

```csharp
FeedIterator<User> InteratorWithPOCOS = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(), ChangeFeedMode.Incremental);
```

Этот пример получает `FeedIterator`, который возвращает `Stream`:

```csharp
FeedIterator iteratorWithStreams = container.GetChangeFeedStreamIterator<User>(ChangeFeedStartFrom.Beginning(), ChangeFeedMode.Incremental);
```

Если вы не передаете `FeedRange` в `FeedIterator` , вы можете обрабатывать веб-канал изменений всего контейнера в удобном для вас темпе. Ниже приведен пример, с которого начинается чтение всех изменений, начиная с текущего времени:

```csharp
FeedIterator iteratorForTheEntireContainer = container.GetChangeFeedStreamIterator<User>(ChangeFeedStartFrom.Now(), ChangeFeedMode.Incremental);

while (iteratorForTheEntireContainer.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorForTheEntireContainer.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch {
        Console.WriteLine($"No new changes");
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
}
```

Так как канал изменений фактически является бесконечным списком элементов, охватывающих все будущие операции записи и обновления, значение `HasMoreResults` всегда равно true. При попытке чтения веб-канала изменений и отсутствии новых изменений вы получите исключение. В приведенном выше примере исключение обрабатывается в течение 5 секунд, после чего выполняется повторная проверка на наличие изменений.

## <a name="consuming-a-partition-keys-changes"></a>Обработка изменений по ключу секции

В некоторых случаях требуется обработка изменений только для определенного ключа раздела. Вы можете получить `FeedIterator` для определенного ключа секции и обработать изменения так же, как и для всего контейнера.

```csharp
FeedIterator<User> iteratorForPartitionKey = container.GetChangeFeedIterator<User>(
    ChangeFeedStartFrom.Beginning(FeedRange.FromPartitionKey(new PartitionKey("PartitionKeyValue")), ChangeFeedMode.Incremental));

while (iteratorForThePartitionKey.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorForThePartitionKey.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch (CosmosException exception) when (exception.StatusCode == System.Net.HttpStatusCode.NotModified)
    {
        Console.WriteLine($"No new changes");
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
}
```

## <a name="using-feedrange-for-parallelization"></a>Использование FeedRange для параллелизации

[Обработчик канала изменений](change-feed-processor.md) автоматически распределяет нагрузку между несколькими потребителями. В модели извлечения веб-канала изменений можно применить `FeedRange` для параллелизации обработки веб-канала изменений. `FeedRange` представляет определенный диапазон значений ключа секции.

Следующий пример демонстрирует получение списка диапазонов для контейнера.

```csharp
IReadOnlyList<FeedRange> ranges = await container.GetFeedRangesAsync();
```

При получении списка FeedRange для контейнера вы получите один `FeedRange` для каждой [физической секции](partitioning-overview.md#physical-partitions).

С помощью `FeedRange` можно создать `FeedIterator` для параллелизации обработки веб-канала изменений на нескольких компьютерах или потоках. В отличие от предыдущего примера, который показал, как получить `FeedIterator` для всего контейнера или одного ключа секции, можно использовать фидранжес для получения нескольких фидитераторс, которые могут обрабатывать веб-канал изменений параллельно.

Если вы хотите использовать FeedRange, создайте процесс оркестрации, который получает экземпляры FeedRange и распространяет их по компьютерам. Это распространение можно выполнить следующими способами.

* Используйте `FeedRange.ToJsonString` и передайте это строковое значение. Потребители могут использовать это значение для `FeedRange.FromJsonString`.
* Если распределение выполняется в том же процессе, передавайте ссылку на объект `FeedRange`.

Ниже приведен пример чтения из начала веб-канала изменений для контейнера с использованием двух гипотетических компьютеров, работающих параллельно.

Компьютер 1:

```csharp
FeedIterator<User> iteratorA = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(ranges[0]), ChangeFeedMode.Incremental);
while (iteratorA.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorA.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch (CosmosException exception) when (exception.StatusCode == System.Net.HttpStatusCode.NotModified)
    {
        Console.WriteLine($"No new changes");
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
}
```

Компьютер 2:

```csharp
FeedIterator<User> iteratorB = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(ranges[1]), ChangeFeedMode.Incremental);
while (iteratorB.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorA.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch (CosmosException exception) when (exception.StatusCode == System.Net.HttpStatusCode.NotModified)
    {
        Console.WriteLine($"No new changes");
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
}
```

## <a name="saving-continuation-tokens"></a>Сохранение маркеров продолжения

Вы можете сохранить текущее расположение `FeedIterator`, создав маркер продолжения. Маркер продолжения — это строковое значение, которое отслеживает последние изменения, обработанные в FeedIterator. Это позволяет позднее возобновить `FeedIterator` с той же позиции. Следующий код постоянно считывает веб-канал изменений с момента создания контейнера. Когда не останется доступных изменений, он сохраняет маркер продолжения, чтобы позднее возобновить обработку этого веб-канала изменений.

```csharp
FeedIterator<User> iterator = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(), ChangeFeedMode.Incremental);

string continuation = null;

while (iterator.HasMoreResults)
{
   try { 
        FeedResponse<User> users = await iterator.ReadNextAsync();
        continuation = users.ContinuationToken;

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
   }
    catch (CosmosException exception) when (exception.StatusCode == System.Net.HttpStatusCode.NotModified)
    {
        Console.WriteLine($"No new changes");
        await Task.Delay(TimeSpan.FromSeconds(5));
    }   
}

// Some time later
FeedIterator<User> iteratorThatResumesFromLastPoint = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.ContinuationToken(continuation), ChangeFeedMode.Incremental);
```

Пока существует контейнер Cosmos, маркер продолжения FeedIterator считается действительным.

## <a name="next-steps"></a>Дальнейшие действия

* [Работа с поддержкой веб-канала изменений в Azure Cosmos DB](change-feed.md)
* [Использование обработчика для канала изменений](change-feed-processor.md)
* [Активация Функций Azure](change-feed-functions.md)
