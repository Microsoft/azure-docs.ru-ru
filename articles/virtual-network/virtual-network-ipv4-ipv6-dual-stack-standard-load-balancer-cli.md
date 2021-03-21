---
title: Развертывание двойного стека IPv6 с приложением-Load Balancer (цен. категория "Стандартный")-CLI
titlesuffix: Azure Virtual Network
description: В этой статье показано, как развернуть приложение с двумя стеками IPv6 в виртуальной сети Azure с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: 81635a5f81548f0b4edeeb62fae19c567c6942bb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932746"
---
# <a name="deploy-an-ipv6-dual-stack-application-in-azure-virtual-network---cli"></a>Развертывание приложения двойного стека IPv6 в виртуальной сети Azure с помощью интерфейса командной строки

В этой статье показано, как развернуть приложение с двойным стеком (IPv4 + IPv6) с помощью Load Balancer (цен. категория "Стандартный") в Azure, которое включает в себя двойную виртуальную сеть с подсетью с двумя стеками, Load Balancer (цен. категория "Стандартный") с двумя интерфейсными конфигурациями (IPv4 + IPv6), ВМ с сетевыми картами с двумя сетевыми конфигурациями, двойными правилами группы безопасности сети и двумя общедоступными IP

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется версия 2.0.49 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Прежде чем можно будет создать виртуальную сеть с двумя стеками, необходимо создать группу ресурсов с помощью команды [AZ Group Create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *DsResourceGroup01* в расположении *eastus* :

```azurecli-interactive
az group create \
--name DsResourceGroup01 \
--location eastus
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses-for-load-balancer"></a>Создание общедоступных IP-адресов IPv4 и IPv6 для балансировщика нагрузки
Для доступа к конечным точкам IPv4 и IPv6 в Интернете требуются общедоступные IP-адреса IPv4 и IPv6 для балансировщика нагрузки. Создайте общедоступный IP-адрес с помощью команды [az network public-ip create](/cli/azure/network/public-ip). В следующем примере создается общедоступный IP-адрес IPv4 и IPv6 с именем *dsPublicIP_v4* и *dsPublicIP_v6* в группе ресурсов *DsResourceGroup01* :

```azurecli-interactive
# Create an IPV4 IP address
az network public-ip create \
--name dsPublicIP_v4  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku STANDARD  \
--allocation-method static  \
--version IPv4

# Create an IPV6 IP address
az network public-ip create \
--name dsPublicIP_v6  \
--resource-group DsResourceGroup01  \
--location eastus \
--sku STANDARD  \
--allocation-method static  \
--version IPv6

```

## <a name="create-public-ip-addresses-for-vms"></a>Создание общедоступных IP-адресов для виртуальных машин

Для удаленного доступа к виртуальным машинам в Интернете требуются общедоступные IP-адреса IPv4 для виртуальных машин. Создайте общедоступный IP-адрес с помощью команды [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network public-ip create \
--name dsVM0_remote_access  \
--resource-group DsResourceGroup01 \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4

az network public-ip create \
--name dsVM1_remote_access  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4
```

## <a name="create-standard-load-balancer"></a>Создание Load Balancer уровня "Стандартный"

В этом разделе вы настроите сдвоенный интерфейсный IP-адрес (IPv4 и IPv6) и пул внутренних адресов для балансировщика нагрузки, а затем создадите Load Balancer (цен. категория "Стандартный").

### <a name="create-load-balancer"></a>Создание подсистемы балансировки нагрузки

Создайте Load Balancer (цен. категория "Стандартный") с помощью команды [AZ Network фунтов Create](/cli/azure/network/lb) с именем **дслб** , которая включает интерфейсный пул с именем **dsLbFrontEnd_v4**, внутренний пул с именем **dsLbBackEndPool_v4** , связанный с общедоступным IP-адресом IPv4 **dsPublicIP_v4** , созданным на предыдущем шаге. 

```azurecli-interactive
az network lb create \
--name dsLB  \
--resource-group DsResourceGroup01 \
--sku Standard \
--location eastus \
--frontend-ip-name dsLbFrontEnd_v4  \
--public-ip-address dsPublicIP_v4  \
--backend-pool-name dsLbBackEndPool_v4
```

### <a name="create-ipv6-frontend"></a>Создание внешнего интерфейса IPv6

Создайте интерфейсный IP-адрес IPV6 с помощью команды [AZ Network фунтов интерфейсного интерфейса-IP Create](/cli/azure/network/lb/frontend-ip#az-network-lb-frontend-ip-create). В следующем примере создается внешняя IP-конфигурация с именем *dsLbFrontEnd_v6* и присоединяется *dsPublicIP_v6* адрес.

```azurecli-interactive
az network lb frontend-ip create \
--lb-name dsLB  \
--name dsLbFrontEnd_v6  \
--resource-group DsResourceGroup01  \
--public-ip-address dsPublicIP_v6

```

### <a name="configure-ipv6-back-end-address-pool"></a>Настройка пула адресов IPv6 для серверной части

Создайте пулы внутренних адресов IPv6 с помощью команды [AZ Network фунтов Address-Pool Create](/cli/azure/network/lb/address-pool#az-network-lb-address-pool-create). В следующем примере создается пул адресов серверной части с именем *dsLbBackEndPool_v6*  для включения виртуальных машин с КОНФИГУРАЦИЯМИ сетевых адаптеров IPv6:

```azurecli-interactive
az network lb address-pool create \
--lb-name dsLB  \
--name dsLbBackEndPool_v6  \
--resource-group DsResourceGroup01
```

### <a name="create-a-health-probe"></a>Создание пробы работоспособности
Создайте зонд работоспособности с помощью команды [az network lb probe create](/cli/azure/network/lb/probe), чтобы отслеживать работоспособность виртуальных машин. 

```azurecli-interactive
az network lb probe create -g DsResourceGroup01  --lb-name dsLB -n dsProbe --protocol tcp --port 3389
```

### <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки

Правило балансировщика нагрузки позволяет определить распределение трафика между виртуальными машинами. Вы определяете конфигурацию внешнего IP-адреса для входящего трафика и пул внутренних IP-адресов для приема трафика, а также требуемый порт источника и назначения. 

Создайте правило балансировщика нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create). В следующем примере создаются правила балансировщика нагрузки с именами *dsLBrule_v4* и *dsLBrule_v6* и сбалансированный трафик по *TCP* -порту *80* в конфигурациях интерфейсов IPv4 и IPv6.

```azurecli-interactive
az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v4  \
--resource-group DsResourceGroup01  \
--frontend-ip-name dsLbFrontEnd_v4  \
--protocol Tcp  \
--frontend-port 80  \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v4


az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v6  \
--resource-group DsResourceGroup01 \
--frontend-ip-name dsLbFrontEnd_v6  \
--protocol Tcp  \
--frontend-port 80 \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v6

```

## <a name="create-network-resources"></a>Создание сетевых ресурсов
Перед развертыванием некоторых виртуальных машин необходимо создать вспомогательные сетевые ресурсы — группы доступности, группу безопасности сети, виртуальную сеть и виртуальные сетевые карты. 
### <a name="create-an-availability-set"></a>"Создать группу доступности"
Чтобы повысить доступность приложения, разместите виртуальные машины в группе доступности.

Создайте группу доступности с помощью команды [az vm availability-set create](/cli/azure/vm/availability-set). В следующем примере создается группа доступности с именем *дсавсет*:

```azurecli-interactive
az vm availability-set create \
--name dsAVset  \
--resource-group DsResourceGroup01  \
--location eastus \
--platform-fault-domain-count 2  \
--platform-update-domain-count 2  
```

### <a name="create-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети для правил, которые будут управлять входящим и исходящим обменом данными в виртуальной сети.

#### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети с помощью команды [AZ Network NSG Create](/cli/azure/network/nsg#az-network-nsg-create) .


```azurecli-interactive
az network nsg create \
--name dsNSG1  \
--resource-group DsResourceGroup01  \
--location eastus

```

#### <a name="create-a-network-security-group-rule-for-inbound-and-outbound-connections"></a>Создание правила группы безопасности сети для входящих и исходящих соединений

Создайте правило группы безопасности сети, разрешающее подключения по протоколу RDP через порт 3389, подключение к Интернету через порт 80 и исходящие подключения с помощью команды [AZ Network NSG правило Create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create).

```azurecli-interactive
# Create inbound rule for port 3389
az network nsg rule create \
--name allowRdpIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 100  \
--description "Allow Remote Desktop In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges 3389

# Create inbound rule for port 80
az network nsg rule create \
--name allowHTTPIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 200  \
--description "Allow HTTP In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges 80  \
--destination-address-prefixes "*"  \
--destination-port-ranges 80

# Create outbound rule

az network nsg rule create \
--name allowAllOut  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 300  \
--description "Allow All Out"  \
--access Allow  \
--protocol "*"  \
--direction Outbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges "*"
```


### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). В следующем примере создается виртуальная сеть с именем *дсвнет* с подсетями *dsSubNET_v4* и *dsSubNET_v6*.

```azurecli-interactive
# Create the virtual network
az network vnet create \
--name dsVNET \
--resource-group DsResourceGroup01 \
--location eastus  \
--address-prefixes "10.0.0.0/16" "fd00:db8:deca::/48"

# Create a single dual stack subnet

az network vnet subnet create \
--name dsSubNET \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--address-prefixes "10.0.0.0/24" "fd00:db8:deca:deed::/64" \
--network-security-group dsNSG1
```

### <a name="create-nics"></a>Создание сетевых адаптеров

Создайте виртуальные сетевые карты для каждой виртуальной машины с помощью команды [AZ Network NIC Create](/cli/azure/network/nic#az-network-nic-create). В следующем примере создается виртуальный сетевой адаптер для каждой виртуальной машины. Каждая сетевая карта имеет две конфигурации IP-адресов (1 Конфигурация IPv4, 1 Конфигурация IPv6). Создайте конфигурацию IPV6 с помощью команды [AZ Network NIC IP-config Create](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-create).
 
```azurecli-interactive
# Create NICs
az network nic create \
--name dsNIC0  \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1  \
--vnet-name dsVNET  \
--subnet dsSubNet  \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4  \
--lb-name dsLB  \
--public-ip-address dsVM0_remote_access

az network nic create \
--name dsNIC1 \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4 \
--lb-name dsLB \
--public-ip-address dsVM1_remote_access

# Create IPV6 configurations for each NIC

az network nic ip-config create \
--name dsIp6Config_NIC0  \
--nic-name dsNIC0  \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name dsNIC1 \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB
```

### <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальные машины с помощью команды [az vm create](/cli/azure/vm#az-vm-create). В приведенном ниже примере создаются две виртуальные машины и обязательные компоненты виртуальной сети, если их еще нет: 

Создайте виртуальную машину *dsVM0* следующим образом:

```azurecli-interactive
 az vm create \
--name dsVM0 \
--resource-group DsResourceGroup01 \
--nics dsNIC0 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest  
```

Создайте виртуальную машину *dsVM1* следующим образом:

```azurecli-interactive
az vm create \
--name dsVM1 \
--resource-group DsResourceGroup01 \
--nics dsNIC1 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest 
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Просмотр двух виртуальных сетей IPv6 с двумя стеками в портал Azure
Виртуальную сеть с двумя стеками IPv6 можно просмотреть в портал Azure следующим образом:
1. На панели поиска портала введите *дсвнет*.
2. Когда в результатах поиска появится пункт **myVirtualNetwork**, выберите его. Откроется страница **обзора** для виртуальной сети с двумя стеками с именем *дсвнет*. В виртуальной сети с двумя стеками показаны две сетевые карты с конфигурациями IPv4 и IPv6, расположенными в подсети с двойным стеком с именем *дссубнет*.

  ![Виртуальная сеть с двумя стеками IPv6 в Azure](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-vnet.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
 az group delete --name DsResourceGroup01
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали Load Balancer (цен. категория "Стандартный") с двойной интерфейсной IP-конфигурацией (IPv4 и IPv6). Вы также создали две виртуальные машины, которые включали сетевые карты с двумя IP-конфигурациями (IPV4 + IPv6), которые были добавлены в пул внутренних подсистем подсистемы балансировки нагрузки. Дополнительные сведения о поддержке IPv6 в виртуальных сетях Azure см. в статье [что такое IPv6 для виртуальной сети Azure?](ipv6-overview.md)
