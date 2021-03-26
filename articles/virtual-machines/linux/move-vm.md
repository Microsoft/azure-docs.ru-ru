---
title: Перемещение виртуальной машины в с помощью Azure CLI
description: Переместите виртуальную машину в другую подписку или группу ресурсов Azure с помощью Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 09/12/2018
ms.author: cynthn
ms.openlocfilehash: 7dbe06a9f2fff8abf59adbdfc9e41055c85e8f2c
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889303"
---
# <a name="move-a-vm-to-another-subscription-or-resource-group"></a>Перемещение виртуальной машины в другую подписку или группу ресурсов
В этой статье описывается перемещение виртуальной машины (ВМ) между группами ресурсов или подписками. Перемещение ВМ между подписками может понадобиться, если вы создали виртуальную машину в личной подписке и вам нужно переместить ее в корпоративную подписку.

> [!IMPORTANT]
>Во время перемещения создаются новые идентификаторы ресурсов. После перемещения виртуальной машины вам нужно будет обновить средства и сценарии, чтобы использовать новые идентификаторы ресурсов.
>


## <a name="use-the-azure-cli-to-move-a-vm"></a>Использование Azure CLI для перемещения виртуальной машины


Перед перемещением виртуальной машины с помощью Azure CLI удостоверьтесь, что исходная и целевая подписки находятся в пределах одного клиента. Чтобы убедиться, что обе подписки имеют один и тот же идентификатор клиента, выполните команду [az account show](/cli/azure/account).

```azurecli-interactive
az account show --subscription mySourceSubscription --query tenantId
az account show --subscription myDestinationSubscription --query tenantId
```
Если идентификаторы клиентов для исходной и целевой подписок не совпадают, необходимо обратиться в [службу поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) для перемещения ресурсов в новый клиент.

Для успешного перемещения виртуальной машины необходимо перенести саму виртуальную машину и все вспомогательные ресурсы. Чтобы вывести список всех ресурсов в группе и их идентификаторы, используйте команду [az resource list](/cli/azure/resource). Желательно сохранить выходные данные этой команды в файл, чтобы позже можно было копировать и вставлять идентификаторы в командах.

```azurecli-interactive
az resource list --resource-group "mySourceResourceGroup" --query "[].{Id:id}" --output table
```
`table`Выходные данные недоступны при использовании `--interactive` . Измените выходные данные на другой вариант `json` , например.

Чтобы переместить виртуальную машину и ее ресурсы в другую группу ресурсов, выполните команду [az resource move](/cli/azure/resource). Ниже показано, как переместить ВМ и наиболее распространенные ресурсы, которые для этого понадобятся. Используйте параметр **-ids** и передайте разделенный запятыми список (без пробелов) идентификаторов ресурсов для перемещения.

```azurecli-interactive
vm=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
nic=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkInterfaces/myNIC
nsg=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG
pip=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIPAddress
vnet=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/virtualNetworks/myVNet
diag=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mydiagnosticstorageaccount
storage=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccountname    

az resource move \
    --ids $vm,$nic,$nsg,$pip,$vnet,$storage,$diag \
    --destination-group "myDestinationResourceGroup"
```

Если нужно переместить виртуальную машину и ее ресурсы в другую подписку, добавьте параметр **--destination-subscriptionId**, чтобы указать целевую подписку.

При появлении запроса на подтверждение перемещения указанных ресурсов введите **Да** для подтверждения.

[!INCLUDE [virtual-machines-common-move-vm](../../../includes/virtual-machines-common-move-vm.md)]

## <a name="next-steps"></a>Дальнейшие действия
Вы можете перемещать разные типы ресурсов между группами ресурсов и подписками. Дополнительные сведения см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](../../azure-resource-manager/management/move-resource-group-and-subscription.md).    
