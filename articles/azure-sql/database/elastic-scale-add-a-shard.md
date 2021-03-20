---
title: Добавление сегмента с помощью инструментов эластичных баз данных
description: Информация о том, как использовать интерфейсы API эластичного масштабирования для добавления новых сегментов в набор сегментов.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/03/2019
ms.openlocfilehash: efab0234d428a8283845946289cdd1e8a17ded26
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92792061"
---
# <a name="adding-a-shard-using-elastic-database-tools"></a>Добавление сегмента с использованием средств эластичных баз данных
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="to-add-a-shard-for-a-new-range-or-key"></a>Добавление сегмента для нового диапазона или ключа

Часто необходимо, чтобы приложение добавило в существующую карту сегментов новые сегменты для обработки данных, поступление которых ожидается из новых ключей и диапазонов ключей. Например, приложение с сегментированием по идентификатору клиента должно подготовить новый сегмент для нового клиента или при обработке данных с сегментированием по месяцам требуется создание нового сегмента перед началом следующего месяца.

Если новый диапазон ключей не входит в существующее сопоставление, добавление сегмента и привязка к нему нового ключа или диапазона становится простой задачей.

### <a name="example--adding-a-shard-and-its-range-to-an-existing-shard-map"></a>Пример: добавление сегмента и его диапазона в имеющееся сопоставление сегментов

В этом примере используются методы TryGetShard ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.trygetshard), [.NET](/previous-versions/azure/dn823929(v=azure.100))), CreateShard ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.createshard), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.createshard)), CreateRangeMapping ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.createrangemapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) и создается экземпляр класса ShardLocation ([Java](/java/api/com.microsoft.azure.elasticdb.shard.base.shardlocation), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardlocation)). В примере ниже для хранения диапазона [300, 400) создается база данных с именем **sample_shard_2** и все необходимые объекты схемы внутри нее.  

```csharp
// sm is a RangeShardMap object.
// Add a new shard to hold the range being added.
Shard shard2 = null;

if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2))
{
    shard2 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_2"));  
}

// Create the mapping and associate it with the new shard
sm.CreateRangeMapping(new RangeMappingCreationInfo<long>
                            (new Range<long>(300, 400), shard2, MappingStatus.Online));
```

Для версии .NET можно также использовать PowerShell в качестве альтернативы, чтобы создать диспетчер карты сегментов. Пример представлен [здесь](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

## <a name="to-add-a-shard-for-an-empty-part-of-an-existing-range"></a>Добавление сегмента для пустой части существующего диапазона

В некоторых случаях к этому моменту диапазон может быть уже сопоставлен с сегментом и частично заполнен данными, но теперь последующие данные должны направляться в другой сегмент. Такая ситуация, например, возникает, если сегментация выполняется по диапазону с разбиением на дни и при этом для сегмента уже выделено 50 дней, однако на 24 день понадобилось, чтобы будущие данные поступали в другой сегмент. [Средство разбиения и объединения](elastic-scale-overview-split-and-merge.md) эластичных баз данных может выполнить эту операцию. Но если перемещать данные необязательно (например, данные для диапазона дней 25–50, т. е. с 25-го дня включительно по 50-й день не включительно, еще не существуют), эту операцию можно выполнить полностью непосредственно с помощью интерфейсов API управления картой сегментов.

### <a name="example-splitting-a-range-and-assigning-the-empty-portion-to-a-newly-added-shard"></a>Пример: разбиение диапазона и присвоение пустой части новому добавленному сегменту

В рамках примера были созданы база данных с именем sample_shard_2 и все необходимые объекты схемы, содержащиеся в ней.  

```csharp
// sm is a RangeShardMap object.
// Add a new shard to hold the range we will move
Shard shard2 = null;

if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2))
{
    shard2 = sm.CreateShard(new ShardLocation(shardServer,"sample_shard_2"));  
}

// Split the Range holding Key 25
sm.SplitMapping(sm.GetMappingForKey(25), 25);

// Map new range holding [25-50) to different shard:
// first take existing mapping offline
sm.MarkMappingOffline(sm.GetMappingForKey(25));

// now map while offline to a different shard and take online
RangeMappingUpdate upd = new RangeMappingUpdate();
upd.Shard = shard2;
sm.MarkMappingOnline(sm.UpdateMapping(sm.GetMappingForKey(25), upd));
```

**Важно**. Используйте этот метод только в том случае, если вы уверены, что диапазон для обновленного сопоставления пуст.  Методы выше не позволяют проверить данные для перемещаемого диапазона, так что лучше включить проверки в код.  Если перемещаемый диапазон содержит строки, фактическое распределение данных не будет соответствовать обновленному сопоставлению сегментов. В таких случаях для выполнения операции используйте [средство разбиения и объединения](elastic-scale-overview-split-and-merge.md) .  

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]