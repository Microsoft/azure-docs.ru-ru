---
title: Создание частной конечной точки с помощью Azure PowerShell
description: Создание частной конечной точки для Аттестации Azure с помощью Azure PowerShell
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 03/26/2021
ms.author: mbaldwin
ms.openlocfilehash: 8ff2e73a8557c6b1761c852ac58a46037a122ddb
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105628532"
---
# <a name="quickstart-create-a-private-endpoint-using-azure-powershell"></a>Краткое руководство. Создание частной конечной точки с помощью Azure PowerShell

Чтобы начать работу с Приватным каналом Azure, воспользуйтесь частной конечной точкой для безопасного подключения к Аттестации Azure.

Из этого краткого руководства вы узнаете, как создать частную конечную точку для Аттестации Azure и развернуть виртуальную машину для тестирования частного подключения.  

> [!NOTE]
> Текущая реализация включает только параметр автоматического утверждения. Чтобы продолжить создание частной конечной точки, нужно добавить подписку в список разрешений. Прежде чем продолжить, обратитесь к группе обслуживания или отправьте запрос на поддержку Azure на [странице поддержки Azure](https://azure.microsoft.com/support/options/).

## <a name="prerequisites"></a>Предварительные требования

* Подробнее о [Приватном канале Azure](../private-link/private-link-overview.md)
* [Настройка службы "Аттестация Azure" с помощью Azure PowerShell](./quickstart-powershell.md)

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
## Create to your Azure account subscription and create a resource group in a desired location. ##
Connect-AzAccount
Set-AzSubscription “mySubscription”
$rg = “CreateAttestationPrivateLinkTutorial-rg”
$loc= "eastus”
New-AzResourceGroup -Name $rg -Location $loc
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Создание виртуальной сети и узла бастиона

В этом разделе вы создадите виртуальную сеть, подсеть и узел бастиона. 

Для безопасного подключения к виртуальной машине с целью тестирования частной конечной точки будет использоваться узел бастиона.

Создание виртуальной сети и узла бастиона с помощью:

* [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)
* [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress)
* [New-AzBastion](/powershell/module/az.network/new-azbastion).

```azurepowershell-interactive
## Create backend subnet config. ##
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name myBackendSubnet -AddressPrefix 10.0.0.0/24

## Create Azure Bastion subnet. ##
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig -Name AzureBastionSubnet -AddressPrefix 10.0.1.0/24

## Create the virtual network. ##
$vnet = New-AzVirtualNetwork -Name "myAttestationTutorialVNet" -ResourceGroupName $rg -Location $loc -AddressPrefix "10.0.0.0/16" -Subnet $subnetConfig, $bastsubnetConfig

## Create public IP address for bastion host. ##
$publicip = New-AzPublicIpAddress -Name "myBastionIP" -ResourceGroupName $rg -Location $loc -Sku "Standard" -AllocationMethod "Static"

## Create bastion host ##
New-AzBastion -ResourceGroupName $rg -Name "myBastion" -PublicIpAddress $publicip -VirtualNetwork $vnet
```

Развертывание узла-бастиона может занять несколько минут.

## <a name="create-test-virtual-machine"></a>Создание тестовой виртуальной машины

В этом разделе вы создадите виртуальную машину, которая будет использоваться для тестирования частной конечной точки.

Создайте виртуальную машину с помощью следующих командлетов:

  * [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential)
  * [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 
  * [New-AzVM](/powershell/module/az.compute/new-azvm)
  * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
  * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
  * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
  * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)


```azurepowershell-interactive
## Set credentials for server admin and password. ##
$cred = Get-Credential

## Command to create network interface for VM ##
$nicVM = New-AzNetworkInterface -Name "myNicVM" -ResourceGroupName $rg -Location $loc -Subnet $vnet.Subnets[0] 

## Create a virtual machine configuration.##
$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_DS1_v2" | Set-AzVMOperatingSystem -Windows -ComputerName "myVM" -Credential $cred | Set-AzVMSourceImage -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2019-Datacenter" -Version "latest" | Add-AzVMNetworkInterface -Id $nicVM.Id 

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig
```

## <a name="create-an-attestation-provider"></a>Создание поставщика аттестации

```azurepowershell-interactive
## Create an attestation provider ##
$attestationProviderName = "myattestationprovider"
$attestationProvider = New-AzAttestation -Name $attestationProviderName -ResourceGroupName $rg -Location $loc
$attestationProviderId = $attestationProvider.Id

## Access the attestation provider from local machine ##
Enter nslookup <provider-name>.attest.azure.net. Replace <provider-name> with the name of the attestation provider instance you created in the previous steps. 

You'll receive a message similar to what is displayed below:

## PowerShell copy. ##
nslookup myattestationprovider.eus.attest.azure.net

Server:  cdns01.comcast.net
Address:  2001:558:feed::1

Non-authoritative answer:
Name:    eus.service.attest.azure.net
Address:  20.62.219.160
Aliases:  myattestationprovider.eus.attest.azure.net
    attesteusatm.trafficmanager.net
```

## <a name="create-private-endpoint"></a>Создание частной конечной точки

В рамках этого раздела вы создадите частную конечную точку и подключение с помощью:

* [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/New-AzPrivateLinkServiceConnection)
* [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint)

```azurepowershell-interactive
## Create private endpoint connection. ##
$privateEndpointConnection = New-AzPrivateLinkServiceConnection -Name "myConnection" -PrivateLinkServiceId $attestationProviderId -GroupID "Standard"

## Disable private endpoint network policy ##
 $vnet.Subnets[0].PrivateEndpointNetworkPolicies = "Disabled" 
$vnet | Set-AzVirtualNetwork

## Create private endpoint
New-AzPrivateEndpoint  -ResourceGroupName $rg -Name "myPrivateEndpoint" -Location $loc -Subnet $vnet.Subnets[0] -PrivateLinkServiceConnection $privateEndpointConnection
```
## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS

В рамках этого раздела вы создадите и настроите частную зону DNS с помощью:

* [New-AzPrivateDnsZone](/powershell/module/az.privatedns/new-azprivatednszone)
* [New-AzPrivateDnsVirtualNetworkLink](/powershell/module/az.privatedns/new-azprivatednsvirtualnetworklink)
* [New-AzPrivateDnsZoneConfig](/powershell/module/az.network/new-azprivatednszoneconfig)
* [New-AzPrivateDnsZoneGroup](/powershell/module/az.network/new-azprivatednszonegroup)

```azurepowershell-interactive
## Create private dns zone. ##
$zone = New-AzPrivateDnsZone -ResourceGroupName $rg -Name "privatelink.attest.azure.net"

## Create dns network link. ##
$link = New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $rg -ZoneName "privatelink.attest.azure.net" -Name "myLink" -VirtualNetworkId $vnet.Id

## Create DNS configuration ##
$config = New-AzPrivateDnsZoneConfig -Name "privatelink.attest.azure.net" -PrivateDnsZoneId $zone.ResourceId

## Create DNS zone group. ##
New-AzPrivateDnsZoneGroup -ResourceGroupName $rg -PrivateEndpointName "myPrivateEndpoint" -Name "myZoneGroup" -PrivateDnsZoneConfig $config
```

## <a name="test-connectivity-to-private-endpoint"></a>Проверка подключения к частной конечной точке

В этом разделе описано, как использовать виртуальную машину, созданную на предыдущем шаге, для подключения к серверу SQL через частную конечную точку.

1. Войдите на [портал Azure](https://portal.azure.com) 
 
2. В области навигации слева выберите **Группы ресурсов**.

3. Выберите **CreateAttestationPrivateLinkTutorial-rg**.

4. Выберите **myVM**.

5. На обзорной странице для **myVM** выберите **Подключиться**, а затем — **Бастион**.

6. Нажмите синюю кнопку **Использовать Бастион**.

7. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

8. После подключения откройте на сервере Windows PowerShell.

9. Введите `nslookup <provider-name>.attest.azure.net`. Замените **\<provider-name>** именем экземпляра поставщика аттестации, созданного на предыдущих шагах. Вы получите сообщение, аналогичное показанному ниже:

    ```powershell

    ## Access the attestation provider from local machine ##
    nslookup myattestationprovider.eus.attest.azure.net

    Server:  cdns01.comcast.net
    Address:  2001:558:feed::1

    cdns01.comcast.net can't find myattestationprovider.eus.attest.azure.net: Non-existent domain

    ## Access the attestation provider from the VM created in the same virtual network as the private endpoint.   ##
    nslookup myattestationprovider.eus.attest.azure.net

    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    myattestationprovider.eastus.test.attest.azure.net
    ```
