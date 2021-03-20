---
title: Настройка ссылки на & управление содержимым
description: Найдите справочные материалы по настройке базы данных SQL Azure и управлению ею.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 01/14/2020
ms.openlocfilehash: 19cec1deeef344ea5897061db0fec88066b90b05
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92791772"
---
# <a name="configure-and-manage-content-reference---azure-sql-database"></a>Настройка и управление ссылкой на содержимое в базе данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье содержатся ссылки на материалы по различным руководствам, сценариям и объяснениям, которые могут помочь в управлении базой данных SQL Azure и ее настройке. 

## <a name="load-data"></a>Загрузка данных

- [Миграция в Базу данных SQL](migrate-to-database-from-sql-server.md)
- Узнайте, как [управлять базой данных SQL после миграции](manage-data-after-migrating-to-database.md).
- [Копирование базы данных](database-copy.md)
- [Импорт базы данных из BACPAC-файла](database-import.md)
- [Экспорт базы данных в BACPAC-файл](database-export.md)
- [Загрузка данных с помощью BCP](../load-from-csv-with-bcp.md)
- [Загрузка данных с помощью ADF](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="configure-features"></a>Настройка компонентов

- [Настройка проверки подлинности Azure Active Directory (Azure AD)](authentication-aad-configure.md)
- [Настройка условного доступа](conditional-access-configure.md)
- [Многофакторная проверка подлинности Azure AD](authentication-mfa-ssms-overview.md)
- [Настройка многофакторной проверки подлинности](authentication-mfa-ssms-configure.md)
- [Настройка политики временного хранения](temporal-tables-retention-policy.md)
- [Настройка BYOK для TDE](transparent-data-encryption-byok-configure.md)
- [Смена ключей BYOK для TDE](transparent-data-encryption-byok-key-rotation.md)
- [Удаление предохранителя TDE](transparent-data-encryption-byok-remove-tde-protector.md)
- [Настройка выполняющейся в памяти OLTP](../in-memory-oltp-configure.md)
- [Настройка службы автоматизации Azure](automation-manage.md)
- [Настройка репликации транзакций](replication-to-sql-database.md) для репликации данных между базами данных.
- [Настройка обнаружения угроз](threat-detection-configure.md), чтобы База данных SQL Azure могла идентифицировать подозрительные действия, такие как внедрение кода SQL или доступ из подозрительных мест.
- [Настройка динамического маскирования данных](dynamic-data-masking-configure-portal.md) для защиты конфиденциальных данных.
- [Настройка хранения резервных копий ](long-term-backup-retention-configure.md) для базы данных, чтобы сохранять резервные копии в хранилище BLOB-объектов Azure. 
- [Настройка георепликации](active-geo-replication-overview.md), чтобы хранить реплику базы данных в другом регионе.
- [Настройка безопасности для геореплик](active-geo-replication-security-configure.md).

## <a name="monitor-and-tune-your-database"></a>Мониторинг и настройка базы данных

- [Настройка вручную](performance-guidance.md)
- [Использование динамических административных представлений для мониторинга производительности](monitoring-with-dmvs.md)
- [Использование хранилища запросов для мониторинга производительности](/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Включение автоматической настройки](automatic-tuning-enable.md), чтобы База данных SQL Azure имела возможность оптимизировать производительность рабочей нагрузки.
- [Включение уведомлений по электронной почте об автоматической настройке](automatic-tuning-email-notifications-configure.md), чтобы получать сведения о рекомендациях по настройке.
- [Применение рекомендаций по производительности](database-advisor-find-recommendations-portal.md) и оптимизация своей базы данных.
- [Создание оповещений](alerts-insights-configure-portal.md) для получения уведомлений от Базы данных SQL Azure.
- [Устранение неполадок с подключением](troubleshoot-common-errors-issues.md) при возникновении проблем с подключением между приложениями и базой данных. Вы также можете воспользоваться службой [Работоспособность ресурсов](resource-health-to-troubleshoot-connectivity.md).
- [Устранение проблем с производительностью с помощью Intelligent Insights](intelligent-insights-troubleshoot-performance.md)
- [Управление файловым пространством](file-space-manage.md), чтобы отслеживать использование хранилища в базе данных.
- [Использование журнала диагностики Intelligent Insights](intelligent-insights-use-diagnostics-log.md)
- [Мониторинг используемого пространства в хранилище выполняющейся в памяти OLTP](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Расширенные события

- [Расширенные события](xevent-db-diff-from-svr.md)
- [Хранение расширенных событий в файлах событий](xevent-code-event-file.md)
- [Хранение расширенных событий в кольцевом буфере](xevent-code-ring-buffer.md)

## <a name="query-distributed-data"></a>Запрашивание распределенных данных

- [Запрос вертикально секционированных данных](elastic-query-getting-started-vertical.md) между несколькими базами данных.
- [Отчет по масштабируемым уровням данных](elastic-query-horizontal-partitioning.md).
- [Запрос к нескольким таблицам с разными схемами](elastic-query-vertical-partitioning.md).

### <a name="data-sync"></a>Синхронизация данных

- [Синхронизация данных SQL](sql-data-sync-data-sql-server-sql-database.md)
- [Data Sync Agent](sql-data-sync-agent-overview.md)
- [Реплицировать изменения схемы](sql-data-sync-update-sync-schema.md)
- [Мониторинг с помощью OMS](./monitor-tune-overview.md)
- [Рекомендации по Синхронизации данных](sql-data-sync-best-practices.md)
- [Устранение ошибок Синхронизации данных](sql-data-sync-troubleshoot.md)

## <a name="elastic-database-jobs"></a>Задания обработки эластичных баз данных

- [Создание и управление](elastic-jobs-powershell-create.md) Задания обработки эластичных баз данных с помощью PowerShell.
- [Создание заданий эластичной базы данных и управление ими с помощью Transact-SQL (T-SQL)](elastic-jobs-tsql-create-manage.md).
- [Перенос данных из старых заданий обработки эластичных БД](elastic-jobs-migrate.md)

## <a name="database-sharding"></a>Сегментирование базы данных

- [Обновите клиентскую библиотеку эластичной базы данных](elastic-scale-upgrade-client-library.md).
- [Создайте Сегментированное приложение](elastic-scale-get-started.md).
- [Запрос горизонтально секционированных данных](elastic-query-getting-started.md).
- [Выполнение запросов к нескольким сегментам](elastic-scale-multishard-querying.md).
- [Перемещение сегментированных данных](elastic-scale-configure-deploy-split-and-merge.md).
- [Настройка параметров безопасности для сегментов базы данных](elastic-scale-split-merge-security-configuration.md).
- [Добавление сегмента](elastic-scale-add-a-shard.md) в текущий набор сегментов базы данных.
- [Устранение проблем с картой сегментов](elastic-database-recovery-manager.md).
- [Перенос сегментированной базы данных](elastic-convert-to-use-elastic-tools.md).
- [Создание счетчиков](elastic-database-perf-counters.md).
- [Использование Entity Framework](elastic-scale-use-entity-framework-applications-visual-studio.md) для запроса сегментированных данных.
- [Использование платформы Dapper](elastic-scale-working-with-dapper.md) для запроса сегментированных данных.

## <a name="develop-applications"></a>Разработка приложений

- [Соединение](connect-query-content-reference-guide.md#libraries)
- [Использование соединителя Spark](spark-connector.md)
- [Аутентификация приложения](application-authentication-get-client-id-keys.md)
- [Использование пакетной обработки для лучшей производительности](../performance-improve-use-batching.md)
- [Руководство по настройке подключений](troubleshoot-common-connectivity-issues.md)
- [Псевдонимы DNS](dns-alias-overview.md)
- [Настройка DNS-псевдонимов в PowerShell](dns-alias-powershell-create.md)
- [Порты: ADO.NET](adonet-v12-develop-direct-route-ports.md)
- [C и C++](develop-cplusplus-simple.md)
- [Excel](connect-excel.md)

## <a name="design-applications"></a>Разработка приложений

- [Разработка для аварийного восстановления](designing-cloud-solutions-for-disaster-recovery.md)
- [Разработка для эластичных пулов](disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Разработка для обновления приложений](manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-software-as-a-service-saas-applications"></a>Проектирование приложений для нескольких клиентов как услуги (SaaS)

- [Шаблоны проектирования SaaS](saas-tenancy-app-design-patterns.md)
- [Индексатор видео SaaS](saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Безопасность приложений SaaS](saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [руководствах для управляемый экземпляр Azure SQL](../managed-instance/how-to-content-reference-guide.md)