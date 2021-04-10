---
title: Распределение нагрузки виртуальных машин в пределах зон доступности с помощью Azure CLI — Azure Load Balancer
description: В этом примере скрипта Azure CLI показано, как распределить нагрузку трафика на виртуальных машинах в пределах зон доступности
documentationcenter: load-balancer
author: asudbring
ms.service: load-balancer
ms.devlang: azurecli
ms.topic: sample
ms.workload: infrastructure
ms.date: 06/14/2018
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: e30c90152151dc51f74fb889b16c92208b1edf3d
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106056051"
---
# <a name="azure-cli-script-example-load-balance-vms-across-availability-zones"></a>Пример скрипта Azure CLI. Балансировка нагрузки виртуальных машин в пределах зон доступности

Этот пример скрипта Azure CLI позволяет создать все необходимые компоненты для запуска нескольких виртуальных машин Ubuntu, настроенных в высокодоступной конфигурации с распределением нагрузки. После выполнения этого сценария будут созданы три виртуальные машины в пределах всех зон доступности. Доступ к ним можно будет получить через Azure Load Balancer. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

```azurecli-interactive
  #!/bin/bash

  # Create a resource group.
   az group create \
    --name myResourceGroup \
    --location westeurope

  # Create a virtual network.
   az network vnet create \
    --resource-group myResourceGroup \
    --location westeurope \
    --name myVnet \
    --subnet-name mySubnet

  # Create a zonal Standard public IP address.
   az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --sku Standard    

  # Create an Azure Load Balancer.
   az network lb create \
    --resource-group myResourceGroup \
    --name myLoadBalancer \
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --sku Standard

  # Creates an LB probe on port 80.
   az network lb probe create \
    --resource-group myResourceGroup \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80

  # Creates an LB rule for port 80.
   az network lb rule create \
    --resource-group myResourceGroup \
    --lb-name myLoadBalancer \
    --name myLoadBalancerRuleWeb \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe

  # Create three NAT rules for port 22.
   for i in `seq 1 3`; do
    az network lb inbound-nat-rule create \
     --resource-group myResourceGroup \
     --lb-name myLoadBalancer \
     --name myLoadBalancerRuleSSH$i \
     --protocol tcp \
     --frontend-port 422$i \
     --backend-port 22 \
     --frontend-ip-name myFrontEndPool
   done

  # Create a network security group
   az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup

  # Create a network security group rule for port 22.
   az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleSSH \
    --protocol tcp \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*'  \
    --destination-address-prefix '*' \
    --destination-port-range 22 \
    --access allow \
    --priority 1000

  # Create a network security group rule for port 80.
    az network nsg rule create \
     --resource-group myResourceGroup \
     --nsg-name myNetworkSecurityGroup \
     --name myNetworkSecurityGroupRuleHTTP \
     --protocol tcp \
     --direction inbound \
     --source-address-prefix '*' \
     --source-port-range '*' \
     --destination-address-prefix '*' \
     --destination-port-range 80 \
     --access allow \
     --priority 2000

  # Create three virtual network cards and associate with load balancer and NSG.
  for i in `seq 1 3`; do
   az network nic create \
     --resource-group myResourceGroup \
     --name myNic$i \
     --vnet-name myVnet \
     --subnet mySubnet \
     --network-security-group myNetworkSecurityGroup \
     --lb-name myLoadBalancer \
     --lb-address-pools myBackEndPool \
     --lb-inbound-nat-rules myLoadBalancerRuleSSH$i
  done

# Create three virtual machines, this creates SSH keys if not present.
for i in `seq 1 3`; do
  az vm create \
    --resource-group myResourceGroup \
    --name myVM$i \
    --zone $i \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
done

```

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, виртуальную машину и все связанные с ней ресурсы.

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, виртуальной машины, группы доступности, балансировщика нагрузки и всех связанных ресурсов этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) | Создает виртуальную сеть и подсеть Azure. |
| [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) | Создает общедоступный IP-адрес со статическим IP-адресом и связанным DNS-именем. |
| [az network lb create](/cli/azure/network/lb#az-network-lb-create) | Создает службу Azure Load Balancer. |
| [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create) | Создает зонд подсистемы балансировки нагрузки. Зонд подсистемы балансировки нагрузки используется для мониторинга каждой виртуальной машины в наборе подсистемы балансировки нагрузки. Если любая виртуальная машина становится недоступной, к ней не направляется трафик. |
| [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) | Создает правило подсистемы балансировки нагрузки. В этом примере создается правило для порта 80. Так как трафик HTTP поступает в подсистему балансировки нагрузки, он перенаправляется на порт 80 виртуальных машин в наборе подсистемы балансировки нагрузки. |
| [az network lb inbound-nat-rule create](/cli/azure/network/lb/inbound-nat-rule#az-network-lb-inbound-nat-rule-create) | Создает правило преобразования сетевых адресов (NAT) подсистемы балансировки нагрузки.  Правила NAT сопоставляют порт подсистемы балансировки нагрузки с портом виртуальной машины. В этом примере создается правило NAT для SSH-трафика к каждой виртуальной машине в наборе подсистемы балансировки нагрузки.  |
| [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) | Создает группу безопасности сети (NSG), которая выполняет роль периметра безопасности между Интернетом и виртуальной машиной. |
| [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) | Создает правило NSG, разрешающее входящий трафик. В этом примере открывается порт 22 для трафика SSH. |
| [az network nic create](/cli/azure/network/nic#az-network-nic-create) | Создает виртуальную сетевую карту и подключает ее к виртуальной сети, подсети и группе безопасности сети. |
| [az vm create](/cli/azure/vm#az-vm-create) | Создает виртуальную машину и подключает ее к сетевой карте, виртуальной сети, подсети и группе безопасности сети. Эта команда также указывает образ виртуальной машины, который будет использоваться, и учетные данные администратора.  |
| [az group delete](/cli/azure/vm/extension#az-vm-extension-set) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для сетей Azure см. в [документации по сетям Azure](../cli-samples.md).