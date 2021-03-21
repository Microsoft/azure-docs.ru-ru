---
title: Переход на брандмауэр Azure Premium Preview
description: Узнайте, как перейти с уровня "Стандартный" брандмауэра Azure на предварительную версию Azure Firewall Premium Preview.
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 02/16/2021
ms.author: victorh
ms.openlocfilehash: ffdcad33568af9955dbdf5aafb1b6ffe4a51681d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100549855"
---
# <a name="migrate-to-azure-firewall-premium-preview"></a>Переход на брандмауэр Azure Premium Preview

Вы можете перенести стандарт брандмауэра Azure в предварительную версию Azure Firewall Premium Preview, чтобы воспользоваться преимуществами новых возможностей уровня "Премиум". Дополнительные сведения о функциях предварительной версии Azure Firewall Premium см. в разделе [функции брандмауэра Azure Premium Preview](premium-features.md).

В следующих двух примерах показано, как:
- Перенос существующей стандартной политики с помощью Azure PowerShell
- Перенос существующего стандартного брандмауэра (с классическими правилами) в Azure Firewall Premium с применением политики Premium.

## <a name="migrate-an-existing-policy-using-azure-powershell"></a>Перенос существующей политики с помощью Azure PowerShell

`Transform-Policy.ps1` сценарий Azure PowerShell, создающий политику уровня "Премиум" из существующей стандартной политики.

При наличии стандартного идентификатора политики брандмауэра этот сценарий преобразует его в политику брандмауэра Azure уровня "Премиум". Сначала сценарий подключается к учетной записи Azure, извлекает политику, преобразует или добавляет различные параметры, а затем отправляет новую политику Premium. Новая политика Premium называется `<previous_policy_name>_premium` .

Пример использования

`Transform-Policy -PolicyId /subscriptions/XXXXX-XXXXXX-XXXXX/resourceGroups/some-resource-group/providers/Microsoft.Network/firewallPolicies/policy-name`

> [!IMPORTANT]
> Скрипт не переносит параметры аналитики угроз. Эти параметры необходимо отметить перед продолжением и переносом вручную.

```azurepowershell
<#
    .SYNOPSIS
        Given an Azure firewall policy id the script will transform it to a Premium Azure firewall policy. 
        The script will first pull the policy, transform/add various parameters and then upload a new premium policy. 
        The created policy will be named <previous_policy_name>_premium if no new name provided else new policy will be named as the parameter passed.  
    .Example
        Transform-Policy -PolicyId /subscriptions/XXXXX-XXXXXX-XXXXX/resourceGroups/some-resource-group/providers/Microsoft.Network/firewallPolicies/policy-name -NewPolicyName <optional param for the new policy name>
#>

param (
    #Resource id of the azure firewall policy. 
    [Parameter(Mandatory=$true)]
    [string]
    $PolicyId,

    # #new filewallpolicy name, if not specified will be the previous name with the '_premium' suffix
    [Parameter(Mandatory=$false)]
    [string]
    $NewPolicyName = ""
)
$ErrorActionPreference = "Stop"
$script:PolicyId = $PolicyId
$script:PolicyName = $NewPolicyName

function ValidatePolicy {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [Object]
        $Policy
    )

    Write-Host "Validating resource is as expected"

    if ($null -eq $Policy) {
        Write-Error "Recived null policy"
        exit(1)
    }
    if ($Policy.GetType().Name -ne "PSAzureFirewallPolicy") {
        Write-Host "Resource must be of type Microsoft.Network/firewallPolicies" -ForegroundColor Red
        exit(1)
    }

    if ($Policy.Sku.Tier -eq "Premium") {
        Write-Host "Policy is already premium"
        exit(1)
    }
}

function GetPolicyNewName {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [Microsoft.Azure.Commands.Network.Models.PSAzureFirewallPolicy]
        $Policy
    )

    if (-not [string]::IsNullOrEmpty($script:PolicyName)) {
        return $script:PolicyName
    }

    return $Policy.Name + "_premium"
}

function TransformPolicyToPremium {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [Microsoft.Azure.Commands.Network.Models.PSAzureFirewallPolicy]
        $Policy
    )    
    $NewPolicyParameters = @{
                        Name = (GetPolicyNewName -Policy $Policy) 
                        ResourceGroupName = $Policy.ResourceGroupName 
                        Location = $Policy.Location 
                        ThreatIntelMode = $Policy.ThreatIntelMode 
                        BasePolicy = $Policy.BasePolicy 
                        DnsSetting = $Policy.DnsSettings 
                        Tag = $Policy.Tag 
                        SkuTier = "Premium" 
    }

    Write-Host "Creating new policy"
    $premiumPolicy = New-AzFirewallPolicy @NewPolicyParameters

    Write-Host "Populating rules in new policy"
    foreach ($ruleCollectionGroup in $Policy.RuleCollectionGroups) {
        $ruleResource = Get-AzResource -ResourceId $ruleCollectionGroup.Id
        $ruleToTransfom = Get-AzFirewallPolicyRuleCollectionGroup -AzureFirewallPolicy $Policy -Name $ruleResource.Name
        $ruleCollectionGroup = @{
            FirewallPolicyObject = $premiumPolicy
            Priority = $ruleToTransfom.Properties.Priority
            Name = $ruleToTransfom.Name
        }

        if ($ruleToTransfom.Properties.RuleCollection.Count) {
            $ruleCollectionGroup["RuleCollection"] = $ruleToTransfom.Properties.RuleCollection
        }

        Set-AzFirewallPolicyRuleCollectionGroup @ruleCollectionGroup
    }
}

function ValidateAzNetworkModuleExists {
    Write-Host "Validating needed module exists"
    $networkModule = Get-InstalledModule -Name "Az.Network" -ErrorAction SilentlyContinue
    if (($null -eq $networkModule) -or ($networkModule.Version -lt 4.5)){
        Write-Host "Please install Az.Network module version 4.5.0 or higher, see instructions: https://github.com/Azure/azure-powershell#installation"
        exit(1)
    }
    Import-Module Az.Network -MinimumVersion 4.5.0
}

ValidateAzNetworkModuleExists
$policy = Get-AzFirewallPolicy -ResourceId $script:PolicyId
ValidatePolicy -Policy $policy
TransformPolicyToPremium -Policy $policy
```

