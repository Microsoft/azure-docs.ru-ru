---
title: Использование API таблиц и Java для создания приложения — Azure Cosmos DB
description: В этом руководстве показано, как использовать API таблицы Azure Cosmos DB для создания приложения с помощью портала Azure и Java
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: java
ms.topic: quickstart
ms.date: 05/28/2020
ms.author: sngun
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 48335e577ed248a42914bdaa1b1e662daf2d26cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93090175"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-table-api-data"></a>Краткое руководство. Создание приложения Java для управления данными API таблиц Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-table-dotnet.md)
> * [Java](create-table-java.md)
> * [Node.js](create-table-nodejs.md)
> * [Python](./table-storage-how-to-use-python.md)
> 

В этом кратком руководстве объясняется, как создать учетную запись API таблиц Azure Cosmos DB, а также как с помощью обозревателя данных и приложения Java, клонированного из GitHub, создать таблицы и сущности. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) с URI `https://localhost:8081` и ключом `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`.
- [Комплект SDK для Java (JDK) версии 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Укажите для переменной среды `JAVA_HOME` папку, в которой установлен комплект JDK.
- [Двоичный архив Maven](https://maven.apache.org/download.cgi). 
- [Git](https://www.git-scm.com/downloads). 

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

> [!IMPORTANT] 
> Для работы с общедоступными пакетами SDK для API таблиц нужно создать новую учетную запись API таблиц. Учетные записи API таблиц, созданные во время работы с предварительной версией, не поддерживаются в общедоступных пакетах SDK.
>

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="add-a-table"></a>Добавление таблицы

[!INCLUDE [cosmos-db-create-table](../../includes/cosmos-db-create-table.md)]

## <a name="add-sample-data"></a>Добавление демонстрационных данных

[!INCLUDE [cosmos-db-create-table-add-sample-data](../../includes/cosmos-db-create-table-add-sample-data.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь необходимо клонировать приложение "Таблица" из GitHub. Задайте строку подключения и выполните ее. Вы узнаете, как можно упростить работу с данными программным способом.

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    md "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/storage-table-java-getting-started.git 
    ```

> [!TIP]
> Более подробное пошаговое руководство по аналогичному коду см. в статье с [примером API таблиц для Cosmos DB](table-storage-how-to-use-java.md). 

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Если вас это не интересует, можете сразу переходить к разделу [Обновление строки подключения](#update-your-connection-string) в этом документе.

* В следующем коде показано, как создать таблицу в службе хранилища Azure.

  ```java
  private static CloudTable createTable(CloudTableClient tableClient, String tableName) throws StorageException, RuntimeException, IOException, InvalidKeyException,   IllegalArgumentException, URISyntaxException, IllegalStateException {
  
    // Create a new table
    CloudTable table = tableClient.getTableReference(tableName);
    try {
        if (table.createIfNotExists() == false) {
            throw new IllegalStateException(String.format("Table with name \"%s\" already exists.", tableName));
        }
    }
    catch (StorageException s) {
        if (s.getCause() instanceof java.net.ConnectException) {
            System.out.println("Caught connection exception from the client. If running with the default configuration please make sure you have started the storage emulator.");
        }
        throw s;
    }

    return table;
  }
  ```

* В следующем коде показано, как вставить данные в таблицу.

  ```javascript
  private static void batchInsertOfCustomerEntities(CloudTable table) throws StorageException {
  
  // Create the batch operation
  TableBatchOperation batchOperation1 = new TableBatchOperation();
  for (int i = 1; i <= 50; i++) {
      CustomerEntity entity = new CustomerEntity("Smith", String.format("%04d", i));
      entity.setEmail(String.format("smith%04d@contoso.com", i));
      entity.setHomePhoneNumber(String.format("425-555-%04d", i));
      entity.setWorkPhoneNumber(String.format("425-556-%04d", i));
      batchOperation1.insertOrMerge(entity);
  }
  
  // Execute the batch operation
  table.execute(batchOperation1);
  }
  ```

* В следующем коде показано, как выполнить запрос данных из таблицы.

  ```java
  private static void partitionScan(CloudTable table, String partitionKey) throws StorageException {
  
      // Create the partition scan query
      TableQuery<CustomerEntity> partitionScanQuery = TableQuery.from(CustomerEntity.class).where(
          (TableQuery.generateFilterCondition("PartitionKey", QueryComparisons.EQUAL, partitionKey)));
  
      // Iterate through the results
      for (CustomerEntity entity : table.execute(partitionScanQuery)) {
          System.out.println(String.format("\tCustomer: %s,%s\t%s\t%s\t%s", entity.getPartitionKey(), entity.getRowKey(), entity.getEmail(), entity.getHomePhoneNumber(), entity.  getWorkPhoneNumber()));
      }
  }
  ```

* В следующем коде показано, как удалить данные из таблицы.

  ```java
  
  System.out.print("\nDelete any tables that were created.");
  
  if (table1 != null && table1.deleteIfExists() == true) {
      System.out.println(String.format("\tSuccessfully deleted the table: %s", table1.getName()));
  }
  
  if (table2 != null && table2.deleteIfExists() == true) {
      System.out.println(String.format("\tSuccessfully deleted the table: %s", table2.getName()));
  }
  ```

## <a name="update-your-connection-string"></a>Обновление строки подключения

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения. Скопируйте эти данные в приложение. Так вы обеспечите обмен данными между приложением и размещенной базой данных. 

1. Выберите элемент **Строка подключения** в своей учетной записи Azure Cosmos DB на [портале Azure](https://portal.azure.com/). 

   :::image type="content" source="./media/create-table-java/cosmos-db-quickstart-connection-string.png" alt-text="Просмотр сведений о строке подключения на панели Строка подключения":::

2. Скопируйте основную строку подключения с помощью кнопки копирования справа.

3. Откройте файл *config.properties* из папки *C:\git-samples\storage-table-java-getting-started\src\main\resources*. 

5. Закомментируйте первую строку и раскомментируйте вторую. После этого первые две строки будут выглядеть так:

    ```xml
    #StorageConnectionString = UseDevelopmentStorage=true
    StorageConnectionString = DefaultEndpointsProtocol=https;AccountName=[ACCOUNTNAME];AccountKey=[ACCOUNTKEY]
    ```

6. Вставьте значение основной строки подключения, полученное на портале, вместо значения параметра StorageConnectionString в строке 2. 

    > [!IMPORTANT]
    > Если для вашей конечной точки указан адрес documents.azure.com, это значит, что вы используете учетную запись для предварительной версии. В этом случае для работы с общедоступными пакетами SDK для API таблиц вам нужно создать [новую учетную запись API таблиц](#create-a-database-account).
    >

7. Сохраните файл *config.properties*.

Теперь приложение со всеми сведениями, необходимыми для взаимодействия с Azure Cosmos DB, обновлено. 

## <a name="run-the-app"></a>Запустите приложение

1. В окне терминала git перейдите к папке storage-table-java-getting-started с помощью команды `cd`.

    ```git
    cd "C:\git-samples\storage-table-java-getting-started"
    ```

2. В окне терминала Git выполните приведенные ниже команды, чтобы запустить приложение Java.

    ```git
    mvn compile exec:java 
    ```

    В окне консоли отобразятся данные таблицы, добавленные к новой базе данных таблиц в Azure Cosmos DB.

    Вернитесь в обозреватель данных, где вы можете просматривать, запрашивать и изменять новые данные, а также работать с ними. 

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись Azure Cosmos DB и таблицу с помощью обозревателя данных, а также как запустить приложение Java для добавления данных таблицы.  Теперь вы можете выполнить запрос данных с помощью API таблиц.  

> [!div class="nextstepaction"]
> [Импорт данных таблиц в API таблицы](table-import.md)