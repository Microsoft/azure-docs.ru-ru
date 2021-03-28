---
title: Загрузка данных из CSV-файла в базу данных (BCP)
description: Для импорта небольших объемов данных в базу данных SQL Azure используйте программу bcp.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 09ae46ec6455b6998bcf4da5648d2ceaef4d5b19
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105644789"
---
# <a name="load-data-from-csv-into-azure-sql-database-or-sql-managed-instance-flat-files"></a>Загрузка данных из CSV-файла в базу данных SQL Azure или Управляемый экземпляр SQL (неструктурированные файлы)
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Вы можете использовать программу командной строки bcp для импорта данных из CSV-файла в базу данных SQL Azure или Azure SQL Управляемый экземпляр.

## <a name="before-you-begin"></a>Подготовка

### <a name="prerequisites"></a>Предварительные условия

Для выполнения задач из этой статьи необходимо следующее:

* База данных в базе данных SQL Azure
* установленная служебная программа командной строки bcp;
* установленная служебная программа командной строки sqlcmd.

Вы можете скачать служебные программы bcp и sqlcmd из [документации по Microsoft sqlcmd](/sql/tools/sqlcmd-utility?view=sql-server-ver15&preserve-view=true).

### <a name="data-in-ascii-or-utf-16-format"></a>Данные в формате ASCII или UTF-16

Чтобы выполнить действия, описанные в этом руководстве, необходимо использовать данные в формате ASCII или UTF-16, так как bcp не поддерживает кодировку UTF-8.

## <a name="1-create-a-destination-table"></a>1. Создание целевой таблицы

Определите таблицу в базе данных SQL как целевую таблицу. Столбцы в таблице должны соответствовать данным в каждой строке файла данных.

Чтобы создать таблицу, откройте окно командной строки и используйте sqlcmd.exe для выполнения следующей команды:

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
    CREATE TABLE DimDate2
    (
        DateId INT NOT NULL,
        CalendarQuarter TINYINT NOT NULL,
        FiscalQuarter TINYINT NOT NULL
    )
    ;
"
```

## <a name="2-create-a-source-data-file"></a>2. Создание файла исходных данных

Откройте блокнот и скопируйте следующие строки данных в новый текстовый файл, а затем сохраните этот файл в локальный временный каталог (C:\Temp\DimDate2.txt). Эти данные имеют формат ASCII.

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```

Чтобы экспортировать данные из базы данных SQL Server, откройте окно командной строки и выполните команду ниже (необязательно). Замените значения TableName, ServerName, DatabaseName, Username и Password на собственные.

```bcp
bcp <TableName> out C:\Temp\DimDate2_export.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <Password> -q -c -t ,
```

## <a name="3-load-the-data"></a>3. Загрузка данных

Чтобы загрузить данные, откройте окно командной строки и выполните следующую команду, подставив собственные значения имени сервера, базы данных, пользователя и пароль.

```bcp
bcp DimDate2 in C:\Temp\DimDate2.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <password> -q -c -t  ,
```

Используйте команду ниже, чтобы убедиться, что данные загружены правильно.

```bcp
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "SELECT * FROM DimDate2 ORDER BY 1;"
```

Результат должен выглядеть следующим образом:

| DateId | CalendarQuarter | FiscalQuarter |
| --- | --- | --- |
| 20150101 |1 |3 |
| 20150201 |1 |3 |
| 20150301 |1 |3 |
| 20150401 |2 |4 |
| 20150501 |2 |4 |
| 20150601 |2 |4 |
| 20150701 |3 |1 |
| 20150801 |3 |1 |
| 20150801 |3 |1 |
| 20151001 |4 |2 |
| 20151101 |4 |2 |
| 20151201 |4 |2 |

## <a name="next-steps"></a>Дальнейшие шаги

Миграция базы данных SQL Server описана в статье [Миграция базы данных SQL Server в базу данных SQL в облаке](database/migrate-to-database-from-sql-server.md).

<!--MSDN references-->
[bcp]: /sql/tools/bcp-utility
[CREATE TABLE syntax]: /sql/t-sql/statements/create-table-azure-sql-data-warehouse

<!--Other Web references-->
[Microsoft Download Center]: https://www.microsoft.com/download/details.aspx?id=36433
