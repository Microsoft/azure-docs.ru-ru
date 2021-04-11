---
title: 'Регистрация подключения Службы пиринга (Azure PowerShell) '
description: В этом руководстве описано, как с помощью PowerShell зарегистрировать подключение Службы пиринга.
services: peering-service
author: derekolo
ms.service: peering-service
ms.topic: tutorial
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: ed4f628f6c3b888a640d61048c39f945a81901d7
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106060845"
---
# <a name="tutorial-register-a-peering-service-connection-using-azure-powershell"></a>Руководство по регистрации подключения Службы пиринга с помощью Azure PowerShell

Из этого руководства вы узнаете, как зарегистрировать Службу пиринга с помощью Azure PowerShell.

Служба пиринга Azure — это сетевая служба, расширяющая возможности клиентов по подключению к облачным службам Майкрософт, таким как Microsoft 365, Dynamics 365, SaaS (программное обеспечение как услуга), Azure и любым службам корпорации Майкрософт, которые работают через общедоступный Интернет. Из этой статьи вы узнаете, как зарегистрировать подключение Службы пиринга с помощью Azure PowerShell.

Если у вас еще нет подписки Azure, [создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) прямо сейчас.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, для работы с этим быстрым началом вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Сведения об установке и обновлении см. в статье [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

Наконец, если модуль PowerShell запущен локально, необходимо также выполнить `Connect-AzAccount`. Эта команда создает подключение к Azure.

Для регистрации Службы пиринга и управления ею используйте модуль Azure PowerShell. Вы можете зарегистрировать Службу пиринга или управлять ею с помощью командной строки или скриптов PowerShell.


## <a name="prerequisites"></a>Предварительные требования  
Необходимо следующее:

### <a name="azure-account"></a>Учетная запись Azure

Вам потребуется действительная и активная учетная запись Microsoft Azure. Эта учетная запись необходима для настройки подключения Службы пиринга. Ресурс Службы пиринга размещается в подписках Azure.

### <a name="connectivity-provider"></a>Поставщик услуг подключения

Вы можете обратиться к поставщику услуг Интернета или к партнеру по обмену интернет-трафиком, чтобы получить Службу пиринга для подключения собственной сети к сети Майкрософт.

Убедитесь, что поставщики услуг подключения имеют партнерские отношения с корпорацией Майкрософт.

### <a name="register-a-subscription-with-the-resource-provider-and-feature-flag"></a>Регистрация подписки с использованием поставщика ресурсов и флага компонента

Прежде чем перейти к регистрации Службы пиринга, зарегистрируйте подписку с помощью Azure PowerShell, используя поставщик ресурсов и флаг компонента. Необходимые команды Azure PowerShell указаны ниже.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName AllowPeeringService ProviderNamespace Microsoft.Peering 

Register-AzResourceProvider -ProviderNamespace Microsoft.Peering 

```

### <a name="fetch-the-location-and-service-provider"></a>Получение расположения и поставщика услуг 

Выполните следующие команды в Azure PowerShell, чтобы получить расположение и поставщика услуг, для которых должна быть включена Служба пиринга. 

Получение расположений Службы пиринга:

```azurepowershell-interactive
# Gets a list of available countries
Get-AzPeeringServiceCountry 
# Gets a list of metro locations serviced by country
Get-AzPeeringServiceLocation -Country "United States"
```

Получение поставщиков Службы пиринга:
              
```azurepowershell-interactive
Get-AzPeeringServiceProvider
```

### <a name="register-the-peering-service-connection"></a>Регистрация подключения Службы пиринга

Чтобы зарегистрировать подключение Службы пиринга, выполните следующий набор команд в Azure PowerShell. В этом примере регистрируется Служба пиринга с именем myPeeringService.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
```

### <a name="register-the-peering-service-prefix"></a>Регистрация префикса Службы пиринга

Зарегистрируйте префикс, который предоставлен поставщиком услуг подключения, выполнив следующие команды в Azure PowerShell. В этом примере регистрируется префикс с именем myPrefix.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
$prefixName = "myPrefix"
$prefix = “192.168.1.0/24”
$serviceKey = "6f48cdd6-2c2e-4722-af89-47e27b2513af"
$prefixService = $peeringService | New-AzPeeringServicePrefix -Name $prefixName -Prefix $prefix -ServiceKey $serviceKey
```

### <a name="list-all-the-peering-services-connections"></a>Получение списка всех подключений Служб пиринга

Чтобы просмотреть полный список Служб пиринга, выполните следующую команду:

```azurepowershell-interactive
$peeringService = Get-AzPeeringService
```

### <a name="list-all-the-peering-service-prefixes"></a>Получение списка префиксов Службы пиринга

Чтобы просмотреть список всех префиксов Службы пиринга, выполните следующую команду:

```azurepowershell-interactive
 $prefixName = "myPrefix"
```

```azurepowershell-interactive
$prefix = Get-AzPeeringServicePrefix -PeeringServiceName "myPeeringService" -ResourceGroupName "MyResourceGroup" -Name "myPrefix"
```

### <a name="remove-the-peering-service-prefix"></a>Удаление префикса Службы пиринга

Чтобы удалить префикс Службы пиринга, выполните следующую команду:

```azurepowershell-interactive
Remove-AzPeeringServicePrefix -ResourceGroupName  "MyResourceGroup" -Name "myPrefix" -PeeringServiceName "myPeeringService"
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о подключении Службы пиринга см. в статье [Подключение Службы пиринга](connection.md).
- Дополнительные сведения о телеметрии см. в статье [Телеметрия для подключения Службы пиринга](connection-telemetry.md).
- Регистрация подключения Службы пиринга с помощью портала Azure описана в [этой статье](azure-portal.md).
- Регистрация подключения Службы пиринга с помощью Azure CLI описана в [этой статье](cli.md).
