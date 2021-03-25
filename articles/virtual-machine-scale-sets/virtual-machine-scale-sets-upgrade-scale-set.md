---
title: Изменение масштабируемого набора виртуальных машин Azure
description: Узнайте, как изменить и обновить масштабируемый набор виртуальных машин Azure с помощью интерфейсов REST API, Azure PowerShell и Azure CLI.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 03/10/2020
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 45c52e141ed2d93f89c9d7c1426a9d7cbbbd47f5
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105023934"
---
# <a name="modify-a-virtual-machine-scale-set"></a>Изменение масштабируемого набора виртуальных машин

На протяжении жизненного цикла приложений может потребоваться изменить или обновить масштабируемый набор виртуальных машин. Это может быть обновление конфигурации масштабируемого набора или изменение конфигурации приложения. В этой статье описывается, как можно изменить существующий масштабируемый набор с помощью интерфейсов REST API, Azure PowerShell или Azure CLI.

## <a name="fundamental-concepts"></a>Базовые понятия

### <a name="the-scale-set-model"></a>Модель масштабируемого набора
Масштабируемый набор содержит модель, которая фиксирует *требуемое* состояние набора в целом. Для запроса модели масштабируемого набора можно использовать: 

- REST API с [compute/virtualmachinescalesets/get](/rest/api/compute/virtualmachinescalesets/get), как показано ниже.

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Командлет [Get-AzVmss](/powershell/module/az.compute/get-azvmss) в Azure PowerShell:

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
    ```

- Команду [az vmss show](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet
    ```

- Можно также использовать [resources.azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/) для конкретного языка.

