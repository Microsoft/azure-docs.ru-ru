---
title: Модуль PowerShell для виртуального рабочего стола Windows — Azure
description: Как установить и настроить модуль PowerShell для виртуального рабочего стола Windows.
author: Heidilohr
ms.topic: how-to
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f2f01e2b58c997db08ad4427de7eef1ee3760c4a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96016817"
---
# <a name="set-up-the-powershell-module-for-windows-virtual-desktop"></a>Настройка модуля PowerShell для виртуального рабочего стола Windows

>[!IMPORTANT]
>Это содержимое относится к виртуальному рабочему столу Windows с Azure Resource Managerной интеграцией.

Модуль PowerShell для виртуального рабочего стола Windows интегрирован в модуль Azure PowerShell. В этой статье вы узнаете, как настроить модуль PowerShell таким образом, чтобы можно было выполнять командлеты для виртуального рабочего стола Windows.

## <a name="set-up-your-powershell-environment"></a>Настройка среды PowerShell

Прежде чем приступать к работе с модулем, установите [последнюю версию PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core). Командлеты для виртуального рабочего стола Windows пока работают только с PowerShell Core.

После этого установите модуль виртуализации рабочего стола для использования в сеансе PowerShell.

Для установки этого модуля выполните следующий командлет PowerShell в режиме с повышенными правами:

```powershell
Install-Module -Name Az.DesktopVirtualization
```

>[!NOTE]
> Если этот командлет не работает, попробуйте выполнить его еще раз с повышенными разрешениями.

Затем выполните следующий командлет для подключения к Azure:

```powershell
Connect-AzAccount
```

>[!IMPORTANT]
>Если вы подключаетесь к US Gov порталу, выполните этот командлет.
> 
> ```powershell
> Connect-AzAccount -EnvironmentName AzureUSGovernment
> ```

Для входа в учетную запись Azure требуется код, который создается при выполнении командлета Connect. Чтобы войти, перейдите в <https://microsoft.com/devicelogin>, введите код, а затем выполните вход, указав учетные данные администратора Azure.

```powershell
Account SubscriptionName TenantId Environment

------- ---------------- -------- -----------

Youradminupn subscriptionname AzureADTenantID AzureCloud
```

Вход будет выполнен в ту подписку, которая задана для ваших учетных данных администратора по умолчанию.

## <a name="change-the-default-subscription"></a>Изменение подписки по умолчанию

Если вы хотите изменить подписку по умолчанию после того, как выполнили вход, выполните следующий командлет:

```powershell
Select-AzSubscription -Subscription <preferredsubscriptionname>
```

Можно также выбрать один из списка с помощью командлета Out-GridView:

```powershell
Get-AzSubscription | Out-GridView -PassThru | Select-AzSubscription
```

При выборе новой подписки указывать идентификатор этой подписки в запускаемых после этого командлетах не нужно. Например, следующий командлет извлекает указанный узел сеанса, не требуя идентификатора подписки:

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname>
```

Можно также изменить подписки для каждого командлета, добавив нужное имя подписки в качестве параметра. Следующий командлет аналогичен предыдущему, но использует другую подписку, идентификатор которой добавлен к нему как параметр.

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname> -SubscriptionId <subscriptionGUID>
```

## <a name="get-locations"></a>Получение расположений

Параметр Location является обязательным для всех командлетов **New-AzWVD**, которые создают объекты.

Выполните следующий командлет, чтобы получить список расположений, поддерживаемых вашей подпиской:

```powershell
Get-AzLocation
```

Выходные данные командлета **Get-AzLocation** будут выглядеть следующим образом:

```powershell
Location : eastasia

DisplayName : East Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : southeastasia

DisplayName : Southeast Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : centralus

DisplayName : Central US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}

Location : eastus

DisplayName : East US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}
```

Узнав расположение своей учетной записи, вы сможете использовать его в командлете. Например, следующий командлет создает пул узлов в расположении southeastasia:

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -Location “southeastasia”
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили модуль PowerShell, командлеты можно использовать для выполнения всех видов действий в виртуальном рабочем столе Windows. Вот некоторые места, где можно использовать модуль:

- Сведения о настройке собственной среды виртуального рабочего стола Windows см. в наших [руководствах по виртуальному рабочему столу Windows]().
- [Create a host pool with PowerShell](create-host-pools-powershell.md) (Создание пула узлов с помощью PowerShell)
- [Настройка метода балансировки нагрузки виртуального рабочего стола Windows](configure-host-pool-load-balancing.md)
- [Настройка типа назначения пула узлов личного рабочего стола](configure-host-pool-personal-desktop-assignment-type.md)
- И многие другие!

Если у вас возникнут проблемы, обращайтесь за помощью к статье [Устранение неполадок в PowerShell](troubleshoot-powershell.md).

