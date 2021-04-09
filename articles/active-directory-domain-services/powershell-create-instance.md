---
title: Включение доменных служб Azure AD с помощью PowerShell | Документация Майкрософт
description: Узнайте, как настраивать и включать доменные службы Azure Active Directory с помощью Azure AD PowerShell и Azure PowerShell.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: d4bc5583-6537-4cd9-bc4b-7712fdd9272a
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 03/10/2021
ms.author: justinha
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8056e95b731b1818e10d7415cb813d6aba0ec7fa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103149107"
---
# <a name="enable-azure-active-directory-domain-services-using-powershell"></a>Включение доменных служб Azure Active Directory с помощью PowerShell

Доменные службы Azure Active Directory (Azure AD DS) предоставляют управляемые доменные службы, отвечающие за присоединение к домену, применение групповой политики, использование протокола LDAP, а также выполнение аутентификации Kerberos или NTLM (полностью поддерживается Windows Server Active Directory). Эти доменные службы можно использовать без необходимости развертывать, администрировать и обновлять контроллеры домена. Azure AD DS интегрируется с существующим клиентом Azure AD. Такая интеграция позволяет пользователям входить в систему с корпоративными учетными данными. При этом вы можете использовать существующие группы и учетные записи пользователей для защиты доступа к ресурсам.

В этой статье описано, как включить Azure AD DS с помощью PowerShell.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей требуются следующие ресурсы:

* Установите и настройте Azure PowerShell.
    * При необходимости выполните инструкции по [установке модуля Azure PowerShell и его подключению к подписке Azure](/powershell/azure/install-az-ps).
    * Войдите в подписку Azure с помощью командлета [Connect-AzAccount][Connect-AzAccount].
* Установите и настройте Azure AD PowerShell.
    * При необходимости выполните инструкции по [установке модуля Azure AD PowerShell и подключению к Azure AD](/powershell/azure/active-directory/install-adv2).
    * Войдите в клиент Azure AD с помощью командлета [Connect-AzureAD][Connect-AzureAD].
* Привилегии *глобального администратора* для клиента Azure AD, чтобы включить доменные службы Azure AD.
* Для создания нужных ресурсов Azure AD DS требуются привилегии *участника* в подписке Azure.

  > [!IMPORTANT]
  > Так как модуль PowerShell **Az.ADDomainServices** предоставляется в предварительной версии, его нужно установить отдельно с помощью командлета `Install-Module`.

  ```azurepowershell-interactive
  Install-Module -Name Az.ADDomainServices
  ```

## <a name="create-required-azure-ad-resources"></a>Создание необходимых ресурсов Azure AD

Доменным службам Azure Active Directory требуется субъект-служба для аутентификации и взаимодействия, а также группа Azure AD для определения того, какие пользователи имеют административные разрешения в управляемом домене.

Сначала создайте субъект-службу Azure AD, используя конкретный идентификатор приложения с именем *Службы контроллера домена*. В общедоступном облаке Azure используется значение идентификатора *2565bd9d-da50-47d4-8b85-4c97f669dc36*. В других облаках — значение *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. Не изменяйте этот идентификатор приложения.

Создайте субъект-службу Azure AD с помощью командлета [New-AzureADServicePrincipal][New-AzureADServicePrincipal]:

```powershell
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
```

Создайте группу Azure AD с именем *Администраторы контроллера домена AAD*. Пользователи, добавленные в эту группу, получают разрешения на выполнение задач администрирования в управляемом домене.

Сначала получите объект *Администраторы контроллера домена AAD* с помощью командлета [Get-AzureADGroup][Get-AzureADGroup]: Если группа не существует, создайте ее с помощью группы *Администраторы контроллера домена AAD* и командлета [New-AzureADGroup][New-AzureADGroup]:

```powershell
# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# If the group doesn't exist, create it
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}
```

Создав группу *Администраторы контроллера домена AAD*, получите идентификатор объекта нужного пользователя с помощью командлета [Get-AzureADUser][Get-AzureADUser], а затем добавьте пользователя в группу с помощью командлета [Add-AzureADGroupMember][Add-AzureADGroupMember].

В приведенном ниже примере показан идентификатор объекта пользователя для учетной записи с именем участника-пользователя `admin@contoso.onmicrosoft.com`. Используйте вместо этой учетной записи пользователя имя участника-пользователя, которого нужно добавить в группу *Администраторы контроллера домена AAD*

