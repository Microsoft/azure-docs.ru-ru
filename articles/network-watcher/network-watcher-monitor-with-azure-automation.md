---
title: Устранение неполадок и наблюдение за VPN-шлюзами — служба автоматизации Azure
titleSuffix: Azure Network Watcher
description: В этой статье описывается, как диагностировать локальное подключение с помощью службы автоматизации Azure и Наблюдателя за сетями.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: af671996722524de9af1a90ae8dfde27f814c8c2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96011818"
---
# <a name="monitor-vpn-gateways-with-network-watcher-troubleshooting"></a>Мониторинг VPN-шлюзов с помощью средства устранения неполадок Наблюдателя за сетями

Чтобы предоставлять клиентам надежное обслуживание, вы должны знать все о производительности своей сети. Поэтому важно быстро обнаруживать аварийные состояния сети и принимать необходимые меры, чтобы свести к минимуму последствия этих состояний. Служба автоматизации Azure позволяет реализовать и выполнить эту задачу программными средствами с помощью модулей runbook. С помощью службы автоматизации Azure вы можете выполнять упреждающий непрерывный мониторинг сети и создавать оповещения.

## <a name="scenario"></a>Сценарий

На следующем изображении показано многоуровневое приложение, локальное подключение к которому обеспечивают VPN-шлюз и туннель. Для производительности приложений крайне важно обеспечить включение и работу VPN-шлюза.

Модуль runbook создается со скриптом для проверки состояния подключения VPN-туннеля с помощью API устранения неполадок в ресурсах. Если туннель не работает, администраторам электронной почтой отправляется триггер.

![Пример сценария][scenario]

Вы узнаете:

- Как создать модуль runbook, вызывающий командлет `Start-AzureRmNetworkWatcherResourceTroubleshooting` для устранения неполадок подключения.
- Подключение расписания к модулю runbook

## <a name="before-you-begin"></a>Перед началом

Прежде чем приступить к работе с этим сценарием, следует подготовить такие необходимые компоненты:

- Учетная запись службы автоматизации Azure. Убедитесь, что учетная запись службы автоматизации содержит последние версии модулей, а также модуль AzureRM.Network. Чтобы добавить модуль AzureRM.Network в учетную запись службы автоматизации Модуль, см. коллекцию модулей.
- Набор учетных данных, настроенных в службе автоматизации Azure. См. дополнительные сведения о [безопасности службы автоматизации Azure](../automation/automation-security-overview.md).
- Допустимый SMTP-сервер (Microsoft 365, ваш локальный адрес электронной почты или другой) и учетные данные, определенные в службе автоматизации Azure.
- Настроенный шлюз виртуальной сети в Azure.
- Существующая учетная запись хранения с существующим контейнером для хранения журналов.

> [!NOTE]
> Инфраструктура, показанная на предыдущем рисунке, предназначена для иллюстрации. Она не создается с помощью выполнения инструкций из этой статьи.

### <a name="create-the-runbook"></a>Создание модуля runbook

Чтобы настроить пример, сначала нужно создать модуль runbook. В этом примере используется учетная запись запуска от имени. Сведения об учетных записях запуска от имени см. в разделе [Создание учетной записи службы автоматизации на портале Azure](../automation/manage-runas-account.md).

### <a name="step-1"></a>Шаг 1

Перейдите к службе автоматизации Azure на [портале Azure](https://portal.azure.com) и нажмите кнопку **Модули Runbook**.

![Обзор учетной записи службы автоматизации][1]

### <a name="step-2"></a>Шаг 2

Нажмите кнопку **Добавить Runbook**, чтобы начать создание модуля Runbook.

![Колонка модулей runbook][2]

### <a name="step-3"></a>Шаг 3

В разделе **Быстрое создание**, нажмите кнопку **Создать новый Runbook**, чтобы создать модуль runbook.

![Колонка добавления модуля runbook][3]

### <a name="step-4"></a>Шаг 4

На этом этапе присвойте модулю runbook имя. В этом примере он называется **Get-VPNGatewayStatus**. Необходимо указать описательное имя модуля runbook. Кроме того, рекомендуется присвоить ему имя, которое соответствует стандартам именования PowerShell. Тип модуля runbook в этом примере — **PowerShell**. Другие варианты: графический, рабочий процесс PowerShell и графический рабочий процесс PowerShell.

![Колонка модуля runbook][4]

### <a name="step-5"></a>Шаг 5

На этом шаге создается модуль runbook. В примере ниже содержится весь необходимый код. Элементы в коде, которые содержат, \<value\> должны быть заменены значениями из подписки.

Используйте следующий код и нажмите кнопку **Сохранить**.

```powershell
# Set these variables to the proper values for your environment
$automationCredential = "<work or school account>"
$fromEmail = "<from email address>"
$toEmail = "<to email address>"
$smtpServer = "<smtp.office365.com>"
$smtpPort = 587
$runAsConnectionName = "<AzureRunAsConnection>"
$subscriptionId = "<subscription id>"
$region = "<Azure region>"
$vpnConnectionName = "<vpn connection name>"
$vpnConnectionResourceGroup = "<resource group name>"
$storageAccountName = "<storage account name>"
$storageAccountResourceGroup = "<resource group name>"
$storageAccountContainer = "<container name>"

# Get credentials for work or school account
$cred = Get-AutomationPSCredential -Name $automationCredential

# Get the connection "AzureRunAsConnection "
$servicePrincipalConnection=Get-AutomationConnection -Name $runAsConnectionName

"Logging in to Azure..."
Connect-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $servicePrincipalConnection.TenantId `
    -ApplicationId $servicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint
"Setting context to a specific subscription"
Set-AzureRmContext -SubscriptionId $subscriptionId

$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $region }
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name $vpnConnectionName -ResourceGroupName $vpnConnectionResourceGroup
$sa = Get-AzureRmStorageAccount -Name $storageAccountName -ResourceGroupName $storageAccountResourceGroup 
$storagePath = "$($sa.PrimaryEndpoints.Blob)$($storageAccountContainer)"
$result = Start-AzureRmNetworkWatcherResourceTroubleshooting -NetworkWatcher $networkWatcher -TargetResourceId $connection.Id -StorageId $sa.Id -StoragePath $storagePath

