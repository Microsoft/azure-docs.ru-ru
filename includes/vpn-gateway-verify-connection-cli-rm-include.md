---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/19/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4e958026b1167d65f47bddbe5e89ec4d6fed7ee3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92217962"
---
Убедиться в успешном выполнении подключения можно с помощью команды [az network vpn-connection show](/cli/azure/network/vpn-connection). В примере параметр --name — это имя подключения, которое требуется проверить. Когда подключение устанавливается, это отображено соответствующим образом (состояние "Подключение"). После установки подключения его состояние изменяется на "Подключено".

```azurecli-interactive
az network vpn-connection show --name VNet1toSite2 --resource-group TestRG1
```
