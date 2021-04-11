---
title: Руководство. Подключение к серверу SQL Azure с использованием частной конечной точки Azure с помощью PowerShell
description: Из этого руководства вы узнаете, как создать сервер SQL Azure с частной конечной точкой с помощью Azure PowerShell
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 10/31/2020
ms.author: allensu
ms.custom: fasttrack-edit
ms.openlocfilehash: 8668bdb9dc391582234bf5741c0dd287d026defd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102554944"
---
# <a name="tutorial-connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-powershell"></a>Руководство. Подключение к серверу SQL Azure с использованием частной конечной точки Azure с помощью Azure PowerShell

Частная конечная точка Azure — ключевой компонент для построения Приватного канала в Azure. Это позволяет ресурсам Azure, таким как виртуальные машины (ВМ), обмениваться данными с ресурсами Приватного канала в частном порядке.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание виртуальной сети и узла бастиона.
> * Создайте виртуальную машину.
> * Создание Azure SQL Server и частной конечной точки.
> * Проверка подключения к частной конечной точке SQL Server.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreateSQLEndpointTutorial-rg' -Location 'eastus'
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
$parameters1 = @{
    Name = 'MyVNet'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    AddressPrefix = '10.0.0.0/16'
    Subnet = $subnetConfig, $bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @parameters1

## Create public IP address for bastion host. ##
$parameters2 = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @parameters2

## Create bastion host ##
$parameters3 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @parameters3
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

## Command to get virtual network configuration. ##
$vnet = Get-AzVirtualNetwork -Name myVNet -ResourceGroupName CreateSQLEndpointTutorial-rg

## Command to create network interface for VM ##
$parameters1 = @{
    Name = 'myNicVM'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
}
$nicVM = New-AzNetworkInterface @parameters1

## Create a virtual machine configuration.##
$parameters2 = @{
    VMName = 'myVM'
    VMSize = 'Standard_DS1_v2'
}
$parameters3 = @{
    ComputerName = 'myVM'
    Credential = $cred
}
$parameters4 = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'
}
$vmConfig = 
New-AzVMConfig @parameters2 | Set-AzVMOperatingSystem -Windows @parameters3 | Set-AzVMSourceImage @parameters4 | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Location 'eastus' -VM $vmConfig
```

## <a name="create-an-azure-sql-server"></a>Создание сервера SQL Azure

В рамках этого раздела вы создадите сервер и базу данных SQL с помощью:

* [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver)
* [New-AzSQlDatabase](/powershell/module/az.sql/new-azsqldatabase).

Создание сервера и базы данных SQL. Замените **\<sql-server-name>** уникальным именем сервера:

```azurepowershell-interactive
## Set and administrator and password for the SQL server. ##
$cred = Get-Credential

## Create SQL server ##
$parameters1 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ServerName = '<sql-server-name>'
    SqlAdministratorCredentials = $cred
    Location = 'eastus'
}
New-AzSqlServer @parameters1

## Create database. ##
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ServerName = '<sql-server-name>'
    DatabaseName = 'mysqldatabase'
    RequestedServiceObjectiveName = 'S0'
    SampleName = 'AdventureWorksLT'
}
New-AzSqlDatabase @parameters2
```

## <a name="create-private-endpoint"></a>Создание частной конечной точки

В рамках этого раздела вы создадите частную конечную точку и подключение с помощью:

* [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/New-AzPrivateLinkServiceConnection)
* [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint)

```azurepowershell-interactive
## Place SQL server into variable. Replace <sql-server-name> with your server name ##
$server = Get-AzSqlServer -ResourceGroupName CreateSQLEndpointTutorial-rg -ServerName <sql-server-name>

## Create private endpoint connection. ##
$parameters1 = @{
    Name = 'myConnection'
    PrivateLinkServiceId = $server.ResourceID
    GroupID = 'sqlserver'
}
$privateEndpointConnection = New-AzPrivateLinkServiceConnection @parameters1

## Place virtual network into variable. ##
$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Name 'myVNet'

## Disable private endpoint network policy ##
$vnet.Subnets[0].PrivateEndpointNetworkPolicies = "Disabled"
$vnet | Set-AzVirtualNetwork

