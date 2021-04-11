---
title: Виртуальные машины в шаблоне Azure Resource Manager | Microsoft Azure
description: Сведения о том, как ресурс виртуальной машины определяется в шаблоне Azure Resource Manager.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/03/2019
ms.author: cynthn
ms.openlocfilehash: 32b73be3faf6eedb92220725b292a3e69cf7f965
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555998"
---
# <a name="virtual-machines-in-an-azure-resource-manager-template"></a>Виртуальные машины в шаблоне Azure Resource Manager

В этой статье описываются аспекты шаблона Azure Resource Manager, которые относятся к виртуальным машинам. Здесь не описывается полный шаблон для создания виртуальной машины. Для этого требуются определения ресурсов для учетных записей хранения, сетевых интерфейсов, общедоступных IP-адресов и виртуальных сетей. Дополнительные сведения о том, как эти ресурсы могут быть определены вместе, см. в статье [Пошаговое руководство по созданию шаблона Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

[В коллекции есть много шаблонов](https://azure.microsoft.com/documentation/templates/?term=VM), которые включают в себя ресурс виртуальной машины. Здесь описываются не все элементы, которые могут быть включены в шаблон.

 

В этом примере показан типичный раздел ресурсов шаблона для создания указанного числа виртуальных машин.

```json
"resources": [
  { 
    "apiVersion": "2016-04-30-preview", 
    "type": "Microsoft.Compute/virtualMachines", 
    "name": "[concat('myVM', copyindex())]", 
    "location": "[resourceGroup().location]",
    "copy": {
      "name": "virtualMachineLoop", 
      "count": "[parameters('numberOfInstances')]"
    },
    "dependsOn": [
      "[concat('Microsoft.Network/networkInterfaces/myNIC', copyindex())]" 
    ], 
    "properties": { 
      "hardwareProfile": { 
        "vmSize": "Standard_DS1" 
      }, 
      "osProfile": { 
        "computername": "[concat('myVM', copyindex())]", 
        "adminUsername": "[parameters('adminUsername')]", 
        "adminPassword": "[parameters('adminPassword')]" 
      }, 
      "storageProfile": { 
        "imageReference": { 
          "publisher": "MicrosoftWindowsServer", 
          "offer": "WindowsServer", 
          "sku": "2012-R2-Datacenter", 
          "version": "latest" 
        }, 
        "osDisk": { 
          "name": "[concat('myOSDisk', copyindex())]",
          "caching": "ReadWrite", 
          "createOption": "FromImage" 
        },
        "dataDisks": [
          {
            "name": "[concat('myDataDisk', copyindex())]",
            "diskSizeGB": "100",
            "lun": 0,
            "createOption": "Empty"
          }
        ] 
      }, 
      "networkProfile": { 
        "networkInterfaces": [ 
          { 
            "id": "[resourceId('Microsoft.Network/networkInterfaces',
              concat('myNIC', copyindex()))]" 
          } 
        ] 
      },
      "diagnosticsProfile": {
        "bootDiagnostics": {
          "enabled": "true",
          "storageUri": "[concat('https://', variables('storageName'), '.blob.core.windows.net')]"
        }
      } 
    },
    "resources": [ 
      { 
        "name": "Microsoft.Insights.VMDiagnosticsSettings", 
        "type": "extensions", 
        "location": "[resourceGroup().location]", 
        "apiVersion": "2016-03-30", 
        "dependsOn": [ 
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
        ], 
        "properties": { 
          "publisher": "Microsoft.Azure.Diagnostics", 
          "type": "IaaSDiagnostics", 
          "typeHandlerVersion": "1.5", 
          "autoUpgradeMinorVersion": true, 
          "settings": { 
            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
            variables('wadmetricsresourceid'), 
            concat('myVM', copyindex()),
            variables('wadcfgxend')))]", 
            "storageAccount": "[variables('storageName')]" 
          }, 
          "protectedSettings": { 
            "storageAccountName": "[variables('storageName')]", 
            "storageAccountKey": "[listkeys(variables('accountid'), 
              '2015-06-15').key1]", 
            "storageAccountEndPoint": "https://core.windows.net" 
          } 
        } 
      },
      {
        "name": "MyCustomScriptExtension",
        "type": "extensions",
        "apiVersion": "2016-03-30",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
        ],
        "properties": {
          "publisher": "Microsoft.Compute",
          "type": "CustomScriptExtension",
          "typeHandlerVersion": "1.7",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "fileUris": [
              "[concat('https://', variables('storageName'),
                '.blob.core.windows.net/customscripts/start.ps1')]" 
            ],
            "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
          }
        }
      } 
    ]
  } 
]
``` 

> [!NOTE] 
>Этот пример основан на учетной записи хранения, которая была создана ранее. Учетную запись хранения можно создать, развернув ее из шаблона. В этом примере также используется сетевой интерфейс и его зависимые ресурсы, которые должны быть определены в шаблоне. Эти ресурсы не показаны в примере.
>
>

## <a name="api-version"></a>Версия API

При развертывании ресурсов с помощью шаблона необходимо указать версию API, которую следует использовать. В примере ниже показан ресурс виртуальной машины, использующий такой элемент apiVersion:

```json
"apiVersion": "2016-04-30-preview",
```

Версия API, которая указывается в шаблоне, влияет на то, какие свойства можно указать в шаблоне. Как правило, при создании шаблонов следует выбирать последнюю версию API. Для существующих шаблонов можно решить, хотите ли вы продолжать использовать более раннюю версию API или хотите обновить шаблон до последней версии, чтобы воспользоваться новыми возможностями.

Последние доступные версии API для разных типов ресурсов можно узнать следующим образом.

- Для REST API выполните операцию [вывода списка всех поставщиков ресурсов](/rest/api/resources/providers).
- PowerShell: [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider)
- Azure CLI: [az provider show](/cli/azure/provider)


## <a name="parameters-and-variables"></a>Параметры и переменные

[Параметры](../../azure-resource-manager/templates/template-syntax.md) позволяют легко задавать значения для шаблона при его запуске. В примере используются следующие параметры.

```json
"parameters": {
  "adminUsername": { "type": "string" },
  "adminPassword": { "type": "securestring" },
  "numberOfInstances": { "type": "int" }
},
```

При развертывании примера шаблона необходимо ввести значения для имени и пароля учетной записи администратора на каждой виртуальной машине и количество виртуальных машин, которое необходимо создать. Можно указывать значения параметров в отдельном файле, который управляется с помощью шаблона, или ввести значения при появлении запроса.

[Переменные](../../azure-resource-manager/templates/template-syntax.md) позволяют легко настраивать в шаблоне значения, которые в нем многократно используются или могут изменяться со временем. В примере используются следующие переменные.

```json
"variables": { 
  "storageName": "mystore1",
  "accountid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name,
  '/providers/','Microsoft.Storage/storageAccounts/', variables('storageName'))]", 
  "wadlogs": "<WadCfg> 
    <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> 
      <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> 
      <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > 
        <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" />
      </WindowsEventLog>", 
  "wadperfcounters": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\">
      <PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\">
        <annotation displayName=\"Threads\" locale=\"en-us\"/>
      </PerformanceCounterConfiguration>
    </PerformanceCounters>", 
  "wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters'), 
    '<Metrics resourceId=\"')]", 
  "wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name , 
    '/providers/', 'Microsoft.Compute/virtualMachines/')]", 
  "wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/>
    <MetricAggregation scheduledTransferPeriod=\"PT1M\"/>
    </Metrics></DiagnosticMonitorConfiguration>
    </WadCfg>"
}, 
```

При развертывании нового шаблона значения переменных используются для имени и идентификатора учетной записи хранения, созданной ранее. Переменные также используются для предоставления параметров модулю диагностики. В статье [Рекомендации по созданию шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-best-practices.md) содержатся сведения, которые помогут вам решить, как структурировать параметры и переменные в шаблоне.

## <a name="resource-loops"></a>цикла ресурсов

Если для приложения требуется несколько виртуальных машин, в шаблоне можно использовать элемент копирования. Этот необязательный элемент циклически создает число виртуальных машин, заданное с помощью параметра.

```json
"copy": {
  "name": "virtualMachineLoop", 
  "count": "[parameters('numberOfInstances')]"
},
```

Обратите внимание, что в примере при указании некоторых значений для ресурса используется индекс цикла. Например, если введенное число экземпляров равно трем, то именами дисков ОС будут myOSDisk1, myOSDisk2 и myOSDisk3.

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
}
```

> [!NOTE] 
>В этом примере для виртуальных машин используются управляемые диски.
>
>

При создании цикла для одного ресурса в шаблоне может потребоваться использовать цикл при создании других ресурсов или получении к ним доступа. Например, несколько виртуальных машин не могут использовать сетевой интерфейс. Поэтому если шаблон циклически создает три виртуальные машины, он также циклически создает три сетевых интерфейса. При назначении сетевого интерфейса виртуальной машине для его идентификации используется индекс цикла.

```json
"networkInterfaces": [ { 
  "id": "[resourceId('Microsoft.Network/networkInterfaces',
    concat('myNIC', copyindex()))]" 
} ]
```

## <a name="dependencies"></a>Зависимости

Корректная работа большинства ресурсов зависит от других ресурсов. Виртуальные машины должны быть связаны с виртуальной сетью, для чего необходим сетевой интерфейс. Элемент [dependsOn](../../azure-resource-manager/templates/define-resource-dependency.md) используется, чтобы перед созданием виртуальных машин убедиться в том, что сетевой интерфейс готов к использованию.

```json
"dependsOn": [
  "[concat('Microsoft.Network/networkInterfaces/', 'myNIC', copyindex())]" 
],
```

Resource Manager параллельно развертывает все ресурсы, которые не зависят от развертывания других ресурсов. Будьте осторожны при установке зависимостей, так как можно случайно замедлить развертывания, указав ненужные зависимости. С помощью зависимостей можно связать несколько ресурсов. Например, сетевой интерфейс зависит от общедоступного IP-адреса и ресурсов виртуальной сети.

Как узнать, требуется ли зависимость? Взгляните на значения, заданные в шаблоне. Если элемент в ресурсе виртуальной машины указывает на другой ресурс, который развернут в том же шаблоне, зависимость необходима. Например, в примере виртуальный машины определен профиль сети.

```json
"networkProfile": { 
  "networkInterfaces": [ { 
    "id": "[resourceId('Microsoft.Network/networkInterfaces',
      concat('myNIC', copyindex())]" 
  } ] 
},
```

Чтобы задать это свойство, необходим сетевой интерфейс. Поэтому требуется зависимость. Зависимость также необходимо задать, если один ресурс (дочерний) определен внутри другого ресурса (родительского). Например, параметры диагностики и расширения пользовательского скрипта определены как дочерние ресурсы виртуальной машины. Их нельзя создать, пока существует виртуальная машина. Таким образом, оба ресурса помечаются как зависимые от виртуальной машины.

## <a name="profiles"></a>Профили

При определении ресурса виртуальной машины используются несколько элементов профиля. Некоторые являются обязательными, а другие — необязательными. Например, элементы hardwareProfile, osProfile, storageProfile и networkProfile являются обязательными, а diagnosticsProfile — необязательным. С помощью этих профилей задаются следующие параметры:
   
- [size](../sizes.md)
- [имя](/azure/architecture/best-practices/resource-naming) и учетные данные;
- диск и [параметры операционной системы;](cli-ps-findimage.md)
- [сетевой интерфейс;](/previous-versions/azure/virtual-network/virtual-network-deploy-multinic-classic-ps) 
- диагностика загрузки.

## <a name="disks-and-images"></a>Диски и образы
   
В Azure файлы VHD могут представлять [диски или образы](../managed-disks-overview.md). Если операционная система в файле VHD предназначена для конкретной виртуальной машины, она называется диском. Если операционная система в файле VHD обобщена для создания нескольких виртуальных машин, она называется образом.   
    
### <a name="create-new-virtual-machines-and-new-disks-from-a-platform-image"></a>Создание новых виртуальных машин и новых дисков с помощью образа платформы

При создании виртуальной машины необходимо решить, какую операционную систему использовать. Элемент imageReference используется для определения операционной системы новой виртуальной машины. В примере ниже показано определение для операционной системы Windows Server.

```json
"imageReference": { 
  "publisher": "MicrosoftWindowsServer", 
  "offer": "WindowsServer", 
  "sku": "2012-R2-Datacenter", 
  "version": "latest" 
},
```

Если вы хотите создать операционную систему Linux, можно использовать следующее определение.

```json
"imageReference": {
  "publisher": "Canonical",
  "offer": "UbuntuServer",
  "sku": "14.04.2-LTS",
  "version": "latest"
},
```

Параметры конфигурации диска ОС назначаются с помощью элемента osDisk. В примере определяется новый управляемый диск с режимом кэширования **ReadWrite** и указывается, что диск создается из [образа платформы](cli-ps-findimage.md).

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
},
```

