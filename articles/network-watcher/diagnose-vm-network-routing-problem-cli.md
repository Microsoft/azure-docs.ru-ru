---
title: Диагностика проблемы сетевой маршрутизации виртуальной машины — Azure CLI
titleSuffix: Azure Network Watcher
description: В этой статье вы узнаете, как использовать Azure CLI для диагностики проблемы сетевой маршрутизации виртуальной машины с помощью возможности следующего прыжка наблюдателя за сетями Azure.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 01/07/2021
ms.author: damendo
ms.custom: ''
ms.openlocfilehash: 5dac16e5cc1e88d833bf6d3c2660570fcf8b8e9e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102216961"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-cli"></a>Диагностика проблем с маршрутизацией в сети виртуальной машины с помощью Azure CLI

В этой статье вы развернете виртуальную машину и проверите доступ к IP-адресу и URL-адресу. Затем вы определите причину проблемы с подключением и найдете способ ее устранения.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

- Azure CLI команды в этой статье форматируются для выполнения в оболочке bash.

## <a name="create-a-vm"></a>Создание виртуальной машины

Прежде чем создать виртуальную машину, создайте группу ресурсов, которая будет содержать эту виртуальную машину. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az-vm-create). Также команда создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`. В следующем примере создается виртуальная машина с именем *myVM*:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

Создание виртуальной машины занимает несколько минут. Переходите к остальным шагам только после того, как завершится создание виртуальной машины и Azure CLI вернет выходные данные.

## <a name="test-network-communication"></a>Тестирование взаимодействия по сети

Чтобы проверить сетевое подключение с помощью Наблюдателя за сетями, сначала включите Наблюдатель за сетями в регионе, где размещается тестируемая виртуальная машина. Затем примените в Наблюдателе за сетями возможность следующего прыжка.

### <a name="enable-network-watcher"></a>Включение Наблюдателя за сетями

Если в регионе "Восточная часть США" уже включена служба "Наблюдатель за сетями", перейдите к разделу по [использованию следующего прыжка](#use-next-hop). С помощью команды [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) создайте Наблюдателя за сетями в регионе "Восточная часть США":

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-next-hop"></a>Использование следующего прыжка

Azure автоматически создает маршруты к пунктам назначения по умолчанию. Вы можете создать настраиваемые маршруты, которые переопределяют маршруты по умолчанию. В некоторых случаях настраиваемые маршруты могут привести к сбою связи. Чтобы проверить маршрутизацию из виртуальной машины, выполните команду [az network watcher show-next-hop](/cli/azure/network/watcher#az-network-watcher-show-next-hop) для определения следующего прыжка маршрутизации при поступлении трафика на определенный адрес.

Проверьте исходящее подключение виртуальной машины по любому из IP-адресов сайта www.bing.com.

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 13.107.21.200 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

Через несколько секунд выходные данные уведомляются о том, что **nextHopType** находится в **Интернете** и что **раутетаблеид** является **системным маршрутом**. Из этого результата вы узнаете, что существует допустимый системный маршрут к назначению.

Проверьте исходящее подключение виртуальной машины к адресу 172.31.0.100:

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 172.31.0.100 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

Возвращаемые выходные данные уведомляют о том  , что нет **NextHopType** и что **Раутетаблеид** также является **системным маршрутом**. Так мы узнаем, что, хотя есть допустимый системный маршрут к месту назначения, следующий прыжок для маршрутизации трафика в пункт назначения отсутствует.

## <a name="view-details-of-a-route"></a>Просмотр сведений о маршруте

Чтобы проанализировать маршрутизацию дальше, просмотрите действующие маршруты для сетевого интерфейса, выполнив команду [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table):

```azurecli-interactive
az network nic show-effective-route-table \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

В возвращенные данные включен следующий текст:

```console
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
  },
  "addressPrefix": [
    "0.0.0.0/0"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "Internet",
  "source": "Default",
  "state": "Active"
},
```

При выполнении команды `az network watcher show-next-hop` для проверки исходящего подключения по IP-адресу 13.107.21.200 в разделе [Использование следующего прыжка](#use-next-hop) маршрут с **addressPrefix** 0.0.0.0/0** использовался для маршрутизации трафика на адрес, так как другие маршруты в выходных данных не включали адрес. По умолчанию все адреса, не указанные в префиксе адреса другого маршрута, маршрутизируются в Интернет.

Однако при выполнении команды `az network watcher show-next-hop` для проверки исходящего подключения по IP-адресу 172.31.0.100 вы узнали, что тип следующего прыжка отсутствовал. В возвращенные данные также включен следующий текст:

```console
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
      },
  "addressPrefix": [
    "172.16.0.0/12"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "None",
  "source": "Default",
  "state": "Active"
},
```

Как можно увидеть в выходных данных выполнения команды `az network watcher nic show-effective-route-table`, хотя есть маршрут по умолчанию к префиксу 172.16.0.0/12, который включает адрес 172.31.0.100, **nextHopType** имеет значение **Нет**. Azure создает маршрут по умолчанию 172.16.0.0/12, но не указывает тип следующего прыжка, пока для этого не возникнут основания. Если, например, добавить диапазон адресов 172.16.0.0/12 в адресное пространство виртуальной сети, Azure изменит **nextHopType** для маршрута на **Виртуальная сеть**. Затем проверка отобразит **Виртуальная сеть** в качестве значения **nextHopType**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали виртуальную машину и выполнили диагностику сетевой маршрутизации из виртуальной машины. Вы узнали, что Azure создает несколько маршрутов по умолчанию и тестирует маршрутизацию в два разные пункта назначения. Дополнительные сведения см. в статье [Маршрутизация трафика в виртуальной сети](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) и разделе [Создание маршрута](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Для исходящих подключений виртуальной машины можно определить задержку, разрешенный и запрещенный сетевой трафик между виртуальной машиной и конечной точкой с помощью функции [устранения неполадок с подключениями](network-watcher-connectivity-cli.md) Наблюдателя за сетями. Вы можете отслеживать обмен данными между виртуальной машиной и конечной точкой, представленных IP-адресом или URL-адресом, с течением времени, используя возможность мониторинга подключения Наблюдателя за сетями. Инструкции см. в статье [Руководство. Мониторинг сетевого взаимодействия между двумя виртуальными машинами с помощью портала Azure](connection-monitor.md).
