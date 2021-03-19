---
title: Использование виртуализации расширения с масштабируемыми наборами виртуальных машин Azure
description: Узнайте, как упорядочить подготовку расширений при развертывании нескольких расширений в масштабируемых наборах виртуальных машин.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 01/30/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 3271041b9f4db100cd05588129c7d714d4478f10
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83121037"
---
# <a name="sequence-extension-provisioning-in-virtual-machine-scale-sets"></a>Подготовка последовательности расширений в масштабируемых наборах виртуальных машин
Расширения виртуальных машин Azure предоставляют такие возможности, как настройка и управление после развертывания, мониторинг, безопасность и многое другое. Для достижения желаемых результатов в рабочих развертываниях обычно используется сочетание нескольких расширений, настроенных для экземпляров виртуальной машины.

При использовании нескольких расширений на виртуальной машине важно убедиться, что расширения, которым требуются одни и те же ресурсы ОС, не пытаются получить эти ресурсы одновременно. Некоторые расширения также зависят от других расширений в контексте предоставления необходимых конфигураций, таких как параметры среды и секреты. Без правильного порядка и последовательности развертывания зависимых расширений может произойти сбой.

В этой статье описывается порядок настройки расширений для экземпляров виртуальных машин в масштабируемых наборах виртуальных машин.

## <a name="prerequisites"></a>Предварительные требования
Для работы с этой статьей нужны знания в следующих областях:
-   [Расширения](../virtual-machines/extensions/overview.md) виртуальной машины Azure
-   [Изменением](virtual-machine-scale-sets-upgrade-scale-set.md) масштабируемых наборов виртуальных машин.

## <a name="when-to-use-extension-sequencing"></a>Когда следует использовать последовательность расширений
Последовательность расширений необязательна для масштабируемых наборов и, если не указано иное, расширения можно подготовить в экземпляре масштабируемого набора в любом порядке.

Например, если масштабируемая модель имеет два расширения — ExtensionA и ExtensionB, указанные в модели, то можно выполнить любую из следующих последовательностей подготовки.
-   ExtensionA -> ExtensionB
-   ExtensionВ -> ExtensionА

Если приложению требуется, чтобы ExtensionA всегда подготавливалось перед ExtensionВ, следует использовать последовательность расширений, как описано в этой статье. При использовании последовательности расширений теперь будет выполняться только одна последовательность:
-   ExtensionA -> ExtensionB

Любые расширения, не указанные в определенной последовательности подготовки, могут быть подготовлены в любое время, в том числе до, после или во время заданной последовательности. Последовательность расширений только указывает, что определенное расширение будет подготовлено после другого расширения. Это не влияет на подготовку любого другого расширения, определенного в модели.

Например, если масштабируемая модель c тремя расширениями — ExtensionA, ExtensionB и ExtensionC, указанные в модели, а ExtensionC настроено для подготовки после ExtensionA, то можно выполнить любую из следующих последовательностей подготовки.
-   ExtensionA -> ExtensionC -> ExtensionB
-   ExtensionB -> ExtensionA -> ExtensionC
-   ExtensionA -> ExtensionB -> ExtensionC

Если во время выполнения определенной последовательности расширений необходимо убедиться, что другие расширения не подготавливаются, мы советуем упорядочить все расширения в масштабируемой модели. В приведенном выше примере ExtensionB может быть настроено для на подготовки после ExtensionC таким образом, чтобы могла выполняться только одна последовательность.
-   ExtensionA -> ExtensionC -> ExtensionB


## <a name="how-to-use-extension-sequencing"></a>Как следует использовать последовательность расширений
Чтобы реализовать последовательность подготовки расширений, необходимо обновить определение расширения в модели масштабируемого набора и включить свойство provisionAfterExtensions, которое принимает массив имен расширений. Расширения, указанные в значении массива свойств, должны быть полностью определены в модели масштабируемого набора.

### <a name="template-deployment"></a>Развертывание шаблона
В следующем примере определяется шаблон, в котором масштабируемый набор с тремя расширениями — ExtensionA, ExtensionB и ExtensionC, таким образом, расширения подготавливаются в следующем порядке:
-   ExtensionA -> ExtensionB -> ExtensionC

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA"
          ],
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

