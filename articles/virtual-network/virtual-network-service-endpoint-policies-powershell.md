---
title: Ограничение утечка данных в службе хранилища Azure Azure PowerShell
description: Из этой статьи вы узнаете, как ограничить и ограничить утечка данных виртуальной сети ресурсами хранилища Azure с помощью политик конечной точки службы виртуальной сети, используя Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: RDhillon
manager: narayan
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: ''
ms.openlocfilehash: d8e300c9be8f3e59dc9443bf1f1806e4228992ad
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790399"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-azure-powershell"></a>Управление данными утечка в учетные записи хранения Azure с помощью политик конечной точки службы виртуальной сети с использованием Azure PowerShell

Политики конечной точки службы виртуальной сети позволяют применять управление доступом к учетным записям хранения Azure из виртуальной сети через конечные точки службы. Это ключ к обеспечению безопасности рабочих нагрузок, управлению учетными записями хранения и разрешениим утечка данных.
Вы узнаете, как выполнять следующие задачи:

* Создайте виртуальную сеть.
* Добавьте подсеть и включите конечную точку службы для хранилища Azure.
* Создайте две учетные записи хранения Azure и разрешите сетевой доступ к ней из созданной выше подсети.
* Создайте политику конечной точки службы, чтобы разрешить доступ только к одной из учетных записей хранения.
* Разверните виртуальную машину (ВМ) в подсети.
* Подтвердите доступ к разрешенной учетной записи хранения из подсети.
* Убедитесь, что доступ запрещен учетной записи хранения, неразрешенной из подсети.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Перед созданием виртуальной сети необходимо создать для нее группу ресурсов и другие компоненты, указанные в этой статье. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов *myResourceGroup*. 

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS
```

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork* и префиксом адреса *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

## <a name="enable-a-service-endpoint"></a>Включение конечной точки службы

Создайте подсеть в виртуальной сети. В этом примере создается подсеть *Private* с конечной точкой службы для службы *Microsoft.Storage*. 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-for-the-subnet"></a>Ограничение сетевого доступа для подсети

Создайте правила безопасности группы безопасности сети с помощью [New-азнетворксекуритирулеконфиг](/powershell/module/az.network/new-aznetworksecurityruleconfig). Приведенное ниже правило разрешает исходящий доступ к общедоступным IP-адресам, назначенным службе хранилища Azure. 

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 -Protocol * `
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
  -Priority 110 -Protocol * `
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
  -AddressPrefix 10.0.0.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-to-azure-storage-accounts"></a>Ограничение сетевого доступа к учетным записям хранения Azure

Действия, необходимые для ограничения сетевого доступа к ресурсам, созданным с помощью служб Azure, использующих конечные точки службы, отличаются в зависимости службы. Ознакомьтесь с документацией по отдельным службам, чтобы получить точные инструкции для них. В оставшейся части этой статьи в качестве примера приведены инструкции по ограничению сетевого доступа для учетной записи хранения Azure.

### <a name="create-two-storage-accounts"></a>Создание двух учетных записей хранения

Создайте учетную запись хранения Azure с помощью [New-азсторажеаккаунт](/powershell/module/az.storage/new-azstorageaccount).

```azurepowershell-interactive
$storageAcctName1 = 'allowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName1 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

После создания учетной записи хранения получите ключ для учетной записи хранения в переменной с помощью [Get-азсторажеаккаунткэй](/powershell/module/az.storage/get-azstorageaccountkey):

```azurepowershell-interactive
$storageAcctKey1 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName1).Value[0]
```

Этот ключ потребуется для создания файлового ресурса на более позднем этапе. Введите `$storageAcctKey` и запишите значение, так как позже его потребуется ввести вручную при подключении файлового ресурса как диска на виртуальной машине.

Теперь повторите описанные выше действия, чтобы создать вторую учетную запись хранения.

```azurepowershell-interactive
$storageAcctName2 = 'notallowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName2 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Также извлеките ключ учетной записи хранения из этой учетной записи, чтобы использовать его позже для создания общей папки.

```azurepowershell-interactive
$storageAcctKey2 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName2).Value[0]
```

### <a name="create-a-file-share-in-each-of-the-storage-account"></a>Создайте общую папку в каждой учетной записи хранения.

Создайте контекст для учетной записи хранения и ключ с помощью [New-азсторажеконтекст](/powershell/module/az.storage/new-AzStoragecontext). Этот контекст инкапсулирует имя учетной записи хранения и ее ключ.

```azurepowershell-interactive
$storageContext1 = New-AzStorageContext $storageAcctName1 $storageAcctKey1

$storageContext2 = New-AzStorageContext $storageAcctName2 $storageAcctKey2
```

Создайте общую папку с помощью [New-азсторажешаре](/powershell/module/az.storage/new-azstorageshare):

```azurepowershell-interactive
$share1 = New-AzStorageShare my-file-share -Context $storageContext1

