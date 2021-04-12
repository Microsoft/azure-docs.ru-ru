---
title: Скрипты PowerShell для операций с пропускной способностью (ЕЗ/с) для API Gremlin службы Azure Cosmos DB
description: Скрипты PowerShell для операций с пропускной способностью (ЕЗ/с) для API Gremlin
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
ms.date: 10/07/2020
ms.author: mjbrown
ms.openlocfilehash: 50f3f05bd06101caa096b8126225c7b49ef1f3fa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98675510"
---
# <a name="throughput-rus-operations-with-powershell-for-a-database-or-graph-for-azure-cosmos-db---gremlin-api"></a>Выполнение операций с пропускной способностью (ЕЗ/с) с помощью PowerShell для базы данных или графа в Azure Cosmos DB — API Gremlin
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Для этого примера требуется модуль Azure PowerShell Az 5.4.0 или более поздней версии. Запустите `Get-Module -ListAvailable Az` для просмотра установленных версий.
Если необходимо выполнить установку, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Запустите [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) для входа Azure.

## <a name="get-throughput"></a>Оценка пропускной способности.

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/gremlin/ps-gremlin-ru-get.ps1 "Get throughput on a database or graph for Gremlin API")]

## <a name="update-throughput"></a>Обновление данных пропускной способности

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/gremlin/ps-gremlin-ru-update.ps1 "Update throughput on a database or graph for Gremlin API")]

## <a name="migrate-throughput"></a>Переход с одной пропускной способности на другую

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/gremlin/ps-gremlin-ru-migrate.ps1 "Migrate between standard and autoscale throughput on a database or graph for Gremlin API")]

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
| [Get-AzCosmosDBGremlinDatabaseThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbgremlindatabasethroughput) | Возвращает значение пропускной способности базы данных API Gremlin. |
| [Get-AzCosmosDBGremlinGraphThroughput](/powershell/module/az.cosmosdb/get-azcosmosdbgremlingraphthroughput) | Возвращает значение пропускной способности графа API Gremlin. |
| [Update-AzCosmosDBGremlinDatabaseThroughput](/powershell/module/az.cosmosdb/update-azcosmosdbgremlindatabasethroughput) | Обновляет значение пропускной способности базы данных API Gremlin. |
| [Update-AzCosmosDBGremlinGraphThroughput](/powershell/module/az.cosmosdb/update-azcosmosdbgremlingraphthroughput) | Обновляет значение пропускной способности графа API Gremlin. |
| [Invoke-AzCosmosDBGremlinDatabaseThroughputMigration](/powershell/module/az.cosmosdb/invoke-azcosmosdbgremlindatabasethroughputmigration) | Переход с одной пропускной способности на другую для базы данных API Gremlin. |
| [Invoke-AzCosmosDBGremlinGraphThroughputMigration](/powershell/module/az.cosmosdb/invoke-azcosmosdbgremlingraphthroughputmigration) | Переход с одной пропускной способности на другую для графа API Gremlin. |
|**Группы ресурсов Azure**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).