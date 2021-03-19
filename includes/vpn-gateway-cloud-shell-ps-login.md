---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/29/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 24d146da7946176c92902698d0f52ae01baf79ee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93061652"
---
Если вы используете PowerShell локально, откройте консоль PowerShell с повышенными привилегиями и подключитесь к учетной записи Azure. Командлет *Connect-AzAccount* запросит учетные данные. После проверки подлинности он загрузит параметры учетной записи, чтобы они были доступны в Azure PowerShell.

Если вы используете Azure Cloud Shell вместо запуска PowerShell локально, вы заметите, что не нужно выполнять команду *Connect-азаккаунт*. Azure Cloud Shell автоматически подключается к учетной записи Azure после нажатия на кнопку " **попробовать**".

1. Если вы используете PowerShell локально, выполните вход.

   ```azurepowershell
   Connect-AzAccount
   ```

1. Если у вас есть несколько подписок Azure, запросите их список.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

1. Укажите подписку, которую нужно использовать.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```
