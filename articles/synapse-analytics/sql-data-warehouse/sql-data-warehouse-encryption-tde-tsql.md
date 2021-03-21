---
title: Прозрачное шифрование данных (T-SQL)
description: Прозрачное шифрование данных (TDE) в Azure синапсе Analytics (T-SQL)
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/30/2019
ms.author: jrasnick
ms.reviewer: rortloff
ms.custom: seo-lt-2019
ms.openlocfilehash: 80e5da6bb281806afe6bc980e35d70732bcd609c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98676278"
---
# <a name="get-started-with-transparent-data-encryption-tde"></a>Начало работы с прозрачным шифрованием данных (TDE)

> [!div class="op_single_selector"]
>
> * [Обзор безопасности](sql-data-warehouse-overview-manage-security.md)
> * [Аутентификация](sql-data-warehouse-authentication.md)
> * [Шифрование (портал)](sql-data-warehouse-encryption-tde.md)
> * [Шифрование (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)

## <a name="required-permissions"></a>Необходимые разрешения

Чтобы включить прозрачное шифрование данных, необходимо иметь права администратора или участника роли dbmanager.

## <a name="enabling-encryption"></a>Включение шифрования

Чтобы включить TDE, выполните следующие действия.

1. Подключитесь к *главной* базе данных на сервере, где находится база данных, указав логин администратора или члена роли **dbmanager** в главной базе данных.
2. Выполните следующий оператор для шифрования базы данных:

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;
```

## <a name="disabling-encryption"></a>Отключение шифрования

Чтобы отключить TDE, выполните следующие действия.

1. Подключитесь к *главной* базе данных, указав логин администратора или члена роли **dbmanager** в главной базе данных.
2. Выполните следующий оператор для шифрования базы данных:

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION OFF;
```

> [!NOTE]
> Приостановка выделенного пула SQL должна быть возобновлена перед внесением изменений в параметры TDE.

## <a name="verifying-encryption"></a>Проверка шифрования

Чтобы проверить состояние шифрования, выполните следующие действия:

1. Подключитесь к *главной* базе данных или к базе данных экземпляра, указав логин администратора или члена роли **dbmanager** в главной базе данных.
2. Выполните следующий оператор для шифрования базы данных:

```sql
SELECT
    [name],
    [is_encrypted]
FROM
    sys.databases;
```

Результат ```1``` означает зашифрованную, а ```0``` — незашифрованную базу данных.

## <a name="encryption-dmvs"></a>Динамические административные представления шифрования

* [sys.databases](/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
* [sys.dm_pdw_nodes_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
