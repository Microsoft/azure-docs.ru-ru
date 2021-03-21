---
title: 'VPN-шлюз Azure: Настройка оповещений для событий журнала диагностики'
description: Узнайте, как настроить оповещения на основе событий журнала ресурсов из VPN-шлюза Azure с помощью Azure Monitor Log Analytics.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/18/2020
ms.author: alzam
ms.openlocfilehash: 70cac6ef566ef52409cd9667708c2fc297e046f7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656620"
---
# <a name="set-up-alerts-on-resource-log-events-from-vpn-gateway"></a>Настройка оповещений о событиях журнала ресурсов из VPN-шлюза

Эта статья поможет вам настроить оповещения на основе событий журнала ресурсов из VPN-шлюза Azure с помощью Azure Monitor Log Analytics. 

В Azure доступны следующие журналы ресурсов:

|***Имя** _ | _ *_Описание_** |
|--- | --- |
|GatewayDiagnosticLog | Содержит журналы ресурсов для событий настройки шлюза, основные изменения и события обслуживания. |
|TunnelDiagnosticLog | Содержит события изменения состояния туннеля. События подключения и отключения туннеля имеют обобщенную причину изменения состояния, если применимо |
|RouteDiagnosticLog | Регистрирует изменения статических маршрутов и событий BGP, происходящих в шлюзе. |
|IKEDiagnosticLog | Регистрирует сообщения управления IKE и события в шлюзе |
|P2SDiagnosticLog | Регистрирует сообщения управления точка-сеть и события на шлюзе. Сведения об источнике подключения предоставляются только для подключений по протоколу IKEv2 и Опенвпн |

## <a name="set-up-alerts-in-the-azure-portal"></a><a name="setup"></a>Настройка оповещений на портале Azure

В следующем примере показано, как создать предупреждение для события отключения, включающего туннель VPN типа "сеть — сеть".


1. В портал Azure найдите **log Analytics** в разделе **все службы** и выберите **log Analytics рабочие области**.

   ![Параметры для перехода в Log Analytics рабочие области](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert0.png "Создать")

2. На странице **log Analytics** выберите **создать** .

   ![Страница Log Analytics с кнопкой "создать"](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert1.png  "Выберите пункт")

3. Выберите **создать** и введите сведения.

   ![Сведения о создании рабочей области Log Analytics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert2.png  "Выберите пункт")

4. Найдите VPN-шлюз в колонке **мониторинг**  >  **параметров диагностики** .

   ![Варианты поиска VPN-шлюза в параметрах диагностики](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert3.png  "Выберите пункт")

5. Чтобы включить диагностику, дважды щелкните шлюз, а затем выберите **включить диагностику**.

   ![Параметры для включения диагностики](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert4.png  "Выберите пункт")

6. Введите сведения и убедитесь, что выбран параметр **Send to log Analytics** и **туннелдиагностиклог** . Выберите рабочую область Log Analytics, созданную на шаге 3.

   ![Выбранные флажки](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert5.png  "Выберите пункт")

   > [!NOTE]
   > Первоначальное отображение данных может занять несколько часов.

