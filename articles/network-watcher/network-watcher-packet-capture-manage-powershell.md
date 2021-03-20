---
title: Управление захватом пакетов — Azure PowerShell
titleSuffix: Azure Network Watcher
description: На этой странице объясняется, как управлять функцией записи пакетов службы наблюдения за сетями с помощью PowerShell
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: 8779381425d4180cc7320f251de9c2cfdd1f0510
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99223429"
---
# <a name="manage-packet-captures-with-azure-network-watcher-using-powershell"></a>Управление записью пакетов с помощью Наблюдателя за сетями Azure в PowerShell

> [!div class="op_single_selector"]
> - [Портал Azure](network-watcher-packet-capture-manage-portal.md)
> - [PowerShell](network-watcher-packet-capture-manage-powershell.md)
> - [Azure CLI](network-watcher-packet-capture-manage-cli.md)
> - [Azure REST API](network-watcher-packet-capture-manage-rest.md)

Возможность записи пакетов Наблюдателя за сетями позволяет создавать сеансы записи для отслеживания входящего и исходящего трафика виртуальной машины. Для сеанса записи предоставляются фильтры, которые позволяют убедиться, что записывается только требуемый трафик. Записи пакетов помогают выявить аномалии в работе сети по факту или заранее. Они также помогают выполнять сбор сетевой статистики, получать сведения о сетевых вторжениях, выполнять отладку передачи данных между клиентом и сервером и многое другое. Так как запись пакетов активируется удаленно, ее не нужно запускать вручную. К тому же она сразу выполняется на требуемой виртуальной машине, что также позволяет сэкономить ценное время.

В этой статье вы ознакомитесь с разными задачами управления, доступными в настоящее время для записи пакетов.