## <a name="migrate-an-existing-standard-firewall-using-the-azure-portal"></a>Перенос существующего стандартного брандмауэра с помощью портал Azure

В этом примере показано, как использовать портал Azure для переноса стандартного брандмауэра (классические правила) в Azure Firewall Premium с применением политики Premium.

1. В портал Azure выберите свой стандартный брандмауэр. На странице **Обзор** выберите **Миграция в политику брандмауэра**.

   :::image type="content" source="media/premium-migrate/firewall-overview-migrate.png" alt-text="Миграция в политику брандмауэра":::

1. На странице **Миграция на политику брандмауэра** выберите **проверить и создать**.
1. Нажмите кнопку **создания**.

   Развертывание занимает несколько минут.
1. Используйте `Transform-Policy.ps1` [Скрипт Azure PowerShell](#migrate-an-existing-policy-using-azure-powershell) , чтобы преобразовать новую стандартную политику в политику Premium.
1. На портале выберите стандартный ресурс брандмауэра. 
1. В разделе **Автоматизация** выберите **Экспорт шаблона**. Не закрывайте эту вкладку браузера. Вы вернетесь к нему позже.
   > [!TIP]
   > Чтобы предотвратить потерю шаблона, скачайте и сохраните его на случай закрытия или обновления вкладки браузера.
1. Откройте новую вкладку браузера, перейдите к портал Azure и откройте группу ресурсов, содержащую брандмауэр.
1. Удалите существующий стандартный экземпляр брандмауэра.

   Операция займет всего несколько минут.

1. Вернитесь на вкладку браузер с экспортированным шаблоном.
1. Выберите **развернуть**, а затем на странице **Настраиваемое развертывание** выберите **изменить шаблон**.
1. Измените текст шаблона:
   
   1. Под `Microsoft.Network/azureFirewalls` ресурсом в разделе `Properties` , `sku` измените `tier` с "Standard" на "Премиум".
   1. В шаблоне `Parameters` измените `defaultValue` значение параметра `firewallPolicies_FirewallPolicy_,<your policy name>_externalid` с:
      
       `"/subscriptions/<subscription id>/resourceGroups/<your resource group>/providers/Microsoft.Network/firewallPolicies/FirewallPolicy_<your policy name>"`

      на:

      `"/subscriptions/<subscription id>/resourceGroups/<your resource group>/providers/Microsoft.Network/firewallPolicies/FirewallPolicy_<your policy name>_premium"`
1. Щелкните **Сохранить**.
1. Выберите **Review + Create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

После завершения развертывания теперь можно настроить все новые функции предварительной версии брандмауэра Azure Premium.

## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные сведения о расширенных возможностях брандмауэра Azure](premium-features.md)