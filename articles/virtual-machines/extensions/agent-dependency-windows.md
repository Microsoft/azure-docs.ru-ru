---
title: Расширение виртуальной машины зависимостей Azure Monitor для Windows
description: Развертывание агента зависимостей Azure Monitor на виртуальной машине Windows с помощью расширения виртуальной машины.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: mgoedtel
ms.author: magoedte
ms.collection: windows
ms.date: 03/29/2019
ms.openlocfilehash: 429cc01f466c55283985729c3395bb2137e38fa6
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102566312"
---
# <a name="azure-monitor-dependency-virtual-machine-extension-for-windows"></a>Расширение виртуальной машины зависимостей Azure Monitor для Windows

Функция схемы в Azure Monitor для виртуальных машин получает данные от Microsoft Dependency Agent. Расширение виртуальной машины агента зависимостей Azure для Windows опубликовано и поддерживается корпорацией Майкрософт. Это расширение устанавливает Dependency Agent на виртуальных машинах Azure. В этом документе подробно описаны поддерживаемые платформы, конфигурации и варианты развертывания для расширения виртуальной машины агента зависимостей виртуальных машин Azure для Windows.

## <a name="operating-system"></a>Операционная система

Расширение агента зависимостей виртуальной машины Azure для Windows можно выполнить для поддерживаемых операционных систем, перечисленных в разделе " [Поддерживаемые операционные системы](../../azure-monitor/vm/vminsights-enable-overview.md#supported-operating-systems) " статьи о развертывании Azure Monitor для виртуальных машин.

## <a name="extension-schema"></a>Схема расширения

В следующем JSON показана схема для расширения агента зависимостей виртуальной машины Azure на виртуальной машине Windows Azure.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Azure VM. Supported Windows Server versions:  2008 R2 and above (x64)."
      }
    }
  },
  "variables": {
    "vmExtensionsApiVersion": "2017-03-30"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "[variables('vmExtensionsApiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
      }
    }
  ],
    "outputs": {
    }
}
```

### <a name="property-values"></a>Значения свойств

| Имя | Значение или пример |
| ---- | ---- |
| версия_API | 2015-01-01 |
| publisher | Microsoft.Azure.Monitoring.DependencyAgent |
| type | депенденциажентвиндовс |
| typeHandlerVersion | 9,5 |

## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальных машин Azure можно развернуть с помощью шаблонов Azure Resource Manager. Чтобы запустить расширение Azure VM Dependency Agent во время развертывания шаблона Azure Resource Manager, в нем можно использовать схему JSON, описанную в предыдущем разделе.

Код в формате JSON для расширения виртуальной машины можно вложить в ресурс виртуальной машины. Вы также можете поместить его на корневом или верхнем уровне шаблона JSON Resource Manager. Размещение JSON влияет на значения имени и типа ресурса. Дополнительные сведения см. в разделе [Указание имени и типа дочернего ресурса в шаблоне Resource Manager](../../azure-resource-manager/templates/child-resource-name-type.md).

В примере ниже предполагается, что расширение Dependency Agent вложено в ресурс виртуальной машины. При вложении ресурса расширения JSON помещается в объект `"resources": []` виртуальной машины.


```json
{
    "type": "extensions",
    "name": "DAExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
    }
}
```

При размещении JSON расширения в корне шаблона имя ресурса содержит ссылку на родительскую виртуальную машину. Тип отражает вложенную конфигурацию.

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "<parentVmResource>/DAExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
    }
}
```

## <a name="powershell-deployment"></a>Развертывание с помощью PowerShell

С помощью команды можно `Set-AzVMExtension` развернуть расширение виртуальной машины агента зависимостей на существующей виртуальной машине. Перед выполнением команды необходимо сохранить общедоступные и частные конфигурации в хэш-таблице PowerShell.

```powershell

Set-AzVMExtension -ExtensionName "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ExtensionType "DependencyAgentWindows" `
    -TypeHandlerVersion 9.5 `
    -Location WestUS 
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертываний расширений можно получить из портал Azure и с помощью модуля Azure PowerShell. Чтобы просмотреть состояние развертывания расширений для данной виртуальной машины, выполните следующую команду с помощью модуля Azure PowerShell:

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

Выходные данные выполнения расширения регистрируются в файле, расположенном в следующем каталоге:

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Monitoring.DependencyAgent\
```

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Также можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**. Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Azure](https://azure.microsoft.com/support/faq/).
