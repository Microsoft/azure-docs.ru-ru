---
title: Подключение с помощью sqlcmd
description: Используйте программу командной строки sqlcmd для подключения к выделенному пулу SQL и выполнения запросов к нему в Azure синапсе Analytics.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f8b4d54585bc70c3ee5f24846e216f75e985cf84
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675733"
---
# <a name="connect-to-a-dedicated-sql-pool-in-azure-synapse-analytics-with-sqlcmd"></a>Подключение к выделенному пулу SQL в Azure синапсе Analytics с помощью программы sqlcmd

> [!div class="op_single_selector"]
>
> * [Power BI](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect)
> * [Машинное обучение Azure](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md)
> * [SSMS](sql-data-warehouse-query-ssms.md)

Используйте программу командной строки [sqlcmd] [sqlcmd] для подключения к выделенному пулу SQL и запроса к нему.  

## <a name="1-connect"></a>1. Подключение

Чтобы приступить к работе с [sqlcmd] [sqlcmd], откройте командную строку и введите **sqlcmd** , после чего строка подключения для выделенного пула SQL. В строке подключения обязательно укажите следующие параметры.

* **Server (-S):** сервер в формате `<`имя_сервера`>`.database.windows.net
* **База данных (-d):** выделенное имя пула SQL.
* **Включить заключенные в кавычки идентификаторы (-I):** Для подключения к выделенному экземпляру пула SQL необходимо включить заключенные в кавычки идентификаторы.

Чтобы использовать проверку подлинности SQL Server, необходимо добавить параметры имени пользователя и пароля.

* **User (-U):** пользователь сервера в формате `<`Пользователь`>`.
* **Пароль (-P):** Пароль, связанный с пользователем.

Например, строка подключения может выглядеть так:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

Чтобы использовать встроенную проверку подлинности Azure Active Directory, необходимо добавить параметры Azure Active Directory.

* **Проверки подлинности Azure Active Directory (-G):** — использовать Azure Active Directory для проверки подлинности.

Например, строка подключения может выглядеть так:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> Необходимо [включить проверку подлинности Azure Active Directory](sql-data-warehouse-authentication.md) , чтобы выполнять проверку подлинности с помощью Active Directory.

## <a name="2-query"></a>2. Запрос

После подключения можно подавать любые поддерживаемые инструкции Transact-SQL для экземпляра.  В этом примере запросы отправляются в интерактивном режиме.

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

В следующих примерах показано, как выполнить запросы в пакетном режиме, используя параметр -Q или передав SQL программе sqlcmd.

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о параметрах, доступных в программе sqlcmd, см. в [документации по sqlcmd](/sql/tools/sqlcmd-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).