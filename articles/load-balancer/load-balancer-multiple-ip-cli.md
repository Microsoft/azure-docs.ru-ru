---
title: Балансировка нагрузки в конфигурациях с несколькими IP-адресами с помощью Azure CLI
titleSuffix: Azure Load Balancer
description: Узнайте, как назначить несколько IP-адресов виртуальной машине с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: asudbring
ms.custom: seodec18, devx-track-azurecli
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: allensu
ms.openlocfilehash: 67f3ffa15b46eec9f352e08b0fe7c8b1bb15ac7e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103417796"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-azure-cli"></a>Балансировка нагрузки в конфигурациях с несколькими IP-адресами с помощью Azure CLI

> [!div class="op_single_selector"]
> * [Портал](load-balancer-multiple-ip.md)
> * [CLI](load-balancer-multiple-ip-cli.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)

В этой статье описывается, как использовать Azure Load Balancer в конфигурации, когда каждому дополнительному сетевому интерфейсу (сетевой карты) назначено несколько IP-адресов. В этом сценарии у нас есть две виртуальные машины под управлением Windows, оснащенные основной и дополнительной сетевыми картами. В каждом из дополнительных сетевых интерфейсов есть по две IP-конфигурации. На каждой виртуальной машине размещены веб-сайты contoso.com и fabrikam.com. Каждый веб-сайт привязан к одной из IP-конфигураций дополнительной сетевой карты. Мы используем Azure Load Balancer, чтобы предоставить два интерфейсных IP-адреса, по одному для каждого веб-сайта. Это позволит направлять трафик в соответствующую IP-конфигурацию для веб-сайта. В данном сценарии используется одинаковый номер порта для обоих внешних интерфейсов, как и для обоих IP-адресов внутренних пулов.

![Схема балансировки нагрузки для сценария](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>Инструкции по балансировке нагрузки в конфигурациях с несколькими IP-адресами

Чтобы реализовать сценарий, описанный в этой статье, сделайте следующее:

1. [Установите и настройте Azure CLI](/cli/azure/install-azure-cli), следуя инструкциям в соответствующей статье, а затем войдите в свою учетную запись Azure.
2. [Создайте группу ресурсов](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-resource-group)*contosofabrikam* следующим образом:

    ```azurecli
    az group create contosofabrikam westcentralus
    ```

3. [Создайте группу доступности](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-an-availability-set) для двух виртуальных машин. В рамках данного сценария воспользуйтесь следующей командой.

    ```azurecli
    az vm availability-set create --resource-group contosofabrikam --location westcentralus --name myAvailabilitySet
    ```

4. [Создайте виртуальную сеть](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-virtual-network-and-subnet)*myVNet* и подсеть *mySubnet*.

    ```azurecli
    az network vnet create --resource-group contosofabrikam --name myVnet --address-prefixes 10.0.0.0/16  --location westcentralus --subnet-name MySubnet --subnet-prefix 10.0.0.0/24

    ```

5. [Создайте подсистему балансировки нагрузки](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json)*mylb*.

    ```azurecli
    az network lb create --resource-group contosofabrikam --location westcentralus --name mylb
    ```

6. Создайте два динамических общедоступных IP-адреса для интерфейсных IP-конфигураций подсистемы балансировки нагрузки.

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name PublicIp1 --domain-name-label contoso --allocation-method Dynamic

    az network public-ip create --resource-group contosofabrikam --location westcentralus --name PublicIp2 --domain-name-label fabrikam --allocation-method Dynamic
    ```

7. Создайте две интерфейсные IP-конфигурации, *contosofe* и *fabrikamfe*.

    ```azurecli
    az network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp1 --name contosofe
    az network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp2 --name fabrkamfe
    ```

8. Создайте внутренние пулы адресов, *contosopool* и *fabrikampool*, [пробу](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json) - *HTTP* и правила балансировки нагрузки, *HTTPc* и *HTTPf*.

    ```azurecli
    az network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name contosopool
    azure network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name fabrikampool

    az network lb probe create --resource-group contosofabrikam --lb-name mylb --name HTTP --protocol "http" --interval 15 --count 2 --path index.html

    az network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPc --protocol tcp --probe-name http--frontend-port 5000 --backend-port 5000 --frontend-ip-name contosofe --backend-address-pool-name contosopool
    az network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPf --protocol tcp --probe-name http --frontend-port 5000 --backend-port 5000 --frontend-ip-name fabrkamfe --backend-address-pool-name fabrikampool
    ```

9. Чтобы проверить, правильно ли создана [подсистема балансировки нагрузки](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json), выполните следующую команду и проверьте выходные данные:

    ```azurecli
    az network lb show --resource-group contosofabrikam --name mylb
    ```

10. [Создайте общедоступный IP-адрес](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-public-ip-address)*myPublicIp* и [учетную запись хранения](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json)*mystorageaccont1* для первой виртуальной машины VM1, как показано ниже.

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name myPublicIP --domain-name-label mypublicdns345 --allocation-method Dynamic

    az storage account create --location westcentralus --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount1
    ```

11. [Создайте сетевые интерфейсы](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-virtual-nic) для VM1 и добавьте вторую IP-конфигурацию — *VM1-ipconfig2*. Затем [создайте виртуальную машину](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-vm), как показано ниже.

    ```azurecli
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic1 --ip-config-name NIC1-ipconfig1
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic2 --ip-config-name VM1-ipconfig1 --public-ip-name myPublicIP --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
    az network nic ip-config create --resource-group contosofabrikam --nic-name VM1Nic2 --name VM1-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
    az vm create --resource-group contosofabrikam --name VM1 --location westcentralus --os-type linux --nic-names VM1Nic1,VM1Nic2  --vnet-name VNet1 --vnet-subnet-name Subnet1 --availability-set myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount1 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>
    ```

12. Повторите шаги 10–11 для второй виртуальной машины.

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name myPublicIP2 --domain-name-label mypublicdns785 --allocation-method Dynamic
    az storage account create --location westcentralus --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount2
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic1
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic2 --ip-config-name VM2-ipconfig1 --public-ip-name myPublicIP2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
    az network nic ip-config create --resource-group contosofabrikam --nic-name VM2Nic2 --name VM2-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
    az vm create --resource-group contosofabrikam --name VM2 --location westcentralus --os-type linux --nic-names VM2Nic1,VM2Nic2 --vnet-name VNet1 --vnet-subnet-name Subnet1 --availability-set myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount2 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>
    ```

13. Наконец, необходимо настроить записи ресурсов DNS, чтобы они указывали на соответствующие интерфейсные IP-адреса подсистемы балансировки нагрузки. Домены можно разместить в Azure DNS. Дополнительные сведения об использовании Azure DNS с подсистемой балансировки нагрузки см. в разделе [Использование Azure DNS с другими службами Azure](../dns/dns-for-azure-services.md).

## <a name="next-steps"></a>Дальнейшие действия
- Узнайте больше о том, как объединять службы балансировки нагрузки, в статье [Использование служб балансировки нагрузки в Azure](../traffic-manager/traffic-manager-load-balancing-azure.md).
- Узнайте, как в Azure можно использовать журналы разных типов для управления подсистемой балансировки нагрузки и устранения неполадок в ее работе, ознакомившись со статьей [Служба анализа журналов для балансировщика нагрузки Azure](../load-balancer/load-balancer-monitor-log.md).