---
title: Настройка важности рабочей нагрузки для выделенного пула SQL
description: Узнайте, как задать уровень важности запроса в Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 05/15/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 38fb842cf90c110266f53b79a9ab2ef6157025b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98681291"
---
# <a name="configure-workload-importance-in-dedicated-sql-pool-for-azure-synapse-analytics"></a>Настройка важности рабочей нагрузки в выделенном пуле SQL для Azure синапсе Analytics

Настройка важности в выделенном пуле SQL для Azure синапсе позволяет повлиять на планирование запросов. Запросы с более высоким уровнем важности будут выполняться перед запросами с более низким уровнем важности. Чтобы назначить важность запросам, необходимо создать классификатор рабочей нагрузки.

## <a name="create-a-workload-classifier-with-importance"></a>Создание классификатора рабочих нагрузок с уровнем важности

Часто в сценариях хранилища данных в занятой системе имеются пользователи, которым необходимо быстро выполнить запросы.  Таким пользователем может быть руководитель компании, которому требуется создать отчет, или аналитик, выполняющий нерегламентированный запрос. Чтобы присвоить уровень важности, необходимо создать классификатор рабочей нагрузки и задать уровень важности для запроса.  В примерах ниже для создания двух классификаторов используется синтаксис [create workload classifier](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). `Membername` может быть одним пользователем или группой.  Чтобы найти существующих выделенных пользователей пула SQL, выполните:

```sql
Select name from sys.sysusers
```

Чтобы создать классификатор рабочей нагрузки для пользователя с более высоким уровнем важности, выполните следующую команду:

```sql
CREATE WORKLOAD CLASSIFIER ExecReportsClassifier
    WITH (WORKLOAD_GROUP = 'xlargerc'
         ,MEMBERNAME     = 'name' 
         ,IMPORTANCE     = above_normal);
```

Чтобы создать классификатор рабочей нагрузки для пользователя, выполняющего нерегламентированные запросы с более низким уровнем важности, выполните следующую команду:  

```sql
CREATE WORKLOAD CLASSIFIER AdhocClassifier
    WITH (WORKLOAD_GROUP = 'xlargerc'
         ,MEMBERNAME     = 'name' 
         ,IMPORTANCE     = below_normal);
```

## <a name="next-steps"></a>Next Steps

- Дополнительные сведения об управлении рабочей нагрузкой см. в разделе [Классификация рабочих нагрузок](sql-data-warehouse-workload-classification.md).
- Дополнительные сведения о важности см. в разделе [Уровень важности рабочей нагрузки](sql-data-warehouse-workload-importance.md).

> [!div class="nextstepaction"]
> [Перейти к статье "Администрирование и мониторинг уровня важности рабочей нагрузки"](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md)
