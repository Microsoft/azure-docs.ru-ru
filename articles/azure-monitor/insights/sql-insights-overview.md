---
title: Мониторинг развертываний SQL с помощью SQL Insights (Предварительная версия)
description: Обзор SQL Insights в Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/15/2021
ms.openlocfilehash: d01f80a803c5b0f9da067dd23ab8cdb4cc591a79
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104610121"
---
# <a name="monitor-your-sql-deployments-with-sql-insights-preview"></a>Мониторинг развертываний SQL с помощью SQL Insights (Предварительная версия)
SQL Insights отслеживает производительность и работоспособность развертываний SQL.  Это помогает обеспечить предсказуемую производительность и доступность важных рабочих нагрузок, построенных на базе серверной части SQL, выявляя узкие места и проблемы производительности. SQL Insights хранит свои данные в [журналах Azure Monitor](../logs/data-platform-logs.md), что позволяет ему обеспечивать эффективную агрегирование и фильтрацию, а также анализировать тенденции данных с течением времени. Эти данные можно просмотреть Azure Monitor в представлениях, поставляемых в рамках этого предложения, и вы можете углубиться непосредственно в данные журнала для выполнения запросов и анализа тенденций.

SQL Insights не устанавливает ничего в развертываниях IaaS SQL. Вместо этого он использует выделенные виртуальные машины мониторинга для удаленного сбора данных для развертываний SQL PaaS и IaaS SQL.  Профиль мониторинга SQL Insights позволяет управлять наборами данных, которые должны быть собраны на основе типа SQL, в том числе базы данных SQL Azure, Управляемый экземпляр Azure SQL и SQL Server, работающего на виртуальной машине Azure.

## <a name="pricing"></a>Цены

Нет прямой стоимости для SQL Insights, но за ее действие взимается плата в Log Analytics рабочей области. В соответствии с ценами, опубликованными на [странице цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/), счета SQL Insights выставляются за:

- Данные, полученные от агентов и хранимые в рабочей области.
- Правила генерации оповещений на основе данных журнала.
- Уведомления, отправленные из правил генерации оповещений.

Размер журнала зависит от длины строк собираемых данных и может увеличиваться с учетом объема активности базы данных. 

## <a name="supported-versions"></a>Поддерживаемые версии 
SQL Insights поддерживает следующие версии SQL Server:

- SQL Server 2012 и более поздних версий

SQL Insights поддерживает SQL Server, выполняющихся в следующих средах:

- База данных SQL Azure
- Управляемый экземпляр SQL Azure
- Виртуальные машины SQL Azure
- Виртуальные машины Azure

SQL Insights не поддерживает или не ограничивает поддержку следующих ограничений:

- SQL Server, выполняемые на виртуальных машинах за пределами Azure, сейчас не поддерживаются.
- Эластичные пулы базы данных SQL Azure. Ограниченная поддержка в общедоступной предварительной версии. Будет полностью поддерживаться в общедоступной доступности.  
- Бессерверные развертывания Azure SQL: как и активное географическое восстановление, текущие агенты мониторинга будут препятствовать переходу в спящий режим для бессерверного развертывания. Это может привести к увеличению ожидаемых затрат от бессерверных развертываний.  
- Доступные для чтения реплики: в настоящее время поддерживаются только типы развертывания с одной доступной для чтения вторичной конечной точкой (критически важный для бизнеса или масштабирование). Если в развернутых развертываниях поддерживаются именованные реплики, мы сможем поддерживать несколько доступных для чтения вторичных конечных точек для одной логической базы данных.  
- Azure Active Directory: в настоящее время для агента мониторинга поддерживаются только имена для входа SQL. Мы планируем поддерживать Azure Active Directory в предстоящем выпуске и не поддерживают проверку подлинности виртуальных машин SQL с помощью Active Directory на контроллере домена.  


## <a name="open-sql-insights"></a>Открыть SQL Insights
Откройте SQL Insights, выбрав **SQL (Предварительная версия)** в разделе **Insights** в меню **Azure Monitor** в портал Azure. Щелкните плитку, чтобы загрузить интерфейс для типа отслеживаемого объекта SQL.

:::image type="content" source="media/sql-insights/portal.png" alt-text="SQL Insights в портал Azure.":::


## <a name="enable-sql-insights"></a>Включить SQL Insights 
Подробное описание процедуры включения SQL Insights в дополнение к действиям по устранению неполадок см. в разделе [Включение SQL Insights](sql-insights-enable.md) .


## <a name="data-collected-by-sql-insights"></a>Данные, собираемые SQL Insights
В общедоступной предварительной версии SQL Insights поддерживает только удаленный метод мониторинга. Агент Telegraf не установлен на SQL Server. Он использует подключаемый модуль ввода SQL Server для Telegraf и использует три группы запросов для различных типов мониторов SQL: база данных SQL Azure, Управляемый экземпляр SQL Azure, SQL Server, работающий на виртуальной машине Azure. 

В следующих таблицах представлены следующие сводные данные.

