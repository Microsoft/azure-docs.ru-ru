---
title: Руководство по Добавление Управляемого экземпляра SQL в группу отработки отказа
titleSuffix: Azure SQL Managed Instance
description: Из этого учебника вы узнаете, как создать группу отработки отказа между основным и вторичным Управляемыми экземплярами SQL Azure.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: sashan
ms.date: 08/27/2019
ms.openlocfilehash: 1609f188af8ffb58251edc806e19f7820a6b0869
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99525727"
---
# <a name="tutorial-add-sql-managed-instance-to-a-failover-group"></a>Руководство по Добавление Управляемого экземпляра SQL в группу отработки отказа
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Узнайте, как добавить управляемые экземпляры SQL Azure в группу отработки отказа. В этой статье раскрываются следующие темы:

> [!div class="checklist"]
> - создание основного управляемого экземпляра;
> - создание вторичного управляемого экземпляра в составе [группы отработки отказа](../database/auto-failover-group-overview.md); 
> - тестовая отработка отказа.

  > [!NOTE]
  > - При работе с этим учебником необходимо выполнить настройку ресурсов с учетом [предварительных требований по настройке групп отработки отказа для Управляемого экземпляра SQL](../database/auto-failover-group-overview.md#enabling-geo-replication-between-managed-instances-and-their-vnets). 
  > - Создание управляемого экземпляра может занять много времени. Это означает, что на работу с этим учебником может потребоваться несколько часов. Дополнительные сведения о времени подготовки см. в разделе об [операциях управления в Управляемом экземпляре SQL](sql-managed-instance-paas-overview.md#management-operations). 
  > - Для управляемых экземпляров, которые включены в группу отработки отказа, требуется [Azure ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md), глобальный пиринг виртуальных сетей или два подключенных VPN-шлюза. В этом учебнике содержатся инструкции по созданию и подключению VPN-шлюзов. Если вы уже настроили ExpressRoute, пропустите эти шаги. 


## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/azure-portal)
В рамках этого руководства вам потребуются: 

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Для работы с этим учебником требуется следующее:

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.
- [Azure PowerShell](/powershell/azure/)

---


## <a name="create-a-resource-group-and-primary-managed-instance"></a>Создание группы ресурсов и основного управляемого экземпляра

На этом шаге вы создадите группу ресурсов и основной управляемый экземпляр для группы отработки отказа с помощью портала Azure или PowerShell. 

Разверните оба управляемых экземпляра в [парные регионы](../../best-practices-availability-paired-regions.md), чтобы обеспечить максимальную производительность. Управляемые экземпляры, размещенные в геопарных регионах, обеспечат гораздо более высокую производительность по сравнению с размещением в непарных регионах. 


# <a name="portal"></a>[Портал](#tab/azure-portal) 

Создайте группу ресурсов и основной управляемый экземпляр с помощью портала Azure. 

1. На портале Azure в меню слева выберите **Azure SQL**. Если **SQL Azure** отсутствует в списке, выберите **Все службы** и в поле поиска введите `Azure SQL`. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева. 
1. Щелкните **+ Добавить**, чтобы открыть страницу **Выбор варианта развертывания SQL**. Чтобы просмотреть дополнительные сведения о различных базах данных, выберите **Показать сведения** на плитке **Базы данных**.
1. Выберите **Создать** на плитке **Управляемые экземпляры SQL**. 

    ![Выбор Управляемого экземпляра SQL](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. На странице **Создание Управляемого экземпляра SQL Azure** выполните следующие действия на вкладке **Основные сведения**:
    1. В разделе **Сведения о проекте** выберите **подписку** из раскрывающегося списка, а затем щелкните **Создать** для создания группы ресурсов. Введите имя для группы ресурсов, например `myResourceGroup`. 
    1. В разделе **Сведения об Управляемом экземпляре SQL** укажите имя управляемого экземпляра и регион, в котором вы хотите развернуть его. Сохраните значения по умолчанию в разделе **Вычисления и хранилище**. 
    1. В разделе **Учетная запись администратора** укажите для администратора имя входа, например `azureuser`, и сложный пароль. 

    ![Создание основного управляемого экземпляра](./media/failover-group-add-instance-tutorial/primary-sql-mi-values.png)

1. Сохраните значения по умолчанию для остальных параметров и выберите **Проверка и создание**, чтобы проверить параметры Управляемого экземпляра SQL. 
1. Щелкните **Создать**, чтобы создать основной управляемый экземпляр. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте группу ресурсов и основной управляемый экземпляр с помощью PowerShell. 

   ```powershell-interactive
   # Connect-AzAccount
   # The SubscriptionId in which to create these objects
   $SubscriptionId = '<Subscription-ID>'
   # Create a random identifier to use as subscript for the different resource names
   $randomIdentifier = $(Get-Random)
   # Set the resource group name and location for SQL Managed Instance
   $resourceGroupName = "myResourceGroup-$randomIdentifier"
   $location = "eastus"
   $drLocation = "eastus2"
   
   # Set the networking values for your primary managed instance
   $primaryVNet = "primaryVNet-$randomIdentifier"
   $primaryAddressPrefix = "10.0.0.0/16"
   $primaryDefaultSubnet = "primaryDefaultSubnet-$randomIdentifier"
   $primaryDefaultSubnetAddress = "10.0.0.0/24"
   $primaryMiSubnetName = "primaryMISubnet-$randomIdentifier"
   $primaryMiSubnetAddress = "10.0.0.0/24"
   $primaryMiGwSubnetAddress = "10.0.255.0/27"
   $primaryGWName = "primaryGateway-$randomIdentifier"
   $primaryGWPublicIPAddress = $primaryGWName + "-ip"
   $primaryGWIPConfig = $primaryGWName + "-ipc"
   $primaryGWAsn = 61000
   $primaryGWConnection = $primaryGWName + "-connection"
   
   
   # Set the networking values for your secondary managed instance
   $secondaryVNet = "secondaryVNet-$randomIdentifier"
   $secondaryAddressPrefix = "10.128.0.0/16"
   $secondaryDefaultSubnet = "secondaryDefaultSubnet-$randomIdentifier"
   $secondaryDefaultSubnetAddress = "10.128.0.0/24"
   $secondaryMiSubnetName = "secondaryMISubnet-$randomIdentifier"
   $secondaryMiSubnetAddress = "10.128.0.0/24"
   $secondaryMiGwSubnetAddress = "10.128.255.0/27"
   $secondaryGWName = "secondaryGateway-$randomIdentifier"
   $secondaryGWPublicIPAddress = $secondaryGWName + "-IP"
   $secondaryGWIPConfig = $secondaryGWName + "-ipc"
   $secondaryGWAsn = 62000
   $secondaryGWConnection = $secondaryGWName + "-connection"
   
   
   
   # Set the SQL Managed Instance name for the new managed instances
   $primaryInstance = "primary-mi-$randomIdentifier"
   $secondaryInstance = "secondary-mi-$randomIdentifier"
   
   # Set the admin login and password for SQL Managed Instance
   $secpasswd = "PWD27!"+(New-Guid).Guid | ConvertTo-SecureString -AsPlainText -Force
   $mycreds = New-Object System.Management.Automation.PSCredential ("azureuser", $secpasswd)
   
   
   # Set the SQL Managed Instance service tier, compute level, and license mode
   $edition = "General Purpose"
   $vCores = 8
   $maxStorage = 256
   $computeGeneration = "Gen5"
   $license = "LicenseIncluded" #"BasePrice" or LicenseIncluded if you have don't have SQL Server license that can be used for AHB discount
   
   # Set failover group details
   $vpnSharedKey = "mi1mi2psk"
   $failoverGroupName = "failovergroup-$randomIdentifier"
   
   # Show randomized variables
   Write-host "Resource group name is" $resourceGroupName
   Write-host "Password is" $secpasswd
   Write-host "Primary Virtual Network name is" $primaryVNet
   Write-host "Primary default subnet name is" $primaryDefaultSubnet
   Write-host "Primary SQL Managed Instance subnet name is" $primaryMiSubnetName
   Write-host "Secondary Virtual Network name is" $secondaryVNet
   Write-host "Secondary default subnet name is" $secondaryDefaultSubnet
   Write-host "Secondary SQL Managed Instance subnet name is" $secondaryMiSubnetName
   Write-host "Primary SQL Managed Instance name is" $primaryInstance
   Write-host "Secondary SQL Managed Instance name is" $secondaryInstance
   Write-host "Failover group name is" $failoverGroupName
   
   # Suppress networking breaking changes warning (https://aka.ms/azps-changewarnings
   Set-Item Env:\SuppressAzurePowerShellBreakingChangeWarnings "true"
   
   # Set the subscription context
   Set-AzContext -SubscriptionId $subscriptionId 
   
   # Create the resource group
   Write-host "Creating resource group..."
   $resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
   $resourceGroup
   
   # Configure the primary virtual network
   Write-host "Creating primary virtual network..."
   $primaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -Name $primaryVNet `
                         -AddressPrefix $primaryAddressPrefix
   
                     Add-AzVirtualNetworkSubnetConfig `
                         -Name $primaryMiSubnetName `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                     | Set-AzVirtualNetwork
   $primaryVirtualNetwork
   
   
   # Configure the primary managed instance subnet
   Write-host "Configuring primary MI subnet..."
   $primaryVirtualNetwork = Get-AzVirtualNetwork -Name $primaryVNet -ResourceGroupName $resourceGroupName
   
   
   $primaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $primaryMiSubnetName `
                           -VirtualNetwork $primaryVirtualNetwork
   $primaryMiSubnetConfig
   
   # Configure the network security group management service
   Write-host "Configuring primary MI subnet..."
   
   $primaryMiSubnetConfigId = $primaryMiSubnetConfig.Id
   
   $primaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'primaryNSGMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryNSGMiManagementService
   
   # Configure the route table management service
   Write-host "Configuring primary MI route table management service..."
   
   $primaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'primaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryRouteTableMiManagementService
   
   # Configure the primary network security group
   Write-host "Configuring primary network security group..."
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -Name $primaryMiSubnetName `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NetworkSecurityGroup $primaryNSGMiManagementService `
                         -RouteTable $primaryRouteTableMiManagementService | `
                       Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryNSGMiManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $PrimaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $PrimaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   Write-host "Primary network security group configured successfully."
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "primaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Primary network route table configured successfully."
   
   
   # Create the primary managed instance
   
   Write-host "Creating primary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $primaryInstance `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -SubnetId $primaryMiSubnetConfigId `
                         -AdministratorCredential $mycreds `
                         -StorageSizeInGB $maxStorage `
                         -VCore $vCores `
                         -Edition $edition `
                         -ComputeGeneration $computeGeneration `
                         -LicenseType $license
   Write-host "Primary SQL Managed Instance created successfully."
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группы ресурсов Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Создает группу безопасности сети. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Создает таблицу маршрутизации. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Обновляет конфигурацию подсети для виртуальной сети.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Возвращает группу безопасности сети. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Добавляет в группу безопасности сети конфигурацию правила безопасности сети. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Обновляет группу безопасности сети.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Добавляет маршрут в таблицу маршрутизации. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Обновляет таблицу маршрутизации.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Создает управляемый экземпляр  |

---

## <a name="create-secondary-virtual-network"></a>Создание вторичной виртуальной сети

Если для создания управляемого экземпляра вы используете портал Azure, нужно отдельно создать виртуальную сеть, так как подсети основного и вторичного управляемых экземпляров не должны пересекаться между собой. Если вы используете для настройки управляемого экземпляра PowerShell, сразу перейдите к шагу 3. 

# <a name="portal"></a>[Портал](#tab/azure-portal) 

Чтобы проверить диапазон подсети для основной виртуальной сети, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) перейдите к группе ресурсов и выберите виртуальную сеть для основного экземпляра.  
2. Щелкните **Подсети** в разделе **Параметры** и запомните или запишите значение параметра **Диапазон адресов**. Этот диапазон не должен пересекаться с диапазоном адресов подсети виртуальной сети для вторичного управляемого экземпляра. 


   ![Первичная подсеть](./media/failover-group-add-instance-tutorial/verify-primary-subnet-range.png)

Чтобы создать виртуальную сеть, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) щелкните **Создать ресурс** и выполните поиск по строке *виртуальная сеть*. 
1. Выберите вариант **Виртуальная сеть**, опубликованный корпорацией Майкрософт, а на следующей странице щелкните **Создать**. 
1. Заполните обязательные поля, чтобы настроить виртуальную сеть для вторичного управляемого экземпляра, а затем выберите **Создать**. 

   В приведенной ниже таблице представлены необходимые значения для вторичной виртуальной сети:

    | **Поле** | Значение |
    | --- | --- |
    | **Имя** |  Имя виртуальной сети для вторичного управляемого экземпляра, например `vnet-sql-mi-secondary`. |
    | **Адресное пространство** | Адресное пространство для виртуальной сети, например `10.128.0.0/16`. | 
    | **Подписка** | Подписка, в которой находятся основной управляемый экземпляр и группа ресурсов. |
    | **Регион** | Расположение, в которое будет развернут вторичный управляемый экземпляр. |
    | **Подсеть** | Имя для этой подсети. По умолчанию здесь подставляется значение `default`. |
    | **Диапазон адресов**| Диапазон адресов для этой подсети. Это значение должно отличаться от диапазона адресов подсети, используемого виртуальной сетью основного управляемого экземпляра, например `10.128.0.0/24`.  |
    | &nbsp; | &nbsp; |

    ![Значения для вторичной виртуальной сети](./media/failover-group-add-instance-tutorial/secondary-virtual-network.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Этот шаг необходим только в том случае, если для развертывания Управляемого экземпляра SQL вы используете портал Azure. Если вы используете PowerShell, перейдите к шагу 3. 

---

## <a name="create-a-secondary-managed-instance"></a>Создание вторичного управляемого экземпляра
На этом шаге вы создадите вторичный управляемый экземпляр на портале Azure, одновременно настроив сетевое подключение между двумя управляемыми экземплярами. 

Второй управляемый экземпляр должен соответствовать следующим условиям:
- быть пустым; 
- иметь подсеть с диапазоном IP-адресов, который не пересекается с диапазоном основного управляемого экземпляра. 

# <a name="portal"></a>[Портал](#tab/azure-portal) 

Создайте вторичный управляемый экземпляр с помощью портала Azure. 

1. На портале Azure в меню слева выберите **Azure SQL**. Если **SQL Azure** отсутствует в списке, выберите **Все службы** и в поле поиска введите `Azure SQL`. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева. 
1. Щелкните **+ Добавить**, чтобы открыть страницу **Выбор варианта развертывания SQL**. Чтобы просмотреть дополнительные сведения о различных базах данных, выберите **Показать сведения** на плитке **Базы данных**.
1. Выберите **Создать** на плитке **Управляемые экземпляры SQL**. 

    ![Выбор Управляемого экземпляра SQL](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. На вкладке **Основные сведения** на странице **Создание Управляемого экземпляра SQL Azure** заполните обязательные поля, чтобы настроить вторичный управляемый экземпляр. 

   В приведенной ниже таблице представлены необходимые значения для вторичного управляемого экземпляра:
 
    | **Поле** | Значение |
    | --- | --- |
    | **Подписка** |  Подписка, в которой находится основной управляемый экземпляр. |
    | **Группа ресурсов**| Группа ресурсов, в которой находится основной управляемый экземпляр. |
    | **Имя Управляемого экземпляра SQL** | Имя создаваемого вторичного управляемого экземпляра, например `sql-mi-secondary`.  | 
    | **Регион**| Расположение для вторичного управляемого экземпляра.  |
    | **Имя для входа администратора Управляемого экземпляра SQL** | Имя пользователя для входа на вторичный управляемый экземпляр, например `azureuser`. |
    | **Пароль** | Сложный пароль, который будет использоваться в сочетании с именем администратора для входа на вторичный управляемый экземпляр.  |
    | &nbsp; | &nbsp; |

1. На вкладке **Сеть** выберите из раскрывающегося списка в разделе **Виртуальная сеть** ту виртуальную сеть, которую создали ранее для вторичного управляемого экземпляра.

   ![Сеть для вторичного управляемого экземпляра](./media/failover-group-add-instance-tutorial/networking-settings-for-secondary-mi.png)

1. На вкладке **Дополнительные параметры** выберите в разделе **Георепликация** значение **Да** для параметра _Использовать в качестве вторичного экземпляра для отработки отказа_. Выберите основной управляемый экземпляр из раскрывающегося списка. 
    
   Проследите, чтобы параметры сортировки и часовой пояс совпадали с теми, которые указаны для основного управляемого экземпляра. Для основного управляемого экземпляра, созданного в этом учебнике, для параметров сортировки по умолчанию указано значение `SQL_Latin1_General_CP1_CI_AS`, а для часового пояса — `(UTC) Coordinated Universal Time`. 

   ![Сеть для вторичного управляемого экземпляра](./media/failover-group-add-instance-tutorial/secondary-mi-failover.png)

1. Щелкните **Проверка и создание**, чтобы проверить параметры для вторичного управляемого экземпляра. 
1. Щелкните **Создать**, чтобы создать вторичный управляемый экземпляр. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте вторичный управляемый экземпляр с помощью PowerShell. 

   ```powershell-interactive
   # Configure the secondary virtual network
   Write-host "Configuring secondary virtual network..."
   
   $SecondaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $drlocation `
                         -Name $secondaryVNet `
                         -AddressPrefix $secondaryAddressPrefix
   
   Add-AzVirtualNetworkSubnetConfig `
                         -Name $secondaryMiSubnetName `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -AddressPrefix $secondaryMiSubnetAddress `
                       | Set-AzVirtualNetwork
   $SecondaryVirtualNetwork
   
   # Configure the secondary managed instance subnet
   Write-host "Configuring secondary MI subnet..."
   
   $SecondaryVirtualNetwork = Get-AzVirtualNetwork -Name $secondaryVNet `
                                   -ResourceGroupName $resourceGroupName
   
   $secondaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $secondaryMiSubnetName `
                           -VirtualNetwork $SecondaryVirtualNetwork
   $secondaryMiSubnetConfig
   
   # Configure the secondary network security group management service
   Write-host "Configuring secondary network security group management service..."
   
   $secondaryMiSubnetConfigId = $secondaryMiSubnetConfig.Id
   
   $secondaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'secondaryToMIManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryNSGMiManagementService
   
   # Configure the secondary route table MI management service
   Write-host "Configuring secondary route table MI management service..."
   
   $secondaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'secondaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryRouteTableMiManagementService
   
   # Configure the secondary network security group
   Write-host "Configuring secondary network security group..."
   
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -Name $secondaryMiSubnetName `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NetworkSecurityGroup $secondaryNSGMiManagementService `
                         -RouteTable $secondaryRouteTableMiManagementService `
                       | Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryToMIManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $secondaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $secondaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "secondaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Secondary network security group configured successfully."
   
   # Create the secondary managed instance
   
   $primaryManagedInstanceId = Get-AzSqlInstance -Name $primaryInstance -ResourceGroupName $resourceGroupName | Select-Object Id
   
   
   Write-host "Creating secondary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $secondaryInstance `
                     -ResourceGroupName $resourceGroupName `
                     -Location $drLocation `
                     -SubnetId $secondaryMiSubnetConfigId `
                     -AdministratorCredential $mycreds `
                     -StorageSizeInGB $maxStorage `
                     -VCore $vCores `
                     -Edition $edition `
                     -ComputeGeneration $computeGeneration `
                     -LicenseType $license `
                     -DnsZonePartner $primaryManagedInstanceId.Id
   Write-host "Secondary SQL Managed Instance created successfully."
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группы ресурсов Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Создает группу безопасности сети. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Создает таблицу маршрутизации. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Обновляет конфигурацию подсети для виртуальной сети.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Возвращает группу безопасности сети. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Добавляет в группу безопасности сети конфигурацию правила безопасности сети. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Обновляет группу безопасности сети.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Добавляет маршрут в таблицу маршрутизации. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Обновляет таблицу маршрутизации.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Создает управляемый экземпляр  |

---

## <a name="create-a-primary-gateway"></a>Создание первичного шлюза 

Чтобы два управляемых экземпляра работали в группе отработки отказа, необходимо настроить ExpressRoute или шлюз между виртуальными сетями этих двух управляемых экземпляров, чтобы поддерживать сетевое взаимодействие. Если вы решили настроить [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md) вместо двух VPN-шлюзов, перейдите к [шагу 7](#create-a-failover-group).  

В этой статье приводятся шаги по созданию и подключению двух VPN-шлюзов, но вы можете сразу перейти к созданию группы отработки отказа, если вместо них вы настроили ExpressRoute. 

> [!NOTE]
> Пропускная способность зависит от выбранной ценовой категории шлюза. В этом учебнике для развертывания шлюза используется самая низкая ценовая категория (`HwGw1`). Чтобы повысить пропускную способность, разверните шлюз более высокой ценовой категории (например, `VpnGw3`). Все доступные варианты перечислены в [списке ценовых категорий шлюза](../../vpn-gateway/vpn-gateway-about-vpngateways.md#benchmark).

# <a name="portal"></a>[Портал](#tab/azure-portal)

Создайте шлюз для виртуальной сети основного управляемого экземпляра с помощью портала Azure. 


1. На [портале Azure](https://portal.azure.com) перейдите к группе ресурсов и выберите ресурс **Виртуальная сеть** для основного управляемого экземпляра. 
1. Выберите **Подсети** в разделе **Параметры**, а затем щелкните действие **добавления подсети шлюза**. Не изменяйте значения по умолчанию. 

   ![Добавление шлюза для основного управляемого экземпляра](./media/failover-group-add-instance-tutorial/add-subnet-gateway-primary-vnet.png)

1. После создания шлюза подсети выберите **Создать ресурс** в области навигации слева, а затем введите `Virtual network gateway` в поле поиска. Выберите ресурс **Шлюз виртуальной сети**, опубликованный **корпорацией Майкрософт**. 

   ![Создание шлюза виртуальной сети](./media/failover-group-add-instance-tutorial/create-virtual-network-gateway.png)

1. Заполните обязательные поля, чтобы настроить шлюз для основного управляемого экземпляра. 

   В приведенной ниже таблице представлены необходимые значения шлюза для основного управляемого экземпляра:
 
    | **Поле** | Значение |
    | --- | --- |
    | **Подписка** |  Подписка, в которой находится основной управляемый экземпляр. |
    | **Имя** | Имя для шлюза виртуальной сети, например `primary-mi-gateway`. | 
    | **Регион** | Регион, в котором находится основной управляемый экземпляр. |
    | **Тип шлюза** | Выберите **VPN**. |
    | **Тип VPN** | Выберите **На основе маршрута**. |
    | **SKU**| Сохраните значение по умолчанию `VpnGw1`. |
    | **Виртуальная сеть**| Выберите виртуальную сеть, созданную в разделе 2, например `vnet-sql-mi-primary`. |
    | **Общедоступный IP-адрес**| Выберите **Создать**. |
    | **Имя общедоступного IP-адреса**| Укажите имя для IP-адреса, например `primary-gateway-IP`. |
    | &nbsp; | &nbsp; |

1. Сохраните остальные значения по умолчанию, а затем щелкните **Проверка и создание**, чтобы проверить параметры шлюза для виртуальной сети.

   ![Параметры основного шлюза](./media/failover-group-add-instance-tutorial/settings-for-primary-gateway.png)

1. Щелкните **создать**, чтобы создать новый шлюз виртуальной сети. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте шлюз для виртуальной сети основного управляемого экземпляра с помощью PowerShell. 

   ```powershell-interactive
   # Create the primary gateway
   Write-host "Adding GatewaySubnet to primary VNet..."
   Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $primaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $primaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName
   $primaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $primaryVirtualNetwork
   
   Write-host "Creating primary gateway..."
   Write-host "This will take some time."
   $primaryGWPublicIP = New-AzPublicIpAddress -Name $primaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $location -AllocationMethod Dynamic
   $primaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $primaryGWIPConfig `
            -Subnet $primaryGatewaySubnet -PublicIpAddress $primaryGWPublicIP
   
   $primaryGateway = New-AzVirtualNetworkGateway -Name $primaryGWName -ResourceGroupName $resourceGroupName `
       -Location $location -IpConfigurations $primaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $primaryGWAsn
   $primaryGateway
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создает IP-конфигурацию для шлюза виртуальной сети. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создает шлюз виртуальной сети. |


---


## <a name="create-secondary-gateway"></a>Создание вторичного шлюза 
На этом шаге вы создадите шлюз для виртуальной сети вторичного управляемого экземпляра с помощью портала Azure. 


# <a name="portal"></a>[Портал](#tab/azure-portal)

На портале Azure повторите все действия, описанные в предыдущем разделе, чтобы создать подсеть и шлюз виртуальной сети для вторичного управляемого экземпляра. Заполните обязательные поля, чтобы настроить шлюз для вторичного управляемого экземпляра. 

   В приведенной ниже таблице представлены необходимые значения шлюза для вторичного управляемого экземпляра:

   | **Поле** | Значение |
   | --- | --- |
   | **Подписка** |  Подписка, в которой находится вторичный управляемый экземпляр. |
   | **Имя** | Имя для шлюза виртуальной сети, например `secondary-mi-gateway`. | 
   | **Регион** | Регион, в котором находится вторичный управляемый экземпляр. |
   | **Тип шлюза** | Выберите **VPN**. |
   | **Тип VPN** | Выберите **На основе маршрута**. |
   | **SKU**| Сохраните значение по умолчанию `VpnGw1`. |
   | **Виртуальная сеть**| Выберите виртуальную сеть для вторичного управляемого экземпляра, например `vnet-sql-mi-secondary`. |
   | **Общедоступный IP-адрес**| Выберите **Создать**. |
   | **Имя общедоступного IP-адреса**| Укажите имя для IP-адреса, например `secondary-gateway-IP`. |
   | &nbsp; | &nbsp; |

   ![Параметры вторичного шлюза](./media/failover-group-add-instance-tutorial/settings-for-secondary-gateway.png)


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте шлюз для виртуальной сети вторичного управляемого экземпляра с помощью PowerShell. 

   ```powershell-interactive
   # Create the secondary gateway
   Write-host "Creating secondary gateway..."
   
   Write-host "Adding GatewaySubnet to secondary VNet..."
   Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $secondaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $secondaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName
   $secondaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $secondaryVirtualNetwork
   $drLocation = $secondaryVirtualNetwork.Location
   
   Write-host "Creating secondary gateway..."
   Write-host "This will take some time."
   $secondaryGWPublicIP = New-AzPublicIpAddress -Name $secondaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $drLocation -AllocationMethod Dynamic
   $secondaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $secondaryGWIPConfig `
            -Subnet $secondaryGatewaySubnet -PublicIpAddress $secondaryGWPublicIP
   
   $secondaryGateway = New-AzVirtualNetworkGateway -Name $secondaryGWName -ResourceGroupName $resourceGroupName `
       -Location $drLocation -IpConfigurations $secondaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $secondaryGWAsn
   $secondaryGateway
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создает IP-конфигурацию для шлюза виртуальной сети. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создает шлюз виртуальной сети. |

---


## <a name="connect-the-gateways"></a>Подключение шлюзов
На этом шаге вы создадите двунаправленное подключение между двумя шлюзами двух виртуальных сетей. 


# <a name="portal"></a>[Портал](#tab/azure-portal)

Подключите два шлюза с помощью портала Azure. 


1. Выберите **Создать ресурс** на [портале Azure](https://portal.azure.com).
1. Введите `connection` в поле поиска и нажмите клавишу ВВОД, чтобы найти ресурс **Подключение**, опубликованный корпорацией Майкрософт.
1. Щелкните **Создать**, чтобы создать подключение. 
1. На странице **Основные сведения** укажите приведенные ниже значения, после чего щелкните **ОК**. 
    1. Выберите `VNet-to-VNet` для параметра **Тип подключения**. 
    1. Выберите подписку в раскрывающемся списке. 
    1. В раскрывающемся списке выберите группу ресурсов для Управляемого экземпляра SQL. 
    1. Выберите из раскрывающегося списка расположение основного управляемого экземпляра. 
1. На странице **Параметры** введите или выберите приведенные ниже значения, после чего щелкните **ОК**:
    1. Выберите основной сетевой шлюз в поле **Первый шлюз виртуальной сети**, например `primaryGateway`.  
    1. Выберите вторичный сетевой шлюз в поле **Второй шлюз виртуальной сети**, например `secondaryGateway`. 
    1. Установите флажок **Установить двунаправленное взаимодействие**. 
    1. Сохраните предложенное по умолчанию имя основного подключения или присвойте ему любое значение на ваш выбор. 
    1. Укажите **Общий ключ (PSK)** для подключения, например `mi1m2psk`. 
    1. Нажмите кнопку **ОК**, чтобы сохранить параметры. 

    ![Создание подключения шлюза](./media/failover-group-add-instance-tutorial/create-gateway-connection.png)

    

1. На странице **Проверка и создание** проверьте параметры двунаправленного подключения и щелкните **ОК**, чтобы создать его. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Соедините два шлюза с помощью PowerShell. 

   ```powershell-interactive
   # Connect the primary to secondary gateway
   Write-host "Connecting the primary gateway to secondary gateway..."
   New-AzVirtualNetworkGatewayConnection -Name $primaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $primaryGateway -VirtualNetworkGateway2 $secondaryGateway -Location $location `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $primaryGWConnection
   
   # Connect the secondary to primary gateway
   Write-host "Connecting the secondary gateway to primary gateway..."
   
   New-AzVirtualNetworkGatewayConnection -Name $secondaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $secondaryGateway -VirtualNetworkGateway2 $primaryGateway -Location $drLocation `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $secondaryGWConnection
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Создает подключение между двумя шлюзами виртуальной сети.   |

---


## <a name="create-a-failover-group"></a>Создание группы отработки отказа
На этом шаге вы создадите группу отработки отказа и добавите в нее оба управляемых экземпляра. 


# <a name="portal"></a>[Портал](#tab/azure-portal)
Создайте группу отработки отказа с помощью портала Azure. 


1. На [портале Azure](https://portal.azure.com) в меню слева выберите **Azure SQL**. Если **SQL Azure** отсутствует в списке, выберите **Все службы** и в поле поиска введите `Azure SQL`. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева. 
1. Выберите основной управляемый экземпляр, созданный в первом разделе, например `sql-mi-primary`. 
1. В разделе **Параметры** откройте **Группы отработки отказа экземпляра**, а затем выберите **Добавить группу**, чтобы перейти на страницу **Группа отработки отказа экземпляра**. 

   ![Добавление группы отработки отказа](./media/failover-group-add-instance-tutorial/add-failover-group.png)

1. На странице **Группа отработки отказа экземпляра** введите имя группы отработки отказа, например `failovergrouptutorial`. Затем в раскрывающемся списке выберите вторичный управляемый экземпляр, например `sql-mi-secondary`. Щелкните **Создать**, чтобы создать группу отработки отказа. 

   ![Создание группы отработки отказа](./media/failover-group-add-instance-tutorial/create-failover-group.png)

1. Когда завершится развертывание группы отработки отказа, вы будете возвращены на страницу **Группа отработки отказа**. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Создайте группу отработки отказа с помощью PowerShell. 

   ```powershell-interactive
   Write-host "Creating the failover group..."
   $failoverGroup = New-AzSqlDatabaseInstanceFailoverGroup -Name $failoverGroupName `
        -Location $location -ResourceGroupName $resourceGroupName -PrimaryManagedInstanceName $primaryInstance `
        -PartnerRegion $drLocation -PartnerManagedInstanceName $secondaryInstance `
        -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
   $failoverGroup
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Создает новую группу отработки отказа Управляемого экземпляра SQL Azure.  |


---


## <a name="test-failover"></a>Тестовая отработка отказа
На этом шаге вы переключите группу отработки отказа на сервер-получатель, а затем восстановите размещение с помощью портала Azure. 


# <a name="portal"></a>[Портал](#tab/azure-portal)
Выполните тестовую отработку отказа с помощью портала Azure. 


1. Перейдите к _вторичному_ управляемому экземпляру на [портале Azure](https://portal.azure.com) и выберите **Группы отработки отказа экземпляра** в разделе параметров. 
1. Проверьте, какой управляемый экземпляр является основным, а какой — вторичным. 
1. Щелкните **Отработка отказа**, а затем нажмите **Да** в предупреждении о том, что сеансы TDS будут отключены. 

   ![Выполнение отработки отказа для группы отработки отказа](./media/failover-group-add-instance-tutorial/failover-mi-failover-group.png)

1. Проверьте, какой управляемый экземпляр является основным, а какой — вторичным. Если отработка отказа прошла успешно, эти два экземпляра должны поменяться ролями. 

   ![Управляемые экземпляры поменялись ролями после отработки отказа](./media/failover-group-add-instance-tutorial/mi-switched-after-failover.png)

1. Перейдите к новому _вторичному_ управляемому экземпляру и снова щелкните **Отработка отказа**, чтобы вернуть роль основного прежнему основному экземпляру. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Выполните тестовую отработку отказа с помощью PowerShell. 

   ```powershell-interactive
    
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   
   # Fail over the primary managed instance to the secondary role
   Write-host "Failing primary over to the secondary location"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to secondary location"
   ```


Переключите группу отработки отказа на основной сервер:

   ```powershell-interactive
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName
   
   # Fail the primary managed instance back to the primary role
   Write-host "Failing primary back to primary role"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to primary location"
   
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Возвращает или перечисляет группы отработки отказа Управляемого экземпляра SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Выполняет отработку отказа для группы отработки отказа Управляемого экземпляра SQL. | 

---



## <a name="clean-up-resources"></a>Очистка ресурсов
Очистите ресурсы, сначала удалив управляемые экземпляры, затем виртуальный кластер, остальные ресурсы и, наконец, группу ресурсов. 

# <a name="portal"></a>[Портал](#tab/azure-portal)
1. На [портале Azure](https://portal.azure.com) перейдите к используемой группе ресурсов. 
1. Выберите один или несколько управляемых экземпляров и щелкните **Удалить**. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**. Этот процесс некоторое время выполняется в фоновом режиме, и до его завершения вы не сможете удалить *виртуальный кластер* и другие зависимые ресурсы. За процессом удаления управляемого экземпляра можно следить на вкладке **Активность**. 
1. После удаления управляемого экземпляра удалите *виртуальный кластер*, выбрав его в группе ресурсов и щелкнув **Удалить**. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**. 
1. Удалите все остальные ресурсы. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**. 
1. Удалите группу ресурсов, выбрав **Удалить группу ресурсов**, введя в текстовом поле имя группы ресурсов `myResourceGroup` и щелкнув **Удалить**. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Группу ресурсов необходимо будет удалить дважды. При первом удалении группы ресурсов будет удален управляемый экземпляр и виртуальные кластеры, но после этого будет выдано сообщение об ошибке `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Выполните команду Remove-AzResourceGroup еще раз, чтобы удалить все остаточные ресурсы, а также группу ресурсов.

```powershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing SQL Managed Instance and virtual cluster..."
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing residual resources and resource group..."
```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

---

## <a name="full-script"></a>Полный сценарий

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add SQL Managed Instance to a failover group")]

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группы ресурсов Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Создает группу безопасности сети. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Создает таблицу маршрутизации. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Обновляет конфигурацию подсети для виртуальной сети.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Возвращает группу безопасности сети. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Добавляет в группу безопасности сети конфигурацию правила безопасности сети. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Обновляет группу безопасности сети.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Добавляет маршрут в таблицу маршрутизации. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Обновляет таблицу маршрутизации.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Создает управляемый экземпляр  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Возвращает информацию об Управляемом экземпляре SQL Azure. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создает IP-конфигурацию для шлюза виртуальной сети. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создает шлюз виртуальной сети. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Создает подключение между двумя шлюзами виртуальной сети.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Создает группу отработки отказа Управляемого экземпляра SQL.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Возвращает или перечисляет группы отработки отказа Управляемого экземпляра SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Выполняет отработку отказа для группы отработки отказа Управляемого экземпляра SQL. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. | 

# <a name="portal"></a>[Портал](#tab/azure-portal) 

Скрипты для портала Azure не предоставляются.

---

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого учебника вы настроили группу отработки отказа между двумя управляемыми экземплярами. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> - создание основного управляемого экземпляра;
> - создание вторичного управляемого экземпляра в составе [группы отработки отказа](../database/auto-failover-group-overview.md); 
> - тестовая отработка отказа.

Перейдите к следующему краткому руководству, которое посвящено подключению к Управляемому экземпляру SQL и восстановлению базы данных в Управляемом экземпляре SQL. 

> [!div class="nextstepaction"]
> [Подключение к Управляемому экземпляру SQL](connect-vm-instance-configure.md)
> [Восстановление базы данных в Управляемом экземпляре SQL](restore-sample-database-quickstart.md)


