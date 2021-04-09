---
title: Скрипт PowerShell для создания таблицы в API таблиц для Azure Cosmos DB
description: Узнайте, как с помощью скрипта PowerShell обновить показатель пропускной способности для базы данных или контейнера в API таблиц Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
ms.date: 05/13/2020
ms.author: mjbrown
ms.openlocfilehash: 8acf01c242f7ecc50116976d73766356d74fdd23
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98679295"
---
# <a name="create-a-table-for-azure-cosmos-db---table-api"></a>Создание таблицы для Azure Cosmos DB — API таблиц
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Для этого примера требуется модуль Azure PowerShell Az 5.4.0 или более поздней версии. Запустите `Get-Module -ListAvailable Az` для просмотра установленных версий.
Если необходимо выполнить установку, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Запустите [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) для входа Azure.

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/table/ps-table-create.ps1 "Create a table for Table API")]

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
| [New-AzCosmosDBTable](/powershell/module/az.cosmosdb/new-azcosmosdbtable) | Создает таблицу API таблиц. |
|**Группы ресурсов Azure**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).