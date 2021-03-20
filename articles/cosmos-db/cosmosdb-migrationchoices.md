---
title: Параметры миграции Cosmos DB
description: В этом документе описываются различные варианты переноса локальных или облачных данных в Azure Cosmos DB
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: 8721c0eb728f568521e86baecb658dc9c869a7f6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93097589"
---
# <a name="options-to-migrate-your-on-premises-or-cloud-data-to-azure-cosmos-db"></a>Параметры для переноса локальных или облачных данных в Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Можно загружать данные из различных источников данных в Azure Cosmos DB. Поскольку Azure Cosmos DB поддерживает несколько интерфейсов API, целевыми объектами могут быть любые существующие API. Ниже приведены некоторые сценарии, в которых данные переносятся в Azure Cosmos DB:

* Перемещение данных из одного контейнера Azure Cosmos в другой контейнер в той же или другой базе данных.
* Перемещение данных между выделенными контейнерами в контейнеры общих баз данных.
* Перемещение данных из учетной записи Azure Cosmos, расположенной в Region1, в другую учетную запись Azure Cosmos в том же или другом регионе.
* Перемещение данных из источника, такого как хранилище BLOB-объектов Azure, файл JSON, база данных Oracle, Couchbase, DynamoDB в Azure Cosmos DB.

Для поддержки путей перехода из различных источников в различные Azure Cosmos DB API существует несколько решений, которые обеспечивают специализированную обработку для каждого пути переноса. В этом документе перечислены доступные решения и описаны их преимущества и ограничения.

## <a name="factors-affecting-the-choice-of-migration-tool"></a>Факторы, влияющие на выбор средства миграции

Выбор средства миграции зависит от следующих факторов.

* **Оперативная или автономная миграция**. Многие средства миграции предоставляют только возможность однократного переноса. Это означает, что в работе приложений, обращающихся к базе данных, могут быть простои. Некоторые решения для миграции позволяют выполнять динамическую миграцию, при которой между источником и целевым объектом настраивается конвейер репликации.

* **Источник данных**. существующие данные могут находиться в различных источниках данных, таких как Oracle DB2, Datastax Кассанда, база данных SQL Azure, PostgreSQL и т. д. Данные могут также находиться в существующей учетной записи Azure Cosmos DB и цель миграции — изменить модель данных или выполнить секционирование данных в контейнере с помощью другого ключа секции.

* **API Azure Cosmos DB**. Команда Azure Cosmos DB разработала множество средств для API SQL в Azure Cosmos DB, которые удобно использовать для различных сценариев миграции. Для всех остальных API-интерфейсов предусмотрены их собственные специализированные наборы инструментов, которые разработанные и обслуживаются сообществом. Поскольку Azure Cosmos DB поддерживает эти API на уровне протокола коммуникации, при переносе данных в Azure Cosmos DB эти средства должны работать "как есть". Однако им может потребоваться настраиваемая обработка для регулирования, так как для Azure Cosmos DB характерна именно эта концепция.

* **Размер данных**. Большинство средств миграции очень хорошо работают для небольших наборов данных. Если же набор данных превышает несколько сотен гигабайт, выбор средств миграции уже ограничен. 

* **Ожидаемая длительность миграции**. Миграцию можно настроить так, чтобы она выполнялась медленно с постепенным наращиванием скорости. При этом будет потребляться меньше пропускной способности. Однако можно использовать и всю пропускную способность, подготовленную для целевого контейнера Azure Cosmos DB. Тогда для миграции потребуется меньше времени.

## <a name="azure-cosmos-db-sql-api"></a>Azure Cosmos DB SQL API