$share2 = New-AzStorageShare my-file-share -Context $storageContext2
```

### <a name="deny-all-network-access-to-a-storage-accounts"></a>Запрет всех сетевых прав доступа к учетным записям хранения

По умолчанию учетные записи хранения принимают сетевые подключения клиентов в любой сети. Чтобы ограничить доступ к выбранным сетям, измените действие по умолчанию на *запретить* с помощью [Update-азсторажеаккаунтнетворкрулесет](/powershell/module/az.storage/update-azstorageaccountnetworkruleset). После запрещения сетевого доступа учетная запись хранения не будет доступна из любой сети.

```azurepowershell-interactive
Update-AzStorageAccountNetworkRuleSet `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -DefaultAction Deny

Update-AzStorageAccountNetworkRuleSet  `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -DefaultAction Deny
```

### <a name="enable-network-access-only-from-the-vnet-subnet"></a>Включение сетевого доступа только из подсети VNet

Получите созданную виртуальную сеть с помощью [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork) , а затем извлеките объект частной подсети в переменную с помощью [Get-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/get-azvirtualnetworksubnetconfig):

```azurepowershell-interactive
$privateSubnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Name myVirtualNetwork `
  | Get-AzVirtualNetworkSubnetConfig -Name Private
```

Разрешите сетевой доступ к учетной записи хранения из *частной* подсети с помощью [Add-азсторажеаккаунтнетворкруле](/powershell/module/az.network/add-aznetworksecurityruleconfig).

```azurepowershell-interactive
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -VirtualNetworkResourceId $privateSubnet.Id

Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>Применение политики для разрешения доступа к допустимой учетной записи хранения

Чтобы пользователи виртуальной сети могли получить доступ только к надежным и разрешенным учетным записям хранения Azure, можно создать политику конечной точки службы со списком разрешенных учетных записей хранения в определении. Затем эта политика применяется к подсети виртуальной сети, которая подключена к хранилищу через конечные точки службы.

### <a name="create-a-service-endpoint-policy"></a>Создание политики конечной точки службы.

В этом разделе создается определение политики со списком разрешенных ресурсов для доступа к конечной точке службы.

Получение идентификатора ресурса для первой (допустимой) учетной записи хранения 

```azurepowershell-interactive
$resourceId = (Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name $storageAcctName1).id
```

Создайте определение политики, чтобы разрешить указанный выше ресурс.

```azurepowershell-interactive
$policyDefinition = New-AzServiceEndpointPolicyDefinition -Name mypolicydefinition `
  -Description "Service Endpoint Policy Definition" `
  -Service "Microsoft.Storage" `
  -ServiceResource $resourceId
```

Создайте политику конечной точки службы с помощью созданного выше определения политики.

```azurepowershell-interactive
$sepolicy = New-AzServiceEndpointPolicy -ResourceGroupName myresourcegroup `
  -Name mysepolicy -Location EastUS
  -ServiceEndpointPolicyDefinition $policyDefinition
```

### <a name="associate-the-service-endpoint-policy-to-the-virtual-network-subnet"></a>Связывание политики конечной точки службы с подсетью виртуальной сети

После создания политики конечной точки службы ее необходимо связать с целевой подсетью с конфигурацией конечной точки службы для хранилища Azure.

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -NetworkSecurityGroup $nsg `
  -ServiceEndpoint Microsoft.Storage `
  -ServiceEndpointPolicy $sepolicy

$virtualNetwork | Set-AzVirtualNetwork
```
## <a name="validate-access-restriction-to-azure-storage-accounts"></a>Проверка ограничения доступа для учетных записей хранения Azure

### <a name="deploy-the-virtual-machine"></a>Развертывание виртуальной машины

Чтобы проверить сетевой доступ к учетной записи хранения, разверните виртуальную машину в подсети.

Создайте виртуальную машину в *частной* подсети с помощью [New-AzVM](/powershell/module/az.compute/new-azvm). При выполнении следующей команды будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины. Параметр `-AsJob` позволяет создать виртуальную машину в фоновом режиме, чтобы можно было перейти к следующему шагу.

```azurepowershell-interactive
New-AzVm -ResourceGroupName myresourcegroup `
  -Location "East US" `
  -VirtualNetworkName myVirtualNetwork `
  -SubnetName Private `
  -Name "myVMPrivate" -AsJob
```

Будут возвращены выходные данные, как показано ниже.

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```

### <a name="confirm-access-to-the-allowed-storage-account"></a>Подтверждение доступа к *разрешенной* учетной записи хранения

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

На виртуальной машине *myVmPrivate* сопоставьте файловый ресурс Azure с разрешенной учетной записью хранения с диском Z с помощью PowerShell. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList ("Azure\allowedaccount"), $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\allowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

PowerShell вернет выходные данные, как в следующем примере.

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\allowedaccount.file.core.windows.net\my-f...
```

Файловый ресурс Azure успешно подключен как диск Z.

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPrivate*.

### <a name="confirm-access-is-denied-to-non-allowed-storage-account"></a>Подтверждение отказа в доступе к учетной записи хранения, которая *не разрешена*

На той же виртуальной машине *myVmPrivate* попытайтесь соотнести файловый ресурс Azure с диском X. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\notallowedaccount", $acctKey
New-PSDrive -Name X -PSProvider FileSystem -Root "\\notallowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

Доступ к файловому ресурсу будет запрещен, и отобразится сообщение об ошибке `New-PSDrive : Access is denied`. Отказано в доступе, так как учетная запись хранения *ноталловедаккаунт* не входит в список разрешенных ресурсов в политике конечных точек службы. 

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPublic*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы применили политику конечной точки службы к службе хранилища Azure для конечной точки службы виртуальной сети Azure. Вы создали учетные записи хранения Azure и ограниченный сетевой доступ только к определенным учетным записям хранения (и, таким же, отклонили другие) из подсети виртуальной сети. Дополнительные сведения о политиках конечной точки службы см. в статье [Общие сведения о политиках конечных точек служб](virtual-network-service-endpoint-policies-overview.md).
