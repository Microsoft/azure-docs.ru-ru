---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL с помощью C#
description: В этом кратком руководстве представлен пример кода C# (.NET), который можно использовать для подключения к базе данных Azure для MySQL и запроса данных из нее.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc, devx-track-csharp
ms.devlang: csharp
ms.topic: quickstart
ms.date: 10/18/2020
ms.openlocfilehash: 3b90d8819b5d327c3ccd143257198c7ec8538f03
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94535832"
---
# <a name="quickstart-use-net-c-to-connect-and-query-data-in-azure-database-for-mysql"></a>Краткое руководство. Подключение к Базе данных Azure для MySQL и запрос данных с помощью .NET (C#)

В этом кратком руководстве объясняется, как подключиться к базе данных Azure для MySQL с помощью приложения C#. Здесь также показано, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных. 

## <a name="prerequisites"></a>Предварительные требования
Для целей этого краткого руководства понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free) бесплатно.
- Создание отдельного сервера Базы данных Azure для MySQL с помощью [портала Azure](./quickstart-create-mysql-server-database-using-azure-portal.md) <br/> или с помощью [Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md), если он еще не создан.
- Выполнить **ОДНО** из действий (в зависимости от того, пользуетесь вы общим или частным доступом), чтобы настроить возможность подключения.

|Действие| Метод подключения|Практическое руководство|
|:--------- |:--------- |:--------- |
| **Настройка правил брандмауэра** | Общие | [Портал](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
| **Настройка конечной точки службы** | Общие | [Портал](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)| 
| **Настройка приватного канала** | Private | [Портал](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) | 

- [Создать базу данных и пользователя без прав администратора](./howto-create-users.md).

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="create-a-c-project"></a>Создание проекта C#
В окне командной строки выполните следующую команду.

```
mkdir AzureMySqlExample
cd AzureMySqlExample
dotnet new console
dotnet add package MySqlConnector
```

## <a name="get-connection-information"></a>Получение сведений о подключении
Получите сведения о подключении, необходимые для подключения к базе данных Azure.для MySQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В меню слева на портале Azure щелкните **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).
3. Щелкните имя сервера.
4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 :::image type="content" source="./media/connect-csharp/1_server-overview-name-login.png" alt-text="Имя сервера базы данных Azure для MySQL":::

## <a name="step-1-connect-and-insert-data"></a>Шаг 1. Подключение и введение данных
Используйте указанный ниже код для подключения и скачивания данных с помощью инструкций SQL `CREATE TABLE` и `INSERT INTO`. В коде используются методы класса `MySqlConnection`:
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync), чтобы установить соединение с MySQL;
- [CreateCommand ()](/dotnet/api/system.data.common.dbconnection.createcommand), чтобы задать свойство CommandText;
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync), чтобы выполнять команды базы данных. 

Замените параметры `Server`, `Database`, `UserID` и `Password` значениями, указанными при создании сервера и базы данных. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlCreate
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "DROP TABLE IF EXISTS inventory;";
                    await command.ExecuteNonQueryAsync();
                    Console.WriteLine("Finished dropping table (if existed)");

                    command.CommandText = "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";
                    await command.ExecuteNonQueryAsync();
                    Console.WriteLine("Finished creating table");

                    command.CommandText = @"INSERT INTO inventory (name, quantity) VALUES (@name1, @quantity1),
                        (@name2, @quantity2), (@name3, @quantity3);";
                    command.Parameters.AddWithValue("@name1", "banana");
                    command.Parameters.AddWithValue("@quantity1", 150);
                    command.Parameters.AddWithValue("@name2", "orange");
                    command.Parameters.AddWithValue("@quantity2", 154);
                    command.Parameters.AddWithValue("@name3", "apple");
                    command.Parameters.AddWithValue("@quantity3", 100);

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows inserted={0}", rowCount));
                }

                // connection will be closed by the 'using' block
                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)


## <a name="step-2-read-data"></a>Шаг 2. Чтение данных

Используйте указанный ниже код с инструкцией SQL `SELECT` для подключения и чтения данных. В коде используется класс `MySqlConnection` со следующими методами:
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync), чтобы установить соединение с MySQL.
- [CreateCommand ()](/dotnet/api/system.data.common.dbconnection.createcommand), чтобы задать свойство CommandText.
- [ExecuteReaderAsync()](/dotnet/api/system.data.common.dbcommand.executereaderasync), чтобы выполнять команды базы данных. 
- [ReadAsync()](/dotnet/api/system.data.common.dbdatareader.readasync#System_Data_Common_DbDataReader_ReadAsync), чтобы перейти к записям в результирующем наборе. Затем используются метод GetInt32 и метод GetString для анализа значений в записи.


Замените параметры `Server`, `Database`, `UserID` и `Password` значениями, указанными при создании сервера и базы данных. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlRead
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "SELECT * FROM inventory;";

                    using (var reader = await command.ExecuteReaderAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            Console.WriteLine(string.Format(
                                "Reading from table=({0}, {1}, {2})",
                                reader.GetInt32(0),
                                reader.GetString(1),
                                reader.GetInt32(2)));
                        }
                    }
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="step-3-update-data"></a>Шаг 3. Обновление данных
Используйте указанный ниже код с инструкцией SQL `UPDATE` для подключения и чтения данных. В коде используется класс `MySqlConnection` со следующими методами:
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync), чтобы установить соединение с MySQL; 
- [CreateCommand()](/dotnet/api/system.data.common.dbconnection.createcommand), чтобы задать свойство CommandText;
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync), чтобы выполнять команды базы данных. 


Замените параметры `Server`, `Database`, `UserID` и `Password` значениями, указанными при создании сервера и базы данных. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlUpdate
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "UPDATE inventory SET quantity = @quantity WHERE name = @name;";
                    command.Parameters.AddWithValue("@quantity", 200);
                    command.Parameters.AddWithValue("@name", "banana");

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows updated={0}", rowCount));
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```
[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="step-4-delete-data"></a>Шаг 4. Удаление данных
Используйте указанный ниже код с инструкцией SQL `DELETE` для подключения и удаления данных. 

В коде используется класс `MySqlConnection` со следующими методами:
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync), чтобы установить соединение с MySQL;
- [CreateCommand ()](/dotnet/api/system.data.common.dbconnection.createcommand), чтобы задать свойство CommandText;
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync), чтобы выполнять команды базы данных. 


Замените параметры `Server`, `Database`, `UserID` и `Password` значениями, указанными при создании сервера и базы данных. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlDelete
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "DELETE FROM inventory WHERE name = @name;";
                    command.Parameters.AddWithValue("@name", "orange");

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows deleted={0}", rowCount));
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для MySQL" через портал](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для MySQL" с помощью CLI](./how-to-manage-single-server-cli.md)

[Не можете найти нужную информацию? Сообщите нам об этом.](https://aka.ms/mysql-doc-feedback)
