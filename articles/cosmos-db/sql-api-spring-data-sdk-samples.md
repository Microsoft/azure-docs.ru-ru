---
title: API SQL для Azure Cosmos DB. Примеры Spring Data версии 3
description: Здесь вы найдете примеры Spring Data версии 3 на GitHub для типичных задач с использованием API SQL для Azure Cosmos DB, включая операции CRUD.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 09/23/2020
ms.custom: devx-track-java
ms.author: anfeldma
ms.openlocfilehash: 378bb891c8539a6cf3d61f6511a0f58377d2bfd2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93091146"
---
# <a name="azure-cosmos-db-sql-api-spring-data-azure-cosmos-db-v3-examples"></a>API SQL для Azure Cosmos DB. Примеры Spring Data Azure Cosmos DB версии 3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Примеры пакета SDK для .NET версии 2](sql-api-dotnet-samples.md)
> * [Примеры пакета SDK для .NET версии 3](sql-api-dotnet-v3sdk-samples.md)
> * [Примеры для пакета SDK Java версии 4](sql-api-java-sdk-samples.md)
> * [Примеры для пакета SDK для Spring Data версии 3](sql-api-spring-data-sdk-samples.md)
> * [Примеры Node.js](sql-api-nodejs-samples.md)
> * [Примеры Python](sql-api-python-samples.md)
> * [Коллекция примеров кода Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

> [!IMPORTANT]  
> Эти заметки о выпуске применимы к Spring Data Azure Cosmos DB версии 3. Заметки о выпуске для версии 2 см. [здесь](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB поддерживает только API SQL.
>
> Сведения о Spring Data в других API-интерфейсах Azure Cosmos DB см. в этих статьях:
> * [Как использовать API Apache Cassandra Spring Data с Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Как использовать API MongoDB Spring Data с Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Использование начального приложения Spring Data Gremlin с API SQL Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>

> [!IMPORTANT]  
>[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
>  
>- Вы можете [активировать преимущества подписчика Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): в вашей подписке Visual Studio каждый месяц зачисляются деньги на счет, которые можно использовать для оплаты служб Azure.
>
>[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]
>

Новейшие примеры приложений, которые выполняют операции CRUD и другие распространенные операции с ресурсами Azure Cosmos DB, содержатся в репозитории GitHub [azure-spring-data-cosmos-java-sql-api-samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples). Эта статья содержит:

* Ссылки на задачи в каждом из примеров файлов проектов Spring Data Azure Cosmos DB. 
* Ссылки на соответствующие справочные материалы по API.

**Предварительные требования**

Чтобы запустить пример приложения, необходимы следующие компоненты:

* комплект разработчика Java 8;
* Spring Data Azure Cosmos DB версии 3.

При необходимости можно использовать Maven, чтобы получить последние двоичные файлы Spring Data Azure Cosmos DB версии 3 для использования в этом проекте. Maven автоматически добавляет все необходимые зависимости. В противном случае можно непосредственно скачать зависимости, которые перечислены в файле **pom.xml**, и добавить их в путь сборки.

```bash
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-cosmos</artifactId>
    <version>LATEST</version>
</dependency>
```

**Запуск примеров приложений**

Клонирование примера репозитория:
```bash
$ git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples

$ cd azure-spring-data-cosmos-java-sql-api-samples
```

Примеры можно запустить, используя интегрированною среду разработки (Eclipse, IntelliJ или VSCODE), или с помощью командной строки в Maven.

Необходимо задать эти переменные среды в **application.properties**.

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

Это предоставит для примеров доступ на чтение и запись для вашей учетной записи, базы данных и контейнеров.

Интегрированная среда разработки может предоставить возможность выполнения примера кода Spring Data. В противном случае для выполнения примера можно использовать следующую команду терминала:

```bash
mvn spring-boot:run
```

## <a name="document-crud-examples"></a>Примеры документации CRUD
В файле [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) показано, как выполнять следующие задачи. Чтобы узнать больше о документах Azure Cosmos перед выполнением приведенных ниже примеров, ознакомьтесь с тематической статьей [Работа с базами данных, контейнерами и элементами](account-databases-containers-items.md).

| Задача | Справочник по API |
| --- | --- |
| [Создание документа](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L46-L47) | CosmosRepository.save |
| [Чтение документа по идентификатору](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L56-L58) | CosmosRepository.derivedQueryMethod |
| [Удаление всех документов](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L39-L41) | CosmosRepository.deleteAll |

## <a name="derived-query-method-examples"></a>Примеры производных методов запроса
В файле [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) показано, как выполнять следующие задачи. Чтобы узнать больше о запросах Azure Cosmos DB перед выполнением следующих примеров, ознакомьтесь со статьей [Производные методы запросов Baeldung в Spring](https://www.baeldung.com/spring-data-derived-queries).

| [Запрос документов](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L73-L77) | CosmosRepository.derivedQueryMethod |

## <a name="custom-query-examples"></a>Примеры пользовательских запросов
В файле [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) показано, как выполнять следующие задачи с помощью грамматики SQL-запроса. Чтобы узнать больше об SQL-запросах в Azure Cosmos DB перед выполнением приведенных ниже примеров, ознакомьтесь со статьей о [примерах SQL-запросов для Azure Cosmos DB](./sql-query-getting-started.md). 


| Задача | Справочник по API |
| --- | --- |
| [Запрос всех документов](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L20-L22) | Заметка к @Query |
| [Выполнение запроса на соответствие условию с использованием оператора равенства ==](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L24-L26) | Заметка к @Query |
| [Выполнение запроса на несоответствие условию с использованием операторов != и NOT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L28-L38) | Заметка к @Query |
| [Выполнение запроса с использованием операторов диапазона, таких как >, <, >=, <=](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L40-L42) | Заметка к @Query |
| [Выполнение запроса к строкам с использованием операторов диапазона](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L44-L46) | Заметка к @Query |
| [Выполнение запроса с использованием предложения ORDER BY](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L48-L50) | Заметка к @Query |
| [Создание запросов с помощью DISTINCT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L52-L54) | Заметка к @Query |
| [Выполнение запроса с использованием агрегатных функций](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L56-L62) | Заметка к @Query |
| [Работа с вложенными документами](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L64-L66) | Заметка к @Query |
| [Выполнение запроса с использованием внутридокументных соединений](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L68-L85) | Заметка к @Query |
| [Выполнение запроса с использованием операторов string, math и array](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L87-L97) | Заметка к @Query |