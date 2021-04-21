---
title: Развертывание виртуальных машин и экземпляров масштабируемых наборов на выделенных узлах с помощью интерфейса командной строки
description: Развертывание виртуальных машин и экземпляров масштабируемых наборов на выделенных узлах с помощью интерфейса командной строки Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: dedicated-hosts
ms.topic: how-to
ms.date: 11/12/2020
ms.author: cynthn
ms.openlocfilehash: 9d4117cafd665556fb60278aa4dc60dc14a27ada
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101670526"
---
# <a name="deploy-to-dedicated-hosts-using-the-azure-cli"></a>Выполнение развертывания на выделенных узлах с помощью интерфейса командной строки Azure
 

В этой статье рассказывается, как создать [выделенный узел](../dedicated-hosts.md) Azure для размещения виртуальных машин. 

Убедитесь, что у вас установлен интерфейс командной строки Azure версии 2.16.0 или новее, и что вы вошли в учетную запись Azure с помощью команды `az login`. 


## <a name="limitations"></a>Ограничения

- Размеры и типы оборудования, доступные для выделенных узлов, зависят от региона. Дополнительные сведения см. на [странице с ценами узлов](https://aka.ms/ADHPricing).

## <a name="create-resource-group"></a>Создать группу ресурсов 
Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов с помощью команды az group create. В приведенном ниже примере создается группа ресурсов с именем *myDHResourceGroup* в расположении *Восточная часть США*.

```azurecli-interactive
az group create --name myDHResourceGroup --location eastus 
```
 
## <a name="list-available-host-skus-in-a-region"></a>Получение списка доступных номеров SKU узлов в регионе

Не все номера SKU узлов доступны во всех регионах и зонах доступности. 

Перед тем как приступать к подготовке выделенных узлов, получите сведения о доступности узлов и ограничениях предложений. 

```azurecli-interactive
az vm list-skus -l eastus2  -r hostGroups/hosts  -o table  
```
 
## <a name="create-a-host-group"></a>Создание группы узлов 

**Группа узлов** — это ресурс, который представляет собой коллекцию выделенных узлов. Группа узлов создается в регионе и зоне доступности, и в нее добавляются узлы. При планировании высокого уровня доступности имеются дополнительные параметры. Для выделенных узлов можно использовать один или оба следующих варианта: 
- Развертывание в нескольких зонах доступности. В этом случае группа узлов должна быть в каждой из зон, которую необходимо использовать.
- Развертывание в нескольких доменах сбоя, сопоставленных с физическими стойками. 
 
В любом случае необходимо указать количество доменов сбоя для группы узлов. Если группа не должна охватывать несколько доменов сбоя, используйте число доменов сбоя, равное 1. 

Можно также использовать как зоны доступности, так и домены сбоя. 


В этом примере мы используем команду [az vm host group create](/cli/azure/vm/host/group#az-vm-host-group-create) для создания группы узлов с использованием как зон доступности, так и доменов сбоя. 

```azurecli-interactive
az vm host group create \
   --name myHostGroup \
   -g myDHResourceGroup \
   -z 1 \
   --platform-fault-domain-count 2 
``` 

Добавьте параметр `--automatic-placement true`, чтобы виртуальные машины и экземпляры масштабируемых наборов автоматически размещались на узлах, в определенной группе узлов. Дополнительные сведения см. в разделе [Ручное и автоматическое размещение ](../dedicated-hosts.md#manual-vs-automatic-placement).


### <a name="other-examples"></a>Другие примеры

Вы также можете использовать команду [az vm host group create](/cli/azure/vm/host/group#az-vm-host-group-create) для создания группы узлов в зоне доступности 1 (без доменов сбоя).

```azurecli-interactive
az vm host group create \
   --name myAZHostGroup \
   -g myDHResourceGroup \
   -z 1 \
   --platform-fault-domain-count 1 
```
 
В приведенном ниже примере команда [az vm host group create](/cli/azure/vm/host/group#az-vm-host-group-create) применяется для создания группы узлов с использованием только доменов сбоя (для регионов, где зоны доступности не поддерживаются). 

```azurecli-interactive
az vm host group create \
   --name myFDHostGroup \
   -g myDHResourceGroup \
   --platform-fault-domain-count 2 
```
 
## <a name="create-a-host"></a>Создание узла 

Теперь создадим выделенный узел в группе узлов. Помимо имени узла, необходимо указать его номер SKU. Номер SKU узла определяет поддерживаемую серию виртуальных машин, а также поколение оборудования для выделенного узла.  

Дополнительные сведения о номерах SKU и ценах узлов см. на странице [цен на выделенные узлы Azure](https://aka.ms/ADHPricing).

Создайте узел с помощью команды [az vm host create](/cli/azure/vm/host#az-vm-host-create). Если для группы узлов задано число доменов сбоя, вам будет предложено указать домен сбоя для узла.  

```azurecli-interactive
az vm host create \
   --host-group myHostGroup \
   --name myHost \
   --sku DSv3-Type1 \
   --platform-fault-domain 1 \
   -g myDHResourceGroup
```


 
## <a name="create-a-virtual-machine"></a>Создание виртуальной машины 
Создайте виртуальную машину в выделенном узле с помощью команды [az vm create](/cli/azure/vm#az-vm-create). Если при создании группы узлов была указана зона доступности, то при создании виртуальной машины необходимо использовать ту же зону.

```azurecli-interactive
az vm create \
   -n myVM \
   --image debian \
   --host-group myHostGroup \
   --generate-ssh-keys \
   --size Standard_D4s_v3 \
   -g myDHResourceGroup \
   --zone 1
```

Чтобы разместить виртуальную машину на определенном узле, используйте `--host` вместо указания группы узлов с помощью `--host-group`.
 
> [!WARNING]
> Если виртуальная машина создается в узле, в котором недостаточно ресурсов, она будет создана в состоянии сбоя. 

## <a name="create-a-scale-set"></a>Создание масштабируемого набора 

При развертывании масштабируемого набора необходимо указывать группу узлов.

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --host-group myHostGroup \
  --generate-ssh-keys \
  --size Standard_D4s_v3 \
  -g myDHResourceGroup \
  --zone 1
```

Если вы хотите вручную выбрать узел для развертывания масштабируемого набора, добавьте `--host` и имя узла.


## <a name="check-the-status-of-the-host"></a>Проверка состояния узла

Вы можете проверить состояние работоспособности узла и оставшееся количество виртуальных машин, которое можно развернуть в нем, с помощью команды [az vm host get-instance-view](/cli/azure/vm/host#az-vm-host-get-instance-view).

```azurecli-interactive
az vm host get-instance-view \
   -g myDHResourceGroup \
   --host-group myHostGroup \
   --name myHost
```
 Выходные данные будут выглядеть следующим образом.
 
```json
{
  "autoReplaceOnFailure": true,
  "hostId": "6de80643-0f45-4e94-9a4c-c49d5c777b62",
  "id": "/subscriptions/10101010-1010-1010-1010-101010101010/resourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts/myHost",
  "instanceView": {
    "assetId": "12345678-1234-1234-abcd-abc123456789",
    "availableCapacity": {
      "allocatableVms": [
        {
          "count": 31.0,
          "vmSize": "Standard_D2s_v3"
        },
        {
          "count": 15.0,
          "vmSize": "Standard_D4s_v3"
        },
        {
          "count": 7.0,
          "vmSize": "Standard_D8s_v3"
        },
        {
          "count": 3.0,
          "vmSize": "Standard_D16s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32-8s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32-16s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D48s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64-16s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64-32s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64s_v3"
        }
      ]
    },
    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Provisioning succeeded",
        "level": "Info",
        "message": null,
        "time": "2019-07-24T21:22:40.604754+00:00"
      },
      {
        "code": "HealthState/available",
        "displayStatus": "Host available",
        "level": "Info",
        "message": null,
        "time": null
      }
    ]
  },
  "licenseType": null,
  "location": "eastus2",
  "name": "myHost",
  "platformFaultDomain": 1,
  "provisioningState": "Succeeded",
  "provisioningTime": "2019-07-24T21:22:40.604754+00:00",
  "resourceGroup": "myDHResourceGroup",
  "sku": {
    "capacity": null,
    "name": "DSv3-Type1",
    "tier": null
  },
  "tags": null,
  "type": null,
  "virtualMachines": [
    {
      "id": "/subscriptions/10101010-1010-1010-1010-101010101010/resourceGroups/MYDHRESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/MYVM",
      "resourceGroup": "MYDHRESOURCEGROUP"
    }
  ]
}

```
 
## <a name="export-as-a-template"></a>Экспорт в качестве шаблона 
Вы можете экспортировать шаблон, если вам требуется создать дополнительную среду разработки с теми же параметрами или соответствующую рабочую среду. Resource Manager использует шаблоны JSON, которые определяют все параметры среды. Можно полностью создавать среды, ссылаясь на этот шаблон JSON. Вы можете создавать шаблоны JSON вручную или экспортировать существующую среду для создания шаблона JSON автоматически. Чтобы экспортировать группу ресурсов, используйте команду [az group export](/cli/azure/group#az-group-export).

```azurecli-interactive
az group export --name myDHResourceGroup > myDHResourceGroup.json 
```

С помощью этой команды в текущем рабочем каталоге создается файл `myDHResourceGroup.json`. При создании среды на основе этого шаблона запрашиваются все имена ресурсов. Эти имена можно указать в файле шаблона, добавив параметр `--include-parameter-default-value` в команду `az group export`. Измените шаблон JSON, чтобы указать имена ресурсов, или создайте файл parameters.json, задающий их.
 
Чтобы создать среду с помощью шаблона, используйте команду [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create).

```azurecli-interactive
az deployment group create \ 
    --resource-group myNewResourceGroup \ 
    --template-file myDHResourceGroup.json 
```


## <a name="clean-up"></a>Очистка 

Вы платите за выделенные узлы, даже если виртуальные машины не развернуты. Для экономии затрат следует удалить все узлы, которые в настоящее время не используются.  

Узел можно удалить только в том случае, если он больше не используется виртуальными машинами. Удалите виртуальные машины с помощью команды [az vm delete](/cli/azure/vm#az-vm-delete).

```azurecli-interactive
az vm delete -n myVM -g myDHResourceGroup
```

После удаления виртуальных машин можно удалить узел с помощью команды [az vm host delete](/cli/azure/vm/host#az-vm-host-delete).

```azurecli-interactive
az vm host delete -g myDHResourceGroup --host-group myHostGroup --name myHost 
```
 
После удаления всех узлов можно удалить группу узлов с помощью команды [az vm host group delete](/cli/azure/vm/host/group#az-vm-host-group-delete).  
 
```azurecli-interactive
az vm host group delete -g myDHResourceGroup --host-group myHostGroup  
```
 
Вы также можете удалить всю группу ресурсов одной командой. Это приведет к удалению всех ресурсов, созданных в группе, включая все виртуальные машины, узлы и группы узлов.
 
```azurecli-interactive
az group delete -n myDHResourceGroup 
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения см. в [обзоре выделенных узлов](../dedicated-hosts.md).

- Выделенные узлы можно также создавать с помощью [портала Azure](../dedicated-hosts-portal.md).

- [Здесь](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md) можно найти пример шаблона, в котором используются как зоны, так и домены сбоя для обеспечения максимальной устойчивости в регионе.
