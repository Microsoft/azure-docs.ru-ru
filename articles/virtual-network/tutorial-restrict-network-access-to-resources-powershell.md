---
title: Ограничение сетевого доступа к ресурсам PaaS — Azure PowerShell
description: Из этой статьи вы узнаете, как ограничить сетевой доступ к ресурсам Azure, таким как служба хранилища Azure и служба "База данных SQL Azure", с помощью конечных точек служб для виртуальной сети и Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6770486158b9c5f2e896951d91ff41643b6c8813
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790183"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-powershell"></a>Ограничение сетевого доступа к ресурсам PaaS посредством конечных точек служб для виртуальной сети с помощью PowerShell

Конечные точки служб для виртуальной сети позволяют ограничить сетевой доступ к некоторым ресурсам службы Azure определенной подсетью виртуальной сети. Можно также запретить доступ к ресурсам через Интернет. Конечные точки службы предоставляют прямое подключение из виртуальной сети к поддерживаемым службам Azure. Это позволяет использовать закрытый диапазон адресов виртуальной сети для доступа к службам Azure. Трафик, поступающий к ресурсам Azure через конечные точки службы, всегда остается в магистральной сети Microsoft Azure. Вы узнаете, как выполнять следующие задачи:

* Создание виртуальной сети с одной подсетью.
* Добавление подсети и включение конечной точки службы.
* Создание ресурса Azure и разрешение сетевого доступа к нему только из подсети.
* Развертывание виртуальной машины в каждой подсети.
* Подтверждение прав доступа к ресурсу из подсети.
* Подтверждение запрета доступа к ресурсу из подсети и Интернета.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Перед созданием виртуальной сети необходимо создать для нее группу ресурсов и другие компоненты, указанные в этой статье. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов *myResourceGroup*. 

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork* и префиксом адреса *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Создайте конфигурацию подсети с помощью [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). В следующем примере создается конфигурация подсети *Public*.

```azurepowershell-interactive
$subnetConfigPublic = Add-AzVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

Создайте подсеть в виртуальной сети, записав конфигурацию подсети в виртуальную сеть с помощью [Set-азвиртуалнетворк](/powershell/module/az.network/Set-azVirtualNetwork):

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="enable-a-service-endpoint"></a>Включение конечной точки службы

Вы можете включить конечные точки службы только для служб, поддерживающих эту функцию. Просмотрите службы с поддержкой конечных точек службы, доступные в расположении Azure, с помощью [Get-азвиртуалнетворкаваилаблиндпоинтсервице](/powershell/module/az.network/get-azvirtualnetworkavailableendpointservice). В следующем примере возвращается список служб с поддержкой конечных точек службы, доступных в регионе *eastus*. Список возвращаемых служб со временем будет увеличиваться, так как поддержка конечных точек службы будет реализовываться во все большем числе служб Azure.

```azurepowershell-interactive
Get-AzVirtualNetworkAvailableEndpointService -Location eastus | Select Name
```

Создайте дополнительную подсеть в виртуальной сети. В этом примере создается подсеть *Private* с конечной точкой службы для службы *Microsoft.Storage*. 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-for-a-subnet"></a>Ограничение сетевого доступа для подсети

Создайте правила безопасности группы безопасности сети с помощью [New-азнетворксекуритирулеконфиг](/powershell/module/az.network/new-aznetworksecurityruleconfig). Приведенное ниже правило разрешает исходящий доступ к общедоступным IP-адресам, назначенным службе хранилища Azure. 

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 `
  -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

Следующее правило запрещает доступ ко всем общедоступным IP-адресам. Предыдущее правило переопределяет это правило ввиду более высокого приоритета. Оно предоставляет доступ к общедоступным IP-адресам службы хранилища Azure.

```azurepowershell-interactive
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name Deny-Internet-All `
  -Access Deny `
  -DestinationAddressPrefix Internet `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 110 `
  -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

Следующее правило разрешает для подсети входящий трафик протокола удаленного рабочего стола (RDP), поступающий из любого расположения. В подсети разрешены подключения к удаленному рабочему столу, чтобы позже можно было подтвердить сетевой доступ к ресурсу.

```azurepowershell-interactive
$rule3 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-RDP-All `
  -Access Allow `
  -DestinationAddressPrefix VirtualNetwork `
  -DestinationPortRange 3389 `
  -Direction Inbound `
  -Priority 120 `
  -Protocol * `
  -SourceAddressPrefix * `
  -SourcePortRange *
```

Создайте группу безопасности сети с помощью командлета [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). В следующем примере создается группа безопасности сети *myNsgPrivate*.

```azurepowershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myNsgPrivate `
  -SecurityRules $rule1,$rule2,$rule3
```

