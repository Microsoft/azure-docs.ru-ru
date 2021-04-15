---
title: Включить файл
description: Включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: f222d4a7f4724506112a47eff61ccc48354dd622
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96028417"
---
### <a name="to-modify-local-network-gateway-ip-address-prefixes---no-gateway-connection"></a><a name="noconnection"></a>Изменение префиксов IP-адресов для шлюза локальной сети при отсутствии подключения шлюза

Если нет подключения шлюза и нужно добавить или удалить префиксы IP-адресов, выполните ту же команду, что и для создания шлюза локальной сети ([az network local-gateway create](/cli/azure/network/local-gateway)). Эта команда также используется для обновления IP-адреса шлюза VPN-устройства. Чтобы перезаписать текущие параметры, укажите существующее имя шлюза локальной сети. Если используется другое имя, необходимо создать новый шлюз локальной сети вместо перезаписи существующего.

При каждом изменении нужно указать полный список префиксов, а не только те префиксы, которые вы хотите изменить. Укажите только те префиксы, которые нужно сохранить. В этом случае это 10.0.0.0/24 и 20.0.0.0/24.

```azurecli
az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 -g TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24
```

### <a name="to-modify-local-network-gateway-ip-address-prefixes---existing-gateway-connection"></a><a name="withconnection"></a>Изменение префиксов IP-адресов для шлюза локальной сети при наличии подключения шлюза

Если подключение шлюза существует и требуется добавить или удалить префиксы IP-адресов, обновите их с помощью команды [az network local-gateway update](/cli/azure/network/local-gateway). После этого VPN-подключение будет некоторое время недоступно. При изменении префиксов IP-адресов не нужно удалять VPN-шлюз.

При каждом изменении нужно указать полный список префиксов, а не только те префиксы, которые вы хотите изменить. В этом примере 10.0.0.0/24 и 20.0.0.0/24 уже существуют. Добавьте префиксы 30.0.0.0/24 и 40.0.0.0/24 и укажите все 4 при обновлении.

```azurecli
az network local-gateway update --local-address-prefixes 10.0.0.0/24 20.0.0.0/24 30.0.0.0/24 40.0.0.0/24 --name VNet1toSite2 -g TestRG1
```