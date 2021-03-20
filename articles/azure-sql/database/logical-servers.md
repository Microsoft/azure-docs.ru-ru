---
title: Что такое сервер в базе данных SQL Azure и Azure синапсе Analytics?
titleSuffix: ''
description: Сведения о логических серверах SQL Server, используемых базой данных SQL Azure и Azure синапсе Analytics, а так же об управлении ими.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: e0736e58983d4c28b8f2f4a20241cd36ac84fdfa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96445374"
---
# <a name="what-is-a-logical-sql-server-in-azure-sql-database-and-azure-synapse"></a>Что такое логический сервер SQL Server в базе данных SQL Azure и Azure синапсе?
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

В базе данных SQL Azure и Azure синапсе Analytics сервер — это логическая конструкция, которая выступает в качестве центральной точки администрирования для коллекции баз данных. На уровне сервера можно администрировать [имена входа](logins-create-manage.md), [правила брандмауэра](firewall-configure.md), [правила аудита](../../azure-sql/database/auditing-overview.md), [политики обнаружения угроз](threat-detection-configure.md)и [группы автоматической отработки отказа](auto-failover-group-overview.md). Сервер может находиться в разных регионах, чем группа ресурсов. Сервер должен существовать до того, как вы сможете создать базу данных в базе данных SQL Azure или базу данных хранилища данных в Azure синапсе Analytics. Все базы данных, управляемые одним сервером, создаются в том же регионе, что и сервер.

Этот сервер отличается от экземпляра SQL Server, который может быть знаком с локальным миром. В частности, нет никаких гарантий относительно расположения баз данных или базы данных хранилища данных относительно сервера, управляющего ими. Кроме того, ни база данных SQL Azure, ни Azure синапсе не предоставляют доступ на уровне экземпляра или функции. В отличие от этого, базы данных экземпляра в управляемом экземпляре являются физически размещенными — так же, как вы знакомы с SQL Server в локальной среде или в мире виртуальных машин.

При создании сервера необходимо указать учетную запись входа на сервер и пароль с правами администратора на этом сервере и всех базах данных, созданных на этом сервере. Эта начальная учетная запись является учетной записью входа SQL. База данных SQL Azure и Azure синапсе Analytics поддерживают проверку подлинности SQL и Azure Active Directory проверку подлинности для проверки подлинности. Дополнительную информацию об именах для входа и аутентификации можно найти в разделе [Предоставление доступа к базе данных SQL Azure и управление им](logins-create-manage.md). Проверка подлинности Windows не поддерживается.

Сервер в базе данных SQL и Azure синапсе:

