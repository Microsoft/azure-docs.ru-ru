---
title: Развертывание виртуальных машин Azure на месте с помощью интерфейса командной строки
description: Узнайте, как использовать интерфейс командной строки для развертывания виртуальных машин Azure на месте, чтобы сэкономить затраты.
author: cynthn
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/26/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 0a7be682f921efdfae486e8f6545758964a941ae
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102098865"
---
# <a name="deploy-azure-spot-virtual-machines-using-the-azure-cli"></a>Развертывание виртуальных машин с использованием точки Azure с помощью Azure CLI

Используя [виртуальные машины Azure](../spot-vms.md) , вы можете использовать преимущества нашей неиспользуемой емкости с значительной экономией затрат. В любой момент, когда Azure нужна емкость, инфраструктура Azure будет выключать виртуальные машины Azure. Таким образом, виртуальные машины Azure для виртуальных машин отлично подходят для рабочих нагрузок, которые могут обрабатывать прерывания, такие как задания пакетной обработки, среды разработки и тестирования, большие вычислительные рабочие нагрузки и многое другое.

Цены на виртуальные машины Azure для машинного хранения являются переменными на основе региона и SKU. Дополнительные сведения см. на страницах с информацией о ценах на виртуальные машины [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) и [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 

Вы можете задать максимальную цену, которую вы хотите платить, в час, для виртуальной машины. Максимальная цена для виртуальной машины точки Azure может быть задана в долларах США (USD), в которой используется до 5 десятичных разрядов. Например, значение `0.98765` определяет максимальную цену 0,98765 долларов США в час. Если вы укажете для максимальной цены значение `-1`, виртуальная машине не будет вытесняться по критерию цены. Цена для виртуальной машины будет соответствовать текущей цене на виртуальную машину Azure для машинного курса или стоимость для стандартной виртуальной машины, которая когда-либо будет меньше, если есть доступная емкость и квота. Дополнительные сведения о настройке максимальной цены см. в статье о [ценах на виртуальные машины Azure](../spot-vms.md#pricing).

Процесс создания виртуальной машины Azure с помощью Azure CLI аналогичен описанному в [статье Краткое руководство](./quick-create-cli.md). Просто добавьте параметр "--Priority плашка", установите для параметра значение " `--eviction-policy` освобождение" (по умолчанию) или `Delete` и укажите максимальную цену или `-1` . 


## <a name="install-azure-cli"></a>Установка Azure CLI

Чтобы создать виртуальные машины Azure для точки, необходимо запустить Azure CLI версии 2.0.74 или более поздней. Чтобы узнать версию, выполните команду **az --version**. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). 

Затем войдите в Azure с помощью команды [az login](/cli/azure/reference-index#az-login).

```azurecli
az login
```

## <a name="create-an-azure-spot-virtual-machine"></a>Создание виртуальной машины с помощью точки Azure

В этом примере показано, как развернуть виртуальную машину Linux в Azure, которая не будет исключена на основе цены. Политика вытеснения устанавливается таким образом, чтобы освободить виртуальную машину, чтобы ее можно было перезапустить позднее. Если вы хотите удалить виртуальную машину и базовый диск при удалении виртуальной машины, задайте для параметра значение `--eviction-policy` `Delete` .

```azurecli
az group create -n mySpotGroup -l eastus
az vm create \
    --resource-group mySpotGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --max-price -1 \
    --eviction-policy Deallocate
```



После создания виртуальной машины можно выполнить запрос, чтобы увидеть максимальную стоимость выставления счетов для всех виртуальных машин в группе ресурсов.

```azurecli
az vm list \
   -g mySpotGroup \
   --query '[].{Name:name, MaxPrice:billingProfile.maxPrice}' \
   --output table
```

## <a name="simulate-an-eviction"></a>Имитация вытеснения

Вы можете [имитировать вытеснение](/rest/api/compute/virtualmachines/simulateeviction) виртуальной машины в машинном коде Azure, чтобы проверить, насколько хорошо ваше приложение будет ответил внезапного вытеснения. 

Замените следующие сведения: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```
`Response Code: 204` означает, что имитация вытеснения прошла успешно. 

**Дальнейшие действия**

Вы также можете создать виртуальную машину Azure с помощью [Azure PowerShell](../windows/spot-powershell.md), [портала](../spot-portal.md)или [шаблона](spot-template.md).

Запросите текущие сведения о ценах с помощью [API розничных цен Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) , чтобы получить сведения о виртуальной машине Azure для точки. Объект `meterName` и `skuName` будет содержать `Spot` .

Если возникает ошибка, см. раздел [коды ошибок](../error-codes-spot.md).
