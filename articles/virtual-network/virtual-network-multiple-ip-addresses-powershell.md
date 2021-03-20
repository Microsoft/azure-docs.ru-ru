---
title: Назначение виртуальным машинам Azure нескольких IP-адресов с помощью PowerShell | Документация Майкрософт
description: Узнайте, как назначить виртуальной машине несколько IP-адресов с помощью PowerShell. | диспетчер ресурсов
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/24/2017
ms.author: allensu
ms.openlocfilehash: d86d4248b449ad3961a7798fd36a320eb6a74009
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98217079"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-powershell"></a>Назначение виртуальным машинам нескольких IP-адресов с помощью PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

В этой статье описывается создание виртуальной машины с помощью модели развертывания Azure Resource Manager с использованием PowerShell. Для ресурсов, созданных с помощью классической модели развертывания, нельзя назначить несколько IP-адресов. Дополнительные сведения о моделях развертывания Azure см. в статье [Azure Resource Manager vs. classic deployment: Understand deployment models and the state of your resources](../azure-resource-manager/management/deployment-models.md) (Развертывание с помощью Azure Resource Manager и классическое развертывание. Общие сведения о моделях развертывания и состоянии ресурсов).

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name="create-a-vm-with-multiple-ip-addresses"></a><a name = "create"></a>Создание виртуальной машины с несколькими IP-адресами

Вы можете создать пример виртуальной машины с несколькими IP-адресами, как описано в нашем сценарии. Измените имена переменных в соответствии с требованиями своей реализации.

1. Откройте командную строку PowerShell и выполните остальные действия в этом разделе в пределах одного сеанса PowerShell. Если вы еще не установили и не настроили PowerShell, выполните шаги, описанные в статье [Установка и настройка Azure PowerShell](/powershell/azure/) .
2. Войдите в свою учетную запись с помощью команды `Connect-AzAccount`.
3. Замените *myResourceGroup* и *westus* именем и расположением по своему выбору. Создайте группу ресурсов. Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

   ```powershell
   $RgName   = "MyResourceGroup"
   $Location = "westus"

   New-AzResourceGroup `
   -Name $RgName `
   -Location $Location
   ```

4. Создайте виртуальную сеть и подсеть в расположении, где находится группа ресурсов:

   ```powershell

   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

5. Создайте группу безопасности сети (NSG) и правило. Группа NSG защищает виртуальную машину с помощью правил входящего и исходящего трафика. В нашем случае создается правило для входящего трафика порта 3389, которое разрешает входящие подключения к удаленному рабочему столу.

    ```powershell
    
    # Create an inbound network security group rule for port 3389

    $NSGRule = New-AzNetworkSecurityRuleConfig `
    -Name MyNsgRuleRDP `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 1000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow
    
    # Create a network security group
    $NSG = New-AzNetworkSecurityGroup `
    -ResourceGroupName $RgName `
    -Location $Location `
    -Name MyNetworkSecurityGroup `
    -SecurityRules $NSGRule
    ```

6. Определите основную конфигурацию IP для сетевого адаптера. Замените 10.0.0.4 действительным адресом в созданной подсети, если вы не использовали значение, определенное ранее. Прежде чем назначать статический IP-адрес, рекомендуем сначала убедиться, что он не используется. Введите команду `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.4 -VirtualNetwork $VNet`. Если адрес доступен, в выходных данных возвращается значение *True*, Если он недоступен, выходные данные возвращают *значение false* и список доступных адресов. 

    В следующих командах **замените \<replace-with-your-unique-name> уникальным DNS-именем для использования.** Имя должно быть уникальным для всех общедоступных IP-адресов в регионе Azure. Этот параметр является необязательным. Его можно удалить, если вам нужно подключаться к виртуальной машине, используя только общедоступный IP-адрес.

    ```powershell
    
    # Create a public IP address
    $PublicIP1 = New-AzPublicIpAddress `
    -Name "MyPublicIP1" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -DomainNameLabel <replace-with-your-unique-name> `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName1 = "IPConfig-1"
    $IpConfig1     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName1 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.4 `
    -PublicIpAddress $PublicIP1 `
    -Primary
    ```

    При назначении нескольких IP-конфигураций сетевой карте одна конфигурация должна быть назначена в качестве *первичной*.

    > [!NOTE]
    > За общедоступные IP-адреса взимается номинальная плата. Дополнительные сведения о ценах на IP-адреса см. на странице [Цены на IP-адреса](https://azure.microsoft.com/pricing/details/ip-addresses). Число общедоступных IP-адресов, которые можно использовать в одной подписке, ограничено. Сведения об ограничениях см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

7. Определите дополнительную конфигурацию IP для сетевого адаптера. Конфигурации можно добавлять и удалять по мере необходимости. Каждой конфигурации IP должен быть назначен частный IP-адрес. Каждой конфигурации может быть назначен один необязательный общедоступный IP-адрес.

    ```powershell
    
    # Create a public IP address
    $PublicIP2 = New-AzPublicIpAddress `
    -Name "MyPublicIP2" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName2 = "IPConfig-2"
    $IpConfig2     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName2 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.5 `
    -PublicIpAddress $PublicIP2
        
    $IpConfigName3 = "IpConfig-3"
    $IpConfig3 = New-AzNetworkInterfaceIpConfig `
    -Name $IPConfigName3 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.6
    ```

