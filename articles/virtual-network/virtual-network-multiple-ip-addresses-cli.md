---
title: Назначение виртуальной машине несколько IP-адресов с помощью Azure CLI
titlesuffix: Azure Virtual Network
description: Узнайте, как назначить виртуальной машине несколько IP-адресов с помощью интерфейса командной строки Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/17/2016
ms.author: allensu
ms.openlocfilehash: 0b5e8f28be9960a2bb4e1406a293ab5e03962681
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98680575"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-the-azure-cli"></a>Назначение виртуальной машине нескольких IP-адресов с помощью Azure CLI

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

В этой статье описывается создание виртуальной машины с помощью модели развертывания Azure Resource Manager с использованием Azure CLI. Для ресурсов, созданных с помощью классической модели развертывания, нельзя назначить несколько IP-адресов. Дополнительные сведения о моделях развертывания Azure см. в статье [Azure Resource Manager vs. classic deployment: Understand deployment models and the state of your resources](../azure-resource-manager/management/deployment-models.md) (Развертывание с помощью Azure Resource Manager и классическое развертывание. Общие сведения о моделях развертывания и состоянии ресурсов).

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name="create-a-vm-with-multiple-ip-addresses"></a><a name = "create"></a>Создание виртуальной машины с несколькими IP-адресами

Ниже мы пошагово создадим пример виртуальной машины с несколькими IP-адресами, как описано в нашем сценарии. Измените имена переменных в кавычках и типы IP-адресов в соответствии с требованиями своей реализации. 

