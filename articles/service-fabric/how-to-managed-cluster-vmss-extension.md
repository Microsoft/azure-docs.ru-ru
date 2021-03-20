---
title: Добавление расширения масштабируемого набора виртуальных машин в Service Fabric тип узла управляемого кластера (Предварительная версия)
description: Вот как можно добавить расширение масштабируемого набора виртуальных машин Service Fabric тип узла управляемого кластера.
ms.topic: article
ms.date: 09/28/2020
ms.openlocfilehash: a47908b511f79c18482e9d21e623f1cb4dc70ed1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101737772"
---
# <a name="add-a-virtual-machine-scale-set-extension-to-a-service-fabric-managed-cluster-node-type-preview"></a>Добавление расширения масштабируемого набора виртуальных машин в Service Fabric тип узла управляемого кластера (Предварительная версия)

Каждый тип узла в управляемом кластере Service Fabric поддерживается масштабируемым набором виртуальных машин. Это позволяет добавлять [расширения масштабируемых наборов виртуальных машин](../virtual-machines/extensions/overview.md) в типы управляемых узлов кластера Service Fabric.

Расширение масштабируемого набора виртуальных машин можно добавить в тип узла с помощью команды PowerShell [Add-азсервицефабрикманажеднодетипевмекстенсион](/powershell/module/az.servicefabric/add-azservicefabricmanagednodetypevmextension) .

Кроме того, можно создать расширение масштабируемого набора виртуальных машин на Service Fabric тип узла управляемого кластера в шаблоне Azure Resource Manager, например:

```json
{
    "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
    "apiVersion": "[variables('sfApiVersion')]",
    "name": "[concat(parameters('clusterName'), '/', parameters('nodeTypeName'))]",
    "dependsOn": [
        "[concat('Microsoft.ServiceFabric/managedclusters/', parameters('clusterName'))]"
    ],
    "location": "[resourceGroup().location]",
    "properties": {
        "isPrimary": true,
        "vmInstanceCount": 3,
        "dataDiskSizeGB": 100,
        "vmSize": "Standard_D2",
        "vmImagePublisher": "MicrosoftWindowsServer",
        "vmImageOffer": "WindowsServer",
        "vmImageSku": "2019-Datacenter",
        "vmImageVersion": "latest",
        "vmExtensions": [{
            "name": "ExtensionA",
            "properties": {
                "publisher": "ExtensionA.Publisher",
                "type": "KeyVaultForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                }
            }
        }]
    }
}
```

Дополнительные сведения о настройке типов управляемых узлов кластера Service Fabric см. в разделе [тип узла управляемого кластера](/azure/templates/microsoft.servicefabric/2020-01-01-preview/managedclusters/nodetypes).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляемых кластерах Service Fabric см. в статье:

> [!div class="nextstepaction"]
> [Распространенные вопросы об управляемых кластерах Service Fabric](./faq-managed-cluster.md)
