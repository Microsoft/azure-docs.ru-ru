---
title: Запрос сегментированных баз данных
description: Выполнение запросов по сегментам с использованием клиентской библиотеки эластичной базы данных.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.topic: how-to
ms.custom: sqldbrb=1
author: stevestein
ms.author: sstein
ms.date: 01/25/2019
ms.openlocfilehash: 5a0dd12efb9d94bda264b3bd04b05cdc3df917e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92786638"
---
# <a name="multi-shard-querying-using-elastic-database-tools"></a>Многосегментное формирование запросов с использованием средств работы с эластичными базами данных
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="overview"></a>Обзор

[Средства работы с эластичными базами данных](elastic-scale-introduction.md)позволяют создавать решения сегментированных баз данных. **Многосегментное формирование запросов** используется для таких задач, как сбор данных и создание отчетов, требующих запуска запроса, распространяющегося на несколько сегментов. (Сравните с [маршрутизацией, зависящей от данных](elastic-scale-data-dependent-routing.md), когда все действия выполняются в пределах одного сегмента.)

1. Получение **RangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) или **ListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) с помощью методов **TryGetRangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), **TryGetListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)) или **GetShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)). См. раздел [Создание объекта ShardMapManager](elastic-scale-shard-map-management.md#constructing-a-shardmapmanager) и [Получение RangeShardMap или ListShardMap](elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap).
2. Создание объекта **MultiShardConnection** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardconnection), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection)).
3. Создание объекта **MultiShardStatement или MultiShardCommand** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
4. Задание для **свойства CommandText** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)) команды T-SQL.
5. Выполнение команды путем вызова метода **ExecuteQueryAsync или ExecuteReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement.executeQueryAsync), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
6. Просмотр результатов с помощью класса **MultiShardResultSet или MultiShardDataReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardresultset), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader)).

## <a name="example"></a>Пример

В следующем примере кода показано применение многосегментного формирования запросов с использованием заданного сопоставления **ShardMap** с именем *myShardMap*.

```csharp
using (MultiShardConnection conn = new MultiShardConnection(myShardMap.GetShards(), myShardConnectionString))
{
    using (MultiShardCommand cmd = conn.CreateCommand())
    {
        cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable";
        cmd.CommandType = CommandType.Text;
        cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn;
        cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults;

        using (MultiShardDataReader sdr = cmd.ExecuteReader())
        {
            while (sdr.Read())
            {
                var c1Field = sdr.GetString(0);
                var c2Field = sdr.GetFieldValue<int>(1);
                var c3Field = sdr.GetFieldValue<Int64>(2);
            }
        }
    }
}
```

Основное различие состоит в построении многосегментных подключений. Когда **SqlConnection** работает с отдельной базой данных, **MultiShardConnection** принимает **_коллекцию сегментов_*_ в качестве входа. Заполните коллекцию сегментов на карте сегментов. Затем запрос выполняется в коллекции сегментов с помощью* семантики _ UNION ALL** для сборки одного общего результата. При необходимости к выходным данным можно добавить имя сегмента, из которого поступает строка, используя свойство **ExecutionOptions** команды.

Обратите внимание на вызов **myShardMap.GetShards()**. С помощью этого метода все сегменты извлекаются из карты сегментов, и обеспечивается простой способ выполнения запроса по всем соответствующим базам данных. Коллекцию сегментов для многосегментного запроса можно дополнительно ограничить, выполнив запрос LINQ к коллекции, полученной в результате вызова **myShardMap.GetShards()**. Эта возможность в многосегментном формировании запросов, в сочетании с политикой частичных результатов, разработана для применения к сегментам в количестве от десятков до сотен.

В настоящий момент существует ограничение для многосегментного формирования запросов, состоящее в отсутствии проверки для сегментов и шардлетов, к которым обращен запрос. Если при маршрутизации, зависящей от данных, выполняется проверка того, что указанный сегмент является частью карты сегментов на момент выполнения запроса, то при многосегментном формировании запросов такая проверка не происходит. Это может привести к тому, что многосегментные запросы будут обращаться к базам данных, удаленным из карты сегментов.

## <a name="multi-shard-queries-and-split-merge-operations"></a>Многосегментные запросы и операции разбиения-слияния

При многосегментных запросах не выполняется проверка на участие шардлетов из запрашиваемой базы данных в операциях разбиения или слияния. (См. раздел [масштабирование с помощью инструмента разбиения и объединения эластичной базы данных](elastic-scale-overview-split-and-merge.md).) Это может привести к несоответствию, когда строки из одного шардлета отображаются для нескольких баз данных в одном многосегментном запросе. Следует учитывать эти ограничения, фильтрование текущих операций разбиения или слияния, а также изменения сопоставления сегментов при выполнении многосегментных запросов.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]