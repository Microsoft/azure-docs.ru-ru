---
title: Краткое руководство. Диагностика проблемы с фильтром сетевого трафика на виртуальной машине с помощью Azure CLI
titleSuffix: Azure Network Watcher
description: Узнаете, как с помощью Azure CLI диагностировать проблему с фильтром сетевого трафика на виртуальной машине, используя функцию проверки IP-потока в Наблюдателе за сетями Azure.
services: network-watcher
documentationcenter: network-watcher
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 01/07/2021
ms.author: kumud
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 18e380345ef0caab52e9b1c537bada73e36d8b48
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106063327"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-cli"></a>Краткое руководство. Диагностика проблемы с фильтром сетевого трафика на виртуальной машине с помощью Azure CLI

При работе с этим кратким руководством вы развернете виртуальную машину и проверите доступ к IP- и URL-адресу и от IP-адреса. Затем вы определите причину проблемы с подключением и найдете способ ее устранения.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим кратким руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

- Команды Azure CLI в этом кратком руководстве отформатированы для выполнения в оболочке Bash.

## <a name="create-a-vm"></a>Создание виртуальной машины

Прежде чем создать виртуальную машину, создайте группу ресурсов, которая будет содержать эту виртуальную машину. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). Также команда создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`. В следующем примере создается виртуальная машина с именем *myVM*:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

Создание виртуальной машины занимает несколько минут. Переходите к остальным шагам только после того, как завершится создание виртуальной машины и Azure CLI вернет выходные данные.

## <a name="test-network-communication"></a>Тестирование взаимодействия по сети

Чтобы проверить сетевое подключение с помощью Наблюдателя за сетями, сначала включите Наблюдатель за сетями в регионе, где размещается тестируемая виртуальная машина. Затем примените в Наблюдателе за сетями функцию проверки IP-потока.

### <a name="enable-network-watcher"></a>Включение Наблюдателя за сетями

Если в регионе "Восточная часть США" уже включена служба "Наблюдатель за сетями", перейдите к разделу по [проверке IP-потока](#use-ip-flow-verify). Чтобы создать Наблюдатель за сетями в регионе EastUS, используйте команду [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure).

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-ip-flow-verify"></a>Применение проверки IP-потока

При создании виртуальной машины Azure применяет к ней стандартные правила разрешений и запретов трафика. Позднее вы можете переопределить эти значения по умолчанию, чтобы разрешить или запретить дополнительные типы трафика. Чтобы проверить, разрешен или запрещен трафик до определенных мест назначения от исходного IP-адреса, используйте команду [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow).

Проверьте исходящее подключение виртуальной машины по любому из IP-адресов сайта www.bing.com.

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Через несколько секунд вы получите результат с информацией о том, что такой доступ разрешается правилом безопасности с именем **AllowInternetOutbound**.

Проверьте исходящее подключение виртуальной машины к адресу 172.31.0.100:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Результат свидетельствует о том, что такой доступ запрещен правилом безопасности с именем **DefaultOutboundDenyAll**.

Проверьте исходящее подключение с адреса 172.31.0.100 к виртуальной машине:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Вернется результат с информацией о том, что такой доступ запрещается правилом безопасности с именем **DefaultInboundDenyAll**. Теперь вы знаете, какие правила безопасности разрешают или запрещают трафик к виртуальной машине или от нее. Это поможет вам решить проблему.

## <a name="view-details-of-a-security-rule"></a>Просмотр сведений о правиле безопасности

Чтобы определить, почему в разделе [проверки IP-потока](#use-ip-flow-verify) мы получили такие ответы о разрешении и (или) запрете доступа, изучите правила безопасности для сетевого интерфейса с помощью команды [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg):

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

Возвращаемые данные содержат следующий текст для правила **AllowInternetOutbound**, которое разрешало исходящий доступ к www.bing.com в разделе [проверки IP-потока](#use-ip-flow-verify) выше:

```console
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