7. Перейдите к обзору ресурса шлюза виртуальной сети и выберите **оповещения** на вкладке **мониторинг** . Затем создайте новое правило генерации оповещений или измените существующее правило генерации оповещений.

   ![Варианты создания нового правила генерации оповещений](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Выберите пункт")

   ![точка — сеть](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Выберите пункт")
8. Выберите рабочую область Log Analytics и ресурс.

   ![Параметры для рабочей области и ресурса](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert7.png  "Выберите пункт")

9. Выберите **Пользовательский поиск по журналу** в качестве логики сигнала в разделе **Добавить условие**.

   ![Элементы, выделенные для пользовательского поиска по журналам](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert8.png  "Выберите пункт")

10. Введите следующий запрос в текстовом поле **поисковый запрос** . Замените значения в <> и TimeGenerated соответствующим образом.

    ```
    AzureDiagnostics
    | where Category == "TunnelDiagnosticLog"
    | where _ResourceId == tolower("<RESOURCEID OF GATEWAY>")
    | where TimeGenerated > ago(5m) 
    | where remoteIP_s == "<REMOTE IP OF TUNNEL>"
    | where status_s == "Disconnected"
    | project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId 
    | sort by TimeGenerated asc
    ```

    Установите пороговое значение 0 и нажмите кнопку **Готово**.

    ![Ввод запроса и выбор порогового значения](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert9.png  "Выберите пункт")

11. На странице **Создание правила** выберите **создать** в разделе **группы действий** . Введите сведения и нажмите кнопку **ОК**.

    ![Сведения о новой группе действий](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert10.png  "Выберите пункт")

12. На странице **Создание правила** введите сведения о **действиях настройки** и убедитесь, что в разделе **имя группы действий** отображается правильное имя. Выберите **создать правило генерации оповещений** , чтобы создать правило.

    ![Параметры для создания правила](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert11.png  "Выберите пункт")

## <a name="set-up-alerts-by-using-powershell"></a><a name="setuppowershell"></a>Настройка оповещений с помощью PowerShell

В следующих примерах шагов создается оповещение о событии отключения, включающем туннель VPN типа "сеть — сеть".

1. Создайте рабочую область Log Analytics:

   ```powershell
   $Location           = 'westus2'
   $ResourceGroupName  = 'TestRG1'
   $Sku                = 'pergb2018'
   $WorkspaceName      = 'LogAnalyticsWS123'

   New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku $Sku -ResourceGroupName $ResourceGroupName
   ```

2. Включите диагностику для VPN-шлюза:

   ```powershell
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   Set-AzDiagnosticSetting `
       -Name 'VPN tunnel' `
       -ResourceId $VpnGateway.Id `
       -WorkspaceId $Workspace.ResourceId `
       -Enabled $true `
       -Category 'TunnelDiagnosticLog'
   ```

3. Создайте группу действий.

   Этот код создает группу действий, которая отправляет уведомление по электронной почте при активации оповещения:

   ```powershell
   $ActionGroupName            = 'EmailAdmins'   # Max. 60 characters long
   $ActionGroupShortName       = 'EmailAdmins'   # Max. 12 characters long
   $ActionGroupReceiverName    = 'My receiver Name'
   $EmailAddress               = 'xyz@contoso.com'
   $ResourceGroupName          = 'TestRG1'

   $ActionGroupReceiver = New-AzActionGroupReceiver -Name $ActionGroupReceiverName -UseCommonAlertSchema -EmailReceiver -EmailAddress $EmailAddress

   Set-AzActionGroup `
      -ResourceGroupName $ResourceGroupName `
      -Name $ActionGroupName `
      -ShortName $ActionGroupShortName `
      -Receiver @($ActionGroupReceiver)
   ```

4. Создайте правило генерации оповещений на основе пользовательского поиска по журналам:

   ```powershell
   $ActionGroupName    = 'EmailAdmins'
   $EmailSubject       = 'Redmond VPN tunnel is disconnected'
   $Location           = 'westus2'
   $RemoteIp           = '104.42.209.46'
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   $Query = @"
   AzureDiagnostics |
   where Category == "TunnelDiagnosticLog" |
   where TimeGenerated > ago(5m) |
   where _ResourceId == tolower("$($VpnGateway.id)") |
   where remoteIP_s == "$($RemoteIp)" |
   where status_s == "Disconnected" |
   project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId |
   sort by TimeGenerated asc
   "@

   $Source             = New-AzScheduledQueryRuleSource -Query $Query -DataSourceId $Workspace.ResourceId
   $Schedule           = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 5 -TimeWindowInMinutes 5
   $TriggerCondition   = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator 'GreaterThan' -Threshold 0

   $ActionGroup        = Get-AzActionGroup -ResourceGroupName $ResourceGroupName -Name $ActionGroupName
   $AznsActionGroup    = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup $ActionGroup.Id -EmailSubject $EmailSubject
   $AlertingAction     = New-AzScheduledQueryRuleAlertingAction -AznsAction $AznsActionGroup -Severity '1' -Trigger $TriggerCondition

   New-AzScheduledQueryRule `
       -ResourceGroupName $ResourceGroupName `
       -Location $Location `
       -Action $AlertingAction `
       -Enabled $true `
       -Description 'The tunnel between Azure and Redmond with IP address 104.42.209.46 is disconnected' `
       -Schedule $Schedule `
       -Source $Source `
       -Name 'The Azure to Redmond tunnel is disconnected'
   ```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о настройке оповещений в метриках туннеля см. в разделе [Настройка оповещений в метриках VPN-шлюза](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md).