Точное представление выходных данных зависит от параметров, введенных в команде. Ниже показан сокращенный пример выходных данных Azure CLI.

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "overprovision": true,
  "plan": null,
  "singlePlacementGroup": true,
  "sku": {
    "additionalProperties": {},
    "capacity": 1,
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Эти свойства применяются к масштабируемому набору в целом.


### <a name="the-scale-set-instance-view"></a>Представление экземпляра масштабируемого набора
Масштабируемый набор также имеет представление экземпляра, которое фиксирует текущее состояние *среды выполнения* масштабируемого набора в целом. Для запроса представления экземпляра масштабируемого набора можно использовать следующие средства.

- REST API с [compute/virtualmachinescalesets/getinstanceview](/rest/api/compute/virtualmachinescalesets/getinstanceview), как показано ниже.

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/instanceView?api-version={apiVersion}
    ```

- Командлет [Get-AzVmss](/powershell/module/az.compute/get-azvmss) в Azure PowerShell:

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceView
    ```

- Azure CLI с помощью команды [AZ vmss Get-instance-View](/cli/azure/vmss):

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
    ```

- Вы также можете использовать [Resources.Azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/) для конкретного языка.

Точное представление выходных данных зависит от параметров, введенных в команде. Ниже показан сокращенный пример выходных данных Azure CLI.

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
{
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": null,
      "time": "{time}"
    }
  ],
  "virtualMachine": {
    "additionalProperties": {},
    "statusesSummary": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "count": 1
      }
    ]
  }
}
```

Эти свойства предоставляют сводку текущего состояния среды выполнения виртуальных машин в масштабируемом наборе, включая состояние расширений, применяемых к набору.


### <a name="the-scale-set-vm-model-view"></a>Представление модели виртуальной машины масштабируемого набора
Как и масштабируемый набор, каждый экземпляр виртуальной машины в наборе имеет свое представление модели. Для запроса представления модели экземпляра определенной виртуальной машины в масштабируемом наборе можно использовать следующие средства.

- REST API с [compute/virtualmachinescalesetvms/get](/rest/api/compute/virtualmachinescalesetvms/get), как показано ниже.

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId?api-version={apiVersion}
    ```

- Командлет [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm) в Azure PowerShell:

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Команду [az vmss show](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Можно также использовать [resources.azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/).

Точное представление выходных данных зависит от параметров, введенных в команде. Ниже показан сокращенный пример выходных данных Azure CLI.

```azurecli
$ az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "name": "{name}",
  "sku": {
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Эти свойства описывают конфигурацию экземпляра виртуальной машины в масштабируемом наборе, а не конфигурацию масштабируемого набора в целом. Например, модель масштабируемого набора имеет свойство `overprovision`, которого нет у модели экземпляра виртуальной машины в наборе. Это отличие вызвано тем, что избыточная подготовка является свойством масштабируемого набора в целом, а не отдельных экземпляров виртуальных машин в наборе (дополнительные сведения об избыточной подготовке см. [в этом разделе](virtual-machine-scale-sets-design-overview.md#overprovisioning)).


### <a name="the-scale-set-vm-instance-view"></a>Представление экземпляра виртуальной машины в масштабируемом наборе
Как и масштабируемый набор, каждый экземпляр виртуальной машины в наборе имеет свое представление. Для запроса представления определенного экземпляра виртуальной машины в масштабируемом наборе можно использовать следующие средства.

- REST API с [compute/virtualmachinescalesetvms/getinstanceview](/rest/api/compute/virtualmachinescalesetvms/getinstanceview), как показано ниже.

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId/instanceView?api-version={apiVersion}
    ```

- Командлет [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm) в Azure PowerShell:

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -InstanceView
    ```

- Можно использовать команду [az vmss get-instance-view](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Вы также можете использовать [Resources.Azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/) .

Точное представление выходных данных зависит от параметров, введенных в команде. Ниже показан сокращенный пример выходных данных Azure CLI.

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
{
  "additionalProperties": {
    "osName": "ubuntu",
    "osVersion": "16.04"
  },
  "disks": [
    {
      "name": "{name}",
      "statuses": [
        {
          "additionalProperties": {},
          "code": "ProvisioningState/succeeded",
          "displayStatus": "Provisioning succeeded",
          "time": "{time}"
        }
      ]
    }
  ],
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "time": "{time}"
    },
    {
      "additionalProperties": {},
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ],
  "vmAgent": {
    "statuses": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready",
        "level": "Info",
        "message": "Guest Agent is running",
        "time": "{time}"
      }
    ],
    "vmAgentVersion": "{version}"
  },
}
```

Эти свойства описывают текущее состояние среды выполнения экземпляра виртуальной машины в масштабируемом наборе, включая любые расширения, применяемые к масштабируемому набору.


## <a name="how-to-update-global-scale-set-properties"></a>Как обновлять глобальные свойства масштабируемого набора
Для обновления глобального свойства масштабируемого набора необходимо обновить свойство в модели набора. Это можно сделать следующим образом.

- Можно использовать REST API с [compute/virtualmachinescalesets/createorupdate](/rest/api/compute/virtualmachinescalesets/createorupdate), как показано ниже.

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Вы можете развернуть шаблон Resource Manager, обновив глобальные свойства масштабируемого набора с помощью свойств REST API.

- Командлет [Update-AzVmss](/powershell/module/az.compute/update-azvmss) в Azure PowerShell:

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -VirtualMachineScaleSet {scaleSetConfigPowershellObject}
    ```

- Можно использовать команду [az vmss update](/cli/azure/vmss) в Azure CLI.
    - Чтобы изменить свойство, выполните следующую команду.

        ```azurecli
        az vmss update --set {propertyPath}={value}
        ```

    - Чтобы добавить объект в свойство списка масштабируемого набора, выполните следующую команду. 

        ```azurecli
        az vmss update --add {propertyPath} {JSONObjectToAdd}
        ```

    - Чтобы удалить объект из свойства списка масштабируемого набора, выполните следующую команду. 

        ```azurecli
        az vmss update --remove {propertyPath} {indexToRemove}
        ```

    - Если вы ранее развернули масштабируемый набор с помощью команды `az vmss create`, то вы можете снова выполнить команду `az vmss create`, чтобы его обновить. Проверьте, совпадают ли все свойства в команде `az vmss create` с предыдущими, за исключением тех, которые вы хотите изменить.

- Можно также использовать [resources.azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/).

После обновления модели масштабируемого набора новая конфигурация применяется ко всем новым виртуальным машинам, созданным в наборе. Тем не менее модели для существующих виртуальных машин в наборе все равно должны быть обновлены в соответствии с последней общей моделью масштабируемого набора. В модели каждой виртуальной машины есть логическое свойство `latestModelApplied`. Оно указывает, актуальна ли виртуальная машина по отношению к последней общей модели масштабируемого набора (`true` означает, что виртуальная машина обновлена).


## <a name="how-to-bring-vms-up-to-date-with-the-latest-scale-set-model"></a>Как обновить виртуальные машины в соответствии с последней моделью масштабируемого набора
Масштабируемые наборы имеют политику обновления, которая определяет, как виртуальные машины обновляются в соответствии с последней моделью масштабируемого набора. Доступны три режима политики обновления:

- **Автоматический**. В этом режиме масштабируемый набор не дает никаких гарантий относительно порядка отключения виртуальных машин. Масштабируемый набор может одновременно завершить работу всех виртуальных машин. 
- **Последовательный**. В этом режиме масштабируемый набор развертывает обновления в виде пакетов с необязательной паузой между обработкой пакетов.
- **Вручную**. В этом режиме при обновлении модели масштабируемого набора с имеющимися виртуальными машинами ничего не происходит.
 
Чтобы обновить существующие виртуальные машины, для каждой из них нужно выполнить обновление в ручном режиме. Это можно сделать следующим образом.

- Можно использовать REST API с [compute/virtualmachinescalesets/updateinstances](/rest/api/compute/virtualmachinescalesets/updateinstances), как показано ниже.

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/manualupgrade?api-version={apiVersion}
    ```

- Командлет [Update-AzVmssInstance](/powershell/module/az.compute/update-azvmssinstance) в Azure PowerShell:
    
    ```powershell
    Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Можно использовать команду [az vmss update-instances](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
    ```

- Можно также использовать [пакеты SDK Azure](https://azure.microsoft.com/downloads/) для конкретного языка.

>[!NOTE]
> В кластерах Service Fabric можно использовать только *автоматический* режим, но обновление обрабатывается по-разному. Дополнительные сведения см. в разделе [Service Fabric обновления приложения](../service-fabric/service-fabric-application-upgrade.md).

Существует один тип изменения глобальных свойств масштабируемого набора, который не соответствует политике обновления. Изменения в профилях ОС и дисков данных масштабируемого набора (например, имя пользователя и пароль администратора) можно изменить только в API версии *2017-12-01* или более поздней. Эти изменения применяются только к виртуальным машинам, созданным после изменения модели масштабируемого набора. Чтобы обеспечить актуальность существующих виртуальных машин, необходимо пересоздать образ каждой существующей виртуальной машины. Это можно сделать следующим образом.

- Можно использовать REST API с [compute/virtualmachinescalesets/reimage](/rest/api/compute/virtualmachinescalesets/reimage), как показано ниже.

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/reimage?api-version={apiVersion}
    ```

- Командлет [Set-AzVmssVm](/powershell/module/az.compute/set-azvmssvm) в Azure PowerShell:

    ```powershell
    Set-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -Reimage
    ```

- Можно использовать команду [az vmss reimage](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss reimage --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Можно также использовать [пакеты SDK Azure](https://azure.microsoft.com/downloads/) для конкретного языка.


## <a name="properties-with-restrictions-on-modification"></a>Свойства с ограничениями на изменение

### <a name="create-time-properties"></a>Свойства времени создания
Некоторые свойства можно задать только при создании масштабируемого набора. Эти свойства включают в себя:

- Зоны доступности
- издатель ссылки на образ;
- предложение ссылки на образ.
- тип учетной записи хранения управляемого диска ОС.

### <a name="properties-that-can-only-be-changed-based-on-the-current-value"></a>Свойства, которые можно изменить только на основе текущего значения
Некоторые свойства можно изменить только в зависимости от текущего значения. Эти свойства включают в себя:

- **singlePlacementGroup**. Если это свойство имеет значение true, его можно изменить на false. Однако, если значением является false, его **не возможно** изменить на true.
- **subnet**. Подсеть масштабируемого набора можно изменить, если исходная и новая подсети находятся в одной и той же виртуальной сети.
- **имажереференцеску** -SKU Reference может быть обновлен для подтвержденных [дистрибутивов Linux](../virtual-machines/linux/endorsed-distros.md), образов Windows Server, клиентов и образов без [сведений о плане](../virtual-machines/linux/cli-ps-findimage.md#check-the-purchase-plan-information). 

### <a name="properties-that-require-deallocation-to-change"></a>Свойства, для изменения которых требуется освободить виртуальные машины
Для некоторых свойств можно задать определенные значения, только если виртуальные машины в масштабируемом наборе освобождены. Эти свойства включают в себя:

- **Имя SKU**. Если новый номер SKU виртуальной машины не поддерживается на оборудовании, на котором в настоящее время находится масштабируемый набор, необходимо отменить распределение виртуальных машин в масштабируемом наборе перед изменением имени SKU. Дополнительные сведения см. в статье [Изменение размера виртуальной машины Windows](../virtual-machines/windows/resize-vm.md). 

## <a name="vm-specific-updates"></a>Обновления отдельных виртуальных машин
Некоторые изменения можно применить к отдельным виртуальным машинам, а не к глобальным свойствам масштабируемого набора. В настоящее время единственным обновлением отдельных виртуальных машин, которое поддерживается, является подключение или отключение дисков данных виртуальных машин в масштабируемом наборе. Эта функция предоставляется в предварительной версии. Дополнительные сведения см. в [документации по предварительной версии](https://github.com/Azure/vm-scale-sets/tree/master/preview/disk).


## <a name="scenarios"></a>Сценарии

### <a name="application-updates"></a>Обновления приложений
Если приложение развертывается в масштабируемом наборе с помощью расширений, то при обновлении конфигурации расширения приложение будет обновлено в соответствии с политикой обновления. Например, если у вас есть новая версия сценария для запуска в расширении пользовательских сценариев, вы можете обновить свойство *fileUris*, чтобы указать новый сценарий. В некоторых случаях вы можете принудительно выполнить обновление, даже если конфигурация расширения не изменилась (например, вы обновили сценарий без изменения его универсального кода ресурса (URI)). В таких случаях можно изменить *forceUpdateTag* для принудительного обновления. Это свойство не интерпретируется платформой Azure. Если изменить его значение, это не повлияет работу расширения. Изменение просто приведет к повторному запуску расширения. Дополнительные сведения о *forceUpdateTag* см. в [документации по REST API для расширений](/rest/api/compute/virtualmachineextensions/createorupdate). Обратите внимание, что *forceUpdateTag* может использоваться со всеми расширениями, а не только с расширением пользовательских скриптов.

Приложения также часто развертывают с помощью пользовательского образа. Этот сценарий описан в следующем разделе.

### <a name="os-updates"></a>обновления ОС;
При использовании образов платформы Azure их можно обновить, изменив *imageReference* (дополнительные сведения см. в [документации по REST API](/rest/api/compute/virtualmachinescalesets/createorupdate)).

>[!NOTE]
> Для образов платформы обычно указывается последняя версия в качестве эталонной версии образа. Во время создания, масштабирования и повторного создания образов виртуальные машины создаются на основе последней доступной версии. Тем не менее это **не** означает, что образ ОС будет автоматически обновляться по мере выпуска новых версий. Отдельный компонент, обеспечивающий автоматическое обновление ОС, в настоящий момент находится на этапе предварительной версии. Дополнительные сведения см. в [документации по автоматическому обновлению ОС](virtual-machine-scale-sets-automatic-upgrade.md).

При использовании пользовательских образов их можно обновить, изменив идентификатор *imageReference* (дополнительные сведения см. в [документации по REST API](/rest/api/compute/virtualmachinescalesets/createorupdate)).

## <a name="examples"></a>Примеры

### <a name="update-the-os-image-for-your-scale-set"></a>Обновление образа ОС масштабируемого набора
Предположим, что у вас есть масштабируемый набор, который работает под управлением старой версии Ubuntu LTS 16.04. Его нужно обновить до более новой версии Ubuntu LTS 16.04 (например, *16.04.201801090*). Свойство эталонной версии образа не входит в список, поэтому можно непосредственно изменить эти свойства с помощью одной из следующих команд.

- Командлет [Update-AzVmss](/powershell/module/az.compute/update-azvmss) в Azure PowerShell, как показано ниже.

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -ImageReferenceVersion 16.04.201801090
    ```

- Можно использовать команду [az vmss update](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss update --resource-group myResourceGroup --name myScaleSet --set virtualMachineProfile.storageProfile.imageReference.version=16.04.201801090
    ```

Также может потребоваться изменить образ, используемый масштабируемым набором. Например, может потребоваться обновить или изменить пользовательский образ, используемый масштабируемым набором. Чтобы изменить образ, используемый масштабируемым набором, измените свойство идентификатора эталонного изображения. Свойство идентификатора эталонного образа не входит в список, поэтому можно непосредственно изменить это свойство с помощью одной из следующих команд.

- Командлет [Update-AzVmss](/powershell/module/az.compute/update-azvmss) в Azure PowerShell, как показано ниже.

    ```powershell
    Update-AzVmss `
        -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -ImageReferenceId /subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```

- Можно использовать команду [az vmss update](/cli/azure/vmss) в Azure CLI.

    ```azurecli
    az vmss update \
        --resource-group myResourceGroup \
        --name myScaleSet \
        --set virtualMachineProfile.storageProfile.imageReference.id=/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```


### <a name="update-the-load-balancer-for-your-scale-set"></a>Обновление подсистемы балансировки нагрузки масштабируемого набора
Предположим, у вас есть масштабируемый набор с Azure Load Balancer и вы хотите заменить Azure Load Balancer на шлюз приложений Azure. Свойства подсистемы балансировки нагрузки и шлюза приложений для масштабируемого набора являются частью списка, поэтому вы можете использовать команды для удаления и добавления элементов списка вместо прямого изменения свойств.

- Azure PowerShell:

    ```powershell
    # Get the current model of the scale set and store it in a local PowerShell object named $vmss
    $vmss=Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
    
    # Create a local PowerShell object for the new desired IP configuration, which includes the reference to the application gateway
    $ipconf = New-AzVmssIPConfig "myNic" -ApplicationGatewayBackendAddressPoolsId /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendAddressPoolName} -SubnetId $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Subnet.Id –Name $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Name
    
    # Replace the existing IP configuration in the local PowerShell object (which contains the references to the current Azure Load Balancer) with the new IP configuration
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0] = $ipconf
    
    # Update the model of the scale set with the new configuration in the local PowerShell object
    Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -virtualMachineScaleSet $vmss
    ```

- Azure CLI:

    ```azurecli
    # Remove the load balancer backend pool from the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools 0
    
    # Remove the load balancer backend pool from the scale set model; only necessary if you have NAT pools configured on the scale set
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools 0
    
    # Add the application gateway backend pool to the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].ApplicationGatewayBackendAddressPools '{"id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendPoolName}"}'
    ```

>[!NOTE]
> Эти команды предполагают наличие только одной конфигурации IP и балансировщика нагрузки в масштабируемом наборе. Если их несколько, может потребоваться использовать индекс списка, отличный от *0*.


## <a name="next-steps"></a>Следующие шаги
Общие задачи управления масштабируемыми наборами можно также выполнять с помощью [Azure CLI](virtual-machine-scale-sets-manage-cli.md) или [Azure PowerShell](virtual-machine-scale-sets-manage-powershell.md).