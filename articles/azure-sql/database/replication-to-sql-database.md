---
title: Репликация Azure SQL Server в базу данных SQL Azure
description: Вы можете настроить базу данных в базе данных SQL Azure в качестве извещающего подписчика в односторонней топологии репликации транзакций или моментальных снимков.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 6ff1d485ab4c0662ae8a9d754ce67b1446b76fcc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92780960"
---
# <a name="replication-to-azure-sql-database"></a>Репликация в базу данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Вы можете настроить базу данных SQL Azure в качестве извещающего подписчика в односторонней топологии репликации транзакций или моментальных снимков.

> [!NOTE]
> В этой статье описывается использование [репликации транзакций](/sql/relational-databases/replication/transactional/transactional-replication) в базе данных SQL Azure. Он не связан с [активной георепликацией](./active-geo-replication-overview.md), компонентом базы данных SQL Azure, который позволяет создавать полные доступные для чтения реплики отдельных баз данных.

## <a name="supported-configurations"></a>Поддерживаемые конфигурации
  
- База данных SQL Azure может быть только принудительным подписчиком SQL Server издателя и распространителя.  
- Экземпляр SQL Server, выступающий в роли издателя и (или) распространителя, может быть экземпляром [SQL Server, работающего локально](https://www.microsoft.com/sql-server/sql-server-downloads), [управляемый экземпляр Azure SQL](../managed-instance/instance-create-quickstart.md)или экземпляром [SQL Server, работающего на виртуальной машине Azure в облаке](../virtual-machines/windows/sql-vm-create-portal-quickstart.md). 
- База данных распространителя и агенты репликации не могут быть помещены в базу данных в базе данных SQL Azure.  
- Поддерживаются [моментальные снимки](/sql/relational-databases/replication/snapshot-replication) и [односторонняя](/sql/relational-databases/replication/transactional/transactional-replication) репликация транзакций. Одноранговая репликация транзакций и репликация слиянием не поддерживаются.

### <a name="versions"></a>Версии  

Для успешной репликации в базу данных в базе данных SQL Azure SQL Server издателей и распространителям следует использовать (по крайней мере) одну из следующих версий:

Публикация в любой базе данных SQL Azure из базы данных SQL Server поддерживается следующими версиями SQL Server.

- SQL Server 2016 и выше
- SQL Server 2014 [RTM Cu10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) или [пакет обновления 1 (SP1) CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [с пакетом обновления 2 (SP2) CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) или [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

> [!NOTE]
> Попытка настроить репликацию с использованием неподдерживаемой версии может привести к появлению номера ошибки MSSQL_REPL20084 (процессу не удалось подключиться к подписчику.) и MSSQL_REPL40532 (не удается открыть сервер, \<name> запрошенный именем для входа. Вход в систему не выполнен").  

Чтобы использовать все функции Базы данных SQL Azure, необходимо использовать последние версии [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt).  

### <a name="types-of-replication"></a>Типы репликации

Существуют разные [типы репликации](/sql/relational-databases/replication/types-of-replication).

| Репликация | База данных SQL Azure | Управляемый экземпляр SQL Azure |
| :----| :------------- | :--------------- |
| [**Стандартная транзакционная**](/sql/relational-databases/replication/transactional/transactional-replication) | Да (только в качестве подписчика) | Да | 
| [**Моментальный снимок**](/sql/relational-databases/replication/snapshot-replication) | Да (только в качестве подписчика) | Да|
| [**Репликация слиянием**](/sql/relational-databases/replication/merge/merge-replication) | Нет | Нет|
| [**Одноранговой**](/sql/relational-databases/replication/transactional/peer-to-peer-transactional-replication) | Нет | Нет|
| [**Двунаправленная репликация**](/sql/relational-databases/replication/transactional/bidirectional-transactional-replication) | Нет | Да|
| [**Обновляемые подписки**](/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication) | Нет | Нет|
| &nbsp; | &nbsp; | &nbsp; |

  
## <a name="remarks"></a>Remarks

- В Базе данных SQL Azure поддерживаются только принудительные подписки.  
- Репликацию можно настроить с помощью [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) или путем выполнения инструкций Transact-SQL в издателе. Репликацию невозможно настроить с помощью портала Azure.  
- Для подключения к базе данных SQL Azure репликация может использовать только SQL Server проверки подлинности.
- У реплицируемых таблиц должен быть первичный ключ.  
- У вас должна быть подписка Azure.  
- Подписчик базы данных SQL Azure может находиться в любом регионе.  
- Одна публикация на сервере SQL Server может поддерживать подписчиков и Базы данных SQL Azure, и SQL Server (локально и SQL Server на виртуальной машине Azure).  
- Управление репликацией, мониторинг и устранение неполадок необходимо выполнять из SQL Server, а не из базы данных SQL Azure.  
- В **sp_addsubscription** для Базы данных SQL поддерживается только `@subscriber_type = 0`.  
- База данных SQL Azure не поддерживает двунаправленную, немедленную, обновляемую или одноранговую репликацию.

## <a name="replication-architecture"></a>Архитектура репликации  

![На схеме показана архитектура репликации с базой данных SQL Azure, которая содержит несколько кластеров подписчиков в разных регионах, а также локальные виртуальные машины Azure, которые содержат издатель, исполняемый файл программу logread и исполняемые файлы распространителя, подключающиеся к удаленным кластерам.](./media/replication-to-sql-database/replication-to-sql-database.png)  

## <a name="scenarios"></a>Сценарии  

### <a name="typical-replication-scenario"></a>Распространенный сценарий репликации  

1. Создание публикации репликации транзакций в базе данных SQL Server.  
2. В SQL Server используйте **Мастер создания подписки** или инструкции TRANSACT-SQL, чтобы создать принудительную подписку на базу данных SQL Azure.  
3. Для отдельных баз данных и баз данных в пуле исходный набор данных обычно является моментальным снимком, который создается агентом моментальных снимков, а затем распределяется и применяется агентом распространения. С помощью издателя SQL Управляемый экземпляр можно также использовать резервную копию базы данных для заполнения подписчика базы данных SQL Azure.

### <a name="data-migration-scenario"></a>Сценарий переноса данных  

1. Используйте репликацию транзакций для репликации данных из базы данных SQL Server в базу данных SQL Azure.  
2. Перенаправьте клиент или приложения среднего уровня, чтобы обновить копию базы данных.  
3. Остановите обновление версии SQL Server таблицы и удалите публикацию.  

## <a name="limitations"></a>Ограничения

В подписках Базы данных SQL Azure не поддерживаются следующие действия:

- копирование сопоставления групп файлов;  
- копирование схем секционирования таблиц;  
- копирование схем секционирования индексов;  
- копирование определяемой пользователем статистики;  
- копирование привязок по умолчанию;  
- копирование привязок правил;  
- копирование полнотекстовых индексов;  
- копирование XSD XML;  
- копирование XML-индексов;  
- копирование разрешений.  
- копирование пространственных индексов;  
- копирование фильтруемых индексов;  
- копирование атрибута сжатия данных;  
- копирование атрибута разреженного столбца;  
- конвертирование файлового потока в типы данных MAX;  
- копирование hierarchyid в типы данных MAX;  
- копирование пространственных типов в типы данных MAX;  
- копирование расширенных свойств;  

### <a name="limitations-to-be-determined"></a>Ограничения, которые должны быть подтверждены

- Копирование параметров сортировки.  
- Выполнение в сериализованной транзакции хранимой процедуры.  

## <a name="examples"></a>Примеры

Создайте публикацию и принудительную подписку. Дополнительные сведения см. в разделе:
  
- [Create a Publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Создайте принудительную подписку](/sql/relational-databases/replication/create-a-push-subscription/) , используя имя сервера в качестве подписчика (например, **N'azuresqldbdns. Database. Windows. NET**) и имя базы данных SQL Azure в качестве целевой базы данных (например, **AdventureWorks**).  

## <a name="see-also"></a>См. также:  

- [Репликация транзакций](../managed-instance/replication-transactional-overview.md)
- [Create a Publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Создание принудительной подписки](/sql/relational-databases/replication/create-a-push-subscription/)
- [Типы репликации](/sql/relational-databases/replication/types-of-replication)
- [Мониторинг (репликация)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Инициализация подписки](/sql/relational-databases/replication/initialize-a-subscription)