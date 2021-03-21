---
title: Использование Azure PowerShell для включения диагностики на виртуальной машине Windows
description: Описание процедуры включения системы диагностики Azure на виртуальной машине под управлением Windows с помощью PowerShell.
ms.topic: article
author: amjads1
ms.author: amjads
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.date: 12/15/2015
ms.openlocfilehash: e242a45e2c0f8247a1c23bf4d43a6e8b0a033d9b
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560101"
---
# <a name="use-powershell-to-enable-azure-diagnostics-in-a-virtual-machine-running-windows"></a>Включение системы диагностики Azure на виртуальной машине под управлением Windows с помощью PowerShell

Система диагностики Azure позволяет выполнять сбор диагностических данных в развернутом приложении. Для сбора диагностических данных, таких как журналы приложений или счетчики производительности, на виртуальной машине Azure под управлением Windows можно использовать расширение диагностики. 

 

## <a name="enable-the-diagnostics-extension-if-you-use-the-resource-manager-deployment-model"></a>Включения расширения диагностики при использовании модели развертывания диспетчера ресурсов
Вы можете включить расширение диагностики при создании виртуальной машины Windows, используя модель развертывания диспетчера ресурсов Azure. Для этого в шаблон диспетчера ресурсов нужно добавить конфигурацию расширения. Дополнительные сведения см. в статье [Создание виртуальной машины Windows с мониторингом и диагностикой с использованием шаблона диспетчера ресурсов Azure](diagnostics-template.md).

Чтобы включить расширение диагностики на уже существующей виртуальной машине, созданной с помощью модели развертывания Resource Manager, можете использовать командлет PowerShell [Set-AzVMDiagnosticsExtension](/powershell/module/az.compute/set-azvmdiagnosticsextension), как показано ниже.

```azurepowershell
$vm_resourcegroup = "myvmresourcegroup"
$vm_name = "myvm"
$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"

Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path
```


