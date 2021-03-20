---
title: Развертывание дисков с помощью шаблонов Azure Resource Manager
description: Сведения об использовании управляемых и неуправляемых дисков в шаблонах Azure Resource Manager для виртуальных машин Azure.
documentationcenter: ''
author: jboeshart
manager: ''
ms.service: virtual-machines
ms.topic: how-to
ms.workload: storage
ms.date: 06/01/2017
ms.author: jaboes
ms.subservice: disks
ms.openlocfilehash: 7c66a8b8483673a9d8fbdc9922b9cc377781bab3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91976674"
---
# <a name="using-disks-in-azure-resource-manager-templates"></a>Использование дисков в шаблонах Azure Resource Manager

В этом документе рассматриваются различия между управляемым и неуправляемыми дисками при использовании шаблонов Azure Resource Manager для подготовки виртуальных машин. Приведенные примеры помогут вам обновить существующие шаблоны, в которых используются неуправляемые диски, заменив их управляемыми дисками. В качестве примера мы используем шаблон [101-vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows). Вы можете просмотреть шаблон, использующий [управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows/azuredeploy.json), и его предыдущую версию, использующую [неуправляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/93b5f72a9857ea9ea43e87d2373bf1b4f724c6aa/101-vm-simple-windows/azuredeploy.json), если вы хотите непосредственно сравнить их.

## <a name="unmanaged-disks-template-formatting"></a>Формат шаблона с неуправляемыми дисками

Сначала рассмотрим, как развертываются неуправляемые диски. При создании неуправляемых дисков необходима учетная запись хранения для хранения файлов VHD-файлов. Можно создать новую учетную запись хранения или использовать существующую. В этой статье показано, как создать учетную запись хранения. Создайте ресурс учетной записи хранения в блоке ресурсов, как показано ниже.

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2018-07-01",
    "name": "[variables('storageAccountName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage",
    "properties": {}
}
```

В объект виртуальной машины добавьте зависимость от учетной записи хранения, чтобы гарантировать, что она будет создана до виртуальной машины. В разделе `storageProfile` укажите полный универсальный код ресурса (URI) расположения VHD, который ссылается на учетную запись хранения и требуется для диска ОС и дисков данных.

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "name": "osdisk",
                "vhd": {
                    "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "name": "datadisk1",
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "vhd": {
                        "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/datadisk1.vhd')]"
                    },
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

## <a name="managed-disks-template-formatting"></a>Формат шаблона с управляемыми дисками

При использовании службы "Управляемые диски Azure" диск становится ресурсом верхнего уровня и для него больше не требуется учетная запись хранения, созданная пользователем. Управляемые диски впервые были представлены в версии API `2016-04-30-preview`. Они доступны во всех последующих версиях API и теперь являются типом диска по умолчанию. В следующих разделах рассматриваются параметры по умолчанию и описывается дальнейшая настройка дисков.

> [!NOTE]
> Рекомендуем использовать более позднюю версию API, чем `2016-04-30-preview`, так как между версиями `2016-04-30-preview` и `2017-03-30` были внесены критически важные изменения.
>
>

### <a name="default-managed-disk-settings"></a>Параметры управляемого диска по умолчанию

Чтобы создать виртуальную машину с управляемыми дисками, вам больше не нужно создавать ресурс учетной записи хранения. В приведенном ниже примере шаблона есть некоторые отличия от предыдущих примеров неуправляемого диска, которые следует отметить:

- `apiVersion`— Это версия, поддерживающая управляемые диски.
- `osDisk` и `dataDisks` больше не ссылаются на КОНКРЕТНЫЙ URI для виртуального жесткого диска.
- При развертывании без указания дополнительных свойств диск будет использовать тип хранилища в зависимости от размера виртуальной машины. Например, если вы используете размер виртуальной машины, поддерживающий хранилище класса Premium (размеры с именем "s", например Standard_D2s_v3), то диски уровня "Премиум" будут настроены по умолчанию. Это можно изменить с помощью параметра SKU диска, чтобы указать тип хранилища.
- Если имя диска не указано, оно принимает формат `<VMName>_OsDisk_1_<randomstring>` для диска операционной системы и `<VMName>_disk<#>_<randomstring>` для каждого диска данных.
  - Если виртуальная машина создается из пользовательского образа, то параметры по умолчанию для типа учетной записи хранения и имени диска извлекаются из свойств диска, определенных в ресурсе пользовательского образа. Их можно переопределить, указав значения для них в шаблоне.
