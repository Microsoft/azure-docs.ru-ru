---
title: Краткое руководство. Использование Spring Data Azure Cosmos DB версии 3 для создания базы данных документов с помощью Azure Cosmos DB
description: В этом кратком руководстве представлен пример кода Spring Data Azure Cosmos DB версии 3, который можно использовать для подключения к API SQL Azure Cosmos DB и выполнения запросов к ней.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: quickstart
ms.date: 10/06/2020
ms.author: anfeldma
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: f31eb0fa6dbb881f7a09b21b9dd4842fdfd291f5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93090296"
---
# <a name="quickstart-build-a-spring-data-azure-cosmos-db-v3-app-to-manage-azure-cosmos-db-sql-api-data"></a>Краткое руководство. Создание приложения Spring Data Azure Cosmos DB версии 3 для управления данными API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET версии 3](create-sql-api-dotnet.md)
> * [.NET версии 4](create-sql-api-dotnet-V4.md)
> * [Пакет SDK для Java версии 4](create-sql-api-java.md)
> * [Spring Data версии 3](create-sql-api-spring-data.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

В этом кратком руководстве вы научитесь создавать учетную запись API SQL Azure Cosmos DB и управлять ею на портале Azure, а также с помощью приложения Spring Data Azure Cosmos DB версии 3, клонированного с GitHub. Сначала создайте учетную запись API SQL Azure Cosmos DB с помощью портала Azure, а затем создайте приложение Spring Boot с помощью соединителя Spring Data Azure Cosmos DB версии 3, а затем добавьте ресурсы в учетную запись Cosmos DB с помощью приложения Spring Boot. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создать и запрашивать документы, таблицы, пары "ключ-значение", а также графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

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

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) с URI `https://localhost:8081` и ключом `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`.
- [Комплект SDK для Java (JDK) версии 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Укажите для переменной среды `JAVA_HOME` папку, в которой установлен комплект JDK.
- [Двоичный архив Maven](https://maven.apache.org/download.cgi). В Ubuntu выполните команду `apt-get install maven`, чтобы установить Maven.
- [Git](https://www.git-scm.com/downloads). В Ubuntu выполните команду `sudo apt-get install git`, чтобы установить Git.

## <a name="introductory-notes"></a>Вступительные примечания

*Структура учетной записи Cosmos DB.* Независимо от API или языка программирования, *учетная запись* Cosmos DB содержит несколько *баз данных* или не содержит их вовсе. *База данных* содержит несколько *контейнеров* или не содержит их вовсе, а *контейнер* содержит несколько элементов или не содержит их вовсе, как показано на схеме ниже:

:::image type="content" source="./media/account-databases-containers-items/cosmos-entities.png" alt-text="Сущности учетной записи Azure Cosmos" border="false":::

Дополнительные сведения о базах данных, контейнерах и элементах см. [здесь](account-databases-containers-items.md). На уровне контейнера определены несколько важных свойств, среди которых *подготовленная пропускная способность* и *ключ секции*. 

Подготовленная пропускная способность измеряется в единицах запроса (*ЕЗ*), которые имеют денежную цену и являются существенным определяющим фактором в операционных расходах учетной записи. Подготовленную пропускную способность можно выбрать с учетом степени детализации для каждого контейнера или для каждой базы данных, но обычно предпочтительной является спецификация пропускной способности на уровне контейнера. Дополнительные сведения о подготовке пропускной способности см. [здесь](set-throughput.md).

Когда элементы вставляются в контейнер Cosmos DB, база данных увеличивается горизонтально, добавляя больше ресурсов хранения и вычисления для обработки запросов. Емкость хранилища и вычислительных ресурсов добавляется в дискретные единицы, известные как *секции*. Вам необходимо выбрать одно поле в документах в качестве ключа секции, который сопоставляет каждый документ с секцией. Способ управления секциями заключается в том, что каждой секции присваивается примерно равный срез из диапазона значений ключа секции. Таким образом рекомендуется выбирать ключ секции, который является относительно случайным или равномерно распределенным. В противном случае для одних секций будет значительно больше запросов (*секции с высокой нагрузкой*), в то время как для других секций будет значительно меньше запросов (*секции с низкой нагрузкой*), и этого следует избегать. Дополнительные сведения о секционировании см. [здесь](partitioning-overview.md).

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

Перед созданием базы данных документов необходимо создать в Azure Cosmos DB учетную запись API SQL.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-container"></a>Добавление контейнера

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

<a id="add-sample-data"></a>
## <a name="add-sample-data"></a>Добавление демонстрационных данных

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Обращение к данным

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь перейдем к работе с кодом. Давайте клонируем приложение API SQL из GitHub. Задайте строку подключения и выполните ее. Вы узнаете, как можно упростить работу с данными программным способом. 

Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

```bash
git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started.git
```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Вы можете сразу перейти к [запуску приложения](#run-the-app). 

### <a name="application-configuration-file"></a>Файл конфигурации приложения

В этой статье показано, как Spring Boot и Spring Data улучшают взаимодействие с пользователем: в процессе установки клиента Cosmos и подключения к ресурсам Cosmos теперь быстрее использовать конфигурацию, а не писать код. Во время стартовой загрузки приложения Spring Boot обрабатывает все эти стандартные параметры используя настройки из **application.properties**:

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

После создания учетной записи, базы данных и контейнера Azure Cosmos DB просто заполните пустое значение в файле конфигурации, и Spring Boot/Spring Data автоматически будут выполнять следующие действия: (1) создавать базовый экземпляр Java SDK `CosmosClient` с URI и ключом и (2) подключаться к базе данных и контейнеру. Все готово — **и не нужно писать никакого кода для управления ресурсами!**

### <a name="java-source"></a>Источник Java

Ценность Spring Data также заключается в простом, удобном, стандартизированном и независимом от платформы интерфейсе для работы с хранилищами. На основе образца Spring Data из GitHub, приведенного выше, ниже приведены примеры CRUD и запросов для управления документами Azure Cosmos DB с помощью Spring Data Azure Cosmos DB.

* Создание и обновление элементов с помощью метода `save`.

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Create)]
   
* Чтение с помощью метода производного запроса, определенного в репозитории. `findByIdAndLastName` выполняет операцию чтения для `UserRepository`. Поля, упомянутые в имени метода, вынуждают Spring Data выполнить чтение, определяемое полями `id` и `lastName`:

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Read)]

