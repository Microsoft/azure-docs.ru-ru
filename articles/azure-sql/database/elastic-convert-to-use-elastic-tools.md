---
title: Перенос существующих баз данных для развертывания
description: Преобразование сегментированных баз данных для использования инструментов эластичных баз данных путем создания диспетчера карт сегментов
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: c6ad8b4c80f4b9c2fdb3c1a14209dcf0febc89e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92787148"
---
# <a name="migrate-existing-databases-to-scale-out"></a>Перенос существующих баз данных для развертывания
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Простое управление существующими масштабируемыми сегментированными базами данных с помощью средств (таких как [Клиентская библиотека эластичной базы данных](elastic-database-client-library.md)). Для использования [диспетчера сопоставления сегментов](elastic-scale-shard-map-management.md) следует сначала преобразовать существующий набор баз данных.

## <a name="overview"></a>Обзор

Для переноса существующей сегментированной базы данных выполните следующие действия.

1. Подготовка [базы данных диспетчера карты сегментов](elastic-scale-shard-map-management.md).
2. Создание карты сегментов.
3. Подготовка отдельных сегментов.  
4. Добавление сопоставлений на карту сегментов.

Эти методы можно реализовать с помощью [клиентской библиотеки платформа .NET Framework](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)или сценариев PowerShell, найденных в [базе данных SQL Azure — скрипты инструментов эластичной базы данных](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db). В приведенных здесь примерах используются скрипты PowerShell.

Чтобы больше узнать о ShardMapManager, ознакомьтесь с [управлением картами сегментов](elastic-scale-shard-map-management.md). Общие сведения о средствах эластичных баз данных см. в разделе [Общие сведения о функциях эластичных баз данных](elastic-scale-introduction.md).

## <a name="prepare-the-shard-map-manager-database"></a>Подготовка базы данных диспетчера карты сегментов

Диспетчер карты сегментов — это специальная база данных, хранящая данные для управления масштабируемыми базами данных. Можно использовать существующую базу данных или создать новую. Для диспетчера сопоставления сегментов и для сегмента следует использовать разные базы данных. Сценарий PowerShell не создает базу данных автоматически.

## <a name="step-1-create-a-shard-map-manager"></a>Шаг 1. Создание диспетчера сопоставления сегментов

```powershell
# Create a shard map manager
New-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
#<server_name> and <smm_db_name> are the server name and database name
# for the new or existing database that should be used for storing
# tenant-database mapping information.
```

### <a name="to-retrieve-the-shard-map-manager"></a>Извлечение диспетчера сопоставления сегментов

Созданный диспетчер сопоставления сегментов можно извлечь с помощью этого командлета. Этот шаг требуется выполнять при каждом использовании объекта ShardMapManager.

```powershell
# Try to get a reference to the Shard Map Manager  
$ShardMapManager = Get-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
```

## <a name="step-2-create-the-shard-map"></a>Шаг 2. Создание сопоставления сегментов

Выберите тип создаваемого сопоставления сегментов. Этот выбор зависит от архитектуры базы данных.

1. Один клиент на базу данных (определения терминов см. в [глоссарии](elastic-scale-glossary.md)).
2. Несколько клиентов на одну базу данных (два типа):
   1. Сопоставление по спискам
   2. Сопоставление по диапазонам

Для модели с одним клиентом создайте карту сегментов с **сопоставлением по списку** . В модели с одним клиентом каждому клиенту назначается по одной базе данных. Эта модель подходит для разработчиков SaaS, так как она упрощает управление.

![Сопоставление по спискам][1]

Мультитенантная модель назначает несколько клиентов отдельной базе данных (а группы клиентов вы можете распределять по нескольким базам данных). Эту модель можно использовать, когда у каждого отдельного клиента низкие потребности в обработке данных. В этой модели клиенты соотносятся с базой данных с использованием **сопоставления по диапазонам**.

![Сопоставление по диапазонам][2]

Вы можете также реализовать модель с несколькими клиентами с помощью *сопоставления по списку*, чтобы назначить несколько клиентов отдельной базе данных. Например, база данных DB1 используется для хранения информации о клиенте 1 и 5, а DB2 хранит данные о клиенте 7 и 10.

![Несколько клиентов для отдельной базы данных][3]

**Выберите один из вариантов, соответствующий вашему выбору.**

### <a name="option-1-create-a-shard-map-for-a-list-mapping"></a>Вариант 1. Создание карты сегментов для сопоставления списка

Создайте карту сегментов, используя объект ShardMapManager.

