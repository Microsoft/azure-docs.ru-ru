---
title: Использование Хранилища таблиц Azure и API таблиц для Azure Cosmos DB с C++
description: Хранение структурированных данных в облаке с помощью хранилища таблиц Azure или API таблиц Azure Cosmos DB, используя C++.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: cpp
ms.topic: sample
ms.date: 10/07/2019
author: sakash279
ms.author: akshanka
ms.openlocfilehash: 79b58b76954cf15289e85dbf763b7a399897635d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94489885"
---
# <a name="how-to-use-azure-table-storage-and-azure-cosmos-db-table-api-with-c"></a>Как использовать хранилище таблиц Azure и API таблиц Azure Cosmos DB в C++
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

В этом руководстве показаны типичные сценарии с использованием службы табличного хранилища Azure или API таблиц Azure Cosmos DB. Примеры написаны на C++ и используют [клиентскую библиотеку хранилища Azure для C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md). В этой статье рассматриваются следующие сценарии:

* Создание и удаление таблицы.
* Работа с сущностями таблиц.

> [!NOTE]
> Данное руководство предназначено для клиентской библиотеки хранилища Azure для С++ версии 1.0.0 и выше. Рекомендуемая версия клиентской библиотеки хранилища — 2.2.0. Она доступна на сайте [NuGet](https://www.nuget.org/packages/wastorage) или [GitHub](https://github.com/Azure/azure-storage-cpp/).
>

## <a name="create-accounts"></a>Создание учетных записей

### <a name="create-an-azure-service-account"></a>Создание учетной записи службы Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

### <a name="create-an-azure-cosmos-db-table-api-account"></a>Создание учетной записи API таблиц Azure Cosmos DB

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-c-application"></a>Создание приложения на C++

В этом руководстве вы используете функции хранилища из приложения C++. Для этого установите клиентскую библиотеку службы хранилища Azure для C++.

Чтобы установить клиентскую библиотеку службы хранилища для C++, используйте следующие методы:

* **Linux:** следуйте инструкциям, указанным в файле README [клиентской библиотеки хранилища Azure для C++ на странице для начала работы в Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux).
* **Windows:** В Windows в качестве диспетчера зависимостей используйте [vcpkg](https://github.com/microsoft/vcpkg). Следуйте инструкциям из этого [краткого руководства](https://github.com/microsoft/vcpkg#quick-start), чтобы инициализировать vcpkg. Затем, чтобы установить библиотеку, используйте следующую команду:

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Руководство по созданию исходного кода и экспорту в Nuget можно просмотреть в файле [README](https://github.com/Azure/azure-storage-cpp#download--install).

### <a name="configure-access-to-the-table-client-library"></a>Настройка доступа к клиентской библиотеке таблиц

Чтобы использовать API-интерфейсы хранилища Azure, добавьте следующие инструкции `include` в начало файла C++:

```cpp
#include <was/storage_account.h>
#include <was/table.h>
```

Клиент службы хранилища Azure или Cosmos DB хранит в строке подключения идентификаторы конечных точек и учетные данные для доступа к службам управления данными. При запуске клиентского приложения следует предоставить строку для подключения к хранилищу или к Azure Cosmos DB в правильном формате.

### <a name="set-up-an-azure-storage-connection-string"></a>Настройка строки подключения к службе хранилища Azure

В этом примере показано, как объявить статическое поле для размещения строки подключения к службе хранилища Azure:  

```cpp
// Define the Storage connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_storage_account>;AccountKey=<your_storage_account_key>"));
```

Используйте имя учетной записи хранения для `<your_storage_account>`. Для <your_storage_account_key> используйте ключ доступа для учетной записи хранения, указанной на [портале Azure](https://portal.azure.com). Сведения об учетных записях хранения и ключах доступа см. в статье [Create a storage account](../storage/common/storage-account-create.md) (Создание учетной записи хранения).

### <a name="set-up-an-azure-cosmos-db-connection-string"></a>Настройка строки подключения к Azure Cosmos DB

В этом примере показано, как объявить статическое поле для размещения строки подключения к Azure Cosmos DB:

```cpp
// Define the Azure Cosmos DB connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_cosmos_db_account>;AccountKey=<your_cosmos_db_account_key>;TableEndpoint=<your_cosmos_db_endpoint>"));
```

Используйте имя учетной записи Azure Cosmos DB для `<your_cosmos_db_account>`. Введите первичный ключ для `<your_cosmos_db_account_key>`. Введите конечную точку, указанную на [портале Azure](https://portal.azure.com) для `<your_cosmos_db_endpoint>`.

Чтобы протестировать приложение на локальном компьютере с Windows, можно использовать Эмулятор службы хранилища Azure, устанавливаемый с пакетом [Azure SDK](https://azure.microsoft.com/downloads/). Эмулятор службы хранилища — это служебная программа, моделирующая службы больших двоичных объектов, очередей и таблиц, которые доступны в Azure на локальном компьютере разработки. В следующем примере показано, как объявить статическое поле для размещения строки подключения для эмулятора локального хранилища:  

```cpp
// Define the connection string with Azure Storage Emulator.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

Чтобы запустить Эмулятор службы хранилища Azure с рабочего стола Windows, нажмите кнопку **Пуск** или клавишу Windows. Введите и запустите *эмулятор хранения Microsoft Azure*. Дополнительные сведения см. в статье [Использование Эмулятора службы хранилища Azure для разработки и тестирования](../storage/common/storage-use-emulator.md).

### <a name="retrieve-your-connection-string"></a>Получить строку подключения

Информацию о своей учетной записи хранения можно представить с помощью класса `cloud_storage_account`. Чтобы получить данные учетной записи хранения из строки подключения хранилища, можно использовать метод `parse`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

Затем получите ссылку на класс `cloud_table_client`. Этот класс позволяет извлечь объекты ссылки для таблиц и сущностей, которые хранятся в службе хранилища таблиц. Следующий код создает объект `cloud_table_client` с помощью объекта учетной записи хранения, полученного выше:  

```cpp
// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();
```

## <a name="create-and-add-entities-to-a-table"></a>Создание и добавление сущностей в таблицу

### <a name="create-a-table"></a>Создание таблицы

Объект `cloud_table_client` позволяет получить объекты ссылки для таблиц и сущностей. Следующий код создает объект `cloud_table_client` и использует его для создания таблицы.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);  

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();  
```

### <a name="add-an-entity-to-a-table"></a>Добавление сущности в таблицу

Чтобы добавить сущность в таблицу, создайте объект `table_entity` и передайте его в `table_operation::insert_entity`. Следующий код использует имя пользователя в качестве ключа строки, а фамилию клиента — как ключ раздела. Вместе ключ раздела и ключ строки сущности уникальным образом идентифицируют сущность в таблице. Сущности с одним ключом раздела можно запрашивать быстрее, чем сущности с разными ключами раздела. Использование различных ключей разделов позволяет повысить масштабируемость параллельных операций. Дополнительные сведения см. в статье [Производительность хранилища Microsoft Azure и контрольный список масштабируемости](../storage/blobs/storage-performance-checklist.md).

Следующий код создает экземпляр `table_entity` с подлежащими сохранению данными клиента. Затем код вызывает `table_operation::insert_entity`, чтобы создать объект `table_operation` для вставки сущности в таблицу, и связывает с ним новую сущность таблицы. В завершение код вызывает метод `execute` для объекта `cloud_table`. Новый `table_operation` отправляет запрос в службу таблиц для вставки сущности нового клиента в таблицу `people`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();

// Create a new customer entity.
azure::storage::table_entity customer1(U("Harp"), U("Walter"));

azure::storage::table_entity::properties_type& properties = customer1.properties();
properties.reserve(2);
properties[U("Email")] = azure::storage::entity_property(U("Walter@contoso.com"));

properties[U("Phone")] = azure::storage::entity_property(U("425-555-0101"));

// Create the table operation that inserts the customer entity.
azure::storage::table_operation insert_operation = azure::storage::table_operation::insert_entity(customer1);

// Execute the insert operation.
azure::storage::table_result insert_result = table.execute(insert_operation);
```

### <a name="insert-a-batch-of-entities"></a>Вставка пакета сущностей

Вы можете вставить пакет сущностей в службу таблиц за одну операцию записи. Следующий код создает объект `table_batch_operation`, а затем добавляет в него три операции вставки. Каждая операция вставки добавляется путем создания объекта сущности, установки его значений и последующего вызова метода `insert` для объекта `table_batch_operation`, чтобы связать сущность с новой операцией вставки. Затем код вызывает `cloud_table.execute` для выполнения операции.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define a batch operation.
azure::storage::table_batch_operation batch_operation;

// Create a customer entity and add it to the table.
azure::storage::table_entity customer1(U("Smith"), U("Jeff"));

azure::storage::table_entity::properties_type& properties1 = customer1.properties();
properties1.reserve(2);
properties1[U("Email")] = azure::storage::entity_property(U("Jeff@contoso.com"));
properties1[U("Phone")] = azure::storage::entity_property(U("425-555-0104"));

// Create another customer entity and add it to the table.
azure::storage::table_entity customer2(U("Smith"), U("Ben"));

azure::storage::table_entity::properties_type& properties2 = customer2.properties();
properties2.reserve(2);
properties2[U("Email")] = azure::storage::entity_property(U("Ben@contoso.com"));
properties2[U("Phone")] = azure::storage::entity_property(U("425-555-0102"));

// Create a third customer entity to add to the table.
azure::storage::table_entity customer3(U("Smith"), U("Denise"));

azure::storage::table_entity::properties_type& properties3 = customer3.properties();
properties3.reserve(2);
properties3[U("Email")] = azure::storage::entity_property(U("Denise@contoso.com"));
properties3[U("Phone")] = azure::storage::entity_property(U("425-555-0103"));

// Add customer entities to the batch insert operation.
batch_operation.insert_or_replace_entity(customer1);
batch_operation.insert_or_replace_entity(customer2);
batch_operation.insert_or_replace_entity(customer3);

// Execute the batch operation.
std::vector<azure::storage::table_result> results = table.execute_batch(batch_operation);
```

Некоторые другие примечания к пакетным операциям:

* В одном пакете можно выполнить до 100 операций `insert`, `delete`, `merge`, `replace`, `insert-or-merge` и `insert-or-replace` в любых сочетаниях.  
* Пакетная операция может содержать операцию извлечения, если она является единственной в пакете.  
* У всех сущностей в одной пакетной операции должен быть одинаковый ключ раздела.  
* Максимальный объем полезных данных пакетной операции составляет 4 МБ.  

## <a name="query-and-modify-entities"></a>Запросы к сущностям и их изменение

### <a name="retrieve-all-entities-in-a-partition"></a>Получение всех сущностей в разделе

Чтобы запросить все сущности из таблицы, используйте объект `table_query`. Следующий пример кода задает фильтр для сущностей с ключом раздела `Smith`. Этот пример выводит на консоль поля каждой сущности в результатах запроса.  

> [!NOTE]
> Эти методы сейчас не поддерживаются для C++ в базе данных Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Construct the query operation for all customer entities where PartitionKey="Smith".
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::generate_filter_condition(U("PartitionKey"), azure::storage::query_comparison_operator::equal, U("Smith")));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Print the fields for each customer.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

В этом примере запрос возвращает все сущности, соответствующие условиям фильтра. При наличии больших таблиц и необходимости часто загружать сущности таблицы мы рекомендуем хранить данные в хранилище BLOB-объектов Azure.

### <a name="retrieve-a-range-of-entities-in-a-partition"></a>Получение диапазона сущностей в разделе

Если вы не хотите запрашивать все сущности в разделе, можно указать диапазон. Объедините фильтр ключей секций с фильтром ключей строк. В следующем примере кода используются два фильтра для получения всех сущностей в разделе `Smith`, где ключ строки (имя) начинается с буквы алфавита, предшествующей `E`, после чего результаты запроса выводятся в консоль.  

> [!NOTE]
> Эти методы сейчас не поддерживаются для C++ в базе данных Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table query.
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::combine_filter_conditions(
    azure::storage::table_query::generate_filter_condition(U("PartitionKey"),
    azure::storage::query_comparison_operator::equal, U("Smith")),
    azure::storage::query_logical_operator::op_and,
    azure::storage::table_query::generate_filter_condition(U("RowKey"), azure::storage::query_comparison_operator::less_than, U("E"))));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Loop through the results, displaying information about the entity.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

### <a name="retrieve-a-single-entity"></a>Извлечение одной сущности

Можно написать запрос для получения отдельной сущности. В следующем примере кода с помощью объекта `table_operation::retrieve_entity` задается клиент `Jeff Smith`. Данный метод возвращает только одну сущность, а не множество, и возвращенное значение находится в `table_result`. Указание ключа раздела и ключа строки в запросе — самый быстрый способ извлечь одну сущность из службы таблиц.  

```cpp
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Output the entity.
azure::storage::table_entity entity = retrieve_result.entity();
const azure::storage::table_entity::properties_type& properties = entity.properties();

std::wcout << U("PartitionKey: ") << entity.partition_key() << U(", RowKey: ") << entity.row_key()
    << U(", Property1: ") << properties.at(U("Email")).string_value()
    << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
```

### <a name="replace-an-entity"></a>Замена сущности

Чтобы заменить сущность, извлеките ее из службы таблиц и измените объект сущности, а затем сохраните изменения в службе таблиц. Следующий код изменяет существующий номер телефона и адрес электронной почты клиента. Вместо вызова `table_operation::insert_entity` этот код использует `table_operation::replace_entity`. Из-за этого сущность будет полностью заменена на сервере, если только сущность на сервере не была изменена с момента извлечения. Если она была изменена, операция завершается ошибкой. Это необходимо, чтобы приложение случайно не перезаписало изменения, внесенные с момента извлечения до обновления другим компонентом. Правильный алгоритм обработки этой ошибки заключается в том, чтобы снова получить сущность, внести необходимые изменения (если это еще возможно), а затем выполнить еще одну операцию `table_operation::replace_entity`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Replace an entity.
azure::storage::table_entity entity_to_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_replace = entity_to_replace.properties();
properties_to_replace.reserve(2);

// Specify a new phone number.
properties_to_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0106"));

// Specify a new email address.
properties_to_replace[U("Email")] = azure::storage::entity_property(U("JeffS@contoso.com"));

// Create an operation to replace the entity.
azure::storage::table_operation replace_operation = azure::storage::table_operation::replace_entity(entity_to_replace);

// Submit the operation to the Table service.
azure::storage::table_result replace_result = table.execute(replace_operation);
```

### <a name="insert-or-replace-an-entity"></a>Вставка и замена сущностей

Операции `table_operation::replace_entity` завершаются ошибкой, если сущность изменена с момента получения с сервера. Более того, для успешного выполнения операции `table_operation::replace_entity` сначала нужно получить сущность с сервера. Иногда неизвестно, существует ли сущность на сервере. Текущие значения, хранящиеся на нем, несущественны, так как при обновлении все они перезаписываются. Чтобы достичь такого результата, используйте операцию `table_operation::insert_or_replace_entity`. Эта операция вставляет сущность, если она не существует, и заменяет ее, если она существует. В следующем примере кода сущность клиента для `Jeff Smith` все равно извлекается, но затем она сохраняется на сервере с помощью `table_operation::insert_or_replace_entity`. Любые обновления сущности, внесенные между операциями извлечения и обновления, будут перезаписаны.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Insert or replace an entity.
azure::storage::table_entity entity_to_insert_or_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_insert_or_replace = entity_to_insert_or_replace.properties();

properties_to_insert_or_replace.reserve(2);

// Specify a phone number.
properties_to_insert_or_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0107"));

// Specify an email address.
properties_to_insert_or_replace[U("Email")] = azure::storage::entity_property(U("Jeffsm@contoso.com"));

// Create an operation to insert or replace the entity.
azure::storage::table_operation insert_or_replace_operation = azure::storage::table_operation::insert_or_replace_entity(entity_to_insert_or_replace);

// Submit the operation to the Table service.
azure::storage::table_result insert_or_replace_result = table.execute(insert_or_replace_operation);
```

### <a name="query-a-subset-of-entity-properties"></a>Запрос подмножества свойств сущности

Запрос к таблице может получить лишь несколько свойств сущности. Запрос в следующем коде использует метод `table_query::set_select_columns`, чтобы возвратить только адреса электронной почты сущностей в таблице.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define the query, and select only the Email property.
azure::storage::table_query query;
std::vector<utility::string_t> columns;

columns.push_back(U("Email"));
query.set_select_columns(columns);

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Display the results.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key();

    const azure::storage::table_entity::properties_type& properties = it->properties();
    for (auto prop_it = properties.begin(); prop_it != properties.end(); ++prop_it)
    {
        std::wcout << ", " << prop_it->first << ": " << prop_it->second.str();
    }

    std::wcout << std::endl;
}
```

> [!NOTE]
> Запрос нескольких свойств сущности является более эффективной операцией, чем запрос всех свойств.
>

## <a name="delete-content"></a>Удаление содержимого

### <a name="delete-an-entity"></a>Удаление сущности

После извлечения сущности ее можно удалить. После извлечения сущности вызовите `table_operation::delete_entity` с сущностью для удаления. Затем вызовите метод `cloud_table.execute`. Следующий код извлекает и удаляет сущность с ключом раздела `Smith` и ключом строки `Jeff`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Create an operation to delete the entity.
azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

// Submit the delete operation to the Table service.
azure::storage::table_result delete_result = table.execute(delete_operation);  
```

### <a name="delete-a-table"></a>Удаление таблицы

Наконец, следующий пример кода удаляет таблицу из учетной записи хранения. Удаленную таблицу нельзя воссоздать в течение определенного времени после удаления.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Delete the table if it exists
if (table.delete_table_if_exists())
{
    std::cout << "Table deleted!";
}
else
{
    std::cout << "Table didn't exist";
}
```

## <a name="troubleshooting"></a>Устранение неполадок

Для выпуска Visual Studio Community: если сборка проекта завершается ошибкой из-за включаемых файлов *storage_account.h* и *table.h*, удалите параметр компилятора **/permissive-** :

1. В **обозревателе решений** щелкните проект правой кнопкой мыши и выберите **Свойства**.
1. В диалоговом окне **Страницы свойств** разверните узел **Свойства конфигурации**, затем разверните **C/C++** и выберите **Язык**.
1. Для параметра **Режим совместимости** установите значение **Нет**.

## <a name="next-steps"></a>Дальнейшие действия

[Обозреватель хранилищ Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.

Перейдите по приведенным ниже ссылкам, чтобы больше узнать о службе хранилища Azure и API таблиц в Azure Cosmos DB.

* [Общие сведения об API таблиц Azure Cosmos DB](table-introduction.md)
* [Перечисление ресурсов хранилища Azure в C++](../storage/common/storage-c-plus-plus-enumeration.md)
* [Справочник по клиентской библиотеке хранилища для C++](https://azure.github.io/azure-storage-cpp)
* [Документация по службе хранилища Azure](https://azure.microsoft.com/documentation/services/storage/)
