---
title: Пример сценария Azure CLI. Балансировка нагрузки на нескольких веб-сайтах с помощью Azure CLI | Документация Майкрософт
description: Пример сценария Azure CLI для балансировки нагрузки нескольких веб-сайтов на одной виртуальной машине.
services: load-balancer
documentationcenter: load-balancer
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: allensu
ms.openlocfilehash: 7a3bd4cab8123814fe360fe4ab724650c785e9f1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98234121"
---
# <a name="load-balance-multiple-websites"></a>Балансировка нагрузки на нескольких веб-сайтах

Этот пример сценария создает виртуальную сеть с двумя виртуальными машинами, которые входят в группу доступности. Подсистема балансировки нагрузки направляет трафик с двух отдельных IP-адресов на две виртуальные машины. После выполнения сценария можно развернуть ПО веб-сервера на виртуальных машин и разместить несколько веб-сайтов, каждый из которых имеет собственный IP-адрес.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта


[!code-azurecli-interactive[main](../../../cli_scripts/load-balancer/load-balance-multiple-web-sites-vm/load-balance-multiple-web-sites-vm.sh  "Load balance multiple web sites")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, виртуальную машину и все связанные с ней ресурсы.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, виртуальной сети, подсистемы балансировки нагрузки и всех связанных ресурсов этот сценарий использует приведенные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network vnet create](/cli/azure/network/vnet) | Создает виртуальную сеть и подсеть Azure. |
| [az network public-ip create](/cli/azure/network/public-ip) | Создает общедоступный IP-адрес со статическим IP-адресом и связанным DNS-именем. |
| [az network lb create](/cli/azure/network/lb) | Создает службу Azure Load Balancer. |
| [az network lb probe create](/cli/azure/network/lb/probe) | Создает зонд подсистемы балансировки нагрузки. Зонд подсистемы балансировки нагрузки используется для мониторинга каждой виртуальной машины в наборе подсистемы балансировки нагрузки. Если любая виртуальная машина становится недоступной, к ней не направляется трафик. |
| [az network lb rule create](/cli/azure/network/lb/rule) | Создает правило подсистемы балансировки нагрузки. В этом примере создается правило для порта 80. Так как трафик HTTP поступает в подсистему балансировки нагрузки, он перенаправляется на порт 80 одной из виртуальных машин в наборе подсистемы балансировки нагрузки. |
| [az network lb frontend-ip create](/cli/azure/network/lb/frontend-ip) | Создает интерфейсный IP-адрес для подсистемы балансировки нагрузки. |
| [az network lb address-pool create](/cli/azure/network/lb/address-pool) | Создает внутренний пул адресов. |
| [az network nic create](/cli/azure/network/nic) | Создает виртуальную сетевую карту и подключает ее к виртуальной сети и подсети. |
| [az vm availability-set create](/cli/azure/network/lb/rule) | Создает группу доступности. Группы доступности обеспечивают непрерывную работу приложения, распределяя виртуальные машины по физическим ресурсам. Таким образом, в случае сбоя он не затронет весь набор ресурсов. |
| [az network nic ip-config create](/cli/azure/network/nic/ip-config) | Создает IP-конфигурацию. Для подписки должна быть включена функция Microsoft.Network/AllowMultipleIpConfigurationsPerNic. В качестве основной IP-конфигурации каждой сетевой карте можно назначить только одну конфигурацию. Для этого используется флаг --make-primary. |
| [az vm create](/cli/azure/vm/availability-set) | Создает виртуальную машину и подключает ее к сетевой карте, виртуальной сети, подсети и группе безопасности сети. Эта команда также указывает образ виртуальной машины, который будет использоваться, и учетные данные администратора.  |
| [az group delete](/cli/azure/vm/extension) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для сетей см. в статье [Примеры Azure CLI](../cli-samples.md?toc=%2fazure%2fnetworking%2ftoc.json).