Та как свойство provisionAfterExtensions принимает массив имен расширений, приведенный выше пример можно изменить таким образом, что ExtensionC будет подготавливаться после ExtensionA и ExtensionB, но упорядочивание для ExtensionA и ExtensionB не потребуется. Для реализации этого сценария можно использовать следующий шаблон.

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA","ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

### <a name="rest-api"></a>REST API
В следующем примере в масштабируемую модель добавляется новое расширение ExtensionC. ExtensionC зависит от ExtensionA и ExtensionB, которые уже заданы в модели масштабируемого набора.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "name": "ExtensionC",
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ],
    "publisher": "ExtensionC.Publisher",
    "settings": {},
    "typeHandlerVersion": "3.0",
    "autoUpgradeMinorVersion": true,
    "type": "ExtensionC" 
  }                  
}
```

Если расширение ExtensionC было задано ранее в модели масштабируемого набора и теперь требуется добавить его зависимости, можно выполнить `PATCH`, чтобы изменить свойства уже развернутого расширения.

```
PATCH on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ]
  }                  
}
```
Изменения имеющихся экземпляров масштабируемого набора применяются при следующем [обновлении](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model).

### <a name="azure-powershell"></a>Azure PowerShell
Используйте командлет [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension), чтобы добавить расширение "Работоспособность приложения" в определение модели масштабируемого набора. Подследственность расширений требует использования Az PowerShell 1.2.0 или выше.

В следующем примере расширение [Работоспособность приложения](virtual-machine-scale-sets-health-extension.md) добавляется в атрибут `extensionProfile` в модели масштабируемого набора на основе Windows. Расширение "Работоспособность приложения" будет подготовлено после подготовки [расширения пользовательских сценариев](../virtual-machines/extensions/custom-script-windows.md), уже указанного в масштабируемом наборе.

```azurepowershell-interactive
# Define the scale set variables
$vmScaleSetName = "myVMScaleSet"
$vmScaleSetResourceGroup = "myVMScaleSetResourceGroup"

# Define the Application Health extension properties
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"};
$extensionName = "myHealthExtension"
$extensionType = "ApplicationHealthWindows"
$publisher = "Microsoft.ManagedServices"

# Get the scale set object
$vmScaleSet = Get-AzVmss `
  -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName

# Add the Application Health extension to the scale set model
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet `
  -Name $extensionName `
  -Publisher $publisher `
  -Setting $publicConfig `
  -Type $extensionType `
  -TypeHandlerVersion "1.0" `
  -ProvisionAfterExtension "CustomScriptExtension" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
Используйте команду [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set), чтобы добавить расширение "Работоспособность приложения" в определение модели масштабируемого набора. Подследственность расширений требует использования Azure CLI 2.0.55 или выше.

В следующем примере расширение [Работоспособность приложения](virtual-machine-scale-sets-health-extension.md) добавляется для модели масштабируемого набора на основе Windows. Расширение "Работоспособность приложения" будет подготовлено после подготовки [расширения пользовательских сценариев](../virtual-machines/extensions/custom-script-windows.md), уже указанного в масштабируемом наборе.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthWindows \
  --publisher Microsoft.ManagedServices \
  --version 1.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --provision-after-extensions CustomScriptExtension \
  --settings ./extension.json
```


## <a name="troubleshoot"></a>Диагностика

### <a name="not-able-to-add-extension-with-dependencies"></a>Не удается добавить расширение с зависимостями
1. Убедитесь, что расширения, указанные в provisionAfterExtensions, определены в модели масштабируемого набора.
2. Убедитесь в отсутствии циклических зависимостей. Например, следующая последовательность не допускается: Extension a-> Екстенсионб-> Екстенсионк-> Extension a
3. Убедитесь, что для всех расширений с зависимостями свойство "settings" указано под полем "properties" расширения. Например, если расширение ExtentionB должно быть подготовлено после ExtensionA, то для ExtensionA поле "settings" должно быть указано под полем "properties" этого же расширения. Вы можете указать пустое свойство "settings", если для расширения не требуются какие-либо обязательные параметры.

### <a name="not-able-to-remove-extensions"></a>Не удается удалить расширения
Убедитесь, что удаляемые расширения не перечислены в разделе provisionAfterExtensions для других расширений.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как [развертывать приложение](virtual-machine-scale-sets-deploy-app.md) в масштабируемых наборах виртуальных машин.
