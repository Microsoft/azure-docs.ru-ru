---
title: Использование хранилища таблиц Azure и API таблиц Azure Cosmos DB в Java
description: Хранение структурированных данных в облаке с помощью Хранилища таблиц Azure или API таблиц Azure Cosmos DB в Java.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: Java
ms.topic: sample
ms.date: 12/10/2020
author: ThomasWeiss
ms.author: thweiss
ms.custom: devx-track-java
ms.openlocfilehash: a5da5e1717f897d2236fd73f0fff525e157f7a0e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97093695"
---
# <a name="how-to-use-azure-table-storage-or-azure-cosmos-db-table-api-from-java"></a>Как использовать в Java хранилище таблиц Azure и API таблиц Azure Cosmos DB

[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

В этой статье показано, как создавать таблицы, сохранять данные и выполнять операции CRUD с данными. Вы можете использовать службу таблиц Azure или API таблиц Azure Cosmos DB. Примеры написаны на Java и используют [пакет SDK службы хранилища Azure версии 8 для Java][Azure Storage SDK for Java]. Рассматриваются сценарии **создания**, **перечисления** и **удаления** таблиц, а также **вставки**, **запроса**, **изменения** и **удаления** сущностей в таблице. Дополнительные сведения о таблицах см. в разделе [Дальнейшие действия](#next-steps).

> [!IMPORTANT]
> Последняя версия пакета SDK службы хранилища Azure с поддержкой Хранилища таблиц — [8][Azure Storage SDK for Java]. Новая версия пакета SDK хранилища таблиц для Java выйдет в ближайшее время.

> [!NOTE]
> Пакет SDK доступен разработчикам, использующим хранилище Azure на устройствах Android. Дополнительные сведения см. в разделе [Microsoft Azure Storage SDK for Android][Azure Storage SDK for Android] (Пакет SDK хранилища Azure для Android).
>

## <a name="create-an-azure-service-account"></a>Создание учетной записи службы Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Create an Azure Storage account** (Создание учетной записи хранения Azure)

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Создание учетной записи Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-java-application"></a>Создание приложения Java

В этом руководстве будут использоваться компоненты хранилища, которые можно вызвать в приложении Java локально или в веб-роли или рабочей роли в Azure.

Чтобы применить примеры из этой статьи, установите пакет SDK для Java (JDK) и создайте в подписке Azure учетную запись хранения Azure или учетную запись Azure Cosmos DB. После этого убедитесь, что ваша система разработки отвечает минимальным требованиям и зависимостям, указанным в репозитории [пакета SDK службы хранилища Azure для Java][Azure Storage SDK for Java] на сайте GitHub. Если ваша система отвечает всем требованиям, вы можете перейти к выполнению инструкций по скачиванию и установке библиотек хранилища Azure для Java из указанного репозитория. После завершения предварительных задач, приступайте к созданию приложения Java на основе примеров из этой статьи.

## <a name="configure-your-application-to-access-table-storage"></a>Настройка приложения для доступа к хранилищу таблиц

Если вы намерены обращаться к таблицам через API-интерфейсы службы хранилища Azure или таблиц Azure Cosmos DB, добавьте следующие инструкции импорта в начало файла с кодом Java.

```java
// Include the following imports to use table APIs
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.table.*;
import com.microsoft.azure.storage.table.TableQuery.*;
```

## <a name="add-your-connection-string"></a>Добавление строки подключения

Вы можете подключиться к учетной записи хранения Azure или учетной записи API таблиц Azure Cosmos DB. Получите строку подключения на основе типа используемой учетной записи.

### <a name="add-an-azure-storage-connection-string"></a>Добавление строки подключения к службе хранилища Azure

Клиент хранилища Azure использует строку подключения с целью хранения конечных точек и учетных данных для доступа к службам управления данными. При работе в клиентском приложении необходимо указать для хранилища строку подключения в следующем формате, используя имя своей учетной записи хранения и первичный ключ доступа для учетной записи хранения, указанные на [портале Azure](https://portal.azure.com) значениями **AccountName** и **AccountKey**. 

В этом примере показано, как объявить статическое поле для размещения строки подключения:

```java
// Define the connection-string with your values.
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key";
```

### <a name="add-an-azure-cosmos-db-table-api-connection-string"></a>Добавление строки подключения к API таблиц Azure Cosmos DB

Учетная запись Azure Cosmos DB использует строку подключения для хранения учетных данных и конечной точки таблицы. Запуская клиентское приложение, предоставьте ему строку подключения к Azure Cosmos DB в следующем формате, указав имя учетной записи Azure Cosmos DB и первичный ключ доступа для этой учетной записи. Эти значения вы можете найти на [портале Azure](https://portal.azure.com) в параметрах **AccountName** и **AccountKey**. 

В этом примере показано, как объявить статическое поле для размещения строки подключения к Azure Cosmos DB:

```java
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=https;" + 
    "AccountName=your_cosmosdb_account;" + 
    "AccountKey=your_account_key;" + 
    "TableEndpoint=https://your_endpoint;" ;
```

Если приложение выполняется в роли на платформе Azure, строку подключения можно сохранить в файле конфигурации службы (*ServiceConfiguration.cscfg*), к которому можно обратиться с помощью метода **RoleEnvironment.getConfigurationSettings** . Ниже приведен пример получения строки подключения из элемента **Setting** с именем *StorageConnectionString* в файле конфигурации службы:

```java
// Retrieve storage account from connection-string.
String storageConnectionString =
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");
```

Также вы можете сохранить строку подключения в файле проекта config.properties.

```java
StorageConnectionString = DefaultEndpointsProtocol=https;AccountName=your_account;AccountKey=your_account_key;TableEndpoint=https://your_table_endpoint/
```

В приведенных ниже примерах предполагается, что вы использовали один из описанных выше методов для получения строки подключения к хранилищу.

## <a name="create-a-table"></a>Создание таблицы

Объект `CloudTableClient` позволяет получить объекты ссылки для таблиц и сущностей. Следующий код создает объект `CloudTableClient` и использует его для создания нового объекта `CloudTable`, который представляет таблицу people. 

> [!NOTE]
> Есть и другие способы создать объекты `CloudStorageAccount`. Дополнительные сведения см. в описании `CloudStorageAccount` в статье [справочнике по пакету SDK для клиента службы хранилища Azure].
>

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create the table if it doesn't exist.
    String tableName = "people";
    CloudTable cloudTable = tableClient.getTableReference(tableName);
    cloudTable.createIfNotExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="list-the-tables"></a>Перечисление таблиц

Чтобы получить список таблиц, вызовите метод **CloudTableClient.listTables()** для извлечения пригодного к итерации списка имен таблиц.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Loop through the collection of table names.
    for (String table : tableClient.listTables())
    {
        // Output each table name.
        System.out.println(table);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="add-an-entity-to-a-table"></a>Добавление сущности в таблицу

Сущности сопоставляются с объектами Java с помощью пользовательского класса, реализующего `TableEntity`. Для удобства класс `TableServiceEntity` реализует `TableEntity` и использует отражение для сопоставления свойств с указанными для свойств методами получения и задания. Чтобы добавить сущность в таблицу, сначала создайте класс, который определяет свойства сущности. Следующий код определяет класс сущностей, который использует имя клиента как ключ строки, а фамилию клиента — как ключ раздела. Вместе ключ раздела и ключ строки сущности уникальным образом идентифицируют сущность в таблице. Сущности с одним ключом раздела можно запрашивать быстрее, чем сущности с разными ключами раздела.

```java
public class CustomerEntity extends TableServiceEntity {
    public CustomerEntity(String lastName, String firstName) {
        this.partitionKey = lastName;
        this.rowKey = firstName;
    }

    public CustomerEntity() { }

    String email;
    String phoneNumber;

    public String getEmail() {
        return this.email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPhoneNumber() {
        return this.phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }
}
```

Табличные операции с сущностями требуют наличия объекта `TableOperation`. Этот объект определяет операцию, которая выполняется для сущности с помощью объекта `CloudTable`. Следующий код создает новый экземпляр класса `CustomerEntity` с подлежащими сохранению данными клиента. Затем этот код вызывает `TableOperation`.insertOrReplace**, чтобы создать объект `TableOperation` для вставки сущности в таблицу, а затем связывает с ним новый экземпляр `CustomerEntity`. И наконец, код вызывает метод `execute` объекта `CloudTable`, определяя таблицу people и новый объект `TableOperation`, который затем отправляет запрос в службу хранилища для вставки новой сущности клиента в таблицу people или замены сущности, если она уже существует.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.setEmail("Walter@contoso.com");
    customer1.setPhoneNumber("425-555-0101");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer1 = TableOperation.insertOrReplace(customer1);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer1);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-a-batch-of-entities"></a>Вставка пакета сущностей

Вы можете вставить пакет сущностей в таблицу в одной операции записи. Следующий код создает объект `TableBatchOperation` и добавляет в него три операции вставки. Каждая операция вставки добавляется путем создания объекта сущности, установки его значений и последующего вызова метода `insert` для объекта `TableBatchOperation`, чтобы связать сущность с новой операцией вставки. Затем код вызывает метод `execute` объекта `CloudTable`, определяя таблицу people и объект `TableBatchOperation`, который отправляет пакет операций таблицы в службу хранилища в одном запросе.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Define a batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a customer entity to add to the table.
    CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    customer.setEmail("Jeff@contoso.com");
    customer.setPhoneNumber("425-555-0104");
    batchOperation.insertOrReplace(customer);

    // Create another customer entity to add to the table.
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.setEmail("Ben@contoso.com");
    customer2.setPhoneNumber("425-555-0102");
    batchOperation.insertOrReplace(customer2);

    // Create a third customer entity to add to the table.
    CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
    customer3.setEmail("Denise@contoso.com");
    customer3.setPhoneNumber("425-555-0103");
    batchOperation.insertOrReplace(customer3);

    // Execute the batch of operations on the "people" table.
    cloudTable.execute(batchOperation);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

Некоторые другие примечания к пакетным операциям:

* В отдельном пакете можно выполнить до 100 операций вставки, удаления, объединения, замены, вставки или замены и вставки или замены операций в любом сочетании.
* Пакетная операция может иметь операцию извлечения, если она является единственной операцией в пакете.
* У всех сущностей в одной пакетной операции должен быть одинаковый ключ раздела.
* Максимальный объем полезных данных пакетной операции составляет 4 МБ.

## <a name="retrieve-all-entities-in-a-partition"></a>Получение всех сущностей в разделе

Чтобы запросить из таблицы сущности раздела, можно использовать `TableQuery`. Вызовите `TableQuery.from`, чтобы создать запрос для определенной таблицы, который возвращает результаты заданного типа. Следующий код задает фильтр для сущностей с ключомраздела "Smith". Вспомогательный метод `TableQuery.generateFilterCondition` предназначен для создания фильтров для запросов. Вызовите `where` для ссылки, которую вернул метод `TableQuery.from`, чтобы применить фильтр к запросу. Когда выполняется запрос путем вызова `execute` из объекта `CloudTable`, ответ содержит `Iterator` с типом результата `CustomerEntity`. Затем можно передать полученное значение `Iterator` в цикл ForEach для обработки результатов. Этот код выводит на консоль поля каждой сущности в результатах запроса.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Specify a partition query, using "Smith" as the partition key filter.
    TableQuery<CustomerEntity> partitionQuery =
        TableQuery.from(CustomerEntity.class)
        .where(partitionFilter);

    // Loop through the results, displaying information about the entity.
    for (CustomerEntity entity : cloudTable.execute(partitionQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>Получение диапазона сущностей в разделе

Если вы не хотите запрашивать все сущности в разделе, можно указать диапазон с помощью операторов сравнения в фильтре. В следующем коде совместно используются два фильтра для получения всех сущностей в разделе "Smith", где ключ строки (имя) начинается с буквы до "E" в алфавите. После чего результаты запроса выводятся на консоль. Если вы используете сущности, добавленные в таблицу во время работы с разделом о пакетной вставке, то на этот раз возвращаются только две сущности (Ben Smith и Denise Smith), а Jeff Smith не выводится.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Create a filter condition where the row key is less than the letter "E".
    String rowFilter = TableQuery.generateFilterCondition(
        ROW_KEY,
        QueryComparisons.LESS_THAN,
        "E");

    // Combine the two conditions into a filter expression.
    String combinedFilter = TableQuery.combineFilters(partitionFilter,
        Operators.AND, rowFilter);

    // Specify a range query, using "Smith" as the partition key,
    // with the row key being up to the letter "E".
    TableQuery<CustomerEntity> rangeQuery =
        TableQuery.from(CustomerEntity.class)
        .where(combinedFilter);

    // Loop through the results, displaying information about the entity
    for (CustomerEntity entity : cloudTable.execute(rangeQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-single-entity"></a>Извлечение одной сущности

Можно написать запрос для получения отдельной сущности. Следующий код вызывает `TableOperation.retrieve` с параметрами ключа раздела и ключа строки, где указан клиент Jeff Smith. Это дает такой же результат, как создание `Table Query` и применения фильтров. При выполнении операция извлечения возвращает только одну сущность, а не коллекцию. Метод `getResultAsType` приводит результат к типу целевого объекта назначения (объект `CustomerEntity`). Если этот тип не совместим с типом, указанным в запросе, создается исключение. Значение NULL возвращается, если ни одна сущность не подходит по ключам раздела и строки. Указание ключа раздела и ключа строки в запросе — самый быстрый способ извлечь одну сущность из службы таблиц.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff"
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Output the entity.
    if (specificEntity != null)
    {
        System.out.println(specificEntity.getPartitionKey() +
            " " + specificEntity.getRowKey() +
            "\t" + specificEntity.getEmail() +
            "\t" + specificEntity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="modify-an-entity"></a>Изменение сущности

Чтобы изменить сущность, извлеките ее из службы таблиц, измените объект сущности и сохраните изменения в службе таблиц с помощью операции замены или объединения. Следующий код изменяет существующий номер телефона клиента. Вместо метода **TableOperation.insert**, который мы использовали для вставки данных, этот код вызывает **TableOperation.replace**. Метод **CloudTableClient.execute** вызывает службу таблиц и заменяет сущность, если только другое приложение не изменило ее с момента извлечения данным приложением. Когда это происходит, возникает исключение, и сущность необходимо получить, изменить и сохранить повторно. Этот оптимистичный шаблон повторения в случае конфликтов широко применяется в системе распределенного хранения.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Specify a new phone number.
    specificEntity.setPhoneNumber("425-555-0105");

    // Create an operation to replace the entity.
    TableOperation replaceEntity = TableOperation.replace(specificEntity);

    // Submit the operation to the table service.
    cloudTable.execute(replaceEntity);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="query-a-subset-of-entity-properties"></a>Запрос подмножества свойств сущности

Запрос к таблице может получить лишь несколько свойств сущности. Этот метод, который называется "проекцией", снижает потребление пропускной способности и может повысить производительность запросов, особенно для крупных сущностей. Запрос в следующем коде использует метод `select`, чтобы возвратить только адреса электронной почты сущностей в таблице. Результаты проецируются в коллекцию объектов `String` с помощью метода `Entity Resolver`, который выполняет преобразование типов сущностей, возвращенных с сервера. Дополнительные сведения о проекции см. в записи блога [Azure Tables: Introducing Upsert and Query Projection][Azure Tables: Introducing Upsert and Query Projection] (Таблицы Azure: введение в Upsert и проекции в запросах). Эта проекция не поддерживается в эмуляторе локального хранилища, поэтому этот код выполняется только при использовании учетной записи хранения в службе таблиц.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Define a projection query that retrieves only the Email property
    TableQuery<CustomerEntity> projectionQuery =
        TableQuery.from(CustomerEntity.class)
        .select(new String[] {"Email"});

    // Define an Entity resolver to project the entity to the Email value.
    EntityResolver<String> emailResolver = new EntityResolver<String>() {
        @Override
        public String resolve(String PartitionKey, String RowKey, Date timeStamp, HashMap<String, EntityProperty> properties, String etag) {
            return properties.get("Email").getValueAsString();
        }
    };

    // Loop through the results, displaying the Email values.
    for (String projectedString :
        cloudTable.execute(projectionQuery, emailResolver)) {
            System.out.println(projectedString);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-or-replace-an-entity"></a>Вставка и замена сущностей

Часто требуется добавить сущность в таблицу, не зная, присутствует ли она там. Операция вставки или замены позволяет сделать один запрос, который вставит сущность, если она не существует, или заменит существующую сущность. В продолжение материала предыдущих примеров следующий код вставляет или заменяет сущность "Walter Harp". После создания новой сущности этот код вызывает метод **TableOperation.insertOrReplace**. Далее код вызывает метод **execute** для объекта **Cloud Table**, передавая в качестве параметров таблицу и табличную операцию вставки или замены. Чтобы обновить только часть сущности, можно вместо этого использовать метод **TableOperation.insertOrMerge**. Операция "вставка или замена" не поддерживается в эмуляторе локального хранилища, поэтому этот код выполняется только с учетной записью хранения в службе таблиц. Дополнительные сведения об операциях "вставка или замена" и "вставка или объединение" см. в записи блога [Azure Tables: Introducing Upsert and Query Projection][Azure Tables: Introducing Upsert and Query Projection] (Таблицы Azure: введение в Upsert и проекции в запросах).

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
    customer5.setEmail("Walter@contoso.com");
    customer5.setPhoneNumber("425-555-0106");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer5 = TableOperation.insertOrReplace(customer5);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer5);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-an-entity"></a>Удаление сущности

Сущность можно легко удалить после ее получения. Получив сущность, вызовите `TableOperation.delete` и укажите удаляемую сущность. Затем вызовите `execute` для объекта `CloudTable`. Следующий код извлекает и удаляет сущность клиента.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff = TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    CustomerEntity entitySmithJeff =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Create an operation to delete the entity.
    TableOperation deleteSmithJeff = TableOperation.delete(entitySmithJeff);

    // Submit the delete operation to the table service.
    cloudTable.execute(deleteSmithJeff);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-a-table"></a>Удаление таблицы

Наконец, следующий код удаляет таблицу из учетной записи хранения. После удаления таблицы ее нельзя создать заново в течение примерно 40 секунд. 

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Delete the table and all its data if it exists.
    CloudTable cloudTable = tableClient.getTableReference("people");
    cloudTable.deleteIfExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

[!INCLUDE [storage-check-out-samples-java](../../includes/storage-check-out-samples-java.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Getting Started with Azure Table Service in Java](https://github.com/Azure-Samples/storage-table-java-getting-started) (Приступая к работе со службой таблиц Azure на языке Java)
* [Обозреватель хранилищ Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.
* [Пакет SDK для службы хранилища Azure для Java][Azure Storage SDK for Java]
* [справочнике по пакету SDK для клиента службы хранилища Azure][Azure Storage Client SDK Reference]
* [REST API службы хранилища Azure][Azure Storage REST API]
* [Блог рабочей группы службы хранилища Azure][Azure Storage Team Blog]

Дополнительные сведения см. в разделе [Azure for Java developers](/java/azure) (Azure для разработчиков Java).

[Azure SDK for Java]: https://go.microsoft.com/fwlink/?LinkID=525671
[Azure Storage SDK for Java]: https://github.com/Azure/azure-storage-java/tree/v8.6.5
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[справочнике по пакету SDK для клиента службы хранилища Azure]: https://azure.github.io/azure-storage-java/
[Azure Storage REST API]: /rest/api/storageservices/
[Azure Storage Team Blog]: https://blogs.msdn.microsoft.com/windowsazurestorage/