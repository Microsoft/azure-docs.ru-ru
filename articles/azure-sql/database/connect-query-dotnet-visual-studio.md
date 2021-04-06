---
title: Использование Visual Studio с .NET и C# для отправки запросов
description: Использование Visual Studio для создания приложения C#, которое подключается к базе данных в службе "База данных SQL Azure" или Управляемом экземпляре SQL Azure, и отправляет запросы.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: devx-track-csharp, sqldbrb=2
ms.devlang: dotnet
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/10/2020
ms.openlocfilehash: 1d8859f4790610e72ad517f74bbbbf0cf77d9316
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97705213"
---
# <a name="quickstart-use-net-and-c-in-visual-studio-to-connect-to-and-query-a-database"></a>Краткое руководство. Использование .NET и C# в Visual Studio для подключения и создания запросов к базе данных
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi-asa.md)]

В этом кратком руководстве показано, как использовать [.NET Framework](https://www.microsoft.com/net/) и код C# в Visual Studio для отправки запросов к базе данных в SQL Azure или Synapse SQL с помощью инструкций Transact-SQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- [Visual Studio 2019](https://www.visualstudio.com/downloads/) — выпуск Community, Professional или Enterprise.
- База данных, в которой можно выполнять запрос.

  [!INCLUDE[create-configure-database](../includes/create-configure-database.md)]

## <a name="create-code-to-query-the-database-in-azure-sql-database"></a>Создание кода для запроса к базе данных в службе "База данных SQL Azure"

1. В Visual Studio создайте новый проект. 
   
1. В диалоговом окне **Новый проект** выберите **Visual C#** , а затем — **Консольное приложение (.NET Framework)** .
   
1. Введите название проекта *sqltest*, а затем нажмите кнопку **ОК**. Создается новый проект. 
   
1. Выберите **Проект** > **Управление пакетами NuGet**. 
   
1. В **диспетчере пакетов NuGet** выберите вкладку **Обзор**, а затем найдите и выберите **Microsoft.Data.SqlClient**.
   
1. На странице **Microsoft.Data.SqlClient** щелкните **Установить**. 
   - При выводе запроса нажмите кнопку **ОК**, чтобы продолжить установку. 
   - Если откроется окно **Прием условий лицензионного соглашения**, выберите **Я принимаю**.
   
1. После завершения установки вы можете закрыть **диспетчер пакетов NuGet**. 
   
1. В редакторе кода замените содержимое **Program.cs** следующим кодом. Замените значения `<your_server>`, `<your_username>`, `<your_password>` и `<your_database>`.
   
   ```csharp
   using System;
   using Microsoft.Data.SqlClient;
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
                   builder.DataSource = "<your_server>.database.windows.net"; 
                   builder.UserID = "<your_username>";            
                   builder.Password = "<your_password>";     
                   builder.InitialCatalog = "<your_database>";
   
                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
                       
                       String sql = "SELECT name, collation_name FROM sys.databases";
   
                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           connection.Open();
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
               Console.ReadLine();
           }
       }
   }
   ```

## <a name="run-the-code"></a>Выполнение кода

1. Чтобы запустить приложение, выберите **Отладка** > **Начать отладку** или **Запустить** на панели инструментов либо нажмите клавишу **F5**.
1. Убедитесь, что возвращены имена баз данных и их параметры сортировки, и закройте окно приложения.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [подключиться и отправить запрос к базе данных в службе "База данных SQL Azure" с помощью .NET Core](connect-query-dotnet-core.md) в Windows, Linux и macOS.  
- См. дополнительные сведения о [начале работы с .NET Core в Windows, Linux и Mac OS с помощью командной строки](/dotnet/core/tutorials/using-with-xplat-cli).
- См. дополнительные сведения в разделах [Руководство по разработке первой базы данных в службе "База данных SQL Azure" с помощью SSMS](design-first-database-tutorial.md) или [Руководство по разработке первой базы данных в службе "База данных SQL Azure" с помощью .NET](design-first-database-csharp-tutorial.md).
- Дополнительные сведения о .NET см. в [этой документации](/dotnet/).
- Пример логики повтора: [Выполнение устойчивого подключения к Azure SQL с помощью ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n].


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net
