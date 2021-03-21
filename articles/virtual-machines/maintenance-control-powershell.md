---
title: Управление обслуживанием для виртуальных машин Azure с помощью PowerShell
description: Узнайте, как управлять применением обслуживания к виртуальным машинам Azure с помощью управления обслуживанием и PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/19/2020
ms.author: cynthn
ms.openlocfilehash: 2868d559f0d848095fa7fec174e09e1b9376c4ae
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552462"
---
# <a name="control-updates-with-maintenance-control-and-azure-powershell"></a>Управление обновлениями с помощью управления обслуживанием и Azure PowerShell

Управление обслуживанием позволяет решить, когда следует применять обновления платформы к инфраструктуре узла изолированных виртуальных машин и выделенных узлов Azure. В этом разделе рассматриваются параметры Azure PowerShell для управления обслуживанием. Дополнительные сведения о преимуществах использования управления обслуживанием, его ограничений и других параметров управления см. [в разделе Управление обновлениями платформы с помощью управления обслуживанием](maintenance-control.md).
 
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


## <a name="create-a-maintenance-configuration"></a>Создайте конфигурацию обслуживания.

Создайте группу ресурсов в качестве контейнера для вашей конфигурации. В этом примере в *eastus* создается группа ресурсов с именем *мимаинтенанцерг* . Если у вас уже есть группа ресурсов, которую вы хотите использовать, можно пропустить эту часть и заменить имя группы ресурсов своим владельцем в остальных примерах.

```azurepowershell-interactive
New-AzResourceGroup `
   -Location eastus `
   -Name myMaintenanceRG
```

Используйте [New-азмаинтенанцеконфигуратион](/powershell/module/az.maintenance/new-azmaintenanceconfiguration) для создания конфигурации обслуживания. В этом примере создается конфигурация обслуживания с именем *myConfig* , областью действия которой является узел. 

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup myMaintenanceRG `
   -Name myConfig `
   -MaintenanceScope host `
   -Location  eastus
```

Использование `-MaintenanceScope host` гарантирует, что конфигурация обслуживания будет использоваться для управления обновлениями узла.

При попытке создать конфигурацию с тем же именем, но в другом расположении возникнет ошибка. Имена конфигураций должны быть уникальными для группы ресурсов.

Вы можете запросить доступные конфигурации обслуживания с помощью команды [Get-азмаинтенанцеконфигуратион](/powershell/module/az.maintenance/get-azmaintenanceconfiguration).

```azurepowershell-interactive
Get-AzMaintenanceConfiguration | Format-Table -Property Name,Id
```

### <a name="create-a-maintenance-configuration-with-scheduled-window"></a>Создание конфигурации обслуживания с запланированным окном

Вы также можете объявить запланированное окно, когда Azure будет применять обновления к ресурсам. В этом примере создается конфигурация обслуживания с именем myConfig с запланированным периодом 5 часов в четвертом понедельнике каждого месяца. После создания запланированного окна вам больше не нужно применять обновления вручную.

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup $RGName `
   -Name $MaintenanceConfig `
   -MaintenanceScope Host `
   -Location $location `
   -StartDateTime "2020-10-01 00:00" `
   -TimeZone "Pacific Standard Time" `
   -Duration "05:00" `
   -RecurEvery "Month Fourth Monday"
```
> [!IMPORTANT]
> **Длительность** обслуживания должна составлять *2 часа* или больше. Для **периодичности** обслуживания должно быть задано значение по крайней мере один раз в 35 дней.

**Повторение** обслуживания может быть выражено ежедневно, еженедельно или ежемесячно. Ниже приведены некоторые примеры.
 - **ежедневно**— Рекуревери "Day" **или** "3Days" 
 - **еженедельно**— Рекуревери "3Weeks" **или** "Неделя Суббота, воскресенье" 
 - **Monthly**-Рекуревери "месяц day23, day24" **или** "месяц прошлого воскресенье" **или** "четвертый понедельник четверть месяца"  
      

## <a name="assign-the-configuration"></a>Назначение конфигурации

Используйте [New-азконфигуратионассигнмент](/powershell/module/az.maintenance/new-azconfigurationassignment) , чтобы назначить конфигурацию изолированной виртуальной машине или выделенному узлу Azure.

### <a name="isolated-vm"></a>Изолированная виртуальная машина

Примените конфигурацию к виртуальной машине, используя идентификатор конфигурации. Укажите `-ResourceType VirtualMachines` и введите имя виртуальной машины для `-ResourceName` , а также группу ресурсов виртуальной машины для `-ResourceGroupName` . 

```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName myResourceGroup `
   -Location eastus `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name `
   -MaintenanceConfigurationId $config.Id
