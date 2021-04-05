---
title: Краткое руководство. Настройка изоляции рабочих нагрузок с помощью T-SQL
description: Настройка изоляции рабочих нагрузок с помощью T-SQL.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 04/27/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 9d69c1708e73ad7ce5610a0683835e9f304c3f54
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98679760"
---
# <a name="quickstart-configure-workload-isolation-in-a-dedicated-sql-pool-using-t-sql"></a>Краткое руководство. Настройка изоляции рабочей нагрузки в выделенном пуле SQL с помощью T-SQL

В этом кратком руководстве вы быстро создадите группу рабочей нагрузки и классификатор для резервирования ресурсов для загрузки данных. Группа рабочей нагрузки будет выделять 20 % системных ресурсов для загрузки данных.  Классификатор рабочей нагрузки будет назначать запросы к данным, загружаемым группой рабочей нагрузки.  Благодаря изоляции загрузки данных на уровне 20 % использование ресурсов будет соответствовать действующим соглашениям об уровне обслуживания.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

> [!NOTE]
> Создание экземпляра SQL Synapse в Azure Synapse Analytics может повлечь дополнительные расходы.  Дополнительные сведения см. на странице [цен на Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/sql-data-warehouse/).

## <a name="prerequisites"></a>Предварительные требования

В этом кратком руководстве предполагается, что у вас уже есть экземпляр Synapse SQL в Azure Synapse и права доступа к системе управления базой данных. Если его требуется создать, используйте инструкции из статьи [Краткое руководство. Создание Хранилища данных SQL Azure на портале Azure и выполнение запроса к нему](create-data-warehouse-portal.md), чтобы создать выделенный пул SQL **mySampleDataWarehouse**.

## <a name="create-login-for-dataloads"></a>Создание имени для входа для DataLoads

Создайте имя для входа с аутентификацией SQL Server в базе данных `master`, используя оператор [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для ELTLogin.

```sql
IF NOT EXISTS (SELECT * FROM sys.sql_logins WHERE name = 'ELTLogin')
BEGIN
CREATE LOGIN [ELTLogin] WITH PASSWORD='<strongpassword>'
END
;
```

## <a name="create-user"></a>Создать пользователя

[Создайте пользователя](/sql/t-sql/statements/create-user-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) ELTLogin в mySampleDataWarehouse.

```sql
IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = 'ELTLogin')
BEGIN
CREATE USER [ELTLogin] FOR LOGIN [ELTLogin]
END
;
```

## <a name="create-a-workload-group"></a>Создание группы рабочей нагрузки

Создайте [группу рабочей нагрузки](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для DataLoads с уровнем изоляции в 20 %.

```sql
CREATE WORKLOAD GROUP DataLoads
WITH ( MIN_PERCENTAGE_RESOURCE = 20
      ,CAP_PERCENTAGE_RESOURCE = 100
      ,REQUEST_MIN_RESOURCE_GRANT_PERCENT = 5)
;
```

## <a name="create-a-workload-classifier"></a>Создание классификатора рабочих нагрузок

Создайте [классификатор рабочей нагрузки](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), чтобы связать ELTLogin с группой рабочей нагрузки DataLoads.

```sql
CREATE WORKLOAD CLASSIFIER [wgcELTLogin]
WITH (WORKLOAD_GROUP = 'DataLoads'
      ,MEMBERNAME = 'ELTLogin')
;
```

## <a name="view-existing-workload-groups-and-classifiers-and-run-time-values"></a>Просмотр имеющихся групп рабочей нагрузки, классификаторов и значений во время выполнения

```sql
--Workload groups
SELECT * FROM
sys.workload_management_workload_groups

--Workload classifiers
SELECT * FROM
sys.workload_management_workload_classifiers

--Run-time values
SELECT * FROM
sys.dm_workload_management_workload_groups_stats
```

## <a name="clean-up-resources"></a>Очистка ресурсов

```sql
DROP WORKLOAD CLASSIFIER [wgcELTLogin]
DROP WORKLOAD GROUP [DataLoads]
DROP USER [ELTLogin]
;
```

Плата взимается за единицы хранилища данных и данные, хранящиеся в выделенном пуле SQL. Плата за вычислительные ресурсы и ресурсы хранилища взимается отдельно.

- Если вы хотите сохранить данные в хранилище, то можете приостановить работу вычислительных ресурсов, когда не используете выделенный пул SQL. При приостановке вычислений плата взимается только за хранение данных. Когда вы будете готовы работать с данными, возобновите вычисление.
- Если вы хотите исключить будущие расходы, то можете удалить выделенный пул SQL.

## <a name="next-steps"></a>Дальнейшие действия

- Вы создали группу рабочей нагрузки. Теперь выполните несколько запросов как ELTLogin, чтобы проверить, как они работают. Запросы и назначенную группу рабочей нагрузки можно просмотреть в представлении [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).
- Дополнительные сведения об управлении рабочими нагрузками SQL Synapse см. в статьях [Что такое управление рабочей нагрузкой?](sql-data-warehouse-workload-management.md) и [Изоляция группы рабочей нагрузки Azure Synapse Analytics (предварительная версия)](sql-data-warehouse-workload-isolation.md).
