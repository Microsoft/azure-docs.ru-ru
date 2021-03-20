---
title: Репликация транзакций
titleSuffix: Azure SQL Managed Instance
description: Сведения об использовании SQL Server репликации транзакций с помощью Управляемый экземпляр SQL Azure (Предварительная версия).
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: sstein
ms.date: 04/20/2020
ms.openlocfilehash: 3e4b4fc3d4a6c9529c7c0ac0daef8a28173e0bf3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99225349"
---
# <a name="transactional-replication-with-azure-sql-managed-instance-preview"></a>Репликация транзакций с помощью Управляемый экземпляр Azure SQL (Предварительная версия)
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Репликация транзакций — это функция Управляемый экземпляр Azure SQL и SQL Server, которая позволяет реплицировать данные из таблицы в Управляемый экземпляр SQL Azure или экземпляре SQL Server в таблицы, размещенные в удаленных базах данных. Она обеспечивает синхронизацию множества таблиц в различных базах данных. 

В настоящее время репликация транзакций предоставляется в общедоступной предварительной версии Управляемого экземпляра SQL. 

## <a name="overview"></a>Обзор

Репликацию транзакций можно использовать для отправки изменений, внесенных в Управляемый экземпляр Azure SQL, в:

- SQL Server базы данных локально или на виртуальной машине Azure;
- База данных в базе данных SQL Azure
- База данных экземпляра в Azure SQL Управляемый экземпляр

  > [!NOTE]
  > Чтобы использовать все функции Управляемый экземпляр Azure SQL, необходимо использовать последние версии [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt).

### <a name="components"></a>Компоненты

Ключевыми компонентами в репликации транзакций являются **Издатель**, **распространитель** и **подписчик**, как показано на следующем рисунке:  

![Репликация с Базой данных SQL](./media/replication-transactional-overview/replication-to-sql-database.png)

| Роль | База данных SQL Azure | Управляемый экземпляр SQL Azure |
| :----| :------------- | :--------------- |
| **Издатель** | Нет | Да |
| **Распространитель** | Нет | Да|
| **Подписчик по запросу** | Нет | Да|
| **Принудительная подписка**| Да | Да|
| &nbsp; | &nbsp; | &nbsp; |

**Издатель** публикует изменения, внесенные в некоторые таблицы (статьи), отправляя обновления на распространитель. Издатель может быть Управляемый экземпляром SQL Azure или экземпляром SQL Server.

**Распространитель** собирает изменения в статьях от издателя и распространяет их подписчикам. Распространителем может быть либо Управляемый экземпляр Azure SQL, либо экземпляр SQL Server (любая версия, если она равна или выше версии издателя).

**Подписчик** получает изменения, внесенные на издателе. Экземпляр SQL Server и Azure SQL Управляемый экземпляр могут быть принудительными и опрашивающими подписчиками, хотя подписка по запросу не поддерживается, если распространитель является Управляемый экземпляр Azure SQL, а подписчик — нет. База данных SQL Azure может быть только принудительным подписчиком.

Управляемый экземпляр Azure SQL может поддерживать подписчика из следующих версий SQL Server:

