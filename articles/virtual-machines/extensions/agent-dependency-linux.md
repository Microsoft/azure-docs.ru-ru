---
title: Расширение виртуальной машины Azure Monitor Dependency для Linux
description: Разверните агент Azure Monitor Dependency Agent на виртуальной машине Linux с помощью расширения виртуальной машины.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: mgoedtel
ms.author: magoedte
ms.collection: linux
ms.date: 03/29/2019
ms.openlocfilehash: d28f0a34f47942bba8776a0acd0bfe3aaf25df12
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102566314"
---
# <a name="azure-monitor-dependency-virtual-machine-extension-for-linux"></a>Расширение виртуальной машины Azure Monitor Dependency для Linux

Функция схемы в Azure Monitor для виртуальных машин получает данные от Microsoft Dependency Agent. Расширение виртуальной машины Azure VM Dependency Agent для Linux предоставляет и поддерживает корпорация Майкрософт. Это расширение устанавливает Dependency Agent на виртуальных машинах Azure. В этом документе подробно описаны поддерживаемые платформы, конфигурации и параметры развертывания для расширения виртуальной машины Azure VM Dependency Agent для Linux.

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Расширение Azure VM Dependency Agent для Linux можно запускать для поддерживаемых операционных систем, перечисленных в разделе [Поддерживаемые операционные системы](../../azure-monitor/vm/vminsights-enable-overview.md#supported-operating-systems) статьи о развертывании Azure Monitor для виртуальных машин.

## <a name="extension-schema"></a>Схема расширения

В приведенном ниже коде в формате JSON показана схема для расширения Azure VM Dependency Agent на виртуальной машине Linux в Azure. 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Linux Azure VM."
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
        "type": "DependencyAgentLinux",
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
| type | DependencyAgentLinux |
| typeHandlerVersion | 9,5 |

## <a name="template-deployment"></a>Развертывание шаблона

Вы можете развернуть расширения виртуальной машины Azure с помощью шаблонов Azure Resource Manager. Чтобы запустить расширение Azure VM Dependency Agent во время развертывания шаблона Azure Resource Manager, в нем можно использовать схему JSON, описанную в предыдущем разделе.

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
        "type": "DependencyAgentLinux",
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
        "type": "DependencyAgentLinux",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
    }
}
```

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

Развернуть расширение виртуальной машины Dependency Agent на существующей виртуальной машине можно с помощью Azure CLI.  

```azurecli

az vm extension set \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также с помощью Azure CLI. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните в Azure CLI следующую команду:

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Выходные данные выполнения расширения регистрируются в следующем файле:

```
/opt/microsoft/dependency-agent/log/install.log 
```

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, обратитесь к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Также можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**. Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Azure](https://azure.microsoft.com/support/faq/).
