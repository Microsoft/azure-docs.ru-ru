---
title: Использование соединителя Spark с Microsoft Azure SQL и SQL Server
description: Узнайте, как использовать соединитель Spark с базой данных SQL Azure, Управляемый экземпляр Azure SQL и SQL Server.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: denzilribeiro
ms.author: denzilr
ms.reviewer: sstein
ms.date: 09/02/2020
ms.openlocfilehash: 2e01e74f5086f7f1eb7e85661fbd35f452d8dae8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92790208"
---
# <a name="accelerate-real-time-big-data-analytics-using-the-spark-connector"></a>Ускорение анализа больших данных в реальном времени с помощью соединителя Spark
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

> [!NOTE]
> По отношению к Sep 2020 этот соединитель активно не поддерживается. Однако теперь доступен [соединитель Apache Spark для SQL Server и SQL Azure](/sql/connect/spark/connector) , благодаря поддержке привязок Python и R, проще использовать интерфейс для выполнения операций вставки данных и много других улучшений. Мы настоятельно рекомендуем вам оценить и использовать новый соединитель вместо этого. Сведения о старом соединителе (Эта страница) сохраняется только в целях архивирования.

Соединитель Spark позволяет базам данных SQL Azure, Azure SQL Управляемый экземпляр и SQL Server действовать в качестве источника входных данных или приемника выходных данных для заданий Spark. Она позволяет использовать данные о транзакциях в реальном времени в аналитике больших данных и сохранять результаты для нерегламентированных запросов или отчетов. По сравнению со встроенным соединителем JDBC этот соединитель обеспечивает возможность выполнения операций вставки данных в базу данных. Он может более эффективны вставку строк с помощью 10 раз в 10x более высокую производительность. Соединитель Spark поддерживает проверку подлинности Azure Active Directory (Azure AD) для подключения к базе данных SQL Azure и Управляемый экземпляр SQL Azure, что позволяет подключать базу данных из Azure Databricks с помощью учетной записи Azure AD. Она предоставляет аналогичные интерфейсы со встроенным соединителем JDBC. Имеющиеся задания Spark можно легко перенести, чтобы использовать этот соединитель.

## <a name="download-and-build-a-spark-connector"></a>Скачайте и создайте соединитель Spark.

Репозиторий GitHub для старого соединителя, ранее связанного с этой страницей, не поддерживается активно. Вместо этого мы настоятельно рекомендуем вам оценить и использовать [новый соединитель](https://github.com/microsoft/sql-spark-connector).

### <a name="official-supported-versions"></a>Официальные Поддерживаемые версии

| Компонент                             | Версия                  |
| :-----------------------------------  | :----------------------- |
| Apache Spark                          | 2.0.2 или более поздние версии           |
| Scala                                 | 2.10 или более поздние версии            |
| Microsoft JDBC Driver для SQL Server  | 6.2 или более поздние версии             |
| Microsoft SQL Server                  | SQL Server 2008 или более поздняя версия |
| База данных SQL Azure                    | Поддерживается                |
| Управляемый экземпляр SQL Azure            | Поддерживается                |

Соединитель Spark использует драйвер Microsoft JDBC для SQL Server для перемещения данных между рабочими узлами Spark и базами данных:

Поток данных выглядит следующим образом:

1. Главный узел Spark подключается к базам данных SQL или SQL Server и загружает данные из определенной таблицы или с помощью определенного SQL-запроса.
2. Главный узел Spark распределяет данные в рабочие узлы для преобразования.
3. Рабочий узел соединяется с базами данных, которые подключаются к базе данных SQL и SQL Server и записывает данные в базу данных. Пользователь может использовать вставку по строкам или массовую вставку.

На схеме ниже показан поток данных.

   ![На схеме показан описанный поток, а главный узел соединяется непосредственно с базой данных и подключается к трем рабочим узлам, которые подключаются к базе данных.](./media/spark-connector/architecture.png)

### <a name="build-the-spark-connector"></a>Создание соединителя Spark

Сейчас в проекте соединителя используется maven. Чтобы создать соединитель без зависимостей, выполните следующую команду:

- mvn clean package;
- скачать последние версии JAR-файла из папки release;
- Включить JAR-файл базы данных SQL Spark

## <a name="connect-and-read-data-using-the-spark-connector"></a>Подключение и чтение данных с помощью соединителя Spark

Для чтения или записи данных можно подключаться к базам данных SQL и SQL Server из задания Spark. Можно также выполнить запрос DML или DDL в базах данных SQL и SQL Server.

### <a name="read-data-from-azure-sql-and-sql-server"></a>Чтение данных из Azure SQL и SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "dbTable"        -> "dbo.Clients",
  "user"           -> "username",
  "password"       -> "*********",
  "connectTimeout" -> "5", //seconds
  "queryTimeout"   -> "5"  //seconds
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="read-data-from-azure-sql-and-sql-server-with-specified-sql-query"></a>Чтение данных из SQL Azure и SQL Server с указанным запросом SQL

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "queryCustom"  -> "SELECT TOP 100 * FROM dbo.Clients WHERE PostalCode = 98074" //Sql query
  "user"         -> "username",
  "password"     -> "*********",
))

