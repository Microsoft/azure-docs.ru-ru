---
title: Настройка предпочтительного варианта маршрутизации для виртуальной машины — Azure CLI
description: Узнайте, как создать виртуальную машину с общедоступным IP-адресом и выбором предпочтительного варианта маршрутизации с помощью интерфейса командной строки (CLI) Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2021
ms.author: mnayak
ms.custom: devx-track-azurecli
ms.openlocfilehash: ad8f2d150c3cf17c4b24c6dc92188be9017dcfa9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101666000"
---
# <a name="configure-routing-preference-for-a-vm-using-azure-cli"></a>Настройка предпочтительного варианта маршрутизации для виртуальной машины с помощью Azure CLI

В этой статье показано, как настроить предпочтительный вариант маршрутизации для виртуальной машины. Связанный с Интернетом трафик от виртуальной машины будет направляться через сеть поставщика услуг Интернета при выборе параметра **Интернет** в качестве предпочтительного варианта маршрутизации. Маршрутизация по умолчанию осуществляется через глобальную сеть Майкрософт.

В этой статье показано, как создать виртуальную машину с общедоступным IP-адресом, настроенным для маршрутизации трафика через общедоступный Интернет с помощью Azure CLI.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
1. При использовании Cloud Shell перейдите к шагу 2. Откройте сеанс командной строки и войдите в Azure с помощью команды `az login`.
2. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов в регионе Azure "Восточная часть США".

    ```azurecli
    az group create --name myResourceGroup --location eastus
    ```

## <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса
Чтобы получить доступ к виртуальным машинам из Интернета, необходимо создать общедоступный IP-адрес. Создайте общедоступный IP-адрес с помощью команды [az network public-ip create](/cli/azure/network/public-ip). В следующем примере создается общедоступный IP-адрес с предпочтительным вариантом маршрутизации *Интернет* в регионе *Восточная часть США*.

```azurecli
az network public-ip create \
--name MyRoutingPrefIP \
--resource-group MyResourceGroup \
--location eastus \
--ip-tags 'RoutingPreference=Internet' \
--sku STANDARD \
--allocation-method static \
--version IPv4
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Перед развертыванием виртуальной машины необходимо создать вспомогательные сетевые ресурсы — группу безопасности сети, виртуальную сеть и виртуальный сетевой адаптер.

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети для правил, которые будут управлять входящим и исходящим обменом данными в виртуальной сети с помощью [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create)

```azurecli
az network nsg create \
--name myNSG  \
--resource-group MyResourceGroup \
--location eastus
```

### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). В следующем примере создается виртуальная сеть *myVNET* с подсетью *mySubNet*.

```azurecli
# Create a virtual network
az network vnet create \
--name myVNET \
--resource-group MyResourceGroup \
--location eastus

# Create a subnet
az network vnet subnet create \
--name mySubNet \
--resource-group MyResourceGroup \
--vnet-name myVNET \
--address-prefixes "10.0.0.0/24" \
--network-security-group myNSG
```

### <a name="create-a-nic"></a>Создание сетевой карты

Создайте виртуальную сетевую карту для виртуальной машины с помощью команды [az network nic create](/cli/azure/network/nic#az-network-nic-create). В следующем примере создается виртуальная сетевая карта, которая будет подключена к виртуальной машине.

```azurecli-interactive
# Create a NIC
az network nic create \
--name mynic  \
--resource-group MyResourceGroup \
--network-security-group myNSG  \
--vnet-name myVNET \
--subnet mySubNet  \
--private-ip-address-version IPv4 \
--public-ip-address MyRoutingPrefIP
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az-vm-create). В приведенном ниже примере создается виртуальная машина windows server 2019 и обязательные компоненты виртуальной сети, если их еще нет:

```azurecli
az vm create \
--name myVM \
--resource-group MyResourceGroup \
--nics mynic \
--size Standard_A2 \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest \
--admin-username myUserName
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду [az group delete](/cli/azure/group#az-group-delete).

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [предпочтительном варианте маршрутизации в общедоступных IP-адресах](routing-preference-overview.md).
- См. дополнительные сведения об [общедоступных IP-адресах в Azure](./public-ip-addresses.md#public-ip-addresses).
- Дополнительные сведения о [параметрах общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address).