- [**Запуск записи пакета**](#start-a-packet-capture)
- [**Прекращение записи пакета**](#stop-a-packet-capture)
- [**Удаление записи пакета**](#delete-a-packet-capture)
- [**Скачивание записи пакета**](#download-a-packet-capture)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Перед началом

В данной статье предполагается, что у вас есть следующие ресурсы:

* экземпляр Наблюдателя за сетями в регионе, в котором нужно создать запись пакетов;

* виртуальная машина с включенным расширением для записи пакетов.

> [!IMPORTANT]
> Для записи пакетов необходимо расширение виртуальной машины `AzureNetworkWatcherExtension`. Информацию об установке расширения для виртуальной машины Windows см. в статье [Расширение виртуальной машины агента Наблюдателя за сетями для Windows](../virtual-machines/extensions/network-watcher-windows.md), а для виртуальной машины Linux — в статье [Расширение виртуальной машины агента Наблюдателя за сетями для Linux](../virtual-machines/extensions/network-watcher-linux.md).

## <a name="install-vm-extension"></a>Установка расширения виртуальной машины

### <a name="step-1"></a>Шаг 1

```powershell
$VM = Get-AzVM -ResourceGroupName testrg -Name VM1
```

### <a name="step-2"></a>Шаг 2

В следующем примере извлекается информация о расширении, необходимом для выполнения командлета `Set-AzVMExtension`. Этот командлет устанавливает агент записи пакетов на гостевой виртуальной машине.

> [!NOTE]
> Для завершения командлетом `Set-AzVMExtension` этой операции может потребоваться несколько минут.

Для виртуальных машин Windows:

```powershell
$AzureNetworkWatcherExtension = Get-AzVMExtensionImage -Location WestCentralUS -PublisherName Microsoft.Azure.NetworkWatcher -Type NetworkWatcherAgentWindows -Version 1.4.585.2
$ExtensionName = "AzureNetworkWatcherExtension"
Set-AzVMExtension -ResourceGroupName $VM.ResourceGroupName  -Location $VM.Location -VMName $VM.Name -Name $ExtensionName -Publisher $AzureNetworkWatcherExtension.PublisherName -ExtensionType $AzureNetworkWatcherExtension.Type -TypeHandlerVersion $AzureNetworkWatcherExtension.Version.Substring(0,3)
```

Для виртуальных машин Linux:

```powershell
$AzureNetworkWatcherExtension = Get-AzVMExtensionImage -Location WestCentralUS -PublisherName Microsoft.Azure.NetworkWatcher -Type NetworkWatcherAgentLinux -Version 1.4.13.0
$ExtensionName = "AzureNetworkWatcherExtension"
Set-AzVMExtension -ResourceGroupName $VM.ResourceGroupName  -Location $VM.Location -VMName $VM.Name -Name $ExtensionName -Publisher $AzureNetworkWatcherExtension.PublisherName -ExtensionType $AzureNetworkWatcherExtension.Type -TypeHandlerVersion $AzureNetworkWatcherExtension.Version.Substring(0,3)
```

В следующем примере приведен успешный ответ после выполнения командлета `Set-AzVMExtension`.

```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   
```

### <a name="step-3"></a>Шаг 3

Чтобы убедиться, что агент установлен, выполните командлет `Get-AzVMExtension` и передайте ему имя виртуальной машины и расширения.

```powershell
Get-AzVMExtension -ResourceGroupName $VM.ResourceGroupName  -VMName $VM.Name -Name $ExtensionName
```

Ниже приведен пример ответа после выполнения операции `Get-AzVMExtension`.

```
ResourceGroupName       : testrg
VMName                  : testvm1
Name                    : AzureNetworkWatcherExtension
Location                : westcentralus
Etag                    : null
Publisher               : Microsoft.Azure.NetworkWatcher
ExtensionType           : NetworkWatcherAgentWindows
TypeHandlerVersion      : 1.4
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1/
                          extensions/AzureNetworkWatcherExtension
PublicSettings          : 
ProtectedSettings       : 
ProvisioningState       : Succeeded
Statuses                : 
SubStatuses             : 
AutoUpgradeMinorVersion : True
ForceUpdateTag          : 
```

## <a name="start-a-packet-capture"></a>Запуск записи пакета

После выполнения предыдущих шагов на виртуальной машине будет установлен агент записи пакетов.

### <a name="step-1"></a>Шаг 1

Далее необходимо извлечь экземпляр Наблюдателя за сетями. Эта переменная передается в командлет `New-AzNetworkWatcherPacketCapture` на шаге 4.

```powershell
$networkWatcher = Get-AzNetworkWatcher  | Where {$_.Location -eq "westcentralus" }
```

### <a name="step-2"></a>Шаг 2

Получите учетную запись хранения. Она используется для хранения файла записи пакетов.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName testrg -Name testrgsa123
```

### <a name="step-3"></a>Шаг 3

С помощью фильтров можно ограничить данные, которые сохраняются при записи пакетов. В следующем примере устанавливаются два фильтра.  Один фильтр собирает исходящий TCP-трафик только с локального IP-адреса 10.0.0.3 на порты назначения 20, 80 и 443.  Второй фильтр собирает только трафик, передаваемый по протоколу UDP.

```powershell
$filter1 = New-AzPacketCaptureFilterConfig -Protocol TCP -RemoteIPAddress "1.1.1.1-255.255.255.255" -LocalIPAddress "10.0.0.3" -LocalPort "1-65535" -RemotePort "20;80;443"
$filter2 = New-AzPacketCaptureFilterConfig -Protocol UDP
```

> [!NOTE]
> Для записи пакетов можно определить несколько фильтров.

### <a name="step-4"></a>Шаг 4

Выполните командлет `New-AzNetworkWatcherPacketCapture`, чтобы начать процесс записи пакетов, передав требуемые значения, полученные на предыдущих шагах.
```powershell

New-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $vm.Id -PacketCaptureName "PacketCaptureTest" -StorageAccountId $storageAccount.id -TimeLimitInSeconds 60 -Filter $filter1, $filter2
```

Ниже приведен пример ожидаемого результата выполнения командлета `New-AzNetworkWatcherPacketCapture`.

```
Name                    : PacketCaptureTest
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatcher
                          s/NetworkWatcher_westcentralus/packetCaptures/PacketCaptureTest
Etag                    : W/"3bf27278-8251-4651-9546-c7f369855e4e"
ProvisioningState       : Succeeded
Target                  : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1
BytesToCapturePerPacket : 0
TotalBytesPerSession    : 1073741824
TimeLimitInSeconds      : 60
StorageLocation         : {
                            "StorageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Storage/storageA
                          ccounts/examplestorage",
                            "StoragePath": "https://examplestorage.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-00000
                          0000000/resourcegroups/testrg/providers/microsoft.compute/virtualmachines/testvm1/2017/02/01/packetcapture_22_42_48_238.cap"
                          }
Filters                 : [
                            {
                              "Protocol": "TCP",
                              "RemoteIPAddress": "1.1.1.1-255.255.255",
                              "LocalIPAddress": "10.0.0.3",
                              "LocalPort": "1-65535",
                              "RemotePort": "20;80;443"
                            },
                            {
                              "Protocol": "UDP",
                              "RemoteIPAddress": "",
                              "LocalIPAddress": "",
                              "LocalPort": "",
                              "RemotePort": ""
                            }
                          ]


```

## <a name="get-a-packet-capture"></a>Получение записи пакета

При выполнении командлета `Get-AzNetworkWatcherPacketCapture` вы получаете сведения о состоянии выполняющейся или завершенной записи пакетов.

```powershell
Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

Ниже представлен пример выходных данных командлета `Get-AzNetworkWatcherPacketCapture`, полученных после завершения записи пакетов. В качестве значения параметра PacketCaptureStatus указано Stopped, а для параметра StopReason задано значение TimeExceeded. По этому значению можно понять, что запись пакетов выполнена успешно за требуемое время.
```
Name                    : PacketCaptureTest
Id                      : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatcher
                          s/NetworkWatcher_westcentralus/packetCaptures/PacketCaptureTest
Etag                    : W/"4b9a81ed-dc63-472e-869e-96d7166ccb9b"
ProvisioningState       : Succeeded
Target                  : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm1
BytesToCapturePerPacket : 0
TotalBytesPerSession    : 1073741824
TimeLimitInSeconds      : 60
StorageLocation         : {
                            "StorageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Storage/storageA
                          ccounts/examplestorage",
                            "StoragePath": "https://examplestorage.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-00000
                          0000000/resourcegroups/testrg/providers/microsoft.compute/virtualmachines/testvm1/2017/02/01/packetcapture_22_42_48_238.cap"
                          }
Filters                 : [
                            {
                              "Protocol": "TCP",
                              "RemoteIPAddress": "1.1.1.1-255.255.255",
                              "LocalIPAddress": "10.0.0.3",
                              "LocalPort": "1-65535",
                              "RemotePort": "20;80;443"
                            },
                            {
                              "Protocol": "UDP",
                              "RemoteIPAddress": "",
                              "LocalIPAddress": "",
                              "LocalPort": "",
                              "RemotePort": ""
                            }
                          ]
CaptureStartTime        : 2/1/2017 10:43:01 PM
PacketCaptureStatus     : Stopped
StopReason              : TimeExceeded
PacketCaptureError      : []
```

## <a name="stop-a-packet-capture"></a>Прекращение записи пакета

Если выполнить командлет `Stop-AzNetworkWatcherPacketCapture` во время сеанса записи пакета, он будет остановлен.

```powershell
Stop-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

> [!NOTE]
> При выполнении во время текущего сеанса записи или в имеющемся остановленном сеансе командлет не возвратит ответ.

## <a name="delete-a-packet-capture"></a>Удаление записи пакета

```powershell
Remove-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName "PacketCaptureTest"
```

> [!NOTE]
> При удалении записи пакета файл в учетной записи хранения не удаляется.

## <a name="download-a-packet-capture"></a>Скачивание записи пакета

После завершения сеанса записи пакета файл записи можно передать в хранилище BLOB-объектов или в локальный файл на виртуальной машине. Место хранения записи пакетов определяется при создании сеанса. Удобное средство для доступа к этим файлам записи, сохраненным в учетной записи хранения, — обозреватель службы хранилища Microsoft Azure, который можно скачать по адресу https://storageexplorer.com/

При указании учетной записи хранения файлы записи пакетов сохраняются в ней по следующему адресу:

```
https://{storageAccountName}.blob.core.windows.net/network-watcher-logs/subscriptions/{subscriptionId}/resourcegroups/{storageAccountResourceGroup}/providers/microsoft.compute/virtualmachines/{VMName}/{year}/{month}/{day}/packetCapture_{creationTime}.cap
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об автоматизации записи пакетов с помощью оповещений на виртуальной машине см. в статье, посвященной [созданию записи пакетов, активируемой с использованием оповещений](network-watcher-alert-triggered-packet-capture.md).

Сведения о состоянии (разрешен или запрещен) входящего и исходящего трафика виртуальной машины см. в статье, посвященной [проверке потока IP-адресов](diagnose-vm-network-traffic-filtering-problem.md).

<!-- Image references -->
