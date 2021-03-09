---
title: Создание среды Linux с Azure CLI
description: Узнайте, как с помощью Azure CLI создать "с нуля" хранилище, виртуальную машину Linux, виртуальную сеть и подсеть, балансировщик нагрузки, сетевую карту, общедоступный IP-адрес и группу безопасности сети.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 12/14/2017
ms.author: cynthn
ms.openlocfilehash: e8c21df3b783241a3a4bfdbe28b28b2bceb89b4d
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102509121"
---
# <a name="create-a-complete-linux-virtual-machine-with-the-azure-cli"></a>Создание полнофункциональной виртуальной машины Linux с помощью Azure CLI
Чтобы быстро создать виртуальную машину в Azure, можно использовать одну команду Azure CLI, использующую значения по умолчанию для создания любых необходимых вспомогательных ресурсов. Ресурсы, такие как виртуальная сеть, общедоступный IP-адрес и правила группы сетевой безопасности, создаются автоматически. Чтобы получить дополнительные возможности управления в рабочей среде, можно создать эти ресурсы заранее и затем добавлять в них виртуальные машины. В этой статье описывается, как создать виртуальную машину и каждый из вспомогательных ресурсов по одному.

Обязательно установите последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войдите в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index).

В следующих примерах замените имена параметров собственными значениями. Примеры имен параметров: *myResourceGroup*, *myVnet* и *myVM*.

