---
title: 'Скрипт PowerShell: список существующих общих ресурсов в общей папке данных Azure'
description: Этот сценарий PowerShell выводит список и отображает подробные сведения об общих ресурсах.
services: data-share
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.openlocfilehash: 54b4b6cedc2dc09e4a9d5c4b7b937e80750e17c6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220828"
---
# <a name="use-powershell-to-view-the-details-of-a-sent-data-share"></a>Использование PowerShell для просмотра сведений о отправленной общей папке данных

Этот сценарий PowerShell перечисляет общие ресурсы данных из существующей учетной записи и получает сведения об определенном ресурсе.


## <a name="sample-script"></a>Пример скрипта

```powershell

# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"

#Lists all data shares within an account
Get-AzDataShare -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName

#Gets details of a specific data share
Get-AzDataShare -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -Name $dataShareName

```


## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [Get-Аздаташаре](/powershell/module/az.datashare/get-azdatashare) | Возвращает списки общих ресурсов в учетной записи. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для общего ресурса Azure Data Share можно найти в [примерах PowerShell для общего доступа к данным Azure](../../samples-powershell.md).
