---
title: Устранение проблем с запросами при использовании Azure Cosmos DB API для MongoDB
description: Узнайте, как определять, диагностировать и устранять неполадки Azure Cosmos DB API для запросов MongoDB.
author: timsander1
ms.service: cosmos-db
ms.topic: troubleshooting
ms.subservice: cosmosdb-mongo
ms.date: 03/02/2021
ms.author: tisande
ms.reviewer: sngun
ms.openlocfilehash: 5302cb7bb3f4683d200f6f9ea106991bb934fc17
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101659908"
---
# <a name="troubleshoot-query-issues-when-using-the-azure-cosmos-db-api-for-mongodb"></a>Устранение проблем с запросами при использовании Azure Cosmos DB API для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье рассматривается общий рекомендуемый подход для устранения проблем с запросами в Azure Cosmos DB. Хотя не следует думать, что действия, описанные в этой статье, дают полную защиту от потенциальных проблем с запросами, мы включили сюда наиболее распространенные советы по повышению производительности. Эту статью следует использовать в качестве начального места для устранения недорогих или ресурсоемких запросов в API Azure Cosmos DB для MongoDB. Если вы используете API Azure Cosmos DB Core (SQL), см. статью [руководство по устранению неполадок запросов API SQL](troubleshoot-query-performance.md) .

Оптимизации запросов в Azure Cosmos DB распределены по категориям следующим образом:

- Оптимизация, снижающая затраты единиц запроса (RU) на запрос.
- Оптимизация, которая просто сокращает задержку.

Если вы сократите число единиц запроса, то, как правило, также сокращается задержка.

В этой статье приведены примеры, которые можно создать повторно с помощью [набора данных информации питании](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json).