### <a name="create-new-virtual-machines-from-existing-managed-disks"></a>Создание виртуальных машин из существующих управляемых дисков

Если вы хотите создать виртуальные машины из существующих дисков, удалите элементы imageReference и osProfile и определите параметры диска.

```json
"osDisk": { 
  "osType": "Windows",
  "managedDisk": { 
    "id": "[resourceId('Microsoft.Compute/disks', [concat('myOSDisk', copyindex())])]" 
  }, 
  "caching": "ReadWrite",
  "createOption": "Attach" 
},
```

### <a name="create-new-virtual-machines-from-a-managed-image"></a>Создание виртуальных машин из управляемого образа

Если вы хотите создать виртуальную машину из управляемого образа, измените элемент imageReference и определите приведенные ниже параметры диска.

```json
"storageProfile": { 
  "imageReference": {
    "id": "[resourceId('Microsoft.Compute/images', 'myImage')]"
  },
  "osDisk": { 
    "name": "[concat('myOSDisk', copyindex())]",
    "osType": "Windows",
    "caching": "ReadWrite", 
    "createOption": "FromImage" 
  }
},
```

### <a name="attach-data-disks"></a>Присоединение дисков данных

При необходимости к виртуальным машинам можно добавить диски данных. [Число дисков](../sizes.md) зависит от размера диска операционной системы, который используется. Если размер виртуальных машин установлен как Standard_DS1_v2, максимальное количество дисков данных, которые к ним можно добавить, равно двум. В примере ниже к каждой виртуальной машине добавляется один управляемый диск данных.