*$diagnosticsconfig_path* — это путь к файлу с конфигурацией диагностики в формате XML, как показано в [примере](#sample-diagnostics-configuration) ниже.  

Если файл конфигурации диагностики содержит элемент **StorageAccount** с именем учетной записи хранения, сценарий *Set-AzVMDiagnosticsExtension* автоматически настраивает для расширения диагностики отправку диагностических данных в эту учетную запись. Для этого учетная запись хранения должна входить в ту же подписку, что и виртуальная машина.

Если в конфигурации диагностики нет элемента **StorageAccount** , в командлет необходимо передать параметр *StorageAccountName* . Если указан параметр *StorageAccountName* , то командлет всегда будет использовать учетную запись хранения, указанную в параметре, а не ту, которая указана в файле конфигурации диагностики.

Если учетная запись хранения диагностики и виртуальная машина относятся к разным подпискам, то в командлет необходимо явно передать параметры *StorageAccountName* и *StorageAccountKey*. Параметр *StorageAccountKey* не требуется, если учетная запись хранения диагностических данных находится в той же подписке, так как командлет может автоматически запрашивать и задавать значение ключа при включении расширения диагностики. Однако если учетная запись хранения диагностики находится в другой подписке, командлет не сможет получить ключ автоматически, и необходимо явно указать ключ с помощью параметра *StorageAccountKey* .  

```azurepowershell
Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
```

После включения расширения диагностики на виртуальной машине получить текущие параметры можно с помощью командлета [Get-AzVmDiagnosticsExtension](/powershell/module/az.compute/get-azvmdiagnosticsextension).

```azurepowershell
Get-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name
```

Командлет возвращает значение *PublicSettings*, содержащее конфигурацию диагностики. Поддерживаются два типа конфигурации: WadCfg и xmlCfg. WadCfg — это конфигурации JSON, а xmlCfg — это конфигурация XML в кодировке Base64. Чтобы прочитать XML-файл, его необходимо декодировать.

```azurepowershell
$publicsettings = (Get-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name).PublicSettings
$encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
$xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
Write-Host $xmlconfig
```

Для удаления расширения диагностики с виртуальной машины вы можете использовать командлет [Remove-AzVmDiagnosticsExtension](/powershell/module/az.compute/remove-azvmdiagnosticsextension).  

## <a name="enable-the-diagnostics-extension-if-you-use-the-classic-deployment-model"></a>Включения расширения диагностики при использовании классической модели развертывания

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Включить расширение диагностики на виртуальной машине, созданной на основе классической модели развертывания, можно с помощью командлета [Set-AzureVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension) . В следующем примере показано, как создать новую виртуальную машину, используя классическую модель развертывания при включенном расширении диагностики.

```azurepowershell
$VM = New-AzureVMConfig -Name $VM -InstanceSize Small -ImageName $VMImage
$VM = Add-AzureProvisioningConfig -VM $VM -AdminUsername $Username -Password $Password -Windows
$VM = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
New-AzVM -Location $Location -ServiceName $Service_Name -VM $VM
```

Чтобы включить расширение диагностики на существующей виртуальной машине, созданной с помощью классической модели развертывания, сначала используйте командлет [Get-AzureVM](/powershell/module/servicemanagement/azure.service/get-azurevm) , который позволит получить конфигурацию виртуальной машины. Затем обновите конфигурацию виртуальной машины, чтобы активировать расширение диагностики, с помощью командлета [Set-AzureVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension) . И, наконец, примените обновленную конфигурацию к виртуальной машине с помощью командлета [Update-AzureVM](/powershell/module/servicemanagement/azure.service/update-azurevm).

```azurepowershell
$VM = Get-AzureVM -ServiceName $Service_Name -Name $VM_Name
$VM_Update = Set-AzureVMDiagnosticsExtension  -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
Update-AzureVM -ServiceName $Service_Name -Name $VM_Name -VM $VM_Update.VM
```

## <a name="sample-diagnostics-configuration"></a>Пример конфигурации диагностики
Представленный ниже XML-код можно использовать для открытой конфигурации диагностики с применением описанных выше сценариев. Конфигурация в данном примере передает в учетную запись хранения диагностических данных различные счетчики производительности вместе с ошибками из журналов приложений, событий безопасности и системных каналов в Windows, а также из журналов инфраструктуры диагностики.

Конфигурацию необходимо обновить, чтобы включить в нее следующее:

* Атрибут *resourceID* элемента **Metrics** необходимо обновить, указав идентификатор ресурса для виртуальной машины.
  
  * Идентификатор ресурса может иметь следующий формат: "/subscriptions/{*идентификатор подписки, в которую входит виртуальная машина*}/resourceGroups/{*имя группы ресурсов виртуальной машины*}/providers/Microsoft.Compute/virtualMachines/{*имя виртуальной машины*}".
  * Например, если подписка, в которую входит виртуальная машина, имеет идентификатор **11111111-1111-1111-1111-111111111111**, группа ресурсов называется **MyResourceGroup**, а виртуальная машина — **MyWindowsVM**, то атрибут *resourceID* будет выглядеть следующим образом:
    
      ```xml
      <Metrics resourceId="/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyWindowsVM" >
      ```
  * Дополнительные сведения о генерировании метрик на основе счетчиков производительности и конфигурации метрик см. в статье, посвященной [таблице метрик диагностики Azure в хранилище](diagnostics-template.md#wadmetrics-tables-in-storage).
* Элемент **StorageAccount** необходимо обновить, указав диагностическое имя учетной записи хранения.
  
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
          <DiagnosticMonitorConfiguration overallQuotaInMB="4096">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error"/>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU utilization" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Privileged Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU privileged time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% User Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="CPU user time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Processor Information(_Total)\Processor Frequency" sampleRate="PT15S" unit="Count">
            <annotation displayName="CPU frequency" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\System\Processes" sampleRate="PT15S" unit="Count">
            <annotation displayName="Processes" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Thread Count" sampleRate="PT15S" unit="Count">
            <annotation displayName="Threads" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Handle Count" sampleRate="PT15S" unit="Count">
            <annotation displayName="Handles" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes In Use" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Memory usage" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory available" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory committed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Commit Limit" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory commit limit" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Paged Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory paged pool" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Nonpaged Bytes" sampleRate="PT15S" unit="Bytes">
            <annotation displayName="Memory non-paged pool" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Read Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active read time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Write Time" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk active write time" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Transfers/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Reads/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk read operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Writes/sec" sampleRate="PT15S" unit="CountPerSecond">
            <annotation displayName="Disk write operations" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Read Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk read speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Write Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
            <annotation displayName="Disk write speed" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Read Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average read queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Write Queue Length" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk average write queue length" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\% Free Space" sampleRate="PT15S" unit="Percent">
            <annotation displayName="Disk free space (percentage)" locale="en-us"/>
          </PerformanceCounterConfiguration>
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\Free Megabytes" sampleRate="PT15S" unit="Count">
            <annotation displayName="Disk free space (MB)" locale="en-us"/>
          </PerformanceCounterConfiguration>
        </PerformanceCounters>
        <Metrics resourceId="(Update with resource ID for the VM)" >
            <MetricAggregation scheduledTransferPeriod="PT1H"/>
            <MetricAggregation scheduledTransferPeriod="PT1M"/>
        </Metrics>
        <WindowsEventLog scheduledTransferPeriod="PT1M">
          <DataSource name="Application!*[System[(Level = 1 or Level = 2)]]"/>
          <DataSource name="Security!*[System[(Level = 1 or Level = 2)]"/>
          <DataSource name="System!*[System[(Level = 1 or Level = 2)]]"/>
        </WindowsEventLog>
          </DiagnosticMonitorConfiguration>
        </WadCfg>
        <StorageAccount>(Update with diagnostics storage account name)</StorageAccount>
    </PublicConfig>
    ```

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные рекомендации по использованию системы диагностики Azure и других методов для устранения неполадок см. в статье [Включение системы диагностики Azure в облачных службах Azure](../../cloud-services/cloud-services-dotnet-diagnostics.md).
* Пояснение различных параметров XML-конфигураций для расширения диагностики см. в статье, посвященной [схеме конфигураций диагностики](../../azure-monitor/agents/diagnostics-extension-versions.md).