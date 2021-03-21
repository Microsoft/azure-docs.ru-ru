---
title: Расширение драйвера GPU NVIDIA для виртуальных машин Linux в Azure
description: Расширение Microsoft Azure для установки драйверов GPU NVIDIA на вычислительных виртуальных машинах серии N под управлением Linux.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 01/21/2021
ms.author: amverma
ms.openlocfilehash: fa2b82f3246fd87830010f572ba23aa2df075053
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102566232"
---
# <a name="nvidia-gpu-driver-extension-for-linux"></a>Расширение драйвера GPU NVIDIA для Linux

## <a name="overview"></a>Обзор

Это расширение устанавливает драйверы GPU NVIDIA на виртуальных машинах серии N для Linux. В зависимости от семейства виртуальных машин расширение устанавливает драйверы CUDA или GRID. При установке драйверов NVIDIA с помощью этого расширения требуется принять условия [лицензионного соглашения NVIDIA](https://go.microsoft.com/fwlink/?linkid=874330). Во время установки драйвера виртуальная машина может быть перезагружена для завершения процедуры.

Инструкции по установке драйверов вручную и сведения о поддерживаемых сейчас версиях приводятся [здесь](../linux/n-series-driver-setup.md).
Это расширение также доступно для установки драйверов GPU NVIDIA на [виртуальных машинах Windows серии N](hpccompute-gpu-windows.md).

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Это расширение поддерживает указанные ниже дистрибутивы, в зависимости от поддержки драйвера в конкретной версии ОС.

| Distribution | Версия |
|---|---|
| Linux: Ubuntu | 16.04 LTS, 18.04 LTS |
| Linux: Red Hat Enterprise Linux | 7,3, 7,4, 7,5, 7,6, 7,7, 7,8 |
| Linux: CentOS | 7,3, 7,4, 7,5, 7,6, 7,7, 7,8 |

### <a name="internet-connectivity"></a>Подключение к Интернету

Для работы расширения Microsoft Azure для драйверов GPU NVIDIA требуется, чтобы целевая виртуальная машина была доступна и подключена к Интернету.

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
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
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
| type | NvidiaGpuDriverLinux | строка |
| typeHandlerVersion | 1,3 | INT |

### <a name="settings"></a>Настройки

Все эти параметры не являются обязательными. По умолчанию ядро не обновляется (если это не требуется для установки драйвера), устанавливается последний поддерживаемый драйвер и набор инструментов CUDA Toolkit (если это применимо).

| Имя | Описание | Значение по умолчанию | Допустимые значения | Тип данных |
| ---- | ---- | ---- | ---- | ---- |
| updateOS | Обновление ядра, даже если для установки драйвера это не требуется | false | true, false | Логическое |
| driverVersion | NV: версия драйвера GRID.<br> NC/ND: версия набора инструментов CUDA Toolkit. Последние версии драйверов для выбранного набора CUDA Toolkit устанавливаются автоматически. | последняя | [Список](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json) поддерживаемых версий драйверов | строка |
| installCUDA | Установка набора инструментов CUDA Toolkit. Это относится только к виртуальным машинам серии NC/ND. | Да | true, false | Логическое |


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
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
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
    -ExtensionName "NvidiaGpuDriverLinux" `
    -ExtensionType "NvidiaGpuDriverLinux" `
    -TypeHandlerVersion 1.3 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

В следующем примере показаны приведенные выше Azure Resource Manager и примеры PowerShell.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 
```

В следующем примере также добавляются два дополнительных настраиваемых параметра в качестве примера для установки драйвера, не установленного по умолчанию. В частности, он обновляет ядро ОС до последней версии и устанавливает конкретный драйвер CUDA Toolkit. Обратите внимание, что параметр "--Settings" является необязательным и имеет значение по умолчанию. Обратите внимание, что обновление ядра может увеличить время установки расширения. Кроме того, выбор определенной (более старой) версии CUDA толкит может быть не всегда совместим с новыми ядрами.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 \
  --settings '{ \
    "updateOS": true, \
    "driverVersion": "10.0.130" \
  }'
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

Выходные данные выполнения расширения записываются в следующий файл. Сведения о состоянии установки (любой длительный запуск) и об устранении любых сбоев см. в этом файле.

```bash
/var/log/azure/nvidia-vmext-status
```

### <a name="exit-codes"></a>Коды выхода

| Код завершения | Значение | Возможное действие |
| :---: | --- | --- |
| 0 | Операция выполнена успешно |
| 1 | Неправильное использование расширения | Проверьте выходные данные журнала выполнения |
| 10 | Службы Integration Services в Linux для Hyper-V и Azure недоступны или не установлены | Проверьте выходные данные lspci |
| 11 | Для этого размера виртуальной машины не обнаружен драйвер GPU NVIDIA | Используйте [поддерживаемые размер виртуальной машины и ОС](../linux/n-series-driver-setup.md) |
| 12 | Предложение образа не поддерживается |
| 13 | Размер виртуальной машины не поддерживается | Для развертывания используйте виртуальную машину серии N |
| 14 | Не удалось выполнить операцию. | Проверьте выходные данные журнала выполнения |


### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/community/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о расширениях виртуальных машин см. в обзоре [расширений и компонентов виртуальной машины для Linux](features-linux.md).

См. дополнительные сведения о [размерах виртуальных машин серии N, оптимизированных для GPU](../sizes-gpu.md).
