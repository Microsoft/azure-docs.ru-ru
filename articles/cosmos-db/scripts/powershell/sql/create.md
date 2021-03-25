---
title: Сценарий PowerShell — создание базы данных и контейнера для Azure Cosmos DB с помощью API SQL
description: Сценарий Azure PowerShell — создание базы данных и контейнера для Azure Cosmos DB с помощью API SQL
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 05/13/2020
ms.author: mjbrown
ms.openlocfilehash: 2a098d3eff7bc4a8a78a880386145457b80b42d4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98675568"
---
# <a name="create-a-database-and-container-for-azure-cosmos-db---sql-api"></a>Создание базы данных и контейнера в Azure Cosmos DB — API SQL
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Для этого примера требуется модуль Azure PowerShell Az 5.4.0 или более поздней версии. Запустите `Get-Module -ListAvailable Az` для просмотра установленных версий.
Если необходимо выполнить установку, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Запустите [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) для входа Azure.

## <a name="sample-script"></a>Пример скрипта

С помощью этого сценария создается учетная запись Cosmos для API SQL (Core) в двух регионах с согласованностью на уровне сеанса, базой данных и контейнером с ключом секции, пользовательской политикой индексирования, политикой уникального ключа, сроком жизни, выделенной пропускной способностью и политикой разрешения конфликтов, реализующей подход "Сохраняются изменения, внесенные последними", с задаваемым пользователем путем разрешения конфликтов, который будет использоваться при `multipleWriteLocations=true`.

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-create.ps1 "Create an account, database, and container for SQL API")]

## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
|**Azure Cosmos DB**| |
| [New-AzCosmosDBAccount](/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | Создание учетной записи Cosmos DB. |
| [New-AzCosmosDBSqlDatabase](/powershell/module/az.cosmosdb/new-azcosmosdbsqldatabase) | Создание базы данных SQL Cosmos DB. |
| [New-AzCosmosDBSqlUniqueKey](/powershell/module/az.cosmosdb/new-azcosmosdbsqluniquekey) | Создание объекта PSSqlUniqueKey для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlUniqueKeyPolicy. |
| [New-AzCosmosDBSqlUniqueKeyPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqluniquekeypolicy) | Создание объекта PSSqlUniqueKeyPolicy для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlContainer. |
| [New-AzCosmosDBSqlCompositePath](/powershell/module/az.cosmosdb/new-azcosmosdbsqlcompositepath) | Создание объекта PSCompositePath для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlIndexingPolicy. |
| [New-AzCosmosDBSqlIncludedPathIndex](/powershell/module/az.cosmosdb/new-azcosmosdbsqlincludedpathindex) | Создание объекта PSIndexes для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlIncludedPath. |
| [New-AzCosmosDBSqlIncludedPath](/powershell/module/az.cosmosdb/new-azcosmosdbsqlincludedpath) | Создание объекта PSIncludedPath для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlIndexingPolicy. |
| [New-AzCosmosDBSqlIndexingPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqlindexingpolicy) | Создание объекта PSSqlIndexingPolicy для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlContainer. |
| [New-AzCosmosDBSqlConflictResolutionPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqlconflictresolutionpolicy) | Создание объекта PSSqlConflictResolutionPolicy для SQL в Cosmos DB, который используется в качестве параметра для командлета New-AzCosmosDBSqlContainer. |
| [New-AzCosmosDBSqlContainer](/powershell/module/az.cosmosdb/new-azcosmosdbsqlcontainer) | Создание контейнера SQL Cosmos DB. |
|**Группы ресурсов Azure**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).