8. Создайте сетевой адаптер и свяжите с ним три конфигурации IP:

   ```powershell
   $NIC = New-AzNetworkInterface `
   -Name MyNIC `
   -ResourceGroupName $RgName `
   -Location $Location `
   -NetworkSecurityGroupId $NSG.Id `
   -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3
   ```

   >[!NOTE]
   >Хотя в этой статье все конфигурации назначены одному сетевому адаптеру, каждому сетевому адаптеру, подключенному к виртуальной машине, можно назначить несколько конфигураций IP. Дополнительные сведения о создании виртуальной машины с несколькими сетевыми интерфейсами см. в [этой статье](../virtual-machines/windows/multiple-nics.md).

9. Создайте виртуальную машину с помощью следующих команд:

    ```powershell
    
    # Define a credential object. When you run these commands, you're prompted to enter a username and password for the VM you're creating.
    $cred = Get-Credential
    
    # Create a virtual machine configuration
    $VmConfig = New-AzVMConfig `
    -VMName MyVM `
    -VMSize Standard_DS1_v2 | `
    Set-AzVMOperatingSystem -Windows `
    -ComputerName MyVM `
    -Credential $cred | `
    Set-AzVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
    Add-AzVMNetworkInterface `
    -Id $NIC.Id
    
    # Create the VM
    New-AzVM `
    -ResourceGroupName $RgName `
    -Location $Location `
    -VM $VmConfig
    ```

10. Добавьте в операционную систему виртуальной машины частные IP-адреса, выполнив шаги, соответствующие вашей операционной системе, которые описаны в разделе [Добавление IP-адресов в операционную систему виртуальной машины](#os-config) этой статьи. Не добавляйте в операционную систему общедоступные IP-адреса.

## <a name="add-ip-addresses-to-a-vm"></a><a name="add"></a>Добавление IP-адресов в виртуальную машину

Выполнив следующие шаги, вы сможете назначить частные или общедоступные IP-адреса сетевому интерфейсу Azure. В примерах, приводимых в следующих разделах, предполагается, что у вас уже есть виртуальная машина с тремя IP-конфигурациями, описанными в [сценарии](#scenario) в этой статье, но это не является обязательным требованием.

1. Откройте командную строку PowerShell и выполните остальные действия в этом разделе в пределах одного сеанса PowerShell. Если вы еще не установили и не настроили PowerShell, выполните шаги, описанные в статье [Установка и настройка Azure PowerShell](/powershell/azure/) .
2. Измените значения переменных со знаком "$" на имя сетевого интерфейса, к которому необходимо добавить IP-адрес, а также имя группы ресурсов и расположения, где находится сетевой интерфейс.

   ```powershell
   $NicName  = "MyNIC"
   $RgName   = "MyResourceGroup"
   $Location = "westus"
   ```

   Если имя сетевого интерфейса, который необходимо изменить, неизвестно, введите следующие команды, а затем измените значения предыдущих переменных.

   ```powershell
   Get-AzNetworkInterface | Format-Table Name, ResourceGroupName, Location
   ```

3. Создайте переменную и назначьте ее существующему сетевому интерфейсу, выполнив следующую команду:

   ```powershell
   $MyNIC = Get-AzNetworkInterface -Name $NicName -ResourceGroupName $RgName
   ```

4. В следующих командах замените параметры *MyVNet* и *MySubnet* именами виртуальной сети и подсети, к которым подключен сетевой адаптер. Чтобы получить объекты виртуальной сети и подсети, к которым подключен сетевой интерфейс, выполните следующие команды:

   ```powershell
   $MyVNet = Get-AzVirtualnetwork -Name MyVNet -ResourceGroupName $RgName
   $Subnet = $MyVnet.Subnets | Where-Object { $_.Name -eq "MySubnet" }
   ```

   Если вы не знаете имя виртуальной сети или подсети, к которым подключен сетевой интерфейс, выполните следующую команду:

   ```powershell
   $MyNIC.IpConfigurations
   ```

   В выходных данных найдите текст, похожий на следующий пример:

   ```
   "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet"
   ```

    В этих выходных данных *MyVnet* — это имя виртуальной сети, а *MySubnet* —имя подсети, к которой подключен сетевой адаптер.

5. Выполните действия, описанные в одном из следующих разделов, с учетом задач.

   **Добавление частного IP-адреса**

   Чтобы добавить к сетевому интерфейсу частный IP-адрес, нужно создать IP-конфигурацию. Следующая команда создает конфигурацию со статическим IP-адресом 10.0.0.7. Указываемый статический IP-адрес должен быть свободен в используемой подсети. Мы советуем сначала проверить доступность адреса. Для этого выполните команду `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet`. Если IP-адрес доступен, в выходных данных возвращается значение *True*, а если нет — значение *False* и список доступных адресов.

   ```powershell
   Add-AzNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
   $MyNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7
   ```

   Вы можете создать любое количество конфигураций, указывая для них уникальные имена и уникальные частные IP-адреса (если используются статические IP-адреса).

   Добавьте в операционную систему виртуальной машины частный IP-адрес, выполнив действия, соответствующие вашей операционной системе, как описано в разделе [Добавление IP-адресов в операционную систему виртуальной машины](#os-config) этой статьи.

   **Добавление общедоступного IP-адреса**

   Чтобы добавить общедоступный IP-адрес, его ресурс необходимо связать с новой или существующей IP-конфигурацией. Выполните шаги, описанные в одном из следующих разделов, в соответствии с задачами.

   > [!NOTE]
   > За общедоступные IP-адреса взимается номинальная плата. Дополнительные сведения о ценах на IP-адреса см. на странице [Цены на IP-адреса](https://azure.microsoft.com/pricing/details/ip-addresses). Число общедоступных IP-адресов, которые можно использовать в одной подписке, ограничено. Сведения об ограничениях см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
   >

   **Связывание ресурса общедоступного IP-адреса с новой IP-конфигурацией**

   Чтобы добавить общедоступный IP-адрес в новую IP-конфигурацию, необходимо добавить и частный IP-адрес, так как все IP-конфигурации должны иметь частный IP-адрес. В конфигурацию можно добавить имеющийся ресурс общедоступного IP-адреса или создать новый. Чтобы создать ресурс, выполните следующую команду:

   ```powershell
   $myPublicIp3 = New-AzPublicIpAddress `
   -Name "myPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location `
   -AllocationMethod Static
   ```

   Чтобы создать новую IP-конфигурацию со статическим частным IP-адресом и связанным ресурсом общедоступного IP-адреса *myPublicIp3* , введите следующую команду:

   ```powershell
   Add-AzNetworkInterfaceIpConfig `
   -Name IPConfig-4 `
   -NetworkInterface $myNIC `
   -Subnet $Subnet `
   -PrivateIpAddress 10.0.0.7 `
   -PublicIpAddress $myPublicIp3
   ```

   **Связывание ресурса общедоступного IP-адреса с имеющейся IP-конфигурацией**

   Ресурс общедоступного IP-адреса может быть связан только с такой IP-конфигурацией, у которой еще нет такого ресурса. Чтобы определить, связан ли с конкретной IP-конфигурацией какой-либо общедоступный IP-адрес, выполните следующую команду:

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

   Должен появиться результат, аналогичный приведенному ниже.

   ```
   Name       PrivateIpAddress PublicIpAddress                                           Primary

   IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
   IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
   IpConfig-3 10.0.0.6                                                                     False
   ```

   Столбец **PublicIpAddress** для IP-конфигурации *IpConfig-3* пуст. Это означает, что в настоящее время с этой конфигурацией не связан никакой общедоступный IP-адрес. Вы можете добавить к конфигурации IpConfig-3 имеющийся ресурс общедоступного IP-адреса или создать новый ресурс, выполнив следующую команду:

   ```powershell
   $MyPublicIp3 = New-AzPublicIpAddress `
   -Name "MyPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location -AllocationMethod Static
   ```

   Введите следующую команду, чтобы связать ресурс общедоступного IP-адреса с существующей IP-конфигурацией с именем *ipconfig-3*:

   ```powershell
   Set-AzNetworkInterfaceIpConfig `
   -Name IpConfig-3 `
   -NetworkInterface $mynic `
   -Subnet $Subnet `
   -PublicIpAddress $myPublicIp3
   ```

6. Настройте для сетевого интерфейса IP-конфигурацию, выполнив следующую команду:

   ```powershell
   Set-AzNetworkInterface -NetworkInterface $MyNIC
   ```

7. Просмотрите частные IP адреса и ресурсы общедоступных IP-адресов, назначенные сетевому адаптеру. Для этого введите следующую команду.

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

8. Добавьте в операционную систему виртуальной машины частный IP-адрес, выполнив действия, соответствующие вашей операционной системе, как описано в разделе [Добавление IP-адресов в операционную систему виртуальной машины](#os-config) этой статьи. Не добавляйте в операционную систему общедоступный IP-адрес.

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]