---
title: Отмена связи общедоступного IP-адреса с виртуальной машиной Azure
titlesuffix: Azure Virtual Network
description: Узнайте, как отменить связь общедоступного IP-адреса с виртуальной машиной
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/04/2019
ms.author: allensu
ms.openlocfilehash: e9bfadd3e2453f0241dc2f7b8bfa5c964333bcf5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776546"
---
# <a name="dissociate-a-public-ip-address-from-an-azure-vm"></a>Отмена связи общедоступного IP-адреса с виртуальной машиной Azure 

Из этой статьи вы узнаете, как отменить связь с общедоступным IP-адресом из виртуальной машины Azure.

Для отмены связи общедоступного IP-адреса с виртуальной машиной можно использовать [портал Azure](#azure-portal), [интерфейс командной строки](#azure-cli) Azure (Azure CLI) или [PowerShell](#powershell).

## <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
2. Найдите и выберите виртуальную машину, для которой необходимо отменить связь с общедоступным IP-адресом.
3. На странице виртуальной машины нажмите **Обзор** и выберите общедоступный IP-адрес, как показано на следующем рисунке:

   ![Выбор общедоступного IP-адреса](./media/remove-public-ip-address/remove-public-ip-address-2.png)

4. На странице общедоступного IP-адреса нажмите **Обзор** и выберите **Отменить связь**, как показано на следующем рисунке:

    ![Отмена связи общедоступного IP-адреса](./media/remove-public-ip-address/remove-public-ip-address-3.png)

5. В разделе **Отмена связи общедоступного IP-адреса** нажмите **Да**.

## <a name="azure-cli"></a>Azure CLI

Установите [Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) или используйте Azure Cloud Shell. Azure Cloud Shell — это бесплатная оболочка Bash, которую можно запускать непосредственно на портале Azure. Она включает предварительно установленный интерфейс Azure CLI и настроена для использования с вашей учетной записью. Нажмите кнопку **Попробовать** в следующих командах интерфейса командной строки. Кнопка **Попробовать** вызывает Cloud Shell, с помощью которого можно войти в учетную запись Azure.

1. При локальном использовании интерфейса командной строки в Bash войдите в Azure с помощью `az login`.
2. Общедоступный IP-адрес связан с конфигурацией IP сетевого интерфейса, подключенного к виртуальной машине. Чтобы отменить связь между общедоступным IP-адресом и конфигурацией IP, используйте команду [az network nic-ip-config update](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_update). В следующем примере отменяется связь общедоступного IP-адреса *myVMPublicIP* с конфигурацией IP *ipconfigmyVM* существующего сетевого интерфейса *myVMVMNic*, подключенного к виртуальной машине *myVM* в группе ресурсов с именем *myResourceGroup*.
  
   ```azurecli-interactive
    az network nic ip-config update \
    --name ipconfigmyVM \
    --resource-group myResourceGroup \
    --nic-name myVMVMNic \
    --remove PublicIpAddress
   ```

   Если вы не знаете имя сетевого интерфейса, подключенного к виртуальной машине, воспользуйтесь командой [az vm nic list](/cli/azure/vm/nic#az_vm_nic_list). Например, следующая команда выводит список имен сетевых интерфейсов, подключенных к виртуальной машине *myVM* в группе ресурсов *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру:
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     В предыдущем примере *myVMVMNic* — это имя сетевого интерфейса.

   - Если вы не знаете имя конфигурации IP для сетевого интерфейса, используйте команду [az network nic ip-config list](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_list). Например, следующая команда выводит список имен конфигурации общедоступных IP-адресов для сетевого интерфейса, подключенного к виртуальной машине *myVMVMNic* в группе ресурсов *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

   - Если вы не знаете имя конфигурации общедоступного IP-адреса для сетевого интерфейса, используйте команду [az network nic ip-config show](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_show). Например, следующая команда выводит список имен конфигурации общедоступных IP-адресов для сетевого интерфейса, подключенного к виртуальной машине *myVMVMNic* в группе ресурсов *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config show --name ipconfigmyVM --nic-name myVMVMNic --resource-group myResourceGroup --query publicIPAddress.id
     ```


## <a name="powershell"></a>PowerShell

Установите [PowerShell](/powershell/azure/install-az-ps) или используйте Azure Cloud Shell. Azure Cloud Shell — это бесплатная оболочка, которую можно запускать непосредственно на портале Azure. Он включает предварительно установленный PowerShell и настроен для использования с вашей учетной записью. Нажмите кнопку **Попробовать** в следующих командах PowerShell. Кнопка **Попробовать** вызывает Cloud Shell, с помощью которого можно войти в учетную запись Azure.

1. Если вы используете PowerShell локально, войдите в Azure с помощью `Connect-AzAccount`.
2. Общедоступный IP-адрес связан с конфигурацией IP сетевого интерфейса, подключенного к виртуальной машине. Для получения сетевого интерфейса используйте команду [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface). Задайте для параметра "Общедоступный IP-адрес" значение NULL, а затем используйте команду [Set-AzNetworkInterface](/powershell/module/Az.Network/Set-AzNetworkInterface), чтобы записать новую конфигурацию IP в сетевой интерфейс.

   В следующем примере отменяется связь общедоступного IP-адреса *myVMPublicIP* с сетевым интерфейсом *myVMVMNic*, подключенным к виртуальной машине *myVM*. Все ресурсы находятся в группе ресурсов с именем *myResourceGroup*.
  
   ```azurepowershell
    $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroup myResourceGroup
    $nic.IpConfigurations.publicipaddress.id = $null
    Set-AzNetworkInterface -NetworkInterface $nic
   ```

  - Если вы не знаете имя сетевого интерфейса, подключенного к виртуальной машине, воспользуйтесь командой [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM). Например, следующая команда выводит список имен сетевых интерфейсов, подключенных к виртуальной машине *myVM* в группе ресурсов *myResourceGroup*:

    ```azurepowershell
    $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
    $vm.NetworkProfile
    ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *myVMVMNic* — это имя сетевого интерфейса.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Если вы не знаете имя конфигурации IP для сетевого интерфейса, используйте команду [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface). Например, следующая команда выводит список имен конфигурации IP для сетевого интерфейса, подключенного к виртуальной машине *myVMVMNic* в группе ресурсов *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations.id
     ```

     Выходные данные включают одну или несколько строк, аналогично следующему примеру. В примере выходных данных *ipconfigmyVM* — это имя конфигурации IP.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM"
     ```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [связать общедоступный IP-адрес с виртуальной машиной](associate-public-ip-address-vm.md).
