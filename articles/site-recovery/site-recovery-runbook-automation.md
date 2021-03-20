---
title: Добавление модулей Runbook службы автоматизации Azure в Site Recovery планы восстановления
description: Узнайте, как расширить планы восстановления с помощью службы автоматизации Azure для аварийного восстановления, используя Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: ramamill
ms.openlocfilehash: a141280338632fdad7053cbbe76c8bdf2797443d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89424877"
---
# <a name="add-azure-automation-runbooks-to-recovery-plans"></a>Добавление модулей Runbook службы автоматизации Azure в планы восстановления

В этой статье описывается, как интегрировать модули Runbook службы автоматизации Azure для расширения планов восстановления [Azure Site Recovery](site-recovery-overview.md) . Мы покажем, как автоматизировать основные задачи, в которых в противном случае потребуется вмешательство вручную, и как преобразовать многоэтапное восстановление в действие, выполняемое одним щелчком.

## <a name="recovery-plans"></a>Планы восстановления 

Вы можете использовать планы восстановления при отработки отказа локальных компьютеров или виртуальных машин Azure. Планы восстановления помогают определить систематический процесс восстановления, который определяет, как происходит отработка отказа компьютеров, и как они запускаются и восстанавливаются после перехода на другой ресурс. 

Крупные приложения для восстановления могут быть сложными. Планы восстановления помогают выполнить сортировку, чтобы обеспечить единообразную, повторяемую и автоматизированную процедуру восстановления. Вы можете автоматизировать задачи в плане восстановления с помощью скриптов, а также модулей Runbook службы автоматизации Azure. Типичными примерами могут быть Настройка параметров на виртуальной машине Azure после отработки отказа или перенастройка приложения, которое работает на виртуальной машине.

- [Узнайте подробнее](recovery-plan-overview.md) о планах восстановления.
- Дополнительные [сведения](../automation/automation-runbook-types.md) о модулях Runbook службы автоматизации Azure.



## <a name="runbooks-in-recovery-plans"></a>Модули Runbook в планах восстановления

Вы добавляете учетную запись службы автоматизации Azure и модули Runbook в план восстановления. Модуль Runbook вызывается при выполнении плана восстановления.

- Учетная запись службы автоматизации может находиться в любом регионе Azure. она должна находиться в той же подписке, что и хранилище Site Recovery. 
- Модуль Runbook может выполняться в плане восстановления во время отработки отказа из основного расположения в базу данных-получателя или во время восстановления размещения из дополнительного расположения в базу данных-источник.
- Модули Runbook в плане восстановления последовательно выполняются последовательно, один за другим в заданном порядке.
- Если модули Runbook в плане восстановления настраивают виртуальные машины для запуска в разных группах, план восстановления продолжит работу, только когда Azure сообщит все виртуальные машины как запущенные.
- Планы восстановления продолжают выполняться, даже если сценарий завершается с ошибкой.

### <a name="recovery-plan-context"></a>Контекст плана восстановления

При выполнении скрипта он внедряет в модуль Runbook контекст плана восстановления. Контекст содержит переменные, представленные в таблице.

| **Имя переменной** | **Описание** |
| --- | --- |
| RecoveryPlanName |Имя плана восстановления. Используется в действиях на основе имени. |
| FailoverType |Указывает, является ли он тестовой или рабочей отработкой отказа. 
| FailoverDirection | Указывает, находится ли восстановление в первичном или дополнительном расположении. |
| GroupID |Определяет номер группы в плане восстановления при выполнении плана. |
| VmMap |Массив всех виртуальных машин в группе. |
| Ключ VMMap |Уникальный ключ (GUID) для каждой виртуальной машины. |
| SubscriptionId |Идентификатор подписки Azure, в которой создана виртуальная машина. |
| ResourceGroupName | Имя группы ресурсов, в которой находится виртуальная машина.
| CloudServiceName |Имя облачной службы Azure, в которой создана виртуальная машина. |
| RoleName |Имя виртуальной машины Azure. |
| RecoveryPointId|Метка времени восстановления виртуальной машины. |

>[!Note]
>Значение переменной "FailoverDirection" будет равно "Примаритосекондари" в случае отработки отказа и "Секондаритопримари" в случае восстановления размещения.

Ниже приведен пример того, как выглядит переменная контекста.

```
{"RecoveryPlanName":"hrweb-recovery",
"FailoverType":"Test",
"FailoverDirection":"PrimaryToSecondary",
"GroupId":"1",
"VmMap":{"7a1069c6-c1d6-49c5-8c5d-33bfce8dd183":
    { "SubscriptionId":"7a1111111-c1d6-49c5-8c5d-111ce8dd183",
    "ResourceGroupName":"ContosoRG",
    "CloudServiceName":"pod02hrweb-Chicago-test",
    "RoleName":"Fabrikam-Hrweb-frontend-test",
    "RecoveryPointId":"TimeStamp"}
    }
}
```