- Создается в рамках подписки Azure, но сам логический сервер и все его ресурсы можно переместить в другую подписку.
- Выступает в качестве родительского ресурса для баз данных, эластичных пулов и хранилищ данных.
- Предоставляет пространство имен для баз данных, эластичных пулов и базы данных хранилища данных
- Является логическим контейнером со строгой семантикой времени существования — удаление сервера и удаление его баз данных, эластичных пулов и пулов СКК
- Участвует в [управлении доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/overview.md) . базы данных, эластичные пулы и база данных хранилища данных на сервере наследуют права доступа с сервера.
- — Это элемент с высоким порядком идентификации баз данных, эластичных пулов и базы данных хранилища данных для целей управления ресурсами Azure (см. схему URL-адресов для баз данных и пулов).
- Выравнивает ресурсы в регионе.
- Предоставляет конечную точку подключения к базе данных (`<serverName>`.database.windows.net).
- Предоставляет доступ к метаданным ресурсов через динамические административные представления путем подключения к базе данных master.
- Обеспечивает область для политик управления, применяемых к базам данных: имена для входа, брандмауэр, аудит, обнаружение угроз и т. д.
- Ограничен квотой в рамках родительской подписки (по умолчанию шесть серверов на подписку). Ограничения подписки см. [здесь](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Обеспечивает область для квоты базы данных и квоты DTU или виртуальных ядер для ресурсов, содержащихся на нем (например, 45 000 DTU).
- Выступает в качестве области управления версиями для компонентов, включенных для его ресурсов.
- За счет возможностей входа в систему с использованием субъекта серверного уровня может управлять всеми базами данных на сервере.
- Может содержать имена входа, аналогичные тем, которые находятся в экземплярах SQL Server в локальной среде, которым предоставлен доступ к одной или нескольким базам данных на сервере, и им могут быть предоставлены ограниченные права администратора. Дополнительные сведения см. в статье [Проверка подлинности и авторизация в базе данных SQL: предоставление доступа](logins-create-manage.md).
- Параметры сортировки по умолчанию для всех баз данных, созданных на сервере, — это `SQL_LATIN1_GENERAL_CP1_CI_AS` , где `LATIN1_GENERAL` английский (США), является `CP1` кодовой страницей 1252, `CI` не учитывает регистр и учитывает `AS` диакритические знаки.

## <a name="manage-servers-databases-and-firewalls-using-the-azure-portal"></a>Управление серверами, базами данных и брандмауэрами с помощью портал Azure

Вы можете создать группу ресурсов для сервера заранее или во время создания самого сервера. Существует несколько методов перехода к форме создания сервера SQL Server. Можно создать сервер SQL Server отдельно или при создании базы данных.

### <a name="create-a-blank-server"></a>Создание пустого сервера

Чтобы создать сервер (без базы данных, эластичного пула или базы данных хранилища данных) с помощью [портал Azure](https://portal.azure.com), перейдите к пустой форме SQL Server (логический SQL Server).

### <a name="create-a-blank-or-sample-database-in-azure-sql-database"></a>Создание пустого или образца базы данных в базе данных SQL Azure

Чтобы создать базу данных в базе данных SQL с помощью [портал Azure](https://portal.azure.com), перейдите к пустой форме базы данных SQL и укажите требуемые сведения. Вы можете создать группу ресурсов и сервер заранее или при создании самой базы данных. Можно создать пустую базу данных или пример базы данных на основе Adventure Works LT.

  ![Создание базы данных — 1](./media/logical-servers/create-database-1.png)

> [!IMPORTANT]
> См. подробнее о выборе ценовой категории для базы данных в описании моделей приобретения на основе [единиц DTU](service-tiers-dtu.md) и [виртуальных ядер](service-tiers-vcore.md).

Сведения о создании управляемого экземпляра см. в разделе [Создание управляемого экземпляра](../managed-instance/instance-create-quickstart.md) .

### <a name="manage-an-existing-server"></a>Управление существующим сервером

Для управления существующим сервером перейдите на сервер с помощью нескольких методов, таких как страница «конкретная база данных», « **серверы SQL Server** » или « **все ресурсы** ».

Для управления существующей базой данных перейдите на страницу **Базы данных SQL** и щелкните нужную базу данных. На следующем снимке экрана показано, как можно начать настройку брандмауэра уровня сервера для базы данных на странице **Обзор** для базы данных.

   ![правило брандмауэра для сервера](./media/single-database-create-quickstart/server-firewall-rule.png)

> [!IMPORTANT]
> См. подробнее о настройке свойств производительности для базы данных в описании моделей приобретения на основе [единиц DTU](service-tiers-dtu.md) и [виртуальных ядер](service-tiers-vcore.md).
> [!TIP]
> Краткое руководство по портал Azure см. в разделе [Создание базы данных SQL в портал Azure](single-database-create-quickstart.md).

## <a name="manage-servers-databases-and-firewalls-using-powershell"></a>Управление серверами, базами данных и брандмауэрами с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается, но вся будущая разработка предназначена для модуля AZ. SQL. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны.

Для создания серверов, баз данных и брандмауэров и управления ими с помощью Azure PowerShell используйте следующие командлеты PowerShell. Если вам нужно выполнить установку или обновление PowerShell, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Изучите дополнительные сведения о создании [эластичных пулов](elastic-pool-overview.md) и управлении ими.

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

> [!TIP]
> Краткое руководство по PowerShell см. [в статье Создание базы данных в базе данных SQL Azure с помощью PowerShell](single-database-create-quickstart.md). Примеры сценариев PowerShell см. в статьях [Использование PowerShell для создания базы данных в базе данных SQL Azure и настройка правила брандмауэра](scripts/create-and-configure-database-powershell.md) и [мониторинг и масштабирование базы данных в базе данных SQL Azure с помощью PowerShell](scripts/monitor-and-scale-database-powershell.md).
>

## <a name="manage-servers-databases-and-firewalls-using-the-azure-cli"></a>Управление серверами, базами данных и брандмауэрами с помощью Azure CLI

Для создания серверов, баз данных и брандмауэров и управления ими с помощью [Azure CLI](/cli/azure)используйте следующие команды [Azure CLI базы данных SQL](/cli/azure/sql/db) . Запускайте интерфейс командной строки в браузере с помощью [Cloud Shell](../../cloud-shell/overview.md) либо [установите](/cli/azure/install-azure-cli) его на платформе macOS, Linux или Windows. Изучите дополнительные сведения о создании [эластичных пулов](elastic-pool-overview.md) и управлении ими.

| Командлет | Описание |
| --- | --- |
|[az sql db create](/cli/azure/sql/db#az-sql-db-create) |Создает базу данных.|
|[az sql db list](/cli/azure/sql/db#az-sql-db-list)|Список всех баз данных, управляемых сервером, или всех баз данных в эластичном пуле.|
|[az sql db list-editions](/cli/azure/sql/db#az-sql-db-list-editions)|Выводит список доступных целевых служб и ограничений хранилища.|
|[az sql db list-usages](/cli/azure/sql/db#az-sql-db-list-usages)|Возвращает данные об использовании баз данных.|
|[az sql db show](/cli/azure/sql/db#az-sql-db-show)|Получает базу данных
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

> [!TIP]
> Краткое руководство по Azure CLI см. в разделе [Создание базы данных в базе данных SQL Azure с помощью Azure CLI](az-cli-script-samples-content-guide.md). Примеры сценариев Azure CLI см. в статьях [Использование интерфейса командной строки для создания базы данных в базе данных SQL Azure и настройка правила брандмауэра](scripts/create-and-configure-database-cli.md) и [Использование интерфейса командной строки для мониторинга и масштабирования базы данных в базе данных SQL Azure](scripts/monitor-and-scale-database-cli.md).
>

## <a name="manage-servers-databases-and-firewalls-using-transact-sql"></a>Управление серверами, базами данных и брандмауэрами с помощью Transact-SQL

Для создания серверов, баз данных и брандмауэров и управления ими с помощью Transact-SQL используйте следующие команды T-SQL. Эти команды можно выполнить с помощью портал Azure, [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), [Visual Studio Code](https://code.visualstudio.com/docs)или любой другой программы, которая может подключаться к серверу и ПЕРЕДАВАТЬ команды Transact-SQL. Ознакомьтесь с дополнительными сведениями об управлении [эластичными пулами](elastic-pool-overview.md).

> [!IMPORTANT]
> С помощью Transact-SQL невозможно создать или удалить сервер.

| Команда | Описание |
| --- | --- |
|[CREATE DATABASE (база данных SQL Azure)](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current) | Отвечает за создание базы данных в Базе данных SQL Azure. Для создания новой базы данных необходимо подключение к базе данных master.|
|[Создание базы данных (Azure синапсе)](/sql/t-sql/statements/create-database-transact-sql?view=azure-sqldw-latest) | Создает новую базу данных хранилища данных в Azure синапсе. Для создания новой базы данных необходимо подключение к базе данных master.|
| [CREATE DATABASE (база данных SQL Azure)](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current) |Изменяет базу данных или эластичный пул. |
|[ALTER DATABASE (Azure синапсе Analytics)](/sql/t-sql/statements/alter-database-transact-sql?view=sql-server-ver15)|Изменяет базу данных хранилища данных в Azure синапсе.|
|[DROP DATABASE (Transact-SQL)](/sql/t-sql/statements/drop-database-transact-sql)|Удаляет базу данных.|
|[sys.database_service_objectives (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Возвращает выпуск (уровень служб), Цель обслуживания (ценовая категория) и имя эластичного пула (если таковые имеются) для базы данных. При входе в базу данных master для сервера возвращает сведения обо всех базах данных. Для Azure синапсе необходимо подключиться к базе данных master.|
|[sys.dm_db_resource_stats (база данных SQL Azure)](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Возвращает сведения о потреблении ресурсов ЦП, операций ввода-вывода и памяти для базы данных в базе данных SQL Azure. Существует одна строка для каждых 15 секунд, даже если в базе данных не выполняется никаких действий.|
|[sys.resource_stats (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)|Возвращает данные об использовании ЦП и хранилища для базы данных в базе данных SQL Azure. Эти данные собираются и объединяются с пятиминутными интервалами.|
|[sys.database_connection_stats (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database)|Содержит статистику событий подключения базы данных для базы данных SQL Azure, предоставляя обзор успешных и неудачных попыток подключения к базе данных. |
|[sys.event_log (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database)|Возвращает успешные подключения к базе данных SQL Azure, сбои подключений и взаимоблокировки для базы данных SQL Azure. Эти сведения можно использовать для наблюдения или устранения неполадок в работе базы данных.|
|[sp_set_firewall_rule (база данных SQL Azure)](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database)|Создает или обновляет параметры брандмауэра на уровне сервера для сервера. Эта хранимая процедура доступна только в базе данных master для имени входа субъекта серверного уровня. Правило брандмауэра уровня сервера можно будет создать с помощью Transact-SQL только после того, как пользователь, имеющий разрешения уровня Azure, создаст первое правило брандмауэра уровня сервера.|
|[sys.firewall_rules (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database)|Возвращает сведения о параметрах брандмауэра на уровне сервера, связанных с сервером.|
|[sp_delete_firewall_rule (база данных SQL Azure)](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database)|Удаляет параметры брандмауэра на уровне сервера с сервера. Эта хранимая процедура доступна только в базе данных master для имени входа субъекта серверного уровня.|
|[sp_set_database_firewall_rule (база данных SQL Azure)](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)|Создает или обновляет правила брандмауэра уровня базы данных для базы данных в базе данных SQL Azure. Правила брандмауэра базы данных можно настроить для базы данных master, а также для пользовательских баз данных в базе данных SQL. Правила брандмауэра базы данных полезны в тех случаях, когда применяются пользователи автономной базы данных. Правила брандмауэра базы данных не поддерживаются в Azure синапсе.|
|[sys.database_firewall_rules (база данных SQL Azure)](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database)|Возвращает сведения о параметрах брандмауэра уровня базы данных для базы данных в базе данных SQL Azure. |
|[sp_delete_database_firewall_rule (база данных SQL Azure)](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database)|Удаляет параметр брандмауэра уровня базы данных для базы данных вашего объекта в базе данных SQL Azure. |

> [!TIP]
> Краткое руководство по использованию SQL Server Management Studio в Microsoft Windows приведено в разделе [Подключайтесь к базе данных Azure SQL и создавайте запросы к ней с помощью SQL Server Management Studio](connect-query-ssms.md). Краткое руководство по использованию Visual Studio Code в macOS, Linux и Windows приведено в статье [База данных SQL Azure: подключение и запрос данных с помощью Visual Studio Code](connect-query-vscode.md).

## <a name="manage-servers-databases-and-firewalls-using-the-rest-api"></a>Управление серверами, базами данных и брандмауэрами с помощью REST API

Для создания серверов, баз данных и брандмауэров и управления ими используйте следующие запросы REST API.

| Команда | Описание |
| --- | --- |
|[Серверы — создание или обновление](/rest/api/sql/servers/createorupdate)|Создает или обновляет сервер.|
|[Серверы: удаление](/rest/api/sql/servers/delete)|Удаляет сервер.|
|[Серверы: получение](/rest/api/sql/servers/get)|Получает сервер.|
|[Серверы: вывод списка](/rest/api/sql/servers/list)|Возвращает список серверов.|
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

- Дополнительные сведения о переносе базы данных SQL Server в базу данных SQL Azure см. в статье [Миграция в базу данных SQL](migrate-to-database-from-sql-server.md)Azure.
- Дополнительные сведения о поддерживаемых функциях см. [здесь](features-comparison.md).