---
title: Создание виртуальной машины со статическим общедоступным IP-адресом (Azure CLI) | Документация Майкрософт
description: Узнайте, как создать виртуальную машину со статическим общедоступным IP-адресом с помощью интерфейса командной строки (CLI) Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: 55bc21b0-2a45-4943-a5e7-8d785d0d015c
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: kumud
ms.openlocfilehash: f0f61cc4ef02033a2c21ce5acde68caea483e743
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790137"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-the-azure-cli"></a>Создание виртуальной машины со статическим общедоступным IP-адресом с помощью Azure CLI

Вы можете создать виртуальную машину со статическим общедоступным IP-адресом. Общедоступный IP-адрес позволяет подключиться к виртуальной машине из Интернета. Присвойте статический общедоступный IP-адрес, а не динамический, чтобы гарантировать, что адрес никогда не изменится. Подробнее о [статических общедоступных IP-адресах](./public-ip-addresses.md#allocation-method). Чтобы изменить общедоступный IP-адрес, присвоенный существующей виртуальной машине, с динамического на статический или использовать частные IP-адреса, см. статью [Добавление, изменение и удаление IP-адресов для сетевого интерфейса Azure](virtual-network-network-interface-addresses.md). За использование общедоступных IP-адресов взимается [номинальная плата](https://azure.microsoft.com/pricing/details/ip-addresses). Кроме того, существует [ограничение](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits) на число общедоступных IP-адресов, которые можно использовать в рамках одной подписки.

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Указанные ниже действия можно выполнить с помощью локального компьютера или Azure Cloud Shell. Чтобы использовать локальный компьютер, на нем должен быть [установлен интерфейс командной строки Azure (Azure CLI)](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json). Чтобы использовать Azure Cloud Shell, выберите функцию **Попробовать** в правом верхнем углу любого окна со следующими командами. Через Cloud Shell будет выполнен вход в Azure.

1. При использовании Cloud Shell перейдите к шагу 2. Откройте сеанс командной строки и войдите в Azure с помощью команды `az login`.
2. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). В следующем примере создается группа ресурсов в регионе Azure "Восточная часть США".

   ```azurecli-interactive
   az group create --name myResourceGroup --location eastus
   ```

3. Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm#az_vm_create). Параметр `--public-ip-address-allocation=static` присваивает статический общедоступный IP-адрес виртуальной машине. В следующем примере создается виртуальная машина Ubuntu со статическим общедоступным IP-адресом SKU "Базовый" под названием *myPublicIpAddress*.

   ```azurecli-interactive
   az vm create \
     --resource-group myResourceGroup \
     --name myVM \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys \
     --public-ip-address myPublicIpAddress \
     --public-ip-address-allocation static
   ```

   Если для общедоступного IP-адреса должен использоваться SKU "Стандартный", добавьте `--public-ip-sku Standard` к предыдущей команде. Дополнительные сведения о [номерах SKU общедоступных IP-адресов](./public-ip-addresses.md#sku). Если виртуальная машина добавляется в серверный пул общедоступной подсистемы Azure Load Balancer, номера SKU общедоступных IP-адресов виртуальной машины и подсистемы балансировки нагрузки должны совпадать. Подробные сведения см. в статье [Что такое Azure Load Balancer](../load-balancer/skus.md).

4. Просмотрите общедоступный IP-адрес и убедитесь, что он был создан в качестве статического адреса с SKU "Базовый" с помощью команды [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show):

   ```azurecli-interactive
   az network public-ip show \
     --resource-group myResourceGroup \
     --name myPublicIpAddress \
     --query [ipAddress,publicIpAllocationMethod,sku] \
     --output table
   ```

   Платформа Azure присвоила общедоступный IP-адрес из адресов, используемых в регионе, в котором вы создали виртуальную машину. Вы можете загрузить список диапазонов (префиксов) для облаков Azure: [общедоступное](https://www.microsoft.com/download/details.aspx?id=56519), облако [правительства США](https://www.microsoft.com/download/details.aspx?id=57063), облако для [Китая](https://www.microsoft.com/download/details.aspx?id=57062) и [Германии](https://www.microsoft.com/download/details.aspx?id=57064).

> [!WARNING]
> Не изменяйте параметры IP-адресов в операционной системе виртуальной машины. Общедоступные IP-адреса Azure не поддерживаются в операционной системе. Вы можете добавлять параметры частных IP-адресов в операционную систему, но это рекомендуется делать только при необходимости и только после прочтения раздела о [добавлении частных IP-адресов в операционную систему](virtual-network-network-interface-addresses.md#private).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [общедоступных IP-адресах в Azure](./public-ip-addresses.md#public-ip-addresses).
- Дополнительные сведения обо всех [параметрах общедоступных IP-адресов](virtual-network-public-ip-address.md#create-a-public-ip-address).
- Дополнительные сведения о [частных IP-адресах](./private-ip-addresses.md) и назначении [статического частного IP-адреса](virtual-network-network-interface-addresses.md#add-ip-addresses) виртуальной машине Azure
- Дополнительные сведения о создании виртуальных машин [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) и [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
