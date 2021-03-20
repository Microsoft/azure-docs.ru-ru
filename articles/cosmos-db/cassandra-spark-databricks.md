---
title: Доступ к API Cassandra для Azure Cosmos DB из Azure Databricks
description: В этой статье описывается, как работать с API Cassandra для Azure Cosmos DB из Azure Databricks.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 0a83dd143ae626108fdf8d2645b8cc368a3f3e05
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100516572"
---
# <a name="access-azure-cosmos-db-cassandra-api-data-from-azure-databricks"></a>Доступ к данным API Cassandra для Azure Cosmos DB из Azure Databricks
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье подробно описано, как работать с Azure Cosmos DB API Cassandra из Spark на [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks).

## <a name="prerequisites"></a>Предварительные условия

* [Предоставление учетной записи API Cassandra для Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Просмотр основных сведений о подключении к API Cassandra для Azure Cosmos DB](cassandra-spark-generic.md)

* [Предоставление кластера Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal)

* [Просмотр примеров кода для работы с API Cassandra](cassandra-spark-generic.md#next-steps)

* [Использование при необходимости cqlsh для проверки](cassandra-spark-generic.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Конфигурация экземпляра API Cassandra для соединителя Cassandra:**

  Для инициализации соединителя для API Cassandra как части контекста Spark необходимы сведения о подключении соединителя Cassandra. При запуске записной книжки Databricks контекст Spark уже инициализирован, не рекомендуется останавливать и повторно инициализировать его. Одним из решений является добавление конфигурации экземпляра API Cassandra на уровне кластера в конфигурацию кластера Spark. Это разовое действие для каждого кластера. Добавьте в конфигурацию Spark следующий код как пару значений ключа, разделенных пробелами:
 
  ```scala
  spark.cassandra.connection.host YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com
  spark.cassandra.connection.port 10350
  spark.cassandra.connection.ssl.enabled true
  spark.cassandra.auth.username YOUR_COSMOSDB_ACCOUNT_NAME
  spark.cassandra.auth.password YOUR_COSMOSDB_KEY
  ```

## <a name="add-the-required-dependencies"></a>Добавление необходимых зависимостей

* **Соединитель Cassandra Spark:** с целью интеграции с API Cassandra для Azure Cosmos DB с помощью Spark соединитель Cassandra должен быть подключен к кластеру Azure Databricks. Чтобы подключить кластер:

  * Узнайте версию среды выполнения Databricks и версию Spark. Затем найдите [координаты Maven](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector), совместимые с соединителем Cassandra Spark, и подключите его к кластеру. См. статью ["Отправка пакета Maven или пакета Spark"](https://docs.databricks.com/user-guide/libraries.html), чтобы подключить библиотеку соединителя к кластеру. Например, координата Maven для объектов "Среда выполнения Azure Databricks Runtime версии 4.3", "Spark 2.3.1" и "Scala 2.11" равна `spark-cassandra-connector_2.11-2.3.1`

* **Библиотека API Cassandra для Azure Cosmos DB:** для настройки политики повтора из соединителя Cassandra Spark к API Cassandra для Azure Cosmos DB необходима фабрика настраиваемого подключения. Добавьте `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`[координаты Maven](https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.0.0/jar) для подключения библиотеки к кластеру.

## <a name="sample-notebooks"></a>Записные книжки с примерами

Список Azure Databricks [примеров записных книжек](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/notebooks/scala) доступен в репозитории GitHub для загрузки. Среди них примеры подключения к API Cassandra для Azure Cosmos DB из Spark и выполнения с данными различных операций CRUD. Также можно [импортировать все записные книжки](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/dbc) в свою рабочую область кластера Databricks и запустить ее. 

## <a name="accessing-azure-cosmos-db-cassandra-api-from-spark-scala-programs"></a>Доступ к API Cassandra для Azure Cosmos DB из программ Spark Scala

Программы Spark, выполняемые как автоматизированные процессы в Azure Databricks, передаются в кластер с помощью [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html)) и планируются для выполнения с помощью заданий Azure Databricks.

Следующие ссылки помогут приступить к созданию программы Spark Scala для взаимодействия с API Cassandra для Azure Cosmos DB.
* [Подключение к API Cassandra для Azure Cosmos DB из программ Spark Scala](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/main/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)
* [Запуск программы Spark Scala как автоматического задания для Azure Databricks](/azure/databricks/jobs)
* [Полный список примеров кода для работы с API Cassandra](cassandra-spark-generic.md#next-steps)

## <a name="next-steps"></a>Дальнейшие действия

Начните с [создания учетной записи API Cassandra, базы данных и таблицы](create-cassandra-api-account-java.md) с помощью приложения Java.
