---
title: Настройка частных IP-адресов для виртуальных машин — Azure CLI
description: Узнайте, как настроить частные IP-адреса для виртуальных машин с помощью интерфейса командной строки (CLI) Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: 40b03a1a-ea00-454c-b716-7574cea49ac0
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/16/2017
ms.author: allensu
ms.openlocfilehash: c34ab73422d8dd41feb9da542ed63fdba060fe3f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84708167"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-cli"></a>Настройка частных IP-адресов для виртуальной машины с помощью Azure CLI


[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

> [!NOTE]
> Для работы приведенных в этом руководстве команд Azure CLI требуется наличие готовой среды. Если вы хотите выполнить команды в том виде, в котором они отображаются в этом документе, сначала создайте тестовую среду, описанную в разделе [Создание виртуальной сети](quick-create-cli.md).

## <a name="specify-a-static-private-ip-address-when-creating-a-vm"></a>Указание статического частного IP-адреса при создании виртуальной машины

Чтобы создать виртуальную машину с именем *DNS01* в подсети *FrontEnd* виртуальной сети *TestVNet* со статическим частным IP-адресом *192.168.1.101*, сделайте следующее:

1. Установите и настройте последнюю версию [Azure CLI](/cli/azure/install-azure-cli) (если вы еще этого не сделали), а затем войдите с использованием учетной записи Azure, выполнив команду [az login](/cli/azure/reference-index).

2. Выполните команду [az network nic create](/cli/azure/network/nic), чтобы создать сетевую карту со статическим частным IP-адресом. В списке, который откроется после выполнения команды, будут указаны используемые параметры. 
   
    ```azurecli
    az network nic create \
    --resource-group TestRG \
    --name TestNIC \
    --location centralus \
    --subnet FrontEnd \
    --private-ip-address 192.168.1.101 \
    --vnet-name TestVNet
    ```

    Ожидаемые выходные данные:
   
    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.101",
                "privateIPAllocationMethod": "Static",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "<guid>"
        }
    }
    ```
    
    Параметры

    * `--private-ip-address`: статический частный IP-адрес для сетевой карты.
    * `--vnet-name`: имя виртуальной сети, в которой будет создана сетевая карта.
    * `--subnet`: имя подсети для создания сетевой карты.

3. Выполните команду [azure vm create](/cli/azure/vm/nic), чтобы создать виртуальную машину с созданными ранее общедоступным IP-адресом и сетевой картой. В списке, который откроется после выполнения команды, будут указаны используемые параметры.
   
    ```azurecli
    az vm create \
    --resource-group TestRG \
    --name DNS01 \
    --location centralus \
    --image Debian \
    --admin-username adminuser \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --nics TestNIC
    ```

    Ожидаемые выходные данные:
   
    ```json
    {
        "fqdns": "",
        "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01",
        "location": "centralus",
        "macAddress": "00-0D-3A-92-C1-66",
        "powerState": "VM running",
        "privateIpAddress": "192.168.1.101",
        "publicIpAddress": "",
        "resourceGroup": "TestRG"
    }
    ```
   
   Параметры, отличные от базовых параметров [az vm create](/cli/azure/vm).

   * `--nics` — имя сетевой карты, к которой подключена виртуальная машина.
   
Не рекомендуем статически назначать виртуальной машине Azure частный IP-адрес в ее операционной системе за исключением ситуаций, когда это необходимо, например при [назначении нескольких IP-адресов виртуальной машине Windows](virtual-network-multiple-ip-addresses-cli.md). Если вы будете вручную устанавливать частный IP-адрес в операционной системе, убедитесь, что он соответствует частному IP-адресу, назначенному [сетевому интерфейсу](virtual-network-network-interface-addresses.md#change-ip-address-settings) Azure. Иначе соединение с виртуальной машиной может быть потеряно. Ознакомьтесь с дополнительными сведениями о параметрах [частных IP-адресов](virtual-network-network-interface-addresses.md#private).

## <a name="retrieve-static-private-ip-address-information-for-a-vm"></a>Получение сведений о статическом частном IP-адресе виртуальной машины

Чтобы просмотреть значения параметров *Private IP alloc-method* и *Private IP address*, выполните следующую команду Azure CLI:

```azurecli
az vm show -g TestRG -n DNS01 --show-details --query 'privateIps'
```

Ожидаемые выходные данные:

```json
"192.168.1.101"
```

Чтобы отобразить определенные сведения об IP-адресе сетевой карты для этой виртуальной машины, отправьте запрос к сетевой карте:

```azurecli
az network nic show \
-g testrg \
-n testnic \
--query 'ipConfigurations[0].{PrivateAddress:privateIpAddress,IPVer:privateIpAddressVersion,IpAllocMethod:p
rivateIpAllocationMethod,PublicAddress:publicIpAddress}'
```

Результат будет выглядеть следующим образом.

```json
{
    "IPVer": "IPv4",
    "IpAllocMethod": "Static",
    "PrivateAddress": "192.168.1.101",
    "PublicAddress": null
}
```

## <a name="remove-a-static-private-ip-address-from-a-vm"></a>Удаление статического частного IP-адреса виртуальной машины

Для развертываний Azure Resource Manager статический частный IP-адрес нельзя удалить из сетевой карты с помощью Azure CLI. Необходимо сделать следующее:
- Создайте сетевую карту, использующую динамический IP-адрес.
- Настройте созданную сетевую карту на виртуальной машине. 

Чтобы изменить сетевую карту виртуальной машины, используемой в предыдущих командах, сделайте следующее:

1. Выполните команду **azure network nic create**, чтобы создать сетевую карту с новым IP-адресом путем выделения динамических IP-адресов. Так как IP-адрес не указан, используется **динамический** способ выделения адреса.

    ```azurecli
    az network nic create     \
    --resource-group TestRG     \
    --name TestNIC2     \
    --location centralus     \
    --subnet FrontEnd    \
    --vnet-name TestVNet
    ```        
   
    Ожидаемые выходные данные:

    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.4",
                "privateIPAllocationMethod": "Dynamic",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "0808a61c-476f-4d08-98ee-0fa83671b010"
        }
    }
    ```

2. Выполните команду **azure vm set** , чтобы изменить сетевую карту, используемую виртуальной машиной.
   
    ```azurecli
   az vm nic set --resource-group TestRG --vm-name DNS01 --nics TestNIC2
    ```

    Ожидаемые выходные данные:
   
    ```json
    [
        {
            "id": "/subscriptions/0e220bf6-5caa-4e9f-8383-51f16b6c109f/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC3",
            "primary": true,
            "resourceGroup": "TestRG"
        }
    ]
    ```

    > [!NOTE]
    > Если размер виртуальной машины позволяет подключить несколько сетевых карт, выполните команду **azure network nic delete**, чтобы удалить старую сетевую карту.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с дополнительными сведениями об управлении [параметрами IP-адресов](virtual-network-network-interface-addresses.md).
