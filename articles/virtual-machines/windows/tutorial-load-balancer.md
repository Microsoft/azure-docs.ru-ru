---
title: Учебник. Распределение нагрузки на виртуальных машинах Windows в Azure
description: В этом руководстве описано, как с помощью Azure PowerShell создать подсистему балансировки нагрузки для высокодоступного безопасного приложения, выполняемого на трех виртуальных машинах Windows.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/03/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: d6c872ecfbad6cb8bb01ad5a7c3df8c47aadaebe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102549113"
---
# <a name="tutorial-load-balance-windows-virtual-machines-in-azure-to-create-a-highly-available-application-with-azure-powershell"></a>Руководство. Балансировка нагрузки виртуальных машин Windows в Azure для создания высокодоступного приложения с помощью Azure PowerShell
Балансировка нагрузки обеспечивает более высокий уровень доступности за счет распределения входящих запросов между несколькими виртуальными машинами. В этом руководстве вы узнаете о различных компонентах балансировщика нагрузки Azure Load Balancer, распределяющего трафик и обеспечивающего высокую доступность. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание Azure Load Balancer
> * Создание пробы работоспособности балансировщика нагрузки
> * создавать правила трафика подсистемы балансировки нагрузки;
> * Использование расширения пользовательских скриптов для создания базового сайта IIS
> * создавать виртуальные машины и присоединять их к подсистеме балансировки нагрузки;
> * Просмотр балансировщика нагрузки в действии
> * добавлять и удалять виртуальные машины из подсистемы балансировки нагрузки.

## <a name="azure-load-balancer-overview"></a>Обзор Azure Load Balancer
Azure Load Balancer представляет собой балансировщик нагрузки уровня 4 (TCP, UDP), который обеспечивает высокий уровень доступности, распределяя входящий трафик между работоспособными виртуальными машинами. Проба работоспособности позволяет отслеживать данный порт на каждой виртуальной машине и передавать трафик только в рабочую виртуальную машину.

Вы определяете интерфейсную конфигурацию IP-адресов, содержащую один или несколько общедоступных IP-адресов. Такая интерфейсная конфигурация IP-адресов предоставляет доступ к балансировщику нагрузки и приложениям через Интернет. 

Виртуальные машины подключаются к балансировщику нагрузки с помощью виртуальной сетевой карты. Для распределения трафика между виртуальными машинами внутренний пул адресов содержит IP-адреса виртуальных сетевых карт, подключенных к балансировщику нагрузки.

Чтобы управлять потоком трафика, нужно определить правила балансировщика нагрузки для определенных портов и протоколов, сопоставленных с виртуальными машинами.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="create-azure-load-balancer"></a>Создание Azure Load Balancer
Этот раздел подробно описывает, как создать и настроить каждый из компонентов балансировщика нагрузки. Прежде чем создать подсистему балансировки нагрузки, выполните командлет [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) для создания группы ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroupLoadBalancer* в расположении *EastUS*.

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS"
```

### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса
Для доступа к приложению через Интернет требуется общедоступный IP-адрес для балансировщика нагрузки. Создайте общедоступный IP-адрес с помощью командлета [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). В следующем примере создается общедоступный IP-адрес *myPublicIP* в группе ресурсов *myResourceGroupLoadBalancer*.

```azurepowershell-interactive
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -AllocationMethod "Static" `
  -Name "myPublicIP"
```

### <a name="create-a-load-balancer"></a>Создание балансировщика нагрузки
Создайте внешний пул IP-адресов с помощью командлета [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). В следующем примере создается пул внешних IP-адресов *myFrontEndPool* и включается адрес *myPublicIP*: 

```azurepowershell-interactive
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
```

Создайте серверный пул адресов с помощью командлета [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). Виртуальные машины присоединяются в этот внутренний пул на следующих шагах. В следующем примере создается внутренний пул адресов *myBackEndPool*.

```azurepowershell-interactive
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig `
  -Name "myBackEndPool"
```

Теперь создайте подсистему балансировки нагрузки с помощью командлета [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). В следующем примере создается подсистема балансировки нагрузки *myLoadBalancer*, использующая пулы серверных и интерфейсных IP-адресов, созданных на предыдущих шагах:

```azurepowershell-interactive
$lb = New-AzLoadBalancer `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myLoadBalancer" `
  -Location "EastUS" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool
