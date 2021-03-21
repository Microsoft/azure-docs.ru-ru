---
title: Настройка ссылки на & управление содержимым
titleSuffix: Azure SQL Managed Instance
description: Справочное руководство по содержимому, в котором описывается настройка Управляемый экземпляр Azure SQL и управление ими.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 04/16/2019
ms.openlocfilehash: b7f2f060f32cf3bf92660a2e8b75a45b7ec76b82
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92779770"
---
# <a name="azure-sql-managed-instance-content-reference"></a>Справочник по содержимому Управляемый экземпляр Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этой статье содержатся ссылки на материалы по различным руководствам, сценариям и объяснениям, которые помогут в управлении Управляемый экземпляр SQL Azure.

## <a name="load-data"></a>Загрузка данных

- [Переход на управляемый экземпляр Azure SQL](migrate-to-instance-from-sql-server.md): сведения о рекомендуемых процессах и средствах миграции для миграции на управляемый экземпляр SQL Azure.
- [Перенос сертификата TDE в Azure SQL управляемый экземпляр](tde-certificate-migrate.md). Если база данных SQL Server защищена с помощью прозрачного шифрования данных (TDE), необходимо перенести сертификат, который управляемый экземпляр SQL может использовать для расшифровки резервной копии, которую вы хотите восстановить в Azure.
- [Импорт базы данных из BACPAC-файла](../database/database-import.md)
- [Экспорт базы данных в BACPAC-файл](../database/database-export.md)
- [Загрузка данных с помощью BCP](../load-from-csv-with-bcp.md)
- [Загрузка данных с помощью фабрики данных Azure](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="network-configuration"></a>Сетевая конфигурация

- [Определение размера подсети](vnet-subnet-determine-size.md). Поскольку размер подсети невозможно изменить после развертывания SQL управляемый экземпляр, необходимо вычислить диапазон IP-адресов, необходимый для числа и типов управляемых экземпляров, которые планируется развернуть в подсети. 
- [Создание новой виртуальной сети и подсети](virtual-network-subnet-create-arm-template.md). настройте виртуальную сеть и подсеть в соответствии с [требованиями сети](connectivity-architecture-overview.md#network-requirements). 
- [Настройка существующей виртуальной сети и подсети](vnet-existing-add-subnet.md). Проверьте требования к сети и настройте существующую виртуальную сеть и подсеть для развертывания управляемый экземпляр SQL. 
- [Настройка настраиваемой службы DNS](custom-dns-configure.md). Настройте настраиваемый DNS, чтобы предоставить доступ к внешним ресурсам для пользовательских доменов из управляемый экземпляр SQL с помощью связанного сервера профилей электронной почты базы данных. 
- [Синхронизация конфигурации сети](azure-app-sync-network-configuration.md). Обновите план конфигурации сети, если не удается установить подключение после [интеграции приложения с виртуальной сетью Azure](../../app-service/web-sites-integrate-with-vnet.md).
- [Поиск IP-адреса конечной точки управления](management-endpoint-find-ip-address.md). Определите общедоступную конечную точку, которую SQL управляемый экземпляр использует для целей управления. 
- [Проверьте встроенную защиту брандмауэра](management-endpoint-verify-built-in-firewall.md). Убедитесь, что управляемый экземпляр SQL разрешает трафик только на необходимых портах и других встроенных правилах брандмауэра. 
- [Подключение приложений](connect-application-instance.md). Узнайте о различных шаблонах подключения приложений к SQL управляемый экземпляр.

## <a name="feature-configuration"></a>Конфигурация компонентов

- [Настройка аутентификации Azure AD](../database/authentication-aad-configure.md)
- [Настройка условного доступа](../database/conditional-access-configure.md)
- [Многофакторная проверка подлинности Azure AD](../database/authentication-mfa-ssms-overview.md)
- [Настройка многофакторной проверки подлинности](../database/authentication-mfa-ssms-configure.md)
- [Настройка временной политики хранения](../database/temporal-tables-retention-policy.md)
- [Настройка BYOK для TDE](../database/transparent-data-encryption-byok-configure.md)
- [Смена ключей BYOK для TDE](../database/transparent-data-encryption-byok-key-rotation.md)
- [Удаление предохранителя TDE](../database/transparent-data-encryption-byok-remove-tde-protector.md)
- [Настройка выполняющейся в памяти OLTP](../in-memory-oltp-configure.md)
- [Настройка службы автоматизации Azure](../database/automation-manage.md)
- [Репликация транзакций](replication-between-two-instances-configure-tutorial.md) позволяет реплицировать данные между управляемыми экземплярами или локальным SQL Server управляемый экземпляр SQL, и наоборот.
- [Настройка обнаружения угроз](threat-detection-configure.md) — [обнаружение угроз](../database/threat-detection-overview.md) — это встроенная функция управляемый экземпляр Azure SQL, которая обнаруживает различные потенциальные атаки, такие как внедрение кода SQL или доступ из подозрительных расположений. 
- [Создание оповещений](alerts-create.md) позволяет настроить оповещения для наблюдаемых метрик, таких как загрузка ЦП, использование дискового пространства, операций ввода-вывода в секунду и других для SQL управляемый экземпляр. 

## <a name="monitoring-and-tuning"></a>Мониторинг и настройка

- [Настройка вручную](../database/performance-guidance.md)
- [Использование динамических административных представлений для мониторинга производительности](../database/monitoring-with-dmvs.md)
- [Использование хранилища запросов для наблюдения за производительностью](/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Устранение проблем с производительностью с помощью Intelligent Insights](../database/intelligent-insights-troubleshoot-performance.md)
- [Использование журнала диагностики Intelligent Insights](../database/intelligent-insights-use-diagnostics-log.md)
- [Мониторинг пространства In-Memory OLTP](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Расширенные события

- [Расширенные события](../database/xevent-db-diff-from-svr.md)
- [Хранение расширенных событий в файле событий](../database/xevent-code-event-file.md)
- [Хранение расширенных событий в кольцевом буфере](../database/xevent-code-ring-buffer.md)

### <a name="alerting"></a>Оповещение

- [Создание оповещений об управляемом экземпляре](alerts-create.md)

## <a name="operations"></a>Операции

- [Инициированный пользователем переход на другой ресурс вручную на Управляемом экземпляре SQL](user-initiated-failover.md)

## <a name="develop-applications"></a>Разработка приложений

- [Соединение](../database/connect-query-content-reference-guide.md#libraries)
- [Использование соединителя Spark](../../cosmos-db/spark-connector.md)
- [Проверка подлинности приложения](../database/application-authentication-get-client-id-keys.md)
- [Использование пакетной обработки для лучшей производительности](../performance-improve-use-batching.md)
- [Руководство по настройке подключений](../database/troubleshoot-common-connectivity-issues.md)
- [Псевдонимы DNS](../database/dns-alias-overview.md)
- [Настройка DNS-псевдонима с помощью PowerShell](../database/dns-alias-powershell-create.md)
- [Порты: ADO.NET](../database/adonet-v12-develop-direct-route-ports.md)
- [C и C++](../database/develop-cplusplus-simple.md)
- [Excel](../database/connect-excel.md)

## <a name="design-applications"></a>Разработка приложений

- [Разработка для аварийного восстановления](../database/designing-cloud-solutions-for-disaster-recovery.md)
- [Разработка для эластичных пулов](../database/disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Разработка для обновления приложений](../database/manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-saas-applications"></a>Разработка мультитенантных приложений SaaS

- [Шаблоны проектирования SaaS](../database/saas-tenancy-app-design-patterns.md)
- [Индексатор видео SaaS](../database/saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Безопасность приложений SaaS](../database/saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Дальнейшие действия

Начните с [развертывания управляемый экземпляр SQL](instance-create-quickstart.md).