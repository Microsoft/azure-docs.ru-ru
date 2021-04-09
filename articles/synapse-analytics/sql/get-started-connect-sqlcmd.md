---
title: Подключение к Synapse SQL с помощью sqlcmd
description: Используйте служебную программу командной строки sqlcmd для подключения и отправки запроса к бессерверному и выделенному пулам SQL.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 7d6675f9584f90b67d8520091dcd4b04dd89e462
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101667582"
---
# <a name="connect-to-synapse-sql-with-sqlcmd"></a>Подключение к Synapse SQL с помощью sqlcmd

> [!div class="op_single_selector"]
> * [Azure Data Studio](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md)
> * [SSMS](get-started-ssms.md)

В Synapse SQL для подключения и отправки запроса к бессерверному и выделенному пулам SQL можно использовать служебную программу командной строки [sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true).  

## <a name="1-connect"></a>1. Подключение
Чтобы начать использовать [sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true), откройте командную строку и введите **sqlcmd** и строку подключения к базе данных Synapse SQL. В строке подключения обязательно укажите следующие параметры.

* **Server (-S):** сервер в формате `<`имя_сервера`>`.database.windows.net
* **Database (-d):** Имя базы данных
* **Enable Quoted Identifiers (-I):** для подключения к экземпляру Synapse SQL следует включить заключенные в кавычки идентификаторы.

Чтобы использовать проверку подлинности SQL Server, необходимо добавить параметры имени пользователя и пароля.

* **User (-U):** пользователь сервера в формате `<`Пользователь`>`.
* **Password (-P):** пароль, связанный с пользователем

Строка подключения может выглядеть так:

**Бессерверный пул SQL**

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
```

**Выделенный пул SQL**

```
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

Чтобы использовать встроенную проверку подлинности Azure Active Directory, необходимо добавить параметры Azure Active Directory.

* **Проверки подлинности Azure Active Directory (-G):** — использовать Azure Active Directory для проверки подлинности.

Строка подключения может выглядеть так:

**Бессерверный пул SQL**

```
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -G -I
```

**Выделенный пул SQL**

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> Необходимо [включить проверку подлинности Azure Active Directory](../sql-data-warehouse/sql-data-warehouse-authentication.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) , чтобы выполнять проверку подлинности с помощью Active Directory.

## <a name="2-query"></a>2. Запрос

### <a name="use-dedicated-sql-pool"></a>Использование выделенного пула SQL

После подключения можно подавать любые поддерживаемые инструкции [Transact-SQL](/sql/t-sql/language-reference?view=azure-sqldw-latest&preserve-view=true) (T-SQL) для экземпляра. В этом примере запросы отправляются в интерактивном режиме.

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

В следующих примерах для выделенного пула SQL показано, как выполнять запросы в пакетном режиме, используя параметр -Q или передав SQL программе sqlcmd.

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

### <a name="use-serverless-sql-pool"></a>Использование бессерверного пула SQL

После подключения для экземпляра можно подавать любые поддерживаемые инструкции [Transact-SQL](/sql/t-sql/language-reference?view=azure-sqldw-latest&preserve-view=true) (T-SQL).  В следующем примере запросы отправляются в интерактивном режиме.

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
1> SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')
2> GO
3> QUIT
```

В приведенных ниже примерах для бессерверного пула SQL показано, как выполнять запросы в пакетном режиме, используя параметр -Q или передав SQL программе sqlcmd.

```sql
sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I -Q "SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')"
```

```sql
"SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')" | sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I > ./tables.out
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о параметрах sqlcmd см. в [документации по sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true).