1. Установите [Azure CLI](/cli/azure/install-azure-cli), если вы еще этого не сделали.
2. Создайте пару открытого и закрытого ключей SSH для виртуальных машин Linux, выполнив действия, описанные в [этой статье](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
3. Выполните вход из командной оболочки, применив команду `az login`, и выберите используемую подписку.
4. Создайте виртуальную машину, выполнив следующий скрипт на компьютере Mac или Linux. Скрипт создает группу ресурсов, одну виртуальную сеть, одну сетевую карту с тремя конфигурациями IP-адреса и виртуальную машину с двумя подключенными сетевыми картами. Сетевая карта, общедоступный IP-адрес, виртуальная сеть и виртуальная машина должны находиться в одном расположении и в одной подписке. Хотя эти ресурсы и не должны находиться в одной группе ресурсов, в следующем скрипте они находятся в одной группе ресурсов.

```azurecli
    
#!/bin/sh
    
RgName="myResourceGroup"
Location="westcentralus"
az group create --name $RgName --location $Location
    
# Create a public IP address resource with a static IP address using the `--allocation-method Static` option. If you
# do not specify this option, the address is allocated dynamically. The address is assigned to the resource from a pool
# of IP addresses unique to each Azure region. Download and view the file from
# https://www.microsoft.com/en-us/download/details.aspx?id=41653 that lists the ranges for each region.

PipName="myPublicIP"

# This name must be unique within an Azure location.
DnsName="myDNSName"

az network public-ip create \
--name $PipName \
--resource-group $RgName \
--location $Location \
--dns-name $DnsName\
--allocation-method Static

# Create a virtual network with one subnet

VnetName="myVnet"
VnetPrefix="10.0.0.0/16"
VnetSubnetName="mySubnet"
VnetSubnetPrefix="10.0.0.0/24"

az network vnet create \
--name $VnetName \
--resource-group $RgName \
--location $Location \
--address-prefix $VnetPrefix \
--subnet-name $VnetSubnetName \
--subnet-prefix $VnetSubnetPrefix

# Create a network interface connected to the subnet and associate the public IP address to it. Azure will create the
# first IP configuration with a static private IP address and will associate the public IP address resource to it.

NicName="MyNic1"
az network nic create \
--name $NicName \
--resource-group $RgName \
--location $Location \
--subnet $VnetSubnet1Name \
--private-ip-address 10.0.0.4
--vnet-name $VnetName \
--public-ip-address $PipName
    
# Create a second public IP address, a second IP configuration, and associate it to the NIC. This configuration has a
# static public IP address and a static private IP address.

az network public-ip create \
--resource-group $RgName \
--location $Location \
--name myPublicIP2 \
--dns-name mypublicdns2 \
--allocation-method Static

az network nic ip-config create \
--resource-group $RgName \
--nic-name $NicName \
--name IPConfig-2 \
--private-ip-address 10.0.0.5 \
--public-ip-name myPublicIP2

# Create a third IP configuration, and associate it to the NIC. This configuration has  static private IP address and   # no public IP address.

az network nic ip-config create \
--resource-group $RgName \
--nic-name $NicName \
--private-ip-address 10.0.0.6 \
--name IPConfig-3

# Note: Though this article assigns all IP configurations to a single NIC, you can also assign multiple IP configurations
# to any NIC in a VM. To learn how to create a VM with multiple NICs, read the Create a VM with multiple NICs 
# article: https://docs.microsoft.com/azure/virtual-network/virtual-network-deploy-multinic-arm-cli.

# Create a VM and attach the NIC.

VmName="myVm"

# Replace the value for the following **VmSize** variable with a value from the
# https://docs.microsoft.com/azure/virtual-machines/sizes article. The script fails if the VM size
# is not supported in the location you select. Run the `azure vm sizes --location eastcentralus` command to get a full list
# of VMs in US West Central, for example.

VmSize="Standard_DS1"

# Replace the value for the OsImage variable value with a value for *urn* from the output returned by entering the
# `az vm image list` command.

OsImage="credativ:Debian:8:latest"

Username="adminuser"

# Replace the following value with the path to your public key file. If you're creating a Windows VM, remove the following
# line and you'll be prompted for the password you want to configure for the VM.

SshKeyValue="~/.ssh/id_rsa.pub"

az vm create \
--name $VmName \
--resource-group $RgName \
--image $OsImage \
--location $Location \
--size $VmSize \
--nics $NicName \
--admin-username $Username \
--ssh-key-value $SshKeyValue
```

Помимо виртуальной машины с сетевой картой и конфигураций из трех IP-адресов сценарий также создает следующие ресурсы.

- Один управляемый диск уровня "Премиум" по умолчанию. Вы можете создать диск другого типа. Дополнительные сведения см. в статье [Краткое руководство по созданию виртуальной машины Linux с помощью Azure CLI](../virtual-machines/linux/quick-create-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Виртуальная сеть с одной подсетью и двумя общедоступными IP-адресами. Кроме того, можно использовать *имеющиеся* виртуальные сети, подсети, сетевые карты или общедоступные IP-адреса. Чтобы узнать, как использовать имеющиеся сетевые ресурсы, а не создавать дополнительные, введите `az vm create -h`.

За общедоступные IP-адреса взимается номинальная плата. Дополнительные сведения о ценах на IP-адреса см. на странице [Цены на IP-адреса](https://azure.microsoft.com/pricing/details/ip-addresses). Число общедоступных IP-адресов, которые можно использовать в одной подписке, ограничено. Сведения об ограничениях см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

Создав виртуальную машину, введите команду `az network nic show --name MyNic1 --resource-group myResourceGroup`, чтобы просмотреть конфигурацию сетевой карты. Введите команду `az network nic ip-config list --nic-name MyNic1 --resource-group myResourceGroup --output table` для просмотра списка конфигураций IP-адресов, связанных с сетевой картой.

Добавьте в операционную систему виртуальной машины частные IP-адреса, выполнив шаги, соответствующие вашей операционной системе, которые описаны в разделе [Добавление IP-адресов в операционную систему виртуальной машины](#os-config) этой статьи.

## <a name="add-ip-addresses-to-a-vm"></a><a name="add"></a>Добавление IP-адресов в виртуальную машину

Выполнив следующие шаги, вы сможете назначить дополнительные частные или общедоступные IP-адреса существующему сетевому интерфейсу Azure. Примеры созданы на основе [сценария](#scenario), описанного в этой статье.

1. Откройте командную оболочку и в рамках одного сеанса командной строки выполните все действия, описанные в этом разделе. Если Azure CLI еще не установлен и не настроен, выполните инструкции из статьи об [установке Azure CLI](/cli/azure/install-az-cli2?toc=%2fazure%2fvirtual-network%2ftoc.json), а затем войдите в учетную запись Azure, выполнив команду `az-login`.

2. Выполните действия, описанные в одном из следующих разделов, с учетом задач.

    **Добавление частного IP-адреса**
    
    Чтобы добавить к сетевой карте частный IP-адрес, создайте конфигурацию IP-адресов с помощью следующей команды. Статический IP-адрес должен быть свободен в используемой подсети.

    ```azurecli
    az network nic ip-config create \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --private-ip-address 10.0.0.7 \
    --name IPConfig-4
    ```
    
    Вы можете создать любое количество конфигураций, указывая для них уникальные имена и уникальные частные IP-адреса (если используются статические IP-адреса).

    **Добавление общедоступного IP-адреса**
    
    Чтобы добавить общедоступный IP-адрес, его нужно связать с новой или имеющейся IP-конфигурацией. Выполните шаги, описанные в одном из следующих разделов, в соответствии с задачами.

    За общедоступные IP-адреса взимается номинальная плата. Дополнительные сведения о ценах на IP-адреса см. на странице [Цены на IP-адреса](https://azure.microsoft.com/pricing/details/ip-addresses). Число общедоступных IP-адресов, которые можно использовать в одной подписке, ограничено. Сведения об ограничениях см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

    - **Связывание ресурса с новой IP-конфигурацией**
    
        Чтобы добавить общедоступный IP-адрес в новую IP-конфигурацию, необходимо добавить и частный IP-адрес, так как все IP-конфигурации должны иметь частный IP-адрес. В конфигурацию можно добавить имеющийся ресурс общедоступного IP-адреса или создать новый. Чтобы создать ресурс, выполните следующую команду:
    
        ```azurecli
        az network public-ip create \
        --resource-group myResourceGroup \
        --location westcentralus \
        --name myPublicIP3 \
        --dns-name mypublicdns3
        ```

        Чтобы создать новую конфигурацию IP с частным статическим IP-адресом и связанным ресурсом общедоступного IP-адреса *myPublicIP3*, введите следующую команду:

        ```azurecli
        az network nic ip-config create \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --name IPConfig-5 \
        --private-ip-address 10.0.0.8
        --public-ip-address myPublicIP3
        ```

    - **Связывание ресурса с имеющейся IP-конфигурацией** Ресурс общедоступного IP-адреса может быть связан только с IP-конфигурацией, у которой еще нет такого ресурса. Чтобы определить, связан ли с конкретной IP-конфигурацией какой-либо общедоступный IP-адрес, выполните следующую команду:

        ```azurecli
        az network nic ip-config list \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --query "[?provisioningState=='Succeeded'].{ Name: name, PublicIpAddressId: publicIpAddress.id }" --output table
        ```

        Возвращаемые выходные данные:
    
        ```output
        Name        PublicIpAddressId
        
        ipconfig1   /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
        IPConfig-2  /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
        IPConfig-3
        ```

        Столбец **PublicIpAddressId** для конфигурации *IpConfig-3* в выходных данных пуст. Это означает, что в настоящее время с этой конфигурацией не связан никакой общедоступный IP-адрес. Вы можете добавить к конфигурации IpConfig-3 имеющийся ресурс общедоступного IP-адреса или создать новый ресурс, выполнив следующую команду:

        ```azurecli
        az network public-ip create \
        --resource-group  myResourceGroup
        --location westcentralus \
        --name myPublicIP3 \
        --dns-name mypublicdns3 \
        --allocation-method Static
        ```
    
        Чтобы связать ресурс общедоступного IP-адреса с имеющейся IP-конфигурацией с именем *IPConfig-3*, выполните следующую команду:
    
        ```azurecli
        az network nic ip-config update \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --name IPConfig-3 \
        --public-ip myPublicIP3
        ```

3. Просмотрите идентификаторы ресурсов частных и общедоступных IP-адресов, назначенных сетевой карте. Для этого введите следующую команду.

    ```azurecli
    az network nic ip-config list \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --query "[?provisioningState=='Succeeded'].{ Name: name, PrivateIpAddress: privateIpAddress, PrivateIpAllocationMethod: privateIpAllocationMethod, PublicIpAddressId: publicIpAddress.id }" --output table
    ```

    Возвращаемые выходные данные: <br>

    ```output
    Name        PrivateIpAddress    PrivateIpAllocationMethod   PublicIpAddressId
    
    ipconfig1   10.0.0.4            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
    IPConfig-2  10.0.0.5            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
    IPConfig-3  10.0.0.6            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP3
    ```
    
4. Добавьте в операционную систему виртуальной машины частные IP-адреса, которые вы ранее назначили сетевой карте. Для этого выполните инструкции из раздела [Добавление IP-адресов в операционную систему виртуальной машины](#os-config) этой статьи. Не добавляйте в операционную систему общедоступные IP-адреса.

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]