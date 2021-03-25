---
title: Краткое руководство. Создание базы данных документов с помощью Java и Azure Cosmos DB
description: В этом кратком руководстве представлен пример кода Java, который можно использовать для подключения и выполнения запросов к API SQL в Azure Cosmos DB
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: anfeldma
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 4b62b591c408f663fd28d5077af924f785ee66c8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93090415"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-sql-api-data"></a>Краткое руководство. Создание приложения Java для управления данными API SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET версии 3](create-sql-api-dotnet.md)
> * [.NET версии 4](create-sql-api-dotnet-V4.md)
> * [Пакет SDK для Java версии 4](create-sql-api-java.md)
> * [Spring Data версии 3](create-sql-api-spring-data.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

В этом кратком руководстве вы научитесь создавать учетную запись API SQL Azure Cosmos DB и управлять ею на портале Azure, а также с помощью приложения Java, клонированного с GitHub. Сначала создайте учетную запись API SQL для Azure Cosmos DB с помощью портала Azure, затем создайте приложение Java с использованием пакета SDK Java для SQL и добавьте ресурсы в учетную запись Cosmos DB с помощью приложения Java. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

> [!IMPORTANT]  
> Это краткое руководство применимо только к пакету SDK Java для Azure Cosmos DB версии 4. Дополнительные сведения см. в [заметках о выпуске](sql-api-sdk-java-v4.md) для пакета средств разработки Java для Azure Cosmos DB версии 4, [репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), [рекомендациях по повышению производительности](performance-tips-java-sdk-v4-sql.md) и [руководстве по устранению неполадок](troubleshoot-java-sdk-v4-sql.md) для пакета средств разработки Java для Azure Cosmos DB версии 4. Если сейчас вы используете более раннюю версию, чем версия 4, руководство [Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4](migrate-java-v4-sdk.md) поможет вам обновить его до версии 4.
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
git clone https://github.com/Azure-Samples/azure-cosmos-java-getting-started.git
```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Вы можете сразу перейти к [запуску приложения](#run-the-app). 


# <a name="sync-api"></a>[Синхронный API](#tab/sync)

### <a name="managing-database-resources-using-the-synchronous-sync-api"></a>Управление ресурсами базы данных с помощью синхронного API (sync)

* Инициализация `CosmosClient`. `CosmosClient` является логическим представлением службы баз данных Azure Cosmos на стороне клиента. Этот клиент позволяет настраивать и выполнять запросы к службе.
    
    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateSyncClient)]

* Создание `CosmosDatabase`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateDatabaseIfNotExists)]

* Создание `CosmosContainer`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateContainerIfNotExists)]

* Создание элемента с помощью метода `createItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateItem)]
   
* Считывание точек выполняется с помощью метода `readItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=ReadItem)]

* SQL-запросы через JSON выполняются с помощью метода `queryItems`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=QueryItems)]

# <a name="async-api"></a>[Асинхронный API](#tab/async)

### <a name="managing-database-resources-using-the-asynchronous-async-api"></a>Управление ресурсами базы данных с помощью асинхронного API (async)

* Вызовы асинхронного API возвращаются немедленно, не дожидаясь ответа от сервера. В свете этого в следующих фрагментах кода показаны правильные шаблоны проектирования для выполнения всех предыдущих задач управления с использованием асинхронного API.

* Инициализация `CosmosAsyncClient`. `CosmosAsyncClient` является логическим представлением службы баз данных Azure Cosmos на стороне клиента. Этот клиент позволяет настраивать и выполнять асинхронные запросы к службе.
    
    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=CreateAsyncClient)]

* Создание `CosmosAsyncDatabase`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateDatabaseIfNotExists)]

* Создание `CosmosAsyncContainer`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateContainerIfNotExists)]

* Как и в случае с синхронным API, элементы создаются с помощью метода `createItem`. В этом примере показано, как эффективно выдавать многочисленные асинхронные запросы `createItem`, подписавшись на реактивный поток, который выдает запросы и печатает уведомления. Поскольку этот простой пример выполняется до завершения и прерывания, экземпляры `CountDownLatch` используются, чтобы гарантировать, что программа не будет прервана во время создания элемента. **Правильная методика асинхронного программирования заключается не в блокировке асинхронных вызовов, а в реалистичных запросах на использование, которые генерируются из цикла main(), который выполняется бессрочно, устраняя необходимость кратковременной блокировки асинхронных вызовов.**

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=CreateItem)]
   
* Как и в случае с синхронным API, операции чтения точки выполняются с помощью метода `readItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=ReadItem)]

* Как и в случае с синхронным API, запросы SQL через JSON выполняются с помощью метода `queryItems`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=QueryItems)]

---

## <a name="run-the-app"></a>Запустите приложение

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения, и запустите приложение, используя данные конечной точки. Так вы обеспечите обмен данными между приложением и размещенной базой данных.

1. В окне терминала Git выполните команду `cd`, чтобы перейти к папке с примером кода.

    ```bash
    cd azure-cosmos-java-getting-started
    ```

2. В окне терминала Git введите приведенную ниже команду, чтобы установить необходимые пакеты Java.

    ```bash
    mvn package
    ```

3. В окне терминала git используйте следующую команду для запуска приложения Java (замените SYNCASYNCMODE на `sync` или `async` в зависимости от того, какой пример кода вы хотите запустить, замените YOUR_COSMOS_DB_HOSTNAME на заключенное в кавычки значение URI на портале, и замените YOUR_COSMOS_DB_MASTER_KEY на первичный ключ в кавычках на портале).

    ```bash
    mvn exec:java@SYNCASYNCMODE -DACCOUNT_HOST=YOUR_COSMOS_DB_HOSTNAME -DACCOUNT_KEY=YOUR_COSMOS_DB_MASTER_KEY

    ```

    В окне терминала отобразится уведомление о создании базы данных FamilyDB. 
    
4. Приложение создает базу данных с именем `AzureSampleFamilyDB`.
5. Приложение создает контейнер с именем `FamilyContainer`.
6. Приложение выполнит считывание точек по идентификаторам объектов и значению ключа раздела (в нашем примере — lastName). 
7. Приложение будет запрашивать элементы для получения всех семейств с фамилией из списка (Andersen, Wakefield, Johnson).

7. Это приложение не позволяет удалять созданные ресурсы. Вернитесь на портал и [удалите там ресурсы](#clean-up-resources)  из учетной записи, чтобы за них не взималась плата.

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве было показано, как создать учетную запись API SQL Azure Cosmos, базу данных документов и контейнер с помощью обозревателя данных и как запустить приложение Java, чтобы эти операции выполнялись программно. Теперь вы можете импортировать дополнительные данные в учетную запись Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Импорт данных в DocumentDB с помощью средства миграции базы данных](import-data.md)
