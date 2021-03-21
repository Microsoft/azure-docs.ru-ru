---
title: Включение VM insights с помощью PowerShell
description: Описание включения аналитики виртуальной машины для виртуальных машин Azure или масштабируемых наборов виртуальных машин с помощью Azure PowerShell.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: b51a49abda76a4d1433336cf73dc9a0dd6e40787
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031888"
---
# <a name="enable-vm-insights-using-powershell"></a>Включение VM insights с помощью PowerShell
В этой статье описывается, как включить аналитику ВМ на виртуальных машинах Azure с помощью PowerShell. Эту процедуру можно использовать для следующих действий:

- Виртуальная машина Azure
- Масштабируемый набор виртуальных машин Azure

## <a name="prerequisites"></a>Предварительные условия

- [Создание и Настройка рабочей области log Analytics](./vminsights-configure-workspace.md).
- Ознакомьтесь с [поддерживаемыми операционными системами](./vminsights-enable-overview.md#supported-operating-systems) , чтобы убедиться, что операционная система, которую вы включаете, поддерживается. 


## <a name="powershell-script"></a>Сценарий PowerShell

Чтобы включить аналитику ВМ для нескольких виртуальных машин или масштабируемых наборов виртуальных машин, используйте скрипт PowerShell [Install-VMInsights.ps1](https://www.powershellgallery.com/packages/Install-VMInsights), доступный в коллекции Azure PowerShell. Этот сценарий выполняет итерацию:

- Каждая виртуальная машина и масштабируемый набор виртуальных машин в вашей подписке.
- Группа ресурсов с областью действия, заданная параметром *ResourceGroup*.
- Одна виртуальная машина или масштабируемый набор виртуальных машин, указанный по *имени*.

Для каждой виртуальной машины или масштабируемого набора виртуальных машин скрипт проверяет, установлены ли уже расширения виртуальной машины для агента Log Analytics и агента зависимостей. Если установлены оба расширения, сценарий пытается переустановить его. Если оба расширения не установлены, сценарий установит их.

Убедитесь, что используется модуль Azure PowerShell AZ Version 1.0.0 или более поздней версии с `Enable-AzureRM` включенными псевдонимами совместимости. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

Чтобы получить список сведений об аргументах сценария и пример использования, выполните команду `Get-Help`.

```powershell
Get-Help .\Install-VMInsights.ps1 -Detailed

SYNOPSIS
    This script installs VM extensions for Log Analytics and the Dependency agent as needed for VM Insights.


SYNTAX
    .\Install-VMInsights.ps1 [-WorkspaceId] <String> [-WorkspaceKey] <String> [-SubscriptionId] <String> [[-ResourceGroup]
    <String>] [[-Name] <String>] [[-PolicyAssignmentName] <String>] [-ReInstall] [-TriggerVmssManualVMUpdate] [-Approve] [-WorkspaceRegion] <String>
    [-WhatIf] [-Confirm] [<CommonParameters>]


DESCRIPTION
    This script installs or reconfigures the following on VMs and virtual machine scale sets:
    - Log Analytics VM extension configured to supplied Log Analytics workspace
    - Dependency agent VM extension

    Can be applied to:
    - Subscription
    - Resource group in a subscription
    - Specific VM or virtual machine scale set
    - Compliance results of a policy for a VM or VM extension

    Script will show you a list of VMs or virtual machine scale sets that will apply to and let you confirm to continue.
    Use -Approve switch to run without prompting, if all required parameters are provided.

    If the extensions are already installed, they will not install again.
    Use -ReInstall switch if you need to, for example, update the workspace.

    Use -WhatIf if you want to see what would happen in terms of installs, what workspace configured to, and status of the extension.


PARAMETERS
    -WorkspaceId <String>
        Log Analytics WorkspaceID (GUID) for the data to be sent to

    -WorkspaceKey <String>
        Log Analytics Workspace primary or secondary key

    -SubscriptionId <String>
        SubscriptionId for the VMs/VM Scale Sets
        If using PolicyAssignmentName parameter, subscription that VMs are in

    -ResourceGroup <String>
        <Optional> Resource Group to which the VMs or VM Scale Sets belong

    -Name <String>
        <Optional> To install to a single VM/VM Scale Set

    -PolicyAssignmentName <String>
        <Optional> Take the input VMs to operate on as the Compliance results from this Assignment
        If specified will only take from this source.

    -ReInstall [<SwitchParameter>]
        <Optional> If VM/VM Scale Set is already configured for a different workspace, set this to change to the new workspace

    -TriggerVmssManualVMUpdate [<SwitchParameter>]
        <Optional> Set this flag to trigger update of VM instances in a scale set whose upgrade policy is set to Manual

    -Approve [<SwitchParameter>]
        <Optional> Gives the approval for the installation to start with no confirmation prompt for the listed VMs/VM Scale Sets

    -WorkspaceRegion <String>
        Region the Log Analytics Workspace is in
        Supported values: "East US","eastus","Southeast Asia","southeastasia","West Central US","westcentralus","West Europe","westeurope"
        For Health supported is: "East US","eastus","West Central US","westcentralus"

    -WhatIf [<SwitchParameter>]
        <Optional> See what would happen in terms of installs.
        If extension is already installed will show what workspace is currently configured, and status of the VM extension

    -Confirm [<SwitchParameter>]
        <Optional> Confirm every action

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (https:/go.microsoft.com/fwlink/?LinkID=113216).

    -------------------------- EXAMPLE 1 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup>

    Install for all VMs in a resource group in a subscription

    -------------------------- EXAMPLE 2 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup> -ReInstall

    Specify to reinstall extensions even if already installed, for example, to update to a different workspace

    -------------------------- EXAMPLE 3 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -PolicyAssignmentName a4f79f8ce891455198c08736 -ReInstall

    Specify to use a PolicyAssignmentName for source and to reinstall (move to a new workspace)
```

В следующем примере показано использование команд PowerShell в папке для включения аналитики виртуальной машины и понимания ожидаемых выходных данных:

```powershell
$WorkspaceId = "<GUID>"
$WorkspaceKey = "<Key>"
$SubscriptionId = "<GUID>"
.\Install-VMInsights.ps1 -WorkspaceId $WorkspaceId -WorkspaceKey $WorkspaceKey -SubscriptionId $SubscriptionId -WorkspaceRegion eastus

Getting list of VMs or virtual machine scale sets matching criteria specified

VMs or virtual machine scale sets matching criteria:

db-ws-1 VM running
db-ws2012 VM running

This operation will install the Log Analytics and Dependency agent extensions on the previous two VMs or virtual machine scale sets.
VMs in a non-running state will be skipped.
Extension will not be reinstalled if already installed. Use -ReInstall if desired, for example, to update workspace.

Confirm
Continue?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

db-ws-1 : Deploying DependencyAgentWindows with name DAExtension
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Deploying DependencyAgentWindows with name DAExtension
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Summary:

Already onboarded: (0)

Succeeded: (4)
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Connected to different workspace: (0)

Not running - start VM to configure: (0)

Failed: (0)
```

## <a name="next-steps"></a>Дальнейшие действия

* См. раздел [Использование схемы VM Insights](vminsights-maps.md) для просмотра зависимостей обнаруженных приложений. 
* См. статью [Просмотр производительности виртуальной машины Azure](vminsights-performance.md) для выявления узких мест, общего использования и производительности виртуальной машины.