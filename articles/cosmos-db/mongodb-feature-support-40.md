---
title: Поддержка возможностей и синтаксиса в API Azure Cosmos DB для MongoDB (версии сервера 4.0)
description: Сведения о поддержке возможностей и синтаксиса API Azure Cosmos DB для MongoDB (версии сервера 4.0). Сведения о командах базы данных, поддержке языка запросов, типах данных, командах конвейера агрегирования и поддерживаемых операторах.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 03/02/2021
author: gahl-levy
ms.author: gahllevy
ms.openlocfilehash: 9eebc77c5b3d9402c766320fddfdaf05d50b574f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102485407"
---
# <a name="azure-cosmos-dbs-api-for-mongodb-40-server-version-supported-features-and-syntax"></a>API Azure Cosmos DB для MongoDB (версии сервера 4.0): поддержка возможностей и синтаксиса
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB — это глобально распределенная многомодельная служба базы данных Майкрософт. С API Azure Cosmos DB для MongoDB можно взаимодействовать, используя любой из [драйверов](https://docs.mongodb.org/ecosystem/drivers) с открытым кодом для клиента MongoDB. API Azure Cosmos DB для MongoDB позволяет использовать имеющиеся драйверы клиента благодаря [сетевому протоколу](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol) MongoDB.

С помощью API Azure Cosmos DB для MongoDB вы можете пользоваться всеми функциями MongoDB, к которым вы привыкли, и возможностями Cosmos DB для организаций: [глобальное распространение](distribute-data-globally.md), [автоматическое сегментирование](partitioning-overview.md), гарантии по уровням доступности и задержкам, шифрование неактивных данных, резервное копирование и многое другое.

## <a name="protocol-support"></a>Поддержка протоколов

Ниже перечислены поддерживаемые операторы, а также ограничения и исключения. Любой драйвер клиента, который распознает эти протоколы, должен иметь возможность подключения к API Azure Cosmos DB для MongoDB. При использовании API Azure Cosmos DB для учетных записей MongoDB учетные записи версий 3.6+ будут иметь конечную точку в формате `*.mongo.cosmos.azure.com`, в то время как в версии 3.2 конечная точка будет в формате `*.documents.azure.com`.

> [!NOTE]
> В этой статье приведены только поддерживаемые серверные команды и не указаны функции-оболочки на стороне клиента. Функции-оболочки на стороне клиента, такие как `deleteMany()` и `updateMany()`, внутренне используют серверные команды `delete()` и `update()`. Функции, использующие поддерживаемые серверные команды, совместимы с API Azure Cosmos DB для MongoDB.

## <a name="query-language-support"></a>Поддержка языка запросов

API Azure Cosmos DB для MongoDB предоставляет полную поддержку всех конструкций языка запросов MongoDB. Ниже приводится подробный список поддерживаемых операций, операторов, этапов, команд и параметров.

## <a name="database-commands"></a>Команды базы данных

API Azure Cosmos DB для MongoDB поддерживает приведенные ниже команды базы данных.

### <a name="query-and-write-operation-commands"></a>Команды для запросов и записи

| Get-Help | Поддерживается |
|---------|---------|
| [потоки изменений](mongodb-change-streams.md) | Да |
| удалить | Да |
| eval | Нет |
| поиск | Да |
| findAndModify | Да |
| getLastError | Да |
| getMore | Да |
| getPrevError | нет |
| insert | Да |
| parallelCollectionScan | Нет |
| resetError | нет |
| обновить | Да |

### <a name="transaction-commands"></a>Команды транзакций

| Get-Help | Поддерживается |
|---------|---------|
| abortTransaction | Да |
| commitTransaction | Да |

### <a name="authentication-commands"></a>Команды для аутентификации

| Get-Help | Поддерживается |
|---------|---------|
| authenticate | Да |
| getnonce | Да |
| logout | Да |

### <a name="administration-commands"></a>Команды для администрирования

| Get-Help | Поддерживается |
|---------|---------|
| cloneCollectionAsCapped | нет |
| collMod | нет |
| connectionStatus | нет |
| convertToCapped | нет |
| copydb | нет |
| create | Да |
| createIndexes | Да |
| currentOp | Да |
| drop | Да |
| dropDatabase | Да |
| dropIndexes | Да |
| filemd5 | Да |
| killCursors | Да |
| killOp | нет |
| listCollections | Да |
| listDatabases | Да |
| listIndexes | Да |
| reIndex | Да |
| renameCollection | нет |

### <a name="diagnostics-commands"></a>Команды для диагностики

| Get-Help | Поддерживается |
|---------|---------|
| buildInfo | Да |
| collStats | Да |
| connPoolStats | нет |
| connectionStatus | нет |
| dataSize | нет |
| dbHash | нет |
| dbStats | Да |
| explain | Да |
| features | нет |
| hostInfo | Да |
| listDatabases | Да |
| listCommands | нет |
| профилировщик | нет |
| serverStatus | нет |
| top | нет |
| whatsmyuri | Да |

## <a name="aggregation-pipeline"></a><a name="aggregation-pipeline"></a>Конвейер агрегирования

### <a name="aggregation-commands"></a>Команды статистической обработки

| Get-Help | Поддерживается |
|---------|---------|
| статистическое выражение | Да |
| count | Да |
| distinct | Да |
| mapReduce | нет |

### <a name="aggregation-stages"></a>Этапы статистической обработки

| Get-Help | Поддерживается |
|---------|---------|
| $addFields | Да |
| $bucket | нет |
| $bucketAuto | нет |
| $changeStream | Да |
| $collStats | нет |
| $count | Да |
| $currentOp | нет |
| $facet | Да |
| $geoNear | Да |
| $graphLookup | Да |
| $group | Да |
| $indexStats | нет |
| $limit | Да |
| $listLocalSessions | нет |
| $listSessions | нет |
| $lookup | Да |
| $match | Да |
| $out | Да |
| $project | Да |
| $redact | Да |
| $replaceRoot | Да |
| $replaceWith | Нет |
| $sample | Да |
| $skip | Да |
| $sort | Да |
| $sortByCount | Да |
| $unwind | Да |

### <a name="boolean-expressions"></a>Логические выражения

| Get-Help | Поддерживается |
|---------|---------|
| $and | Да |
| $not | Да |
| $or | Да |

### <a name="conversion-expressions"></a>Выражения преобразования

| Get-Help | Поддерживается |
|---------|---------|
| $convert | Да |
| $toBool | Да |
| $toDate | Да |
| $toDecimal | Да |
| $toDouble | Да |
| $toInt | Да |
| $toLong | Да |
| $toObjectId | Да |
| $toString | Да |

### <a name="set-expressions"></a>Выражения для наборов

| Get-Help | Поддерживается |
|---------|---------|
| $setEquals | Да |
| $setIntersection | Да |
| $setUnion | Да |
| $setDifference | Да |
| $setIsSubset | Да |
| $anyElementTrue | Да |
| $allElementsTrue | Да |

### <a name="comparison-expressions"></a>Выражения сравнения

| Get-Help | Поддерживается |
|---------|---------|
| $cmp | Да |
| $eq | Да | 
| $gt | Да | 
| $gte | Да | 
| $lt | Да |
| $lte | Да | 
| $ne | Да | 
| $in | Да | 
| $nin | Да | 

### <a name="arithmetic-expressions"></a>Арифметические выражения

| Get-Help | Поддерживается |
|---------|---------|
| $abs | Да |
| $add | Да |
| $ceil | Да |
| $divide | Да |
| $exp | Да |
| $floor | Да |
| $ln | Да |
| $log | Да |
| $log10 | Да |
| $mod | Да |
| $multiply | Да |
| $pow | Да |
| $sqrt | Да |
| $subtract | Да |
| $trunc | Да |

### <a name="string-expressions"></a>Строковые выражения

| Get-Help | Поддерживается |
|---------|---------|
| $concat | Да |
| $indexOfBytes | Да |
| $indexOfCP | Да |
| $ltrim | Да |
| $rtrim | Да |
| $trim | Да |
| $split | Да |
| $strLenBytes | Да |
| $strLenCP | Да |
| $strcasecmp | Да |
| $substr | Да |
| $substrBytes | Да |
| $substrCP | Да |
| $toLower | Да |
| $toUpper | Да |

### <a name="text-search-operator"></a>Оператор поиска в тексте

| Get-Help | Поддерживается |
|---------|---------|
| $meta | нет |

### <a name="array-expressions"></a>Выражения для массивов

| Get-Help | Поддерживается |
|---------|---------|
| $arrayElemAt | Да |
| $arrayToObject | Да |
| $concatArrays | Да |
| $filter | Да |
| $indexOfArray | Да |
| $isArray | Да |
| $objectToArray | Да |
| $range | Да |
| $reverseArray | Да |
| $reduce | Да |
| $size | Да |
| $slice | Да |
| $zip | Да |
| $in | Да |

### <a name="variable-operators"></a>Операторы переменных

| Get-Help | Поддерживается |
|---------|---------|
| $map | Да |
| $let | Да |

### <a name="system-variables"></a>Системные переменные

| Get-Help | Поддерживается |
|---------|---------|
| $$CURRENT | Да |
| $$DESCEND | Да |
| $$KEEP | Да |
| $$PRUNE | Да |
| $$REMOVE | Да |
| $$ROOT | Да |

### <a name="literal-operator"></a>Литеральный оператор

| Get-Help | Поддерживается |
|---------|---------|
| $literal | Да |

### <a name="date-expressions"></a>Выражения для дат

| Get-Help | Поддерживается |
|---------|---------|
| $dayOfYear | Да |
| $dayOfMonth | Да |
| $dayOfWeek | Да |
| $year | Да |
| $month | Да | 
| $week | Да |
| $hour | Да |
| $minute | Да | 
| $second | Да |
| $millisecond | Да | 
| $dateToString | Да |
| $isoDayOfWeek | Да |
| $isoWeek | Да |
| $dateFromParts | нет | 
| $dateToParts | нет |
| $dateFromString | нет |
| $isoWeekYear | Да |

### <a name="conditional-expressions"></a>Условные выражения

| Get-Help | Поддерживается |
|---------|---------|
| $cond | Да |
| $ifNull | Да |
| $switch | Да |

### <a name="data-type-operator"></a>Оператор типа данных

| Get-Help | Поддерживается |
|---------|---------|
| $type | Да |

### <a name="accumulator-expressions"></a>Выражения аккумулятора

| Get-Help | Поддерживается |
|---------|---------|
| $sum | Да |
| $avg | Да |
| $first | Да |
| $last | Да |
| $max | Да |
| $min | Да |
| $push | Да |
| $addToSet | Да |
| $stdDevPop | Да |
| $stdDevSamp | Да |

### <a name="merge-operator"></a>Оператор объединения

| Get-Help | Поддерживается |
|---------|---------|
| $mergeObjects | Да |

## <a name="data-types"></a>Типы данных

API Azure Cosmos DB для MongoDB поддерживает документы, закодированные в формате BSON MongoDB. Версия API 4.0 расширяет внутреннее использование этого формата для повышения производительности и снижения затрат. Это преимущество распространяется на документы, написанные или обновленные с помощью конечной точки с версией 4.0.
 
В [сценарии обновления](mongodb-version-upgrade.md) документы, написанные до обновления до версии 4.0, не получают преимущество повышенной производительности, пока не будут обновлены с помощью операции записи через конечную точку версии 4.0.

| Get-Help | Поддерживается |
|---------|---------|
| Double | Да |
| Строка | Да |
| Объект | Да |
| Array | Да |
| Binary Data | Да | 
| ObjectId | Да |
| Логическое | Да |
| Дата | Да |
| NULL | Да |
| 32-разрядное целое число (цч) | Да |
| Отметка времени | Да |
| 64-разрядное целое число (длинное) | Да |
| MinKey | Да |
| MaxKey | Да |
| Decimal128 | Да | 
| Регулярное выражение | Да |
| JavaScript | Да |
| JavaScript (с инструкциями SCOPE)| Да |
| Не определено. | Да |

## <a name="indexes-and-index-properties"></a>Индексы и свойства индекса

### <a name="indexes"></a>Индексы

| Get-Help | Поддерживается |
|---------|---------|
| Индекс одного поля | Да |
| Составной индекс | Да |
| Многоключевой индекс | Да |
| Текстовый индекс | нет |
| 2dsphere | Да |
| Двухмерный индекс | нет |
| Хэшированный индекс | Да |

### <a name="index-properties"></a>Свойства индекса

| Get-Help | Поддерживается |
|---------|---------|
| Срок жизни | Да |
| Уникальная идентификация | Да |
| Частично | нет |
| Без учета регистра | нет |
| разреженные; | нет |
| Историческая справка | Да |

## <a name="operators"></a>Операторы

### <a name="logical-operators"></a>Логические операторы

| Get-Help | Поддерживается |
|---------|---------|
| $or | Да |
| $and | Да |
| $not | Да |
| $nor | Да | 

### <a name="element-operators"></a>Операторы элементов

| Get-Help | Поддерживается |
|---------|---------|
| $exists | Да |
| $type | Да |

### <a name="evaluation-query-operators"></a>Операторы запросов вычисления

| Get-Help | Поддерживается |
|---------|---------|
| $expr | нет |
| $jsonSchema | нет |
| $mod | Да |
| $regex | Да |
| $text | Нет (не поддерживается. Вместо этого используйте $regex.)| 
| $where | нет | 

В запросах $regex выражения, привязанные слева, поддерживают поиск по индексу. Но если добавить модификатор i (отключение учета регистра) и m (многостроковое выражение), коллекция будет проверяться во всех выражениях.

Если есть необходимость включить $ или |, желательно создать два (или более) запроса regex. Например, исходный запрос вида `find({x:{$regex: /^abc$/})` следует изменить следующим образом:

`find({x:{$regex: /^abc/, x:{$regex:/^abc$/}})`

Тогда первая часть будет использовать индекс, чтобы ограничить поиск только теми документами, имя которых начинается со стороки ^abc, а вторая часть будет проверять соответствие строк. Оператор вертикальной черты | действует как функция or. Например, запрос `find({x:{$regex: /^abc |^def/})` отбирает документы, в которых значение поля x начинается со строк abc или def. Чтобы использовать индекс, мы рекомендуем разбивать запрос на два разных запроса, соединенных оператором $or, вот так: `find( {$or : [{x: $regex: /^abc/}, {$regex: /^def/}] })`.

### <a name="array-operators"></a>Операторы массива

| Get-Help | Поддерживается | 
|---------|---------|
| $all | Да | 
| $elemMatch | Да | 
| $size | Да | 

### <a name="comment-operator"></a>Оператор комментариев

| Get-Help | Поддерживается | 
|---------|---------|
| $comment | Да | 

### <a name="projection-operators"></a>Операторы проецирования

| Get-Help | Поддерживается |
|---------|---------|
| $elemMatch | Да |
| $meta | нет |
| $slice | Да |

### <a name="update-operators"></a>Обновление операторов

#### <a name="field-update-operators"></a>Операторы обновления полей

| Get-Help | Поддерживается |
|---------|---------|
| $inc | Да |
| $mul | Да |
| $rename | Да |
| $setOnInsert | Да |
| $set | Да |
| $ не задано | Да |
| $min | Да |
| $max | Да |
| $currentDate | Да |

#### <a name="array-update-operators"></a>Операторы обновления массивов

| Get-Help | Поддерживается |
|---------|---------|
| $ | Да |
| $[]| Да |
| $[<identifier>]| Да |
| $addToSet | Да |
| $pop | Да |
| $pullAll | Да |
| $pull | Да |
| $push | Да |
| $pushAll | Да |

#### <a name="update-modifiers"></a>Модификаторы обновления

| Get-Help | Поддерживается |
|---------|---------|
| $each | Да |
| $slice | Да |
| $sort | Да |
| $position | Да |

#### <a name="bitwise-update-operator"></a>Оператор побитового обновления

| Get-Help | Поддерживается |
|---------|---------|
| $bit | Да | 
| $bitsAllSet | нет |
| $bitsAnySet | нет |
| $bitsAllClear | нет |
| $bitsAnyClear | нет |

### <a name="geospatial-operators"></a>Геопространственные операторы

Оператор | Поддерживается | 
--- | --- |
$geoWithin | Да |
$geoIntersects | Да | 
$near | Да |
$nearSphere | Да |
$geometry | Да |
$minDistance | Да |
$maxDistance | Да |
$center | Нет |
$centerSphere | Нет |
$box | Нет |
$polygon | Нет |

## <a name="sort-operations"></a>Сортирование операций

При использовании операции `findOneAndUpdate` поддерживается сортировка операций по одному полю, а по нескольким полям — нет.

## <a name="unique-indexes"></a>Уникальные индексы

[Уникальные индексы](mongodb-indexing.md#unique-indexes) гарантируют, что среди всех документов в коллекции не будет повторяющихся значений определенного поля, то есть для него сохраняется уникальность, как для ключа по умолчанию _id. Вы можете создать уникальные индексы в Azure Cosmos DB, выполнив команду `createIndex` с параметром ограничения `unique`.

```javascript
globaldb:PRIMARY> db.coll.createIndex( { "amount" : 1 }, {unique:true} )
{
    "_t" : "CreateIndexesResponse",
    "ok" : 1,
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 4
}
```

## <a name="compound-indexes"></a>Составные индексы

[Составные индексы](mongodb-indexing.md#compound-indexes-mongodb-server-version-36) дают возможность создать индекс для групп, содержащих до восьми полей. Этот тип индекса отличается от собственных составных индексов MongoDB. В Azure Cosmos DB составные индексы используются для операций сортировки, применяемых к нескольким полям. Чтобы создать составной индекс, необходимо указать в качестве параметра более одного свойства.

```javascript
globaldb:PRIMARY> db.coll.createIndex({"amount": 1, "other":1})
{
    "createdCollectionAutomatically" : false, 
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1
}
```

## <a name="gridfs"></a>GridFS

Azure Cosmos DB поддерживает GridFS через любой драйвер Mongo, совместимый с GridFS.

## <a name="replication"></a>Репликация

Azure Cosmos DB поддерживает собственный механизм автоматической репликации на самых нижних уровнях. Этот механизм применяется и для организации глобальной репликации с низкой задержкой. Cosmos DB не поддерживает запуск или настройку репликации вручную.

## <a name="retryable-writes"></a>Повторяемые операции записи

Cosmos DB пока не поддерживает повторяемые операции записи. Клиентские драйверы должны добавить параметр URL-адреса retryWrites=false в свою строку подключения. Дополнительные параметры URL-адреса можно добавить с помощью префикса &. 

## <a name="sharding"></a>Сегментирование

Azure Cosmos DB поддерживает автоматическое сегментирование на стороне сервера. Он автоматически управляет созданием, размещением и балансировкой сегментов. Azure Cosmos DB не поддерживает выполняемые вручную команды сегментирования. Это означает, что вам не нужно вызывать такие команды, как addShard, balancerStart, moveChunk и т. д. Ключ сегмента необходимо указывать только при создании контейнеров или при запросе данных.

## <a name="sessions"></a>Сеансы

Azure Cosmos DB пока не поддерживает команды сеансов на стороне сервера.

## <a name="time-to-live-ttl"></a>Срок жизни (TTL)

Azure Cosmos DB поддерживает срок жизни (TTL), определяемый по метке времени для документа. Срок жизни можно включить для коллекций, перейдя на [портал Azure](https://portal.azure.com).

## <a name="transactions"></a>Transactions

Операции над несколькими документами поддерживаются в несегментированной коллекции. Операции над несколькими документами не поддерживаются в обычных или сегментированных коллекциях. Время ожидания транзакций составляет 5 секунд.

## <a name="user-and-role-management"></a>Управление пользователями и ролями

Azure Cosmos DB пока не поддерживает концепцию пользователей и ролей. Тем не менее Cosmos DB поддерживает управление доступом Azure на основе ролей (Azure RBAC), а также пароли (ключи) для чтения и записи или только для чтения, которые можно получить с помощью [портала Azure](https://portal.azure.com) (на странице "Строка подключения").

## <a name="write-concern"></a>Проблемы с записью

Некоторые приложения используют [опросы записи](https://docs.mongodb.com/manual/reference/write-concern/), указывающие число ответов, требуемых во время выполнения операции записи. Так как Cosmos DB выполняет репликацию в фоновом режиме, все операции записи, автоматически происходят в режиме кворума по умолчанию. Любые проблемы с записью, определенные в клиентском коде, игнорируются. Дополнительные сведения см. в статье [Настраиваемые уровни согласованности данных в DocumentDB](consistency-levels.md).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Studio 3T](mongodb-mongochef.md) с API Azure Cosmos DB для MongoDB.
- Узнайте, как [использовать Robo 3T](mongodb-robomongo.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.
