---
title: 'Сценарий PowerShell: принятие приглашения из общей папки данных Azure'
description: Этот сценарий PowerShell принимает приглашения от существующей общей папки данных.
services: data-share
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.openlocfilehash: 24091bafac7ad5c558b975d52064f12715b3d622
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92221406"
---
# <a name="use-powershell-to-accept-a-data-share-invitation"></a>Использование PowerShell для принятия приглашения для общего доступа к данным

Этот сценарий PowerShell принимает приглашения, отправленные потребителю.

## <a name="sample-script"></a>Пример скрипта
```powershell
#List invitations sent to a consumer
Get-AzDataShareInvitation

# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"
$invitationId = "<Invitation id>"

#Accept a specific invitation by creating a share subscription
New-AzDataShareSubscription -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -Name $dataShareName -InvitationId $invitationId

```

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [Get-Аздаташареинвитатион](/powershell/module/az.datashare/get-azdatashareinvitation) | Получение и перечисление отправленных приглашений на обмен данными. |
| [New-Аздаташаресубскриптион](/powershell/module/az.datashare/get-azdatasharesubscription) | Создайте подписку на общую папку данных. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для общего ресурса Azure Data Share можно найти в [примерах PowerShell для общего доступа к данным Azure](../../samples-powershell.md).
