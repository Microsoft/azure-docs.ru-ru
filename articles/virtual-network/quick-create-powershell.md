---
title: Краткое руководство. Создание виртуальной сети с помощью Azure PowerShell
titlesuffix: Azure Virtual Network
description: Из этого краткого руководства вы узнаете, как создать виртуальную сеть с помощью портала Azure. Виртуальная сеть позволяет ресурсам Azure взаимодействовать между собой и с Интернетом.
author: KumudD
ms.service: virtual-network
ms.topic: quickstart
ms.date: 03/06/2021
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 7ee10327ab95a3e66e5592593ae72d6e5cd8d606
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106060607"
---
# <a name="quickstart-create-a-virtual-network-using-powershell"></a>Краткое руководство. Создание виртуальной сети с помощью PowerShell

Виртуальная сеть позволяет ресурсам Azure, таким как виртуальные машины, обмениваться данными в частном порядке и взаимодействовать через Интернет. 

Из этого краткого руководства вы узнаете, как создать виртуальную сеть. Создав виртуальную сеть, разверните в ней две виртуальные машины. Затем вы подключитесь к виртуальным машинам из Интернета и установите частную связь через виртуальную сеть.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group-and-a-virtual-network"></a>Создание группы ресурсов и виртуальной сети

Для настройки группы ресурсов и виртуальной сети необходимо выполнить несколько шагов.

### <a name="create-the-resource-group"></a>Создание группы ресурсов

Перед созданием виртуальной сети необходимо создать группу ресурсов, которая будет содержать эту виртуальную сеть. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). В этом примере создается группа ресурсов с именем **CreateVNetQS-rg** в расположении **EastUS**:

```azurepowershell-interactive
$rg = @{
    Name = 'CreateVNetQS-rg'
    Location = 'EastUS'
}
New-AzResourceGroup @rg
```

### <a name="create-the-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В этом примере создается виртуальная сеть по умолчанию **myVNet** в расположении **EastUS**:

```azurepowershell-interactive
$vnet = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreateVNetQS-rg'
    Location = 'EastUS'
    AddressPrefix = '10.0.0.0/16'    
}
$virtualNetwork = New-AzVirtualNetwork @vnet
```

### <a name="add-a-subnet"></a>Добавление подсети

Azure развертывает ресурсы в подсети виртуальной сети, поэтому необходимо создать подсеть. Создайте конфигурации подсети с именем **default** с помощью командлета [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig):

```azurepowershell-interactive
$subnet = @{
    Name = 'default'
    VirtualNetwork = $virtualNetwork
    AddressPrefix = '10.0.0.0/24'
}
$subnetConfig = Add-AzVirtualNetworkSubnetConfig @subnet
```

### <a name="associate-the-subnet-to-the-virtual-network"></a>Связка подсети с виртуальной сетью

Вы можете записать конфигурацию подсети в виртуальную сеть с помощью командлета [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork). Эта команда создает подсеть:

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте две виртуальные машины в виртуальной сети.

### <a name="create-the-first-vm"></a>Создание первой виртуальной машины

Создайте первую виртуальную машину с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). При запуске приведенной ниже команды запрашиваются учетные данные. Введите имя пользователя и пароль виртуальной машины:

```azurepowershell-interactive
$vm1 = @{
    ResourceGroupName = 'CreateVNetQS-rg'
    Location = 'EastUS'
    Name = 'myVM1'
    VirtualNetworkName = 'myVNet'
    SubnetName = 'default'
}
New-AzVM @vm1 -AsJob
```

Параметр `-AsJob` создает виртуальную машину в фоновом режиме. Перейдите к следующему шагу.

