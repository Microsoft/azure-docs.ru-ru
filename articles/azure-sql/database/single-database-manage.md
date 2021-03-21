---
title: Создание & управление серверами и отдельными базами данных
description: Узнайте, как создавать и администрировать серверы и отдельные базы данных в базе данных SQL Azure с помощью портал Azure, PowerShell, Azure CLI, Transact-SQL (T-SQL) и API-интерфейса RESTful.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: e50cce74f7291a6673e5d43f3485a1c63c81d827
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93319285"
---
# <a name="create-and-manage-servers-and-single-databases-in-azure-sql-database"></a>Создание серверов и отдельных баз данных и управление ими в базе данных SQL Azure

Вы можете создавать и администрировать серверы и отдельные базы данных в базе данных SQL Azure с помощью портал Azure, PowerShell, Azure CLI, REST API и Transact-SQL.

## <a name="the-azure-portal"></a>Портал Azure

Вы можете создать группу ресурсов для базы данных SQL Azure заранее или во время создания самого сервера.

### <a name="create-a-server"></a>Создание сервера

Чтобы создать сервер с помощью [портал Azure](https://portal.azure.com), создайте новый ресурс [сервера](logical-servers.md) из Azure Marketplace. Кроме того, можно создать сервер при развертывании базы данных SQL Azure.

  ![создать сервер](./media/single-database-manage/create-logical-sql-server.png)

### <a name="create-a-blank-or-sample-database"></a>Создание пустого или образца базы данных

Чтобы создать отдельную базу данных SQL Azure с помощью [портал Azure](https://portal.azure.com), выберите ресурс базы данных SQL Azure в Azure Marketplace. Вы можете создать группу ресурсов и сервер заранее или при создании отдельной базы данных. Можно создать пустую базу данных или пример базы данных на основе Adventure Works LT.

  ![Создание базы данных — 1](./media/single-database-manage/create-database-1.png)

> [!IMPORTANT]
> См. подробнее о выборе ценовой категории для базы данных в описании моделей приобретения на основе [единиц DTU](service-tiers-dtu.md) и [виртуальных ядер](service-tiers-vcore.md).

## <a name="manage-an-existing-server"></a>Управление существующим сервером

Чтобы управлять существующим сервером, перейдите на сервер с помощью нескольких методов, например из конкретной страницы базы данных, со страницы **серверы SQL** или со страницы **все ресурсы** .

Чтобы управлять существующей базой данных, перейдите на страницу **базы данных SQL** и выберите базу данных, которой требуется управлять. На следующем снимке экрана показано, как можно начать настройку брандмауэра уровня сервера для базы данных на странице **Обзор** для базы данных.

   ![правило брандмауэра для сервера](./media/single-database-manage/server-firewall-rule.png)

> [!IMPORTANT]
> См. подробнее о настройке свойств производительности для базы данных в описании моделей приобретения на основе [единиц DTU](service-tiers-dtu.md) и [виртуальных ядер](service-tiers-vcore.md).
> [!TIP]
> Краткое руководство по портал Azure см. в разделе [Создание базы данных SQL в портал Azure](single-database-create-quickstart.md).

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается базой данных SQL Azure, но вся будущая разработка сосредоточена на модуле Az.Sql. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны.

Для создания и управления серверами, одними и базами данных в составе пула, а также брандмауэрами уровня сервера с Azure PowerShell используйте следующие командлеты PowerShell. Если вам нужно выполнить установку или обновление PowerShell, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!TIP]
> Примеры сценариев PowerShell см. в статьях [Использование PowerShell для создания базы данных в базе данных SQL и настройка правила брандмауэра уровня сервера](scripts/create-and-configure-database-powershell.md) и [мониторинг и масштабирование базы данных в базе данных SQL с помощью PowerShell](scripts/monitor-and-scale-database-powershell.md).

| Командлет | Описание |
| --- | --- |
|[New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)|Создает базу данных. |
|[Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase)|Получает одну или несколько баз данных.|
|[Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)|Определяет свойства базы данных или перемещает ее в эластичный пул.|
|[Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase)|Удаляет базу данных.|
|[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)|Создает группу ресурсов.|
|[New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver)|Создает сервер.|
|[Get-Азсклсервер](/powershell/module/az.sql/get-azsqlserver)|Возвращает сведения о серверах.|
|[Set-Азсклсервер](/powershell/module/az.sql/set-azsqlserver)|Изменяет свойства сервера.|
|[Remove-Азсклсервер](/powershell/module/az.sql/remove-azsqlserver)|Удаляет сервера.|
|[New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule)|Создает правило брандмауэра уровня сервера. |
|[Get-Азсклсерверфиреваллруле](/powershell/module/az.sql/get-azsqlserverfirewallrule)|Получает правила брандмауэра для сервера.|
|[Set-Азсклсерверфиреваллруле](/powershell/module/az.sql/set-azsqlserverfirewallrule)|Изменяет правило брандмауэра на сервере.|
|[Remove-Азсклсерверфиреваллруле](/powershell/module/az.sql/remove-azsqlserverfirewallrule)|Удаляет правило брандмауэра с сервера.|
| New-AzSqlServerVirtualNetworkRule | Создает [*правило виртуальной сети*](vnet-service-endpoint-rule-overview.md) на основе подсети, которая является конечной точкой службы для виртуальной сети. |

## <a name="the-azure-cli"></a>Azure CLI

Для создания серверов, баз данных и брандмауэров и управления ими с помощью [Azure CLI](/cli/azure)используйте следующие [Azure CLI](/cli/azure/sql/db) команды. Запускайте интерфейс командной строки в браузере с помощью [Cloud Shell](../../cloud-shell/overview.md) либо [установите](/cli/azure/install-azure-cli) его на платформе macOS, Linux или Windows. Изучите дополнительные сведения о создании [эластичных пулов](elastic-pool-overview.md) и управлении ими.

> [!TIP]
> Краткое руководство по Azure CLI см. [в разделе Создание отдельной базы данных SQL Azure с помощью Azure CLI](az-cli-script-samples-content-guide.md). Примеры сценариев Azure CLI см. в статьях [Использование CLI для создания базы данных в базе данных SQL Azure и настройка правила брандмауэра базы данных SQL](scripts/create-and-configure-database-cli.md) и [Использование интерфейса командной строки для мониторинга и масштабирования базы данных в базе данных SQL Azure](scripts/monitor-and-scale-database-cli.md).
>

| Командлет | Описание |
| --- | --- |
|[az sql db create](/cli/azure/sql/db#az-sql-db-create) |Создает базу данных.|
|[az sql db list](/cli/azure/sql/db#az-sql-db-list)|Выводит список всех баз данных и хранилищ данных на сервере или всех баз данных в эластичном пуле.|
|[az sql db list-editions](/cli/azure/sql/db#az-sql-db-list-editions)|Выводит список доступных целевых служб и ограничений хранилища.|
|[az sql db list-usages](/cli/azure/sql/db#az-sql-db-list-usages)|Возвращает данные об использовании баз данных.|
|[az sql db show](/cli/azure/sql/db#az-sql-db-show)|Возвращает сведения о базе данных или хранилище данных.|
|[az sql db update](/cli/azure/sql/db#az-sql-db-update)|Обновляет базу данных.|
|[az sql db delete](/cli/azure/sql/db#az-sql-db-delete)|Удаляет базу данных.|
|[az group create](/cli/azure/group#az-group-create)|Создает группу ресурсов.|
|[az sql server create](/cli/azure/sql/server#az-sql-server-create)|Создает сервер.|
|[az sql server list](/cli/azure/sql/server#az-sql-server-list)|Выводит список серверов.|
|[az sql server list-usages](/cli/azure/sql/server#az-sql-server-list-usages)|Возвращает данные об использовании серверов.|
|[az sql server show](/cli/azure/sql/server#az-sql-server-show)|Возвращает сервер.|
|[az sql server update](/cli/azure/sql/server#az-sql-server-update)|Обновляет сервер.|
|[az sql server delete](/cli/azure/sql/server#az-sql-server-delete)|Удаляет сервер.|
|[az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create)|Создает правило брандмауэра для сервера.|
|[az sql server firewall-rule list](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-list)|Выводит список правил брандмауэра на сервере.|
|[az sql server firewall-rule show](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-show)|Отображает сведения о правиле брандмауэра.|
|[az sql server firewall-rule update](/cli/azure/sql/server/firewall-rule##az-sql-server-firewall-rule-update)|Обновляет правило брандмауэра.|
|[az sql server firewall-rule delete](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-delete)|Удаляет правило брандмауэра.|

## <a name="transact-sql-t-sql"></a>Transact-SQL (T-SQL)

Для создания серверов, баз данных и брандмауэров и управления ими с помощью Transact-SQL используйте следующие команды T-SQL. Эти команды можно выполнить с помощью портал Azure, [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), [Visual Studio Code](https://code.visualstudio.com/docs)или любой другой программы, которая может подключаться к серверу в базе данных SQL и ПЕРЕДАВАТЬ команды Transact-SQL. Ознакомьтесь с дополнительными сведениями об управлении [эластичными пулами](elastic-pool-overview.md).

> [!TIP]
> Краткое руководство по использованию SQL Server Management Studio в Microsoft Windows приведено в разделе [Подключайтесь к базе данных Azure SQL и создавайте запросы к ней с помощью SQL Server Management Studio](connect-query-ssms.md). Краткое руководство по использованию Visual Studio Code в macOS, Linux и Windows приведено в статье [База данных SQL Azure: подключение и запрос данных с помощью Visual Studio Code](connect-query-vscode.md).
> [!IMPORTANT]
> С помощью Transact-SQL невозможно создать или удалить сервер.

| Команда | Описание |
| --- | --- |
|[CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current&preserve-view=true)|Создает базу отдельную данных. Для создания новой базы данных необходимо подключение к базе данных master.|
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true) |Изменяет базу данных или эластичный пул. |
|[DROP DATABASE](/sql/t-sql/statements/drop-database-transact-sql)|Удаляет базу данных.|
|[sys.database_service_objectives](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Возвращает выпуск (уровень служб), Цель обслуживания (ценовая категория) и имя эластичного пула (если таковые имеются) для базы данных SQL Azure или выделенного пула SQL в Azure синапсе Analytics. При входе в базу данных master на сервере в базе данных SQL возвращает сведения обо всех базах данных. Для Azure синапсе Analytics необходимо подключиться к базе данных master.|
|[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Возвращает сведения о потреблении ресурсов ЦП, операций ввода-вывода и памяти для базы данных в базе данных SQL Azure. Одна строка существует каждые 15 секунд, даже если в базе данных нет действий.|
|[sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)|Возвращает данные об использовании ЦП и хранилища для базы данных в базе данных SQL Azure. Эти данные собираются и объединяются с пятиминутными интервалами.|
|[sys.database_connection_stats](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database)|Содержит статистику для событий подключения к базе данных SQL, предоставляя обзор успешных и неудачных попыток подключения к базе данных. |
|[sys.event_log](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database)|Возвращает успешные подключения к базе данных SQL Azure, сбои подключений и взаимоблокировки. С помощью этой информации можно отслеживать действия с базой данных в базе данных SQL или устранять возникшие неполадки.|
|[sp_set_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database)|Создает или обновляет параметры брандмауэра на уровне сервера для сервера. Эта хранимая процедура доступна только в базе данных master для имени входа субъекта серверного уровня. Правило брандмауэра уровня сервера можно будет создать с помощью Transact-SQL только после того, как пользователь, имеющий разрешения уровня Azure, создаст первое правило брандмауэра уровня сервера.|
|[sys.firewall_rules](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database)|Возвращает сведения о параметрах брандмауэра на уровне сервера, связанных с базой данных в базе данных SQL Azure.|
|[sp_delete_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database)|Удаляет параметры брандмауэра на уровне сервера с сервера. Эта хранимая процедура доступна только в базе данных master для имени входа субъекта серверного уровня.|
|[sp_set_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)|Создает или обновляет правила брандмауэра уровня базы данных для своей базы в базе данных SQL Azure. Можно настроить правила брандмауэра базы данных для базы данных master и пользовательских баз данных в Базе данных SQL. Правила брандмауэра базы данных полезны в тех случаях, когда применяются пользователи автономной базы данных. |
|[sys.database_firewall_rules](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database)|Возвращает сведения о параметрах брандмауэра уровня базы данных, связанных с базой данных в базе данных SQL Azure. |
|[sp_delete_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database)|Удаляет параметр брандмауэра уровня базы данных из базы данных. |

## <a name="rest-api"></a>REST API

Для создания серверов, баз данных и брандмауэров и управления ими используйте следующие запросы REST API.

| Команда | Описание |
| --- | --- |
|[Серверы — создание или обновление](/rest/api/sql/servers/createorupdate)|Создает или обновляет сервер.|
|[Серверы: удаление](/rest/api/sql/servers/delete)|Удаляет сервер SQL Server.|
|[Серверы: получение](/rest/api/sql/servers/get)|Получает сервер.|
|[Серверы: вывод списка](/rest/api/sql/servers/list)|Возвращает список серверов в подписке.|
|[Серверы: вывод списка по группе ресурсов](/rest/api/sql/servers/listbyresourcegroup)|Возвращает список серверов в группе ресурсов.|
|[Серверы: обновление](/rest/api/sql/servers/update)|Обновляет существующий сервер.|
|[Базы данных — создание или обновление](/rest/api/sql/databases/createorupdate)|Создает новую базу данных или обновляет имеющуюся.|
|[Базы данных: удаление](/rest/api/sql/databases/delete)|Удаляет базу данных.|
|[Базы данных: получение](/rest/api/sql/databases/get)|Получает базу данных.|
|[Базы данных — список по эластичному пулу](/rest/api/sql/databases/listbyelasticpool)|Возвращает список баз данных в пуле эластичных баз данных.|
|[Базы данных — список по серверам](/rest/api/sql/databases/listbyserver)|Возвращает список баз данных на сервере.|
|[Базы данных: обновление](/rest/api/sql/databases/update)|Обновляет имеющуюся базу данных.|
|[Правила брандмауэра — создание или обновление](/rest/api/sql/firewallrules/createorupdate)|Создает или обновляет правило брандмауэра.|
|[Правила брандмауэра — удалить](/rest/api/sql/firewallrules/delete)|Удаляет правило брандмауэра.|
|[Правила брандмауэра — получение](/rest/api/sql/firewallrules/get)|Получает правило брандмауэра.|
|[Правила брандмауэра — список по серверам](/rest/api/sql/firewallrules/listbyserver)|Возвращает список правил брандмауэра.|

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы узнать о переносе базы данных SQL Server в Azure, ознакомьтесь с разделом [Миграция базы данных SQL Server в базу данных SQL в облаке](migrate-to-database-from-sql-server.md).
- Дополнительные сведения о поддерживаемых функциях см. [здесь](features-comparison.md).
 