Как видно в выходных данных выше, параметр **destinationAddressPrefix** имеет значение **Internet**. Но при этом неясно, почему 13.107.21.200 относится к категории **Internet**. Рядом в разделе **expandedDestinationAddressPrefix** перечислены несколько префиксов адресов. Один из этих префиксов имеет значение **12.0.0.0/6**, которое охватывает диапазон IP-адресов с 12.0.0.1 по 15.255.255.254. Так как 13.107.21.200 попадает в этот диапазон адресов, правило **AllowInternetOutBound** разрешает исходящий трафик. Кроме того, в выходных данных выше не существует правил с более высоким приоритетом (чем меньше это число, тем выше приоритет), которые переопределяют указанное выше правило. Чтобы запретить исходящий трафик по конкретному IP-адресу, вы можете добавить правило безопасности с более высоким приоритетом, которое запрещает исходящие подключения по этому IP-адресу через порт 80.

Когда вы выполняли команду `az network watcher test-ip-flow`, чтобы проверить исходящее подключение по адресу 172.131.0.100 в разделе [проверки IP-потока](#use-ip-flow-verify), выходные данные указывали на то, что правило **DefaultOutboundDenyAll** запрещает такую связь. Правило **DefaultOutboundDenyAll** аналогично правилу **DenyAllOutBound**, которое указано в следующих выходных данных команды `az network nic list-effective-nsg`:

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

Это правило содержит значение **0.0.0.0/0** для параметра **destinationAddressPrefix**. Такое правило запрещает исходящий трафик по адресу 172.131.0.100, так как он не входит в диапазон **destinationAddressPrefix** любого из правил для исходящего трафика,перечисленных в выходных данных команды `az network nic list-effective-nsg`. Чтобы разрешить исходящий трафик, вы можете добавить правило безопасности с более высоким приоритетом, которое разрешает исходящие подключения по IP-адресу 172.131.0.100 через порт 80.

Когда вы выполняли команду `az network watcher test-ip-flow` в разделе [проверки IP-потока](#use-ip-flow-verify) для проверки входящей связи от адреса 172.131.0.100, выходные данные содержали сведения о том, что правило **DefaultInboundDenyAll** запрещает такую связь. Правило **DefaultInboundDenyAll** аналогично правилу **DenyAllInBound**, которое указано в следующих выходных данных команды `az network nic list-effective-nsg`:

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

Правило **DenyAllInBound** применяется, так как в выходных данных команды `az network nic list-effective-nsg` не существует правил с более высоким приоритетом, которые разрешают входящие подключения к виртуальной машине с адреса 172.131.0.100 через порт 80. Чтобы разрешить входящий трафик, вы можете добавить правило безопасности с более высоким приоритетом, которое разрешает входящий трафик на порт 80 с адреса 172.131.0.100.

Проверки в этом кратком руководстве предназначены для тестирования конфигурации Azure. Если все эти проверки возвращают ожидаемые результаты, но сетевые проблемы при этом сохраняются, проверьте отсутствие брандмауэров между виртуальной машиной и конечной точкой, с которой вы взаимодействуете, а также отсутствие брандмауэра в операционной системы самой виртуальной машины, который разрешает или запрещает обмен данными.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали виртуальную машину и проверили работу фильтров входящего и исходящего трафика. Вы узнали, что правила групп безопасности сети могут разрешать или запрещать исходящий и входящий трафик виртуальной машины. Изучите дополнительные сведения о [правилах безопасности](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) и [создании правил безопасности](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).

Даже если фильтры трафика правильно настроены, обмен данными с виртуальной машиной может завершиться сбоем из-за настроек маршрутизации. Сведения о том, как диагностировать проблемы с маршрутизацией в сети виртуальных машин, см. в [этом кратком руководстве](diagnose-vm-network-routing-problem-cli.md). Единое средство для диагностики проблем с исходящей маршрутизацией, задержками и фильтрацией описано в статье [об устранении неполадок с подключением](network-watcher-connectivity-cli.md).