Когда Azure начнет создание виртуальной машины в фоновом режиме, вы получите результат, аналогичный следующему:

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```

### <a name="create-the-second-vm"></a>Создание второй виртуальной машины

Создайте вторую виртуальную машину с помощью следующей команды:

```azurepowershell-interactive
$vm2 = @{
    ResourceGroupName = 'CreateVNetQS-rg'
    Location = 'EastUS'
    Name = 'myVM2'
    VirtualNetworkName = 'myVNet'
    SubnetName = 'default'
}
New-AzVM @vm2
```

Вам нужно будет создать другого пользователя и пароль. Создание виртуальной машины в Azure занимает несколько минут.

> [!IMPORTANT]
> Не переходите к следующему шагу, пока Azure не закончит создание виртуальной машины.  Процесс создания закончится, когда выходные данные будут возвращены в PowerShell.

## <a name="connect-to-a-vm-from-the-internet"></a>Подключение к виртуальной машине из Интернета

Чтобы получить его, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress).

Приведенный ниже пример возвращает общедоступный IP-адрес виртуальной машины **myVm1**:

```azurepowershell-interactive
$ip = @{
    Name = 'myVM1'
    ResourceGroupName = 'CreateVNetQS-rg'
}
Get-AzPublicIpAddress @ip | select IpAddress
```

Откройте командную строку на локальном компьютере. Выполните команду `mstsc`. Замените `<publicIpAddress>` общедоступным IP-адресом, полученным на последнем шаге:

> [!NOTE]
> Если вы выполняли эти команды в командной строке PowerShell на локальном компьютере и используете модуль Az PowerShell 1.0 или более поздней версии, можете продолжить работу в этом интерфейсе.

```cmd
mstsc /v:<publicIpAddress>
```
1. При появлении запроса выберите **Подключиться**.

1. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

    > [!NOTE]
    > Возможно, потребуется выбрать **More choices** > **Use a different account** (Дополнительные варианты > Использовать другую учетную запись), чтобы указать учетные данные, введенные при создании виртуальной машины.

1. Щелкните **ОК**.

1. Вы можете получить предупреждение о сертификате. В таком случае выберите **Да** или **Продолжить**.

## <a name="communicate-between-vms"></a>Взаимодействие между виртуальными машинами

1. На удаленном рабочем столе **myVm1** откройте PowerShell.

1. Введите `ping myVm2`.

    Вы получите сообщение, аналогичное следующему:

    ```powershell
    PS C:\Users\myVm1> ping myVm2

    Pinging myVm2.ovvzzdcazhbu5iczfvonhg2zrb.bx.internal.cloudapp.net
    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 10.0.0.5:
        Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
    ```

    Проверка связи завершилась ошибкой, так как в ней используется протокол ICMP. По умолчанию ICMP запрещен брандмауэром Windows.

1. Чтобы разрешить виртуальной машине **myVm2** проверять связь с **myVm1** на дальнейшем этапе, введите следующую команду:

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```

    Эта команда разрешает входящий трафик ICMP через брандмауэр Windows.

1. Закройте подключение к удаленному рабочему столу **myVm1**.

1. Повторите шаги, приведенные в разделе [Подключение к виртуальной машине из Интернета](#connect-to-a-vm-from-the-internet). На этот раз подключитесь к **myVm2**.

1. В командной строке на виртуальной машине **myVm2** введите `ping myvm1`.

    Вы получите сообщение, аналогичное следующему:

    ```cmd
    C:\windows\system32>ping myVm1

    Pinging myVm1.e5p2dibbrqtejhq04lqrusvd4g.bx.internal.cloudapp.net [10.0.0.4] with 32 bytes of data:
    Reply from 10.0.0.4: bytes=32 time=2ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128

    Ping statistics for 10.0.0.4:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 2ms, Average = 0ms
    ```

    Вы получите ответы от **myVm1**, так как на предыдущем шаге разрешили использовать ICMP через брандмауэр Windows на виртуальной машине **myVm1**.

1. Закройте подключение к удаленному рабочему столу **myVm2**.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с виртуальной сетью и виртуальными машинами удалите группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'CreateVNetQS-rg' -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве: 

* Вы создали виртуальную сеть по умолчанию и две виртуальные машины. 
* Затем вы подключились к одной виртуальной машине из Интернета и установили частную связь между двумя виртуальными машинами.

Частный обмен данными между виртуальными машинами не ограничен в виртуальной сети. 

Перейдите к следующей статье, чтобы узнать больше о настройке различных типов сетевого взаимодействия с виртуальными машинами:
> [!div class="nextstepaction"]
> [Фильтрация сетевого трафика](tutorial-filter-network-traffic.md)
