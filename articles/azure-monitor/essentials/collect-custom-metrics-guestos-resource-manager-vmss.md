---
title: Получение метрик масштабируемого набора Windows в Azure Monitor с помощью шаблона
description: Отправка метрик гостевой ОС в хранилище метрик Azure Monitor с помощью шаблона Resource Manager для масштабируемого набора виртуальных машин Windows
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.openlocfilehash: c5f536295bce07e70606f1ff286b08f8465d59f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102048959"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-by-using-an-azure-resource-manager-template-for-a-windows-virtual-machine-scale-set"></a>Отправка метрик гостевой ОС в хранилище метрик Azure Monitor с помощью шаблона Azure Resource Manager для масштабируемого набора виртуальных машин Windows

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[Расширение Диагностики Azure для Windows](../agents/diagnostics-extension-overview.md) для Azure Monitor позволяет собирать метрики и журналы из гостевой операционной системы (гостевой ОС), работающей в составе виртуальной машины, облачной службы или кластера Azure Service Fabric. Расширение может отправлять телеметрию во множество различных расположений, перечисленных в предыдущей статье.  

В этой статье описывается процесс отправки метрик производительности гостевой ОС для масштабируемого набора виртуальных машин под управлением Windows в хранилище данных Azure Monitor. Начиная с Диагностики Azure для Windows версии 1.11 метрики можно записывать напрямую в хранилище метрик Azure Monitor, где уже собраны стандартные метрики платформы. Таким образом вы можете получить доступ к тем же действиям, которые доступны для метрик платформы. К этим действиям относятся оповещения практически в реальном времени, построение диаграмм, маршрутизация, доступ из REST API и многое другое. В прошлом расширение Диагностики Azure для Windows обеспечивало запись в службу хранилища Azure, но не в хранилище данных Azure Monitor.  

Если вы не знакомы с шаблонами Resource Manager, изучите сведения о [развертывании шаблонов](../../azure-resource-manager/management/overview.md), их структуре и синтаксисе.  

## <a name="prerequisites"></a>Предварительные требования