```

### <a name="dedicated-host"></a>Выделенный узел

Чтобы применить конфигурацию к выделенному узлу, необходимо также включить `-ResourceType hosts` , `-ResourceParentName` с именем группы узлов и `-ResourceParentType hostGroups` . 


```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName myResourceGroup `
   -Location eastus `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name `
   -MaintenanceConfigurationId $config.Id
```

## <a name="check-for-pending-updates"></a>Проверить наличие ожидающих обновлений

Чтобы узнать, есть ли ожидающие обновления, используйте [Get-азмаинтенанцеупдате](/powershell/module/az.maintenance/get-azmaintenanceupdate) . Используйте `-subscription` , чтобы указать подписку Azure виртуальной машины, если она отличается от той, в которую вы выполнили вход.

Если обновления для отображения отсутствуют, эта команда возвратит значение Nothing. В противном случае он вернет объект Псапплюпдате:

```json
{
   "maintenanceScope": "Host",
   "impactType": "Freeze",
   "status": "Pending",
   "impactDurationInSec": 9,
   "notBefore": "2020-02-21T16:47:44.8728029Z",
   "properties": {
      "resourceId": "/subscriptions/39c6cced-4d6c-4dd5-af86-57499cd3f846/resourcegroups/Ignite2019/providers/Microsoft.Compute/virtualMachines/MCDemo3"
} 
```

### <a name="isolated-vm"></a>Изолированная виртуальная машина

Проверьте наличие ожидающих обновлений для изолированной виртуальной машины. В этом примере выходные данные форматируются в виде таблицы для удобства чтения.

```azurepowershell-interactive
Get-AzMaintenanceUpdate `
  -ResourceGroupName myResourceGroup `
  -ResourceName myVM `
  -ResourceType VirtualMachines `
  -ProviderName Microsoft.Compute | Format-Table
```


### <a name="dedicated-host"></a>Выделенный узел

Проверка наличия ожидающих обновлений для выделенного узла. В этом примере выходные данные форматируются в виде таблицы для удобства чтения. Замените значения для ресурсов собственными.

```azurepowershell-interactive
Get-AzMaintenanceUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute | Format-Table
```


## <a name="apply-updates"></a>Применение обновлений

Используйте [New-азапплюпдате](/powershell/module/az.maintenance/new-azapplyupdate) для применения ожидающих обновлений.

### <a name="isolated-vm"></a>Изолированная виртуальная машина

Создайте запрос на применение обновлений к изолированной виртуальной машине.

```azurepowershell-interactive
New-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute
```

При успешном выполнении эта команда вернет `PSApplyUpdate` объект. Для проверки состояния обновления можно использовать атрибут Name в `Get-AzApplyUpdate` команде. См. раздел [Проверка состояния обновления](#check-update-status).

### <a name="dedicated-host"></a>Выделенный узел

Применить обновления к выделенному узлу.

```azurepowershell-interactive
New-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute
```

## <a name="check-update-status"></a>Проверить состояние обновления
Чтобы проверить состояние обновления, используйте команду [Get-азапплюпдате](/powershell/module/az.maintenance/get-azapplyupdate) . Команды, показанные ниже, показывают состояние последнего обновления с помощью `default` для `-ApplyUpdateName` параметра. Можно заменить имя обновления (возвращаемое командой [New-азапплюпдате](/powershell/module/az.maintenance/new-azapplyupdate) ), чтобы получить состояние конкретного обновления.

```text
Status         : Completed
ResourceId     : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso
LastUpdateTime : 1/1/2020 12:00:00 AM
Id             : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso/providers/Microsoft.Maintenance/applyUpdates/default
Name           : default
Type           : Microsoft.Maintenance/applyUpdates
```
Ластупдатетиме будет считать время завершения обновления, инициированное вами или платформой в случае, если окно самостоятельного обслуживания не использовалось. Если обновление не было применено через контроль обслуживания, будет отображаться значение по умолчанию.

### <a name="isolated-vm"></a>Изолированная виртуальная машина

Проверьте наличие обновлений для определенной виртуальной машины.

```azurepowershell-interactive
Get-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute `
   -ApplyUpdateName default
```

### <a name="dedicated-host"></a>Выделенный узел

Проверьте наличие обновлений для выделенного узла.

```azurepowershell-interactive
Get-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute `
   -ApplyUpdateName myUpdateName
```

## <a name="remove-a-maintenance-configuration"></a>Удаление конфигурации обслуживания

Удалите конфигурацию обслуживания с помощью [Remove-азмаинтенанцеконфигуратион](/powershell/module/az.maintenance/remove-azmaintenanceconfiguration) .

```azurepowershell-interactive
Remove-AzMaintenanceConfiguration `
   -ResourceGroupName myResourceGroup `
   -Name $config.Name
```

## <a name="next-steps"></a>Следующие шаги
Дополнительные сведения см. в разделе [обслуживание и обновления](maintenance-and-updates.md).
