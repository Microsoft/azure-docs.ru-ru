---
title: Расширение драйвера InfiniBand — виртуальные машины Linux в Azure
description: Расширение Microsoft Azure для установки драйверов InfiniBand на виртуальных машинах вычислений серии H и N под управлением Linux.
services: virtual-machines
documentationcenter: ''
author: vermagit
editor: ''
ms.assetid: ''
ms.service: virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/20/2020
ms.author: amverma
ms.openlocfilehash: 1219f5befb4de81850681a53c10f03dc81249d53
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102559738"
---
# <a name="infiniband-driver-extension-for-linux"></a>Расширение драйвера InfiniBand для Linux

Это расширение устанавливает драйверы InfiniBand ОФЕД на виртуальных машинах серии [H](../sizes-hpc.md) и [серии N](../sizes-gpu.md) с поддержкой INFINIBAND и SR-IOV (размер r), работающих под управлением Linux. В зависимости от семейства виртуальных машин расширение устанавливает соответствующие драйверы для сетевого адаптера Connect-X.

Инструкции по установке драйверов ОФЕД вручную доступны [здесь](../workloads/hpc/enable-infiniband.md#manual-installation).

Расширение также доступно для установки драйверов InfiniBand для [виртуальных машин Windows](hpc-compute-infiniband-windows.md).

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Это расширение поддерживает указанные ниже дистрибутивы, в зависимости от поддержки драйвера в конкретной версии ОС.

| Distribution | Версия |
|---|---|
| Linux: Ubuntu | 16,04 LTS, 18,04 LTS, 20,04 LTS |
| Linux: CentOS | 7,4, 7,5, 7,6, 7,7, 8,1, 8, 2 |
| Linux: Red Hat Enterprise Linux | 7,4, 7,5, 7,6, 7,7, 8,1, 8, 2 |

### <a name="internet-connectivity"></a>Подключение к Интернету

Для расширения Microsoft Azure для драйверов InfiniBand требуется, чтобы Целевая виртуальная машина была подключена к и была доступна в Интернете.

## <a name="extension-schema"></a>Схема расширения

В следующем коде JSON показана схема расширения.

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="properties"></a>Свойства

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 2015-06-15 | Дата |
| publisher | Microsoft.HpcCompute | строка |
| type | инфинибанддриверлинукс | строка |
| typeHandlerVersion | 1,1 | INT |



## <a name="deployment"></a>Развертывание


### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager 

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Шаблоны идеально подходят для развертывания одной или нескольких виртуальных машин, требующих настройки после развертывания.

Конфигурацию JSON для расширения виртуальной машины можно вложить в ресурс виртуальной машины или поместить в корень или на верхний уровень JSON-файла шаблона Resource Manager. Размещение конфигурации JSON влияет на значения имени и типа ресурса. Дополнительные сведения см. в разделе [Указание имени и типа дочернего ресурса в шаблоне Resource Manager](../../azure-resource-manager/templates/child-resource-name-type.md). 

В следующем примере предполагается, что расширение виртуальной машины расположено в ресурсе виртуальной машины. При вложении ресурса расширения JSON помещается в объект `"resources": []` виртуальной машины.

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="powershell"></a>PowerShell

```powershell
Set-AzVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "InfiniBandDriverLinux" `
    -ExtensionType "InfiniBandDriverLinux" `
    -TypeHandlerVersion 1.1 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name InfiniBandDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.1 
```

### <a name="add-extension-to-a-virtual-machine-scale-set"></a>Добавление расширения в масштабируемый набор виртуальных машин

В следующем примере устанавливается последняя 1,1 версия расширения Инфинибанддриверлинукс на всех виртуальных машинах с поддержкой RDMA в существующем масштабируемом наборе виртуальных машин с именем *myVMSS* , развернутом в группе ресурсов с именем *myResourceGroup*:

  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverLinux" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverLinux" -TypeHandlerVersion "1.1"
  Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "MyVMSS" -VirtualMachineScaleSet $VMSS
  Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS" -InstanceId "*"
```


## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Сведения о состоянии развертывания расширения можно получить на портале Azure, а также с помощью Azure PowerShell или Azure CLI. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду.

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Выходные данные выполнения расширения записываются в следующий файл. Сведения о состоянии установки, а также об устранении любых сбоев см. в этом файле.

```bash
/var/log/azure/ib-vmext-status
```

### <a name="exit-codes"></a>Коды выхода

В следующей таблице приводится описание и рекомендуемые действия, основанные на кодах выхода процесса установки расширения.

| Код завершения | Значение | Возможное действие |
| :---: | --- | --- |
| 0 | Операция выполнена успешно |
| 1 | Неправильное использование расширения | Проверьте выходные данные журнала выполнения |
| 10 | Службы Integration Services в Linux для Hyper-V и Azure недоступны или не установлены | Проверьте выходные данные lspci |
| 11 | Mellanox InfiniBand не найден на этом размере виртуальной машины | Используйте [поддерживаемые размер виртуальной машины и ОС](../sizes-hpc.md) |
| 12 | Предложение образа не поддерживается |
| 13 | Размер виртуальной машины не поддерживается | Для развертывания используйте виртуальную [машину серии n (r) с](../sizes-hpc.md) поддержкой [InfiniBand и серия n](../sizes-gpu.md) |
| 14 | Не удалось выполнить операцию. | Проверьте выходные данные журнала выполнения |


### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/community/). Обращение в службу поддержки можно также выполнить на [сайте поддержки Azure](https://azure.microsoft.com/support/options/). Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о размерах (r) с поддержкой InfiniBand см. в статье виртуальные машины серии [H](../sizes-hpc.md) и [N](../sizes-gpu.md) .

> [!div class="nextstepaction"]
> [Дополнительные сведения о расширениях и компонентах виртуальных машин Linux](features-linux.md)