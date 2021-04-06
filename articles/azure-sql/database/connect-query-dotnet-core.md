---
title: Использование .NET Core для подключения и выполнения запроса к базе данных
description: Из этой статьи вы узнаете, как использовать .NET Core для создания программы, которая подключается к базе данных в службе "База данных SQL Azure" или Управляемому экземпляру SQL Azure и создает к ней запросы с помощью инструкций Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2, devx-track-csharp
ms.devlang: dotnet
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 1d25f43ef5a694d8b94710055bf1be72a7fcb45c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97705223"
---
# <a name="quickstart-use-net-core-c-to-query-a-database"></a>Краткое руководство. Использование .NET Core (C#) для создания запросов к базе данных
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi-asa.md)]

При работе с этим кратким руководством вы будете использовать [.NET Core](https://www.microsoft.com/net/) и код C# для подключения к базе данных. Затем вы выполните инструкцию Transact-SQL для запроса данных.

> [!TIP]
> Следующий модуль Microsoft Learn поможет вам научиться бесплатно [разрабатывать и настраивать приложение ASP.NET, которое запрашивает базу данных в службе "База данных SQL Azure"](/learn/modules/develop-app-that-queries-azure-sql/).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Установленный [.NET Core для вашей операционной системы](https://www.microsoft.com/net/core).
- База данных, в которой можно выполнять запрос. 

  [!INCLUDE[create-configure-database](../includes/create-configure-database.md)]
  
## <a name="create-a-new-net-core-project"></a>Создание проекта .NET Core

1. Откройте командную строку и создайте папку с именем **sqltest**. Перейдите в эту папку и выполните следующую команду.

    ```cmd
    dotnet new console
    ```

    Эта команда создает файлы проекта нового приложения, включая первоначальный файл кода C# (**Program.cs**), файл конфигурации XML (**sqltest.csproj**) и необходимые двоичные файлы.

2. В текстовом редакторе откройте **sqltest.csproj** и вставьте следующий XML-код между тегами `<Project>`. Этот XML-код добавляет `System.Data.SqlClient` как зависимость.

    ```xml
    <ItemGroup>
        <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
    </ItemGroup>
    ```

## <a name="insert-code-to-query-the-database-in-azure-sql-database"></a>Вставка кода для запроса к Базе данных SQL Azure

1. Откройте файл **Program.cs** в текстовом редакторе.

2. Замените содержимое следующим кодом и добавьте соответствующие значения для сервера, базы данных, имени пользователя и пароля.

> [!NOTE]
> Чтобы использовать строку подключения ADO.NET, замените 4 строки в коде, устанавливающем сервер, базу данных, имя пользователя и пароль, следующей строкой. В строке задайте имя пользователя и пароль.
>
>    `builder.ConnectionString="<your_ado_net_connection_string>";`

```csharp
using System;
using System.Data.SqlClient;
using System.Text;

namespace sqltest
{
    class Program
    {
        static void Main(string[] args)
        {
            try 
            { 
                SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();

                builder.DataSource = "<your_server.database.windows.net>"; 
                builder.UserID = "<your_username>";            
                builder.Password = "<your_password>";     
                builder.InitialCatalog = "<your_database>";
         
                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                {
                    Console.WriteLine("\nQuery data example:");
                    Console.WriteLine("=========================================\n");
                    
                    connection.Open();       

                    String sql = "SELECT name, collation_name FROM sys.databases";

                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        using (SqlDataReader reader = command.ExecuteReader())
                        {
                            while (reader.Read())
                            {
                                Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                            }
                        }
                    }                    
                }
            }
            catch (SqlException e)
            {
                Console.WriteLine(e.ToString());
            }
            Console.WriteLine("\nDone. Press enter.");
            Console.ReadLine(); 
        }
    }
}
```

## <a name="run-the-code"></a>Выполнение кода

1. Выполните следующие команды при выводе запроса.

   ```cmd
   dotnet restore
   dotnet run
   ```

2. Убедитесь, что строки возвращаются.

   ```text
   Query data example:
   =========================================

   master   SQL_Latin1_General_CP1_CI_AS
   tempdb   SQL_Latin1_General_CP1_CI_AS
   WideWorldImporters   Latin1_General_100_CI_AS

   Done. Press enter.
   ```

3. Нажмите клавишу **ВВОД**, чтобы закрыть окно приложения.

## <a name="next-steps"></a>Дальнейшие действия

- [Начало работы с .NET Core в Windows, Linux и Mac OS с помощью командной строки](/dotnet/core/tutorials/using-with-xplat-cli).
- Узнайте, как [подключиться и отправить запрос к Базе данных SQL Azure или Управляемому экземпляру SQL Azure с помощью .NET Framework и Visual Studio](connect-query-dotnet-visual-studio.md).  
- Узнайте, как [спроектировать первую базу данных с помощью SSMS](design-first-database-tutorial.md) или [спроектировать базу данных и выполнить подключение с помощью C# и ADO.NET](design-first-database-csharp-tutorial.md).
- Дополнительные сведения о .NET см. в [этой документации](/dotnet/).
