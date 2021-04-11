---
title: 'Маршрутизация сетевого трафика: Azure CLI | Документация Майкрософт'
description: Из этой статьи вы узнаете, как маршрутизировать трафик с помощью таблицы маршрутов и Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: ed2a97ccfb0a3b9b4b43021ab0fff1460919c9b9
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106060743"
---
# <a name="route-network-traffic-with-a-route-table-using-the-azure-cli"></a>Маршрутизация сетевого трафика с помощью таблицы маршрутов и Azure CLI

По умолчанию Azure автоматически маршрутизирует трафик между всеми подсетями в виртуальной сети. Для переопределения маршрутизации Azure по умолчанию можно создать собственные маршруты. Возможность создания настраиваемых маршрутов полезна, если, к примеру, требуется маршрутизировать трафик между подсетями через виртуальный сетевой модуль. Вы узнаете, как выполнять следующие задачи:

* Создание таблицы маршрутов
* Создание маршрута
* Создание виртуальной сети с несколькими подсетями
* Связывание таблицы маршрутов с подсетью
* создание виртуального сетевого модуля, который перенаправляет трафик;
* развертывание виртуальных машин в разных подсетях;
* направление трафика из одной подсети в другую через виртуальный сетевой модуль.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-route-table"></a>Создание таблицы маршрутов

Перед созданием таблицы маршрутов создайте группу ресурсов с помощью команды [az group create](/cli/azure/group) для всех ресурсов, созданных в рамках этой статьи. 

```azurecli-interactive
# Create a resource group.
az group create \
  --name myResourceGroup \
  --location eastus
```

Создайте таблицу маршрутов с помощью команды [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create). В следующем примере создается таблица маршрутов с именем *myRouteTablePublic*. 

```azurecli-interactive
# Create a route table
az network route-table create \
  --resource-group myResourceGroup \
  --name myRouteTablePublic
```

## <a name="create-a-route"></a>Создание маршрута

Создайте маршрут в таблице маршрутов с помощью команды [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create). 

```azurecli-interactive
az network route-table route create \
  --name ToPrivateSubnet \
  --resource-group myResourceGroup \
  --route-table-name myRouteTablePublic \
  --address-prefix 10.0.1.0/24 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.2.4
```

## <a name="associate-a-route-table-to-a-subnet"></a>Связывание таблицы маршрутов с подсетью

Чтобы связать таблицу маршрутов с подсетью, необходимо создать виртуальную сеть и подсеть. Создайте виртуальную сеть с одной подсетью при помощи команды [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

Создайте две дополнительные подсети с помощью команды [az network vnet subnet create](/cli/azure/network/vnet/subnet).

```azurecli-interactive
# Create a private subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24

# Create a DMZ subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name DMZ \
  --address-prefix 10.0.2.0/24
```

Свяжите таблицу маршрутов *myRouteTablePublic* с подсетью *Public* при помощи команды [az network vnet subnet update](/cli/azure/network/vnet/subnet).

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Public \
  --resource-group myResourceGroup \
  --route-table myRouteTablePublic
```

## <a name="create-an-nva"></a>Создание виртуального сетевого модуля

Виртуальный сетевой модуль представляет собой виртуальную машину, которая выполняет сетевые функции, такие как маршрутизация, защита брандмауэром или оптимизация доступа к глобальной сети.

Создайте виртуальный сетевой модуль в подсети *DMZ* с помощью команды [az vm create](/cli/azure/vm). По умолчанию при создании виртуальной машины Azure создает и назначает ей общедоступный IP-адрес. Параметр `--public-ip-address ""` указывает Azure, что для этой виртуальной машины не нужно создавать и назначать общедоступный IP-адрес, так как ей не требуется подключение из Интернета. Также команда создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmNva \
  --image UbuntuLTS \
  --public-ip-address "" \
  --subnet DMZ \
  --vnet-name myVirtualNetwork \
  --generate-ssh-keys
```

Создание виртуальной машины занимает несколько минут. Переходите к следующему шагу только после того, как Azure завершит создание виртуальной машины и вернет выходные данные о ней. 

В сетевом интерфейсе должна быть включена IP-пересылка, чтобы он мог перенаправлять отправленный ему сетевой трафик, который не предназначен для его собственного IP-адреса. Включите IP-пересылку для сетевого интерфейса с помощью команды [az network nic update](/cli/azure/network/nic).

```azurecli-interactive
az network nic update \
  --name myVmNvaVMNic \
  --resource-group myResourceGroup \
  --ip-forwarding true
```

Операционная система или приложение, работающие на виртуальной машине, также должны уметь перенаправлять сетевой трафик. Чтобы включить IP-пересылку в операционной системе виртуальной машины, используйте [az vm extension set](/cli/azure/vm/extension):

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVmNva \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"sudo sysctl -w net.ipv4.ip_forward=1"}'
```

Выполнение команды может занять до минуты.

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте в виртуальной сети две виртуальные машины, чтобы на следующем шаге с их помощью проверить, что трафик из подсети *Public* направляется в подсеть *Private* через виртуальный сетевой модуль. 

Создайте виртуальную машину в подсети *Public* с помощью команды [az vm create](/cli/azure/vm). Параметр `--no-wait` позволяет Azure выполнять команду в фоновом режиме, поэтому вы можете перейти к следующей команде. Для упрощения работы с этой статьей используется пароль. Ключи обычно используются в рабочей среде. Если вы используете ключи, необходимо также настроить перенаправление агента SSH. Дополнительные сведения см. в документации по своему клиенту SSH. В следующей команде замените значение `<replace-with-your-password>` паролем по своему выбору.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --admin-username azureuser \
  --admin-password $adminPassword \
  --no-wait
```

