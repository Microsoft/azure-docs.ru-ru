---
title: Пакет REST SDK для Phoenix Query Server — Azure HDInsight
description: Установка и использование пакета REST SDK для Phoenix Query Server в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/01/2020
ms.openlocfilehash: 5ce459e2820cfd2ea5cacb036f5d6885c9c5afa0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98933370"
---
# <a name="apache-phoenix-query-server-rest-sdk"></a>Пакет REST SDK для Apache Phoenix Query Server

[Apache Phoenix](https://phoenix.apache.org/) представляет собой реляционную базу данных на основе [Apache HBase](apache-hbase-overview.md) с открытым кодом и высоким уровнем параллелизма. Phoenix позволяет выполнять SQL-подобные запросы к HBase с помощью средств SSH, например [SQLLine](apache-hbase-query-with-phoenix.md). Также Phoenix предоставляет HTTP-сервер, именуемый Phoenix Query Server (PQS), — тонкий клиент, который поддерживает два механизма транспортировки для связи с клиентами: JSON и буферы протокола. Буферы протокола — механизм по умолчанию, предоставляющий более эффективную связь, чем JSON.

В этой статье описывается использование пакета PQS REST SDK для создания таблиц, построчного и массового выполнения операции upsert, а также выбора данных с помощью инструкций SQL. В примерах используется [драйвер Microsoft .NET для Apache Phoenix Query Server](https://www.nuget.org/packages/Microsoft.Phoenix.Client). Этот пакет SDK основан на API-интерфейсах [Apache Calcite Avatica](https://calcite.apache.org/avatica/), где монопольно используются буферы протокола для формата сериализации.

Дополнительные сведения см. в [справочнике по буферам протокола Apache Calcite Avatica](https://calcite.apache.org/avatica/docs/protobuf_reference.html).

## <a name="install-the-sdk"></a>Установка пакета SDK

Драйвер Microsoft .NET для Apache Phoenix Query Server предоставляется в виде пакета NuGet, который можно установить из **консоли диспетчера пакетов NuGet** в Visual Studio с помощью следующей команды:

```console
Install-Package Microsoft.Phoenix.Client
```

## <a name="instantiate-new-phoenixclient-object"></a>Создание экземпляра объекта PhoenixClient

Чтобы начать использовать библиотеку, создайте новый экземпляр объекта `PhoenixClient`, передав в него данные `ClusterCredentials`, содержащие `Uri` вашего кластера, а также имя пользователя и пароль Apache Hadoop.

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net/"), "USERNAME", "PASSWORD");
client = new PhoenixClient(credentials);
```

Замените CLUSTERNAME именем кластера HDInsight HBase, а USERNAME и PASSWORD — учетными данными Hadoop, указанными при создании кластера. Имя пользователя Hadoop по умолчанию — **admin**.

## <a name="generate-unique-connection-identifier"></a>Создание уникального идентификатора подключения

Чтобы отправить один или несколько запросов к PQS, необходимо добавить уникальный идентификатор подключения для связывания запросов с подключением.

```csharp
string connId = Guid.NewGuid().ToString();
```

В каждом примере сначала выполняется вызов метода `OpenConnectionRequestAsync` с передачей уникального идентификатора подключения. Далее следует определить `ConnectionProperties` и `RequestOptions`, передавая эти объекты и созданный идентификатор подключения в метод `ConnectionSyncRequestAsync`. В PQS объект `ConnectionSyncRequest` помогает обеспечить согласованное представление свойств базы данных для клиента и сервера.

## <a name="connectionsyncrequest-and-its-connectionproperties"></a>ConnectionSyncRequest и соответствующий объект ConnectionProperties

Для вызова `ConnectionSyncRequestAsync` передайте объект `ConnectionProperties`.

```csharp
ConnectionProperties connProperties = new ConnectionProperties
{
    HasAutoCommit = true,
    AutoCommit = true,
    HasReadOnly = true,
    ReadOnly = false,
    TransactionIsolation = 0,
    Catalog = "",
    Schema = "",
    IsDirty = true
};
await client.ConnectionSyncRequestAsync(connId, connProperties, options);
```

Ниже приведены некоторые важные свойства.

| Свойство. | Описание |
| -- | -- |
| AutoCommit | Логическое значение, обозначающее, включено ли свойство `autoCommit` для транзакций Phoenix. |
| ReadOnly | Логическое значение, обозначающее, является ли подключение доступным только для чтения. |
| TransactionIsolation | Целое число, указывающее уровень изоляции транзакций в соответствии со спецификацией JDBC (см. следующую таблицу).|
| Каталог | Имя каталога, который следует использовать при получении свойств подключения. |
| схема | Имя схемы, которую следует использовать при получении свойств подключения. |
| IsDirty | Логическое значение, указывающее, были ли изменены свойства. |

Возможные значения `TransactionIsolation`:

| Значение изоляции | Описание |
| -- | -- |
| 0 | Транзакции не поддерживаются. |
| 1 | Могут возникать операции чтения "грязных" данных, а также неповторяемые и фантомные операции чтения. |
| 2 | Операции чтения "грязных" данных предотвращаются, но могут возникать неповторяемые и фантомные операции чтения. |
| 4 | Операции чтения "грязных" данных и неповторяемые операции чтения предотвращаются, но могут возникать фантомные операции чтения. |
| 8 | Все операции чтения "грязных" данных, неповторяемые и фантомные операции чтения предотвращаются. |

## <a name="create-a-new-table"></a>Создание таблицы

В HBase, как и в любой другой реляционной СУБД, данные хранятся в таблицах. Phoenix использует стандартные запросы SQL для создания новых таблиц при определении первичного ключа и типов столбцов.

В этом примере и всех последующих примерах используется экземпляр `PhoenixClient` объекта, как определено в [экземпляре нового объекта PhoenixClient](#instantiate-new-phoenixclient-object).

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// You can set certain request options, such as timeout in milliseconds:
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
CreateStatementResponse createStatementResponse = null;
OpenConnectionResponse openConnResponse = null;

try
{
    // Opening connection
    var info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Create the statement
    createStatementResponse = client.CreateStatementRequestAsync(connId, options).Result;
    
    // Create the table if it does not exist
    string sql = "CREATE TABLE IF NOT EXISTS Customers (Id varchar(20) PRIMARY KEY, FirstName varchar(50), " +
        "LastName varchar(100), StateProvince char(2), Email varchar(255), Phone varchar(15))";
    await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    Console.WriteLine($"Table \"Customers\" created.");
}
catch (Exception e)
{
    Console.WriteLine(e);
    throw;
}
finally
{
    if (createStatementResponse != null)
    {
        client.CloseStatementRequestAsync(connId, createStatementResponse.StatementId, options).Wait();
        createStatementResponse = null;
    }

    if (openConnResponse != null)
    {
        client.CloseConnectionRequestAsync(connId, options).Wait();
        openConnResponse = null;
    }
}
```

Предыдущий пример создает новую таблицу с именем `Customers` с помощью параметра `IF NOT EXISTS`. Вызов `CreateStatementRequestAsync` создает новую инструкцию на сервере Avitica (PQS). Блок `finally` закрывает возвращаемые объекты `CreateStatementResponse` и `OpenConnectionResponse`.

## <a name="insert-data-individually"></a>Вставка данных по отдельности

В этом примере показана отдельная вставка данных, ссылающаяся на коллекцию `List<string>` сокращений штатов и территорий США:

```csharp
var states = new List<string> { "AL", "AK", "AS", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FM", "FL", "GA", "GU", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MH", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "MP", "OH", "OK", "OR", "PW", "PA", "PR", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VI", "VA", "WA", "WV", "WI", "WY" };
```

`StateProvince`Значение столбца таблицы будет использоваться в более поздней операции выбора.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = await client.PrepareRequestAsync(connId, sql, 100, options);
    statementHandle = prepareResponse.Statement;
    
    var r = new Random();

    // Insert 300 rows
    for (int i = 0; i < 300; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0,1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        Console.WriteLine("Inserting customer " + i);

        await client.ExecuteRequestAsync(statementHandle, list, long.MaxValue, true, options);
    }

    await client.CommitRequestAsync(connId, options);

    Console.WriteLine("Upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

Структура выполнения инструкции insert аналогична созданию новой таблицы. В конце `try` блока транзакция зафиксирована явным образом. В этом примере транзакция insert повторяется 300 раз. В следующем примере показан более эффективный процесс пакетной вставки.

## <a name="batch-insert-data"></a>Пакетная вставка данных

Следующий код почти идентичен коду для вставки данных по отдельности. В этом примере используется объект `UpdateBatch` при вызове `ExecuteBatchRequestAsync` вместо повторяющегося вызова `ExecuteRequestAsync` с подготовленной инструкцией.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
CreateStatementResponse createStatementResponse = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Creating statement
    createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = client.PrepareRequestAsync(connId, sql, long.MaxValue, options).Result;
    var statementHandle = prepareResponse.Statement;
    var updates = new pbc.RepeatedField<UpdateBatch>();

    var r = new Random();

    // Insert 300 rows
    for (int i = 300; i < 600; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0, 1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        var batch = new UpdateBatch
        {
            ParameterValues = list
        };
        updates.Add(batch);

        Console.WriteLine($"Added customer {i} to batch");
    }

    var executeBatchResponse = await client.ExecuteBatchRequestAsync(connId, statementHandle.Id, updates, options);

    Console.WriteLine("Batch upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

В одной тестовой среде вставка 300 новых записей по отдельности заняла почти 2 минуты. А для пакетной вставки 300 записей потребовалось всего 6 секунд.

## <a name="select-data"></a>Выбор данных

В этом примере показано, как повторно использовать одно подключение для выполнения нескольких запросов:

1. Выберите все записи, затем отберите оставшиеся записи после возвращения максимального количества по умолчанию, равного 100.
2. Используйте инструкцию select с полным подсчетом строк для получения единичного скалярного результата.
3. Выполните инструкцию select, которая возвращает общее количество клиентов по каждому штату или территории.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;

try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);
    var createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "SELECT * FROM Customers";
    ExecuteResponse executeResponse = await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> rows = executeResponse.Results[0].FirstFrame.Rows;
    // Loop through all of the returned rows and display the first two columns
    for (int i = 0; i < rows.Count; i++)
    {
        Row row = rows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + " " + row.Value[1].ScalarValue.StringValue);
    }
    
    // 100 is hard-coded on the server side as the default firstframe size
    // FetchRequestAsync is called to get any remaining rows
    Console.WriteLine("");
    Console.WriteLine($"Number of rows: {rows.Count}");

    // Fetch remaining rows, offset is not used, simply set to 0
    // When FetchResponse.Frame.Done is true, all rows were fetched
    FetchResponse fetchResponse = await client.FetchRequestAsync(connId, createStatementResponse.StatementId, 0, int.MaxValue, options);
    Console.WriteLine($"Frame row count: {fetchResponse.Frame.Rows.Count}");
    Console.WriteLine($"Fetch response is done: {fetchResponse.Frame.Done}");
    Console.WriteLine("");

    // Running query 2
    string sql2 = "select count(*) from Customers";
    ExecuteResponse countResponse = await client.PrepareAndExecuteRequestAsync(connId, sql2, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);
    long count = countResponse.Results[0].FirstFrame.Rows[0].Value[0].ScalarValue.NumberValue;

    Console.WriteLine($"Total customer records: {count}");
    Console.WriteLine("");

    // Running query 3
    string sql3 = "select StateProvince, count(*) as Number from Customers group by StateProvince order by Number desc";
    ExecuteResponse groupByResponse = await client.PrepareAndExecuteRequestAsync(connId, sql3, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> stateRows = groupByResponse.Results[0].FirstFrame.Rows;
    for (int i = 0; i < stateRows.Count; i++)
    {
        Row row = stateRows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + ": " + row.Value[1].ScalarValue.NumberValue);
    }
}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

Выходные данные инструкций `select` должны выглядеть примерно так:

```output
id0 first0
id1 first1
id10 first10
id100 first100
id101 first101
id102 first102
. . .
id185 first185
id186 first186
id187 first187
id188 first188

Number of rows: 100
Frame row count: 500
Fetch response is done: True

Total customer records: 600

NJ: 21
CA: 19
GU: 17
NC: 16
IN: 16
MA: 16
AZ: 16
ME: 16
IL: 15
OR: 15
. . . 
MO: 10
HI: 10
GA: 10
DC: 9
NM: 9
MD: 9
MP: 9
SC: 7
AR: 7
MH: 6
FM: 5
```

## <a name="next-steps"></a>Дальнейшие действия

* [Apache Phoenix в HDInsight](../hdinsight-phoenix-in-hdinsight.md)
* [Использование пакета REST SDK для Apache HBase](apache-hbase-rest-sdk.md)
