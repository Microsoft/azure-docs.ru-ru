---
title: Изоляция рабочей нагрузки
description: Руководство по настройке изоляции рабочей нагрузки с помощью групп рабочей нагрузки в Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 506aed16f1b8a6c631a759bb1367aef8242859ac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98734786"
---
# <a name="azure-synapse-analytics-workload-group-isolation"></a>Изоляция группы рабочей нагрузки в Azure Synapse Analytics

В этой статье объясняется, как можно использовать группы рабочей нагрузки для настройки изоляции рабочей нагрузки, хранения ресурсов и применения правил среды выполнения для выполнения запросов.

## <a name="workload-groups"></a>Группы рабочей нагрузки

Группы рабочей нагрузки являются контейнерами для набора запросов и служат основой для настройки управления рабочими нагрузками в системе, включая изоляцию рабочей нагрузки.  Группы рабочей нагрузки создаются с помощью синтаксиса [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Управлять загрузкой данных и пользовательскими запросами можно с помощью простой конфигурации управления рабочей нагрузкой.  Например, группа рабочей нагрузки с именем `wgDataLoads` будет определять аспекты рабочей нагрузки для данных, загружаемых в систему. Кроме того, группа рабочей нагрузки с именем `wgUserQueries` определяет аспекты рабочей нагрузки для пользователей, выполняющих запросы для чтения данных из системы.

В следующих разделах будет показано, как группы рабочей нагрузки позволяют настраивать изоляцию и автономность, запрашивать определение ресурса и соблюдать правила выполнения.

## <a name="workload-isolation"></a>Изоляция рабочей нагрузки

Изоляция рабочей нагрузки означает, что ресурсы зарезервированы исключительно для группы рабочей нагрузки.  Изоляция рабочей нагрузки достигается путем установки для параметра MIN_PERCENTAGE_RESOURCE значения больше нуля в синтаксисе [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Для рабочих нагрузок непрерывного выполнения, которые должны соответствовать жестким требованиям соглашений об уровне обслуживания, изоляция гарантирует, что ресурсы всегда будут доступны для группы рабочих нагрузок.

Настройка изоляции рабочей нагрузки неявно определяет гарантированный уровень параллелизма. Например, группа рабочей нагрузки с заданными значениями `MIN_PERCENTAGE_RESOURCE` 30 % и `REQUEST_MIN_RESOURCE_GRANT_PERCENT` 2 % гарантирует уровень параллелизма — 15.  Уровень параллелизма гарантирован, так как 15–2 % слотов ресурсов зарезервированы в группе рабочей нагрузки на постоянной основе (независимо от того, как настроен параметр `REQUEST_*MAX*_RESOURCE_GRANT_PERCENT`).  Если `REQUEST_MAX_RESOURCE_GRANT_PERCENT` больше, чем `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, а `CAP_PERCENTAGE_RESOURCE` больше, чем `MIN_PERCENTAGE_RESOURCE`, то на каждый запрос добавляются дополнительные ресурсы.  Если `REQUEST_MAX_RESOURCE_GRANT_PERCENT` и `REQUEST_MIN_RESOURCE_GRANT_PERCENT` равны, а `CAP_PERCENTAGE_RESOURCE` больше `MIN_PERCENTAGE_RESOURCE`, то возможен дополнительный параллелизм.  Рассмотрим приведенный ниже метод для определения гарантированного параллелизма.

[Гарантированный параллелизм] = [`MIN_PERCENTAGE_RESOURCE`]/[`REQUEST_MIN_RESOURCE_GRANT_PERCENT`]

> [!NOTE]
> Для min_percentage_resource существуют определенные минимально приемлемые значения уровня обслуживания.  Дополнительные сведения см. в разделе об [эффективных значениях](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json?view=azure-sqldw-latest&preserve-view=true#effective-values).

В случае отсутствия изоляции рабочей нагрузки запросы работают в [общем пуле](#shared-pool-resources) ресурсов.  Доступ к ресурсам в общем пуле не гарантирован и назначается в зависимости от степени [важности](sql-data-warehouse-workload-importance.md).

Настройка изоляции рабочей нагрузки должна выполняться с осторожностью, так как ресурсы распределяются по группе рабочей нагрузки, даже если в группе рабочей нагрузки нет активных запросов. Чрезмерная настройка изоляции может привести к снижению общей загрузки системы.

Пользователи должны избегать решения для управления рабочей нагрузкой, которое настраивает 100 % изоляцию рабочей нагрузки: 100 % изоляция достигается, когда сумма min_percentage_resource, настроенная для всех групп рабочей нагрузки, равна 100 %.  Этот тип конфигурации является чрезмерно строгим и жестким, оставляя мало места для запросов ресурсов, которые случайно неправильно классифицируются. Существует возможность, позволяющая выполнять один запрос из групп рабочей нагрузки, не настроенных для изоляции. Ресурсы, выделенные на этот запрос, будут отображаться как ноль в динамическом административном представлении системы и занимать уровень ресурсов smallrc, предоставленных из системных зарезервированных ресурсов.

> [!NOTE]
> Чтобы обеспечить оптимальное использование ресурсов, рассмотрим решение для управления рабочей нагрузкой, которое использует некоторую изоляцию, чтобы обеспечить соблюдение соглашений об уровне обслуживания в комбинации с использованием общих ресурсов, доступ к которым осуществляется на основе [важности рабочей нагрузки](sql-data-warehouse-workload-importance.md).

## <a name="workload-containment"></a>Автономность рабочей нагрузки

Автономность рабочей нагрузки означает ограничение объема ресурсов, которые может использовать группа рабочей нагрузки.  Автономность рабочей нагрузки достигается путем установки для параметра CAP_PERCENTAGE_RESOURCE значения меньше 100 в синтаксисе [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Рассмотрим сценарий, в котором пользователям требуется доступ на чтение к системе, чтобы они могли выполнять анализ "что-если" с помощью нерегламентированных запросов.  Эти типы запросов могут оказать негативное влияние на другие рабочие нагрузки, выполняемые в системе.  Настройка автономности гарантирует, что количество ресурсов ограничено.

Настройка автономности рабочей нагрузки неявно определяет максимальный уровень параллелизма.  Если для CAP_PERCENTAGE_RESOURCE задано значение 60 %, а для REQUEST_MIN_RESOURCE_GRANT_PERCENT значение 1 %, для группы рабочей нагрузки разрешен параллелизм до уровня не более 60.  Рассмотрим приведенный ниже метод, чтобы определить максимальную степень параллелизма:

[Максимальный параллелизм] = [`CAP_PERCENTAGE_RESOURCE`] / [`REQUEST_MIN_RESOURCE_GRANT_PERCENT`]

> [!NOTE]
> Эффективное значение параметра CAP_PERCENTAGE_RESOURCE группы рабочей нагрузки не достигает 100 %, когда создаются группы рабочей нагрузки с параметром MIN_PERCENTAGE_RESOURCE на уровне больше нуля.  Действующие значения времени выполнения см. в статье [sys.dm_workload_management_workload_groups_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-workload-management-workload-group-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="resources-per-request-definition"></a>Ресурсы для определения запроса

Группы рабочей нагрузки предоставляют механизм для определения минимального и максимального объема ресурсов, выделяемых для каждого запроса с помощью параметров REQUEST_MIN_RESOURCE_GRANT_PERCENT и REQUEST_MAX_RESOURCE_GRANT_PERCENT в синтаксисе [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  В этом случае ресурсами являются ЦП и память.  Настройка этих значений определяет, сколько ресурсов и какой уровень параллелизма может быть достигнут в системе.

> [!NOTE]
> REQUEST_MAX_RESOURCE_GRANT_PERCENT — это необязательный параметр, по умолчанию имеющий то же значение, что и параметр REQUEST_MIN_RESOURCE_GRANT_PERCENT.

Как и при выборе класса ресурсов, настройка параметра REQUEST_MIN_RESOURCE_GRANT_PERCENT задает значение для ресурсов, используемых запросом.  Объем ресурсов, указанных заданным значением, гарантированно выделен для запроса, прежде чем он начнет выполнение.  Если вы клиент, выполняющий миграцию из классов ресурсов в группы рабочей нагрузки, ознакомьтесь с [этой](sql-data-warehouse-how-to-convert-resource-classes-workload-groups.md) статьей, чтобы сопоставить классы ресурсов с группами рабочей нагрузки в качестве отправной точки.

При настройке параметра REQUEST_MAX_RESOURCE_GRANT_PERCENT значение больше REQUEST_MIN_RESOURCE_GRANT_PERCENT позволяет системе выделить больше ресурсов на запрос.  При планировании запроса система определяет фактическое выделение ресурсов для запроса, со значением от REQUEST_MIN_RESOURCE_GRANT_PERCENT до REQUEST_MAX_RESOURCE_GRANT_PERCENT, в зависимости от доступности ресурсов в общем пуле и текущей нагрузки в системе.  Ресурсы должны существовать в [общем пуле](#shared-pool-resources) ресурсов при планировании запроса.  

> [!NOTE]
> Параметры REQUEST_MIN_RESOURCE_GRANT_PERCENT и REQUEST_MAX_RESOURCE_GRANT_PERCENT имеют действующие значения, зависящие от действующих значений параметров MIN_PERCENTAGE_RESOURCE и CAP_PERCENTAGE_RESOURCE.  Действующие значения времени выполнения см. в статье [sys.dm_workload_management_workload_groups_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-workload-management-workload-group-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="execution-rules"></a>Правила выполнения

В системах с нерегламентированной системой отчетности клиенты могут случайно выполнять неконтролируемые запросы, которые серьезно влияют на производительность других пользователей.  Системные администраторы вынуждены тратить время на прерывание запросов, чтобы освободить системные ресурсы.  Группы рабочей нагрузки предоставляют возможность настройки правила времени ожидания выполнения запроса для отмены запросов, превышающих указанное значение.  Правило настраивается с помощью параметра `QUERY_EXECUTION_TIMEOUT_SEC` в синтаксисе [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="shared-pool-resources"></a>Ресурсы общего пула

Ресурсы общего пула — это ресурсы, которые не настроены для изоляции.  Группы рабочей нагрузки с параметром MIN_PERCENTAGE_RESOURCE, для которых задано нулевое значение, используют ресурсы в общем пуле для выполнения запросов.  Группы рабочей нагрузки со значением параметра CAP_PERCENTAGE_RESOURCE больше значения параметра MIN_PERCENTAGE_RESOURCE также используют общие ресурсы.  Объем ресурсов, доступных в общем пуле, вычисляется следующим образом.

[Общий пул] = 100 — [сумма `MIN_PERCENTAGE_RESOURCE` во всех группах рабочей нагрузки]

Доступ к ресурсам в общем пуле размещается по уровню [важности](sql-data-warehouse-workload-importance.md).  Запросы с одинаковым уровнем важности будут получать доступ к ресурсам общего пула в порядке поступления.

## <a name="next-steps"></a>Дальнейшие действия

- [Краткое руководство. Настройка изоляции рабочих нагрузок с помощью T-SQL](quickstart-configure-workload-isolation-tsql.md)
- [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Преобразование классов ресурсов в группы рабочей нагрузки](sql-data-warehouse-how-to-convert-resource-classes-workload-groups.md)
- [Azure Synapse Analytics — мониторинг портала управления рабочими нагрузками](sql-data-warehouse-workload-management-portal-monitor.md)  