Свяжите группу безопасности сети с *частной* подсетью с помощью [Set-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) , а затем запишите конфигурацию подсети в виртуальную сеть. В следующем примере подсеть *Private* привязывается к группе безопасности сети *myNsgPrivate*.

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-to-a-resource"></a>Ограничение сетевого доступа к ресурсу

Действия, необходимые для ограничения сетевого доступа к ресурсам, созданным с помощью служб Azure, использующих конечные точки службы, отличаются в зависимости службы. Ознакомьтесь с документацией по отдельным службам, чтобы получить точные инструкции для них. В оставшейся части этой статьи в качестве примера приведены инструкции по ограничению сетевого доступа для учетной записи хранения Azure.

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте учетную запись хранения Azure с помощью [New-азсторажеаккаунт](/powershell/module/az.storage/new-azstorageaccount). Замените `<replace-with-your-unique-storage-account-name>` именем, которое является уникальным для всех расположений Azure, содержащим только цифры и строчные буквы (длиной от 3 до 24 знаков).

```azurepowershell-interactive
$storageAcctName = '<replace-with-your-unique-storage-account-name>'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

После создания учетной записи хранения получите ключ для учетной записи хранения в переменной с помощью [Get-азсторажеаккаунткэй](/powershell/module/az.storage/get-azstorageaccountkey):

```azurepowershell-interactive
$storageAcctKey = (Get-AzStorageAccountKey `
  -ResourceGroupName myResourceGroup `
  -AccountName $storageAcctName).Value[0]
```

Этот ключ потребуется для создания файлового ресурса на более позднем этапе. Введите `$storageAcctKey` и запишите значение, так как позже его потребуется ввести вручную при подключении файлового ресурса как диска на виртуальной машине.

### <a name="create-a-file-share-in-the-storage-account"></a>Создание файлового ресурса в учетной записи хранения

Создайте контекст для учетной записи хранения и ключ с помощью [New-азсторажеконтекст](/powershell/module/az.storage/new-AzStoragecontext). Этот контекст инкапсулирует имя учетной записи хранения и ее ключ.

```azurepowershell-interactive
$storageContext = New-AzStorageContext $storageAcctName $storageAcctKey
```

Создайте общую папку с помощью [New-азсторажешаре](/powershell/module/az.storage/new-azstorageshare):

$share = New-AzStorageShare My-файл-Share-context $storageContext

### <a name="deny-all-network-access-to-a-storage-account"></a>Запрет любого сетевого доступа к учетной записи хранения

По умолчанию учетные записи хранения принимают сетевые подключения клиентов в любой сети. Чтобы ограничить доступ к выбранным сетям, измените действие по умолчанию на *запретить* с помощью [Update-азсторажеаккаунтнетворкрулесет](/powershell/module/az.storage/update-azstorageaccountnetworkruleset). После запрещения сетевого доступа учетная запись хранения не будет доступна из любой сети.

```azurepowershell-interactive
Update-AzStorageAccountNetworkRuleSet  `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -DefaultAction Deny
```

### <a name="enable-network-access-from-a-subnet"></a>Включение сетевого доступа из подсети

Получите созданную виртуальную сеть с помощью [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork) , а затем извлеките объект частной подсети в переменную с помощью [Get-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/get-azvirtualnetworksubnetconfig):

```azurepowershell-interactive
$privateSubnet = Get-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Name "myVirtualNetwork" `
  | Get-AzVirtualNetworkSubnetConfig `
  -Name "Private"
```

Разрешите сетевой доступ к учетной записи хранения из *частной* подсети с помощью [Add-азсторажеаккаунтнетворкруле](/powershell/module/az.network/add-aznetworksecurityruleconfig).

```azurepowershell-interactive
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Чтобы проверить сетевой доступ к учетной записи хранения, разверните виртуальную машину в каждой подсети.

### <a name="create-the-first-virtual-machine"></a>Создание первой виртуальной машины

Создайте виртуальную машину в подсети *Public* с помощью [New-AzVM](/powershell/module/az.compute/new-azvm). При выполнении следующей команды будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины. Параметр `-AsJob` позволяет создать виртуальную машину в фоновом режиме, чтобы можно было перейти к следующему шагу.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Public" `
    -Name "myVmPublic" `
    -AsJob
```

Будут возвращены выходные данные, как показано ниже.

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command                  
--     ----            -------------   -----         -----------     --------             -------                  
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM     
```

### <a name="create-the-second-virtual-machine"></a>Создание второй виртуальной машины

Создайте виртуальную машину в подсети *Private*.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Private" `
    -Name "myVmPrivate"
