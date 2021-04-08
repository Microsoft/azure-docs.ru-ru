---
title: Скрипты Azure PowerShell для операций пропускной способности (ЕЗ/с) базы данных или контейнера API SQL в Azure Cosmos DB
description: Скрипты Azure PowerShell для операций пропускной способности (ЕЗ/с) базы данных или контейнера API SQL в Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 10/07/2020
ms.author: mjbrown
ms.openlocfilehash: 73fdfa06fab42a5687e3aa354e3ba92937b92605
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98684297"
---
# <a name="throughput-rus-operations-with-powershell-for-a-database-or-container-for-azure-cosmos-db-core-sql-api"></a>Выполнение операций с пропускной способностью (ЕЗ/с) с помощью PowerShell для базы данных или контейнера в API Azure Cosmos DB Core (SQL)
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Для этого примера требуется модуль Azure PowerShell Az 5.4.0 или более поздней версии. Запустите `Get-Module -ListAvailable Az` для просмотра установленных версий.
Если необходимо выполнить установку, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Запустите [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) для входа Azure.

## <a name="get-throughput"></a>Оценка пропускной способности.

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-ru-get.ps1 "Get throughput (RU/s) for Azure Cosmos DB Core (SQL) API database or container")]

## <a name="update-throughput"></a>Обновление данных пропускной способности

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-ru-update.ps1 "Update throughput (RU/s) for an Azure Cosmos DB Core (SQL) API database or container")]

## <a name="migrate-throughput"></a>Переход с одной пропускной способности на другую

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-ru-migrate.ps1 "Migrate between standard and autoscale throughput on an Azure Cosmos DB Core (SQL) API database or container")]

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
| [Get-AzCosmosDBSqlDatabaseThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbsqldatabasethroughput) | Получение данных о пропускной способности, подготовленной в базе данных API Azure Cosmos DB Core (SQL). |
| [Get-AzCosmosDBSqlContainerThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbsqlcontainerthroughput) | Получение пропускной способности, подготовленной в контейнере API Azure Cosmos DB Core (SQL). |
| [Update-AzCosmosDBSqlDatabaseThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbsqldatabase) | Обновляет значение пропускной способности базы данных API Azure Cosmos DB Core (SQL). |
| [Update-AzCosmosDBSqlContainerThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbsqlcontainer) | Обновляет значение пропускной способности в контейнере API Azure Cosmos DB Core (SQL). |
| [Invoke-AzCosmosDBSqlDatabaseThroughputMigration](/powershell/module/az.cosmosdb/invoke-azcosmosdbsqldatabasethroughputmigration) | Переходит с одной пропускной способности на другую в базе данных API Azure Cosmos DB Core (SQL). |
| [Invoke-AzCosmosDBSqlContainerThroughputMigration](/powershell/module/az.cosmosdb/invoke-azcosmosdbsqlcontainerthroughputmigration) | Переходит с одной пропускной способности на другую в контейнере API Azure Cosmos DB Core (SQL). |
|**Группы ресурсов Azure**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).