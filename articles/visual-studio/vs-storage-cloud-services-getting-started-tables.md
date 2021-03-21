---
title: Приступая к работе с хранилищем таблиц с помощью Visual Studio (облачные службы)
description: Как приступить к работе, используя табличное хранилище Azure в проекте облачной службы в Visual Studio после подключения к учетной записи хранения с помощью подключенных служб Visual Studio
services: storage
author: ghogen
manager: jillfra
ms.assetid: a3a11ed8-ba7f-4193-912b-e555f5b72184
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure, devx-track-csharp
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: e5e687b172b49fec5f77615e332d0a2204162c43
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95545795"
---
# <a name="getting-started-with-azure-table-storage-and-visual-studio-connected-services-cloud-services-projects"></a>Начало работы с табличным хранилищем Azure и подключенными службами Visual Studio (проектами облачных служб)
[!INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## <a name="overview"></a>Обзор
В этой статье описывается, как приступить к использованию хранилища таблиц Azure в Visual Studio после создания учетной записи хранения Azure или указания ссылки на нее в проекте облачных служб с помощью диалогового окна **Добавление подключенных служб** в Visual Studio. Операция **Добавить подключенные службы** устанавливает соответствующие пакеты NuGet для доступа к службе хранилища Azure в вашем проекте и добавляет строку подключения для учетной записи хранения в файлы конфигурации проекта.

В службе хранилища таблиц Azure можно хранить большие объемы структурированных данных. Эта служба — хранилище данных NoSQL, которое принимает вызовы внутри и снаружи облака Azure с проверкой подлинности. Таблицы Azure идеально подходят для хранения нереляционных структурированных данных.

Чтобы начать работу, сначала необходимо создать таблицу в учетной записи хранения. Мы покажем, как создать таблицу Azure в коде, и как выполнять базовые операции с таблицами и сущностями, например добавлять, изменять и читать сущности таблицы. Примеры написаны на языке C \# и используют [клиентскую библиотеку служба хранилища Microsoft Azure для .NET](/previous-versions/azure/dn261237(v=azure.100)).

**Примечание**. Некоторые интерфейсы API, которые выполняют вызовы к службе хранилища Azure, являются асинхронными. Дополнительные сведения см. в статье [Asynchronous Programming with async and await (C#)](/previous-versions/hh191443(v=vs.140)) (Асинхронное программирование с использованием ключевых слов Async и Await (C#)). В следующем примере кода предполагается, что используются асинхронные методы программирования.

* Дополнительные сведения о выполнении программных операций с таблицами см. в статье [Приступая к работе с хранилищем таблиц Azure с помощью .NET](../cosmos-db/tutorial-develop-table-dotnet.md).
* Общие сведения о службе хранилища Azure см. в [документации по хранилищу](https://azure.microsoft.com/documentation/services/storage/) .
* Общие сведения об облачных службах Azure см. в [документации по облачным службам](https://azure.microsoft.com/documentation/services/cloud-services/).
* Дополнительную информацию о программировании для ASP.NET см. в разделе [ASP.NET](https://www.asp.net).

## <a name="access-tables-in-code"></a>Доступ к таблицам в коде
Для доступа к таблицам в проектах облачных служб необходимо включить следующие элементы во все файлы исходного кода C#, которые обращаются к табличному хранилищу Azure.

1. Убедитесь, что объявления пространств имен в верхней части файла C# содержат указанные ниже выражения **using** .
   
    ```csharp
    using Microsoft.Framework.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Table;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Framework.Logging.LogLevel;
    ```
2. Получите объект **CloudStorageAccount** , представляющий данные учетной записи хранения. Используйте следующий код, чтобы получить строку подключения и сведения об учетной записи хранения из конфигурации службы Azure.
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("<storage account name>
    _AzureStorageConnectionString"));
    ```
   > [!NOTE]
   > Вставьте весь код, представленный выше, перед кодом в следующих примерах.
   > 
   > 
3. Получите объект **CloudTableClient** , чтобы указать ссылку на объекты таблицы в своей учетной записи хранения.
   
    ```csharp
    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```
4. Получите объект ссылки **CloudTable** для указания ссылки на определенную таблицу и сущности.
   
    ```csharp
    // Get a reference to a table named "peopleTable".
    CloudTable peopleTable = tableClient.GetTableReference("peopleTable");
    ```

## <a name="create-a-table-in-code"></a>Создание таблицы в коде
Чтобы создать таблицу Azure, просто добавьте вызов **CreateIfNotExistsAsync** после получения объекта **CloudTable**, как описано в разделе "Доступ к таблицам в коде".

```csharp
// Create the CloudTable if it does not exist.
await peopleTable.CreateIfNotExistsAsync();
```

## <a name="add-an-entity-to-a-table"></a>Добавление сущности в таблицу
Чтобы добавить сущность в таблицу, создайте класс, который определяет свойства сущности. Следующий код определяет класс сущностей с именем **CustomerEntity** , который использует имя клиента как ключ строки, а фамилию клиента — как ключ раздела.

```csharp
public class CustomerEntity : TableEntity
{
    public CustomerEntity(string lastName, string firstName)
    {
        this.PartitionKey = lastName;
        this.RowKey = firstName;
    }

    public CustomerEntity() { }

    public string Email { get; set; }

    public string PhoneNumber { get; set; }
}
```

Табличные операции с участием сущностей выполняются с использованием объекта **CloudTable** , ранее созданного в разделе «Доступ к таблицам в коде». Объект **TableOperation** представляет операции, которые необходимо выполнить. В следующем примере кода показано создание объектов **CloudTable** и **CustomerEntity**. Чтобы подготовить операцию, создается **TableOperation** для вставки сущности customer в таблицу. Наконец, эта операция выполняется с помощью вызова **CloudTable.ExecuteAsync**.

```csharp
// Create a new customer entity.
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
customer1.Email = "Walter@contoso.com";
customer1.PhoneNumber = "425-555-0101";

// Create the TableOperation that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer1);

// Execute the insert operation.
await peopleTable.ExecuteAsync(insertOperation);
```


## <a name="insert-a-batch-of-entities"></a>Вставка пакета сущностей
В таблицу можно вставить несколько сущностей с помощью одной операции записи. Указанный ниже пример кода создает два объекта сущностей (Jeff Smith и Ben Smith), добавляет их в объект **TableBatchOperation** с помощью метода Insert и запускает операцию с помощью вызова **CloudTable.ExecuteBatchAsync**.

```csharp
// Create the batch operation.
TableBatchOperation batchOperation = new TableBatchOperation();

// Create a customer entity and add it to the table.
CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
customer1.Email = "Jeff@contoso.com";
customer1.PhoneNumber = "425-555-0104";

// Create another customer entity and add it to the table.
CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
customer2.Email = "Ben@contoso.com";
customer2.PhoneNumber = "425-555-0102";

// Add both customer entities to the batch insert operation.
batchOperation.Insert(customer1);
batchOperation.Insert(customer2);

// Execute the batch operation.
await peopleTable.ExecuteBatchAsync(batchOperation);
```

## <a name="get-all-of-the-entities-in-a-partition"></a>Получение всех сущностей в разделе
Чтобы запросить все сущности из таблицы, используйте объект **TableQuery** . Следующий пример кода задает фильтр для сущностей с ключом раздела Smith. Этот пример выводит на консоль поля каждой сущности в результатах запроса.

```csharp
// Construct the query operation for all customer entities where PartitionKey="Smith".
TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
    .Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

// Print the fields for each customer.
TableContinuationToken token = null;
do
{
    TableQuerySegment<CustomerEntity> resultSegment = await peopleTable.ExecuteQuerySegmentedAsync(query, token);
    token = resultSegment.ContinuationToken;

    foreach (CustomerEntity entity in resultSegment.Results)
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
    }
} while (token != null);

return View();
```


## <a name="get-a-single-entity"></a>Получение одной сущности
Можно написать запрос для получения отдельной сущности. Следующий пример кода использует **TableOperation** для указания клиента "Ben Smith". Этот метод возвращает только одну сущность, а не коллекцию, а возвращаемое значение в **TableResult. Result** является объектом **CustomerEntity** . Указание ключей разделов и строк в запросе — самый быстрый способ извлечь одну сущность из службы **таблиц** .

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

// Print the phone number of the result.
if (retrievedResult.Result != null)
    Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
else
    Console.WriteLine("The phone number could not be retrieved.");
```

## <a name="delete-an-entity"></a>Удаление сущности
После нахождения сущности ее можно удалить. В следующем примере код выполняет поиск сущности с именем "Ben Smith" и при нахождении удаляет ее.

```csharp
// Create a retrieve operation that expects a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the operation.
TableResult retrievedResult = peopleTable.Execute(retrieveOperation);

// Assign the result to a CustomerEntity object.
CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

// Create the Delete TableOperation and then execute it.
if (deleteEntity != null)
{
    TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

    // Execute the operation.
    await peopleTable.ExecuteAsync(deleteOperation);

    Console.WriteLine("Entity deleted.");
}

else
    Console.WriteLine("Couldn't delete the entity.");
```

## <a name="next-steps"></a>Дальнейшие действия
[!INCLUDE [vs-storage-dotnet-tables-next-steps](../../includes/vs-storage-dotnet-tables-next-steps.md)]