---
title: 'Сброс канала с ошибками — ExpressRoute: PowerShell: Azure | Документация Майкрософт'
description: Эта статья поможет вам сбросить канал ExpressRoute, который находится в неисправном состоянии.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 01/07/2021
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 84c8275b7a7257530a735e8612047ef42e2f8a7c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98011345"
---
# <a name="reset-a-failed-expressroute-circuit"></a>Сброс канала ExpressRoute после сбоя

Если операция с каналом ExpressRoute не завершается успешно, канал может переключиться в состояние Failed. Эта статья поможет вам сбросить неисправный канал ExpressRoute Azure.

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="reset-a-circuit"></a>Сброс канала

1. Установите последнюю версию командлетов PowerShell для Azure Resource Manager. Дополнительные сведения см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps).

2. Откройте консоль PowerShell с повышенными привилегиями и подключитесь к своей учетной записи. Для подключения используйте следующий пример кода:

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
3. Если у вас есть несколько подписок Azure, проверьте подписки для этой учетной записи.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```
4. Укажите подписку, которую нужно использовать.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
   ```
5. Чтобы сбросить канал, который находится в состоянии сбоя, выполните следующие команды:

   ```azurepowershell-interactive
   $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

Теперь канал должен быть работоспособным. Откройте запрос в [службу поддержки Майкрософт](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade), если канал по-прежнему находится в состоянии сбоя.

## <a name="next-steps"></a>Дальнейшие действия

Если у вас по-прежнему возникают проблемы, откройте запрос в службу поддержки [Майкрософт](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) .
