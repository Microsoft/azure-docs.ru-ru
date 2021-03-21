---
title: Заметки о выпуске и ресурсы для API SQL в пружинных данных Azure Cosmos DB v2
description: Сведения о пружинных данных Azure Cosmos DB v2 для API SQL, включая даты выпуска, даты выбытия и изменения, внесенные в каждую версию пакета SDK для Java для Azure Cosmos DB SQL.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 1f78d7185415b66445c78f3b4cdb54ed0d63cb0a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102216621"
---
# <a name="spring-data-azure-cosmos-db-v2-for-core-sql-api-release-notes-and-resources"></a>Пружинные данные Azure Cosmos DB v2 для API ядра (SQL): заметки о выпуске и ресурсы
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

 Пружинные данные Azure Cosmos DB версии 2 для Core (SQL) позволяют разработчикам использовать Azure Cosmos DB в пружинных приложениях. Пружинные данные Azure Cosmos DB предоставляют интерфейс пружинных данных для управления базами данных и коллекциями, работы с документами и выдача запросов. И синхронизация, и асинхронные (реактивные) API поддерживаются в одном и том же артефакте Maven. 

[Пружинная платформа](https://spring.io/projects/spring-framework) — это модель программирования и конфигурации, которая упрощает разработку приложений Java. Пружина упрощает "работу" приложений с помощью внедрения зависимостей. Многие разработчики, например пружины, так как делают создание и тестирование приложений более простыми. [Пружинная Загрузка](https://spring.io/projects/spring-boot) расширяет эту обработку связи с продвижением к разработке веб-приложений и микрослужб. [Пружинные данные](https://spring.io/projects/spring-data) — это модель программирования для доступа к хранилищам данных, таким как Azure Cosmos DB, из контекста пружинного или пружинного приложения загрузки. 

Вы можете использовать пружинные данные Azure Cosmos DB в [облачных приложениях Azure для весны](https://azure.microsoft.com/services/spring-cloud/) .

> [!IMPORTANT]  
> Эти заметки о выпуске предназначены для предварительных Azure Cosmos DB данных версии 2. [Заметки о выпуске для версии 3](sql-api-sdk-java-spring-v3.md)можно найти здесь. 
>
> Spring Data Azure Cosmos DB поддерживает только API SQL.
>
> Сведения о пружинных данных в других Azure Cosmos DB API см. в следующих статьях:
> * [Как использовать API Apache Cassandra Spring Data с Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Как использовать API MongoDB Spring Data с Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Использование начального приложения Spring Data Gremlin с API SQL Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>
> Хотите быстро перейти к работе?
> 1. Установите [минимально поддерживаемую среду выполнения Java JDK 8](/java/azure/jdk/), чтобы можно было использовать пакет SDK.
> 2. Создание пружинного Azure Cosmos DB данных с помощью [начального](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db)приложения. Это просто!
> 3. Работа с [пружинными данными Azure Cosmos DB руководство разработчика](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb), которое проходит через базовые запросы Azure Cosmos DB.
>
> Вы можете быстро загрузить начальные стартовые приложения с помощью [пружины Initializr](https://start.spring.io/)!
>

## <a name="resources"></a>Ресурсы

| Ресурс | Ссылка |
|---|---|
| **Скачивание пакета SDK** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb) |
|**Документация по API** | [Справочная документация по пружинным данным Azure Cosmos DB]() |
|**Участие в пакете SDK** | [Репозиторий с пружинными данными Azure Cosmos DB на GitHub](https://github.com/microsoft/spring-data-cosmosdb) | 
|**Начальная загрузка весны**| [Azure Cosmos DBная клиентская библиотека начальной загрузки для Java](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Пример приложения с пружинным TODO с Azure Cosmos DB**| [Сквозной интерфейс Java в службе приложений Linux (часть 2)](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2) |
|**Руководством разработчика** | [Руководство для разработчиков Spring Data Azure Cosmos DB](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) | 
|**Использование начального** | [Как использовать пружинную загрузку начального уровня с API Azure Cosmos DB SQL](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) <br> [Репозиторий GitHub для начальной загрузки Azure Cosmos DB пружины](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Пример с помощью службы приложений Azure** | [Использование Spring и Cosmos DB со Службой приложений в Linux](/azure/developer/java/spring-framework/configure-spring-app-with-cosmos-db-on-app-service-linux) <br> [Пример приложения TODO](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git) |

## <a name="release-history"></a>История выпусков

### <a name="230-may-21-2020"></a>2.3.0 (21 мая, 2020)
#### <a name="new-features"></a>новые функции;
* Обновляет версию пружинной загрузки до 2.3.0.


### <a name="225-may-19-2020"></a>2.2.5 (19 мая, 2020)
#### <a name="new-features"></a>новые функции;
* Обновляет версию Azure Cosmos DB до 3.7.3.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Содержит исправления утечки памяти и NETTY обновления версий из пакета SDK 3.7.3 для Azure Cosmos DB.

### <a name="224-april-6-2020"></a>2.2.4 (6 апреля, 2020)
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправление `allowTelemetry` флага, с которым следует учитывать учетные записи `CosmosDbConfig` .
* Исправление `TTL` свойства в контейнере.

### <a name="223-february-25-2020"></a>2.2.3 (25 февраля 2020 г.)
#### <a name="new-features"></a>новые функции;
* Добавляет новый `findAll` с помощью API ключа секции.
* Обновляет версию Azure Cosmos DB до 3.7.0.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправления `collectionName`  ->  `containerName` .
* Исправления `entityClass` и `domainClass`  ->  `domainType` .
* Исправление "коллекция возвращаемых сущностей, сохраненная репозиторием вместо входной сущности".

### <a name="2110-february-25-2020"></a>2.1.10 (25 февраля 2020 г.)
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Обратные порты исправляются для "коллекция возвращаемых сущностей, сохраненных репозиторием вместо входной сущности".

### <a name="222-january-15-2020"></a>2.2.2 (15 января, 2020)
#### <a name="new-features"></a>новые функции;
* Обновляет версию Azure Cosmos DB до 3.6.0.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок

### <a name="221-december-31-2019"></a>2.2.1 (31 декабря 2019 г.)
#### <a name="new-features"></a>новые функции;
* Обновляет Azure Cosmos DB версии пакета SDK до 3.5.0.
* Добавляет поле заметки для включения или отключения автоматического создания коллекции.
* Улучшает обработку исключений. Предоставляет `CosmosClientException` через `CosmosDBAccessException` .
* Предоставляет доступ `requestCharge` и `activityId` через `ResponseDiagnostics` .
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Устранение ошибки обновления 3.5.0 в пакете SDK, Cosmos DB если размер заголовка ответа HTTP превышает 8192 байт, "" Консистенциполици. Дефаултконсистенцилевел () завершается сбоем при ограниченном устаревания и соответствующем префиксе ".
* Исправляет `findById` поведение метода. Ранее этот метод возвращает значение Empty, если сущность не найдена вместо создания исключения.
* Исправляет ошибку, при которой сортировка не была применена на следующей странице при `CosmosPageRequest` использовании.

### <a name="219-december-26-2019"></a>2.1.9 (26 декабря 2019 г.)
#### <a name="new-features"></a>новые функции;
* Добавляет поле заметки для включения или отключения автоматического создания коллекции.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
*  Исправляет `findById` поведение метода. Ранее этот метод возвращает значение Empty, если сущность не найдена вместо создания исключения.

### <a name="220-october-21-2019"></a>2.2.0 (21 октября, 2019)
#### <a name="new-features"></a>новые функции;
* Завершите поддержку реактивного репозитория Cosmos.
* Azure Cosmos DB поддержки метрик запросов и строк диагностики.
* Azure Cosmos DB обновление версии пакета SDK для 3.3.1.
* Обновление версии каркасной платформы до 5.2.0. RELEASE.
* Обновление версии Commons Attribution данных до 2.2.0. RELEASE.
* Добавляет `findByIdAndPartitionKey` `deleteByIdAndPartitionKey` API-интерфейсы и.
* Удаляет зависимость из Azure-documentdb.
* Перефирменная символика DocumentDB на Azure Cosmos DB.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправления "при сортировке возникает исключение, когда pageSize меньше общего количества элементов в репозитории".

### <a name="218-october-18-2019"></a>2.1.8 (18 октября, 2019)
#### <a name="new-features"></a>новые функции;
* Не рекомендуется использовать API DocumentDB.
* Добавляет `findByIdAndPartitionKey` `deleteByIdAndPartitionKey` API-интерфейсы и.
* Добавляет оптимистичную блокировку на основе `_etag` .
* Включает выражение спел для имени коллекции документов.
* Добавляет `ObjectMapper` улучшения.

### <a name="217-october-18-2019"></a>2.1.7 (18 октября, 2019)
#### <a name="new-features"></a>новые функции;
* Добавляет зависимость Azure Cosmos DB SDK версии 3.
* Добавляет реактивный репозиторий Cosmos.
* Обновляет реализацию `DocumentDbTemplate` для использования пакета SDK для Azure Cosmos DB версии 3.
* Добавляет другие изменения конфигурации для поддержки реактивного репозитория Cosmos.

### <a name="212-march-19-2019"></a>2.1.2 (19 марта 2019 г.)
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Удаляет `applicationInsights` зависимость для:
    * Потенциальный риск замусоривать зависимостей.
    * Несовместимость с Java 11.
    * Предотвращение потенциального влияния на производительность ЦП и/или памяти.

### <a name="207-march-20-2019"></a>2.0.7 (20 марта 2019 г.)
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Независимый порт удаляет `applicationInsights` зависимость для:
    * Потенциальный риск замусоривать зависимостей.
    * Несовместимость с Java 11.
    * Предотвращение потенциального влияния на производительность ЦП и/или памяти.

### <a name="211-march-7-2019"></a>от – 7 марта 2019 г.
#### <a name="new-features"></a>новые функции;
* Обновляет основную версию до версии.

### <a name="206-march-7-2019"></a>2.0.6 (7 марта 2019 г.)
#### <a name="new-features"></a>новые функции;
* Игнорировать все исключения из телеметрии.

### <a name="210-december-17-2018"></a>2.1.0 (17 декабря, 2018)
#### <a name="new-features"></a>новые функции;
* Обновляет версию до 2.1.0, чтобы устранить проблему.

### <a name="205-september-13-2018"></a>2.0.5 (13 сентября 2018 г.)
#### <a name="new-features"></a>новые функции;
* Добавляет ключевые слова `exists` и `startsWith` .
* Файл сведений об обновлениях.
#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправления "не удается вызвать Self href непосредственно для сущности".
* Исправления "findAll завершится ошибкой, если коллекция не будет создана".

### <a name="204-prerelease-august-23-2018"></a>2.0.4 (Предварительная версия) (23 августа 2018)
#### <a name="new-features"></a>новые функции;
* Переименовывает пакет из documentdb в cosmosdb.
* Добавляет новую функцию ключевого слова метода запроса. Теперь поддерживаются 16 ключевых слов из API SQL.
* Добавляет новую функцию запроса с разбиением на страницы и сортировкой.
* Упрощает настройку пружинных данных — cosmosdb.
* Добавляет `deleteCollection` `deleteAll` API-интерфейсы и.

#### <a name="key-bug-fixes"></a>Основные исправления ошибок
* Исправление ошибок и устранение дефектов.

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Дальнейшие действия
См. дополнительные сведения об [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Узнайте больше о [пружинной платформе](https://spring.io/projects/spring-framework).

Дополнительные сведения о [пружинной загрузке](https://spring.io/projects/spring-boot).

Дополнительные сведения о [пружинных данных](https://spring.io/projects/spring-data).