---
title: Связывание общедоступного IP-адреса с виртуальной машиной
titlesuffix: Azure Virtual Network
description: Узнайте, как связать общедоступный IP-адрес с виртуальной машиной (ВМ) с помощью портал Azure или Azure CLI.
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2019
ms.author: allensu
ms.openlocfilehash: 6ea16da3844b8098d87d65e1016f92c69ae34067
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945149"
---
# <a name="associate-a-public-ip-address-to-a-virtual-machine"></a>Связывание общедоступного IP-адреса с виртуальной машиной

Из этой статьи вы узнаете, как связать общедоступный IP-адрес с существующей виртуальной машиной (ВМ). Если вы хотите подключиться к виртуальной машине из Интернета, виртуальная машина должна иметь связанный с ней общедоступный IP-адрес. Если вы хотите создать новую виртуальную машину с общедоступным IP-адресом, это можно сделать с помощью [портал Azure](virtual-network-deploy-static-pip-arm-portal.md), [интерфейса командной строки Azure (CLI)](virtual-network-deploy-static-pip-arm-cli.md)или [PowerShell](virtual-network-deploy-static-pip-arm-ps.md). За общедоступные IP-адреса взимается номинальная плата. Подробные сведения см. в разделе [Цены](https://azure.microsoft.com/pricing/details/ip-addresses/). Количество общедоступных IP-адресов, которые можно использовать для каждой подписки, ограничено. Дополнительные сведения см. в разделе [ограничения](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#publicip-address).

Для связывания общедоступного IP-адреса с виртуальной машиной можно использовать [портал Azure](#azure-portal), [интерфейс командной строки](#azure-cli) Azure (CLI) или [PowerShell](#powershell) .

## <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите на виртуальную машину, в которую необходимо добавить общедоступный IP-адрес, или найдите ее, а затем выберите ее.
3. В разделе **Параметры** выберите **сеть**, а затем выберите сетевой интерфейс, к которому нужно добавить общедоступный IP-адрес, как показано на следующем рисунке:

   ![Выбор сетевого интерфейса](./media/associate-public-ip-address-vm/select-nic.png)

   > [!NOTE]
   > Общедоступные IP-адреса связаны с сетевыми интерфейсами, подключенными к виртуальной машине. На предыдущем рисунке виртуальная машина имеет только один сетевой интерфейс. Если виртуальная машина имеет несколько сетевых интерфейсов, они отображались бы и вы выбрали сетевой интерфейс, к которому нужно связать общедоступный IP-адрес.

4. Выберите **IP-** конфигурации, а затем выберите конфигурацию IP-адресов, как показано на следующем рисунке:

   ![Выберите IP-конфигурацию](./media/associate-public-ip-address-vm/select-ip-configuration.png)

   > [!NOTE]
   > Общедоступные IP-адреса связаны с IP-конфигурациями для сетевого интерфейса. На предыдущем рисунке сетевой интерфейс имеет одну IP-конфигурацию. Если сетевой интерфейс имеет несколько IP-конфигураций, все они будут отображаться в списке, а также будет выбрана IP-конфигурация, которой нужно связать общедоступный IP-адрес.

5. Выберите **включено**, а затем выберите **IP-адрес (*настроить обязательные параметры*)**. Выберите существующий общедоступный IP-адрес, который автоматически закроет поле **выберите общедоступный IP-адрес** . Если в списке нет доступных общедоступных IP-адресов, необходимо создать его. Дополнительные сведения см. в разделе [Создание общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address). Выберите **сохранить**, как показано на следующем рисунке, а затем закройте поле для IP-конфигурации.

   ![Включить общедоступный IP-адрес](./media/associate-public-ip-address-vm/enable-public-ip-address.png)

   > [!NOTE]
   > Отобразятся общедоступные IP-адреса, которые находятся в том же регионе, что и виртуальная машина. Если в регионе создано несколько общедоступных IP-адресов, здесь будут отображаться все. Если они неактивны, это связано с тем, что адрес уже связан с другим ресурсом.

6. Просмотрите общедоступный IP-адрес, назначенный IP-конфигурации, как показано на рисунке ниже. Отображение IP-адреса может занять несколько секунд.

   ![Просмотр назначенного общедоступного IP-адреса](./media/associate-public-ip-address-vm/view-assigned-public-ip-address.png)

   > [!NOTE]
   > Адрес назначается из пула адресов, используемого в каждом регионе Azure. Список пулов адресов, используемых в каждом регионе, см. в разделе [Microsoft Azure IP ranges Center](https://www.microsoft.com/download/details.aspx?id=41653). Назначенный адрес может быть любым адресом в пулах, используемых для региона. Если требуется назначить адрес из определенного пула в регионе, используйте [префикс общедоступного IP-адреса](public-ip-address-prefix.md).

7. [Разрешить сетевой трафик к виртуальной машине](#allow-network-traffic-to-the-vm) с помощью правил безопасности в группе безопасности сети.

## <a name="azure-cli"></a>Azure CLI

Установите [Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) или используйте Azure Cloud Shell. Azure Cloud Shell — это бесплатная оболочка Bash, которую можно запускать непосредственно на портале Azure. Она включает предварительно установленный интерфейс Azure CLI и настроена для использования с вашей учетной записью. Нажмите кнопку **Попробовать** в следующих командах интерфейса командной строки. Кнопка **Попробовать** вызывает Cloud Shell, с помощью которого можно войти в учетную запись Azure.

1. При локальном использовании интерфейса командной строки в Bash войдите в Azure с помощью `az login`.
2. Общедоступный IP-адрес связан с конфигурацией IP сетевого интерфейса, подключенного к виртуальной машине. Используйте команду [AZ Network NIC-IP-config Update](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-update) , чтобы связать общедоступный IP-адрес с IP-конфигурацией. В следующем примере существующий общедоступный IP-адрес, именуемый *myVMPublicIP* , связывается с IP-конфигурацией с именем *ипконфигмивм* существующего сетевого интерфейса с именем *myVMVMNic* , который существует в группе ресурсов с именем *myResourceGroup*.
  
   ```azurecli-interactive
   az network nic ip-config update \
     --name ipconfigmyVM \
     --nic-name myVMVMNic \
     --resource-group myResourceGroup \
     --public-ip-address myVMPublicIP
   ```

   - Если у вас нет общедоступного IP-адреса, создайте его с помощью команды [AZ Network public-IP Create](/cli/azure/network/public-ip#az-network-public-ip-create) . Например, следующая команда создает общедоступный IP-адрес с именем *myVMPublicIP* в группе ресурсов с именем *myResourceGroup*.
  
     ```azurecli-interactive
     az network public-ip create --name myVMPublicIP --resource-group myResourceGroup
     ```

     > [!NOTE]
     > Предыдущая команда создает общедоступный IP-адрес со значениями по умолчанию для нескольких параметров, которые можно настроить. Дополнительные сведения о параметрах общедоступного IP-адреса см. в разделе [Создание общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address). Адрес назначается из пула общедоступных IP-адресов, используемых для каждого региона Azure. Список пулов адресов, используемых в каждом регионе, см. в разделе [Microsoft Azure IP ranges Center](https://www.microsoft.com/download/details.aspx?id=41653).

   - Если вы не знаете имя сетевого интерфейса, подключенного к виртуальной машине, воспользуйтесь командой [az vm nic list](/cli/azure/vm/nic#az-vm-nic-list). Например, следующая команда выводит список имен сетевых интерфейсов, подключенных к виртуальной машине *myVM* в группе ресурсов *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру:
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     В предыдущем примере *myVMVMNic* — это имя сетевого интерфейса.

   - Если вы не знаете имя конфигурации IP для сетевого интерфейса, используйте команду [az network nic ip-config list](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-list). Например, следующая команда выводит список имен конфигурации IP для сетевого интерфейса, подключенного к виртуальной машине *myVMVMNic* в группе ресурсов *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

3. Просмотрите общедоступный IP-адрес, назначенный конфигурации IP-адресов, с помощью команды [AZ VM List-IP-](/cli/azure/vm#az-vm-list-ip-addresses) addresss. В следующем примере показаны IP-адреса, назначенные существующей виртуальной машине с именем *myVM* в группе ресурсов с именем *myResourceGroup*.

   ```azurecli-interactive
   az vm list-ip-addresses --name myVM --resource-group myResourceGroup --out table
   ```

   > [!NOTE]
   > Адрес назначается из пула адресов, используемого в каждом регионе Azure. Список пулов адресов, используемых в каждом регионе, см. в разделе [Microsoft Azure IP ranges Center](https://www.microsoft.com/download/details.aspx?id=41653). Назначенный адрес может быть любым адресом в пулах, используемых для региона. Если требуется назначить адрес из определенного пула в регионе, используйте [префикс общедоступного IP-адреса](public-ip-address-prefix.md).

4. [Разрешить сетевой трафик к виртуальной машине](#allow-network-traffic-to-the-vm) с помощью правил безопасности в группе безопасности сети.

## <a name="powershell"></a>PowerShell

Установите [PowerShell](/powershell/azure/install-az-ps) или используйте Azure Cloud Shell. Azure Cloud Shell — это бесплатная оболочка, которую можно запускать непосредственно на портале Azure. Он включает предварительно установленный PowerShell и настроен для использования с вашей учетной записью. Нажмите кнопку **Попробовать** в следующих командах PowerShell. Кнопка **Попробовать** вызывает Cloud Shell, с помощью которого можно войти в учетную запись Azure.

1. Если вы используете PowerShell локально, войдите в Azure с помощью `Connect-AzAccount`.
2. Общедоступный IP-адрес связан с конфигурацией IP сетевого интерфейса, подключенного к виртуальной машине. Используйте команды [Get-азвиртуалнетворк](/powershell/module/Az.Network/Get-AzVirtualNetwork) и [Get-азвиртуалнетворксубнетконфиг](/powershell/module/Az.Network/Get-AzVirtualNetworkSubnetConfig) , чтобы получить виртуальную сеть и подсеть, в которой находится сетевой интерфейс. Затем используйте команду [Get-азнетворкинтерфаце](/powershell/module/Az.Network/Get-AzNetworkInterface) , чтобы получить сетевой интерфейс и команду [Get-азпублиЦипаддресс](/powershell/module/az.network/get-azpublicipaddress) , чтобы получить существующий общедоступный IP-адрес. Затем с помощью команды [Set-азнетворкинтерфацеипконфиг](/powershell/module/Az.Network/Set-AzNetworkInterfaceIpConfig) свяжите общедоступный IP-адрес с IP-конфигурацией и команду [Set-азнетворкинтерфаце](/powershell/module/Az.Network/Set-AzNetworkInterface) , чтобы записать новую IP-конфигурацию в сетевой интерфейс.

   В следующем примере существующий общедоступный IP-адрес, именуемый *myVMPublicIP* , связывается с IP-конфигурацией с именем *ипконфигмивм* существующего сетевого интерфейса с именем *myVMVMNic* , который существует в подсети с именем *мивмсубнет* в виртуальной сети с именем *myVMVNet*. Все ресурсы находятся в группе ресурсов с именем *myResourceGroup*.
  
   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name myVMVNet -ResourceGroupName myResourceGroup
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name myVMSubnet -VirtualNetwork $vnet
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $pip = Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup
   $nic | Set-AzNetworkInterfaceIpConfig -Name ipconfigmyVM -PublicIPAddress $pip -Subnet $subnet
   $nic | Set-AzNetworkInterface
   ```

   - Если у вас нет общедоступного IP-адреса, создайте его с помощью команды [New-азпублиЦипаддресс](/powershell/module/Az.Network/New-AzPublicIpAddress) . Например, следующая команда создает *динамический* общедоступный IP-адрес с именем *myVMPublicIP* в группе ресурсов с именем *myResourceGroup* в регионе *eastus* .
  
     ```azurepowershell-interactive
     New-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup -AllocationMethod Dynamic -Location eastus
     ```

     > [!NOTE]
     > Предыдущая команда создает общедоступный IP-адрес со значениями по умолчанию для нескольких параметров, которые можно настроить. Дополнительные сведения о параметрах общедоступного IP-адреса см. в разделе [Создание общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address). Адрес назначается из пула общедоступных IP-адресов, используемых для каждого региона Azure. Список пулов адресов, используемых в каждом регионе, см. в разделе [Microsoft Azure IP ranges Center](https://www.microsoft.com/download/details.aspx?id=41653).

   - Если вы не знаете имя сетевого интерфейса, подключенного к виртуальной машине, воспользуйтесь командой [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM). Например, следующая команда выводит список имен сетевых интерфейсов, подключенных к виртуальной машине *myVM* в группе ресурсов *myResourceGroup*:

     ```azurepowershell-interactive
     $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
     $vm.NetworkProfile
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *myVMVMNic* — это имя сетевого интерфейса.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Если вы не знакомы с именем виртуальной сети или подсети, в которой находится сетевой интерфейс, используйте `Get-AzNetworkInterface` команду для просмотра информации. Например, следующая команда возвращает сведения о виртуальной сети и подсети для сетевого интерфейса с именем *myVMVMNic* в группе ресурсов с именем *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $ipConfigs = $nic.IpConfigurations
     $ipConfigs.Subnet | Select Id
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *myVMVNET* — это имя виртуальной сети, а *мивмсубнет* — имя подсети.
  
     ```
     "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVMVNET/subnets/myVMSubnet",
     ```

   - Если вы не знаете имя конфигурации IP для сетевого интерфейса, используйте команду [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface). Например, следующая команда выводит список имен конфигурации IP для сетевого интерфейса, подключенного к виртуальной машине *myVMVMNic* в группе ресурсов *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *ipconfigmyVM* — это имя конфигурации IP.
  
     ```
     Id     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM
     ```

3. Просмотрите общедоступный IP-адрес, назначенный IP-конфигурации, с помощью команды [Get-азпублиЦипаддресс](/powershell/module/az.network/get-azpublicipaddress) . В следующем примере показан адрес, назначенный общедоступному IP-адресу с именем *myVMPublicIP* в группе ресурсов с именем *myResourceGroup*.

   ```azurepowershell-interactive
   Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup | Select IpAddress
   ```

   Если вы не знакомы с именем общедоступного IP-адреса, назначенного IP-конфигурации, выполните следующие команды, чтобы получить его:

   ```azurepowershell-interactive
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $nic.IPConfigurations
   $address = $nic.IPConfigurations.PublicIpAddress
   $address | Select Id
   ```

   Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *myVMPublicIP* — это имя общедоступного IP-адреса, НАЗНАЧЕННОГО IP-конфигурации.

   ```
   "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP"
   ```

   > [!NOTE]
   > Адрес назначается из пула адресов, используемого в каждом регионе Azure. Список пулов адресов, используемых в каждом регионе, см. в разделе [Microsoft Azure IP ranges Center](https://www.microsoft.com/download/details.aspx?id=41653). Назначенный адрес может быть любым адресом в пулах, используемых для региона. Если требуется назначить адрес из определенного пула в регионе, используйте [префикс общедоступного IP-адреса](public-ip-address-prefix.md).

4. [Разрешить сетевой трафик к виртуальной машине](#allow-network-traffic-to-the-vm) с помощью правил безопасности в группе безопасности сети.

## <a name="allow-network-traffic-to-the-vm"></a>Разрешение сетевого трафика к виртуальной машине

Прежде чем подключиться к общедоступному IP-адресу из Интернета, убедитесь, что в каждой группе безопасности сети, связанной с сетевым интерфейсом, открыты необходимые порты, подсеть, в которой находится сетевой интерфейс, или оба. Несмотря на то, что группы безопасности фильтруют трафик по частному IP-адресу сетевого интерфейса, после того, как входящий Интернет-трафик поступает по общедоступному, Azure преобразует общедоступный адрес в частный IP-адрес, поэтому если группа безопасности сети предотвращает поток трафика, связь с общедоступным IP-адресом завершается ошибкой. Действующие правила безопасности для сетевого интерфейса и его подсети можно просмотреть с помощью [портала](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), [CLI](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) или [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="next-steps"></a>Дальнейшие действия

Разрешите входящий Интернет-трафик к виртуальной машине с помощью группы безопасности сети. Сведения о создании группы безопасности сети см. в статье [Работа с группами безопасности сети](manage-network-security-group.md#work-with-network-security-groups). Дополнительные сведения о группах безопасности сети см. в разделе [группы безопасности](./network-security-groups-overview.md).