- По умолчанию шифрование дисков Azure отключено.
- По умолчанию кэширование диска доступно для чтения и записи на диске операционной системы и отсутствует для дисков данных.
- В приведенном ниже примере существует зависимость учетной записи хранения, хотя она предназначена только для хранения диагностических данных и не требуется для дискового хранилища.

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="using-a-top-level-managed-disk-resource"></a>Использование ресурса управляемого диска верхнего уровня

В качестве альтернативы указанию конфигурации дисков в объекте виртуальной машины можно создать ресурс диска верхнего уровня и подключить его при создании виртуальной машины. Например, можно создать дисковый ресурс, как показано ниже, и использовать его в качестве диска данных.

```json
{
    "type": "Microsoft.Compute/disks",
    "apiVersion": "2018-06-01",
    "name": "[concat(variables('vmName'),'-datadisk1')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "properties": {
        "creationData": {
            "createOption": "Empty"
        },
        "diskSizeGB": 1023
    }
}
```

Затем в объекте виртуальной машины укажите ссылку на него, чтобы подключить этот объект диска. Указав идентификатор ресурса созданного управляемого диска в свойстве `managedDisk`, можно подключить этот диск при создании виртуальной машины. Для параметра `apiVersion` ресурса виртуальной машины задано значение `2017-03-30`. Кроме того, добавлена зависимость от дискового ресурса, чтобы обеспечить его успешное создание перед созданием виртуальной машины. 

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
        "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "lun": 0,
                    "name": "[concat(variables('vmName'),'-datadisk1')]",
                    "createOption": "attach",
                    "managedDisk": {
                        "id": "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
                    }
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="create-managed-availability-sets-with-vms-using-managed-disks"></a>Создание управляемых групп доступности с виртуальными машинами, использующими управляемые диски

Чтобы создать управляемые группы доступности с виртуальными машинами, использующими управляемые диски, добавьте объект `sku` в ресурс группы доступности и задайте для свойства `name` значение `Aligned`. Это свойство гарантирует, что диски каждой виртуальной машины будут достаточно изолированы друг от друга, чтобы избежать образования единых точек отказа. Также обратите внимание, что для параметра `apiVersion` ресурса группы доступности задано значение `2018-10-01`.

```json
{
    "type": "Microsoft.Compute/availabilitySets",
    "apiVersion": "2018-10-01",
    "location": "[resourceGroup().location]",
    "name": "[variables('avSetName')]",
    "properties": {
        "PlatformUpdateDomainCount": 3,
        "PlatformFaultDomainCount": 2
    },
    "sku": {
        "name": "Aligned"
    }
}
```

### <a name="standard-ssd-disks"></a>Диски SSD категории "Стандартный"

Ниже приведены параметры, которые необходимо указать в шаблоне Resource Manager для создания дисков SSD категории "Стандартный".

* Для *apiVersion* для Microsoft.Compute нужно задать значение `2018-04-01` (или более позднюю версию).
* Укажите для параметра *managedDisk.storageAccountType* значение `StandardSSD_LRS`.

В следующем примере показан раздел *properties.storageProfile.osDisk* для виртуальной машины, которая использует диски SSD категории "Стандартный".

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

Полный пример создания диска SSD категории "Стандартный" с помощью шаблона приведен в разделе [Create a Virtual Machine from a Windows Image with multiple empty Standard SSD Data Disks](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/) (Создание виртуальной машины из образа Windows с помощью дисков данных SSD категории "Стандартный").

### <a name="additional-scenarios-and-customizations"></a>Дополнительные сценарии и настройки

Полные сведения о спецификациях REST API можно найти в [документации по созданию управляемых дисков с помощью REST API](/rest/api/manageddisks/disks/disks-create-or-update). В ней вы найдете дополнительные сценарии, а также значения по умолчанию и допустимые значения, которые можно передавать в API при развертывании с помощью шаблонов. 

## <a name="next-steps"></a>Дальнейшие действия

* Полные шаблоны, использующие управляемые диски, можно получить, перейдя по приведенным ниже ссылкам на репозиторий кратких руководств по Azure.
    * [Виртуальная машина Windows с управляемыми дисками](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
    * [Виртуальная машина Linux с управляемыми дисками](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
* Ознакомьтесь с документом [Обзор управляемых дисков Azure](managed-disks-overview.md), чтобы узнать больше об управляемых дисках.
* Ознакомьтесь с информацией по шаблонам для ресурсов виртуальных машин, приведенной в [справочнике по шаблону Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines).
* Ознакомьтесь с информацией по шаблонам дисковых ресурсов, приведенной в [справочнике по шаблону Microsoft.Compute/disks](/azure/templates/microsoft.compute/disks).
* Сведения об использовании управляемых дисков в масштабируемых наборах виртуальных машин Azure см. в документе [об использовании дисков данных с масштабируемыми наборами](../virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks.md).