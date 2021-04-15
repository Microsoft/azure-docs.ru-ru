---
title: Включить файл
description: Включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/17/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 49f48c2d0bf865cf8c8fde831e7a597f8701d213
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185115"
---
Убедиться в успешном выполнении подключения можно с помощью командлета Get-AzVirtualNetworkGatewayConnection с параметром -Debug или без него. 

1. Используйте командлет из следующего примера, подставив свои значения. При появлении запроса выберите "A", чтобы выполнить команду All (Все). В примере параметр --name — это имя подключения, которое требуется проверить.

   ```azurepowershell-interactive
   Get-AzVirtualNetworkGatewayConnection -Name VNet1toSite1 -ResourceGroupName TestRG1
   ```
2. После завершения работы командлета просмотрите результаты. В следующем примере показано, что подключение установлено (состояние "Подключено"), а также указан объем полученных и отправленных данных в байтах.
   
   ```
   "connectionStatus": "Connected",
   "ingressBytesTransferred": 33509044,
   "egressBytesTransferred": 4142431
   ```