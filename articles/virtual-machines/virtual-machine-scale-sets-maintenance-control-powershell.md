---
title: Управление обновлением образа ОС в масштабируемых наборах виртуальных машин Azure с помощью PowerShell
description: Узнайте, как управлять развертыванием автоматических обновлений образов ОС в масштабируемых наборах виртуальных машин Azure с помощью управления обслуживанием и PowerShell.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: jushiman
ms.openlocfilehash: d8acab17e9d8dfc078b46f6a279cc671a70b0a50
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91974844"
---
# <a name="preview-maintenance-control-for-os-image-upgrades-on-azure-virtual-machine-scale-sets-using-powershell"></a>Предварительный просмотр. Управление обновлением образа ОС в масштабируемых наборах виртуальных машин Azure с помощью PowerShell

Управление обслуживанием позволяет решить, когда следует применять автоматические обновления образов гостевой ОС к масштабируемым наборам виртуальных машин. В этом разделе рассматриваются параметры Azure PowerShell для управления обслуживанием. Дополнительные сведения об использовании управления обслуживанием см. в разделе [Управление обслуживанием для масштабируемых наборов виртуальных машин Azure](virtual-machine-scale-sets-maintenance-control.md).

> [!IMPORTANT]
> Управление обслуживанием для обновления образа ОС в масштабируемых наборах виртуальных машин Azure сейчас находится в общедоступной предварительной версии.
> Предварительная версия предоставляется без соглашения об уровне обслуживания. Мы не рекомендуем использовать ее в рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="enable-the-powershell-module"></a>Включение модуля PowerShell

Убедитесь `PowerShellGet` , что оно не устарело.    

```azurepowershell-interactive  
Install-Module -Name PowerShellGet -Repository PSGallery -Force 
``` 

Установите `Az.Maintenance` модуль PowerShell.     

```azurepowershell-interactive  
Install-Module -Name Az.Maintenance
``` 

Если выполняется локальная установка, убедитесь, что вы откроете командную строку PowerShell с правами администратора.

Вам также может быть предложено подтвердить установку из *ненадежного репозитория*. Введите `Y` или выберите **Да для всех** , чтобы установить модуль.

## <a name="connect-to-an-azure-account"></a>Подключение к учетной записи Azure

Подключитесь к нужной учетной записи Azure с помощью команды [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) и [Set-азаккаунт](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Connect-AzAccount
Set-AzContext 00a000aa-0a00-0a0a-00aa-a00a000aaa00

$RGName="myMaintenanceRG"
$MaintenanceConfig="myMaintenanceConfig"
$location="eastus2"
$vmss="myMaintenanceVMSS"
```

## <a name="create-a-maintenance-configuration"></a>Создайте конфигурацию обслуживания.

Создайте группу ресурсов в качестве контейнера для вашей конфигурации. В этом примере в *eastus2* создается группа ресурсов с именем *мимаинтенанцерг* . Если у вас уже есть группа ресурсов, которую вы хотите использовать, эту часть можно пропустить. Просто замените имя группы ресурсов своим собственным в остальных примерах.

```azurepowershell-interactive
New-AzResourceGroup `
   -Location $location `
   -Name $RGName
```

Используйте [New-азмаинтенанцеконфигуратион](/powershell/module/az.maintenance/new-azmaintenanceconfiguration) для создания конфигурации обслуживания. В этом примере создается конфигурация обслуживания с именем *myConfig* Scope в образе ОС. 

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup $RGName `
   -Name $MaintenanceConfig `
   -MaintenanceScope OSImage `
   -Location $location `
   -StartDateTime "2020-10-01 00:00" `
   -TimeZone "Pacific Standard Time" `
   -Duration "05:00" `
   -RecurEvery "Day"
```

> [!IMPORTANT]
> **Длительность** обслуживания должна быть не более *5 часов* . Для **периодичности** обслуживания необходимо задать значение " *день*".

Использование `-MaintenanceScope OSImage` гарантирует, что конфигурация обслуживания будет использоваться для управления обновлениями гостевой ОС.

При попытке создать конфигурацию с тем же именем, но в другом расположении вы получите сообщение об ошибке. Имена конфигураций должны быть уникальными для группы ресурсов.

Вы можете запросить доступные конфигурации обслуживания с помощью команды [Get-азмаинтенанцеконфигуратион](/powershell/module/az.maintenance/get-azmaintenanceconfiguration).

```azurepowershell-interactive
Get-AzMaintenanceConfiguration | Format-Table -Property Name,Id
```

## <a name="associate-your-virtual-machine-scale-set-to-the-maintenance-configuration"></a>Свяжите масштабируемый набор виртуальных машин с конфигурацией обслуживания

Масштабируемый набор виртуальных машин можно связать с любой конфигурацией обслуживания независимо от региона и подписки конфигурации обслуживания. После включения в конфигурацию обслуживания новые обновления образа ОС для масштабируемого набора будут автоматически запланированы в следующем доступном периоде обслуживания.

Используйте [New-азконфигуратионассигнмент](/powershell/module/az.maintenance/new-azconfigurationassignment) , чтобы связать масштабируемый набор виртуальных машин с конфигурацией обслуживания.

```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName $RGName `
   -Location $location `
   -ResourceName $vmss `
   -ResourceType VirtualMachineScaleSets `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name`
   -MaintenanceConfigurationId $config.Id
``` 

## <a name="enable-automatic-os-upgrade"></a>Включить автоматическое обновление ОС

Вы можете включить автоматическое обновление ОС для каждого масштабируемого набора виртуальных машин, который будет использовать управление обслуживанием. См. статью документирование [автоматического обновления образа ОС в масштабируемом наборе виртуальных машин Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) для включения автоматического обновления ОС в масштабируемом наборе виртуальных машин. 


## <a name="next-steps"></a>Дальнейшие действия

Сведения об обслуживании и обновлениях для виртуальных машин, работающих в Azure.

> [!div class="nextstepaction"]
> [Обслуживание и обновления](maintenance-and-updates.md)