```powershell
# Retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Сначала зарегистрируйте поставщика ресурсов доменных служб Azure AD с помощью командлета [Register-AzResourceProvider][Register-AzResourceProvider]:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

Затем создайте группу ресурсов с помощью командлета [New-AzResourceGroup][New-AzResourceGroup]. В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе *westus*. Используйте желаемое имя и регион:

```azurepowershell-interactive
$ResourceGroupName = "myResourceGroup"
$AzureLocation = "westus"

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation
```

Создайте виртуальную сеть и подсети для доменных служб Azure AD. Создаются две подсети — для доменных служб (*DomainServices*) и для рабочих нагрузок (*Workloads*). Службы Azure AD DS развертываются в выделенной подсети *DomainServices*. Не развертывайте другие приложения или рабочие нагрузки в этой подсети. Используйте отдельные подсети *Workloads* или другие подсети для остальных виртуальных машин.

Создайте подсети с помощью командлета [New-AzVirtualNetworkSubnetConfig][New-AzVirtualNetworkSubnetConfig], а затем создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork][New-AzVirtualNetwork].

```azurepowershell-interactive
$VnetName = "myVnet"

# Create the dedicated subnet for Azure AD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -AddressPrefix 10.0.0.0/24

# Create an additional subnet for your own VM workloads
$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet= New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location westus `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
```

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Azure AD DS требуется группа безопасности сети, чтобы защитить порты, необходимые для управляемого домена, и блокировать весь остальной входящий трафик. [Группа безопасности сети][nsg-overview] содержит перечень правил, которые разрешают или запрещают передачу трафика в виртуальную сеть Azure. В Azure AD DS группа безопасности сети выступает в качестве дополнительного уровня защиты для блокировки доступа к управляемому домену. Дополнительные сведения см. в разделе [Группы безопасности сети и необходимые порты][network-ports].

Указанные ниже командлеты PowerShell используют [New-AzNetworkSecurityRuleConfig][New-AzNetworkSecurityRuleConfig] для создания правил, а затем [New-AzNetworkSecurityGroup][New-AzNetworkSecurityGroup] для создания группы безопасности сети. Затем группа безопасности сети и правила связываются с подсетью виртуальной сети с помощью командлета [Set-AzVirtualNetworkSubnetConfig][Set-AzVirtualNetworkSubnetConfig].

```azurepowershell-interactive
$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

## <a name="create-a-managed-domain"></a>Создание управляемого домена

Теперь мы переходим к созданию управляемого домена. Задайте идентификатор подписки Azure, а затем укажите имя управляемого домена, например *aaddscontoso.com*. Вы можете получить идентификатор подписки с помощью командлета [Get-AzSubscription][Get-AzSubscription].

Если вы выбрали регион, который поддерживает зоны доступности, ресурсы Azure AD DS распределяются между зонами для дополнительной избыточности.

Зоны доступности — уникальные физические расположения в пределах одного региона Azure. Каждая зона состоит из одного или нескольких центров обработки данных, оснащенных независимыми системами электроснабжения, охлаждения и сетевого взаимодействия. Чтобы обеспечить устойчивость, во всех включенных областях используются минимум три отдельные зоны.

Вы не можете настроить распределение Azure AD DS между зонами. Платформа Azure автоматически обрабатывает распределение ресурсов зоны. См. [дополнительные сведения о зонах доступности и регионах][availability-zones].

```azurepowershell-interactive
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Enable Azure AD Domain Services for the directory.
$replicaSetParams = @{
  Location = $AzureLocation
  SubnetId = "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"
}
$replicaSet = New-AzADDomainServiceReplicaSetObject @replicaSetParams

