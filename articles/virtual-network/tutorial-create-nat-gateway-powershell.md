---
title: Учебник по созданию шлюза NAT — PowerShell
titlesuffix: Azure Virtual Network NAT
description: Начало работы с созданием шлюза NAT с помощью Azure PowerShell.
author: asudbring
ms.author: allensu
ms.service: virtual-network
ms.subservice: nat
ms.topic: tutorial
ms.date: 03/09/2021
ms.custom: template-tutorial
ms.openlocfilehash: 884697cee84c05916fe19fb8f9435de86bda291e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102619887"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-powershell"></a>Руководство по Создание шлюза NAT с помощью Azure PowerShell

В этом руководстве показано, как использовать услугу преобразования сетевых адресов (NAT) в виртуальной сети Azure. Вы узнаете, как создать шлюз NAT для обеспечения исходящего подключения в виртуальных машинах в Azure. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создайте виртуальную сеть.
> * Создайте виртуальную машину.
> * Создание шлюза NAT и связывание его с виртуальной сетью.
> * Подключение к виртуальной машине и проверка IP-адреса NAT.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

В следующем примере создается группа ресурсов с именем **myResourceGroupNAT** в расположении **eastus2**:

```azurepowershell-interactive
$rsg = @{
    Name = 'myResourceGroupNAT'
    Location = 'eastus2'
}
New-AzResourceGroup @rsg
```
## <a name="create-the-nat-gateway"></a>Создание шлюза NAT

Мы создадим в этом разделе шлюз NAT и вспомогательные ресурсы.

* Для доступа к Интернету требуется один или несколько общедоступных IP-адресов для шлюза NAT. Выполните командлет [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress), чтобы создать ресурс общедоступного IP-адреса с именем **myPublicIP** в группе **myResourceGroupNAT**. 

* Создайте глобальный шлюз Azure NAT с помощью командлета [New-AzNatGateway](/powershell/module/az.network/new-aznatgateway). В результате ее выполнения будет создан ресурс шлюза с именем **myNATgateway**, использующий общедоступный IP-адрес **myPublicIP**. Установите время ожидания простоя на 10 минут.  

* Создайте виртуальную сеть с именем **myVnet** и подсетью **mySubnet** с помощью командлета [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) в группе **myResourceGroup**, используя командлет [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). Диапазон IP-адресов для виртуальной сети — **10.1.0.0/16**. Подсеть в виртуальной сети — **10.1.0.0/24**.

* Создайте узел Бастиона Azure с именем **myBastionHost**, чтобы получить доступ к виртуальной машине. Используйте командлет [New-AzBastion](/powershell/module/az.network/new-azbastion), чтобы создать узел-бастион. Чтобы создать общедоступный IP-адрес узла-бастиона, используйте командлет [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

```azurepowershell-interactive
## Create public IP address for NAT gateway ##
$ip = @{
    Name = 'myPublicIP'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicIP = New-AzPublicIpAddress @ip

## Create NAT gateway resource ##
$nat = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Name = 'myNATgateway'
    IdleTimeoutInMinutes = '10'
    Sku = 'Standard'
    Location = 'eastus2'
    PublicIpAddress = $publicIP
}
$natGateway = New-AzNatGateway @nat

## Create subnet config and associate NAT gateway to subnet##
$subnet = @{
    Name = 'mySubnet'
    AddressPrefix = '10.1.0.0/24'
    NatGateway = $natGateway
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

```

## <a name="virtual-machine"></a>Виртуальная машина

Используя сведения из этого раздела, вы создадите виртуальную машину для тестирования шлюза NAT и проверки общедоступного IP-адреса исходящего подключения.

* Создайте сетевой интерфейс с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface).

* Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential).

* Создайте виртуальную машину с помощью следующих командлетов:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)
    
```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'myResourceGroupNAT'

## Create network interface for virtual machine. ##
$nic = @{
    Name = "myNicVM"
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Subnet = $vnet.Subnets[0]
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM"
    VMSize = 'Standard_DS1_v2'  
}
$vmos = @{
    ComputerName = "myVM"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    VM = $vmConfig
}
New-AzVM @vm

```

Прежде чем перейти к следующему разделу, дождитесь завершения создания виртуальной машины.

## <a name="test-nat-gateway"></a>Тестирование шлюза NAT

Используя сведения из этого раздела, мы протестируем шлюз NAT. Сначала мы определим общедоступный IP-адрес шлюза NAT. Затем мы подключимся к тестовой виртуальной машине и проверим исходящее подключение через шлюз NAT.
    
1. Войдите на [портал Azure](https://portal.azure.com)

1. Найдите общедоступный IP-адрес для шлюза NAT на экране **обзора**. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем — **myPublicIP**.

2. Запишите общедоступный IP-адрес:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/find-public-ip.png" alt-text="Определение общедоступного IP-адреса шлюза NAT" border="true":::

3. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем в списке ресурсов выберите виртуальную машину **myVM**, расположенную в группе ресурсов **myResourceGroupNAT**.

4. На странице **Обзор** выберите **Подключиться** и **Бастион**.

5. Нажмите синюю кнопку **Использовать Бастион**.

6. Введите имя пользователя и пароль, введенные в процессе создания виртуальной машины.

7. На виртуальной машине **myTestVM** откройте браузер **Internet Explorer**.

8. В адресной строке введите **https://whatsmyip.com** .

9. Убедитесь, что отображаемый IP-адрес соответствует адресу шлюза NAT, записанному на предыдущем этапе:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/my-ip.png" alt-text="Внешний исходящий IP-адрес в Internet Explorer" border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь продолжать использовать это приложение, удалите виртуальную сеть, виртуальную машину и шлюз NAT, выполнив следующие действия:

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'myResourceGroupNAT' -Force
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о NAT виртуальных сетей см. в следующей статье:
> [!div class="nextstepaction"]
> [Общие сведения о NAT виртуальных сетей](nat-overview.md)
