---
title: Создание виртуальных машин Windows в Azure и управление ими, которые используют несколько сетевых интерфейсов
description: Узнайте, как создать виртуальную машину Windows с несколькими сетевыми адаптерами с помощью Azure PowerShell или шаблонов Resource Manager, а также, как управлять ими.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/26/2017
ms.author: cynthn
ms.openlocfilehash: 2664d175818a6e29dcd3f704c6938987bae050cd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102555182"
---
# <a name="create-and-manage-a-windows-virtual-machine-that-has-multiple-nics"></a>Создание виртуальной машины Windows, использующей несколько сетевых адаптеров, и управление ею
Виртуальные машины (VM) в Azure могут иметь несколько виртуальных сетевых адаптеров (NIC). Распространенный сценарий состоит в том, чтобы иметь разные подсети для интерфейсных и внутренних подключений. Вы можете связать несколько сетевых адаптеров на виртуальной машине с несколькими подсетями, но эти подсети должны находиться в одной и той же виртуальной сети (vNet). В этой статье подробно описывается, как создать виртуальную машину с несколькими сетевыми адаптерами. Вы также узнаете, как добавить или удалить сетевые адаптеры на существующей виртуальной машине. Различные [размеры виртуальных машин](../sizes.md) поддерживают разное число сетевых карт, так что выбирайте соответствующий размер виртуальной машины.

## <a name="prerequisites"></a>Предварительные условия

В следующих примерах замените имена параметров собственными значениями. Примеры имен параметров: *myResourceGroup*, *myVnet* и *myVM*.

 

## <a name="create-a-vm-with-multiple-nics"></a>Создание виртуальной машины с несколькими сетевыми адаптерами
Сначала создайте группу ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *EastUs* :

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

### <a name="create-virtual-network-and-subnets"></a>Создание виртуальных сетей и подсетей
Общий сценарий для виртуальной сети с двумя или большим количеством подсетей. Одна подсеть может предназначаться для интерфейсного трафика, а другая — для внутреннего. Чтобы подключиться к двум подсетям, вам необходимо будет использовать несколько сетевых адаптеров на виртуальной машине.

1. Определите две подсети виртуальной сети с помощью [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). В следующем примере определяются подсети для *mySubnetFrontEnd* и *mySubnetBackEnd*:

    ```powershell
    $mySubnetFrontEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetFrontEnd" `
        -AddressPrefix "192.168.1.0/24"
    $mySubnetBackEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetBackEnd" `
        -AddressPrefix "192.168.2.0/24"
    ```

2. Создайте виртуальную сеть и подсети с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVnet*:

    ```powershell
    $myVnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "EastUs" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $mySubnetFrontEnd,$mySubnetBackEnd
    ```


### <a name="create-multiple-nics"></a>Создание нескольких сетевых карт
Создайте два сетевых адаптера с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). Подключите один сетевой адаптер к интерфейсной подсети, а другой — к внутренней. В следующем примере создаются два сетевых адаптера с именами *myNic1* и *myNic2*:

```powershell
$frontEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetFrontEnd'}
$myNic1 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic1" `
    -Location "EastUs" `
    -SubnetId $frontEnd.Id

$backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}
$myNic2 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic2" `
    -Location "EastUs" `
    -SubnetId $backEnd.Id
```

Обычно также создается [группа безопасности сети](../../virtual-network/network-security-groups-overview.md) для фильтрации трафика для виртуальной машины и [подсистема балансировки нагрузки](../../load-balancer/load-balancer-overview.md) для распределения трафика между несколькими виртуальными машинами.

### <a name="create-the-virtual-machine"></a>Создание виртуальной машины
Теперь начните создание конфигурации виртуальной машины. Для каждого размера виртуальной машины существует ограничение на общее количество сетевых карт, которые можно в нее добавить. Дополнительные сведения см. в статье [Размеры виртуальных машин Windows в Azure](../sizes.md).

1. Задайте переменной `$cred` учетные данные виртуальной машины, как показано ниже:

    ```powershell
    $cred = Get-Credential
    ```

2. Определите виртуальную машину с помощью [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). В следующем примере определяется виртуальная машина с именем *myVM* и используется размер виртуальной машины, поддерживающий более двух сетевых адаптеров (*Standard_DS3_v2*):

    ```powershell
    $vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_DS3_v2"
    ```

3. Создайте оставшуюся часть конфигурации виртуальной машины с помощью [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) и [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage). В следующем примере создается виртуальная машина Windows Server 2016:

    ```powershell
    $vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
        -Windows `
        -ComputerName "myVM" `
        -Credential $cred `
        -ProvisionVMAgent `
        -EnableAutoUpdate
    $vmConfig = Set-AzVMSourceImage -VM $vmConfig `
        -PublisherName "MicrosoftWindowsServer" `
        -Offer "WindowsServer" `
        -Skus "2016-Datacenter" `
        -Version "latest"
   ```

4. Подключите два созданных ранее сетевых адаптера с помощью [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface):

    ```powershell
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic1.Id -Primary
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic2.Id
    ```

