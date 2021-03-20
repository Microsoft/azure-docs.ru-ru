---
title: Управление важностью рабочих нагрузок и их мониторинг в выделенном пуле SQL
description: Узнайте, как управлять и отслеживать важность уровня запросов выделенного пула SQL для Azure синапсе Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 12b75ad3746cd0f54e27e474e0fd13bb0bba0e05
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98685248"
---
# <a name="manage-and-monitor-workload-importance-in-dedicated-sql-pool-for-azure-synapse-analytics"></a>Управление и мониторинг важности рабочей нагрузки в выделенном пуле SQL для Azure синапсе Analytics

Управляйте и контролируйте выделенную важность уровня запросов пула SQL в Azure синапсе с помощью динамических административных представлений и представления каталога.

## <a name="monitor-importance"></a>Мониторинг важности

Отслеживайте важность с помощью столбца New важность в динамическом административном представлении [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) .
В приведенном ниже запросе наблюдения показано время и время начала отправки запросов. Просмотрите время отправки и время начала, а также важность, чтобы увидеть, как важно планирование.

```sql
SELECT s.login_name, r.status, r.importance, r.submit_time, r.start_time
  FROM sys.dm_pdw_exec_sessions s
  JOIN sys.dm_pdw_exec_requests r ON s.session_id = r.session_id
  WHERE r.resource_class is not null
ORDER BY r.start_time
```

Чтобы просмотреть дальнейшие сведения о расписании запросов, используйте представления каталога.

## <a name="manage-importance-with-catalog-views"></a>Управление важностью с помощью представлений каталога

Представление каталога sys.workload_management_workload_classifiers содержит сведения о классификаторах. Чтобы исключить определенные системой классификаторы, которые сопоставляются с классами ресурсов, выполните следующий код:

```sql
SELECT *
  FROM sys.workload_management_workload_classifiers
  WHERE classifier_id > 12
```

Представление каталога [sys.workload_management_workload_classifier_details](/sql/relational-databases/system-catalog-views/sys-workload-management-workload-classifier-details-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)содержит сведения о параметрах, используемых при создании классификатора.  В приведенном ниже запросе показано, что Ексекрепортсклассифиер был создан в ```membername``` параметре для значений с ексекутиверепортс:

```sql
SELECT c.name,cd.classifier_type, classifier_value
  FROM sys.workload_management_workload_classifiers c
  JOIN sys.workload_management_workload_classifier_details cd
    ON cd.classifier_id = c.classifier_id
  WHERE c.name = 'ExecReportsClassifier'
```

![результаты запроса](./media/sql-data-warehouse-how-to-manage-and-monitor-workload-importance/wlm-query-results.png)

Чтобы упростить устранение неполадок классификации, рекомендуется удалить сопоставления ролей класса ресурсов при создании классификаторов рабочей нагрузки. Приведенный ниже код возвращает членство в роли существующего класса ресурсов. Запустите sp_droprolemember для каждого ```membername``` из соответствующих классов ресурсов.
Ниже приведен пример проверки существования перед удалением классификатора рабочей нагрузки.

```sql
IF EXISTS (SELECT 1 FROM sys.workload_management_workload_classifiers WHERE name = 'ExecReportsClassifier')
  DROP WORKLOAD CLASSIFIER ExecReportsClassifier;
GO
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о классификации см. в разделе [классификация рабочей нагрузки](sql-data-warehouse-workload-classification.md).
- Дополнительные сведения о важности см. в разделе [Уровень важности рабочей нагрузки](sql-data-warehouse-workload-importance.md).

> [!div class="nextstepaction"]
> [Перейдите к разделу Настройка важности рабочей нагрузки](sql-data-warehouse-how-to-configure-workload-importance.md)