Создайте виртуальную машину в подсети *Private*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --admin-username azureuser \
  --admin-password $adminPassword
```

Создание виртуальной машины занимает несколько минут. После создания виртуальной машины в Azure CLI отображаются примерно такие данные: 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPrivate",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Запишите значение **publicIpAddress**. На следующем шаге этот адрес потребуется для доступа к виртуальной машине из Интернета.

## <a name="route-traffic-through-an-nva"></a>Перенаправление трафика через виртуальный сетевой модуль

Используйте следующую команду для создания сеанса SSH с виртуальной машиной *myVmPrivate*. Замените *\<publicIpAddress>* общедоступным IP-адресом, который присвоен виртуальной машине. В примере выше используется IP-адрес *13.90.242.231*.

```bash
ssh azureuser@<publicIpAddress>
```

При запросе пароля введите пароль, выбранный при работе с разделом [Создание виртуальных машин](#create-virtual-machines).

С помощью следующей команды установите средство трассировки маршрута на виртуальную машину *myVmPrivate*:

```bash
sudo apt-get install traceroute
```

Используя следующую команду, протестируйте маршрутизацию сетевого трафика к виртуальной машине *myVmPublic* от виртуальной машины *myVmPrivate*.

```bash
traceroute myVmPublic
```

Ответ будет выглядеть примерно так:

```output
traceroute to myVmPublic (10.0.0.4), 30 hops max, 60 byte packets
1  10.0.0.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Вы увидите, что трафик направляется непосредственно от виртуальной машины *myVmPrivate* на виртуальную машину *myVmPublic*. Маршруты Azure по умолчанию выполняют маршрутизацию трафика напрямую между подсетями. 

Используя следующую команду, создайте SSH-подключение к виртуальной машине *myVmPublic* с виртуальной машины *myVmPrivate*.

```bash
ssh azureuser@myVmPublic
```

С помощью следующей команды установите средство трассировки маршрута на виртуальную машину *myVmPublic*:

```bash
sudo apt-get install traceroute
```

Используя следующую команду, протестируйте маршрутизацию сетевого трафика к виртуальной машине *myVmPrivate* от виртуальной машины *myVmPublic*.

```bash
traceroute myVmPrivate
```

Ответ будет выглядеть примерно так:

```output
traceroute to myVmPrivate (10.0.1.4), 30 hops max, 60 byte packets
1  10.0.2.4 (10.0.2.4)  0.781 ms  0.780 ms  0.775 ms
2  10.0.1.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Можно заметить, что первым прыжком указан частный IP-адрес виртуального сетевого модуля (10.0.2.4). Второй прыжок — 10.0.1.4. Это частный IP-адрес виртуальной машины *myVmPrivate*. Из-за маршрута, добавленного в таблицу маршрутов *myRouteTablePublic* и связанного с подсетью *Public*, трафик в Azure был маршрутизирован через NVA, а не напрямую в подсеть *Private*.

Закройте сеансы SSH к виртуальным машинам *myVmPublic* и *myVmPrivate*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполнив команду [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этой статьей вы создали таблицу маршрутов и связали ее с подсетью. Вы также создали простой виртуальный сетевой модуль, который направляет трафик из общедоступной подсети в частную подсеть. [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) предоставляет широкий выбор предварительно настроенных виртуальных сетевых модулей для разных сетевых функций, например модули брандмауэра и оптимизация доступа к глобальной сети. См. дополнительные сведения о [маршрутизации](virtual-networks-udr-overview.md) и [управлении таблицей маршрутов](manage-route-table.md).

Хотя в виртуальной сети можно развернуть множество ресурсов Azure, ресурсы для некоторых служб PaaS Azure развернуть невозможно. Тем не менее вы все же можете ограничить доступ к ресурсам некоторых служб PaaS в Azure трафиком только из подсети виртуальной сети. Сведения о том, как это сделать, см. в статье [Ограничение сетевого доступа к ресурсам PaaS посредством конечных точек службы виртуальной сети с помощью Azure CLI](tutorial-restrict-network-access-to-resources-cli.md).
