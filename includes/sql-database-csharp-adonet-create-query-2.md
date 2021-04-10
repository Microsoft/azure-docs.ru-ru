---
author: rothja
ms.author: jroth
ms.service: sql-database
ms.topic: include
ms.date: 12/10/2018
ms.openlocfilehash: 5509c640fbf232cb6c613d96c30cb0e8d9d10ec5
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551698"
---
## <a name="c-program-example"></a>Пример программы C#

В следующих разделах этой статьи представлена программа C#, в которой используется ADO.NET для отправки инструкций Transact-SQL (T-SQL) в Базу данных SQL. В программе C# демонстрируются следующие действия:

- [Подключение к Базе данных SQL с помощью ADO.NET](#cs_1_connect)
- [Методы, возвращающие инструкции T-SQL](#cs_2_return)
    - Создание таблиц
    - Заполнение таблиц данными
    - Обновление, удаление и выбор данных
- [Отправка T-SQL в базе данных](#cs_3_submit)

### <a name="entity-relationship-diagram-erd"></a>Схема отношения элементов (ERD)

Инструкции `CREATE TABLE` включают ключевое слово **REFERENCES** для создания отношения *внешнего ключа* (FK) между двумя таблицами. При использовании *tempdb* закомментируйте ключевое слово `--REFERENCES` с помощью пары тире в начале.

На схеме ERD представлено отношение между двумя таблицами. Значения в *дочернем* столбце **tabEmployee.DepartmentCode** ограничены значениями *родительского* столбца **tabDepartment.DepartmentCode**.

![Схема ERD с внешним ключом](./media/sql-database-csharp-adonet-create-query-2/erd-dept-empl-fky-2.png)

> [!NOTE]
> Вы можете отредактировать T-SQL, чтобы добавить начальные `#` к именам таблиц, чтобы они создавались как временные таблицы в *tempdb*. Это удобно при демонстрации, когда тестовая база данных недоступна. Все ссылки на внешние ключи не применяются во время их использования, а временные таблицы автоматически удаляются при закрытии подключения после завершения работы программы.

### <a name="to-compile-and-run"></a>Компиляция и запуск

Программа C# логически является одним CS-файлом. Она физически разделена на несколько блоков кода для удобства изучения каждого блока. Чтобы скомпилировать и запустить программу, выполните шаги ниже.

1. Создайте проект C# в Visual Studio. Необходимо выбрать тип проекта *Консоль*. Это можно сделать в разделе **Шаблоны** > **Visual C#**  > **Классическое приложение Windows** > **Консольное приложение (.NET Framework)** .

1. В файле *Program.cs* замените соответствующие начальные строки кода, сделав следующее:

    1. Скопируйте и вставьте следующие блоки кода в той же последовательности, в которой они представлены (дополнительные сведения см. в разделах, посвященных [подключению к базе данных](#cs_1_connect), [созданию T-SQL](#cs_2_return) и [отправке в базу данных](#cs_3_submit)).

    1. Измените следующие значения в методе `Main`:

        - *cb.DataSource*
        - *cb.UserID*
        - *cb.Password*
        - *cb.InitialCatalog*

1. Убедитесь, что на сборку *System.Data.dll* есть ссылка. Для этого разверните узел **Ссылки** на панели **обозревателя решений**.

1. Чтобы создать программу в Visual Studio и запустить ее, щелкните **Начало**. Выходные данные отчета отображаются в окне программы. Значения GUID могут отличаться для разных тестовых запусков.

    ```Output
    =================================
    T-SQL to 2 - Create-Tables...
    -1 = rows affected.

    =================================
    T-SQL to 3 - Inserts...
    8 = rows affected.

    =================================
    T-SQL to 4 - Update-Join...
    2 = rows affected.

    =================================
    T-SQL to 5 - Delete-Join...
    2 = rows affected.

    =================================
    Now, SelectEmployees (6)...
    8ddeb8f5-9584-4afe-b7ef-d6bdca02bd35 , Alison , 20 , acct , Accounting
    9ce11981-e674-42f7-928b-6cc004079b03 , Barbara , 17 , hres , Human Resources
    315f5230-ec94-4edd-9b1c-dd45fbb61ee7 , Carol , 22 , acct , Accounting
    fcf4840a-8be3-43f7-a319-52304bf0f48d , Elle , 15 , NULL , NULL
    View the report output here, then press any key to end the program...
    ```

<a name="cs_1_connect"></a>

### <a name="connect-to-sql-database-using-adonet"></a>Подключение к Базе данных SQL с помощью ADO.NET

```csharp
using System;
using System.Data.SqlClient;   // System.Data.dll
//using System.Data;           // For:  SqlDbType , ParameterDirection

namespace csharp_db_test
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                var cb = new SqlConnectionStringBuilder();
                cb.DataSource = "your_server.database.windows.net";
                cb.UserID = "your_user";
                cb.Password = "your_password";
                cb.InitialCatalog = "your_database";

                using (var connection = new SqlConnection(cb.ConnectionString))
                {
                    connection.Open();

                    Submit_Tsql_NonQuery(connection, "2 - Create-Tables", Build_2_Tsql_CreateTables());

                    Submit_Tsql_NonQuery(connection, "3 - Inserts", Build_3_Tsql_Inserts());

                    Submit_Tsql_NonQuery(connection, "4 - Update-Join", Build_4_Tsql_UpdateJoin(),
                        "@csharpParmDepartmentName", "Accounting");

                    Submit_Tsql_NonQuery(connection, "5 - Delete-Join", Build_5_Tsql_DeleteJoin(),
                        "@csharpParmDepartmentName", "Legal");

                    Submit_6_Tsql_SelectEmployees(connection);
                }
            }
            catch (SqlException e)
            {
                Console.WriteLine(e.ToString());
            }

            Console.WriteLine("View the report output here, then press any key to end the program...");
            Console.ReadKey();
        }
```

<a name="cs_2_return"></a>

### <a name="methods-that-return-t-sql-statements"></a>Методы, возвращающие инструкции T-SQL

```csharp
static string Build_2_Tsql_CreateTables()
{
    return @"
        DROP TABLE IF EXISTS tabEmployee;
        DROP TABLE IF EXISTS tabDepartment;  -- Drop parent table last.

        CREATE TABLE tabDepartment
        (
            DepartmentCode  nchar(4)          not null    PRIMARY KEY,
            DepartmentName  nvarchar(128)     not null
        );

        CREATE TABLE tabEmployee
        (
            EmployeeGuid    uniqueIdentifier  not null  default NewId()    PRIMARY KEY,
            EmployeeName    nvarchar(128)     not null,
            EmployeeLevel   int               not null,
            DepartmentCode  nchar(4)              null
            REFERENCES tabDepartment (DepartmentCode)  -- (REFERENCES would be disallowed on temporary tables.)
        );
    ";
}

static string Build_3_Tsql_Inserts()
{
    return @"
        -- The company has these departments.
        INSERT INTO tabDepartment (DepartmentCode, DepartmentName)
        VALUES
            ('acct', 'Accounting'),
            ('hres', 'Human Resources'),
            ('legl', 'Legal');

        -- The company has these employees, each in one department.
        INSERT INTO tabEmployee (EmployeeName, EmployeeLevel, DepartmentCode)
        VALUES
            ('Alison'  , 19, 'acct'),
            ('Barbara' , 17, 'hres'),
            ('Carol'   , 21, 'acct'),
            ('Deborah' , 24, 'legl'),
            ('Elle'    , 15, null);
    ";
}

static string Build_4_Tsql_UpdateJoin()
{
    return @"
        DECLARE @DName1  nvarchar(128) = @csharpParmDepartmentName;  --'Accounting';

        -- Promote everyone in one department (see @parm...).
        UPDATE empl
        SET
            empl.EmployeeLevel += 1
        FROM
            tabEmployee   as empl
        INNER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        WHERE
            dept.DepartmentName = @DName1;
    ";
}

static string Build_5_Tsql_DeleteJoin()
{
    return @"
        DECLARE @DName2  nvarchar(128);
        SET @DName2 = @csharpParmDepartmentName;  --'Legal';

        -- Right size the Legal department.
        DELETE empl
        FROM
            tabEmployee   as empl
        INNER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        WHERE
            dept.DepartmentName = @DName2

        -- Disband the Legal department.
        DELETE tabDepartment
            WHERE DepartmentName = @DName2;
    ";
}

static string Build_6_Tsql_SelectEmployees()
{
    return @"
        -- Look at all the final Employees.
        SELECT
            empl.EmployeeGuid,
            empl.EmployeeName,
            empl.EmployeeLevel,
            empl.DepartmentCode,
            dept.DepartmentName
        FROM
            tabEmployee   as empl
        LEFT OUTER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        ORDER BY
            EmployeeName;
    ";
}
```

<a name="cs_3_submit"></a>

### <a name="submit-t-sql-to-the-database"></a>Отправка T-SQL в базе данных

```csharp
static void Submit_6_Tsql_SelectEmployees(SqlConnection connection)
{
    Console.WriteLine();
    Console.WriteLine("=================================");
    Console.WriteLine("Now, SelectEmployees (6)...");

    string tsql = Build_6_Tsql_SelectEmployees();

    using (var command = new SqlCommand(tsql, connection))
    {
        using (SqlDataReader reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine("{0} , {1} , {2} , {3} , {4}",
                    reader.GetGuid(0),
                    reader.GetString(1),
                    reader.GetInt32(2),
                    (reader.IsDBNull(3)) ? "NULL" : reader.GetString(3),
                    (reader.IsDBNull(4)) ? "NULL" : reader.GetString(4));
            }
        }
    }
}

static void Submit_Tsql_NonQuery(
    SqlConnection connection,
    string tsqlPurpose,
    string tsqlSourceCode,
    string parameterName = null,
    string parameterValue = null
    )
{
    Console.WriteLine();
    Console.WriteLine("=================================");
    Console.WriteLine("T-SQL to {0}...", tsqlPurpose);

    using (var command = new SqlCommand(tsqlSourceCode, connection))
    {
        if (parameterName != null)
        {
            command.Parameters.AddWithValue(  // Or, use SqlParameter class.
                parameterName,
                parameterValue);
        }
        int rowsAffected = command.ExecuteNonQuery();
        Console.WriteLine(rowsAffected + " = rows affected.");
    }
}
} // EndOfClass
}
```