```json
"dataDisks": [
  {
    "name": "[concat('myDataDisk', copyindex())]",
    "diskSizeGB": "100",
    "lun": 0, 
    "caching": "ReadWrite",
    "createOption": "Empty"
  }
],
```

## <a name="extensions"></a>Модули

Хотя функции [расширения](../extensions/features-windows.md) являются отдельным ресурсом, они тесно связаны с виртуальными машинами. Расширения можно добавить как дочерний ресурс виртуальной машины или как отдельный ресурс. В примере показано добавление [расширения системы диагностики](../extensions/diagnostics-template.md) к виртуальным машинам.

```json
{ 
  "name": "Microsoft.Insights.VMDiagnosticsSettings", 
  "type": "extensions", 
  "location": "[resourceGroup().location]", 
  "apiVersion": "2016-03-30", 
  "dependsOn": [ 
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
  ], 
  "properties": { 
    "publisher": "Microsoft.Azure.Diagnostics", 
    "type": "IaaSDiagnostics", 
    "typeHandlerVersion": "1.5", 
    "autoUpgradeMinorVersion": true, 
    "settings": { 
      "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
      variables('wadmetricsresourceid'), 
      concat('myVM', copyindex()),
      variables('wadcfgxend')))]", 
      "storageAccount": "[variables('storageName')]" 
    }, 
    "protectedSettings": { 
      "storageAccountName": "[variables('storageName')]", 
      "storageAccountKey": "[listkeys(variables('accountid'), 
        '2015-06-15').key1]", 
      "storageAccountEndPoint": "https://core.windows.net" 
    } 
  } 
},
```

