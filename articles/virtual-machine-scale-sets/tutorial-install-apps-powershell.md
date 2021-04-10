---
title: Руководство по установке приложений в масштабируемом наборе с помощью Azure PowerShell
description: Узнайте, как с помощью Azure PowerShell устанавливать приложения в масштабируемые наборы виртуальных машин с использованием расширения пользовательских скриптов.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.date: 11/08/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 7c365e83c0208a5105d9396fb788966bbd4643ac
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105934589"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-azure-powershell"></a>Руководство. Установка приложений в масштабируемые наборы виртуальных машин с помощью Azure PowerShell

Для запуска приложений в экземплярах виртуальных машин в масштабируемом наборе необходимо сначала установить компоненты и необходимые файлы этих приложений. Из предыдущего руководства вы узнали, как создать и использовать настраиваемый образ виртуальной машины для развертывания экземпляров виртуальных машин. Этот настраиваемый образ включал ручную установку и конфигурацию приложения. Также можно автоматизировать установку приложений в масштабируемом наборе после развертывания каждого экземпляра виртуальной машины или обновить приложение, которое уже выполняется в масштабируемом наборе. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * автоматически устанавливать приложения в масштабируемый набор;
> * использовать расширения пользовательских скриптов;
> * обновлять приложение, работающее в масштабируемом наборе.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="what-is-the-azure-custom-script-extension"></a>Что такое расширение пользовательских скриптов Azure?
Расширение настраиваемых сценариев скачивает и выполняет сценарии на виртуальных машинах Azure. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления. Сценарии можно скачать из службы хранилища Azure или GitHub или передать на портал Azure во время выполнения расширения.

Расширение настраиваемых скриптов интегрируется с шаблонами Azure Resource Manager. Эту возможность можно реализовать с помощью Azure CLI, Azure PowerShell, портала Azure или REST API. Дополнительные сведения см. в статье [Расширение Custom Script в ОС Windows](../virtual-machines/extensions/custom-script-windows.md).

Чтобы увидеть расширение пользовательских скриптов в действии, создайте масштабируемый набор, который устанавливает веб-сервер IIS и выводит имя узла экземпляра виртуальной машины масштабируемого набора. Определение расширения пользовательских скриптов скачивает пример скрипта из репозитория GitHub, устанавливает необходимые пакеты, а затем записывает имя узла экземпляра виртуальной машины на базовую HTML-страницу.


## <a name="create-a-scale-set"></a>Создание масштабируемого набора
Теперь создайте масштабируемый набор виртуальных машин с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss). Чтобы распределить трафик между отдельными экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Подсистема балансировки нагрузки включает правила для распределения трафика через TCP-порт 80. Она также разрешает трафик с удаленного рабочего стола через TCP-порт 3389 и удаленное взаимодействие с PowerShell через TCP-порт 5985. При появлении запроса можно задать собственные административные учетные данные для экземпляров виртуальных машин в масштабируемом наборе:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.


## <a name="create-custom-script-extension-definition"></a>Создание определения для расширения настраиваемых скриптов
Azure PowerShell использует хэш-таблицу для хранения скачиваемых файлов и выполняемых команд. Ниже используется пример скрипта из репозитория GitHub. Сначала создается объект конфигурации следующим образом.

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```


Теперь примените расширение пользовательских скриптов с помощью командлета [Add-AzVmssExtension](/powershell/module/az.Compute/Add-azVmssExtension). Ранее определенный объект конфигурации передается расширению. Обновите и запустите расширение на экземплярах виртуальных машин с помощью командлета [Update-AzVmss](/powershell/module/az.compute/update-azvmss).


```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Каждый экземпляр виртуальной машины в масштабируемом наборе скачивает и запускает скрипт из репозитория GitHub. В более сложном примере можно установить несколько компонентов и файлов приложения. Если масштабируемый набор развернут, новые экземпляры виртуальных машин автоматически будут применять одно и то же расширение настраиваемых скриптов и устанавливать нужное приложение.


## <a name="allow-traffic-to-application"></a>Разрешение передачи трафика в приложение

Чтобы разрешить доступ к базовому веб-приложению, создайте сетевую группу безопасности с помощью командлетов [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) и [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Дополнительные сведения см. в статье [Сеть для масштабируемых наборов виртуальных машин Azure](virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive

#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroup" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroup" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

```



## <a name="test-your-scale-set"></a>Проверка масштабируемого набора
Чтобы проверить работу веб-сервера, получите общедоступный IP-адрес своей подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). Следующий пример кода отображает IP-адрес, созданный в группе ресурсов *myResourceGroup*:

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

Введите в браузере общедоступный IP-адрес подсистемы балансировки нагрузки. Подсистема балансировки нагрузки передаст запрос на один из экземпляров виртуальной машины, как показано в следующем примере:

![Базовая веб-страница IIS](media/tutorial-install-apps-powershell/running-iis.png)

Не закрывайте веб-браузер, чтобы увидеть обновленную версию на следующем шаге.


## <a name="update-app-deployment&quot;></a>Обновление развертывания приложения
На протяжении жизненного цикла масштабируемого набора может понадобиться развернуть обновленную версию приложения. Расширение пользовательских скриптов позволяет ссылаться на обновленный скрипт развертывания и повторно применять расширение к масштабируемому набору. Когда на предыдущем шаге создавался масштабируемый набор, для параметра `-UpgradePolicyMode` было задано значение *Automatic*. Этот параметр позволяет экземплярам виртуальных машин в масштабируемом наборе автоматически обновлять и применять последнюю версию приложения.

Создайте определение конфигурации с именем *customConfigv2*. Это определение позволяет запустить обновленную версию *v2* скрипта установки приложения.

```azurepowershell-interactive
$customConfigv2 = @{
  &quot;fileUris&quot; = (,&quot;https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

Обновите конфигурацию расширения настраиваемых скриптов к экземплярам виртуальных машин в своем масштабируемом наборе. Определение *customConfigv2* используется для применения обновленной версии приложения.

```azurepowershell-interactive
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"
 
$vmss.VirtualMachineProfile.ExtensionProfile[0].Extensions[0].Settings = $customConfigv2
 
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Все экземпляры виртуальной машины в масштабируемом наборе автоматически обновляются до последней версии примера веб-страницы. Чтобы просмотреть обновленную версию, обновите веб-сайт в браузере:

![Обновленная веб-страница IIS](media/tutorial-install-apps-powershell/running-iis-updated.png)


## <a name="clean-up-resources"></a>Очистка ресурсов
Чтобы удалить масштабируемый набор и дополнительные ресурсы, удалите группу ресурсов и все входящие в нее ресурсы с помощью команды [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Параметр `-Force` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса. При использовании параметра `-AsJob` управление возвращается в командную строку без ожидания завершения операции.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как автоматически устанавливать и обновлять приложения в масштабируемом наборе с помощью Azure PowerShell, в частности, как выполнять такие задачи:

> [!div class="checklist"]
> * автоматически устанавливать приложения в масштабируемый набор;
> * использовать расширения пользовательских скриптов;
> * обновлять приложение, работающее в масштабируемом наборе.

Перейдите к следующему руководству, чтобы узнать, как автоматически масштабировать масштабируемый набор.

> [!div class="nextstepaction"]
> [Автоматическое масштабирование масштабируемых наборов](tutorial-autoscale-powershell.md)
