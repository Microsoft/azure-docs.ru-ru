---
title: Обновление расширения наблюдателя за сетями до последней версии
description: Узнайте, как обновить расширение наблюдателя за сетями Azure до последней версии.
services: virtual-machines
documentationcenter: ''
author: damendo
manager: balar
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: article
ms.workload: infrastructure-services
ms.date: 09/23/2020
ms.author: damendo
ms.openlocfilehash: 42efe2927b4d711f7fa66a96ebd25f1a62bf654a
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102563614"
---
# <a name="update-the-network-watcher-extension-to-the-latest-version"></a>Обновление расширения наблюдателя за сетями до последней версии

## <a name="overview"></a>Обзор

[Наблюдатель за сетями Azure](../../network-watcher/network-watcher-monitoring-overview.md) — это служба наблюдения за производительностью сети, диагностики и аналитики, которая наблюдает за сетями Azure. Расширение виртуальной машины агента наблюдателя за сетями — это требование для записи сетевого трафика по требованию и использования других расширенных функций на виртуальных машинах Azure. Расширение наблюдателя за сетями используется такими функциями, как монитор подключения, монитор подключения (Предварительная версия), устранение неполадок подключения и запись пакетов.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что на виртуальной машине установлено расширение наблюдателя за сетями.

## <a name="latest-version"></a>Последняя версия

В настоящее время доступна последняя версия расширения наблюдателя за сетями `1.4.1693.1` .

## <a name="update-your-extension-using-a-powershell-script"></a>Обновление расширения с помощью сценария PowerShell
Клиенты с большими развертываниями, которым необходимо одновременно обновить несколько виртуальных машин. Сведения об обновлении выбранных виртуальных машин вручную см. в следующем разделе. 

```powershell
<#
    .SYNOPSIS
    This script will scan all VMs in the provided subscription and upgrade any out of date AzureNetworkWatcherExtensions

    .DESCRIPTION
    This script should be no-op if AzureNetworkWatcherExtensions are up to date
    Requires Azure PowerShell 4.2 or higher to be installed (e.g. Install-Module AzureRM).

    .EXAMPLE
    .\UpdateVMAgentsInSub.ps1 -SubID F4BC4873-5DAB-491E-B713-1358EF4992F2 -NoUpdate

#>
[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [string] $SubID,
    [Parameter(Mandatory=$false)]
    [Switch] $NoUpdate = $false,
    [Parameter(Mandatory=$false)]
    [string] $MinVersion = "1.4.1654.1"
)


function NeedsUpdate($version)
{
    if ($version -eq $MinVersion)
    {
        return $false
    }

    $lessThan = $true;
    $versionParts = $version -split '\.';
    $minVersionParts = $MinVersion -split '\.';
    for ($i = 0; $i -lt $versionParts.Length; $i++)
    {
        if ([int]$versionParts[$i] -gt [int]$minVersionParts[$i])
        {
            $lessThan = $false;
            break;
        }
    }

    return $lessThan
}

Write-Host "Scanning all VMs in the subscription: $($SubID)"
Select-AzSubscription -SubscriptionId $SubID;
$vms = Get-AzVM;
$foundVMs = $false;
Write-Host "Starting VM search, this may take a while"

foreach ($vmName in $vms)
{
    # Get Detailed VM info
    $vm = Get-AzVM -ResourceGroupName $vmName.ResourceGroupName -Name $vmName.name -Status;
    $isWindows = $vm.OsVersion -match "Windows";
    foreach ($extension in $vm.Extensions)
    {
        if ($extension.Name -eq "AzureNetworkWatcherExtension")
        {
            if (NeedsUpdate($extension.TypeHandlerVersion))
            {
                $foundVMs = $true;
                if (-not ($NoUpdate))
                {
                    Write-Host "Found VM that needs to be updated: subscriptions/$($SubID)/resourceGroups/$($vm.ResourceGroupName)/providers/Microsoft.Compute/virtualMachines/$($vm.Name) -> Updating " -NoNewline
                    Remove-AzVMExtension -ResourceGroupName $vm.ResourceGroupName -VMName $vm.Name -Name "AzureNetworkWatcherExtension" -Force
                    Write-Host "... " -NoNewline
                    $type = if ($isWindows) { "NetworkWatcherAgentWindows" } else { "NetworkWatcherAgentLinux" };
                    Set-AzVMExtension -ResourceGroupName $vm.ResourceGroupName -Location $vmName.Location -VMName $vm.Name -Name "AzureNetworkWatcherExtension" -Publisher "Microsoft.Azure.NetworkWatcher" -Type $type -typeHandlerVersion "1.4"
                    Write-Host "Done"
                }
                else
                {
                    Write-Host "Found $(if ($isWindows) {"Windows"} else {"Linux"}) VM that needs to be updated: subscriptions/$($SubID)/resourceGroups/$($vm.ResourceGroupName)/providers/Microsoft.Compute/virtualMachines/$($vm.Name)"
                }
            }
        }
    }
}

if ($foundVMs)
{
    Write-Host "Finished $(if ($NoUpdate) {"searching"} else {"updating"}) out of date AzureNetworkWatcherExtension on VMs"
}
else
{
    Write-Host "All AzureNetworkWatcherExtensions up to date"
}

```

## <a name="update-your-extension-manually"></a>Обновление расширения вручную

