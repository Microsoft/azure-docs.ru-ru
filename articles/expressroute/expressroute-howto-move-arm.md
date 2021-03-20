---
title: 'Azure ExpressRoute: перемещение классических цепей в диспетчер ресурсов'
description: В этой статье описывается перемещение классического канала в модель развертывания Resource Manager с помощью PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 460ea446fc6dfc43e81a1a57bbba032a61f3a72d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90532551"
---
# <a name="move-expressroute-circuits-from-classic-to-resource-manager-deployment-model-using-powershell"></a>Перемещение каналов ExpressRoute из классической модели развертывания в модель развертывания с помощью Resource Manager с использованием PowerShell

Чтобы канал ExpressRoute можно было использовать в классической модели развертывания и в модели Resource Manager, необходимо переместить его в модель развертывания Resource Manager. Сведения о перемещении канала с помощью PowerShell содержатся в следующих разделах.

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

* Убедитесь, что на компьютере установлены компоненты Classic и AZ Azure PowerShell modules. Подробнее: [Установка и настройка Azure PowerShell](/powershell/azure/).
* Не забудьте изучить [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [рабочие процессы](expressroute-workflows.md), прежде чем приступать к настройке.
* Просмотрите сведения в статье [Перемещение каналов ExpressRoute из классической модели развертывания в модель развертывания с помощью Resource Manager](expressroute-move.md). Убедитесь, что вам полностью понятны пределы и ограничения.
* Убедитесь, что канал полноценно работает в классической модели развертывания.
* Убедитесь в наличии группы ресурсов, созданной в модели развертывания Resource Manager.

## <a name="move-an-expressroute-circuit"></a>Перемещение канала ExpressRoute

### <a name="step-1-gather-circuit-details-from-the-classic-deployment-model"></a>Шаг 1. Соберите сведения о канале из классической модели развертывания

Войдите в классическую среду Azure и получите ключ службы.

1. Войдите в учетную запись Azure.

   ```powershell
   Add-AzureAccount
   ```

2.  Выберите соответствующую подписку Azure.

   ```powershell
   Select-AzureSubscription "<Enter Subscription Name here>"
   ```

3. Импортируйте модули PowerShell для Azure и ExpressRoute.

   ```powershell
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
   ```

4. Используйте указанный ниже командлет, чтобы получить ключи службы для всех каналов ExpressRoute. После получения ключей скопируйте **ключ службы** канала, который требуется переместить в модель развертывания Resource Manager.

   ```powershell
   Get-AzureDedicatedCircuit
   ```

### <a name="step-2-sign-in-and-create-a-resource-group"></a>Шаг 2. Вход и создание группы ресурсов

Войдите в среду Resource Manager и создайте группу ресурсов.

1. Войдите в среду Azure Resource Manager.

   ```powershell
   Connect-AzAccount
   ```

2.  Выберите соответствующую подписку Azure.

   ```powershell
   Get-AzSubscription -SubscriptionName "<Enter Subscription Name here>" | Select-AzSubscription
   ```

3. Измените следующий фрагмент, чтобы создать группу ресурсов, если у вас ее еще нет.

   ```powershell
   New-AzResourceGroup -Name "DemoRG" -Location "West US"
   ```

### <a name="step-3-move-the-expressroute-circuit-to-the-resource-manager-deployment-model"></a>Шаг 3. Перемещение канала ExpressRoute в модель развертывания Resource Manager

Теперь все готово для перемещения канала ExpressRoute из классической модели развертывания в модель Resource Manager. Прежде чем продолжить, просмотрите сведения в статье [Перемещение каналов ExpressRoute из классической модели развертывания в модель развертывания с помощью Resource Manager](expressroute-move.md).

Чтобы переместить канал, измените и выполните следующий фрагмент:

```powershell
Move-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "DemoRG" -Location "West US" -ServiceKey "<Service-key>"
```

В классическом режиме для а ExpressRoute не предусмотрена привязка к региону. Но в Resource Manager каждый ресурс должен быть сопоставлен с регионом Azure. Регион, указанный в командлете Move-AzExpressRouteCircuit, технически может быть любым регионом. В соответствии с задачами организации может потребоваться выбрать регион, который точно представляет ваше расположение пиринга.

> [!NOTE]
> * После перемещения классического канала ExpressRoute в модель развертывания диспетчер ресурсов он по умолчанию будет иметь доступ как к классической, так и по диспетчер ресурсов модели развертывания.
> * Новое имя, указанное в предыдущем командлете, будет использоваться для адресации ресурса. По существу канал будет переименован.

## <a name="modify-circuit-access"></a>Изменение доступа к каналу

### <a name="to-enable-expressroute-circuit-access-for-both-deployment-models"></a>Включение доступа к каналу ExpressRoute для обеих моделей развертывания

Вы можете включить доступ к классической модели развертывания для каналов ExpressRoute, созданных в модели развертывания диспетчер ресурсов. Выполните следующие командлеты, чтобы разрешить доступ для обеих моделей развертывания:

1. Получите сведения о канале.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Задайте параметру "Allow Classic Operations" (Разрешить классические операции) значение TRUE.

   ```powershell
   $ckt.AllowClassicOperations = $true
   ```

3. Обновите канал. После успешного завершения этой операции канал будет отображаться в классической модели развертывания.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

4. Выполните следующий командлет, чтобы получить сведения о канале ExpressRoute. Выходные данные должны содержать ключ службы.

   ```powershell
   get-azurededicatedcircuit
   ```

5. Теперь вы можете управлять связями с каналом ExpressRoute с помощью команд классической модели развертывания для классических виртуальных сетей, а с помощью команд Resource Manager — для виртуальных сетей Resource Manager. Следующие статьи содержат сведения об управлении ссылками на канал ExpressRoute:

    * [Связывание виртуальных сетей с каналами ExpressRoute в модели развертывания диспетчера ресурсов](expressroute-howto-linkvnet-arm.md)
    * [Связывание виртуальных сетей с каналами ExpressRoute в классической модели развертывания](expressroute-howto-linkvnet-classic.md)

### <a name="to-disable-expressroute-circuit-access-to-the-classic-deployment-model"></a>Отключение доступа к каналу ExpressRoute для классической модели развертывания

Выполните следующий командлет, чтобы отключить доступ для классической модели развертывания:

1. Получите сведения о канале ExpressRoute.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Задайте параметру "Allow Classic Operations" (Разрешить классические операции) значение FALSE.

   ```powershell
   $ckt.AllowClassicOperations = $false
   ```

3. Обновите канал. После успешного завершения этой операции канал не будет отображаться в классической модели развертывания.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание и изменение маршрутизации для канала ExpressRoute](expressroute-howto-routing-arm.md)
* [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)