Для предоставления значений этот ресурс расширения использует переменную storageName и переменные диагностики. Если требуется изменить данные, собранные этим модулем, можно добавить дополнительные счетчики производительности для переменной wadperfcounters. Также можно поместить данные диагностики в учетную запись хранения, отличную от записи, в которой хранятся диски виртуальной машины.

Существует множество расширений, которые можно установить на виртуальной машине, но наиболее полезным является [расширение пользовательских скриптов](../extensions/custom-script-windows.md). В примере скрипт PowerShell с именем start.ps1 выполняется на каждой виртуальной машине при первом запуске.

```json
{
  "name": "MyCustomScriptExtension",
  "type": "extensions",
  "apiVersion": "2016-03-30",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.7",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "[concat('https://', variables('storageName'),
          '.blob.core.windows.net/customscripts/start.ps1')]" 
      ],
      "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
    }
  }
}
```

Скрипт start.ps1 может выполнять многие задачи конфигурации. Например, диски с данными, которые добавляются к виртуальным машинам в примере, не инициализированы. Для их инициализации можно использовать настраиваемый скрипт. При наличии нескольких задач запуска можно использовать файл start.ps1 для вызова других скриптов PowerShell из хранилища Azure. В примере используется PowerShell, но вы можете использовать любой метод написания скриптов, доступный в используемой операционной системе.

