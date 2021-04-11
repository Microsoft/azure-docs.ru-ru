---
title: Краткое руководство. Создание виртуальной сети с помощью Azure CLI
titlesuffix: Azure Virtual Network
description: Из этого краткого руководства вы узнаете, как создать виртуальную сеть с помощью Azure CLI. Виртуальная сеть позволяет ресурсам Azure взаимодействовать между собой и с Интернетом.
author: KumudD
ms.service: virtual-network
ms.topic: quickstart
ms.date: 03/06/2021
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0795404c2dc5377d60896863f6a088c4b2ffd1ad
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106060828"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-cli"></a>Краткое руководство. Создание виртуальной сети с помощью интерфейса командной строки Azure

Виртуальная сеть позволяет ресурсам Azure, таким как виртуальные машины, обмениваться данными в частном порядке и взаимодействовать через Интернет. 

Из этого краткого руководства вы узнаете, как создать виртуальную сеть. Создав виртуальную сеть, разверните в ней две виртуальные машины. Затем вы подключитесь к виртуальным машинам из Интернета и установите частную связь через новую виртуальную сеть.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим кратким руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group-and-a-virtual-network"></a>Создание группы ресурсов и виртуальной сети

Перед созданием виртуальной сети необходимо создать группу ресурсов, которая будет содержать эту виртуальную сеть. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). В этом примере создается группа ресурсов с именем **CreateVNetQS-rg** в расположении **EastUS**:

```azurecli-interactive
az group create \
    --name CreateVNetQS-rg \
    --location eastus
```

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create). В этом примере создается виртуальная сеть по умолчанию с именем **myVNet** и одной подсетью с именем **default**:

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group CreateVNetQS-rg \
  --subnet-name default
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте две виртуальные машины в виртуальной сети.

### <a name="create-the-first-vm"></a>Создание первой виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az_vm_create). 

Команда также создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`. 

Параметр `--no-wait` создает виртуальную машину в фоновом режиме. Перейдите к следующему шагу. 

В этом примере создается виртуальная машина с именем **myVM1**:

```azurecli-interactive
az vm create \
  --resource-group CreateVNetQS-rg \
  --name myVM1 \
  --image UbuntuLTS \
  --generate-ssh-keys \
  --public-ip-address myPublicIP-myVM1 \
  --no-wait
```

### <a name="create-the-second-vm"></a>Создание второй виртуальной машины

Вы использовали параметр `--no-wait` на предыдущем шаге. Вы можете создать вторую виртуальную машину с именем **myVM2**.

```azurecli-interactive
az vm create \
  --resource-group CreateVNetQS-rg \
  --name myVM2 \
  --image UbuntuLTS \
  --public-ip-address myPublicIP-myVM2 \
  --generate-ssh-keys
```

### <a name="azure-cli-output-message"></a>Выходное сообщение Azure CLI

Создание виртуальных машин может занять несколько минут. После того как Azure создаст виртуальные машины, Azure CLI вернет выходные данные следующего вида:

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/CreateVNetQS-rg/providers/Microsoft.Compute/virtualMachines/myVM2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "CreateVNetQS-rg"
  "zones": ""
}
```

## <a name="vm-public-ip"></a>Общедоступный IP-адрес виртуальной машины

Чтобы получить общедоступный IP-адрес виртуальной машины **myVM2**, выполните команду [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show):

```azurecli-interactive
az network public-ip show \
  --resource-group CreateVNetQS-rg  \
  --name myPublicIP-myVM2 \
  --query ipAddress \
  --output tsv
```

## <a name="connect-to-a-vm-from-the-internet"></a>Подключение к виртуальной машине из Интернета

В этой команде замените `<publicIpAddress>` общедоступным IP-адресом виртуальной машины **myVM2**:

```bash
ssh <publicIpAddress>
```

## <a name="communicate-between-vms"></a>Взаимодействие между виртуальными машинами

Чтобы проверить частный обмен данными между виртуальными машинами **myVM2** и **myVM1**, введите команду:

```bash
ping myVM1 -c 4
```

Вы получите четыре ответа с адреса *10.0.0.4*.

Завершите сеанс SSH с виртуальной машиной **myVM2**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду [az group delete](/cli/azure/group#az_group_delete):

```azurecli-interactive
az group delete \
    --name CreateVNetQS-rg \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве: 

* Вы создали виртуальную сеть по умолчанию и две виртуальные машины. 
* Затем вы подключились к одной виртуальной машине из Интернета и установили частную связь между двумя виртуальными машинами.

Частный обмен данными между виртуальными машинами не ограничен в виртуальной сети. 

Перейдите к следующей статье, чтобы узнать больше о настройке различных типов сетевого взаимодействия с виртуальными машинами:
> [!div class="nextstepaction"]
> [Фильтрация сетевого трафика](tutorial-filter-network-traffic.md)
