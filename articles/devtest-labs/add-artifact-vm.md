---
title: Добавление артефакта в виртуальную машину в Azure DevTest Labs | Документация Майкрософт
description: Узнайте, как добавить артефакт в виртуальную машину в лаборатории Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b4772755d8077f7a659c4d403961ffaeb9e1d483
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85483896"
---
# <a name="add-an-artifact-to-a-vm"></a>Добавление артефакта на виртуальную машину
При создании виртуальной машины в нее можно добавить существующие артефакты. Эти артефакты могут находиться в [общедоступном репозитории Git DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) или в собственном репозитории Git. В этой статье показано, как добавить артефакты в портал Azure и с помощью Azure PowerShell. 

*Артефакты* Azure DevTest Labs позволяют указывать *действия*, которые выполняются при подготовке виртуальной машины, включая запуск скриптов Windows PowerShell, выполнение команд Bash и установку программного обеспечения. *Параметры* артефакта позволяют настроить артефакт для конкретной ситуации.

Дополнительные сведения о создании пользовательских артефактов см. в статье [Создание пользовательских артефактов](devtest-lab-artifact-author.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="use-azure-portal"></a>Использование портала Azure 
1. Войдите на [портал Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Щелкните **Все службы** и выберите в списке **DevTest Labs**.
1. Из списка лабораторий выберите ту, где содержится виртуальная машина, с которой вы будете работать.  
1. Выберите **My virtual machines** (Мои виртуальные машины).
1. Выберите нужную виртуальную машину.
1. Выберите **Управление артефактами**. 
1. Выберите **Apply artifacts** (Применить артефакты).
1. В области **Применение артефактов** выберите артефакт для добавления к виртуальной машине.
1. В области **Добавление артефакта** введите значения обязательных параметров и другие параметры по желанию.  
1. Выберите **Добавить**, чтобы добавить артефакт и вернуться в область **Применение артефактов**.
1. Продолжайте добавлять артефакты, необходимые для виртуальной машины.
1. После добавления артефактов можно [изменить порядок, в котором они выполняются](#change-the-order-in-which-artifacts-are-run). Также можно вернуться к [просмотру или изменению артефакта](#view-or-modify-an-artifact).
1. После добавления артефактов выберите **Применить**.

### <a name="change-the-order-in-which-artifacts-are-run"></a>Изменение порядка выполнения артефактов
По умолчанию действия артефактов выполняются в порядке их добавления на виртуальную машину. Ниже показано, как изменить порядок, в котором выполняются артефакты.

1. В верхней части области **Применение артефактов** щелкните ссылку, указывающую количество добавленных к виртуальной машине артефактов.
   
    ![Количество артефактов, добавленных к виртуальной машине](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. В области **Выбранные артефакты** расположите артефакты в нужном порядке с помощью перетаскивания. Если у вас возникли проблемы при перетаскивании артефактов, убедитесь, что вы перетаскиваете их за левую сторону. 
1. По окончании нажмите кнопку **ОК** .  

### <a name="view-or-modify-an-artifact"></a>просмотру или изменению артефакта
Для просмотра или изменения параметров артефакта выполните следующие действия:

1. В верхней части области **Применение артефактов** щелкните ссылку, указывающую количество добавленных к виртуальной машине артефактов.
   
    ![Количество артефактов, добавленных к виртуальной машине](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. В области **Выбранные артефакты** выберите артефакт, который нужно просмотреть или изменить.  
1. В области **Добавление артефакта** внесите необходимые изменения. Затем нажмите кнопку **ОК**, чтобы закрыть область **Добавление артефакта**.
1. Нажмите кнопку **ОК**, чтобы закрыть область **Выбранные артефакты**.

## <a name="use-powershell"></a>Использование PowerShell
Следующий скрипт применяет указанный артефакт к указанной виртуальной машине. Команда [Invoke-азресаурцеактион](/powershell/module/az.resources/invoke-azresourceaction) выполняет операцию.  

```powershell
#Requires -Module Az.Resources

param
(
[Parameter(Mandatory=$true, HelpMessage="The ID of the subscription that contains the lab")]
   [string] $SubscriptionId,
[Parameter(Mandatory=$true, HelpMessage="The name of the lab containing the virtual machine")]
   [string] $DevTestLabName,
[Parameter(Mandatory=$true, HelpMessage="The name of the virtual machine")]
   [string] $VirtualMachineName,
[Parameter(Mandatory=$true, HelpMessage="The repository where the artifact is stored")]
   [string] $RepositoryName,
[Parameter(Mandatory=$true, HelpMessage="The artifact to apply to the virtual machine")]
   [string] $ArtifactName,
[Parameter(ValueFromRemainingArguments=$true)]
   $Params
)

# Set the appropriate subscription
Set-AzContext -SubscriptionId $SubscriptionId | Out-Null
 
# Get the lab resource group name
$resourceGroupName = (Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' | Where-Object { $_.Name -eq $DevTestLabName}).ResourceGroupName
if ($resourceGroupName -eq $null) { throw "Unable to find lab $DevTestLabName in subscription $SubscriptionId." }

# Get the internal repo name
$repository = Get-AzResource -ResourceGroupName $resourceGroupName `
                    -ResourceType 'Microsoft.DevTestLab/labs/artifactsources' `
                    -ResourceName $DevTestLabName `
                    -ApiVersion 2016-05-15 `
                    | Where-Object { $RepositoryName -in ($_.Name, $_.Properties.displayName) } `
                    | Select-Object -First 1

if ($repository -eq $null) { "Unable to find repository $RepositoryName in lab $DevTestLabName." }

# Get the internal artifact name
$template = Get-AzResource -ResourceGroupName $resourceGroupName `
                -ResourceType "Microsoft.DevTestLab/labs/artifactSources/artifacts" `
                -ResourceName "$DevTestLabName/$($repository.Name)" `
                -ApiVersion 2016-05-15 `
                | Where-Object { $ArtifactName -in ($_.Name, $_.Properties.title) } `
                | Select-Object -First 1

if ($template -eq $null) { throw "Unable to find template $ArtifactName in lab $DevTestLabName." }

# Find the virtual machine in Azure
$FullVMId = "/subscriptions/$SubscriptionId/resourceGroups/$resourceGroupName`
                /providers/Microsoft.DevTestLab/labs/$DevTestLabName/virtualmachines/$virtualMachineName"

$virtualMachine = Get-AzResource -ResourceId $FullVMId

# Generate the artifact id
$FullArtifactId = "/subscriptions/$SubscriptionId/resourceGroups/$resourceGroupName`
                        /providers/Microsoft.DevTestLab/labs/$DevTestLabName/artifactSources/$($repository.Name)`
                        /artifacts/$($template.Name)"

# Handle the inputted parameters to pass through
$artifactParameters = @()

# Fill artifact parameter with the additional -param_ data and strip off the -param_
$Params | ForEach-Object {
   if ($_ -match '^-param_(.*)') {
      $name = $_.TrimStart('^-param_')
   } elseif ( $name ) {
      $artifactParameters += @{ "name" = "$name"; "value" = "$_" }
      $name = $null #reset name variable
   }
}

# Create structure for the artifact data to be passed to the action

$prop = @{
artifacts = @(
    @{
        artifactId = $FullArtifactId
        parameters = $artifactParameters
    }
    )
}

# Check the VM
if ($virtualMachine -ne $null) {
   # Apply the artifact by name to the virtual machine
   $status = Invoke-AzResourceAction -Parameters $prop -ResourceId $virtualMachine.ResourceId -Action "applyArtifacts" -ApiVersion 2016-05-15 -Force
   if ($status.Status -eq 'Succeeded') {
      Write-Output "##[section] Successfully applied artifact: $ArtifactName to $VirtualMachineName"
   } else {
      Write-Error "##[error]Failed to apply artifact: $ArtifactName to $VirtualMachineName"
   }
} else {
   Write-Error "##[error]$VirtualMachine was not found in the DevTest Lab, unable to apply the artifact"
}

```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о артефактах см. в следующих статьях:

- [Укажите обязательные артефакты для лаборатории](devtest-lab-mandatory-artifacts.md)
- [Создание настраиваемых артефактов](devtest-lab-artifact-author.md)
- [Добавление репозитория артефактов в лабораторию](devtest-lab-artifact-author.md)
- [Диагностика сбоев артефактов](devtest-lab-troubleshoot-artifact-failure.md)
