---
title: Развертывание Azure Monitor для решений SAP с помощью Azure PowerShell
description: Развертывание Azure Monitor для решений SAP с помощью Azure PowerShell
author: sameeksha91
ms.author: sakhare
ms.date: 09/08/2020
ms.topic: quickstart
ms.service: virtual-machines-sap
ms.devlang: azurepowershell
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 0af2611bada7a9aad206ce7463ef72ec930c89a2
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538687"
---
# <a name="quickstart-deploy-azure-monitor-for-sap-solutions-with-azure-powershell"></a>Краткое руководство. Развертывание Azure Monitor для решений SAP с помощью Azure PowerShell

В этой статье описывается создание Azure Monitor для ресурсов решений SAP с помощью модуля [Az.HanaOnAzure](/powershell/module/az.hanaonazure/#sap-hana-on-azure) в PowerShell.

> [!CAUTION]
> Azure Monitor для решений SAP в настоящее время находится на этапе общедоступной предварительной версии. Эта предварительная версия предоставляется без соглашения об уровне обслуживания. Эта версия не рекомендуется для использования с рабочими нагрузками в производственной среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если вы решили использовать Cloud Shell, ознакомьтесь с дополнительными сведениями в [этой статье](../../../cloud-shell/overview.md).

> [!IMPORTANT]
> Так как модуль **Az.HanaOnAzure** в PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно с помощью командлета `Install-Module`. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.HanaOnAzure
```

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../../../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов с указанным именем в выбранном регионе.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location westus2
```

## <a name="sap-monitor"></a>Монитор SAP

Чтобы создать монитор SAP, используйте командлет [New-AzSapMonitor](/powershell/module/az.hanaonazure/new-azsapmonitor). В следующем примере создается монитор SAP для указанной подписки, группы ресурсов и имени ресурса.

```azurepowershell-interactive
$Workspace = New-AzOperationalInsightsWorkspace -ResourceGroupName myResourceGroup -Name sapmonitor-test -Location westus2 -Sku Standard

$WorkspaceKey = Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName myResourceGroup -Name sapmonitor-test

$SapMonitorParams = @{
  Name = 'ps-sapmonitor-t01'
  ResourceGroupName = 'myResourceGroup'
  Location = 'westus2'
  EnableCustomerAnalytic = $true
  MonitorSubnet = '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/vnet-sap/subnets/mysubnet'
  LogAnalyticsWorkspaceSharedKey = $WorkspaceKey.PrimarySharedKey
  LogAnalyticsWorkspaceId = $Workspace.CustomerId
  LogAnalyticsWorkspaceResourceId = $Workspace.ResourceId
}
New-AzSapMonitor @SapMonitorParams
```

Чтобы получить свойства монитора SAP, используйте командлет [Get-AzSapMonitor](/powershell/module/az.hanaonazure/get-azsapmonitor). В следующем примере извлекаются свойства монитора SAP для указанной подписки, группы ресурсов и имени ресурса.

```azurepowershell-interactive
Get-AzSapMonitor -ResourceGroupName myResourceGroup -Name ps-spamonitor-t01
```

## <a name="provider-instance"></a>Экземпляр поставщика

Чтобы создать экземпляр поставщика, используйте командлет [New-AzSapMonitorProviderInstance](/powershell/module/az.hanaonazure/new-azsapmonitorproviderinstance). В следующем примере создается экземпляр поставщика для указанной подписки, группы ресурсов и имени ресурса.

```azurepowershell-interactive
$SapProviderParams = @{
  ResourceGroupName = 'myResourceGroup'
  Name = 'ps-sapmonitorins-t01'
  SapMonitorName = 'yemingmonitor'
  ProviderType = 'SapHana'
  HanaHostname = 'hdb1-0'
  HanaDatabaseName = 'SYSTEMDB'
  HanaDatabaseSqlPort = '30015'
  HanaDatabaseUsername = 'SYSTEM'
  HanaDatabasePassword = (ConvertTo-SecureString 'Manager1' -AsPlainText -Force)
}
New-AzSapMonitorProviderInstance @SapProviderParams
```

Чтобы извлечь свойства экземпляра поставщика, используйте командлет [Get-AzSapMonitorProviderInstance](/powershell/module/az.hanaonazure/get-azsapmonitorproviderinstance). В следующем примере показано извлечение свойств экземпляра поставщика для указанной подписки, группы ресурсов, имени SapMonitor и имени ресурса.

```azurepowershell-interactive
Get-AzSapMonitorProviderInstance -ResourceGroupName myResourceGroup -SapMonitorName ps-spamonitor-t01
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные при работе с этой статьей ресурсы больше не нужны, их можно удалить с помощью команды из следующего примера.

### <a name="delete-the-provider-instance"></a>Удаление экземпляра поставщика

Чтобы удалить экземпляр поставщика, используйте командлет [Remove-AzSapMonitorProviderInstance](/powershell/module/az.hanaonazure/remove-azsapmonitorproviderinstance). В следующем примере удаляется экземпляр поставщика для указанной подписки, группы ресурсов, имени SapMonitor и имени ресурса.

```azurepowershell-interactive
Remove-AzSapMonitorProviderInstance -ResourceGroupName myResourceGroup -SapMonitorName ps-spamonitor-t01 -Name ps-sapmonitorins-t02
```

### <a name="delete-the-sap-monitor"></a>Удаление монитора SAP

Чтобы удалить монитор SAP, используйте командлет [Remove-AzSapMonitor](/powershell/module/az.hanaonazure/remove-azsapmonitor). В следующем примере удаляется монитор SAP для указанной подписки, группы ресурсов и имени монитора.

```azurepowershell
Remove-AzSapMonitor -ResourceGroupName myResourceGroup -Name ps-sapmonitor-t02
```

### <a name="delete-the-resource-group"></a>удаление группы ресурсов.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об [Azure Monitor для решений SAP](azure-monitor-overview.md).
