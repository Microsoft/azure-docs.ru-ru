---
title: Заметки о выпуске и ресурсы для SQL API Cosmos DB Apache Spark
description: Сведения о соединителе Apache Spark Azure Cosmos DB для API SQL, включая даты выпуска, даты прекращения использования и изменения, внесенные в каждую версию пакета SDK для SQL Async Java Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 7ff75bf9d45803dd96927bcf7c70e7c7912db979
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93097164"
---
# <a name="azure-cosmos-db-apache-spark-connector-for-core-sql-api-release-notes-and-resources"></a>Azure Cosmos DB Apache Spark соединителя для API Core (SQL): заметки о выпуске и ресурсы
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Пакет SDK версии 3 для .NET](sql-api-sdk-dotnet-standard.md)
> * [Пакет SDK для .NET версии 2](sql-api-sdk-dotnet.md)
> * [Пакет SDK для .NET Core версии 2](sql-api-sdk-dotnet-core.md)
> * [Пакет SDK для .NET Change Feed версии 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [Пакет SDK для Java версии 4](sql-api-sdk-java-v4.md)
> * [Пакет SDK для Async Java версии 2](sql-api-sdk-async-java.md)
> * [Пакет SDK для Sync Java версии 2](sql-api-sdk-java.md)
> * [Spring Data версии 2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data версии 3](sql-api-sdk-java-spring-v3.md)
> * [Соединитель Spark](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Поставщик ресурсов REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Исполнитель массовых операций — .NET версии 2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Исполнитель массовых операций — Java](sql-api-sdk-bulk-executor-java.md)

Можно ускорить анализ больших данных с помощью соединителя Azure Cosmos DB Apache Spark для Core (SQL). Соединитель Spark позволяет выполнять задания [Spark](https://spark.apache.org/) для данных, хранящихся в Azure Cosmos DB. Поддерживаются Пакетная и Потоковая обработка.

Вы можете использовать соединитель с [Azure Databricks](https://azure.microsoft.com/services/databricks) или [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), который предоставляет управляемые кластеры Spark в Azure. В следующей таблице приведены поддерживаемые версии.

| Компонент | Версия |
|---------|-------|
| Apache Spark | 2,4.*x*, 2,3. *x*, 2,2. *x* и 2,1. *x* |
| Scala | 2,11 |
| Azure Databricks (версия среды выполнения) | Позже 3,4 |

> [!WARNING]
> Этот соединитель поддерживает API ядра (SQL) Azure Cosmos DB.
> Для Cosmos DB API для MongoDB используйте [соединитель MongoDB для Spark](https://docs.mongodb.com/spark-connector/master/).
> Для API Cassandra Cosmos DB используйте [соединитель Spark Cassandra](https://github.com/datastax/spark-cassandra-connector).
>

## <a name="resources"></a>Ресурсы

| Ресурс | Ссылка |
|---|---|
| **Скачивание пакета SDK** | [Скачайте последнюю версию jar-файла](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG), [Maven](https://search.maven.org/search?q=a:azure-cosmosdb-spark_2.4.0_2.11) |
|**Документация по API** | [Справочник по соединителю Spark]() |
|**Участие в пакете SDK** | [Соединитель Azure Cosmos DB для Apache Spark на GitHub](https://github.com/Azure/azure-cosmosdb-spark) | 
|**Начало работы** | [Ускорение аналитики больших данных с помощью соединителя Apache Spark для Azure Cosmos DB](./spark-connector.md#bk_working_with_connector) <br> [Использование структурированной потоковой передачи Apache Spark с Apache Kafka в Azure Cosmos DB](../hdinsight/apache-kafka-spark-structured-streaming-cosmosdb.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json) | 

## <a name="release-history"></a>История выпусков

### <a name="330"></a>3.3.0
#### <a name="new-features"></a>новые функции;
- Добавляет новый параметр конфигурации, `changefeedstartfromdatetime` который можно использовать для указания времени начала обработки пр. Дополнительные сведения см. в разделе [Параметры конфигурации](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references).

### <a name="320"></a>3.2.0
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
- Устраняется регрессия, которая привела к чрезмерному потреблению памяти исполнителям для больших результирующих наборов (например, с миллионами строк), в итоге приводящей к ошибке `java.lang.OutOfMemoryError: GC overhead limit exceeded` .

### <a name="311"></a>3.1.1
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправляет промежуточный вариант контрольной точки потоковой передачи, в котором параметр `ID` содержит символ вертикальной черты (|) с `ChangeFeedMaxPagesPerBatch` примененной конфигурацией.

### <a name="310"></a>3.1.0
#### <a name="new-features"></a>новые функции;
* Добавляет поддержку для выполнения массовых обновлений при использовании вложенных ключей секционирования.
* Добавляет поддержку типов данных Decimal и float во время операций записи в Azure Cosmos DB.
* Добавляет поддержку типов меток времени, если в качестве значения используется Long (эпоха UNIX).

### <a name="308"></a>3.0.8
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправляет исключение приведения, возникающее при `WriteThroughputBudget` использовании конфигурации.

### <a name="307"></a>3.0.7
#### <a name="new-features"></a>новые функции;
* Добавляет сведения об ошибке для массовых сбоев в исключения и журнал.

### <a name="306"></a>3.0.6
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Устраняет проблемы с контрольными точками потоковой передачи.

### <a name="305"></a>3.0.5
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Чтобы уменьшить уровень шума, исправляет непреднамеренное сообщение об ОШИБКе уровня журнала.

### <a name="304"></a>3.0.4
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Устраняет ошибку в структурированной потоковой передаче во время разбиения секций. Ошибка может привести к некоторым отсутствующим записям веб-канала изменений или исключениям NULL для операций записи контрольной точки.

### <a name="303"></a>3.0.3
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Устраняет ошибку, которая приводит к игнорированию пользовательской схемы, предоставленной для Реадстреам.

### <a name="302"></a>3.0.2
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправлена регрессия (незатененный JAR-файл, включающий все затененные зависимости), увеличивающий время сборки на 50 процентов.

### <a name="301"></a>3.0.1
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Устраняет проблему зависимости, которая приводит к сбою прямого транспорта по протоколу TCP с Рекуесттимеаутексцептион.

### <a name="300"></a>3.0.0
#### <a name="new-features"></a>новые функции;
* Улучшает управление подключениями и объединение соединений, чтобы сократить количество вызовов метаданных.

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения об [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Узнайте больше об [Apache Spark](https://spark.apache.org/).