Если вы хотите получить доступ ко всем виртуальным машинам в VMMap, можно использовать следующий код:

```
$VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
$vmMap = $RecoveryPlanContext.VmMap
    foreach($VMID in $VMinfo)
    {
        $VM = $vmMap.$VMID                
            if( !(($VM -eq $Null) -Or ($VM.ResourceGroupName -eq $Null) -Or ($VM.RoleName -eq $Null))) {
            #this check is to ensure that we skip when some data is not available else it will fail
    Write-output "Resource group name ", $VM.ResourceGroupName
    Write-output "Rolename " = $VM.RoleName
            }
        }
```


Блог Аман Шарма, посвященный [сбору облаков](http://harvestingclouds.com) , имеет полезный пример [сценария контекста плана восстановления](http://harvestingclouds.com/post/script-sample-azure-automation-runbook-for-asr-recovery-plan/).



## <a name="before-you-start"></a>Перед началом работы

- Если вы еще не знакомы со службой автоматизации Azure, вы можете [зарегистрироваться](https://azure.microsoft.com/services/automation/) и [скачать примеры сценариев](https://azure.microsoft.com/documentation/scripts/).
- Убедитесь, что в учетную запись службы автоматизации добавлены следующие модули.
    - AzureRM.profile
    - AzureRM.Resources
    - AzureRM.Automation
    - AzureRM.Network
    - AzureRM.Compute

    Все эти модули должны иметь совместимые версии. Самый простой способ — всегда использовать последние версии всех модулей.



## <a name="customize-the-recovery-plan"></a>Настройка плана восстановления

1. В хранилище выберите **планы восстановления (Site Recovery)** .
2. Чтобы создать план восстановления, щелкните **+ план восстановления**. [Подробнее](site-recovery-create-recovery-plans.md). Если у вас уже есть план восстановления, выберите его, чтобы открыть.
3. На странице план восстановления нажмите кнопку **настроить**.

    ![Нажмите кнопку "Настроить"](media/site-recovery-runbook-automation-new/custom-rp.png)

2. Нажмите кнопку с многоточием (...) рядом с **группой 1: начать**  >  **Добавление действия после**.
3. В поле **действие при вставке** убедитесь, что выбран параметр **Скрипт** , и укажите имя скрипта (**Hello World**).
4. Укажите учетную запись службы автоматизации и выберите Runbook. Нажмите кнопку **ОК**, чтобы сохранить этот сценарий. Сценарий будет добавлен в группу **Группа 1: последующие действия**.


## <a name="reuse-a-runbook-script"></a>Повторное использование скрипта Runbook

С помощью внешних переменных можно использовать один скрипт Runbook в нескольких планах восстановления. 

- [Переменные службы автоматизации Azure](../automation/shared-resources/variables.md) используются для хранения параметров для запуска плана восстановления.
- Если перед именем переменной вы укажете имя плана восстановления, для каждого плана восстановления будут созданы собственные переменные, которые можно использовать в качестве параметров.
- Эти параметры можно изменять, чтобы управлять работой скрипта, не изменяя его содержимое, однако принцип работы сценария будет изменен.

### <a name="use-a-simple-string-variable-in-a-runbook-script"></a>Использование простых строковых переменных в сценарии Runbook

В этом примере сценарий принимает входные данные группы безопасности сети (NSG) и применяет ее к виртуальным машинам в плане восстановления. 

1. Чтобы сценарий мог определить, какой план восстановления работает, используйте следующий контекст плана восстановления:

    ```
    workflow AddPublicIPAndNSG {
        param (
              [parameter(Mandatory=$false)]
              [Object]$RecoveryPlanContext
        )

        $RPName = $RecoveryPlanContext.RecoveryPlanName
    ```

2. Обратите внимание на имя NSG и группу ресурсов. Эти переменные используются в качестве входных данных для скриптов плана восстановления. 
1. В ресурсах учетной записи службы автоматизации. Создайте переменную для хранения имени NSG. Добавьте префикс в имя переменной с именем плана восстановления.

    ![Создание переменной для имени NSG](media/site-recovery-runbook-automation-new/var1.png)

2. Создайте переменную для хранения имени группы ресурсов для ресурса NSG. Добавьте префикс в имя переменной с именем плана восстановления.

    ![Создание имени группы ресурсов NSG](media/site-recovery-runbook-automation-new/var2.png)


3.  В скрипте используйте этот справочный код для получения значений переменных:

    ```
    $NSGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSG"
    $NSGRGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSGRG"

    $NSGnameVar = Get-AutomationVariable -Name $NSGValue
    $RGnameVar = Get-AutomationVariable -Name $NSGRGValue
    ```

4.  Используйте переменные в модуле Runbook, что позволит применить NSG к сетевому интерфейсу виртуальной машины, для которой выполняется отработка отказа.

    ```
    InlineScript {
    if (($Using:NSGname -ne $Null) -And ($Using:NSGRGname -ne $Null)) {
            $NSG = Get-AzureRmNetworkSecurityGroup -Name $Using:NSGname -ResourceGroupName $Using:NSGRGname
            Write-output $NSG.Id
            #Apply the NSG to a network interface
            #$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
            #Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
            #  -AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $NSG
        }
    }
    ```


Для каждого плана восстановления создайте независимые переменные, чтобы скрипт можно было использовать повторно. Перед именами переменных добавьте имя плана восстановления. 

Чтобы получить полный сценарий для этого сценария, ознакомьтесь с [этим скриптом](https://gallery.technet.microsoft.com/Add-Public-IP-and-NSG-to-a6bb8fee).


### <a name="use-a-complex-variable-to-store-more-information"></a>Использование сложной переменной для хранения дополнительных сведений

В некоторых случаях создание отдельных переменных для каждого плана восстановления может оказаться невозможным. Рассмотрим ситуацию, в которой необходимо, чтобы один сценарий назначил общедоступный IP-адрес на конкретных виртуальных машинах. Этот же подход будет полезен, если нужно применить разные группы безопасности сети для нескольких (но не всех) виртуальных машин. Обратите внимание на следующее.

- Можно создать сценарий, который будет использоваться для любого плана восстановления.
- Каждый план восстановления может иметь разное число виртуальных машин.
- Например, точка восстановления SharePoint имеет два внешних интерфейса, а простые бизнес-приложения — только один.
- В этом сценарии нельзя создавать отдельные переменные для каждого плана восстановления.

В следующем примере мы создадим [сложную переменную](/powershell/module/servicemanagement/azure.service/set-azureautomationvariable) в учетной записи службы автоматизации Azure.

Для этого нужно указать несколько значений с помощью Azure PowerShell.

1. Используя PowerShell, войдите в свою подписку Azure.

    ```
    Connect-AzureRmAccount
    $sub = Get-AzureRmSubscription -Name <SubscriptionName>
    $sub | Select-AzureRmSubscription
    ```

2. Чтобы сохранить параметры, создайте сложную переменную, используя имя плана восстановления:

    ```
    $VMDetails = @{"VMGUID"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"};"VMGUID2"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"}}
        New-AzureRmAutomationVariable -ResourceGroupName <RG of Automation Account> -AutomationAccountName <AA Name> -Name <RecoveryPlanName> -Value $VMDetails -Encrypted $false
    ```

3. В этой сложной переменной параметр **VMDetails** содержит идентификатор защищенной виртуальной машины. Этот идентификатор указан в свойствах виртуальной машины на портале Azure. На снимке экрана ниже показано, что мы создали переменную для хранения сведений о двух виртуальных машинах.

    ![Используйте идентификатор виртуальной машины в качестве идентификатора GUID](media/site-recovery-runbook-automation-new/vmguid.png)

4. Используйте эту переменную в модуле runbook. Если указанный GUID виртуальной машины находится в контексте плана восстановления, примените группу NSG к виртуальной машине.

    ```
    $VMDetailsObj = (Get-AutomationVariable -Name $RecoveryPlanContext.RecoveryPlanName).ToObject([hashtable])
    ```

4. В модуле Runbook выполните циклический перебор виртуальных машин, входящих в контекст плана восстановления, проверяя наличие этих виртуальных машин в **$VMDetailsObj**. Если встречается совпадение, примените группу NSG, используя свойства переменной.

    ```
        $VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
        $vmMap = $RecoveryPlanContext.VmMap

        foreach($VMID in $VMinfo) {
            $VMDetails = $VMDetailsObj[$VMID].ToObject([hashtable]);
            Write-output $VMDetails
            if ($VMDetails -ne $Null) { #If the VM exists in the context, this will not be Null
                $VM = $vmMap.$VMID
                # Access the properties of the variable
                $NSGname = $VMDetails.NSGName
                $NSGRGname = $VMDetails.NSGResourceGroupName

                # Add code to apply the NSG properties to the VM
            }
        }
    ```

Этот же сценарий можно использовать с разными планами восстановления, сохраняя в разных переменных значения параметров, применимых для соответствующих планов восстановления.

## <a name="sample-scripts"></a>Примеры сценариев

Разверните примеры сценариев в учетной записи службы автоматизации с помощью кнопки **Развертывание в Azure**.

[![Развертывание в Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

В этом видеоролике приведен еще один пример. в котором показано, как выполнить восстановление двухуровневого приложения WordPress в Azure.


> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]


## <a name="next-steps"></a>Дальнейшие действия

- Сведения об [учетной записи запуска от имени службы автоматизации Azure](../automation/manage-runas-account.md)
- Ознакомьтесь с [примерами сценариев службы автоматизации Azure](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=User&f%5B0%5D.Value=SC%20Automation%20Product%20Team&f%5B0%5D.Text=SC%20Automation%20Product%20Team).
- Дополнительные [сведения](site-recovery-failover.md) о выполнении отработки отказа.