5. Создайте виртуальную машину с помощью [New-AzVM](/powershell/module/az.compute/new-azvm):

    ```powershell
    New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "EastUs"
    ```

6. Добавьте в операционную систему маршруты для дополнительных сетевых адаптеров, выполнив инструкции из раздела [Настройка гостевой ОС для нескольких сетевых карт](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-an-existing-vm"></a>Добавление сетевой карты в существующую виртуальную машину
Чтобы добавить виртуальный сетевой адаптер в имеющуюся виртуальную машину, освободите ее, добавьте виртуальный сетевой адаптер, а затем запустите виртуальную машину. Различные [размеры виртуальных машин](../sizes.md) поддерживают разное число сетевых карт, так что выбирайте соответствующий размер виртуальной машины. При необходимости вы можете [изменить размер виртуальной машины](resize-vm.md).

1. Освободите виртуальную машину с помощью командлета [Stop-AzVm](/powershell/module/az.compute/stop-azvm). В следующем примере освобождается виртуальная машина с именем *myVM* в группе ресурсов *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Получите существующую конфигурацию виртуальной машины с помощью командлета [Get-AzVm](/powershell/module/az.compute/get-azvm). В следующем примере возвращаются сведения для виртуальной машины с именем *myVM* в *myResourceGroup*:

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. В следующем примере создается виртуальный сетевой адаптер с помощью [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) с именем *myNic3*, подключенный к *mySubnetBackEnd*. Затем с помощью [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) он подключается к виртуальной машине *myVM* в *myResourceGroup*

    ```powershell
    # Get info for the back end subnet
    $myVnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
    $backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}

    # Create a virtual NIC
    $myNic3 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic3" `
        -Location "EastUs" `
        -SubnetId $backEnd.Id

    # Get the ID of the new virtual NIC and add to VM
    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "MyNic3").Id
    Add-AzVMNetworkInterface -VM $vm -Id $nicId | Update-AzVm -ResourceGroupName "myResourceGroup"
    ```

    ### <a name="primary-virtual-nics"></a>Главные виртуальные сетевые карты
    Один из сетевых адаптеров на виртуальной машине с несколькими сетевыми картами должен быть главным. Если один из имеющихся виртуальных сетевых адаптеров на виртуальной машине уже используется в качестве главного, вы можете пропустить этот шаг. Чтобы перейти к следующему примеру, предполагается, что на виртуальной машине используются два виртуальных сетевых адаптера. Добавим первый сетевой адаптер (`[0]`) в качестве главного:
        
    ```powershell
    # List existing NICs on the VM and find which one is primary
    $vm.NetworkProfile.NetworkInterfaces
    
    # Set NIC 0 to be primary
    $vm.NetworkProfile.NetworkInterfaces[0].Primary = $true
    $vm.NetworkProfile.NetworkInterfaces[1].Primary = $false
    
    # Update the VM state in Azure
    Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"
    ```

4. Запустите виртуальную машину с помощью [Start-AzVm](/powershell/module/az.compute/start-azvm).

    ```powershell
    Start-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

5. Добавьте в операционную систему маршруты для дополнительных сетевых адаптеров, выполнив инструкции из раздела [Настройка гостевой ОС для нескольких сетевых карт](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-an-existing-vm"></a>Удаление сетевой карты из существующей виртуальной машины
Чтобы удалить виртуальный сетевой адаптер с имеющейся виртуальной машины, освободите ее, удалите виртуальный сетевой адаптер, а затем запустите виртуальную машину.

1. Освободите виртуальную машину с помощью командлета [Stop-AzVm](/powershell/module/az.compute/stop-azvm). В следующем примере освобождается виртуальная машина с именем *myVM* в группе ресурсов *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Получите существующую конфигурацию виртуальной машины с помощью командлета [Get-AzVm](/powershell/module/az.compute/get-azvm). В следующем примере возвращаются сведения для виртуальной машины с именем *myVM* в *myResourceGroup*:

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. С помощью [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) получите сведения об удалении сетевого адаптера. В следующем примере возвращаются сведения о *myNic3*:

    ```powershell
    # List existing NICs on the VM if you need to determine NIC name
    $vm.NetworkProfile.NetworkInterfaces

    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "myNic3").Id   
    ```

4. Удалите сетевой адаптер с помощью [Remove-AzVMNetworkInterface](/powershell/module/az.compute/remove-azvmnetworkinterface), а затем обновите виртуальную машину с помощью [Update-AzVm](/powershell/module/az.compute/update-azvm). В следующем примере удаляется сетевой адаптер *myNic3*, полученный `$nicId` на предыдущем шаге:

    ```powershell
    Remove-AzVMNetworkInterface -VM $vm -NetworkInterfaceIDs $nicId | `
        Update-AzVm -ResourceGroupName "myResourceGroup"
    ```   

5. Запустите виртуальную машину с помощью [Start-AzVm](/powershell/module/az.compute/start-azvm).

    ```powershell
    Start-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```   