$domainServiceParams = @{
  Name = $ManagedDomainName
  ResourceGroupName = $ResourceGroupName
  DomainName = $ManagedDomainName
  ReplicaSet = $replicaSet
}
New-AzADDomainService @domainServiceParams
```

Создание ресурса и возврат управления в командную строку PowerShell занимает несколько минут. Подготовка к работе управляемого домена продолжается в фоновом режиме. На завершение развертывания может понадобиться до часа. На портале Azure на странице **Обзор** для управляемого домена на этом этапе развертывания отображаются сведения о текущем состоянии.

Когда на портале Azure отобразятся сведения о том, что подготовка управляемого домена к работе завершена, выполните следующие задачи.

* Обновите параметры DNS для виртуальной сети, чтобы виртуальные машины могли найти управляемый домен для присоединения к нему или для аутентификации.
    * Чтобы настроить DNS, выберите на портале управляемый домен. В окне **Обзор** отобразится запрос на автоматическую настройку этих параметров DNS.
* [Включите синхронизацию паролей с Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds), чтобы пользователи могли входить в управляемый домен с помощью рабочих учетных данных.

## <a name="complete-powershell-script"></a>Полный скрипт PowerShell

Приведенный ниже полный скрипт PowerShell объединяет все задачи, описанные в этой статье. Скопируйте его и сохраните в текстовый файл с расширением `.ps1`. Запустите скрипт в локальной консоли PowerShell или [Azure Cloud Shell][cloud-shell].

> [!NOTE]
> Чтобы включить Azure AD DS, нужно быть глобальным администратором для клиента Azure AD. Кроме того, понадобятся разрешения не ниже уровня *Участник* в подписке Azure.

```azurepowershell-interactive
# Change the following values to match your deployment.
$AaddsAdminUserUpn = "admin@contoso.onmicrosoft.com"
$ResourceGroupName = "myResourceGroup"
$VnetName = "myVnet"
$AzureLocation = "westus"
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Connect to your Azure AD directory.
Connect-AzureAD

# Login to your Azure subscription.
Connect-AzAccount

# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"

# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Create the delegated administration group for Azure AD Domain Services if it doesn't already exist.
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq '$AaddsAdminUserUpn'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId

# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation

# Create the dedicated subnet for AAD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $AzureLocation `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet

$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork

# Enable Azure AD Domain Services for the directory.
$replicaSetParams = @{
  Location = $AzureLocation
  SubnetId = "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"
}
$replicaSet = New-AzADDomainServiceReplicaSetObject @replicaSetParams

$domainServiceParams = @{
  Name = $ManagedDomainName
  ResourceGroupName = $ResourceGroupName
  DomainName = $ManagedDomainName
  ReplicaSet = $replicaSet
}
New-AzADDomainService @domainServiceParams
```

Создание ресурса и возврат управления в командную строку PowerShell занимает несколько минут. Подготовка к работе управляемого домена продолжается в фоновом режиме. На завершение развертывания может понадобиться до часа. На портале Azure на странице **Обзор** для управляемого домена на этом этапе развертывания отображаются сведения о текущем состоянии.

Когда на портале Azure отобразятся сведения о том, что подготовка управляемого домена к работе завершена, выполните следующие задачи.

* Обновите параметры DNS для виртуальной сети, чтобы виртуальные машины могли найти управляемый домен для присоединения к нему или для аутентификации.
    * Чтобы настроить DNS, выберите на портале управляемый домен. В окне **Обзор** отобразится запрос на автоматическую настройку этих параметров DNS.
* [Включите синхронизацию паролей с Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds), чтобы пользователи могли входить в управляемый домен с помощью рабочих учетных данных.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы увидеть управляемый домен в действии, [присоедините к нему виртуальную машину Windows][windows-join], настройте [защищенный протокол LDAP][tutorial-ldaps] и [синхронизацию хэша паролей][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[nsg-overview]: ../virtual-network/network-security-groups-overview.md
[network-ports]: network-considerations.md#network-security-groups-and-required-ports

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[New-AzVirtualNetworkSubnetConfig]: /powershell/module/Az.Network/New-AzVirtualNetworkSubnetConfig
[New-AzVirtualNetwork]: /powershell/module/Az.Network/New-AzVirtualNetwork
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: ../cloud-shell/cloud-shell-windows-users.md
[availability-zones]: ../availability-zones/az-overview.md
[New-AzNetworkSecurityRuleConfig]: /powershell/module/az.network/new-aznetworksecurityruleconfig
[New-AzNetworkSecurityGroup]: /powershell/module/az.network/new-aznetworksecuritygroup
[Set-AzVirtualNetworkSubnetConfig]: /powershell/module/az.network/set-azvirtualnetworksubnetconfig