//Read all data in table dbo.Clients
val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="write-data-to-azure-sql-and-sql-server"></a>Запись данных в Azure SQL и SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

// Aquire a DataFrame collection (val collection)

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "dbTable"      -> "dbo.Clients",
  "user"         -> "username",
  "password"     -> "*********"
))

import org.apache.spark.sql.SaveMode
collection.write.mode(SaveMode.Append).sqlDB(config)
```

### <a name="run-dml-or-ddl-query-in-azure-sql-and-sql-server"></a>Выполнение запроса DML или DDL в SQL Azure и SQL Server

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.query._
val query = """
              |UPDATE Customers
              |SET ContactName = 'Alfred Schmidt', City = 'Frankfurt'
              |WHERE CustomerID = 1;
            """.stripMargin

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "user"         -> "username",
  "password"     -> "*********",
  "queryCustom"  -> query
))

sqlContext.sqlDBQuery(config)
```

## <a name="connect-from-spark-using-azure-ad-authentication"></a>Подключение из Spark с помощью аутентификации Azure AD

Вы можете подключиться к базе данных SQL Azure и Управляемый экземпляр SQL с помощью аутентификации Azure AD. Проверка подлинности AAD используется для централизованного управления удостоверениями пользователей базы данных и в качестве альтернативы проверке подлинности SQL Server.

### <a name="connecting-using-activedirectorypassword-authentication-mode"></a>Подключение с использованием режима аутентификации ActiveDirectoryPassword

#### <a name="setup-requirement"></a>Требование к установке

При использовании режима аутентификации ActiveDirectoryPassword необходимо скачать [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java) и его зависимости, а затем включить их в путь сборки Java.

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "user"           -> "username",
  "password"       -> "*********",
  "authentication" -> "ActiveDirectoryPassword",
  "encrypt"        -> "true"
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

### <a name="connecting-using-an-access-token"></a>Подключение с помощью маркера доступа

#### <a name="setup-requirement"></a>Требование к установке

При использовании режима аутентификации на основе маркера доступа необходимо скачать [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java) и его зависимости, а затем включить их в путь сборки Java.

Сведения о получении маркера доступа к базе данных в базе данных SQL Azure или Azure SQL Управляемый экземпляр см. в статье [использование проверки](authentication-aad-overview.md) подлинности Azure Active Directory.

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"                   -> "mysqlserver.database.windows.net",
  "databaseName"          -> "MyDatabase",
  "accessToken"           -> "access_token",
  "hostNameInCertificate" -> "*.database.windows.net",
  "encrypt"               -> "true"
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```

## <a name="write-data-using-bulk-insert"></a>Запись данных с помощью инструкции INSERT

Традиционный соединитель JDBC записывает данные в базу данных, используя вставку по строкам. Соединитель Spark можно использовать для записи данных в SQL Azure и SQL Server с помощью инструкции INSERT. Это значительно улучшает производительность записи при загрузке больших наборов данных или загрузке данных в таблицы, в которых используется индекс хранилища столбцов.

```scala
import com.microsoft.azure.sqldb.spark.bulkcopy.BulkCopyMetadata
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

/**
  Add column Metadata.
  If not specified, metadata is automatically added
  from the destination table, which may suffer performance.
*/
var bulkCopyMetadata = new BulkCopyMetadata
bulkCopyMetadata.addColumnMetadata(1, "Title", java.sql.Types.NVARCHAR, 128, 0)
bulkCopyMetadata.addColumnMetadata(2, "FirstName", java.sql.Types.NVARCHAR, 50, 0)
bulkCopyMetadata.addColumnMetadata(3, "LastName", java.sql.Types.NVARCHAR, 50, 0)

val bulkCopyConfig = Config(Map(
  "url"               -> "mysqlserver.database.windows.net",
  "databaseName"      -> "MyDatabase",
  "user"              -> "username",
  "password"          -> "*********",
  "dbTable"           -> "dbo.Clients",
  "bulkCopyBatchSize" -> "2500",
  "bulkCopyTableLock" -> "true",
  "bulkCopyTimeout"   -> "600"
))

df.bulkCopyToSqlDB(bulkCopyConfig, bulkCopyMetadata)
//df.bulkCopyToSqlDB(bulkCopyConfig) if no metadata is specified.
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы этого еще не сделали, скачайте соединитель Spark из [репозитория Azure-sqldb-Spark GitHub](https://github.com/Azure/azure-sqldb-spark) и изучите дополнительные ресурсы в репозитории:

- [Примеры записных книжек Azure Databricks](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/notebooks).
- [Пример сценариев (Scala)](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/scripts)

Дополнительные сведения см. также в [руководстве по SQL, таблицам и наборам данных Apache Spark](https://spark.apache.org/docs/latest/sql-programming-guide.html) и в [документации по Azure Databricks](/azure/azure-databricks/).