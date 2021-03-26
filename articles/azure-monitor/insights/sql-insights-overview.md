---
title: Мониторинг развертываний SQL с помощью SQL Insights (Предварительная версия)
description: Обзор SQL Insights в Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/15/2021
ms.openlocfilehash: d0bb5c55d3f7ba0573dfe9b511f4d31dcc64ed85
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105567837"
---
# <a name="monitor-your-sql-deployments-with-sql-insights-preview"></a>Мониторинг развертываний SQL с помощью SQL Insights (Предварительная версия)
SQL Insights — это комплексное решение для мониторинга любого продукта в [семействе SQL Azure](../../azure-sql/index.yml). SQL Insights использует [динамические административные представления](../../azure-sql/database/monitoring-with-dmvs.md) для предоставления данных, необходимых для мониторинга работоспособности, диагностики проблем и настройки производительности.  

SQL Insights выполняет все наблюдения удаленно. Агенты мониторинга на выделенных виртуальных машинах подключаются к ресурсам SQL и позволяют удаленно собирать данные. Собранные данные хранятся в [журналах Azure Monitor](../logs/data-platform-logs.md), что позволяет упростить агрегирование, фильтрацию и анализ тенденций. Собранные данные можно просмотреть в [шаблоне книги](../visualize/workbooks-overview.md)SQL Insights или непосредственно в данные с помощью [запросов журналов](../logs/get-started-queries.md).

## <a name="pricing"></a>Цены
Нет прямой стоимости для SQL Insights. Все расходы создаются виртуальными машинами, которые собирают данные, Log Analytics рабочими областями, в которых хранятся данные, и всеми правилами генерации оповещений, настроенными для данных. 

**Виртуальные машины**

