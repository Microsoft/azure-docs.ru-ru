---
title: Получение уведомлений об обслуживании для виртуальных машин Azure с помощью PowerShell
description: Просмотр уведомлений об обслуживании для виртуальных машин, работающих в Azure, и запуск самостоятельного обслуживания с помощью PowerShell.
author: shants123
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 4278aa563d76b783e02e67ee964764864931fd68
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552377"
---
# <a name="handling-planned-maintenance-using-powershell"></a>Обработка планового обслуживания с помощью PowerShell

**Эта статья относится к виртуальным машинам под управлением Linux и Windows.**

Вы можете использовать Azure PowerShell, чтобы узнать, когда планируется [обслуживание](maintenance-notifications.md)виртуальных машин. Информацию о плановом обслуживании можно получить с помощью командлета [Get-AzVM](/powershell/module/az.compute/get-azvm), используя параметр `-status`.
  
Сведения об обслуживании возвращаются, только если имеется запланированное обслуживание. Если нет запланированного обслуживания, влияющего на виртуальную машину, командлет не возвращает информацию об обслуживании. 


```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Name myVM -Status
```

Выходные данные

```
MaintenanceRedeployStatus               : 
  IsCustomerInitiatedMaintenanceAllowed : True
  PreMaintenanceWindowStartTime         : 5/14/2018 12:30:00 PM
  PreMaintenanceWindowEndTime           : 5/19/2018 12:30:00 PM
  MaintenanceWindowStartTime            : 5/21/2018 4:30:00 PM
  MaintenanceWindowEndTime              : 6/4/2018 4:30
  LastOperationResultCode               : None 
```

В разделе MaintenanceRedeployStatus возвращаются следующие свойства: 

| Значение | Описание   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Указывает, можно ли сейчас запустить обслуживание на виртуальной машине |
| PreMaintenanceWindowStartTime         | Начало периода самообслуживания, когда можно инициировать обслуживание на виртуальной машине |
| PreMaintenanceWindowEndTime           | Завершение периода самообслуживания, когда можно инициировать обслуживание на виртуальной машине |
| MaintenanceWindowStartTime            | Начало периода запланированного обслуживания, в течение которого Azure запускает обслуживание для виртуальной машины. |
| MaintenanceWindowEndTime              | Завершение периода запланированного обслуживания, в течение которого Azure запускает обслуживание для виртуальной машины. |
| LastOperationResultCode               | Результат последней попытки инициирования обслуживания на виртуальной машине |



Вы также можете получить информацию о состоянии обслуживания всех виртуальных машин в группе ресурсов с помощью командлета [Get-AzVM](/powershell/module/az.compute/get-azvm), не указывая виртуальную машину.
 
```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Status
```

Следующий пример PowerShell принимает идентификатор подписки и возвращает список виртуальных машин, для которых запланировано обслуживание.

```powershell

function MaintenanceIterator
{
    Select-AzSubscription -SubscriptionId $args[0]

    $rgList= Get-AzResourceGroup 

    for ($rgIdx=0; $rgIdx -lt $rgList.Length ; $rgIdx++)
    {
        $rg = $rgList[$rgIdx]        
    $vmList = Get-AzVM -ResourceGroupName $rg.ResourceGroupName 
        for ($vmIdx=0; $vmIdx -lt $vmList.Length ; $vmIdx++)
        {
            $vm = $vmList[$vmIdx]
            $vmDetails = Get-AzVM -ResourceGroupName $rg.ResourceGroupName -Name $vm.Name -Status
              if ($vmDetails.MaintenanceRedeployStatus )
            {
                Write-Output "VM: $($vmDetails.Name)  IsCustomerInitiatedMaintenanceAllowed: $($vmDetails.MaintenanceRedeployStatus.IsCustomerInitiatedMaintenanceAllowed) $($vmDetails.MaintenanceRedeployStatus.LastOperationMessage)"               
            }
          }
    }
}

```

### <a name="start-maintenance-on-your-vm-using-powershell"></a>Запуск обслуживания вашей виртуальной машины с помощью PowerShell

Используя информацию из функции в предыдущем разделе, следующая команда запускает обслуживание на виртуальной машине, если для свойства **IsCustomerInitiatedMaintenanceAllowed** задано значение true.

```powershell
Restart-AzVM -PerformMaintenance -name $vm.Name -ResourceGroupName $rg.ResourceGroupName 
```

## <a name="classic-deployments"></a>Классические развертывания

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Если у вас все еще есть устаревшие виртуальные машины, развернутые с использованием классической модели развертывания, вы можете сделать запрос к ним и запустить их обслуживание с помощью PowerShell.

Чтобы узнать состояние обслуживания виртуальной машины, введите:

```
Get-AzureVM -ServiceName <Service name> -Name <VM name>
```

Чтобы начать обслуживание классической виртуальной машины, введите:

```
Restart-AzureVM -InitiateMaintenance -ServiceName <service name> -Name <VM name>
```

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете управлять плановым обслуживанием с помощью [Azure CLI](maintenance-notifications-cli.md) или [портала](maintenance-notifications-portal.md).