## <a name="create-multiple-nics-with-templates"></a>Создание нескольких сетевых адаптеров с помощью шаблонов
Шаблоны Azure Resource Manager дают возможность создать несколько экземпляров ресурса во время развертывания, в том числе создать несколько сетевых адаптеров. В шаблонах Azure Resource Manager используются декларативные JSON-файлы для определения среды. Дополнительные сведения см. в статье [Общие сведения о диспетчере ресурсов Azure](../../azure-resource-manager/management/overview.md). Чтобы указать число создаваемых экземпляров, вы можете использовать объект *copy*:

```json
"copy": {
    "name": "multiplenics",
    "count": "[parameters('count')]"
}
```

Дополнительные сведения см. в статье [Развертывание нескольких экземпляров ресурса или свойства в шаблонах Azure Resource Manager *копирования*](../../azure-resource-manager/templates/copy-resources.md). 

Вы также можете использовать `copyIndex()`, чтобы добавить номер к имени ресурса. Затем вы можете создать сетевые адаптеры с именами *myNic1*, *MyNic2* и т. д. Ниже показан пример добавления значения индекса.

```json
"name": "[concat('myNic', copyIndex())]", 
```

Вы можете ознакомиться с полным примером [создания нескольких сетевых адаптеров с помощью шаблонов Resource Manager](../../virtual-network/template-samples.md).

Добавьте в операционную систему маршруты для дополнительных сетевых адаптеров, выполнив инструкции из раздела [Настройка гостевой ОС для нескольких сетевых карт](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Настройка гостевой ОС для нескольких сетевых карт

Azure назначает шлюз по умолчанию для первого (основного) сетевого интерфейса, подключенного к виртуальной машине. Azure не назначает шлюз по умолчанию для второго (дополнительного) сетевого интерфейса, подключенного к виртуальной машине. Таким образом, невозможно по умолчанию связываться с ресурсами, к которым подключен дополнительный сетевой интерфейс вне подсети. Дополнительные сетевые интерфейсы могут взаимодействовать с ресурсами за пределами их подсети, однако шаги для обеспечения взаимодействия различаются для разных операционных систем.

1. В командной строке Windows выполните команду `route print`, которая выводит примерно следующие выходные данные для виртуальной машины с двумя подключенными сетевыми интерфейсами.

    ```
    ===========================================================================
    Interface List
    3...00 0d 3a 10 92 ce ......Microsoft Hyper-V Network Adapter #3
    7...00 0d 3a 10 9b 2a ......Microsoft Hyper-V Network Adapter #4
    ===========================================================================
    ```
 
    В этом примере **сетевой адаптер Hyper-V (Майкрософт) №4** (интерфейс 7) — это дополнительный сетевой интерфейс, которому не назначен шлюз по умолчанию.

2. В командной строке выполните команду `ipconfig`, чтобы узнать, какой IP-адрес назначен дополнительному сетевому интерфейсу. В этом примере для интерфейса 7 назначен адрес 192.168.2.4. Для дополнительного сетевого интерфейса не возвращен адрес шлюза по умолчанию.

3. Для маршрутизации всего трафика, предназначенного для адресов за пределами подсети дополнительного сетевого интерфейса, в подсеть шлюза выполните следующую команду.

    ```
    route add -p 0.0.0.0 MASK 0.0.0.0 192.168.2.1 METRIC 5015 IF 7
    ```

    Адрес шлюза для подсети — это первый IP-адрес (заканчивается на .1) в диапазоне адресов, определенных для подсети. Если не нужно проводить маршрутизацию всего трафика вне пределов подсети, можно добавить отдельные маршруты в указанные места назначения. Например, если требуется маршрутизация трафика только из дополнительного сетевого интерфейса в сеть 192.168.3.0, введите следующую команду:

      ```
      route add -p 192.168.3.0 MASK 255.255.255.0 192.168.2.1 METRIC 5015 IF 7
      ```
  
4. Для подтверждения успешной связи с ресурсом в сети 192.168.3.0 используйте, например,следующую команду, чтобы проверить связь по адресу 192.168.3.4 с помощью интерфейса 7 (192.168.2.4).

    ```
    ping 192.168.3.4 -S 192.168.2.4
    ```

    Может потребоваться открыть протокол ICMP через брандмауэр Windows устройства, на котором проводится проверка связи, с помощью следующей команды.
  
      ```
      netsh advfirewall firewall add rule name=Allow-ping protocol=icmpv4 dir=in action=allow
      ```
  
5. Чтобы убедиться, что маршрут добавлен в таблицу маршрутизации, введите команду `route print`, которая возвращает выходные данные, аналогичные следующему тексту:

    ```
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.4     15
              0.0.0.0          0.0.0.0      192.168.2.1      192.168.2.4   5015
    ```

    Маршрут указанный в *192.168.1.1* в разделе **Шлюз** — это маршрут по умолчанию для основного сетевого интерфейса. Маршрут *192.168.2.1* в разделе **Шлюз** — это добавленный маршрут.

## <a name="next-steps"></a>Дальнейшие действия
При создании виртуальной машины с несколькими сетевыми адаптерами ознакомьтесь со статьей [Размеры виртуальных машин Windows в Azure](../sizes.md). Обратите внимание на максимальное число сетевых адаптеров, поддерживаемых каждым из размеров виртуальной машины.