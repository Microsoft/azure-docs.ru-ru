---
title: 'Сценарий PowerShell: создание учетной записи общей папки Azure Data'
description: Этот сценарий PowerShell создает новую учетную запись для общей папки данных.
author: joannapea
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.author: joanpo
ms.openlocfilehash: 4608afc16398a4cd1fa47ee1ae2cd4a4489a3f96
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92221372"
---
# <a name="use-powershell-to-create-a-data-share-account-in-azure"></a>Создание учетной записи общей папки данных в Azure с помощью PowerShell

Этот сценарий PowerShell создает новую учетную запись для общей папки данных. 

## <a name="sample-script"></a>Пример скрипта

```powershell
# Set variables with your own values
$resourceGroupName = "<Resource group name"
$location = "<Location>"
$dataShareAccountName = "<Data share account name>"

# Create a new Azure Data Share account
New-AzDataShareAccount -ResourceGroupName $resourceGroupName -Name $dataShareAccountName -Location $location
```

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [New-Аздаташареаккаунт](/powershell/module/az.datashare/new-azdatashareaccount) | Создает учетную запись общего ресурса данных. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для общего ресурса Azure Data Share можно найти в [примерах PowerShell для общего доступа к данным Azure](../../samples-powershell.md).
