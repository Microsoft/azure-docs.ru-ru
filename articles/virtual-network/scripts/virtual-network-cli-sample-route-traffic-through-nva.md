---
title: Пример скрипта Azure CLI для маршрутизации трафика через виртуальный сетевой модуль
description: Маршрутизация трафика через виртуальный сетевой модуль брандмауэра. Пример скрипта Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 964fda8168867c115502c7262dc1d41e55075866
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91317656"
---
# <a name="route-traffic-through-a-network-virtual-appliance---azure-cli-script-sample"></a>Пример сценария Azure CLI для маршрутизации трафика через виртуальный сетевой модуль

В этом примере скрипта создается виртуальная сеть с интерфейсной и внутренней подсетями. Скрипт также создает виртуальную машину с IP-пересылкой, которая позволяет маршрутизировать трафик между двумя подсетями. После выполнения скрипта вы можете развернуть на виртуальной машине программное обеспечение сети, например брандмауэр.

Вы можете выполнить скрипт из Azure [Cloud Shell](https://shell.azure.com/bash) или из локальной установки Azure CLI. Если вы используете CLI локально, для этого скрипта требуется версия 2.0.28 или выше. Выполните командлет `az --version`, чтобы узнать установленную версию. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). Если интерфейс командной строки запущен локально, необходимо также выполнить командлет `az login`, чтобы создать подключение к Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.sh "Route traffic through a network virtual appliance")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, виртуальную машину и все связанные с ней ресурсы:

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, виртуальной сети и групп безопасности сети этот скрипт использует следующие команды. Для каждой команды в следующей таблице приведены ссылки на соответствующую документацию:

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network vnet create](/cli/azure/network/vnet) | Создает виртуальную сеть Azure и интерфейсную подсеть. |
| [az network subnet create](/cli/azure/network/vnet/subnet) | Создает внутреннюю подсеть и подсеть DMZ. |
| [az network public-ip create](/cli/azure/network/public-ip) | Создает общедоступный IP-адрес для доступа к виртуальной машине из Интернета. |
| [az network nic create](/cli/azure/network/nic) | Создает виртуальный сетевой интерфейс и включает IP-пересылку для него. |
| [az network nsg create](/cli/azure/network/nsg) | Создает группу безопасности сети (NSG). |
| [az network nsg rule create](/cli/azure/network/nsg/rule) | Создает правила группы безопасности сети, которые разрешают входящий трафик по портам HTTP и HTTPS к виртуальной машине. |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet)| Связывает группы безопасности сети и таблицы маршрутов с подсетями. |
| [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create)| Создает таблицу маршрутов для всех маршрутов. |
| [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create)| Создает маршруты для маршрутизации трафика между подсетями и Интернетом через виртуальную машину. |
| [az vm create](/cli/azure/vm) | Создает виртуальную машину и присоединяет к ней сетевой адаптер. Эта команда также указывает образ виртуальной машины и учетные данные администратора. |
| [az group delete](/cli/azure/group) | Удаляет группу ресурсов и все содержащиеся в ней ресурсы. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов см. на [странице с примерами CLI для виртуальных сетей](../cli-samples.md).
