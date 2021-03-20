---
title: Создание и Настройка плана защиты Azure от атак DDoS с помощью Azure PowerShell
description: Узнайте, как создать план защиты от атак DDoS с помощью Azure PowerShell
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.openlocfilehash: 69f9b5a74566879ecf8f15f23e689ebb731da45a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97814148"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-powershell"></a>Краткое руководство. Создание и Настройка стандарта защиты от атак DDoS Azure с помощью Azure PowerShell

Приступая к работе со стандартом защиты Azure от атак DDoS с помощью Azure PowerShell. 

План защиты от атак DDoS определяет набор виртуальных сетей, в которых включен стандарт защиты от атак DDoS, для подписок. Вы можете настроить один план защиты от атак DDoS для своей организации и привязать к нему виртуальные сети из нескольких подписок. 

В этом кратком руководстве вы создадите план защиты от атак DDoS и свяжете его с виртуальной сетью. 

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-ddos-protection-plan"></a>Создание плана защиты от атак DDoS

В Azure выделите связанные ресурсы группе ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Чтобы создать группу ресурсов, используйте [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). В этом примере мы назовем нашу группу ресурсов _MyResourceGroup_ и используем расположение _восточной части США_ :

```azurepowershell-interactive
New-AzResourceGroup -Name MyResourceGroup -Location "East US"
```

Теперь создайте план защиты от атак DDoS с именем _миддоспротектионплан_:

```azurepowershell-interactive
New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"
```

## <a name="enable-ddos-for-a-virtual-network"></a>Включение от атак DDoS для виртуальной сети

### <a name="enable-ddos-for-a-new-virtual-network"></a>Включение защиты от атак DDoS для новой виртуальной сети

Вы можете включить защиту от атак DDoS при создании виртуальной сети. В этом примере мы назовем нашу виртуальную сеть _MyVnet_: 

```azurepowershell-interactive
New-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup -Location "East US" -AddressPrefix 10.0.0.0/16
```

### <a name="enable-ddos-for-an-existing-virtual-network"></a>Включение защиты от атак DDoS для существующей виртуальной сети

Вы можете связать существующую виртуальную сеть при создании плана защиты от атак DDoS:

```azurepowershell-interactive
# Creates the DDoS protection plan
$ddosProtectionPlan = New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"

# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = New-Object Microsoft.Azure.Commands.Network.Models.PSResourceId

# Update the properties and enable DDoS protection
$vnet.DdosProtectionPlan.Id = $ddosProtectionPlan.Id
$vnet.EnableDdosProtection = $true
$vnet | Set-AzVirtualNetwork
``` 

## <a name="validate-and-test"></a>Проверка и тестирование

Сначала проверьте сведения о плане защиты от атак DDoS:

```azurepowershell-interactive
Get-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan
```

Убедитесь, что команда возвращает правильные сведения о плане защиты от атак DDoS.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете разместить ресурсы для следующего руководства. Удалите группу ресурсов _MyResourceGroup_ , если она больше не нужна. При удалении группы ресурсов также удаляется план защиты от атак DDoS и все связанные с ним ресурсы. 

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyResourceGroup
```

Чтобы отключить защиту от атак DDoS для виртуальной сети, выполните следующие действия. 

```azurepowershell-interactive
# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = $null
$vnet.EnableDdosProtection = $false
$vnet | Set-AzVirtualNetwork
```

Если вы хотите удалить план защиты от атак DDoS, сначала необходимо отменить связь со всеми виртуальными сетями.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как просмотреть и настроить данные телеметрии для плана защиты от атак DDoS, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Настройка телеметрии защиты от атак DDoS и просмотр данных телеметрии](telemetry.md)