```

### <a name="create-a-health-probe"></a>Создание пробы работоспособности
Чтобы балансировщик нагрузки мог следить за состоянием приложения, необходимо настроить пробу работоспособности. Проба работоспособности динамически добавляет или удаляет виртуальные машины из балансировщика нагрузки на основе их ответа на проверки работоспособности. По умолчанию виртуальная машина удаляется из числа машин, на которые балансировщик распределяет нагрузку, после двух последовательных сбоев с интервалом в 15 секунд. Пробу работоспособности можно создать на основе протокола или конкретной страницы проверки работоспособности приложения. 

В следующем примере создается проба TCP. Вы также можете создать настраиваемую пробу HTTP для более детальных проверок. При использовании настраиваемой пробы HTTP нужно создать страницу проверки работоспособности, например *healthcheck.aspx*. Чтобы обеспечить работоспособность узла, проба должна возвращать ответ **HTTP 200 OK** для балансировщика нагрузки.

Чтобы создать пробу работоспособности TCP, выполните командлет [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig). В следующем примере создается проба работоспособности *myHealthProbe*, которая отслеживает каждую виртуальную машину через *TCP*-порт *80*:

```azurepowershell-interactive
Add-AzLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
```

Чтобы применить пробу работоспособности, обновите подсистему балансировки нагрузки с помощью командлета [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer):

```azurepowershell-interactive
Set-AzLoadBalancer -LoadBalancer $lb
```

### <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки
Правило балансировщика нагрузки позволяет определить распределение трафика между виртуальными машинами. Вы определяете интерфейсную конфигурацию IP-адресов для входящего трафика и внутренний пул IP-адресов для приема трафика, а также требуемый порт источника и назначения. Чтобы обеспечить получение трафика только работоспособными виртуальными машинами, можно также определить используемую пробу работоспособности.

Создайте правило подсистемы балансировки нагрузки с помощью командлета [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). В следующем примере создается правило подсистемы балансировки нагрузки *myLoadBalancerRule* для балансировки трафика через *TCP*-порт *80*:

```azurepowershell-interactive
$probe = Get-AzLoadBalancerProbeConfig -LoadBalancer $lb -Name "myHealthProbe"

Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

Обновите подсистему балансировки нагрузки с помощью командлета [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer):

```azurepowershell-interactive
Set-AzLoadBalancer -LoadBalancer $lb
```

## <a name="configure-virtual-network"></a>Настройка виртуальной сети
Прежде чем развертывать виртуальные машины и тестировать балансировщик нагрузки, создайте вспомогательные ресурсы виртуальной сети. Дополнительные сведения о виртуальных сетях см. в руководстве [Управление виртуальными сетями Azure](tutorial-virtual-network.md).

### <a name="create-network-resources"></a>Создание сетевых ресурсов
Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создаются виртуальная сеть *myVnet* и подсеть *mySubnet*.

```azurepowershell-interactive
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -Name "myVnet" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig
```

Для создания виртуальных сетевых адаптеров используется командлет [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). В следующем примере создаются три виртуальных сетевых адаптера (по одной виртуальной сетевой карте для каждой виртуальной машины, используемой приложением). Вы можете в любое время создать дополнительные виртуальные сетевые карты и виртуальные машины и добавить их в балансировщик нагрузки:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   New-AzNetworkInterface `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -Name myVM$i `
     -Location "EastUS" `
     -Subnet $vnet.Subnets[0] `
     -LoadBalancerBackendAddressPool $lb.BackendAddressPools[0]
}
```


## <a name="create-virtual-machines"></a>Создание виртуальных машин
Чтобы улучшить высокую доступность приложения, поместите виртуальные машины в группу доступности.

Создайте группу доступности с помощью командлета [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). В следующем примере создается группа доступности *myAvailabilitySet*.

```azurepowershell-interactive
$availabilitySet = New-AzAvailabilitySet `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myAvailabilitySet" `
  -Location "EastUS" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Теперь вы можете создать виртуальные машины с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). В следующем примере создаются три виртуальные машины и требуемые компоненты виртуальной сети, если они не существуют:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupLoadBalancer" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

Параметр `-AsJob` создает виртуальную машину как фоновую задачу, поэтому PowerShell отображает запрос о возврате. Подробные сведения о фоновых заданиях можно просмотреть с помощью командлета `Job`. Создание и настройка трех виртуальных машин занимает несколько минут.