> [!NOTE] 
> В этой статье предполагается, что вы используете API Azure Cosmos DB для учетных записей MongoDB с версией 3,6 и более поздними версиями. Некоторые запросы, которые плохо работают в версии 3,2, имеют значительные улучшения в версиях 3.6 и более поздних. Выполните обновление до версии 3,6, выполнив [запрос на поддержку](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="use-explain-command-to-get-metrics"></a>Получение метрик с помощью команды $explain

При оптимизации запроса в Azure Cosmos DB первым шагом всегда будет [Получение оплаты за единицу](find-request-unit-charge-mongodb.md) запроса. В качестве грубой рекомендации следует рассмотреть способы снижения оплаты за запросы с оплатой, превышающими 50. 

Кроме получения оплаты за ЕДИНИЦу, следует использовать `$explain` команду, чтобы получить метрики использования запросов и индексов. Ниже приведен пример, в котором выполняется запрос и используется `$explain` команда для отображения метрик использования запросов и индексов.

**Команда $explain:**

```
db.coll.find({foodGroup: "Baby Foods"}).explain({"executionStatistics": true })
```

**Выходные данные:**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 905.2888,
            "timeExclusiveMS" : 905.2888,
            "in" : 362,
            "out" : 362,
            "details" : {
                "database" : "db-test",
                "collection" : "collection-test",
                "query" : {
                    "foodGroup" : {
                        "$eq" : "Baby Foods"
                    }
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "e68e6bdd-5e89-4ec5-b053-3dbbc2428140",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 788.5867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 362,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 362,
                    "outputDocumentSizeBytes" : 2553535,
                    "indexHitRatio" : 0.0016802042237178,
                    "totalQueryExecutionTimeMS" : 777.72,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.19,
                        "logicalPlanBuildTimeMS" : 0.14,
                        "physicalPlanBuildTimeMS" : 0.09,
                        "queryOptimizationTimeMS" : 0.03
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 687.22,
                    "vmExecutionTimeMS" : 774.09,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 37.45,
                        "systemFunctionExecutionTimeMS" : 10.82,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 49.42
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

`$explain`Выходные данные команды имеют длину и имеют подробные сведения о выполнении запроса. Однако в общем случае следует обратить внимание на несколько разделов, при оптимизации производительности запросов:

| Метрика | Описание | 
| ------ | ----------- |
| `timeInclusiveMS` | Задержка запросов в серверной части |
| `pathsIndexed` | Показывает индексы, используемые запросом | 
| `pathsNotIndexed` | Показывает индексы, которые мог бы использовать запрос, если он доступен | 
| `shardInformation` | Сводка по производительности запросов для определенной [физической секции](./partitioning-overview.md#physical-partitions) | 
| `retrievedDocumentCount` | Число документов, загруженных обработчиком запросов | 
| `outputDocumentCount` | Число документов, возвращенных в результатах запроса | 
| `estimatedDelayFromRateLimitingInMilliseconds` | Оценка дополнительной задержки запросов из-за ограничения скорости | 

После получения метрик запроса Сравните с параметром `retrievedDocumentCount` `outputDocumentCount` для запроса. Используйте это сравнение, чтобы определить соответствующие разделы для проверки в этой статье. `retrievedDocumentCount`— Число документов, которые подсистема запросов должна загрузить. `outputDocumentCount`— Это количество документов, которые были необходимы для результатов запроса. Если значение `retrievedDocumentCount`  значительно выше, то существовала `outputDocumentCount` хотя бы одна часть запроса, которая не смогла использовать индекс и требуется для проверки.

Ознакомьтесь со следующими разделами, чтобы получить представление об актуальных для вашего сценариях оптимизациях запросов.

### <a name="querys-ru-charge-is-too-high"></a>Плата в единицах запроса слишком высока.

#### <a name="retrieved-document-count-is-significantly-higher-than-output-document-count"></a>Число полученных документов значительно превышает число выходных документов.

- [Включить необходимые индексы.](#include-necessary-indexes)

- [Определить, какие операции агрегирования используют индекс.](#understand-which-aggregation-operations-use-the-index)

#### <a name="retrieved-document-count-is-approximately-equal-to-output-document-count"></a>Число полученных документов приблизительно равно числу выходных документов

- [Сведите к минимуму число запросов между секциями.](#minimize-cross-partition-queries)

### <a name="querys-ru-charge-is-acceptable-but-latency-is-still-too-high"></a>Плата в единицах запроса приемлема, но задержка по-прежнему слишком высока.

- [Добейтесь большей близости.](#improve-proximity)

- [Увеличьте подготовленную пропускную способность.](#increase-provisioned-throughput)

## <a name="queries-where-retrieved-document-count-exceeds-output-document-count"></a>Запросы, в которых количество полученных документов превышает число выходных документов

 `retrievedDocumentCount`— Число документов, которые подсистема запросов загрузила. `outputDocumentCount`— Это число документов, возвращенных запросом. Если значение `retrievedDocumentCount` значительно выше, то существовала `outputDocumentCount` хотя бы одна часть запроса, которая не смогла использовать индекс и требуется для проверки.

Ниже приведен пример запроса сканирования, который не был полностью обработан индексом.

**Команда $explain:**

```
db.coll.find(
  {
    $and : [
            { "foodGroup" : "Cereal Grains and Pasta"}, 
            { "description" : "Oat bran, cooked"}
        ]
  }
).explain({"executionStatistics": true })
```

**Выходные данные:**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 436.5716,
            "timeExclusiveMS" : 436.5716,
            "in" : 1,
            "out" : 1,
            "details" : {
                "database" : "db-test",
                "collection" : "indexing-test",
                "query" : {
                    "$and" : [ 
                        {
                            "foodGroup" : {
                                "$eq" : "Cereal Grains and Pasta"
                            }
                        }, 
                        {
                            "description" : {
                                "$eq" : "Oat bran, cooked"
                            }
                        }
                    ]
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "13a5977e-a10a-4329-b68e-87e4f0081cac",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 435.4867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 1,
                    "outputDocumentSizeBytes" : 6064,
                    "indexHitRatio" : 0.0,
                    "totalQueryExecutionTimeMS" : 433.64,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.12,
                        "logicalPlanBuildTimeMS" : 0.09,
                        "physicalPlanBuildTimeMS" : 0.1,
                        "queryOptimizationTimeMS" : 0.02
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 387.44,
                    "vmExecutionTimeMS" : 432.93,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 45.36,
                        "systemFunctionExecutionTimeMS" : 16.86,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 0.13
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

Значение `retrievedDocumentCount` (8618) значительно выше, чем `outputDocumentCount` (1), что подразумевает, что для этого запроса требуется сканирование документа. 

### <a name="include-necessary-indexes"></a>Включить необходимые индексы

Необходимо проверить `pathsNotIndexed` массив и добавить эти индексы. В этом примере пути `foodGroup` и `description` должны быть проиндексированы.

```json
"pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ]
```

Рекомендации по индексированию в API Azure Cosmos DB для MongoDB отличаются от MongoDB. В API Azure Cosmos DB для MongoDB составные индексы используются только в запросах, которые должны эффективно сортироваться по нескольким свойствам. При наличии запросов с фильтрами для нескольких свойств необходимо создать индексы одного поля для каждого из этих свойств. Предикаты запросов могут использовать несколько индексов одного поля.

[Индексы с подстановочными знаками](mongodb-indexing.md#wildcard-indexes) могут упростить индексирование. В отличие от MongoDB, индексы с подстановочными знаками могут поддерживать несколько полей в предикатах запросов. При использовании одного индекса вместо создания отдельного индекса для каждого свойства различие в производительности запросов не будет. Добавление индекса с подстановочными знаками для всех свойств — самый простой способ оптимизации всех запросов.

Новые индексы можно добавлять в любое время без влияния на доступность для записи или чтения. Можно [отслеживать ход преобразования индекса](./how-to-manage-indexing-policy.md#dotnet-sdk).

### <a name="understand-which-aggregation-operations-use-the-index"></a>Сведения об операциях агрегирования, использующих индекс

В большинстве случаев операции агрегирования в API Azure Cosmos DB для MongoDB будут частично использовать индексы. Как правило, обработчик запросов будет применять фильтры по равенству и диапазонам первыми и использовать индексы. После применения этих фильтров обработчик запросов может оценить дополнительные фильтры и прибегнуть к загрузке оставшихся документов для выполнения статистического вычисления, если это необходимо. 

Приведем пример:

```
db.coll.aggregate( [
   { $match: { foodGroup: 'Fruits and Fruit Juices' } },
   {
     $group: {
        _id: "$foodGroup",
        total: { $max: "$version" }
     }
   }
] )
```

В этом случае индексы могут оптимизировать `$match` этап. Добавление индекса для `foodGroup` значительно повысит производительность запросов. Как и в MongoDB, `$match` для максимально эффективного использования индексов следует поместить как можно раньше в конвейере агрегирования.

В API Azure Cosmos DB для MongoDB индексы не используются для фактической статистической обработки, которая в данном случае имеет значение `$max` . Добавление индекса для `version` не повысит производительность запросов.

## <a name="queries-where-retrieved-document-count-is-equal-to-output-document-count"></a>Запросы, в которых число полученных документов равно числу выходных документов

Если значение `retrievedDocumentCount` приблизительно равно значению `outputDocumentCount` , механизму запросов не нужно сканировать много ненужных документов.

### <a name="minimize-cross-partition-queries"></a>Сведите к минимуму число запросов между секциями.

Azure Cosmos DB использует [секционирование](partitioning-overview.md) для масштабирования отдельных контейнеров по мере роста числа единиц запросов и потребностей хранилища данных. Каждый физическая секция имеет отдельный и независимый индекс. Если запрос имеет фильтр проверки на равенство, соответствующий ключу секции контейнера, необходимо проверить только индекс соответствующей секции. Эта оптимизация сокращает общее число единиц запроса, необходимых для запроса. [Узнайте больше о различиях между запросами между секциями и запросами между секциями](how-to-query-container.md).

При наличии большого количества подготовленных единиц запроса (более 30 000) или большого объема хранимых данных (более чем примерно 100 ГБ) контейнер, вероятно, будет достаточно велик, чтобы увидеть значительное стоимости в единицах запроса. 

Можно проверить массив, `shardInformation` чтобы понять метрики запросов для каждой отдельной физической секции. Число уникальных `shardKeyRangeId` значений — это количество физических секций, в которых необходимо выполнить запрос. В этом примере запрос выполнялся в четырех физических секциях. Важно понимать, что выполнение полностью независимо от использования индексов. Иными словами, запросы между секциями по-прежнему могут использовать индексы.

```json
  "shardInformation" : [ 
                    {
                        "activityId" : "42f670a8-a201-4c58-8023-363ac18d9e18",
                        "shardKeyRangeId" : "5",
                        "durationMS" : 24.3859,
                        "preemptions" : 1,
                        "outputDocumentCount" : 463,
                        "retrievedDocumentCount" : 463
                    }, 
                    {
                        "activityId" : "a8bf762a-37b9-4c07-8ed4-ae49961373c0",
                        "shardKeyRangeId" : "2",
                        "durationMS" : 35.8328,
                        "preemptions" : 1,
                        "outputDocumentCount" : 905,
                        "retrievedDocumentCount" : 905
                    }, 
                    {
                        "activityId" : "3754e36b-4258-49a6-8d4d-010555628395",
                        "shardKeyRangeId" : "1",
                        "durationMS" : 67.3969,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1479,
                        "retrievedDocumentCount" : 1479
                    }, 
                    {
                        "activityId" : "a69a44ee-db97-4fe9-b489-3791f3d52878",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 185.1523,
                        "preemptions" : 1,
                        "outputDocumentCount" : 867,
                        "retrievedDocumentCount" : 867
                    }
                ]
```

## <a name="optimizations-that-reduce-query-latency"></a>Оптимизации, сокращающие задержку запросов.

Часто стоимость в единицах запроса может быть приемлемой, когда задержка запроса все еще слишком высока. В следующих разделах представлен обзор советов по сокращению задержки запросов. Если один и тот же запрос выполняется несколько раз для одного и того же набора данных, то каждый раз будет взиматься плата за ЕДИНИЦу. Однако задержка запросов может варьироваться при выполнении.

### <a name="improve-proximity"></a>Добейтесь большей близости.

Запросы, которые выполняются из иного региона, чем тот, где находится учетная запись Azure Cosmos DB, будут иметь большую задержку, чем если бы они выполнялись в одном регионе. Например, если выполнять код на своем настольном компьютере, задержка повысится на десятки и сотни миллисекунд (если не больше), по сравнению с запросом, поступившим с виртуальной машины в том же регионе Azure, что и Azure Cosmos DB. Можно легко [глобально распределить данные в Azure Cosmos DB](tutorial-global-distribution-mongodb.md), чтобы обеспечить близость данных к приложениям.

### <a name="increase-provisioned-throughput"></a>Увеличьте подготовленную пропускную способность.

В Azure Cosmos DB подготовленная пропускная способность измеряется в единицах запроса (ЕЗ). Представьте, что у вас есть запрос, использующий 5 ЕЗ пропускной способности. Например, если вы подготавливаете 1000 ЕЗ, вы сможете выполнить запрос 200 раз в секунду. При попытке выполнить запрос при недостаточной пропускной способности Azure Cosmos DB будет ограничивать количество запросов. API Azure Cosmos DB для MongoDB будет автоматически повторять этот запрос после ожидания в течение короткого времени. Регулируемые запросы занимают больше времени, поэтому увеличение подготовленной пропускной способности может улучшить задержку запросов.

Это значение `estimatedDelayFromRateLimitingInMilliseconds` дает представление о возможных преимуществах задержки при увеличении пропускной способности.

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с производительностью запросов (API SQL)](troubleshoot-query-performance.md)
* [Управление индексированием в API Azure Cosmos DB для MongoDB](mongodb-indexing.md)