- SQL Server 2016 и более поздние версии
- SQL Server 2014 [RTM Cu10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) или [пакет обновления 1 (SP1) CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [с пакетом обновления 2 (SP2) CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) или [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

   > [!NOTE]
   >
   > - Для других версий SQL Server, которые не поддерживают публикацию в объекты в Azure, можно использовать метод [повторной публикации данных](/sql/relational-databases/replication/republish-data), чтобы перемещать данные в более новые версии SQL Server.
   > - Попытка настроить репликацию с использованием более старой версии может привести к ошибке MSSQL_REPL20084 (The process could not connect to Subscriber (Процессу не удалось подключиться к подписчику)) или MSSQ_REPL40532 ("Не удается открыть сервер \<name>, запрашиваемый с использованием имени для входа. Вход в систему не выполнен").

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

### <a name="supportability-matrix"></a>Матрица поддержки

  Матрица поддержки репликации транзакций для Управляемый экземпляр Azure SQL — это то же самое, что и для SQL Server.
  
| **Издатель**   | **Распространитель** | **Подписчик** |
| :------------   | :-------------- | :------------- |
| SQL Server 2019 | SQL Server 2019 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/>  |
| SQL Server 2017 | SQL Server 2019 <br/>SQL Server 2017 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 |
| SQL Server 2016 | SQL Server 2019 <br/>SQL Server 2017 <br/> SQL Server 2016 | SQL Server 2019 <br/> SQL Server 2017 <br/>SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 |
| SQL Server 2014 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>| SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2012 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> | SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2008 R2 <br/> SQL Server 2008 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |  SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 <br/>  |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="when-to-use"></a>Назначение

Репликация транзакций удобна в следующих случаях:

- Опубликуйте изменения, внесенные в одной или нескольких таблицах в базе данных, и распространите их в одну или несколько баз данных в SQL Server или базе данных SQL Azure, подписанные на изменения.
- Поддержание нескольких распределенных баз данных в синхронизированном состоянии.
- Перенос баз данных из одного SQL Server экземпляра или Управляемый экземпляр SQL Azure в другую базу данных путем непрерывной публикации изменений.

### <a name="compare-data-sync-with-transactional-replication"></a>Сравнение синхронизации данных с репликацией транзакций

| Category | Синхронизация данных | репликация транзакций |
|---|---|---|
| Преимущества | — Поддержка режима "активный — активный"<br/>— Двусторонняя передача данных между локальной базой данных и службой "База данных SQL Azure" | — Низкая задержка<br/>— Согласованность транзакций<br/>— Повторное использование существующей топологии после миграции |
| Недостатки | — Отсутствует согласованность транзакций<br/>— Большее влияние на производительность | -Не удается опубликовать из базы данных SQL Azure <br/>— Дорогое обслуживание |

## <a name="common-configurations"></a>Распространенные конфигурации

Как правило, издатель и распространитель должны находиться либо в облаке, либо в локальной среде. Поддерживаются следующие конфигурации:

### <a name="publisher-with-local-distributor-on-sql-managed-instance"></a>Издатель с локальным распространителем на SQL Управляемый экземпляр

![Единственный экземпляр в качестве издателя и распространителя](./media/replication-transactional-overview/01-single-instance-asdbmi-pubdist.png)

Издатель и распространитель настраиваются в одном Управляемый экземпляр SQL и распределяют изменения в другой SQL Управляемый экземпляр, базу данных SQL или экземпляр SQL Server.

### <a name="publisher-with-remote-distributor-on-sql-managed-instance"></a>Издатель с удаленным распространителем на SQL Управляемый экземпляр

В этой конфигурации один управляемый экземпляр публикует изменения на распространителе, размещенном на другом Управляемый экземпляр SQL, который может обслуживать множество управляемых экземпляров SQL и распространять изменения на один или несколько целевых объектов в базе данных SQL Azure, Azure SQL Управляемый экземпляр или SQL Server.

![Раздельные экземпляры для издателя и распространителя](./media/replication-transactional-overview/02-separate-instances-asdbmi-pubdist.png)

Издателя и распространителя можно настроить в двух управляемых экземплярах. В этой конфигурации есть некоторые ограничения:

- Оба управляемых экземпляра находятся в одной виртуальной сети.
- Оба управляемые экземпляры находятся в том же расположении.

### <a name="on-premises-publisherdistributor-with-remote-subscriber"></a>Локальный издатель или распространитель с удаленным подписчиком

![База данных SQL Azure в качестве подписчика](./media/replication-transactional-overview/03-azure-sql-db-subscriber.png)

В этой конфигурации база данных в базе данных SQL Azure или Azure SQL Управляемый экземпляр является подписчиком. Эта конфигурация поддерживает миграцию из локальной среды в Azure. Если подписчик является базой данных в базе данных SQL Azure, она должна находиться в режиме принудительной отправки.  

## <a name="requirements"></a>Требования

- Используйте проверку подлинности SQL для подключения между участниками репликации.
- Используйте общий ресурс учетной записи хранения Azure для рабочего каталога, используемого репликацией.
- Откройте исходящий TCP-порт 445 в правилах безопасности подсети, чтобы получить доступ к файловому ресурсу Azure.
- Откройте TCP-порт исходящего трафика 1433, если SQL Управляемый экземпляр является издателем или распространителем, а подписчик — нет. Также может потребоваться изменить правило безопасности для исходящего трафика NSG SQL Управляемый экземпляр для `allow_linkedserver_outbound` для **тега службы назначения** Port 1433 с `virtualnetwork` на `internet` .
- Разместите издатель и распространитель в облаке или в обеих локальных средах.
- Настройте пиринг VPN между виртуальными сетями участников репликации, если виртуальные сети различаются.

> [!NOTE]
> При подключении к файлу службы хранилища Azure может возникнуть ошибка 53, если порт исходящей сетевой группы безопасности (NSG) 445 заблокирован, если распространитель является базой данных Azure SQL Управляемый экземпляр, а подписчик — локальным. Чтобы устранить эту проблему [, обновите NSG виртуальной сети](../../storage/files/storage-troubleshoot-windows-file-connection-problems.md) .

## <a name="with-failover-groups"></a>С группами отработки отказа

[Активная Георепликация](../database/active-geo-replication-overview.md) не поддерживается для SQL управляемый экземпляр с использованием репликации транзакций. Вместо активной георепликации используйте [группы автоматической отработки отказа](../database/auto-failover-group-overview.md), но обратите внимание на то, что публикацию необходимо [вручную удалить](transact-sql-tsql-differences-sql-server.md#replication) из основного управляемого экземпляра и создать повторно во вторичной управляемый экземпляр SQL после отработки отказа.

Если Георепликация включена на **издателе** или SQL-управляемый экземпляр **распространителя** в [группе отработки отказа](../database/auto-failover-group-overview.md), администратор SQL управляемый экземпляр должен очистить все публикации на старом первичном ресурсе и перенастроить их на новом первичном экземпляре после отработки отказа. В этом сценарии необходимы следующие действия.

1. Останавливает все задания репликации, выполняющиеся в базе данных, если они есть.
1. Удалите метаданные подписки из издателя, выполнив следующий скрипт в базе данных издателя:

   ```sql
   EXEC sp_dropsubscription @publication='<name of publication>', @article='all',@subscriber='<name of subscriber>'
   ```

1. Удаление метаданных подписки с подписчика. Выполните следующий скрипт в базе данных подписки на подписчике SQL Управляемый экземпляр:

   ```sql
   EXEC sp_subscription_cleanup
      @publisher = N'<full DNS of publisher, e.g. example.ac2d23028af5.database.windows.net>',
      @publisher_db = N'<publisher database>',
      @publication = N'<name of publication>';
   ```

1. Принудительно удалите все объекты репликации из издателя, выполнив следующий скрипт в опубликованной базе данных:

   ```sql
   EXEC sp_removedbreplication
   ```

1. Принудительно удалить старый распространитель из исходной основной Управляемый экземпляр SQL (при восстановлении размещения на старом первичном экземпляре, который использовался для создания распространителя). Выполните следующий скрипт в базе данных master в Управляемый экземпляр SQL старого распространителя:

   ```sql
   EXEC sp_dropdistributor 1,1
   ```

Если Георепликация включена на экземпляре **подписчика** в группе отработки отказа, публикация должна быть настроена для подключения к конечной точке прослушивателя группы отработки отказа для управляемого экземпляра подписчика. В случае отработки отказа последующее действие, выполняемое администратором управляемого экземпляра, зависит от типа отработки отказа, которая произошла:

- Для отработки отказа без потери данных репликация продолжит работу после отработки отказа.
- Для отработки отказа с потерей данных также будет работать репликация. После этого будет выполнена повторная репликация потерянных изменений.
- Для отработки отказа с потерей данных, но потери данных выходят за пределы срока хранения базы данных распространителя, администратору SQL Управляемый экземпляр потребуется повторно инициализировать базу данных подписки.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настройке репликации транзакций см. в следующих учебниках:

- [Настройка репликации между издателем SQL Управляемый экземпляр и подписчиком](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Настройка репликации между издателем SQL Управляемый экземпляр, распространителем SQL Управляемый экземпляр и подписчиком SQL Server](../managed-instance/replication-two-instances-and-sql-server-configure-tutorial.md)
- [Создайте публикацию](/sql/relational-databases/replication/publish/create-a-publication).
- [Создайте принудительную подписку](/sql/relational-databases/replication/create-a-push-subscription) , используя имя сервера в качестве подписчика (например, `N'azuresqldbdns.database.windows.net` и базу данных в имени базы данных SQL Azure в качестве целевой базы данных (например, **AdventureWorks**). )

## <a name="see-also"></a>См. также раздел  

- [Репликация с помощью Управляемый экземпляр SQL и группы отработки отказа](transact-sql-tsql-differences-sql-server.md#replication)
- [Репликация в Базу данных SQL](../database/replication-to-sql-database.md)
- [Репликация на управляемый экземпляр](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Create a Publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Создание принудительной подписки](/sql/relational-databases/replication/create-a-push-subscription/)
- [Типы репликации](/sql/relational-databases/replication/types-of-replication)
- [Мониторинг (репликация)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Инициализация подписки](/sql/relational-databases/replication/initialize-a-subscription)