### <a name="install-iis-with-custom-script-extension"></a>Установка IIS с помощью расширения пользовательских сценариев
В предыдущем руководстве [Настройка виртуальной машины Windows](tutorial-automate-vm-deployment.md) вы узнали, как автоматизировать настройку виртуальной машины с помощью расширения настраиваемых сценариев для Windows. Аналогичный подход можно использовать для установки и настройки служб IIS на виртуальных машинах.

Воспользуйтесь командлетом [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension), чтобы установить расширение пользовательских скриптов. Это расширение запускает `powershell Add-WindowsFeature Web-Server` для установки веб-сервера IIS, а затем обновляет страницу *Default.htm* для отображения имени узла виртуальной машины.

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   Set-AzVMExtension `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -ExtensionName "IIS" `
     -VMName myVM$i `
     -Publisher Microsoft.Compute `
     -ExtensionType CustomScriptExtension `
     -TypeHandlerVersion 1.8 `
     -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
     -Location EastUS
}
```

## <a name="test-load-balancer"></a>Проверка балансировщика нагрузки
Получите общедоступный IP-адрес подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). Следующий пример позволяет получить IP-адрес для созданного ранее *myPublicIP*.

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myPublicIP" | select IpAddress
```

После этого можно ввести общедоступный IP-адрес в веб-браузер. Отображается веб-сайт, а также имя узла виртуальной машины, на которую балансировщик нагрузки направил трафик, как показано в следующем примере:

![Выполнение веб-сайта IIS](./media/tutorial-load-balancer/running-iis-website.png)

Чтобы увидеть, как балансировщик нагрузки распределяет трафик между тремя виртуальными машинами, на которых выполняется приложение, принудительно обновите веб-браузер.


## <a name="add-and-remove-vms"></a>Добавление и удаление виртуальных машин
На виртуальных машинах, выполняющих приложение, может потребоваться выполнить обслуживание, например установить обновления операционной системы. Для обработки большего объема трафика в приложении необходимо добавить дополнительные виртуальные машины. В этом разделе показано, как удалить или добавить виртуальную машину из балансировщика нагрузки.

### <a name="remove-a-vm-from-the-load-balancer"></a>Удаление виртуальной машины из балансировщика нагрузки
Получите сетевой адаптер с помощью командлета [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface), а затем задайте для свойства *LoadBalancerBackendAddressPools* виртуального сетевого адаптера значение *$null*. Наконец, обновите виртуальную сетевую карту.

```azurepowershell-interactive
$nic = Get-AzNetworkInterface `
    -ResourceGroupName "myResourceGroupLoadBalancer" `
    -Name "myVM2"
$nic.Ipconfigurations[0].LoadBalancerBackendAddressPools=$null
Set-AzNetworkInterface -NetworkInterface $nic
```

Чтобы увидеть, как балансировщик нагрузки распределяет трафик между оставшимися двумя виртуальными машинами, на которых выполняется приложение, принудительно обновите веб-браузер. Теперь можно выполнить обслуживание на виртуальной машине, например установить обновления операционной системы или перезагрузить виртуальную машину.

### <a name="add-a-vm-to-the-load-balancer"></a>Добавление виртуальной машины в балансировщик нагрузки
После выполнения обслуживания виртуальной машины (или если необходимо увеличить емкость) задайте для свойства *LoadBalancerBackendAddressPools* виртуального сетевого адаптера значение *BackendAddressPool* из [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer).

Получите подсистему балансировки нагрузки:

```azurepowershell-interactive
$lb = Get-AzLoadBalancer `
    -ResourceGroupName myResourceGroupLoadBalancer `
    -Name myLoadBalancer 
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$lb.BackendAddressPools[0]
Set-AzNetworkInterface -NetworkInterface $nic
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы создали балансировщик нагрузки и присоединили к нему виртуальные машины. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание Azure Load Balancer
> * Создание пробы работоспособности балансировщика нагрузки
> * создавать правила трафика подсистемы балансировки нагрузки;
> * Использование расширения пользовательских скриптов для создания базового сайта IIS
> * создавать виртуальные машины и присоединять их к подсистеме балансировки нагрузки;
> * Просмотр балансировщика нагрузки в действии
> * добавлять и удалять виртуальные машины из подсистемы балансировки нагрузки.

Перейдите к следующему руководству, чтобы научиться управлять сетевым взаимодействием виртуальных машин.

> [!div class="nextstepaction"]
> [Управление виртуальными сетями и виртуальными машинами](./tutorial-virtual-network.md)
