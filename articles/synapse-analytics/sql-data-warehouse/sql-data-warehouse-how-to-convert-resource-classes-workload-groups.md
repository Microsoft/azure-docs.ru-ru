---
title: Преобразование класса ресурсов в группу рабочей нагрузки
description: Узнайте, как создать группу рабочей нагрузки, похожую на класс ресурсов в выделенном пуле SQL.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 08/13/2020
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 1207f4856882d8aa0e6d1e41712071536bfecf29
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98728562"
---
# <a name="convert-resource-classes-to-workload-groups"></a>Преобразование классов ресурсов в группы рабочей нагрузки

Группы рабочей нагрузки предоставляют механизм изоляции и хранения системных ресурсов.  Кроме того, группы рабочей нагрузки позволяют задавать правила выполнения для выполняемых в них запросов.  Правило выполнения запроса с превышением времени ожидания позволяет отменять неконтролируемые запросы без вмешательства пользователя.  В этой статье объясняется, как на основе существующего класса ресурсов создать группу рабочей нагрузки с аналогичной конфигурацией.  Кроме того, мы добавим необязательное правило времени ожидания запроса.

> [!NOTE]
> Рекомендации по параллельному использованию групп рабочей нагрузки и классов ресурсов см. в разделе [Смешивание назначений классов ресурсов с помощью классификаторов](sql-data-warehouse-workload-classification.md#mixing-resource-class-assignments-with-classifiers) в статье [Классификация рабочих нагрузок Azure Synapse Analytics](sql-data-warehouse-workload-classification.md).

## <a name="understanding-the-existing-resource-class-configuration"></a>Сведения о существующей конфигурации класса ресурсов

Группам рабочей нагрузки требуется параметр `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, который определяет долю общих системных ресурсов, выделяемых для каждого запроса.  Выделение ресурсов для [классов ресурсов](resource-classes-for-workload-management.md#what-are-resource-classes) выполняется путем выделения слотов параллелизма.  Чтобы определить значение для параметра `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, используйте динамическое административное представление sys.dm_workload_management_workload_groups_stats <link tbd>.  Например, приведенный ниже запрос возвращает значение, которое можно использовать для параметра `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, чтобы создать группу рабочей нагрузки, аналогичную staticrc40.

```sql
SELECT Request_min_resource_grant_percent = Effective_request_min_resource_grant_percent
  FROM sys.dm_workload_management_workload_groups_stats
  WHERE name = 'staticrc40'
```

> [!NOTE]
> Группы рабочей нагрузки распределяют системные ресурсы с учетом их общей доли.  

При этом при увеличении или уменьшении масштаба изменяется доля выделяемых ресурсов для статических классов ресурсов относительно общего доступного объема.  Например, класс staticrc40 на оборудовании DW1000c выделяет 19,2 % от общего объема системных ресурсов.  На оборудовании DW2000c он же выделяет 9,6 %.  Эта ситуация аналогично сценарию, когда вы хотите выполнить вертикальное масштабирование для обеспечения параллелизма в сравнении с выделением большего объема ресурсов на каждый запрос.

## <a name="create-workload-group"></a>Создание группы рабочей нагрузки

Зная `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, вы можете создать группу рабочей нагрузки с помощью синтаксиса CREATE WORKLOAD GROUP <link>.  При желании можно указать для `MIN_PERCENTAGE_RESOURCE` значение больше нуля, чтобы изолировать ресурсы для группы рабочей нагрузки.  Также можно указать для `CAP_PERCENTAGE_RESOURCE` значение меньше 100, чтобы ограничить объем ресурсов, потребляемый группой рабочей нагрузки.  

С помощью mediumrc в качестве основания для примера приведенный ниже код задает `MIN_PERCENTAGE_RESOURCE` для выделения 10% системных ресурсов `wgDataLoads` и гарантирует, что один запрос сможет выполняться все время.  Кроме того, `CAP_PERCENTAGE_RESOURCE` имеет значение 40% и ограничивает эту группу рабочей нагрузки четырьмя одновременными запросами.  Если задать для параметра `QUERY_EXECUTION_TIMEOUT_SEC` значение 3600, все выполняемые более одного часа запросы будут автоматически отменяться.

```sql
CREATE WORKLOAD GROUP wgDataLoads WITH  
( REQUEST_MIN_RESOURCE_GRANT_PERCENT = 10
 ,MIN_PERCENTAGE_RESOURCE = 10
 ,CAP_PERCENTAGE_RESOURCE = 40
 ,QUERY_EXECUTION_TIMEOUT_SEC = 3600)
```

## <a name="create-the-classifier"></a>Создание классификатора

Ранее сопоставление запросов с классами ресурсов выполнялось с помощью [sp_addrolemember](resource-classes-for-workload-management.md#change-a-users-resource-class).  Чтобы обеспечить те же функциональные возможности и сопоставить запросы с группами рабочей нагрузки, используйте синтаксис [CREATE WORKLOAD CLASSIFIER](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Использование sp_addrolemember позволяло сопоставлять ресурсы с запросами только на основе имени входа.  Классификатор предоставляет помимо имени входа дополнительные параметры, например:
    - label
    - сеанс
    - время. Следующий пример направляет запросы от имени входа `AdfLogin` с меткой [OPTION LABEL](sql-data-warehouse-develop-label.md) и значением `factloads`, в ранее созданную группу рабочей нагрузки `wgDataLoads`.

```sql
CREATE WORKLOAD CLASSIFIER wcDataLoads WITH  
( WORKLOAD_GROUP = 'wgDataLoads'
 ,MEMBERNAME = 'AdfLogin'
 ,WLM_LABEL = 'factloads')
```

## <a name="test-with-a-sample-query"></a>Тестирование с использованием примера запроса

Ниже приведены примеры обычного запроса и запроса динамического административного представления, которые позволяют проверить правильность настройки группы рабочей нагрузки и классификатора.

```sql
SELECT SUSER_SNAME() --should be 'AdfLogin'

--change to a valid table AdfLogin has access to
SELECT TOP 10 *
  FROM nation
  OPTION (label='factloads')

SELECT request_id, [label], classifier_name, group_name, command
  FROM sys.dm_pdw_exec_requests
  WHERE [label] = 'factloads'
  ORDER BY submit_time DESC
```

## <a name="next-steps"></a>Дальнейшие действия

- [Изоляция рабочей нагрузки](sql-data-warehouse-workload-isolation.md)
- [Создание группы рабочей нагрузки](quickstart-configure-workload-isolation-tsql.md)
- [Классификатор CREATE WORKLOAD (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql??view=azure-sqldw-latest&preserve-view=true)
- [CREATE WORKLOAD GROUP (Transact-SQL)](/sql/t-sql/statements/create-workload-group-transact-sql?view=azure-sqldw-latest&preserve-view=true)
