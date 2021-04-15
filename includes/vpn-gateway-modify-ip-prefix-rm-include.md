---
title: Включить файл
description: Включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/10/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 7dd255e9767309c7b273dccfa0ee5675eed18568
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100380506"
---
Чтобы добавить дополнительные префиксы адресов:

1. Задайте переменную для локального сетевого шлюза.

   ```azurepowershell-interactive
   $local = Get-AzLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
   ```
1. Измените префиксы.

   ```azurepowershell-interactive
   Set-AzLocalNetworkGateway -LocalNetworkGateway $local `
   -AddressPrefix @('10.101.0.0/24','10.101.1.0/24','10.101.2.0/24')
   ```

Чтобы удалить префиксы адресов, используйте фрагмент кода ниже.

  Не указывайте префиксы, которые больше не нужны. В этом примере нам больше не нужен префикс 10.101.2.0/24 (из предыдущего примера), поэтому мы обновим сетевой шлюз и исключим этот префикс.

1. Задайте переменную для локального сетевого шлюза.

   ```azurepowershell-interactive
   $local = Get-AzLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
   ```
1. Укажите шлюз с обновленными префиксами.

   ```azurepowershell-interactive
   Set-AzLocalNetworkGateway -LocalNetworkGateway $local `
   -AddressPrefix @('10.101.0.0/24','10.101.1.0/24')
   ```
