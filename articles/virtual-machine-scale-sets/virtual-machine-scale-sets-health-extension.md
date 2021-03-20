---
title: Использование расширения работоспособности приложений с масштабируемыми наборами виртуальных машин Azure
description: Узнайте, как использовать расширение "Работоспособность приложения" для наблюдения за работоспособностью приложений, развернутых в масштабируемых наборах виртуальных машин.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 05/06/2020
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: a38a715b45ab4d0810862ef4d016e4187ea507ab
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84783050"
---
# <a name="using-application-health-extension-with-virtual-machine-scale-sets"></a>Использование расширения "Работоспособность приложения" с масштабируемыми наборами виртуальных машин
Мониторинг работоспособности приложения предоставляет важные сведения, которые позволяют определить, когда требуется администрирование и обновление развертывания. Масштабируемые наборы виртуальных машин Azure поддерживают [последовательные обновления](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model), включая [автоматические обновления образа ОС](virtual-machine-scale-sets-automatic-upgrade.md), которые зависят от мониторинга работоспособности отдельных экземпляров для обновления развертывания. Расширение работоспособности также можно использовать для отслеживания работоспособности приложений каждого экземпляра в масштабируемом наборе и выполнения восстановления экземпляров с помощью [автоматического восстановления экземпляров](virtual-machine-scale-sets-automatic-instance-repairs.md).

В этой статье описывается, как использовать расширение "Работоспособность приложения" для наблюдения за работоспособностью приложений, развернутых в масштабируемых наборах виртуальных машин.

## <a name="prerequisites"></a>Предварительные требования
В данной статье предполагается, что вы знакомы с:
-   [Расширения](../virtual-machines/extensions/overview.md) виртуальной машины Azure
-   [Изменением](virtual-machine-scale-sets-upgrade-scale-set.md) масштабируемых наборов виртуальных машин.

## <a name="when-to-use-the-application-health-extension"></a>Когда использовать расширение "Работоспособность приложения"
Расширение "Работоспособность приложения" развертывается внутри экземпляра масштабируемого набора виртуальных машин и сообщает о состоянии работоспособности виртуальной машины изнутри этого экземпляра. Вы можете настроить расширение так, чтобы оно проверяло конечную точку приложения и обновляло состояние приложения на этом экземпляре. Azure проверяет состояние этого экземпляра, чтобы определить, подходит ли экземпляр для операций обновления.

Так как расширение сообщает о состоянии работоспособности из виртуальной машины, это расширение можно применять в ситуациях, когда внешние пробы, например, пробы работоспособности приложений (использующие пользовательские [пробы](../load-balancer/load-balancer-custom-probe-overview.md) Azure Load Balancer), не могут использоваться.

## <a name="extension-schema"></a>Схема расширения

В следующем коде JSON показана схема расширения "Работоспособность приложения". Для расширения требуется как минимум запрос "TCP", "http" или "HTTPS" с соответствующим портом или путем запроса соответственно.

```json
{
  "type": "extensions",
  "name": "HealthExtension",
  "apiVersion": "2018-10-01",
  "location": "<location>",  
  "properties": {
    "publisher": "Microsoft.ManagedServices",
    "type": "< ApplicationHealthLinux or ApplicationHealthWindows>",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "1.0",
    "settings": {
      "protocol": "<protocol>",
      "port": "<port>",
      "requestPath": "</requestPath>"
    }
  }
}  
```

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных
| ---- | ---- | ---- 
| версия_API | `2018-10-01` | Дата |
| publisher | `Microsoft.ManagedServices` | строка |
| type | `ApplicationHealthLinux` (Linux), `ApplicationHealthWindows` (Windows) | строка |
| typeHandlerVersion | `1.0` | INT |

### <a name="settings"></a>Настройки

| Имя | Значение и пример | Тип данных
| ---- | ---- | ----
| protocol | `http`, `https` или `tcp` | строка |
| порт | Необязательно, если параметр Protocol имеет `http` или `https` , является обязательным, если используется протокол `tcp` | INT |
| requestPath | Обязательно, если параметр Protocol имеет `http` или `https` , не разрешен, если используется протокол `tcp` | строка |

## <a name="deploy-the-application-health-extension"></a>Развертывание расширение "Работоспособность приложения"
Существует несколько способов развертывания расширения "Работоспособность приложения" в масштабируемые наборы, которые описаны в приведенных ниже примерах.

### <a name="rest-api"></a>REST API

В следующем примере расширение "Работоспособность приложения" (с именем myHealthExtension) добавляется в атрибут extensionProfile в модели масштабируемого набора на основе Windows.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/myHealthExtension?api-version=2018-10-01`
```

```json
{
  "name": "myHealthExtension",
  "properties": {
    "publisher": " Microsoft.ManagedServices",
    "type": "< ApplicationHealthWindows>",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "1.0",
    "settings": {
      "protocol": "<protocol>",
      "port": "<port>",
      "requestPath": "</requestPath>"
    }
  }
}
```
Используйте `PATCH` для изменения уже развернутого расширения.

### <a name="azure-powershell"></a>Azure PowerShell

Используйте командлет [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension), чтобы добавить расширение "Работоспособность приложения" в определение модели масштабируемого набора.

В следующем примере расширение "Работоспособность приложения" добавляется в `extensionProfile` модели масштабируемого набора на основе Windows. В примере используется новый модуль Azure PowerShell.

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
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
```


### <a name="azure-cli-20"></a>Azure CLI 2.0

Используйте команду [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set), чтобы добавить расширение "Работоспособность приложения" в определение модели масштабируемого набора.

В следующем примере расширение работоспособности приложения добавляется в модель масштабируемого набора для масштабируемого набора на основе Linux.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthLinux \
  --publisher Microsoft.ManagedServices \
  --version 1.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --settings ./extension.json
```
extension.jsсодержимого файла.

```json
{
  "protocol": "<protocol>",
  "port": "<port>",
  "requestPath": "</requestPath>"
}
```


## <a name="troubleshoot"></a>Диагностика
Выходные данные выполнения расширения регистрируются в файле, расположенном в следующих каталогах:

```Windows
C:\WindowsAzure\Logs\Plugins\Microsoft.ManagedServices.ApplicationHealthWindows\<version>\
```

```Linux
/var/lib/waagent/Microsoft.ManagedServices.ApplicationHealthLinux-<extension_version>/status
/var/log/azure/applicationhealth-extension
```

Журналы также периодически записывают состояние работоспособности приложения.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как [развертывать приложение](virtual-machine-scale-sets-deploy-app.md) в масштабируемых наборах виртуальных машин.