Состояние установленных расширений можно увидеть в настройках расширений на портале.

![Получение состояния расширения](./media/template-description/virtual-machines-show-extensions.png)

Сведения о расширении также можно получить с помощью команды PowerShell **Get-AzVMExtension**, команды Azure CLI **vm extension get** или команды REST API **Get extension information**.

## <a name="deployments"></a>Развернутые приложения

При развертывании шаблона Azure отслеживает ресурсы, развернутые в рамках группы, и автоматически присваивает имя этой развернутой группе. Имя этого развертывания совпадает с именем шаблона.

Если вы хотите узнать состояние ресурсов в развертывании, см. колонку "Группа ресурсов" на портале Azure.

![Получение сведений о развертывании](./media/template-description/virtual-machines-deployment-info.png)
    
Один и тот же шаблон можно использовать для создания или для обновления существующих ресурсов. При использовании команд для развертывания шаблонов вы сможете выбрать, какой [режим](../../azure-resource-manager/templates/deploy-powershell.md) хотите использовать. Можно задать **полный** или **добавочный** режим. По умолчанию выполняются добавочные обновления. Будьте внимательны при использовании режима **Полный**, так как можно случайно удалить ресурсы. Если выбран режим **Завершить**, Resource Manager удаляет все ресурсы в группе ресурсов, которых нет в шаблоне.

## <a name="next-steps"></a>Next Steps

- Сведения о создании собственного шаблона см. в статье [Создание шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
- Сведения о развертывании созданного шаблона см. в статье [Создание виртуальной машины Windows с использованием шаблона Resource Manager](ps-template.md).
- Сведения об управлении созданными виртуальными машинами см. в статье [Создание виртуальных машин Windows и управление ими с помощью модуля Azure PowerShell](tutorial-manage-vm.md).
- Сведения о синтаксисе JSON и свойствах типов ресурсов в шаблонах см. в статье [Define resources in Azure Resource Manager templates](/azure/templates/) (Определение ресурсов в шаблонах Azure Resource Manager).