|Тип перемещения|Решение|Поддерживаемые источники|Поддерживаемые целевые объекты|Рекомендации|
|---------|---------|---------|---------|---------|
|Вне сети|[Инструмент переноса данных](import-data.md)| &bull;Файлы JSON/CSV<br/>&bull;Azure Cosmos DB API SQL<br/>&bull;MongoDB<br/>&bull;SQL Server<br/>&bull;Хранилище таблиц<br/>&bull;AWS DynamoDB<br/>&bull;хранилище BLOB-объектов Azure|&bull;Azure Cosmos DB API SQL<br/>&bull;API таблиц Azure Cosmos DB<br/>&bull;Файлы JSON |&bull; Простота настройки и поддержки нескольких источников. <br/>&bull; Не подходит для больших наборов данных.|
|Вне сети|[Фабрика данных Azure](../data-factory/connector-azure-cosmos-db.md).| &bull;Файлы JSON/CSV<br/>&bull;Azure Cosmos DB API SQL<br/>&bull;API Azure Cosmos DB для MongoDB<br/>&bull;MongoDB <br/>&bull;SQL Server<br/>&bull;Хранилище таблиц<br/>&bull;хранилище BLOB-объектов Azure <br/> <br/>Другие поддерживаемые источники см. в статье [фабрика данных Azure](../data-factory/connector-overview.md) .|&bull;Azure Cosmos DB API SQL<br/>&bull;API Azure Cosmos DB для MongoDB<br/>&bull;Файлы JSON <br/><br/> Другие поддерживаемые целевые объекты см. в статье [фабрика данных Azure](../data-factory/connector-overview.md) . |&bull; Простота настройки и поддержки нескольких источников.<br/>&bull; Использует библиотеку Azure Cosmos DBного выполнителя. <br/>&bull; Подходит для больших наборов данных. <br/>&bull; Отсутствие контрольных точек — это означает, что если проблема возникает во время миграции, необходимо перезапустить весь процесс миграции.<br/>&bull; Отсутствие очереди недоставленных сообщений — это означает, что несколько ошибочных файлов могут прерывать весь процесс миграции.|
|Автономная миграция|[Соединитель Azure Cosmos DB Spark](spark-connector.md)|Azure Cosmos DB API SQL. <br/><br/>Вы можете использовать другие источники с дополнительными соединителями из экосистемы Spark.| Azure Cosmos DB API SQL. <br/><br/>Вы можете использовать другие целевые объекты с дополнительными соединителями из экосистемы Spark.| &bull; Использует библиотеку Azure Cosmos DBного выполнителя. <br/>&bull; Подходит для больших наборов данных. <br/>&bull; Требуется пользовательская настройка Spark. <br/>&bull; Spark учитывает несоответствия схем, и это может быть проблемой во время миграции. |
|Автономная миграция|[Пользовательский инструмент с библиотекой Cosmos DBного выполнителя](migrate-cosmosdb-data.md)| Источник зависит от пользовательского кода. | Azure Cosmos DB SQL API| &bull; Предоставляет возможности создания контрольных точек, недоставленных сообщений, что повышает устойчивость к миграции. <br/>&bull; Подходит для очень больших наборов данных (10 ТБ +).  <br/>&bull; Требуется пользовательская установка этого инструмента, запускаемого в качестве службы приложений. |
|Миграция по сети|[Функции Cosmos DB и API пр](change-feed-functions.md)| Azure Cosmos DB SQL API | Azure Cosmos DB SQL API| &bull; Простота настройки. <br/>&bull; Работает только в том случае, если источником является контейнер Azure Cosmos DB. <br/>&bull; Не подходит для больших наборов данных. <br/>&bull; Не фиксирует операции удаления из исходного контейнера. |
|Миграция по сети|[Настраиваемая служба миграции с использованием пр](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator)| Azure Cosmos DB SQL API | Azure Cosmos DB SQL API| &bull; Обеспечивает отслеживание хода выполнения. <br/>&bull; Работает только в том случае, если источником является контейнер Azure Cosmos DB. <br/>&bull; Работает также с большими наборами данных.<br/>&bull; Требует, чтобы пользователь настроил службу приложений для размещения обработчика веб-канала изменений. <br/>&bull; Не фиксирует операции удаления из исходного контейнера.|
|Миграция по сети|[стриим](cosmosdb-sql-api-migrate-data-striim.md)| &bull;Oracle; <br/>&bull;Apache Cassandra<br/><br/> Другие поддерживаемые источники см. на [веб-сайте Стриим](https://www.striim.com/sources-and-targets/) . |&bull;Azure Cosmos DB API SQL <br/>&bull; Azure Cosmos DB API Cassandra<br/><br/> Другие поддерживаемые целевые объекты см. на [веб-сайте Стриим](https://www.striim.com/sources-and-targets/) . | &bull; Работает с большим разнообразием источников, таких как Oracle, DB2 SQL Server.<br/>&bull; Легко создавайте конвейеры ETL и предоставляет панель мониторинга для мониторинга. <br/>&bull; Поддерживает большие наборы данных. <br/>&bull; Поскольку это средство стороннего производителя, оно должно быть приобретено в Marketplace и установлено в среде пользователя.|

## <a name="azure-cosmos-db-mongo-api"></a>Azure Cosmos DB API Mongo

|Тип перемещения|Решение|Поддерживаемые источники|Поддерживаемые целевые объекты|Рекомендации|
|---------|---------|---------|---------|---------|
|Справка в Интернете|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)| MongoDB|API Azure Cosmos DB для MongoDB |&bull; Использует библиотеку Azure Cosmos DBного выполнителя. <br/>&bull; Подходит для больших наборов данных и отвечает за репликацию динамических изменений. <br/>&bull; Работает только с другими источниками MongoDB.|
|Автономная миграция|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)| MongoDB| API Azure Cosmos DB для MongoDB| &bull; Использует библиотеку Azure Cosmos DBного выполнителя. <br/>&bull; Подходит для больших наборов данных и отвечает за репликацию динамических изменений. <br/>&bull; Работает только с другими источниками MongoDB.|
|Вне сети|[Фабрика данных Azure](../data-factory/connector-azure-cosmos-db.md).| &bull;Файлы JSON/CSV<br/>&bull;Azure Cosmos DB API SQL<br/>&bull;API Azure Cosmos DB для MongoDB <br/>&bull;MongoDB<br/>&bull;SQL Server<br/>&bull;Хранилище таблиц<br/>&bull;хранилище BLOB-объектов Azure <br/><br/> Другие поддерживаемые источники см. в статье [фабрика данных Azure](../data-factory/connector-overview.md) . | &bull;Azure Cosmos DB API SQL<br/>&bull;API Azure Cosmos DB для MongoDB <br/>&bull; Файлы JSON <br/><br/> Другие поддерживаемые целевые объекты см. в статье [фабрика данных Azure](../data-factory/connector-overview.md) .| &bull; Простота настройки и поддержки нескольких источников. <br/>&bull; Использует библиотеку Azure Cosmos DBного выполнителя. <br/>&bull; Подходит для больших наборов данных. <br/>&bull; Отсутствие контрольных точек означает, что любые проблемы во время миграции потребуют перезапуска всего процесса миграции.<br/>&bull; Отсутствие очереди недоставленных сообщений означает, что несколько ошибочных файлов могут прерывать весь процесс миграции. <br/>&bull; Требуется пользовательский код для увеличения пропускной способности чтения для определенных источников данных.|
|Вне сети|[Существующие инструменты Mongo (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|MongoDB | API Azure Cosmos DB для MongoDB| &bull; Простота настройки и интеграции. <br/>&bull; Требуется настраиваемая обработка для регулирования.|

## <a name="azure-cosmos-db-cassandra-api"></a>API Cassandra для Azure Cosmos DB

|Тип перемещения|Решение|Поддерживаемые источники|Поддерживаемые целевые объекты|Рекомендации|
|---------|---------|---------|---------|---------|
|Автономная миграция|[cqlsh COPY, команда](cassandra-import-data.md#migrate-data-using-cqlsh-copy-command)|CSV-файлы | API Cassandra для Azure Cosmos DB| &bull; Простота настройки. <br/>&bull; Не подходит для больших наборов данных. <br/>&bull; Работает, только если источником является таблица Cassandra.|
|Автономная миграция|[Копирование таблицы с помощью Spark](cassandra-import-data.md#migrate-data-using-spark) | &bull;Apache Cassandra<br/>&bull;API Cassandra для Azure Cosmos DB| API Cassandra для Azure Cosmos DB | &bull; Может использовать возможности Spark для параллельного преобразования и приема. <br/>&bull; Для управления регулированием требуется конфигурация с настраиваемой политикой повтора.|
|Миграция по сети|[Стриим (из Oracle DB или Apache Cassandra)](cosmosdb-cassandra-api-migrate-data-striim.md)| &bull;Oracle;<br/>&bull;Apache Cassandra<br/><br/> Другие поддерживаемые источники см. на [веб-сайте Стриим](https://www.striim.com/sources-and-targets/) .|&bull;Azure Cosmos DB API SQL<br/>&bull;API Cassandra для Azure Cosmos DB <br/><br/> Другие поддерживаемые целевые объекты см. на [веб-сайте Стриим](https://www.striim.com/sources-and-targets/) .| &bull; Работает с большим разнообразием источников, таких как Oracle, DB2 SQL Server. <br/>&bull; Легко создавайте конвейеры ETL и предоставляет панель мониторинга для мониторинга. <br/>&bull; Поддерживает большие наборы данных. <br/>&bull; Поскольку это средство стороннего производителя, оно должно быть приобретено в Marketplace и установлено в среде пользователя.|
|Миграция по сети|[Блитзз (из Oracle DB или Apache Cassandra)](oracle-migrate-cosmos-db-blitzz.md)|&bull;Oracle;<br/>&bull;Apache Cassandra<br/><br/>Другие поддерживаемые источники см. на [веб-сайте блитзз](https://www.blitzz.io/) . |API Cassandra Azure Cosmos DB. <br/><br/>Другие поддерживаемые целевые объекты см. на [веб-сайте блитзз](https://www.blitzz.io/) . | &bull; Поддерживает большие наборы данных. <br/>&bull; Поскольку это средство стороннего производителя, оно должно быть приобретено в Marketplace и установлено в среде пользователя.|

## <a name="other-apis"></a>Other APIs

Для API-интерфейсов, отличных от API SQL, Mongo API и API Cassandra, существуют различные средства, поддерживаемые всеми экосистемами API. 

**API таблицы**; 

* [Средство миграции данных](table-import.md#data-migration-tool)
* [AzCopy](table-import.md#migrate-data-by-using-azcopy)

**API Gremlin**;

* [Библиотека массовых исполнителей Graph](bulk-executor-graph-dotnet.md)
* [Gremlin Spark](https://github.com/Azure/azure-cosmosdb-spark/blob/2.4/samples/graphframes/main.scala) 

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в примерах приложений, использующих библиотеку небольшого выполнителя в [.NET](bulk-executor-dot-net.md) и [Java](bulk-executor-java.md). 
* Библиотека небольшого Исполнительного исполнителя интегрирована в соединитель Cosmos DB Spark. Дополнительные сведения см. в статье о [соединителе Azure Cosmos DB Spark](spark-connector.md) .  
* Обратитесь к группе разработчиков Azure Cosmos DB, открыв запрос в службу поддержки в подтипе проблем "Общие рекомендации" и "крупные (ТБ +) миграции" для получения дополнительной справки о крупномасштабных миграциях.
