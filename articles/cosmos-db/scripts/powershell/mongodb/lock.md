---
title: Скрипт PowerShell для создания блокировки ресурсов для базы данных и контейнера API MongoDB в Azure Cosmos
description: Создание блокировки ресурса для базы данных и коллекции API MongoDB в Azure Cosmos
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 06/12/2020
ms.openlocfilehash: 0d7b5037e3adb4a8710d117b98e93536e8a278df
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98677953"
---
# <a name="create-a-resource-lock-for-azure-cosmos-mongodb-api-database-and-collection-using-azure-powershell"></a>Создание блокировки ресурсов для базы данных и контейнера API MongoDB в Azure Cosmos с помощью Azure PowerShell
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Для этого примера требуется модуль Azure PowerShell Az 5.4.0 или более поздней версии. Запустите `Get-Module -ListAvailable Az` для просмотра установленных версий.
Если необходимо выполнить установку, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Запустите [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) для входа Azure.

> [!IMPORTANT]
> Блокировки ресурсов не работают для изменений, внесенных пользователями, которые подключаются с помощью любого пакета SDK MongoDB, Mongoshell или портала Azure, если только учетная запись Cosmos DB сначала не будет заблокирована с помощью включения свойства `disableKeyBasedMetadataWriteAccess`. Дополнительные сведения о том, как включить это свойство, см. в разделе [Предотвращение изменений в пакетах SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/mongodb/ps-mongodb-lock.ps1 "Create, list, and remove resource locks")]

## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
|**Ресурс Azure**| |
| [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock) | Создает блокировку ресурса. |
| [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock) | Возвращает блокировку ресурса или список блокировок ресурсов. |
| [Remove-AzResourceLock](/powershell/module/az.resources/remove-azresourcelock) | Удаляет блокировку ресурса. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).