- Подписку необходимо зарегистрировать в [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Требуется установить [Azure PowerShell](/powershell/azure) или использовать [Azure Cloud Shell](../../cloud-shell/overview.md). 

- Ресурс виртуальной машины должен находиться в [регионе, поддерживающем пользовательские метрики](./metrics-custom-overview.md#supported-regions).

## <a name="set-up-azure-monitor-as-a-data-sink"></a>Настройка Azure Monitor в качестве приемника данных 
Расширение система диагностики Azure использует функцию, называемую **приемниками данных** , для маршрутизации метрик и журналов в разные расположения. Ниже показано, как с помощью шаблона Resource Manager и PowerShell развернуть виртуальную машину, используя новый приемник данных Azure Monitor. 

## <a name="author-a-resource-manager-template"></a>Создание шаблона Resource Manager 
В этом примере можно использовать общедоступный [образец шаблона](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-autoscale):  

- **Azuredeploy.jsв** — это предварительно настроенный шаблон диспетчер ресурсов для развертывания масштабируемого набора виртуальных машин.

- **Azuredeploy.Parameters.json** — это файл параметров, в котором хранятся сведения, такие как имя пользователя и пароль, которые вы хотите задать для виртуальной машины. Во время развертывания шаблона Resource Manager используются параметры, заданные в этом файле. 

Скачайте и сохраните оба файла в локальном расположении. 

###  <a name="modify-azuredeployparametersjson"></a>Изменение файла azuredeploy.parameters.json
Откройте **azuredeploy.parameters.jsв** файле:  
 
- Укажите **vmSKU** для развертывания. Рекомендуемое значение — Standard_D2_v3. 
- Укажите **windowsOSVersion** для масштабируемого набора виртуальных машин. Рекомендуемое значение — 2016-Datacenter. 
- Задайте имя для ресурса масштабируемого набора виртуальных машин, который будет развернут, с помощью свойства **vmssName**. Например, **VMSS-WAD-TEST**.    
- Укажите число виртуальных машин, которые следует запускать в масштабируемом наборе виртуальных машин, с помощью свойства **instanceCount**.
- Введите значения **adminUsername** и **adminPassword** для масштабируемого набора виртуальных машин. Эти параметры используются для удаленного доступа к виртуальным машинам в масштабируемом наборе. **Не** используйте параметры, указанные в этом шаблоне, во избежание перехвата виртуальной машины. Боты сканируют Интернет на наличие имен пользователей и паролей в общедоступных репозиториях GitHub. Вероятно, они будут тестировать виртуальные машины с этими значениями по умолчанию. 


###  <a name="modify-azuredeployjson"></a>Изменение файла azuredeploy.json
Откройте **azuredeploy.jsв** файле. 

Добавьте переменную, в которой будут содержаться данные учетной записи хранения в шаблоне Resource Manager. Все журналы или счетчики производительности, указанные в файле конфигурации диагностики, записываются в хранилище метрик Azure Monitor и в указанную учетную запись хранения. 

```json
"variables": { 
//add this line       
"storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]", 
```
 
Найдите определение масштабируемого набора виртуальных машин в разделе ресурсов и добавьте раздел **Identity** в конфигурацию. Это позволит назначить набору системное удостоверение в Azure. Это действие также гарантирует, что виртуальные машины в масштабируемом наборе могут отправлять гостевые метрики о себе в Azure Monitor.  

```json
    { 
      "type": "Microsoft.Compute/virtualMachineScaleSets", 
      "name": "[variables('namingInfix')]", 
      "location": "[resourceGroup().location]", 
      "apiVersion": "2017-03-30", 
      //add these lines below
      "identity": { 
           "type": "systemAssigned" 
       }, 
       //end of lines to add
```

В ресурсе масштабируемого набора виртуальных машин найдите раздел **virtualMachineProfile**. Добавьте новый профиль с именем **extensionsProfile** для управления расширениями.  


В профиле **extensionProfile** добавьте в шаблон новое расширение, как показано в разделе **VMSS-WAD-extension**.  Этот раздел представляет собой расширение управляемых удостоверений для ресурсов Azure, которое обеспечивает прием выдаваемых метрик в Azure Monitor. В поле **name** может содержаться любое имя. 

Приведенный ниже код из расширения MSI также позволяет добавить расширение диагностики и конфигурации в качестве ресурса расширения в масштабируемый набор виртуальных машин. При необходимости счетчики производительности можно добавлять или удалять. 

```json
          "extensionProfile": { 
            "extensions": [ 
            // BEGINNING of added code  
            // Managed identities for Azure resources   
                { 
                 "name": "VMSS-WAD-extension", 
                 "properties": { 
                       "publisher": "Microsoft.ManagedIdentity", 
                       "type": "ManagedIdentityExtensionForWindows", 
                       "typeHandlerVersion": "1.0", 
                       "autoUpgradeMinorVersion": true, 
                       "settings": { 
                             "port": 50342 
                           }, 
                       "protectedSettings": {} 
                     } 
                                
            }, 
            // add diagnostic extension. (Remove this comment after pasting.)
            { 
              "name": "[concat('VMDiagnosticsVmExt','_vmNodeType0Name')]", 
              "properties": { 
                   "type": "IaaSDiagnostics", 
                   "autoUpgradeMinorVersion": true, 
                   "protectedSettings": { 
                        "storageAccountName": "[variables('storageAccountName')]", 
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')),'2015-05-01-preview').key1]", 
                        "storageAccountEndPoint": "https://core.windows.net/" 
                   }, 
                   "publisher": "Microsoft.Azure.Diagnostics", 
                   "settings": { 
                        "WadCfg": { 
                              "DiagnosticMonitorConfiguration": { 
                                   "overallQuotaInMB": "50000", 
                                   "PerformanceCounters": { 
                                       "scheduledTransferPeriod": "PT1M", 
                                       "sinks": "AzMonSink", 
                                       "PerformanceCounterConfiguration": [ 
                                          { 
              "counterSpecifier": "\\Memory\\% Committed Bytes In Use", 
                                              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Available Bytes", 
              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Committed Bytes", 
              "sampleRate": "PT15S" 
           } 
                                       ] 
                                 }, 
                                 "EtwProviders": { 
                                       "EtwEventSourceProviderConfiguration": [ 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Actors", 
                                              "scheduledTransferKeywordFilter": "1", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                              "eventDestination": "ServiceFabricReliableActorEventTable" 
                                           } 
                                           }, 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Services", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricReliableServiceEventTable" 
                                              } 
                                           } 
                                     ], 
                                     "EtwManifestProviderConfiguration": [ 
                                           { 
                                              "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8", 
                                              "scheduledTransferLogLevelFilter": "Information", 
                                              "scheduledTransferKeywordFilter": "4611686018427387904", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricSystemEventTable" 
                                              } 
                                          } 
                                     ] 
                               } 
                               }, 
                               "SinksConfig": { 
                                     "Sink": [ 
                                          { 
                                              "name": "AzMonSink", 
                                              "AzureMonitor": {} 
                                          } 
                                      ] 
                               } 
                         }, 
                         "StorageAccount": "[variables('storageAccountName')]" 
                         }, 
                        "typeHandlerVersion": "1.11" 
                  } 
           } 
            ] 
          }
          }
      }
    },
    //end of added code plus a few brackets. Be sure that the number and type of brackets match properly when done. 
    {
      "type": "Microsoft.Insights/autoscaleSettings",
...
```


Добавьте **dependsOn** для учетной записи хранения, чтобы убедиться, что она создана в правильном порядке: 

```json
"dependsOn": [ 
"[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]", 
"[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]" 
//add this line below
"[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]" 
```

Создайте учетную запись хранения, если она еще не создана в шаблоне. 

```json
"resources": [
// add this code    
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2015-05-01-preview",
    "location": "[resourceGroup().location]",
    "properties": {
    "accountType": "Standard_LRS"
    }
},
// end added code
{ 
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
```

Сохраните и закройте оба файла. 

## <a name="deploy-the-resource-manager-template"></a>Развертывание шаблона Resource Manager 

> [!NOTE]  
> Необходимо запустить расширение система диагностики Azure версии 1,5 или более поздней **и** задать для свойства **autoUpgradeMinorVersion:** в шаблоне диспетчер ресурсов значение **true** . Затем Azure загрузит нужное расширение при запуске виртуальной машины. Если этих параметров нет в шаблоне, внесите их и повторно разверните шаблон. 


Чтобы развернуть шаблон Resource Manager, используйте Azure PowerShell.  

1. Запустите PowerShell. 
1. Войдите в Azure, используя команду `Login-AzAccount`.
1. Получите список подписок с помощью командлета `Get-AzSubscription`.
1. Задайте подписку для создания или обновите виртуальную машину. 

   ```powershell
   Select-AzSubscription -SubscriptionName "<Name of the subscription>" 
   ```
1. Создайте группу ресурсов для развертываемой виртуальной машины. Выполните следующую команду. 

   ```powershell
    New-AzResourceGroup -Name "VMSSWADtestGrp" -Location "<Azure Region>" 
   ```

   > [!NOTE]  
   > Не забывайте использовать регион Azure, включенный для пользовательских метрик. Не забывайте использовать [регион Azure, включенный для пользовательских метрик](./metrics-custom-overview.md#supported-regions).
 
1. Выполните следующие команды, чтобы развернуть виртуальную машину.  

   > [!NOTE]  
   > Если нужно обновить существующий масштабируемый набор, добавьте **-Mode Incremental** в конце команды. 
 
   ```powershell
   New-AzResourceGroupDeployment -Name "VMSSWADTest" -ResourceGroupName "VMSSWADtestGrp" -TemplateFile "<File path of your azuredeploy.JSON file>" -TemplateParameterFile "<File path of your azuredeploy.parameters.JSON file>"  
   ```

1. После успешного развертывания сведения о масштабируемом наборе виртуальных машин отобразятся на портале Azure. Он будет выдавать метрики в Azure Monitor. 

   > [!NOTE]  
   > Вы можете столкнуться с ошибками вокруг выбранного **вмскусизе**. В этом случае вернитесь к файлу **azuredeploy.json** и обновите значение по умолчанию для параметра **vmSkuSize**. Мы рекомендуем указать **Standard_DS1_v2**. 


## <a name="chart-your-metrics"></a>Создание диаграммы метрик 

1. Войдите на портал Azure. 

1. В меню слева выберите **Монитор**. 

1. На странице **мониторинг** выберите **метрики**. 

   ![Монитор — страница метрик](media/collect-custom-metrics-guestos-resource-manager-vmss/metrics.png) 

1. Измените период агрегирования на **Последние 30 минут**.  

1. В раскрывающемся меню ресурсов выберите масштабируемый набор виртуальных машин, который вы только что создали.  

1. В раскрывающемся меню пространства имен выберите **Azure. ВМ. Windows. Guest**. 

1. В раскрывающемся меню метрик выберите **Memory\%Committed Bytes in Use** (Процент используемой выделенной памяти).  

Затем можно использовать измерения этой метрики, чтобы построить диаграмму для определенной виртуальной машины или вывести график по каждой виртуальной машине в масштабируемом наборе. 



## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настраиваемых метриках см. в [этой статье](./metrics-custom-overview.md).