---
title: Заметки о выпуске и ресурсы для SQL API Azure Cosmos DB v3
description: Сведения о пружинных данных Azure Cosmos DB v3 для API SQL, включая даты выпуска, даты выбытия и изменения, внесенные в каждую версию пакета SDK для Java для Azure Cosmos DB SQL.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 03/15/2021
ms.author: kuthapar
ms.custom: devx-track-java
ms.openlocfilehash: 536c0ab78a4b7e08abb1532635aff722df9e15cd
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105563112"
---
# <a name="spring-data-azure-cosmos-db-v3-for-core-sql-api-release-notes-and-resources"></a>Пружинные данные Azure Cosmos DB v3 для API Core (SQL): заметки о выпуске и ресурсы
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

Пружинные данные Azure Cosmos DB версии 3 для Core (SQL) позволяют разработчикам использовать Azure Cosmos DB в пружинных приложениях. Пружинные данные Azure Cosmos DB предоставляют интерфейс пружинных данных для управления базами данных и коллекциями, работы с документами и выдача запросов. И синхронизация, и асинхронные (реактивные) API поддерживаются в одном и том же артефакте Maven. 

> [!IMPORTANT]
> Пружинные данные Azure Cosmos DB зависит от платформы пружины данных.
> 
> версии Azure-пружины-Data-Cosmos от 3.0.0 до 3.4.0 поддерживают пружинные данные версий 2,2 и 2,3.
> 
> Azure-пружины-Data-Cosmos версии 3.5.0 и выше поддерживают пружинные версии данных 2.4.3 и выше.
>

[Пружинная платформа](https://spring.io/projects/spring-framework) — это модель программирования и конфигурации, которая упрощает разработку приложений Java. Пружина упрощает "работу" приложений с помощью внедрения зависимостей. Многие разработчики, например пружины, так как делают создание и тестирование приложений более простыми. [Пружинная Загрузка](https://spring.io/projects/spring-boot) расширяет эту обработку связи с продвижением к разработке веб-приложений и микрослужб. [Пружинные данные](https://spring.io/projects/spring-data) — это модель программирования и платформа для доступа к хранилищам данных, таким как Azure Cosmos DB, из контекста пружинного или пружинного приложения загрузки. 

Вы можете использовать пружинные данные Azure Cosmos DB в [облачных приложениях Azure для весны](https://azure.microsoft.com/services/spring-cloud/) .

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

## <a name="get-started-fast"></a>Быстрое начало работы

  Приготовьтесь к работе с пружинными данными Azure Cosmos DB, следуя нашим [руководству по началу работы с пружинным загрузкой](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db). Для начала работы с пружинным Azure Cosmos DB соединителем рекомендуется использовать «пружинный» начальный вариант.

  Кроме того, можно добавить пружинные данные Azure Cosmos DB зависимость в `pom.xml` файл, как показано ниже:

  ```xml
  <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-spring-data-cosmos</artifactId>
      <version>latest-version</version>
  </dependency>
  ```

## <a name="helpful-content"></a>Полезное содержимое

| Содержимое | Ссылка |
|---|---|
|**Скачивание пакета SDK**| [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-cosmos) |
|**Документация по API** | [Справочная документация по API Java](/java/api/com.azure.spring.data.cosmos) |
|**Участие в разработке пакета SDK** | [Центральный репозиторий пакета SDK Azure для Java в GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-cosmos) | 
|**Начало работы** | [Краткое руководство. Создание пружинного приложения Azure Cosmos DB данных для управления Azure Cosmos DB данных API SQL](./create-sql-api-spring-data.md) <br> [Репозиторий GitHub с кодом краткого руководства](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started) | 
|**Простые примеры кода** | [Azure Cosmos DB: данные Azure Cosmos DB примеры для API SQL](sql-api-spring-data-sdk-samples.md) <br> [Репозиторий GitHub с примером кода](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples)|
| **Советы по улучшению производительности**| [Советы по повышению производительности для пакета SDK для Java V4 (применимо к пружинным данным)](performance-tips-java-sdk-v4-sql.md)| 
| **Устранение неполадок** | [Устранение неполадок пакета SDK для Java V4 (применимо к пружинным данным)](troubleshoot-java-sdk-v4-sql.md) | 
| **Семинары и лабораторные занятия по Azure Cosmos DB** |[Домашняя страница семинаров по Cosmos DB](https://aka.ms/cosmosworkshop)

[!INCLUDE[Release notes](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-spring-data-cosmos/CHANGELOG.md)]

## <a name="additional-notes"></a>Дополнительные замечания

* Пружинные данные Azure Cosmos DB поддерживают Java JDK 8 и Java JDK 11.
* Пружинные данные 2,3 в настоящее время поддерживаются; пружинные данные 2,4 в настоящее время не поддерживаются.

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения об [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Узнайте больше о [пружинной платформе](https://spring.io/projects/spring-framework).

Дополнительные сведения о [пружинной загрузке](https://spring.io/projects/spring-boot).

Дополнительные сведения о [пружинных данных](https://spring.io/projects/spring-data).