if($result.code -ne "Healthy")
    {
        $body = "Connection for $($connection.name) is: $($result.code) `n$($result.results[0].summary) `nView the logs at $($storagePath) to learn more."
        Write-Output $body
        $subject = "$($connection.name) Status"
        Send-MailMessage `
        -To $toEmail `
        -Subject $subject `
        -Body $body `
        -UseSsl `
        -Port $smtpPort `
        -SmtpServer $smtpServer `
        -From $fromEmail `
        -BodyAsHtml `
        -Credential $cred
    }
else
    {
    Write-Output ("Connection Status is: $($result.code)")
    }
```

### <a name="step-6"></a>Шаг 6

После сохранения модуля runbook его необходимо подключить к расписанию, чтобы автоматизировать запуск модуля. Чтобы начать, нажмите кнопку **Расписание**.

![Шаг 6][6]

## <a name="link-a-schedule-to-the-runbook"></a>Подключение расписания к модулю runbook

Необходимо создать новое расписание. Щелкните **Связать расписание с модулем runbook**.

![Шаг 7][7]

### <a name="step-1"></a>Шаг 1

В колонке **Расписание** щелкните **Создать новое расписание**.

![Шаг 8][8]

### <a name="step-2"></a>Шаг 2

В колонке **Новое расписание** заполните сведения о расписании. Вы можете указать значения, перечисленные ниже.

- **Имя** — понятное имя расписания.
- **Описание** — описание расписания.
- **Начало** — это значение включает дату, время и часовой пояс, которые формируют время активации расписания.
- **Повторение** — это значение определяет повторение расписания.  Допустимые значения: **Однократно** или **Периодически**.
- **Повторять каждые** — интервал повторения расписания в часах, днях, неделях или месяцах.
- **Установить срок действия** — значение определяет, есть ли у расписания срок действия. Возможные варианты: **Да** или **Нет**. Если выбран вариант "Да", необходимо указать допустимое значение даты и времени.

> [!NOTE]
> Если модуль runbook необходимо запускать чаще, чем каждый час, необходимо создать несколько расписаний с разными интервалами (то есть 15, 30, 45 минут после часа)

![Шаг 9.][9]

### <a name="step-3"></a>Шаг 3

Нажмите кнопку "Сохранить", чтобы сохранить расписание для модуля runbook.

![Шаг 10][10]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как интегрировать средство устранения неполадок Наблюдателя за сетями со службой автоматизации Azure, научитесь активировать захват пакетов на основе предупреждений виртуальной машины. Инструкции см. в статье о том, как [с помощью Наблюдателя за сетями Azure создать захват пакетов, который активируется оповещениями](network-watcher-alert-triggered-packet-capture.md).

<!-- images -->
[scenario]: ./media/network-watcher-monitor-with-azure-automation/scenario.png
[1]: ./media/network-watcher-monitor-with-azure-automation/figure1.png
[2]: ./media/network-watcher-monitor-with-azure-automation/figure2.png
[3]: ./media/network-watcher-monitor-with-azure-automation/figure3.png
[4]: ./media/network-watcher-monitor-with-azure-automation/figure4.png
[5]: ./media/network-watcher-monitor-with-azure-automation/figure5.png
[6]: ./media/network-watcher-monitor-with-azure-automation/figure6.png
[7]: ./media/network-watcher-monitor-with-azure-automation/figure7.png
[8]: ./media/network-watcher-monitor-with-azure-automation/figure8.png
[9]: ./media/network-watcher-monitor-with-azure-automation/figure9.png
[10]: ./media/network-watcher-monitor-with-azure-automation/figure10.png