- Имя запроса в подключаемом модуле SQLServer Telegraf
- Динамические управляемые представления, вызывающие запросы
- Пространство имен, в котором данные отображаются в таблице *инсигхстметрикс*
- Собираются ли данные по умолчанию
- Частота сбора данных по умолчанию
 
При создании профиля мониторинга можно изменить выполняемые запросы и частоту сбора данных. 

### <a name="azure-sql-db-data"></a>Данные базы данных SQL Azure 

| Имя запроса | DMV | Пространство имен | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|
| азуресклдбваитстатс |  sys.dm_db_wait_stats | sqlserver_azuredb_waitstats | Нет | Н/Д |
| азуресклдбресаурцестатс | sys.dm_db_resource_stats | sqlserver_azure_db_resource_stats | Да | 60 секунд |
| азуресклдбресаурцеговернанце | sys.dm_user_db_resource_governance | sqlserver_db_resource_governance | Да | 60 секунд |
| азуресклдбдатабасеио | sys.dm_io_virtual_file_stats<br>sys.database_files<br>tempdb.sys .database_files | sqlserver_database_io | Да | 60 секунд |
| азуресклдбсерверпропертиес | sys.dm_os_job_object<br>sys.database_files<br>представления. база<br>представления. [database_service_objectives] | sqlserver_server_properties | Да | 60 секунд |
| азуресклдбосваитстатс | sys.dm_os_wait_stats | sqlserver_waitstats | Да | 60 секунд |
| азуресклдбмемориклеркс | sys.dm_os_memory_clerks | sqlserver_memory_clerks | Да | 60 секунд |
| азуресклдбперформанцекаунтерс | sys.dm_os_performance_counters<br>sys.databases | sqlserver_performance | Да | 60 секунд |
| азуресклдбрекуестс | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | sqlserver_requests | Нет | Н/Д |
| азуресклдбсчедулерс | sys.dm_os_schedulers | sqlserver_schedulers | Нет | Н/Д  |

### <a name="azure-sql-managed-instance-data"></a>Данные управляемого экземпляра SQL Azure 

| Имя запроса | DMV | Пространство имен | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|
| азуресклмиресаурцестатс | sys.server_resource_stats | sqlserver_azure_db_resource_stats | Да | 60 секунд |
| азуресклмиресаурцеговернанце | sys.dm_instance_resource_governance | sqlserver_instance_resource_governance | Да | 60 секунд |
| азуресклмидатабасеио | sys.dm_io_virtual_file_stats<br>sys.master_files | sqlserver_database_io | Да | 60 секунд |
| азуресклмисерверпропертиес | sys.server_resource_stats | sqlserver_server_properties | Да | 60 секунд |
| азуресклмиосваитстатс | sys.dm_os_wait_stats | sqlserver_waitstats | Да | 60 секунд |
| азуресклмимемориклеркс | sys.dm_os_memory_clerks | sqlserver_memory_clerks | Да | 60 секунд |
| азуресклмиперформанцекаунтерс | sys.dm_os_performance_counters<br>sys.databases | sqlserver_performance | Да | 60 секунд |
| азуресклмирекуестс | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | sqlserver_requests | Нет | Н/Д |
| азуресклмисчедулерс | sys.dm_os_schedulers | sqlserver_schedulers | Нет | Н/Д |

### <a name="sql-server-data"></a>Данных SQL Server.

| Имя запроса | DMV | Пространство имен | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|
| склсерверперформанцекаунтерс | sys.dm_os_performance_counters | sqlserver_performance | Да | 60 секунд |
| склсерверваитстатскатегоризед | sys.dm_os_wait_stats | sqlserver_waitstats | Да | 60 секунд | 
| склсервердатабасеио | sys.dm_io_virtual_file_stats<br>sys.master_files | sqlserver_database_io | Да | 60 секунд |
| склсерверпропертиес | sys.dm_os_sys_info | sqlserver_server_properties | Да | 60 секунд |
| склсервермемориклеркс | sys.dm_os_memory_clerks | sqlserver_memory_clerks | Да | 60 секунд |
| склсерверсчедулерс | sys.dm_os_schedulers | sqlserver_schedulers | Нет | Н/Д |
| склсерверрекуестс | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | sqlserver_requests | Нет | Н/Д |
| склсерверволумеспаце | sys.master_files | sqlserver_volume_space | Да | 60 секунд |
| склсерверкпу | sys.dm_os_ring_buffers | sqlserver_cpu | Да | 60 секунд |
| склсервераваилабилитирепликастатес | sys.dm_hadr_availability_replica_states<br>sys.availability_replicas<br>sys.availability_groups<br>sys.dm_hadr_availability_group_states | sqlserver_hadr_replica_states | | 60 секунд |
| склсервердатабасерепликастатес | sys.dm_hadr_database_replica_states<br>sys.availability_replicas | sqlserver_hadr_dbreplica_states | | 60 секунд |




## <a name="next-steps"></a>Дальнейшие действия

Подробные инструкции по включению SQL Insights см. в разделе [Включение SQL Insights](sql-insights-enable.md) .
Ответы [на часто задаваемые вопросы](../faq.md#sql-insights-preview) о SQL Insights см. здесь.
