---
title: Счетчики производительности для трассировки диспетчера сопоставления сегментов
description: Класс ShardMapManager и счетчики производительности для маршрутизации, зависящей от данных
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seoapril2019, seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/07/2019
ms.openlocfilehash: 3bfbf56b6e5f2be33b407945490531e6e2e8ac47
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92781266"
---
# <a name="create-performance-counters-to-track-performance-of-shard-map-manager"></a>Создание счетчиков производительности для наблюдения за производительностью диспетчера карт сегментов
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Счетчики производительности используются для наблюдения за производительностью операций [маршрутизации, зависящих от данных](elastic-scale-data-dependent-routing.md) . Эти счетчики можно найти в системном мониторе в категории "Эластичная база данных: управление сегментами".

Вы можете сохранять данные о производительности [диспетчера карты сегментов](elastic-scale-shard-map-management.md), особенно при использовании [маршрутизации, зависящей от данных](elastic-scale-data-dependent-routing.md). Счетчики создаются с помощью методов класса Microsoft.Azure.SqlDatabase.ElasticScale.Client.  


**Новая версия** : [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/). См. также статью [Обновление приложения для использования новой версии клиентской библиотеки эластичной базы данных](elastic-scale-upgrade-client-library.md).

## <a name="prerequisites"></a>Предварительные условия

* Чтобы пользователь мог создавать категории производительности и счетчики, он должен быть членом локальной группы **Администраторы** на компьютере, где размещается приложение.  
* Чтобы пользователь мог создавать экземпляры счетчика производительности и обновлять показания счетчиков, он должен быть членом группы **Администраторы** или **Пользователи системного монитора**.

## <a name="create-performance-category-and-counters"></a>Создание счетчиков и категорий производительности

Чтобы создать счетчик, вызовите метод CreatePerformanceCategoryAndCounters класса [ShardMapManagementFactory](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory). Этот метод может выполнять только администратор:

`ShardMapManagerFactory.CreatePerformanceCategoryAndCounters()`

Для выполнения метода можно также использовать [этот](https://gallery.technet.microsoft.com/scriptcenter/Elastic-DB-Tools-for-Azure-17e3d283) сценарий PowerShell.
Этот метод создает следующие счетчики производительности.  

* **Cached mappings**(Кэшированные сопоставления) — количество сопоставлений, кэшируемых для карты сегментов.
* **DDR operations/sec**(Число операций DDR в секунду) — скорость операций маршрутизации, зависящих от данных, для карты сегментов. Этот счетчик обновляется при успешном подключении к целевому сегменту после вызова метода [OpenConnectionForKey()](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey).
* **Mapping lookup cache hits/sec**(Число попаданий при поиске сопоставлений в кэше в секунду) — скорость успешных операций поиска сопоставлений в кэше для карты сегментов.
* **Mapping lookup cache misses/sec**(Число промахов при поиске сопоставлений в кэше в секунду) — скорость неуспешных операций поиска сопоставлений в кэше для карты сегментов.
* **Mappings added or updated in cache/sec**(Число добавляемых или обновляемых сопоставлений в кэше в секунду) — скорость добавления или обновления сопоставлений в кэше для карты сегментов.
* **Mappings removed from cache/sec**(Число удаляемых из кэша сопоставлений в секунду) — скорость удаления сопоставлений из кэша для карты сегментов.

Счетчики производительности создаются для каждой кэшированной карты сегментов каждого процесса.  

## <a name="notes"></a>Примечания

Создание счетчиков производительности инициируется следующими событиями.  

* Инициализация объекта [ShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager) с [безотложной загрузкой](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerloadpolicy), если в ShardMapManager есть карты сегментов. Сюда относятся методы [GetSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager) и [TryGetSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager).
* Успешный поиск карты сегментов (с помощью [GetShardMap()](/previous-versions/azure/dn824215(v=azure.100)), [GetListShardMap()](/previous-versions/azure/dn824212(v=azure.100)) или [GetRangeShardMap()](/previous-versions/azure/dn824173(v=azure.100))).
* Успешное создание карты сегментов с помощью CreateShardMap().

Счетчики производительности обновляются при выполнении любых операций кэширования, связанных с картой сегментов или сопоставлениями. Успешное удаление сегментов с помощью метода DeleteShardMap() приводит к удалению экземпляра счетчика производительности.  

## <a name="best-practices"></a>Рекомендации

* Создание категории производительности и счетчиков следует выполнять только один раз, до создания объекта ShardMapManager. При каждом выполнении команды CreatePerformanceCategoryAndCounters() предыдущие значения счетчиков удаляются (данные теряются во всех экземплярах) и создаются новые.  
* Экземпляры счетчиков производительности создаются для каждого процесса. Любой сбой приложения или удаление карты сегментов из кэша приведет к удалению экземпляров счетчиков производительности.  

### <a name="see-also"></a>См. также раздел

[Общие сведения о возможностях эластичных баз данных](elastic-scale-introduction.md)  

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->