```

Создание виртуальной машины в Azure занимает несколько минут. Не переходите к следующему шагу только, пока Azure не завершит создание виртуальной машины и не вернет выходные данные в окне PowerShell.

## <a name="confirm-access-to-storage-account"></a>Подтверждение прав доступа к учетной записи хранения

Чтобы получить общедоступный IP-адрес виртуальной машины, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). Приведенный ниже пример возвращает общедоступный IP-адрес виртуальной машины *myVmPrivate*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Замените `<publicIpAddress>` в следующей команде общедоступным IP-адресом, возвращенным предыдущей командой, а затем введите следующую команду:

```powershell
mstsc /v:<publicIpAddress>
```

Будет создан и скачан на ваш компьютер файл протокола удаленного рабочего стола (RDP-файл). Откройте этот RDP-файл. При появлении запроса выберите **Подключиться**. Введите имя пользователя и пароль, указанные при создании виртуальной машины. Возможно, потребуется выбрать **More choices** (Дополнительные варианты), а затем **Use a different account** (Использовать другую учетную запись), чтобы указать учетные данные, введенные при создании виртуальной машины. Щелкните **ОК**. При входе в систему может появиться предупреждение о сертификате. Если вы получили предупреждение, выберите **Да** или **Продолжить**.

На виртуальной машине *myVmPrivate* с помощью PowerShell подключите файловый ресурс Azure как диск Z. Перед выполнением следующих команд замените `<storage-account-key>` и `<storage-account-name>` собственными значениями или значениями, полученными при [создании учетной записи хранения](#create-a-storage-account).

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```

PowerShell вернет выходные данные, как в следующем примере.

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
```

Файловый ресурс Azure успешно подключен как диск Z.

Убедитесь, что виртуальная машина запрещает любые исходящие подключения с любых других общедоступных IP-адресов.

```powershell
ping bing.com
```

Ответы не отобразятся, так как группа безопасности сети, связанная с подсетью *Private*, не разрешает исходящий доступ к общедоступным IP-адресам, отличным от адреса, назначенного службе хранилища Azure.

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Подтверждение запрета доступа к учетной записи хранения

Получите общедоступный IP-адрес виртуальной машины *myVmPublic*.

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPublic `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Замените `<publicIpAddress>` в следующей команде общедоступным IP-адресом, возвращенным предыдущей командой, а затем введите следующую команду: 

```powershell
mstsc /v:<publicIpAddress>
```

На виртуальной машине *myVmPublic* попытайтесь повторно подключить файловый ресурс Azure как диск Z. Перед выполнением приведенных ниже команд замените `<storage-account-key>` и `<storage-account-name>` собственными значениями или значениями, полученными при [создании учетной записи хранения](#create-a-storage-account).

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```

Доступ к файловому ресурсу будет запрещен, и отобразится сообщение об ошибке `New-PSDrive : Access is denied`. В доступе будет отказано, так как виртуальная машина *myVmPublic* развернута в подсети *Public*. Для подсети *Public* не включена конечная точка службы для службы хранилища Azure, и учетная запись хранения предназначена только для сетевого доступа из подсети *Private*, а не *Public*.

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPublic*.

Используя свой компьютер, попытайтесь просмотреть файловые ресурсы в учетной записи хранения, выполнив следующую команду.

```powershell-interactive
Get-AzStorageFile `
  -ShareName my-file-share `
  -Context $storageContext
```

Отказано в доступе, и вы получаете *Get-азсторажефиле: удаленный сервер вернул ошибку: (403) запрещено. Код состояния HTTP: 403-сообщение об ошибке HTTP: этот запрос не имеет разрешений на выполнение этой операции* , так как ваш компьютер не находится в *частной* подсети виртуальной сети *MyVirtualNetwork* .

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этой статьи вы включили конечную точку службы для подсети виртуальной сети. Вы узнали, что конечные точки службы можно включать для ресурсов, развернутых несколькими службами Azure. Вы создали учетную запись хранения Azure и ограничили сетевой доступ к учетной записи хранения ресурсами одной подсети в виртуальной сети. См. дополнительные сведения о [конечных точках службы](virtual-network-service-endpoints-overview.md) и [управляемых подсетях](virtual-network-manage-subnet.md).

Если в вашей учетной записи имеется несколько виртуальных сетей, можно соединить две виртуальные сети, чтобы ресурсы в каждой из них могли взаимодействовать друг с другом. Дополнительные сведения см. в статье о [подключении к виртуальным сетям](tutorial-connect-virtual-networks-powershell.md).
