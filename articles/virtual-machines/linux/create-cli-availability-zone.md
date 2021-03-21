---
title: Создание виртуальной машины с зоной с помощью Azure CLI
description: Создание виртуальной машины в зоне доступности с помощью Azure CLI
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.date: 04/05/2018
ms.author: cynthn
ms.openlocfilehash: 3405468c64a7130bb0d71543403dd4ce5ef9c241
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102558378"
---
# <a name="create-a-virtual-machine-in-an-availability-zone-using-azure-cli"></a>Создание виртуальной машины в зоне доступности с помощью Azure CLI

В этой статье рассматривается использование Azure CLI для создания виртуальной машины Linux в зоне доступности Azure. [Зона доступности](../../availability-zones/az-overview.md) — это физически отдельная зона в регионе Azure. Зоны доступности позволяют защитить приложения и данные от маловероятных сбоев и потери всего центра обработки данных.

Чтобы использовать зону доступности, создайте виртуальную машину в [поддерживаемом регионе Azure](../../availability-zones/az-region.md).

Обязательно установите последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войдите в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index).


## <a name="check-vm-sku-availability"></a>Проверка доступности SKU виртуальной машины
Доступность размеров или номеров SKU виртуальных машин зависит от региона и зоны. Чтобы спланировать использование с учетом зон доступности, можно составить список доступных SKU виртуальных машин по регионам и зонам Azure. Так вы сможете выбрать нужный размер виртуальной машины и получить требуемую степень устойчивости в разных зонах. См. дополнительные сведения о разных типах и [размерах виртуальных машин](../sizes.md).

Доступные номера SKU виртуальной машины можно просмотреть с помощью команды [az vm list-skus](/cli/azure/vm). В следующем примере перечислены доступные номера SKU виртуальных машин в регионе *eastus2*:

```azurecli
az vm list-skus --location eastus2 --output table
```

Вывод команды будет примерно таким, как в следующем сокращенном примере с отображением размеров виртуальных машин, используемых в пределах разных зон доступности:

```output
ResourceType      Locations  Name               [...]    Tier       Size     Zones
----------------  ---------  -----------------           ---------  -------  -------
virtualMachines   eastus2    Standard_DS1_v2             Standard   DS1_v2   1,2,3
virtualMachines   eastus2    Standard_DS2_v2             Standard   DS2_v2   1,2,3
[...]
virtualMachines   eastus2    Standard_F1s                Standard   F1s      1,2,3
virtualMachines   eastus2    Standard_F2s                Standard   F2s      1,2,3
[...]
virtualMachines   eastus2    Standard_D2s_v3             Standard   D2_v3    1,2,3
virtualMachines   eastus2    Standard_D4s_v3             Standard   D4_v3    1,2,3
[...]
virtualMachines   eastus2    Standard_E2_v3              Standard   E2_v3    1,2,3
virtualMachines   eastus2    Standard_E4_v3              Standard   E4_v3    1,2,3
```


## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group).  

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов следует создавать до виртуальной машины. В этом примере создается группа ресурсов с именем *myResourceGroupVM* в регионе *eastus2*. "Восточная часть США 2" – это пример региона в Azure, поддерживающего зоны доступности.

```azurecli 
az group create --name myResourceGroupVM --location eastus2
```

Группа ресурсов указывается при создании или изменении виртуальной машины, что рассматривается в этой статье.

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm). 

При создании виртуальной машины доступно несколько вариантов, таких как образ операционной системы, определение размера диска и учетные данные администратора. В этом примере создается виртуальная машина *myVM* под управлением Ubuntu Server. Она создается в зоне доступности *1*. По умолчанию она имеет размер *Standard_DS1_v2*.

```azurecli-interactive
az vm create --resource-group myResourceGroupVM --name myVM --location eastus2 --image UbuntuLTS --generate-ssh-keys --zone 1
```

Создание виртуальной машины может занять несколько минут. После создания виртуальной машины Azure CLI выводит информацию о ней. Запишите значение `zones`. Оно указывает на зону доступности, в которой запущена виртуальная машина. 

```output
{
  "fqdns": "",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus2",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM",
  "zones": "1"
}
```

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Подтверждение зоны для управляемого диска и IP-адреса

При развертывании виртуальной машины в зоне доступности диск виртуальной машины создается в той же зоне доступности. По умолчанию общедоступный IP-адрес также создается в этой зоне. В примерах ниже содержатся сведения об этих ресурсах.

Чтобы убедиться, что управляемый диск виртуальной машины находится в зоне доступности, используйте команду [AZ VM](/cli/azure/vm) -Command, чтобы получить идентификатор диска. В этом примере идентификатор диска хранится в переменной, которая будет использоваться на более позднем этапе. 

```azurecli-interactive
osdiskname=$(az vm show -g myResourceGroupVM -n myVM --query "storageProfile.osDisk.name" -o tsv)
```
Теперь можно получить сведения об управляемом диске:

```azurecli-interactive
az disk show --resource-group myResourceGroupVM --name $osdiskname
```

В выходных данных видно, что управляемый диск находится в той же зоне доступности, что и виртуальная машина:

```output
{
  "creationData": {
    "createOption": "FromImage",
    "imageReference": {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westeurope/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/16.04-LTS/Versions/latest",
      "lun": null
    },
    "sourceResourceId": null,
    "sourceUri": null,
    "storageAccountId": null
  },
  "diskSizeGb": 30,
  "encryptionSettings": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/disks/osdisk_761c570dab",
  "location": "eastus2",
  "managedBy": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "name": "myVM_osdisk_761c570dab",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroupVM",
  "sku": {
    "name": "Premium_LRS",
    "tier": "Premium"
  },
  "tags": {},
  "timeCreated": "2018-03-05T22:16:06.892752+00:00",
  "type": "Microsoft.Compute/disks",
  "zones": [
    "1"
  ]
}
```

Используйте команду [az vm list-ip-addresses](/cli/azure/vm), чтобы вернуть имя ресурса общедоступного IP-адреса на виртуальную машину *myVM*. В этом примере имя хранится в переменной, которая будет использоваться дальше.

```azurecli
ipaddressname=$(az vm list-ip-addresses -g myResourceGroupVM -n myVM --query "[].virtualMachine.network.publicIpAddresses[].name" -o tsv)
```

Теперь можно получить сведения об IP-адресе:

```azurecli
az network public-ip show --resource-group myResourceGroupVM --name $ipaddressname
```

В выходных данных видно, что IP-адрес находится в той же зоне доступности, что и виртуальная машина:

```output
{
  "dnsSettings": null,
  "etag": "W/\"b7ad25eb-3191-4c8f-9cec-c5e4a3a37d35\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "52.174.34.95",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM",
    "name": null,
    "privateIpAddress": null,
    "privateIpAllocationMethod": null,
    "provisioningState": null,
    "publicIpAddress": null,
    "resourceGroup": "myResourceGroupVM",
    "subnet": null
  },
  "location": "eastUS2",
  "name": "myVMPublicIP",
  "provisioningState": "Succeeded",
  "publicIpAddressVersion": "IPv4",
  "publicIpAllocationMethod": "Dynamic",
  "resourceGroup": "myResourceGroupVM",
  "resourceGuid": "8c70a073-09be-4504-0000-000000000000",
  "tags": {},
  "type": "Microsoft.Network/publicIPAddresses",
  "zones": [
    "1"
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статье вы узнали, как создать виртуальную машину в зоне доступности. Дополнительные сведения о [доступности](../availability.md) для виртуальных машин Azure.