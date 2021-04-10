---
title: Краткое руководство. Создание виртуальной машины Linux с помощью Azure PowerShell
description: Из этого краткого руководства вы узнаете, как с помощью Azure PowerShell создать виртуальную машину Linux.
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/31/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 2cef611fe79ca04303840076b09b4cf6344b7e7d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102616236"
---
# <a name="quickstart-create-a-linux-virtual-machine-in-azure-with-powershell"></a>Краткое руководство. Создание виртуальной машины Linux в Azure с помощью PowerShell

Модуль PowerShell используется для создания ресурсов Azure и управления ими с помощью командной строки PowerShell или сценариев. В этом кратком руководстве показано, как с помощью модуля Azure PowerShell развернуть в Azure виртуальную машину Linux. В этом кратком руководстве используется образ Marketplace Ubuntu 18.04 LTS от Canonical. Чтобы проверить работу виртуальной машины, вы также подключитесь к ней по протоколу SSH и установите веб-сервер NGINX.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="create-ssh-key-pair"></a>Создание пары ключей SSH

Выполните команду [ssh-keygen](https://www.ssh.com/ssh/keygen/), чтобы создать пару ключей SSH. Если у вас уже есть эта пара, можно перейти к следующему шагу.


```azurepowershell-interactive
ssh-keygen -t rsa -b 4096
```

Вам будет предложено указать имя файла для пары ключей, или вы можете нажать клавишу **ВВОД**, чтобы использовать расположение `/home/<username>/.ssh/id_rsa` по умолчанию. При необходимости вы также сможете создать пароль для ключей.

Подробные сведения о создании пар ключей SSH см. в статье [Как использовать ключи SSH с Windows в Azure](ssh-from-windows.md).

Если вы создали пару ключей SSH при помощи Cloud Shell, она будет сохранена в [учетной записи хранения, которую Cloud Shell создает автоматически](../../cloud-shell/persisting-shell-storage.md). Не удаляйте учетную запись хранения или общую папку в ней, пока вы не получите ключи, иначе вы потеряете доступ к виртуальной машине. 

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-virtual-network-resources"></a>Создание ресурсов виртуальной сети

Создайте виртуальную сеть, подсеть и общедоступный IP-адрес. Эти ресурсы используются для того, чтобы установить сетевое подключение к виртуальной машине и подключить ее к Интернету.

```azurepowershell-interactive
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myVNET" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"
```

Создайте группу безопасности сети Azure и правило трафика. Группа безопасности сети защищает виртуальную машину с помощью правил для входящего и исходящего трафика. В следующем примере создается правило входящего подключения для TCP-порта 22, который используется для подключений по протоколу SSH. Чтобы разрешить входящий веб-трафик, также создается правило для входящего трафика по TCP-порту 80.

```azurepowershell-interactive
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleSSH"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 22 `
  -Access "Allow"

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleWWW"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1001 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access "Allow"

# Create a network security group
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myNetworkSecurityGroup" `
  -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

Создайте виртуальный сетевой адаптер с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). Виртуальный сетевой адаптер подключает виртуальную машину к подсети, группе безопасности сети и общедоступному IP-адресу.

```azurepowershell-interactive
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface `
  -Name "myNic" `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Чтобы создать виртуальную машину в PowerShell, создайте конфигурацию, у которой будут такие параметры, как используемый образ, размер и параметры проверки подлинности. Затем эта конфигурация используется для создания виртуальной машины.

Задайте учетные данные для SSH, сведения об ОС и размер виртуальной машины. В этом примере ключ SSH хранится в `~/.ssh/id_rsa.pub`. 

```azurepowershell-interactive
# Define a credential object
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig `
  -VMName "myVM" `
  -VMSize "Standard_D1" | `
Set-AzVMOperatingSystem `
  -Linux `
  -ComputerName "myVM" `
  -Credential $cred `
  -DisablePasswordAuthentication | `
Set-AzVMSourceImage `
  -PublisherName "Canonical" `
  -Offer "UbuntuServer" `
  -Skus "18.04-LTS" `
  -Version "latest" | `
Add-AzVMNetworkInterface `
  -Id $nic.Id

# Configure the SSH key
$sshPublicKey = cat ~/.ssh/id_rsa.pub
Add-AzVMSshPublicKey `
  -VM $vmconfig `
  -KeyData $sshPublicKey `
  -Path "/home/azureuser/.ssh/authorized_keys"
```

Теперь объедините описанные выше определения конфигурации в командлете создания [New-AzVM](/powershell/module/az.compute/new-azvm):

```azurepowershell-interactive
New-AzVM `
  -ResourceGroupName "myResourceGroup" `
  -Location eastus -VM $vmConfig
```

Развертывание виртуальной машины может занять несколько минут. После завершения развертывания перейдите к следующему разделу.

## <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

Установите SSH-подключение к виртуальной машине, используя общедоступный IP-адрес. Чтобы узнать общедоступный IP-адрес виртуальной машины, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress):

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

Для запуска сеанса SSH вставьте следующую команду в ту же оболочку, которую вы использовали для создания пары ключей SSH. Замените *10.111.12.123* IP-адресом вашей виртуальной машины.

```bash
ssh azureuser@10.111.12.123
```

При появлении запроса на имя пользователя для входа введите *azureuser*. Если используется парольная фраза с ключами SSH, предоставьте ее в ответ на соответствующий запрос.


## <a name="install-nginx"></a>Установка nginx

Чтобы проверить работу виртуальной машины, установите веб-сервер NGINX. Из сеанса SSH обновите источники пакетов, а затем установите последнюю версию пакета NGINX.

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

После этого введите `exit`, чтобы выйти из сеанса SSH.


## <a name="view-the-web-server-in-action"></a>Проверка работы веб-сервера

Страницу приветствия NGINX по умолчанию можно просмотреть в любом веб-браузере. Введите общедоступный IP-адрес виртуальной машины в качестве веб-адреса. Общедоступный IP-адрес можно найти на странице общих сведений о виртуальной машине. Также он является частью строки подключения SSH, использованной ранее.

![Страница приветствия NGINX по умолчанию](./media/quick-create-cli/nginix-welcome-page.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководство вы развернули простую виртуальную машину, создали группу безопасности сети и правило для нее, а также установили простой веб-сервер. Дополнительные сведения о виртуальных машинах Azure см. в руководстве для виртуальных машин Linux.

> [!div class="nextstepaction"]
> [Создание виртуальных машин Linux и управление ими с помощью Azure CLI](./tutorial-manage-vm.md)
