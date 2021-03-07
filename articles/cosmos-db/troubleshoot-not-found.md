---
title: Устранение неполадок Azure Cosmos DB не найденных исключений
description: Узнайте, как диагностировать и исправить исключения, не найденные.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 22cce2c620d23ab477de5d92bb8c6d4f5ef5a493
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102425130"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-not-found-exceptions"></a>Диагностика и устранение неполадок Azure Cosmos DB не найденных исключений
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Код состояния HTTP 404 означает, что ресурс больше не существует.

## <a name="expected-behavior"></a>Ожидаемое поведение
Существует множество допустимых сценариев, в которых приложение принимает код 404 и правильно обрабатывает сценарий.

## <a name="a-not-found-exception-was-returned-for-an-item-that-should-exist-or-does-exist"></a>Было возвращено исключение "не найдено" для элемента, который должен существовать или существовать
Ниже приведены возможные причины, по которым код состояния 404 возвращается, если элемент должен существовать или существовать.

### <a name="the-read-session-is-not-available-for-the-input-session-token"></a>The read session is not available for the input session token (Сеанс чтения для входного маркера сеанса недоступен)

#### <a name="solution"></a>Решение.
1. Обновите текущий пакет SDK до последней доступной версии. Наиболее частые причины этой ошибки исправлены в новых версиях пакета SDK.

### <a name="race-condition"></a>Состояние гонки
Существует несколько экземпляров пакета SDK, и чтение произошло до записи.

#### <a name="solution"></a>Решение.
1. Согласованность учетной записи по умолчанию для Azure Cosmos DB — согласованность сеанса. При создании или обновлении элемента ответ возвращает маркер сеанса, который может быть передан между экземплярами пакета SDK, чтобы гарантировать чтение запроса чтения из реплики с таким изменением.
1. Измените [уровень согласованности](./consistency-levels.md) на [более высокий уровень](./consistency-levels.md).

### <a name="invalid-partition-key-and-id-combination"></a>Недопустимая комбинация ключа секции и идентификатора
Сочетание ключа и идентификатора секции не является допустимым.

#### <a name="solution"></a>Решение.
Исправьте логику приложения, которая вызывает неверное сочетание. 

### <a name="invalid-character-in-an-item-id"></a>Недопустимый символ в ИДЕНТИФИКАТОРе элемента
Элемент вставляется в Azure Cosmos DB с [недопустимым символом](/dotnet/api/microsoft.azure.documents.resource.id#remarks) в идентификаторе элемента.

#### <a name="solution"></a>Решение.
Измените идентификатор на другое значение, которое не содержит специальных символов. Если изменить идентификатор не удается, можно закодировать идентификатор с помощью Base64 для экранирования специальных символов. Base64 по-прежнему может создать имя с недопустимым символом "/", который необходимо заменить.

Элементы, уже вставленные в контейнер для идентификатора, могут быть заменены значениями RID вместо ссылок на основе имен.
```c#
// Get a container reference that uses RID values.
ContainerProperties containerProperties = await this.Container.ReadContainerAsync();
string[] selfLinkSegments = containerProperties.SelfLink.Split('/');
string databaseRid = selfLinkSegments[1];
string containerRid = selfLinkSegments[3];
Container containerByRid = this.cosmosClient.GetContainer(databaseRid, containerRid);

// Invalid characters are listed here.
//https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id#remarks
FeedIterator<JObject> invalidItemsIterator = this.Container.GetItemQueryIterator<JObject>(
    @"select * from t where CONTAINS(t.id, ""/"") or CONTAINS(t.id, ""#"") or CONTAINS(t.id, ""?"") or CONTAINS(t.id, ""\\"") ");
while (invalidItemsIterator.HasMoreResults)
{
    foreach (JObject itemWithInvalidId in await invalidItemsIterator.ReadNextAsync())
    {
        // Choose a new ID that doesn't contain special characters.
        // If that isn't possible, then Base64 encode the ID to escape the special characters.
        byte[] plainTextBytes = Encoding.UTF8.GetBytes(itemWithInvalidId["id"].ToString());
        itemWithInvalidId["id"] = Convert.ToBase64String(plainTextBytes).Replace('/', '!');

        // Update the item with the new ID value by using the RID-based container reference.
        JObject item = await containerByRid.ReplaceItemAsync<JObject>(
            item: itemWithInvalidId,
            ID: itemWithInvalidId["_rid"].ToString(),
            partitionKey: new Cosmos.PartitionKey(itemWithInvalidId["status"].ToString()));

        // Validating the new ID can be read by using the original name-based container reference.
        await this.Container.ReadItemAsync<ToDoActivity>(
            item["id"].ToString(),
            new Cosmos.PartitionKey(item["status"].ToString())); ;
    }
}
```

### <a name="time-to-live-purge"></a>Очистка времени на динамическую очистку
Для элемента было задано свойство [срока жизни (TTL)](./time-to-live.md) . Элемент очищен из-за истечения срока действия свойства TTL.

#### <a name="solution"></a>Решение.
Измените свойство TTL, чтобы запретить очистку элемента.

### <a name="lazy-indexing"></a>Отложенное индексирование
[Отложенное индексирование](index-policy.md#indexing-mode) не было перехвачено.

#### <a name="solution"></a>Решение.
Дождитесь завершения индексирования или измените политику индексирования.

### <a name="parent-resource-deleted"></a>Родительский ресурс удален
База данных или контейнер, в котором находится элемент, удалена.

#### <a name="solution"></a>Решение.
1. [Восстановите](./configure-periodic-backup-restore.md#request-restore) родительский ресурс или повторно создайте ресурсы.
1. Создайте новый ресурс для замены удаленного ресурса.

### <a name="7-containercollection-names-are-case-sensitive"></a>7. в именах контейнеров и коллекций учитывается регистр.
В Cosmos DB имена контейнеров и коллекций чувствительны к регистру.

#### <a name="solution"></a>Решение.
Обязательно используйте точное имя при подключении к Cosmos DB.

## <a name="next-steps"></a>Дальнейшие действия
* [Диагностика и устранение неполадок](troubleshoot-dot-net-sdk.md) при использовании пакета SDK для Azure Cosmos DB .NET.
* Ознакомьтесь с рекомендациями по производительности для [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) и [.NET v2](performance-tips.md).
* [Диагностика и устранение неполадок](troubleshoot-java-sdk-v4-sql.md) при использовании пакета SDK для Java версии 4 Azure Cosmos DB.
* Сведения о рекомендациях по производительности для [пакета SDK для Java v4](performance-tips-java-sdk-v4-sql.md).