Чтобы обновить расширение, необходимо знать версию расширения.

### <a name="check-your-extension-version"></a>Проверка версии расширения

Версию расширения можно проверить с помощью портал Azure, Azure CLI или PowerShell.

#### <a name="usetheazureportal"></a>Использование портал Azure

1. Перейдите в область **расширения** виртуальной машины на портал Azure.
1. Выберите расширение **азуренетворкватчер** , чтобы открыть область сведений.
1. В поле **Version (версия** ) щелкните номер версии.  

#### <a name="use-the-azure-cli"></a>Использование Azure CLI

Выполните следующую команду в командной строке Azure CLI:

```azurecli
az vm get-instance-view --resource-group  "SampleRG" --name "Sample-VM"
```
Найдите **"азуренетворкватчерекстенсион"** в выходных данных и выясните номер версии из поля *"TypeHandlerVersion"* в выходных данных.  Примечание. сведения о расширении встречаются в выходных данных JSON несколько раз. Проверьте блок "Extensions", и вы увидите полный номер версии расширения. 

Вы должны увидеть нечто вроде: ![ Azure CLI снимке экрана](./media/network-watcher/azure-cli-screenshot.png)

#### <a name="usepowershell"></a>Использование PowerShell

Выполните следующие команды в командной строке PowerShell:

```powershell
Get-AzVM -ResourceGroupName "SampleRG" -Name "Sample-VM" -Status
```
Найдите в выходных данных расширение наблюдателя за сетями Azure и найдите номер версии из поля *"TypeHandlerVersion"* в выходных данных.   

Вы должны увидеть нечто вроде: ![ снимок экрана PowerShell](./media/network-watcher/powershell-screenshot.png)

### <a name="update-your-extension"></a>Обновление расширения

Если ваша версия ниже последней указанной выше версии, обновите расширение, используя любой из следующих вариантов.

#### <a name="option-1-use-powershell"></a>Вариант 1. Использование PowerShell

Выполните следующие команды:

```powershell
#Linux command
Set-AzVMExtension -ResourceGroupName "myResourceGroup1" -Location "WestUS" -VMName "myVM1" -Name "AzureNetworkWatcherExtension" -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentLinux"

#Windows command
Set-AzVMExtension -ResourceGroupName "myResourceGroup1" -Location "WestUS" -VMName "myVM1" -Name "NetworkWatcherAgentWindows" -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentWindows" -ForceRerun "True"

```

Если это не сработает. Удалите и снова установите расширение, выполнив приведенные ниже действия. При этом будет автоматически добавлена последняя версия.

Удаление расширения 

```powershell
#Same command for Linux and Windows
Remove-AzVMExtension -ResourceGroupName "SampleRG" -VMName "Sample-VM" -Name "AzureNetworkWatcherExtension"
``` 

Повторная установка расширения

```powershell
#Linux command
Set-AzVMExtension -ResourceGroupName "SampleRG" -Location "centralus" -VMName "Sample-VM" -Name "AzureNetworkWatcherExtension" -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentLinux" -typeHandlerVersion "1.4"

#Windows command
Set-AzVMExtension -ResourceGroupName "SampleRG" -Location "centralus" -VMName "Sample-VM" -Name "AzureNetworkWatcherExtension" -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentWindows" -typeHandlerVersion "1.4"
```

#### <a name="option-2-use-the-azure-cli"></a>Вариант 2. Использование Azure CLI

Принудительное обновление.

```azurecli
#Linux command
az vm extension set --resource-group "myResourceGroup1" --vm-name "myVM1" --name "NetworkWatcherAgentLinux" --publisher "Microsoft.Azure.NetworkWatcher" --force-update

#Windows command
az vm extension set --resource-group "myResourceGroup1" --vm-name "myVM1" --name "NetworkWatcherAgentWindows" --publisher "Microsoft.Azure.NetworkWatcher" --force-update
```

Если это не сработает, удалите и снова установите расширение и выполните следующие действия, чтобы автоматически добавить последнюю версию.

Удалите расширение.

```azurecli
#Same for Linux and Windows
az vm extension delete --resource-group "myResourceGroup1" --vm-name "myVM1" -n "AzureNetworkWatcherExtension"

```

Снова установите расширение.

```azurecli
#Linux command
az vm extension set --resource-group "DALANDEMO" --vm-name "Linux-01" --name "NetworkWatcherAgentLinux" --publisher "Microsoft.Azure.NetworkWatcher"

#Windows command
az vm extension set --resource-group "DALANDEMO" --vm-name "Linux-01" --name "NetworkWatcherAgentWindows" --publisher "Microsoft.Azure.NetworkWatcher"

```

#### <a name="option-3-reboot-your-vms"></a>Вариант 3. Перезагрузите виртуальные машины

Если автоматическое обновление установлено в true для расширения наблюдателя за сетями, перезагрузите установку виртуальной машины на последнее расширение.

## <a name="support"></a>Поддержка

Если вам нужна дополнительная помощь в любой момент в этой статье, см. документацию по расширению наблюдателя за сетями для [Linux](./network-watcher-linux.md) или [Windows](./network-watcher-windows.md). Вы также можете обратиться к экспертам Azure на [форумах MSDN Azure и Stack overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зафайлировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/)и выберите **получить поддержку**. Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).