## <a name="create-resource-group"></a>Создать группу ресурсов
Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов нужно создать перед виртуальной машиной и вспомогательными ресурсами виртуальной сети. Создайте группу ресурсов с помощью команды [AZ Group Create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

По умолчанию выходные данные команд Azure CLI в формате JSON (JavaScript Object Notation). Например, чтобы изменить назначение выходных данных на таблицу или список, используйте команду [az configure --output](/cli/azure/reference-index). Параметр `--output` можно добавлять к любой команде — он один раз изменяет формат выходных данных. В следующем примере показаны выходные данные JSON из команды `az group create`:

```json                       
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
  "location": "eastus",
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-a-virtual-network-and-subnet"></a>Создание виртуальной сети и подсети
Затем создается виртуальная сеть в Azure и подсеть, в которой можно создать виртуальные машины. Используйте команду [az network vnet create](/cli/azure/network/vnet), чтобы создать виртуальную сеть с именем *myVnet* с префиксом адреса *192.168.0.0/16*. Кроме того, можно добавить подсеть с именем *mySubnet* с префиксом адреса *192.168.1.0/24*:

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

В выходных данных отображается подсеть, логически созданная внутри виртуальной сети:

```json
{
  "addressSpace": {
    "addressPrefixes": [
      "192.168.0.0/16"
    ]
  },
  "dhcpOptions": {
    "dnsServers": []
  },
  "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
  "location": "eastus",
  "name": "myVnet",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "ed62fd03-e9de-430b-84df-8a3b87cacdbb",
  "subnets": [
    {
      "addressPrefix": "192.168.1.0/24",
      "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
      "ipConfigurations": null,
      "name": "mySubnet",
      "networkSecurityGroup": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceNavigationLinks": null,
      "routeTable": null
    }
  ],
  "tags": {},
  "type": "Microsoft.Network/virtualNetworks",
  "virtualNetworkPeerings": null
}
```


## <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса
Теперь создайте общедоступный IP-адрес с помощью команды [az network public-ip create](/cli/azure/network/public-ip). Этот общедоступный IP-адрес позволяет подключаться к виртуальной машине через Интернет. Так как адрес по умолчанию является динамическим, создайте запись DNS с параметром `--domain-name-label`. В следующем примере создается общедоступный IP-адрес *myPublicIP* с DNS-именем *mypublicdns*. Так как DNS-имя должно быть уникальным, укажите собственное уникальное имя:

```azurecli
az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --dns-name mypublicdns
```

Выходные данные:

```json
{
  "publicIp": {
    "dnsSettings": {
      "domainNameLabel": "mypublicdns",
      "fqdn": "mypublicdns.eastus.cloudapp.azure.com",
      "reverseFqdn": null
    },
    "etag": "W/\"2632aa72-3d2d-4529-b38e-b622b4202925\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": null,
    "ipConfiguration": null,
    "location": "eastus",
    "name": "myPublicIP",
    "provisioningState": "Succeeded",
    "publicIpAddressVersion": "IPv4",
    "publicIpAllocationMethod": "Dynamic",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "4c65de38-71f5-4684-be10-75e605b3e41f",
    "tags": null,
    "type": "Microsoft.Network/publicIPAddresses"
  }
}
```


## <a name="create-a-network-security-group"></a>Создание группы безопасности сети
Для управления входящим и исходящим потоком трафика виртуальных машин к виртуальным сетевым адаптерам или подсети применяются группы безопасности сети. В следующем примере с помощью команды [az network nsg create](/cli/azure/network/nsg) создается группа безопасности сети с именем *myNetworkSecurityGroup*:

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

Вы определяете правила, разрешающие или запрещающие определенный трафик. Чтобы разрешить входящие подключения через порт 22 (для включения доступа по SSH), создайте правило входящего трафика с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule). В следующем примере создается правило *myNetworkSecurityGroupRuleSSH*.

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleSSH \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 22 \
    --access allow
```

Чтобы разрешить входящие подключения через порт 80 (для веб-трафика), добавьте еще одно правило группы безопасности сети. В следующем примере создается правило *myNetworkSecurityGroupRuleHTTP*:

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleWeb \
    --protocol tcp \
    --priority 1001 \
    --destination-port-range 80 \
    --access allow
```

Чтобы проверить группу безопасности сети и правила, выполните команду [az network nsg show](/cli/azure/network/nsg).

```azurecli
az network nsg show --resource-group myResourceGroup --name myNetworkSecurityGroup
```

Выходные данные:

```json
{
  "defaultSecurityRules": [
    {
      "access": "Allow",
      "description": "Allow inbound traffic from all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetInBound",
      "name": "AllowVnetInBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow inbound traffic from azure load balancer",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowAzureLoadBalancerInBou",
      "name": "AllowAzureLoadBalancerInBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "AzureLoadBalancer",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all inbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllInBound",
      "name": "DenyAllInBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetOutBound",
      "name": "AllowVnetOutBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to Internet",
      "destinationAddressPrefix": "Internet",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowInternetOutBound",
      "name": "AllowInternetOutBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all outbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllOutBound",
      "name": "DenyAllOutBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
  "location": "eastus",
  "name": "myNetworkSecurityGroup",
  "networkInterfaces": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "47a9964e-23a3-438a-a726-8d60ebbb1c3c",
  "securityRules": [
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "22",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleSSH",
      "name": "myNetworkSecurityGroupRuleSSH",
      "priority": 1000,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "80",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleWeb",
      "name": "myNetworkSecurityGroupRuleWeb",
      "priority": 1001,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "subnets": null,
  "tags": null,
  "type": "Microsoft.Network/networkSecurityGroups"
}
```

## <a name="create-a-virtual-nic"></a>Создание виртуального сетевого адаптера
Работой сетевых адаптеров можно управлять программно, так как к ним можно применить правила. В зависимости от [размера виртуальной машины](../sizes.md) можно присоединить к ней несколько виртуальных сетевых адаптеров. С помощью команды [az network nic create](/cli/azure/network/nic) создается сетевой адаптер с именем *myNic* и связывается с группой безопасности сети. Общедоступный IP-адрес *myPublicIP* также связывается с виртуальным сетевым адаптером.

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --public-ip-address myPublicIP \
    --network-security-group myNetworkSecurityGroup
```

Выходные данные:

```json
{
  "NewNIC": {
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDnsNameLabel": null,
      "internalDomainNameSuffix": "brqlt10lvoxedgkeuomc4pm5tb.bx.internal.cloudapp.net",
      "internalFqdn": null
    },
    "enableAcceleratedNetworking": false,
    "enableIpForwarding": false,
    "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
    "ipConfigurations": [
      {
        "applicationGatewayBackendAddressPools": null,
        "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic/ipConfigurations/ipconfig1",
        "loadBalancerBackendAddressPools": null,
        "loadBalancerInboundNatRules": null,
        "name": "ipconfig1",
        "primary": true,
        "privateIpAddress": "192.168.1.4",
        "privateIpAddressVersion": "IPv4",
        "privateIpAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "publicIpAddress": {
          "dnsSettings": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
          "idleTimeoutInMinutes": null,
          "ipAddress": null,
          "ipConfiguration": null,
          "location": null,
          "name": null,
          "provisioningState": null,
          "publicIpAddressVersion": null,
          "publicIpAllocationMethod": null,
          "resourceGroup": "myResourceGroup",
          "resourceGuid": null,
          "tags": null,
          "type": null
        },
        "resourceGroup": "myResourceGroup",
        "subnet": {
          "addressPrefix": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
          "ipConfigurations": null,
          "name": null,
          "networkSecurityGroup": null,
          "provisioningState": null,
          "resourceGroup": "myResourceGroup",
          "resourceNavigationLinks": null,
          "routeTable": null
        }
      }
    ],
    "location": "eastus",
    "macAddress": null,
    "name": "myNic",
    "networkSecurityGroup": {
      "defaultSecurityRules": null,
      "etag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
      "location": null,
      "name": null,
      "networkInterfaces": null,
      "provisioningState": null,
      "resourceGroup": "myResourceGroup",
      "resourceGuid": null,
      "securityRules": null,
      "subnets": null,
      "tags": null,
      "type": null
    },
    "primary": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "b3dbaa0e-2cf2-43be-a814-5cc49fea3304",
    "tags": null,
    "type": "Microsoft.Network/networkInterfaces",
    "virtualMachine": null
  }
}
```


## <a name="create-an-availability-set"></a>"Создать группу доступности"
Группы доступности помогают распределить виртуальные машины между доменами сбоя и доменами обновления. Несмотря на то, что прямо сейчас можно создать только одну виртуальную машину, рекомендуется использовать группы доступности для упрощения расширения в будущем. 

Домены сбоя определяют группу виртуальных машин, совместно использующих общий источник питания и сетевой коммутатор. По умолчанию виртуальные машины, настроенные в группе доступности, разделяются между несколькими доменами сбоя, которых может быть не больше трех. Сбой оборудования в одном из этих доменов сбоя не влияет на все виртуальные машины, на которых выполняется ваше приложение.

Домены обновления — это группы виртуальных машин и базовое физическое оборудование, которое может быть перезагружено одновременно. Перезагрузка доменов обновления при запланированном обслуживании может выполняться не по порядку, но одновременно перезагружается только один домен обновления.

Azure автоматически распределяет виртуальные машины между доменами сбоя и обновления при помещении их в группу доступности. Дополнительные сведения см. в статье [Управление доступностью виртуальных машин Linux](../availability.md).

Создайте группу доступности для виртуальных машин с помощью команды [az vm availability-set create](/cli/azure/vm/availability-set). В следующем примере создается группа доступности *myAvailabilitySet*.

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet
```

В выводе указаны домены ошибок и домены обновления:

```json
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/availabilitySets/myAvailabilitySet",
  "location": "eastus",
  "managed": null,
  "name": "myAvailabilitySet",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 5,
  "resourceGroup": "myResourceGroup",
  "sku": {
    "capacity": null,
    "managed": true,
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```


## <a name="create-a-vm"></a>Создание виртуальной машины
Вы создали ресурсы сети для доступных через Интернет виртуальных машин. Теперь создайте виртуальную машину и защитите ее с помощью ключа SSH. В этом примере давайте создадим виртуальную машину Ubuntu с использованием новейшего LTS. Дополнительные образы можно найти с помощью команды [az vm image list](/cli/azure/vm/image), как это описано в статье о [поиске образов виртуальных машин Azure](cli-ps-findimage.md).

Укажите ключ SSH для аутентификации. Если у вас нет пары открытых ключей SSH, их [можно создать](mac-create-ssh-keys.md) или использовать параметр `--generate-ssh-keys` для их создания. Если у вас уже есть пара ключей, этот параметр будет использовать имеющиеся ключи в `~/.ssh`.

Создайте виртуальную машину, собрав воедино все ресурсы и информацию с помощью команды [az vm create](/cli/azure/vm). В следующем примере создается виртуальная машина с именем *myVM*.

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --availability-set myAvailabilitySet \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Подключитесь к виртуальной машине по протоколу SSH с записью DNS, указанной при создании общедоступного IP-адреса. `fqdn` отображается в выходных данных при создании виртуальной машины:

```json
{
  "fqdns": "mypublicdns.eastus.cloudapp.azure.com",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-13-71-C8",
  "powerState": "VM running",
  "privateIpAddress": "192.168.1.5",
  "publicIpAddress": "13.90.94.252",
  "resourceGroup": "myResourceGroup"
}
```

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

Выходные данные:

```bash
The authenticity of host 'mypublicdns.eastus.cloudapp.azure.com (13.90.94.252)' can't be established.
ECDSA key fingerprint is SHA256:SylINP80Um6XRTvWiFaNz+H+1jcrKB1IiNgCDDJRj6A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'mypublicdns.eastus.cloudapp.azure.com,13.90.94.252' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.11.0-1016-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    https://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@myVM:~$
```

Вы можете установить NGINX и просмотреть, как трафик проходит к виртуальной машине. Установите NGINX следующим образом:

```bash
sudo apt-get install -y nginx
```

Для просмотра сайта NGINX по умолчанию откройте браузер и введите полное доменное имя:

![Сайт NGINX по умолчанию на виртуальной машине](media/create-cli-complete/nginx.png)

## <a name="export-as-a-template"></a>Экспорт в качестве шаблона
Допустим, вам требуется создать дополнительную среду разработки с теми же параметрами или соответствующую рабочую среду. Resource Manager использует шаблоны JSON, которые определяют все параметры среды. Можно полностью создавать среды, ссылаясь на этот шаблон JSON. Вы можете создавать [шаблоны JSON вручную](../../azure-resource-manager/templates/template-syntax.md?toc=/azure/virtual-machines/linux/toc.json) или экспортировать существующую среду, чтобы создать шаблон JSON. Чтобы экспортировать группу ресурсов, используйте команду [az group export](/cli/azure/group) следующим образом:

```azurecli
az group export --name myResourceGroup > myResourceGroup.json
```

С помощью этой команды в текущем рабочем каталоге создается файл `myResourceGroup.json`. При создании среды на основе этого шаблона запрашиваются все имена ресурсов. Эти имена можно указать в файле шаблона, добавив параметр `--include-parameter-default-value` в команду `az group export`. Измените шаблон JSON, чтобы указать имена ресурсов, или [создайте parameters.jsдля файла](../../azure-resource-manager/templates/template-syntax.md?toc=/azure/virtual-machines/linux/toc.json) , указывающего имена ресурсов.

Чтобы создать среду из шаблона, используйте команду [AZ Deployment Group Create](/cli/azure/deployment/group) , как показано ниже.

```azurecli
az deployment group create \
    --resource-group myNewResourceGroup \
    --template-file myResourceGroup.json
```

Вы можете прочитать [дополнительные сведения о развертывании из шаблонов](../../azure-resource-manager/templates/deploy-cli.md?toc=/azure/virtual-machines/linux/toc.json). Узнайте, как поэтапно обновлять среды, использовать файл параметров и обращаться к шаблонам из единого места хранения.

## <a name="next-steps"></a>Дальнейшие действия
Теперь вы готовы приступить к работе с несколькими сетевыми компонентами и виртуальными машинами. Используя этот пример среды, можно создать приложение на основе представленных здесь основных компонентов.