Для виртуальных машин плата выставляются на основе цен, опубликованных на [странице цен на виртуальные машины](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/). Необходимое количество виртуальных машин зависит от количества строк подключения, которые необходимо отслеживать. Рекомендуется выделить 1 виртуальную машину размера Standard_B2s для каждой строки подключения 100. Дополнительные сведения см. в статье [требования к виртуальной машине Azure](sql-insights-enable.md#azure-virtual-machine-requirements) .

**Рабочие области Log Analytics**

Для рабочих областей Log Analytics плата выставляются на основе цен, опубликованных на [странице цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). Log Analytics рабочие области, используемые службой SQL Insights, потребуют затрат на прием данных, хранение данных и (необязательно) экспорт данных. Точная плата будет зависеть от объема принимаемых, сохраненных и экспортируемых данных. Объем этих данных впоследствии будет изменяться в зависимости от активности базы данных и параметров коллекции, определенных в [профилях мониторинга](sql-insights-enable.md#create-sql-monitoring-profile).

**Правила оповещения**

Для правил генерации оповещений в Azure Monitor плата выставляются на основе цен, опубликованных на [странице цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). Если вы решили [создать оповещения с помощью SQL Insights](sql-insights-alerts.md), вам будет выставляться плата за все созданные правила генерации оповещений, а также любые отправленные уведомления.

## <a name="supported-versions"></a>Поддерживаемые версии 
SQL Insights поддерживает следующие версии SQL Server:
- SQL Server 2012 и более поздних версий

SQL Insights поддерживает SQL Server, выполняющихся в следующих средах:
- База данных SQL Azure
- Управляемый экземпляр SQL Azure
- SQL Server на виртуальных машинах Azure (SQL Server, работающих на виртуальных машинах, зарегистрированных с помощью поставщика [виртуальных машин SQL](../../azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md) )
- Виртуальные машины Azure (SQL Server, работающие на виртуальных машинах, не зарегистрированных в поставщике [виртуальных машин SQL](../../azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md) )

SQL Insights не поддерживает или не ограничивает поддержку следующих ограничений:
- **Экземпляры, не являющиеся экземплярами Azure**: SQL Server, выполняющиеся на виртуальных машинах за пределами Azure, не поддерживаются.
- **Эластичные пулы базы данных SQL Azure**. метрики не могут быть собраны для эластичных пулов. Метрики не могут быть собраны для баз данных в эластичных пулах.
- **Низкий уровень служб базы данных SQL Azure**: метрики не могут быть собраны для баз данных на [уровнях служб](../../azure-sql/database/resource-limits-dtu-single-databases.md) Basic, S0, S1 и S2
- **Уровень бессерверной базы данных SQL Azure**. метрики можно собирать для баз данных с использованием бессерверного уровня вычислений. Однако процесс сбора метрик приведет к сбросу таймера задержки при автоматической приостановке, предотвращая переход базы данных в режим автоматической остановки.
- **Вторичные реплики**. метрики можно собирать только для одной вторичной реплики на базу данных. Если база данных содержит более 1 вторичной реплики, мониторинг может осуществляться только 1.
- Проверка подлинности **с помощью Azure Active Directory**. единственным поддерживаемым методом [проверки](../../azure-sql/database/logins-create-manage.md#authentication-and-authorization) подлинности для мониторинга является проверка подлинности SQL. Для SQL Server на виртуальной машине Azure проверка подлинности с помощью Active Directory на настраиваемом контроллере домена не поддерживается.  

## <a name="open-sql-insights"></a>Открыть SQL Insights
Откройте SQL Insights, выбрав **SQL (Предварительная версия)** в разделе **Insights** в меню **Azure Monitor** в портал Azure. Щелкните плитку, чтобы загрузить интерфейс для типа отслеживаемого объекта SQL.

:::image type="content" source="media/sql-insights/portal.png" alt-text="SQL Insights в портал Azure.":::

## <a name="enable-sql-insights"></a>Включить SQL Insights 
Инструкции по включению SQL Insights см. в разделе [Включение SQL Insights](sql-insights-enable.md) .

## <a name="troubleshoot-sql-insights"></a>Устранение неполадок SQL Insights 
Инструкции по устранению неполадок SQL Insights см. в разделе [Устранение неполадок SQL Insights](sql-insights-troubleshoot.md) .

## <a name="data-collected-by-sql-insights"></a>Данные, собираемые SQL Insights
SQL Insights выполняет все наблюдения удаленно. На виртуальных машинах, на которых работает SQL Server, не устанавливаются агенты. 

SQL Insights использует выделенные виртуальные машины мониторинга для удаленного сбора данных из ресурсов SQL. На каждой виртуальной машине мониторинга будут установлены [агент Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/agents/azure-monitor-agent-overview) и расширение рабочей нагрузки (вли). Расширение вли включает в себя [Агент Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)с открытым исходным кодом. SQL Insights использует [правила сбора данных](https://docs.microsoft.com/azure/azure-monitor/agents/data-collection-rule-overview) для указания параметров сбора данных для [подключаемого модуля SQL Server](https://www.influxdata.com/integration/microsoft-sql-server/)Telegraf.

Для базы данных SQL Azure, Управляемый экземпляр Azure SQL и SQL Server доступны различные наборы данных. В приведенных ниже таблицах описаны доступные данные. При [создании профиля мониторинга](sql-insights-enable.md#create-sql-monitoring-profile)можно настроить сбор наборов данных и частоту сбора.

Приведенные ниже таблицы содержат следующие столбцы.
- **Понятное имя**: имя запроса, как показано на портал Azure при создании профиля мониторинга.
- **Имя конфигурации**: имя запроса, как показано на портал Azure при изменении профиля мониторинга.
- **Пространство имен**: имя запроса, найденное в log Analytics рабочей области. Этот идентификатор отображается в таблице **инсигхстметрикс** для `Namespace` свойства в `Tags` столбце.
- **DMV**: динамические управляемые представления, используемые для создания набора данных
- **Включено по умолчанию**: собираются ли данные по умолчанию
- **Частота сбора по умолчанию**: Периодичность сбора данных по умолчанию

### <a name="data-for-azure-sql-database"></a>Данные для базы данных SQL Azure 
| Понятное имя | Имя конфигурации | Пространство имен | Динамические административные представления | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|:---|
| Статистика ожидания базы данных | азуресклдбваитстатс | sqlserver_azuredb_waitstats | sys.dm_db_wait_stats | Нет | Н/Д |
| Статистика ожидания DBO | азуресклдбосваитстатс | sqlserver_waitstats |sys.dm_os_wait_stats | Да | 60 секунд |
| Клерки памяти | азуресклдбмемориклеркс | sqlserver_memory_clerks | sys.dm_os_memory_clerks | Да | 60 секунд |
| Ввод-вывод базы данных | азуресклдбдатабасеио | sqlserver_database_io | sys.dm_io_virtual_file_stats<br>sys.database_files<br>tempdb.sys .database_files | Да | 60 секунд |
| Свойства сервера | азуресклдбсерверпропертиес | sqlserver_server_properties | sys.dm_os_job_object<br>sys.database_files<br>представления. база<br>представления. [database_service_objectives] | Да | 60 секунд |
| Счетчики производительности | азуресклдбперформанцекаунтерс | sqlserver_performance | sys.dm_os_performance_counters<br>sys.databases | Да | 60 секунд |
| Статистика ресурсов | азуресклдбресаурцестатс | sqlserver_azure_db_resource_stats | sys.dm_db_resource_stats | Да | 60 секунд |
| Управление ресурсами | азуресклдбресаурцеговернанце | sqlserver_db_resource_governance | sys.dm_user_db_resource_governance | Да | 60 секунд |
| Requests | азуресклдбрекуестс | sqlserver_requests | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | Нет | Н/Д |
| Планировщики| азуресклдбсчедулерс | sqlserver_schedulers | sys.dm_os_schedulers | Нет | Н/Д  |

### <a name="data-for-azure-sql-managed-instance"></a>Данные для Управляемый экземпляр Azure SQL 

| Понятное имя | Имя конфигурации | Пространство имен | Динамические административные представления | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|:---|
| Статистика ожидания | азуресклмиосваитстатс | sqlserver_waitstats | sys.dm_os_wait_stats | Да | 60 секунд |
| Клерки памяти | азуресклмимемориклеркс | sqlserver_memory_clerks | sys.dm_os_memory_clerks | Да | 60 секунд |
| Ввод-вывод базы данных | азуресклмидатабасеио | sqlserver_database_io | sys.dm_io_virtual_file_stats<br>sys.master_files | Да | 60 секунд |
| Свойства сервера | азуресклмисерверпропертиес | sqlserver_server_properties | sys.server_resource_stats | Да | 60 секунд |
| Счетчики производительности | азуресклмиперформанцекаунтерс | sqlserver_performance | sys.dm_os_performance_counters<br>sys.databases| Да | 60 секунд |
| Статистика ресурсов | азуресклмиресаурцестатс | sqlserver_azure_db_resource_stats | sys.server_resource_stats | Да | 60 секунд |
| Управление ресурсами | азуресклмиресаурцеговернанце | sqlserver_instance_resource_governance | sys.dm_instance_resource_governance | Да | 60 секунд |
| Requests | азуресклмирекуестс | sqlserver_requests | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | Нет | Н/Д |
| Планировщики | азуресклмисчедулерс | sqlserver_schedulers | sys.dm_os_schedulers | Нет | Н/Д |

### <a name="data-for-sql-server"></a>Данные для SQL Server

| Понятное имя | Имя конфигурации | Пространство имен | Динамические административные представления | Включено по умолчанию | Частота сбора по умолчанию |
|:---|:---|:---|:---|:---|:---|
| Статистика ожидания | склсерверваитстатскатегоризед | sqlserver_waitstats | sys.dm_os_wait_stats | Да | 60 секунд | 
| Клерки памяти | склсервермемориклеркс | sqlserver_memory_clerks | sys.dm_os_memory_clerks | Да | 60 секунд |
| Ввод-вывод базы данных | склсервердатабасеио | sqlserver_database_io | sys.dm_io_virtual_file_stats<br>sys.master_files | Да | 60 секунд |
| Свойства сервера | склсерверпропертиес | sqlserver_server_properties | sys.dm_os_sys_info | Да | 60 секунд |
| Счетчики производительности | склсерверперформанцекаунтерс | sqlserver_performance | sys.dm_os_performance_counters | Да | 60 секунд |
| Место на томе | склсерверволумеспаце | sqlserver_volume_space | sys.master_files | Да | 60 секунд |
| SQL Server ЦП | склсерверкпу | sqlserver_cpu | sys.dm_os_ring_buffers | Да | 60 секунд |
| Планировщики | склсерверсчедулерс | sqlserver_schedulers | sys.dm_os_schedulers | Нет | Н/Д |
| Requests | склсерверрекуестс | sqlserver_requests | sys.dm_exec_sessions<br>sys.dm_exec_requests<br>sys.dm_exec_sql_text | Нет | Н/Д |
| Состояния реплик доступности | склсервераваилабилитирепликастатес | sqlserver_hadr_replica_states | sys.dm_hadr_availability_replica_states<br>sys.availability_replicas<br>sys.availability_groups<br>sys.dm_hadr_availability_group_states | Нет | 60 секунд |
| Реплики базы данных доступности | склсервердатабасерепликастатес | sqlserver_hadr_dbreplica_states | sys.dm_hadr_database_replica_states<br>sys.availability_replicas | Нет | 60 секунд |

## <a name="next-steps"></a>Дальнейшие действия

- Инструкции по включению SQL Insights см. в разделе [Включение SQL Insights](sql-insights-enable.md) .
- Ответы [на часто задаваемые вопросы о](../faq.md#sql-insights-preview) SQL Insights см. здесь.