## Create private endpoint
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'myPrivateEndpoint'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    PrivateLinkServiceConnection = $privateEndpointConnection
}
New-AzPrivateEndpoint @parameters2
```
## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS

В рамках этого раздела вы создадите и настроите частную зону DNS с помощью:

* [New-AzPrivateDnsZone](/powershell/module/az.privatedns/new-azprivatednszone)
* [New-AzPrivateDnsVirtualNetworkLink](/powershell/module/az.privatedns/new-azprivatednsvirtualnetworklink)
* [New-AzPrivateDnsZoneConfig](/powershell/module/az.network/new-azprivatednszoneconfig)
* [New-AzPrivateDnsZoneGroup](/powershell/module/az.network/new-azprivatednszonegroup)

```azurepowershell-interactive
## Place virtual network into variable. ##
$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Name 'myVNet'

## Create private dns zone. ##
$parameters1 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'privatelink.database.windows.net'
}
$zone = New-AzPrivateDnsZone @parameters1

## Create dns network link. ##
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ZoneName = 'privatelink.database.windows.net'
    Name = 'myLink'
    VirtualNetworkId = $vnet.Id
}
$link = New-AzPrivateDnsVirtualNetworkLink @parameters2

## Create DNS configuration ##
$parameters3 = @{
    Name = 'privatelink.database.windows.net'
    PrivateDnsZoneId = $zone.ResourceId
}
$config = New-AzPrivateDnsZoneConfig @parameters3

## Create DNS zone group. ##
$parameters4 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    PrivateEndpointName = 'myPrivateEndpoint'
    Name = 'myZoneGroup'
    PrivateDnsZoneConfig = $config
}
New-AzPrivateDnsZoneGroup @parameters4
```

## <a name="test-connectivity-to-private-endpoint"></a>Проверка подключения к частной конечной точке

В этом разделе описано, как использовать виртуальную машину, созданную на предыдущем шаге, для подключения к серверу SQL через частную конечную точку.

1. Войдите на [портал Azure](https://portal.azure.com) 
 
2. В области навигации слева выберите **Группы ресурсов**.

3. Выберите группу ресурсов **CreateSQLEndpointTutorial-rg**.

4. Выберите **myVM**.

5. На обзорной странице для **myVM** выберите **Подключиться**, а затем — **Бастион**.

6. Нажмите синюю кнопку **Использовать Бастион**.

7. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

8. После подключения откройте на сервере Windows PowerShell.

9. Введите `nslookup <sqlserver-name>.database.windows.net`. Замените **\<sqlserver-name>** именем сервера SQL, созданного на предыдущих шагах.  Вы получите сообщение, аналогичное показанному ниже:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Для имени сервера SQL возвращен частный IP-адрес **10.0.0.5**.  Этот адрес находится в подсети виртуальной сети, созданной ранее.


10. Установите [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) в **myVM**.

11. Откройте **SQL Server Management Studio**.

12. В окне **Подключение к серверу** введите или выберите следующую информацию:

    | Параметр | Значение |
    | ------- | ----- |
    | Тип сервера | Выберите **Ядро СУБД**.|
    | Имя сервера | Введите **\<sql-server-name>.database.windows.net** |
    | Проверка подлинности | Выберите **Проверка подлинности SQL Server**. |
    | Имя пользователя | Введите имя пользователя, указанное при создании сервера |
    | Пароль | Введите пароль, указанный при создании сервера |
    | Запоминание пароля | Выберите **Да**. |

13. Выберите **Подключиться**.

14. Просмотрите базы данных в меню слева.

15. (Дополнительно) Создание или запрос информации из базы данных **mysqldatabase**.

16. Закройте подключение бастиона к **myVM**. 

## <a name="clean-up-resources"></a>Очистка ресурсов 
После завершения работы с частной конечной точкой,SQL server и виртуальными машинами удалите группу ресурсов и все содержащиеся в ней ресурсы: 

1. Введите **CreateSQLEndpointTutorial-rg** в поле **Поиск** в верхней части портала и выберите **CreateSQLEndpointTutorial-rg** из результатов поиска. 

2. Выберите **Удалить группу ресурсов**. 

3. В поле **TYPE THE RESOURCE GROUP NAME** (Введите имя группы ресурсов) введите **CreateSQLEndpointTutorial-rg** и выберите элемент **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы создали:

* виртуальную сеть и узел-бастион;
* Виртуальная машина.
* Azure SQL Server с частной конечной точкой.

Вы использовали виртуальную машину для безопасного тестирования подключения к SQL Server через частную конечную точку.

В качестве следующего шага вы также можете ознакомиться со сценарием архитектуры **веб-приложения с частным подключением к базе данных SQL Azure**, когда веб-приложение за пределами виртуальной сети подключается к частной конечной точке базы данных.
> [!div class="nextstepaction"]
> [Веб-приложение с частным подключением к базе данных SQL Azure](/azure/architecture/example-scenario/private-web-app/private-web-app)