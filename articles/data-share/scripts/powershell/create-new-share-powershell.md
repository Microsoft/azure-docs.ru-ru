---
title: Сценарий PowerShell. Создание общей папки данных Azure
description: Этот сценарий PowerShell создает новую общую папку данных в существующей учетной записи общей папки данных.
author: joannapea
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.author: joanpo
ms.openlocfilehash: 36c1bdfef2afe4c34796a804784317a5e7fe12ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92221355"
---
# <a name="use-powershell-to-create-a-data-share-in-azure"></a>Создание общей папки данных в Azure с помощью PowerShell

Этот сценарий PowerShell создает новую общую папку данных в существующей учетной записи общей папки данных.

## <a name="sample-script"></a>Пример скрипта

```powershell

# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"

# Create a new Azure Data Share
New-AzDataShare -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -Name $dataShareName

```


## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [New-Аздаташаре](/powershell/module/az.datashare/new-azdatashare) | Создает общую папку данных. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для общего ресурса Azure Data Share можно найти в [примерах PowerShell для общего доступа к данным Azure](../../samples-powershell.md).
