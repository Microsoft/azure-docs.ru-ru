---
title: Краткое руководство. Создание классификатора рабочих нагрузок с помощью T-SQL
description: Использование T-SQL для создания классификатора рабочей нагрузки с высокой важностью.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: e757c8047bf6d634ab6d7cbc8963087c0eccc46a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98677374"
---
# <a name="quickstart-create-a-workload-classifier-using-t-sql"></a>Краткое руководство. Создание рабочей нагрузки классификатора с помощью T-SQL

Благодаря этому краткому руководству вы быстро создадите классификатор рабочей нагрузки с высоким уровнем важности для руководителей своей организации. Этот классификатор рабочей нагрузки позволит запросам руководителей иметь приоритет над другими запросами с меньшей важностью в очереди.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

> [!NOTE]
> Создание экземпляра выделенного пула SQL в Azure Synapse Analytics может повлечь дополнительные расходы.  Дополнительные сведения см. на странице [цен на Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/sql-data-warehouse/).
>
>

## <a name="prerequisites"></a>Предварительные требования

В этом кратком руководстве предполагается, что вы подготовили к работе выделенный пул SQL в Azure Synapse Analytics, а также что у вас есть права доступа CONTROL DATABASE. Если его требуется создать, используйте инструкции из статьи [Краткое руководство. Создание выделенного пула SQL с помощью портала Azure](create-data-warehouse-portal.md), чтобы создать выделенный пул SQL **mySampleDataWarehouse**.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-login-for-theceo"></a>Создание имени для входа TheCEO

Создайте имя для входа с проверкой подлинности SQL Server в базу данных `master`, используя функцию [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для TheCEO.

```sql
IF NOT EXISTS (SELECT * FROM sys.sql_logins WHERE name = 'TheCEO')
BEGIN
CREATE LOGIN [TheCEO] WITH PASSWORD='<strongpassword>'
END
;
```

## <a name="create-user"></a>Создать пользователя

[Создайте пользователя](/sql/t-sql/statements/create-user-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) TheCEO в mySampleDataWarehouse.

```sql
IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = 'THECEO')
BEGIN
CREATE USER [TheCEO] FOR LOGIN [TheCEO]
END
;
```

## <a name="create-a-workload-classifier"></a>Создание классификатора рабочих нагрузок

Для пользователя TheCEO создайте [классификатор рабочей нагрузки](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), задав его параметру относительной важности запроса IMPORTANCE значение HIGH.

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO];
CREATE WORKLOAD CLASSIFIER [wgcTheCEO]
WITH (WORKLOAD_GROUP = 'xlargerc'
      ,MEMBERNAME = 'TheCEO'
      ,IMPORTANCE = HIGH);
```

## <a name="view-existing-classifiers"></a>Просмотр существующих классификаторов

```sql
SELECT * FROM sys.workload_management_workload_classifiers
```

## <a name="clean-up-resources"></a>Очистка ресурсов

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO]
DROP USER [TheCEO]
;
```

Плата взимается за единицы хранилища данных и данные, хранящиеся в выделенном пуле SQL. Плата за вычислительные ресурсы и ресурсы хранилища взимается отдельно.

- Если вы хотите сохранить данные в хранилище, то можете приостановить работу вычислительных ресурсов, когда не используете выделенный пул SQL. При приостановке вычислений плата взимается только за хранение данных. Когда вы будете готовы работать с данными, возобновите вычисление.
- Если вы хотите исключить будущие расходы, то можете удалить выделенный пул SQL.

Выполните следующие действия, чтобы очистить ресурсы.

1. Войдите на [портал Azure](https://portal.azure.com) и выберите выделенный пул SQL.

    ![Очистка ресурсов](./media/load-data-from-azure-blob-storage-using-polybase/clean-up-resources.png)

2. Чтобы приостановить вычисление, нажмите кнопку **Пауза**. Если работа выделенного пула SQL приостановлена, вы увидите кнопку **Запуск**.  Чтобы возобновить вычисление, нажмите кнопку **Пуск**.

3. Чтобы удалить выделенный пул SQL во избежание дальнейших платежей за вычисление или хранение, нажмите кнопку **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

- Теперь вы создали классификатор рабочей нагрузки. Выполните несколько запросов как TheCEO, чтобы проверить, как они работают. Запросы и назначенную важность см. в разделе [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).
- Дополнительные сведения об управлении рабочей нагрузкой выделенного пула SQL см. в статьях [Важность рабочих нагрузок](sql-data-warehouse-workload-importance.md) и [Классификация рабочих нагрузок](sql-data-warehouse-workload-classification.md).
- См. статьи с инструкциями по [настройке уровня важности рабочих нагрузок](sql-data-warehouse-how-to-configure-workload-importance.md), а также по [администрированию и мониторингу рабочих нагрузок](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md).