```powershell
# $ShardMapManager is the shard map manager object
$ShardMap = New-ListShardMap -KeyType $([int]) -ListShardMapName 'ListShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-2-create-a-shard-map-for-a-range-mapping"></a>Вариант 2. Создание карты сегментов для сопоставления диапазонов

Для использования этого шаблона сопоставления потребуются непрерывные диапазоны значений идентификаторов клиентов. Кроме того, можно не включать все диапазоны и пропустить один из них при создании базы данных.

```powershell
# $ShardMapManager is the shard map manager object
# 'RangeShardMap' is the unique identifier for the range shard map.  
$ShardMap = New-RangeShardMap -KeyType $([int]) -RangeShardMapName 'RangeShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-3-list-mappings-on-an-individual-database"></a>Вариант 3. список сопоставлений в отдельной базе данных

Для настройки этого шаблона также требуется создать карту списков, как показано в разделе "Шаг 2, вариант 1".

## <a name="step-3-prepare-individual-shards"></a>Шаг 3. Подготовка отдельных сегментов

Добавьте каждый сегмент (база данных) в диспетчер сопоставления сегментов. Таким образом отдельные базы данных будут подготовлены для хранения сведений о сопоставлении. Подготовьте так каждый сегмент.

```powershell
Add-Shard -ShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
# The $ShardMap is the shard map created in step 2.
```

## <a name="step-4-add-mappings"></a>Шаг 4. Добавление сопоставлений

Добавление сопоставлений зависит от вида созданной карты сегментов. Если создана карта списков, нужно добавить сопоставления по спискам. Если создана карта диапазонов, нужно добавить сопоставления по диапазонам.

### <a name="option-1-map-the-data-for-a-list-mapping"></a>Вариант 1. Сопоставление данных для сопоставления списка

Сопоставьте данные, добавив сопоставление по спискам для каждого клиента.  

```powershell
# Create the mappings and associate it with the new shards
Add-ListMapping -KeyType $([int]) -ListPoint '<tenant_id>' -ListShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="option-2-map-the-data-for-a-range-mapping"></a>Вариант 2. Сопоставление данных для сопоставления диапазонов

Добавьте сопоставления по диапазонам для всех связанных диапазонов идентификаторов клиентов и баз данных.

```powershell
# Create the mappings and associate it with the new shards
Add-RangeMapping -KeyType $([int]) -RangeHigh '5' -RangeLow '1' -RangeShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="step-4-option-3-map-the-data-for-multiple-tenants-on-an-individual-database"></a>Шаг 4. вариант 3. Сопоставьте данные для нескольких клиентов в отдельной базе данных

Для каждого клиента следует выполнить функцию Add-ListMapping (вариант 1).

## <a name="checking-the-mappings"></a>Проверка сопоставлений

Сведения о существующих сегментах и сопоставлениях, связанных с ними, можно запросить с помощью следующих команд:  

```powershell
# List the shards and mappings
Get-Shards -ShardMap $ShardMap
Get-Mappings -ShardMap $ShardMap
```

## <a name="summary"></a>Итоги

После завершения настройки можно начать работу с клиентской библиотекой эластичной базы данных. Кроме того, можно воспользоваться [маршрутизацией, зависящей от данных](elastic-scale-data-dependent-routing.md), и [формированием многосегментных запросов](elastic-scale-multishard-querying.md).

## <a name="next-steps"></a>Дальнейшие действия

Получите скрипты PowerShell из [сценариев инструментов базы данных SQL Azure Database-Elastic](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

Средства также доступны на сайте GitHub: [Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools).

Используйте средство разбиения и слияния для перемещения данных из модели с несколькими клиентами в модель с одним клиентом и наоборот. Ознакомьтесь со статьей о [средстве разбиения и объединения](elastic-scale-get-started.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

Сведения о распространенных шаблонах архитектуры данных для мультитенантных приложений базы данных SaaS см. в статье [Шаблоны разработки для мультитенантных приложений SaaS с использованием Базы данных Azure SQL](saas-tenancy-app-design-patterns.md).

## <a name="questions-and-feature-requests"></a>Вопросы и запросы на функции

Для получения вопросов используйте [страницу Microsoft Q&A вопрос для базы данных SQL](/answers/topics/azure-sql-database.html) , а для запросов функций добавьте их на [Форум обратной связи с базой данных SQL](https://feedback.azure.com/forums/217321-sql-database/).

<!--Image references-->
[1]: ./media/elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/elastic-convert-to-use-elastic-tools/multipleonsingledb.png