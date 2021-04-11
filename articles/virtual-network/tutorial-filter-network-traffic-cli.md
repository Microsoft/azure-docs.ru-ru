---
title: 'Фильтрация сетевого трафика: Azure CLI | Документация Майкрософт'
description: В этой статье вы узнаете, как фильтровать сетевой трафик в подсети с помощью группы безопасности сети, используя Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 57657637afd9e3e527942fb262a085fa4b8b053e
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106066621"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-the-azure-cli"></a>Фильтрация сетевого трафика с помощью групп безопасности сети, используя Azure CLI

Вы можете отфильтровать входящий и исходящий трафик в подсети виртуальной сети с помощью группы безопасности сети. Группы безопасности сети содержат правила безопасности, которые фильтруют трафик по IP-адресу, порту и протоколу. Правила безопасности применяются к ресурсам, развернутым в подсети. Вы узнаете, как выполнять следующие задачи:

* Создание группы безопасности сети и правил безопасности.
* Создание виртуальной сети и привязка группы безопасности сети к подсети.
* Развертывание виртуальных машин в подсеть.
* Тестирование фильтров трафика

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Группа безопасности сети содержит правила безопасности. Правила безопасности указывают источник и назначение. Источники и назначения могут быть группами безопасности приложений.

### <a name="create-application-security-groups"></a>Создание групп безопасности приложений

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group) для всех ресурсов, созданных в рамках этой статьи. В следующем примере создается группа ресурсов в расположении *eastus*. 

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

Создайте группу безопасности приложений с помощью команды [az network asg create](/cli/azure/network/asg). Группа безопасности приложений позволяет группировать серверы с аналогичными требованиями к фильтрации портов. В следующем примере создаются две группы безопасности приложений.

```azurecli-interactive
az network asg create \
  --resource-group myResourceGroup \
  --name myAsgWebServers \
  --location eastus

az network asg create \
  --resource-group myResourceGroup \
  --name myAsgMgmtServers \
  --location eastus
```

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg). В следующем примере создается группа безопасности сети с именем *myNsg*: 

```azurecli-interactive 
# Create a network security group
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsg
```

### <a name="create-security-rules"></a>Создание правил безопасности

Создайте правило безопасности с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule). В следующем примере создается правило, разрешающее входящий поток трафика из Интернета в группу безопасности приложений *myWebServers* через порты 80 и 443:

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-Web-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgWebServers" \
  --destination-port-range 80 443
```

В следующем примере создается правило, разрешающее входящий поток трафика из Интернета в группу безопасности приложений *myMgmtServers* через порт 22:

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgMgmtServers" \
  --destination-port-range 22
```

В этой статье SSH-подключение (порт 22) доступно в Интернете для виртуальной машины *myAsgMgmtServers*. В рабочих средах вместо открытия порта 22 для доступа из Интернета рекомендуется подключиться к ресурсам Azure, которыми вы хотите управлять, с помощью [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) или [частного](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) сетевого подключения.

## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork*:

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16
```

Добавьте подсеть в виртуальную сеть при помощи команды [az network vnet subnet create](/cli/azure/network/vnet/subnet). В следующем примере добавляется подсеть с именем *mySubnet* в виртуальную сеть и ей назначается группа безопасности сети *myNsg*:

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name mySubnet \
  --address-prefix 10.0.0.0/24 \
  --network-security-group myNsg
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте две виртуальные машины в виртуальной сети, чтобы вы могли проверить фильтрацию трафика на более позднем этапе. 

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). В следующем примере создается виртуальная машина, которая будет служить веб-сервером. Параметр `--asgs myAsgWebServers` предписывает Azure делать сетевой интерфейс, создаваемый для виртуальной машины, членом группы безопасности приложений *myAsgWebServers*.

Параметр `--nsg ""` указан для того, чтобы предотвратить создание в Azure группы безопасности сети по умолчанию для сетевого интерфейса, который Azure создает при создании виртуальной машины. Для упрощения работы с этой статьей используется пароль. Ключи обычно используются в рабочей среде. Если вы используете ключи, для выполнения оставшихся инструкций необходимо также настроить перенаправление агента SSH. Дополнительные сведения см. в документации по своему клиенту SSH. В следующей команде замените значение `<replace-with-your-password>` паролем по своему выбору.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmWeb \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgWebServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

Создание виртуальной машины занимает несколько минут. После создания виртуальной машины возвращается результат, аналогичный приведенному ниже. 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmWeb",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Запишите значение **publicIpAddress**. На следующем шаге этот адрес потребуется для доступа к виртуальной машине из Интернета.  Создайте виртуальную машину в качестве сервера управления:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmMgmt \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgMgmtServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

Создание виртуальной машины занимает несколько минут. Посл создания виртуальной машины запишите значение **publicIpAddress** из полученных выходных данных. Этот адрес будет использоваться для доступа к виртуальной машине на следующем шаге. Не переходите к следующему шагу, пока Azure не закончит создание виртуальной машины.

## <a name="test-traffic-filters"></a>Тестирование фильтров трафика

Создайте сеанс SSH с виртуальной машиной *myVmMgmt* при помощи приведенной ниже команды. Замените *\<publicIpAddress>* общедоступным IP-адресом, который присвоен виртуальной машине. В примере выше используется IP-адрес *13.90.242.231*.

```bash 
ssh azureuser@<publicIpAddress>
```

При запросе пароля введите пароль, указанный при работе с разделом [Создание виртуальных машин](#create-virtual-machines).

Подключение будет установлено успешно, так как через порт 22 разрешено получать входящий трафик из Интернета в группу безопасности приложений *myAsgMgmtServers*, в которой находится сетевой интерфейс, подключенный к виртуальной машине *myVmMgmt*.

Используйте следующую команду для установки SSH-подключения к виртуальной машине *myVmWeb* из виртуальной машины *myVmMgmt*:

```bash 
ssh azureuser@myVmWeb
```

Подключение будет установлено успешно, так как правило безопасности по умолчанию в каждой группе безопасности сети разрешает трафик через все порты между всеми IP-адресами в виртуальной сети. Вы не можете установить SSH-подключение к виртуальной машине *myVmWeb* из Интернета, потому что правило безопасности для *myAsgWebServers* не разрешает получение входящего траф ика через порт 22 из Интернета.

Используйте следующие команды, чтобы установить веб-сервер nginx на виртуальную машину *myVmWeb*.

```bash 
# Update package source
sudo apt-get -y update

# Install NGINX
sudo apt-get -y install nginx
```

Виртуальной машине *myVmWeb* разрешено исходящие подключение к Интернету для извлечения nginx, так как правило безопасности по умолчанию разрешает весь исходящий трафик в Интернет. Закройте сеанс SSH с *myVmWeb*, сохраняя подключение `username@myVmMgmt:~$` к виртуальной машине *myVmMgmt*. Чтобы получить экран приветствия nginx из виртуальной машины *myVmWeb*, введите следующую команду.

```bash
curl myVmWeb
```

Выйдите из виртуальной машины *myVmMgmt*. Чтобы убедиться, что вы можете получить доступ к веб-серверу *myVmWeb* извне Azure, введите `curl <publicIpAddress>` на вашем компьютере. Подключение будет установлено успешно, так как через порт 80 разрешено получать входящий трафик из Интернета в группу безопасности приложений *myAsgWebServers*, в которой находится сетевой интерфейс, подключенный к виртуальной машине *myVmWeb*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполнив команду [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали группу безопасности сети и связали ее с подсетью виртуальной сети. Дополнительные сведения о группах безопасности сети см. в статьях [Безопасность сети](./network-security-groups-overview.md) и [Create, change, or delete a network security group](manage-network-security-group.md) (Создание, изменение или удаление группы безопасности сети).

Azure маршрутизирует трафик между подсетями по умолчанию. Вместо этого вы можете перенаправлять трафик между подсетями через виртуальную машину, которая используется в качестве брандмауэра. Дополнительные сведения см. в статье [Маршрутизация сетевого трафика с помощью таблицы маршрутов с использованием портала Azure](tutorial-create-route-table-cli.md).