* Удаление элементов с помощью `deleteAll`:

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Delete)]

* Производный запрос на основе имени метода из репозитория. Spring Data реализует метод `UserRepository` `findByFirstName` как SQL-запрос пакета SDK для Java в поле `firstName` (этот запрос не может быть реализован как чтение):

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Query)]

## <a name="run-the-app"></a>Запустите приложение

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения, и запустите приложение, используя данные конечной точки. Так вы обеспечите обмен данными между приложением и размещенной базой данных.

1. В окне терминала Git выполните команду `cd`, чтобы перейти к папке с примером кода.

    ```bash
    cd azure-spring-data-cosmos-java-sql-api-getting-started/azure-spring-data-cosmos-java-getting-started/
    ```

2. В окне терминала Git используйте следующую команду, чтобы установить необходимые пакеты Spring Data Azure Cosmos DB.

    ```bash
    mvn clean package
    ```

3. В окне терминала Git используйте следующую команду, чтобы запустить приложение Spring Data Azure Cosmos DB:

    ```bash
    mvn spring-boot:run
    ```
    
4. Приложение загружает **application.properties** и подключает ресурсы в учетной записи Azure Cosmos DB.
5. Приложение выполнит операции CRUD с точкой, описанные выше.
6. Приложение выполнит производный запрос.
7. Приложение не удаляет ресурсы. Вернитесь на портал, чтобы [очистить ресурсы](#clean-up-resources) в своей учетной записи, если хотите избежать расходов.

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве было показано, как создать учетную запись API SQL Azure Cosmos DB, базу данных документов и контейнер с помощью обозревателя данных и как запустить приложение Spring Data, чтобы эти операции выполнялись программно. Теперь вы можете импортировать дополнительные данные в учетную запись Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Импорт данных в DocumentDB с помощью средства миграции базы данных](import-data.md)