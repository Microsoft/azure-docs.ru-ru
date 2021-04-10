---
title: Краткое руководство. Создание виртуальной машины Windows с помощью Azure PowerShell
description: Из этого краткого руководства вы узнаете, как создать виртуальную машину Windows с помощью Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/02/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 78b912dd649ff942e0187f9b3602d9213383b8c9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102560741"
---
# <a name="quickstart-create-a-windows-virtual-machine-in-azure-with-powershell"></a>Краткое руководство. Создание виртуальной машины Windows в Azure с помощью PowerShell

Модуль PowerShell используется для создания ресурсов Azure и управления ими с помощью командной строки PowerShell или сценариев. В этом кратком руководстве показано, как с помощью Azure PowerShell развернуть в Azure виртуальную машину под управлением Windows Server 2016. Вы также подключитесь к виртуальной машине по протоколу удаленного рабочего стола (RDP) и установите веб-сервер IIS, чтобы проверить работу виртуальной машины.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.


## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm). Задайте имена для каждого ресурса, и командлет `New-AzVM` создаст ресурсы (если они еще не существуют).

При появлении запроса укажите имя пользователя и пароль в качестве учетных данных для входа на виртуальную машину.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80,3389
```

## <a name="connect-to-virtual-machine"></a>Подключение к виртуальной машине

Когда завершится развертывание, войдите на виртуальную машину по протоколу удаленного рабочего стола. Чтобы проверить работу виртуальной машины, можно установить веб-сервер IIS.

Чтобы узнать общедоступный IP-адрес виртуальной машины, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress):

```powershell
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

На локальном компьютере выполните следующую команду, чтобы создать сеанс удаленного рабочего стола. Замените указанный IP-адрес общедоступным IP-адресом своей виртуальной машины. 

```powershell
mstsc /v:publicIpAddress
```

В окне **Безопасность Windows** выберите **Больше вариантов** и щелкните **Использовать другую учетную запись**. Введите имя пользователя в формате **localhost**\\*имя_пользователя*, а затем введите созданный для этой виртуальной машины пароль и нажмите кнопку **ОК**.

При входе в систему может появиться предупреждение о сертификате. Щелкните **Да** или **Продолжить**, чтобы создать подключение.

## <a name="install-web-server"></a>Установка веб-сервера

Чтобы проверить работу виртуальной машины, установите веб-сервер IIS. На виртуальной машине откройте командную строку PowerShell и выполните следующую команду:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

После этого закройте RDP-подключение к виртуальной машине.

## <a name="view-the-web-server-in-action"></a>Проверка работы веб-сервера

Установив IIS и открыв через Интернет порт 80 на виртуальной машине, откройте страницу приветствия IIS по умолчанию в любом браузере. Используйте общедоступный IP-адрес виртуальной машины, полученный на предыдущем шаге. В следующем примере показан веб-сайт IIS по умолчанию:

![Сайт IIS по умолчанию](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы развернули простую виртуальную машину, открыли сетевой порт для веб-трафика и установили базовый веб-сервер. Дополнительные сведения о виртуальных машинах Azure см. в руководстве по работе с виртуальными машинами Windows.

> [!div class="nextstepaction"]
> [Создание виртуальных машин Windows и управление ими с помощью модуля Azure PowerShell](./tutorial-manage-vm.md)
