---
title: Пример скрипта Azure CLI. Управление веб-трафиком | Документация Майкрософт
description: Пример скрипта Azure CLI. Управление веб-трафиком с помощью шлюза приложений и масштабируемого набора виртуальных машин.
services: application-gateway
documentationcenter: networking
author: vhorne
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 2e766bcdee2afa3ff4ed75476c619326c4c0e0c2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99591673"
---
# <a name="manage-web-traffic-using-the-azure-cli"></a>Управление веб-трафиком с помощью Azure CLI

С помощью этого скрипта создается шлюз приложений, в котором используется масштабируемый набор виртуальных машин для внутренних серверов. Затем шлюз приложений можно настроить для управления веб-трафиком. Выполнив скрипт, вы можете проверить шлюз приложений с помощью его общедоступного IP-адреса.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/application-gateway/create-vmss/create-vmss.sh "Create application gateway")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, шлюз приложений и все связанные ресурсы.

```azurecli-interactive 
az group delete --name myResourceGroupAG --yes
```

## <a name="script-explanation"></a>Описание скрипта

Чтобы создать развертывание, скрипт использует следующие команды. Для каждого элемента в таблице приведены ссылки на документацию по команде.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network vnet create](/cli/azure/network/vnet) | Создает виртуальную сеть. |
| [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) | Создает подсеть в виртуальной сети. |
| [az network public-ip create](/cli/azure/network/public-ip) | Создает общедоступный IP-адрес для шлюза приложений. |
| [az network application-gateway create](/cli/azure/network/application-gateway) | Создание шлюза приложений. |
| [az vmss create](/cli/azure/vmss) | Создает масштабируемый набор виртуальных машин. |
| [az network public-ip show](/cli/azure/network/public-ip) | Получает общедоступный IP-адрес для шлюза приложений. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure/overview).

Дополнительные примеры скриптов CLI шлюза приложений для виртуальных машин см. в [документации по виртуальным машинам Azure под управлением Windows](../cli-samples.md).
