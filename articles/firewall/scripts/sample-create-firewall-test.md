---
title: Создание тестовой среды брандмауэра Azure
description: Этот сценарий создает брандмауэр и тестовую сетевую среду. Сеть имеет одну виртуальную сеть с тремя подсетями.
services: virtual-network
author: vhorne
ms.service: firewall
ms.devlang: powershell
ms.topic: sample
ms.date: 11/19/2019
ms.author: victorh
ms.openlocfilehash: 4158bc07373a2d0aa6fb6ceaf2dce62b50bb6bd7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94658372"
---
# <a name="create-an-azure-firewall-test-environment"></a>Создание тестовой среды брандмауэра Azure

Этот сценарий создает брандмауэр и тестовую сетевую среду. Сеть содержит одну виртуальную сеть с тремя подсетями: *AzureFirewallSubnet*, *ServersSubnet* и *JumpboxSubnet*. ServersSubnet и JumpboxSubnet содержат по одному 2-ядерному серверу Windows Server.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

В AzureFirewallSubnet размещен брандмауэр, которому присваивается коллекция правил приложения, единственное правило которой разрешает доступ к `www.microsoft.com`.

Создается пользовательский маршрут, направляющий сетевой трафик из ServersSubnet через брандмауэр, где к нему применяются правила брандмауэра.

Скрипт можно выполнить из Azure [Cloud Shell](https://shell.azure.com/powershell) или из локальной установки PowerShell. 

При локальном запуске PowerShell для выполнения этого скрипта понадобится модуль Azure PowerShell. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. 

Если необходимо выполнить обновление, можно использовать модуль `PowerShellGet`, встроенный в Windows 10 и Windows Server 2016.

> [!NOTE]
>В других версиях Windows необходимо установить модуль `PowerShellGet` перед его использованием. Можно запустить команду `Get-Module -Name PowerShellGet -ListAvailable | Select-Object -Property Name,Version,Path`, чтобы определить установлен ли этот модуль на компьютере. Если выходные данные не заполнены, необходимо установить последнюю версию [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616).

Дополнительные сведения см. в статье об [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

Любая имеющаяся среда Azure PowerShell, установленная с помощью установщика веб-платформы, будет конфликтовать с установкой PowerShellGet, и ее нужно удалить.

Помните, что при локальном запуске PowerShell необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта


[!code-azurepowershell-interactive[main](../../../powershell_scripts/firewall/create-fw-test.ps1  "Create a firewall test environment")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, виртуальную машину и все связанные с ней ресурсы:

```powershell
Remove-AzResourceGroup -Name AzfwSampleScriptEastUS -Force
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, виртуальной сети и групп безопасности сети в этом скрипте используются приведенные ниже команды. Для каждой команды в следующей таблице приведены ссылки на соответствующую документацию:

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Создает объект конфигурации подсети. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть Azure и интерфейсную подсеть. |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Создает правила безопасности, которые будут добавлены в группу безопасности сети. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) |Создает правила групп безопасности сети, которые разрешают или блокируют определенные порты для конкретных подсетей. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Связывает группы безопасности сети с подсетями. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес для доступа к виртуальной машине из Интернета. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Создает виртуальные сетевые интерфейсы и присоединяет их к интерфейсной и внутренней подсети виртуальной сети. |
| [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) | Создает конфигурацию виртуальной машины. Эта конфигурация включает в себя такие сведения, как имя виртуальной машины, операционную систему и учетные данные администратора. Данная конфигурации используется при создании виртуальной машины. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Создайте виртуальную машину. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |
|[New-AzFirewall.](/powershell/module/az.network/new-azfirewall)| Создает экземпляр службы "Брандмауэр Azure".|
|[Get-AzFirewall](/powershell/module/az.network/get-azfirewall)|Получает объект "Брандмауэр Azure".|
|[New-AzFirewallApplicationRule](/powershell/module/az.network/new-azfirewallapplicationrule)|Создает правило приложения службы "Брандмауэр Azure".|
|[Set-AzFirewall](/powershell/module/az.network/set-azfirewall)|Фиксирует изменения в объекте "Брандмауэр Azure".|

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).