---
title: Включить файл
description: Включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 069d7af9cee0bee673c8f0b32659ce362fe4dd70
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026799"
---
### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>Изменение шлюза локальной сети gateway-ip-address

Если общедоступный IP-адрес VPN-устройства, к которому вы хотите подключиться, изменился, измените шлюз локальной сети в соответствии с изменениями. IP-адрес шлюза можно изменить, не удаляя существующее подключение VPN-шлюза (если имеется). Чтобы изменить IP-адрес шлюза, замените значения Site2 и TestRG1 собственными, используя команду [az network local-gateway update](/cli/azure/network/local-gateway).

```azurecli
az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1
```

Проверьте правильность IP-адреса в выходных данных:

```
"gatewayIpAddress": "23.99.222.170",
```