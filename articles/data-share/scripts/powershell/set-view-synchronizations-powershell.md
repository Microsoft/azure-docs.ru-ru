---
title: 'Скрипт PowerShell: Настройка и Просмотр параметров синхронизации общего ресурса данных Azure'
description: Этот сценарий PowerShell задает и получает общие параметры синхронизации.
author: joannapea
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.author: joanpo
ms.openlocfilehash: 401754436739e5594c0d7baac6507f9d9984fc31
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92221287"
---
# <a name="use-powershell-to-set-azure-data-share-synchronization-settings"></a>Использование PowerShell для задания параметров синхронизации общего ресурса данных Azure

Этот сценарий PowerShell задает и получает общие параметры синхронизации.

## <a name="sample-script"></a>Пример скрипта


```powershell
# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"
$settingName = "<Synchronization setting name>"
$recurrenceInterval = "<Synchronization recurrence interval>"
$synchronizationTime = "<Synchronization time>"

# Create a new synchronization setting
New-AzDataShareSynchronizationSetting -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -ShareName $dataShareName -Name $settingName  -RecurrenceInterval $recurrenceInterval -SynchronizationTime $synchronizationTime

#List share synchronization settings
Get-AzDataShareSynchronizationSetting -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName  -ShareName $dataShareName 

#Get a specific share synchronization setting
Get-AzDataShareSynchronizationSetting -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName  -ShareName $dataShareName -Name $settingName  
```

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [New-Аздаташаресинчронизатионсеттинг](/powershell/module/az.datashare/new-azdatasharesynchronizationsetting) | Создайте синхронизацию общего ресурса. |
| [Get-Аздаташаресинчронизатионсеттинг](/powershell/module/az.datashare/get-azdatasharesynchronizationsetting) | Возвращает параметры синхронизации общей папки. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для общего ресурса Azure Data Share можно найти в [примерах PowerShell для общего доступа к данным Azure](../../samples-powershell.md).
