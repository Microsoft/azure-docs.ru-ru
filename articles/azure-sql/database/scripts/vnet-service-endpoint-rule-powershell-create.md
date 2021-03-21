---
title: PowerShell для конечных точек и правил виртуальной сети для одной и для баз данных в составе пула
description: Предоставляет скрипты PowerShell для создания конечных точек виртуальных служб и управления ими для базы данных SQL Azure и Azure синапсе.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.devlang: PowerShell
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 04/17/2019
ms.custom: sqldbrb=1
tags: azure-synapse
ms.openlocfilehash: 76a1d3aaadcbd1b15966a84f5dd2fe876f82c43a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102177629"
---
# <a name="powershell-create-a-virtual-service-endpoint-and-vnet-rule-for-azure-sql-database"></a>PowerShell: создание конечной точки виртуальной службы и правила виртуальной сети для базы данных SQL Azure
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

*Правила виртуальной сети* — это одна из функций безопасности брандмауэра, которая определяет, будет ли [логический SQL Server](../logical-servers.md) для баз данных [SQL Azure](../sql-database-paas-overview.md) , эластичных пулов или баз данных в [Azure синапсе](../../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) принимать подключения, отправленные из определенных подсетей в виртуальных сетях.

> [!IMPORTANT]
> Эта статья относится к базе данных SQL Azure, включая Azure синапсе (ранее — SQL DW). Для простоты термин "база данных SQL Azure" в этой статье относится к базам данных SQL Azure или Azure синапсе. Эта статья *не* относится к управляемый экземпляр Azure SQL, так как с ней не связана конечная точка службы.

В этой статье демонстрируется сценарий PowerShell, который выполняет следующие действия:

1. Создает *конечную точку службы виртуальной сети* Microsoft Azure в подсети.
2. Добавляет конечную точку в брандмауэр сервера, чтобы создать *правило виртуальной сети*.

Дополнительные сведения см. в статье [конечные точки виртуальной службы для базы данных SQL Azure][sql-db-vnet-service-endpoint-rule-overview-735r].

> [!TIP]
> Если вам нужно только оценить или добавить *имя типа* конечной точки виртуальной службы для базы данных SQL Azure в подсеть, можно перейти к нашему более [прямому сценарию PowerShell](#a-verify-subnet-is-endpoint-ps-100).

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается базой данных SQL Azure, но все будущие разработки используются для [ `Az.Sql` командлетов](/powershell/module/az.sql). Более старый модуль см. в разделе [AzureRM. SQL](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны.

## <a name="major-cmdlets"></a>Основные командлеты

В этой статье рассматривается командлет [ **New-азсклсервервиртуалнетворкруле**](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule) , который добавляет конечную точку подсети в список управления доступом (ACL) сервера, тем самым создавая правило.

В следующем списке показана последовательность других *основных* командлетов, которые необходимо выполнить для подготовки к вызову командлета **New-азсклсервервиртуалнетворкруле**. В этой статье эти вызовы выполняются в [сценарии 3 "Правило виртуальной сети"](#a-script-30):

1. [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig): создает объект подсети.
2. [New-азвиртуалнетворк](/powershell/module/az.network/new-azvirtualnetwork): создает виртуальную сеть, предоставляя ей подсеть.
3. [Set-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/Set-azVirtualNetworkSubnetConfig): назначает конечную точку виртуальной службы вашей подсети.
4. [Set-азвиртуалнетворк](/powershell/module/az.network/Set-azVirtualNetwork): сохраняет обновления, внесенные в виртуальную сеть.
5. [New-азсклсервервиртуалнетворкруле](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule). После того как подсеть является конечной, добавляет подсеть в качестве правила виртуальной сети в список управления доступом сервера.
   - В версии 5.1.1 модуля Azure RM PowerShell и последующих версиях командлет предоставляет параметр **-IgnoreMissingVnetServiceEndpoint**.

## <a name="prerequisites-for-running-powershell"></a>Предварительные требования для запуска PowerShell

- Вы уже можете войти в Azure, например, воспользовавшись [порталом Azure][http-azure-portal-link-ref-477t].
- Вы уже можете выполнять сценарии PowerShell.

> [!NOTE]
> Убедитесь, что конечные точки службы включены для виртуальной сети и подсети, которые требуется добавить на сервер. В противном случае вам не удастся создать правило брандмауэра виртуальной сети.

## <a name="one-script-divided-into-four-chunks"></a>Один сценарий состоит четырех блоков

Наш демонстрационный сценарий PowerShell состоит из последовательности сценариев меньшего размера. Такое разделение облегчает обучение и обеспечивает гибкость. Эти сценарии должны запускаться в указанной последовательности. Если сейчас у вас нет времени для выполнения этих сценариев, вы можете ознакомиться с тестовыми выходными данными сценария 4.

<a name="a-script-10"></a>

### <a name="script-1-variables"></a>Сценарий 1: переменные

Первый сценарий PowerShell присваивает переменным значения. Последующие сценарии зависят от этих переменных.

> [!IMPORTANT]
> При необходимости, прежде чем выполнить этот сценарий, можно изменить значения. Например, если у вас уже есть группа ресурсов, можно изменить присваиваемое значение имени группы ресурсов.
>
> Имя вашей подписки необходимо указать в сценарии.

### <a name="powershell-script-1-source-code"></a>Исходный код сценария 1 PowerShell

```powershell
######### Script 1 ########################################
##   LOG into to your Azure account.                     ##
##   (Needed only one time per powershell.exe session.)  ##
###########################################################

$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]'
if ('yes' -eq $yesno) { Connect-AzAccount }

###########################################################
##  Assignments to variables used by the later scripts.  ##
###########################################################

# You can edit these values, if necessary.
$SubscriptionName = 'yourSubscriptionName'
Select-AzSubscription -SubscriptionName $SubscriptionName

$ResourceGroupName = 'RG-YourNameHere'
$Region = 'westcentralus'

$VNetName = 'myVNet'
$SubnetName = 'mySubnet'
$VNetAddressPrefix = '10.1.0.0/16'
$SubnetAddressPrefix = '10.1.1.0/24'
$VNetRuleName = 'myFirstVNetRule-ForAcl'

$SqlDbServerName = 'mysqldbserver-forvnet'
$SqlDbAdminLoginName = 'ServerAdmin'
$SqlDbAdminLoginPassword = 'ChangeYourAdminPassword1'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql'  # Official type name.

Write-Host 'Completed script 1, the "Variables".'
```

<a name="a-script-20"></a>

### <a name="script-2-prerequisites"></a>Сценарий 2: предварительные требования

Этот сценарий подготавливает среду к следующему сценарию, который выполняет действие с конечной точкой. Этот сценарий создает перечисленные ниже элементы, но только если они еще не существует. Сценарий 2 можно пропустить, если вы уверены, что эти элементы уже существуют:

- Группа ресурсов Azure
- логический сервер SQL Server;

### <a name="powershell-script-2-source-code"></a>Исходный код сценария 2 PowerShell

```powershell
######### Script 2 ########################################
##   Ensure your Resource Group already exists.          ##
###########################################################

Write-Host "Check whether your Resource Group already exists."

$gottenResourceGroup = $null
$gottenResourceGroup = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue

if ($null -eq $gottenResourceGroup) {
    Write-Host "Creating your missing Resource Group - $ResourceGroupName."
    New-AzResourceGroup -Name $ResourceGroupName -Location $Region
} else {
    Write-Host "Good, your Resource Group already exists - $ResourceGroupName."
}

$gottenResourceGroup = $null

###########################################################
## Ensure your server already exists. ##
###########################################################

Write-Host "Check whether your server already exists."

$sqlDbServer = $null
$azSqlParams = @{
    ResourceGroupName = $ResourceGroupName
    ServerName        = $SqlDbServerName
    ErrorAction       = 'SilentlyContinue'
}
$sqlDbServer = Get-AzSqlServer @azSqlParams

if ($null -eq $sqlDbServer) {
    Write-Host "Creating the missing server - $SqlDbServerName."
    Write-Host "Gather the credentials necessary to next create a server."

    $sqlAdministratorCredentials = [pscredential]::new($SqlDbAdminLoginName,(ConvertTo-SecureString -String $SqlDbAdminLoginPassword -AsPlainText -Force))

    if ($null -eq $sqlAdministratorCredentials) {
        Write-Host "ERROR, unable to create SQL administrator credentials.  Now ending."
        return
    }

    Write-Host "Create your server."

    $sqlSrvParams = @{
        ResourceGroupName           = $ResourceGroupName
        ServerName                  = $SqlDbServerName
        Location                    = $Region
        SqlAdministratorCredentials = $sqlAdministratorCredentials
    }
    New-AzSqlServer @sqlSrvParams
} else {
    Write-Host "Good, your server already exists - $SqlDbServerName."
}

$sqlAdministratorCredentials = $null
$sqlDbServer = $null

Write-Host 'Completed script 2, the "Prerequisites".'
```

<a name="a-script-30"></a>

## <a name="script-3-create-an-endpoint-and-a-rule"></a>Сценарий 3: создание конечной точки и правила

Этот сценарий создает виртуальную сеть с подсетью. Затем он присваивает тип конечной точки **Microsoft.Sql** подсети. Наконец, сценарий добавляет подсеть в список управления доступом (ACL), тем самым создавая правило.

### <a name="powershell-script-3-source-code"></a>Исходный код сценария 3 PowerShell

```powershell
######### Script 3 ########################################
##   Create your virtual network, and give it a subnet.  ##
###########################################################

Write-Host "Define a subnet '$SubnetName', to be given soon to a virtual network."

$subnetParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$subnet = New-AzVirtualNetworkSubnetConfig @subnetParams

Write-Host "Create a virtual network '$VNetName'.`nGive the subnet to the virtual network that we created."

$vnetParams = @{
    Name              = $VNetName
    AddressPrefix     = $VNetAddressPrefix
    Subnet            = $subnet
    ResourceGroupName = $ResourceGroupName
    Location          = $Region
}
$vnet = New-AzVirtualNetwork @vnetParams

###########################################################
##   Create a Virtual Service endpoint on the subnet.    ##
###########################################################

Write-Host "Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet."

$vnetSubParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    VirtualNetwork  = $vnet
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$vnet = Set-AzVirtualNetworkSubnetConfig @vnetSubParams

Write-Host "Persist the updates made to the virtual network > subnet."

$vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet

$vnet.Subnets[0].ServiceEndpoints  # Display the first endpoint.

###########################################################
##   Add the Virtual Service endpoint Id as a rule,      ##
##   into SQL Database ACLs.                             ##
###########################################################

Write-Host "Get the subnet object."

$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

$subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vnet

Write-Host "Add the subnet .Id as a rule, into the ACLs for your server."

$ruleParams = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
    VirtualNetworkSubnetId = $subnet.Id
}
New-AzSqlServerVirtualNetworkRule @ruleParams 

Write-Host "Verify that the rule is in the SQL Database ACL."

$rule2Params = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
}
Get-AzSqlServerVirtualNetworkRule @rule2Params

Write-Host 'Completed script 3, the "Virtual-Network-Rule".'
```

<a name="a-script-40"></a>

## <a name="script-4-clean-up"></a>Сценарий 4: очистка

Этот сценарий удаляет ресурсы, созданные предыдущим сценарии в целях демонстрации. Однако этот сценарий запрашивает подтверждение, прежде чем удалить следующие элементы:

- логический сервер SQL Server;
- группа ресурсов Azure.

Сценарий 4 можно запустить в любой момент после завершения сценария 1.

### <a name="powershell-script-4-source-code"></a>Исходный код сценария 4 PowerShell

```powershell
######### Script 4 ########################################
##   Clean-up phase A:  Unconditional deletes.           ##
##                                                       ##
##   1. The test rule is deleted from SQL Database ACL.        ##
##   2. The test endpoint is deleted from the subnet.    ##
##   3. The test virtual network is deleted.             ##
###########################################################

Write-Host "Delete the rule from the SQL Database ACL."

$removeParams = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
    ErrorAction            = 'SilentlyContinue'
}
Remove-AzSqlServerVirtualNetworkRule @removeParams

Write-Host "Delete the endpoint from the subnet."

$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

Remove-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vnet

Write-Host "Delete the virtual network (thus also deletes the subnet)."

$removeParams = @{
    Name              = $VNetName
    ResourceGroupName = $ResourceGroupName
    ErrorAction       = 'SilentlyContinue'
}
Remove-AzVirtualNetwork @removeParams

###########################################################
##   Clean-up phase B:  Conditional deletes.             ##
##                                                       ##
##   These might have already existed, so user might     ##
##   want to keep.                                       ##
##                                                       ##
##   1. Logical SQL server                        ##
##   2. Azure resource group                             ##
###########################################################

$yesno = Read-Host 'CAUTION !: Do you want to DELETE your server AND your resource group?  [yes/no]'
if ('yes' -eq $yesno) {
    Write-Host "Remove the server."

    $removeParams = @{
        ServerName        = $SqlDbServerName
        ResourceGroupName = $ResourceGroupName
        ErrorAction       = 'SilentlyContinue'
    }
    Remove-AzSqlServer @removeParams

    Write-Host "Remove the Azure Resource Group."
    
    Remove-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
} else {
    Write-Host "Skipped over the DELETE of SQL Database and resource group."
}

Write-Host 'Completed script 4, the "Clean-Up".'
```

<a name="a-actual-output"></a>

<a name="a-verify-subnet-is-endpoint-ps-100"></a>

## <a name="verify-your-subnet-is-an-endpoint"></a>Проверка того, является ли подсеть конечной точкой

Возможно, у вас есть подсеть, которой уже было назначено имя типа **Microsoft.Sql**, то есть она уже является конечной точкой службы виртуальной сети. Можно использовать [портал Azure][http-azure-portal-link-ref-477t], чтобы создать правило виртуальной сети для конечной точки.

Возможно, вы не уверены, присвоено ли подсети имя типа **Microsoft.Sql**. Можно выполнить приведенный ниже сценарий PowerShell, чтобы выполнить следующие действия.

1. Выясните, присвоено ли подсети имя типа **Microsoft.Sql**.
2. При необходимости назначьте имя типа, если оно отсутствует.
    - Сценарий запросит *подтверждение*, прежде чем применить отсутствующее имя типа.

### <a name="phases-of-the-script"></a>Этапы сценария

Ниже приведены этапы сценария PowerShell.

1. Вход в учетную запись Azure, требуется только один раз за сеанс PowerShell.  Присвоение значений переменных.
2. Поиск виртуальной сети, а затем подсети.
3. Подсеть помечена именем типа конечной точки службы **Microsoft.Sql**?
4. Добавление имени типа конечной точки службы виртуальной сети **Microsoft.Sql** в подсеть.

> [!IMPORTANT]
> Прежде чем запустить этот сценарий, необходимо изменить значения, присвоенные $-переменным в верхней части сценария.

### <a name="direct-powershell-source-code"></a>Исходный код прямолинейного сценария PowerShell

Этот сценарий PowerShell не обновляет ничего, пока вы не подтвердите это. Этот сценарий может добавить имя типа **Microsoft.Sql** в подсеть. Однако он пытается сделать это, только если у подсети отсутствует имя типа.

```powershell
### 1. LOG into to your Azure account, needed only once per PS session.  Assign variables.
$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]'
if ('yes' -eq $yesno) { Connect-AzAccount }

# Assignments to variables used by the later scripts.
# You can EDIT these values, if necessary.

$SubscriptionName = 'yourSubscriptionName'
Select-AzSubscription -SubscriptionName "$SubscriptionName"

$ResourceGroupName = 'yourRGName'
$VNetName = 'yourVNetName'
$SubnetName = 'yourSubnetName'
$SubnetAddressPrefix = 'Obtain this value from the Azure portal.' # Looks roughly like: '10.0.0.0/24'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql'  # Do NOT edit. Is official value.

### 2. Search for your virtual network, and then for your subnet.
# Search for the virtual network.
$vnet = $null
$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

if ($vnet -eq $null) {
    Write-Host "Caution: No virtual network found by the name '$VNetName'."
    return
}

$subnet = $null
for ($nn = 0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $subnet = $vnet.Subnets[$nn]
    if ($subnet.Name -eq $SubnetName) { break }
    $subnet = $null
}

if ($null -eq $subnet) {
    Write-Host "Caution: No subnet found by the name '$SubnetName'"
    Return
}

### 3. Is your subnet tagged as 'Microsoft.Sql' endpoint server type?
$endpointMsSql = $null
for ($nn = 0; $nn -lt $subnet.ServiceEndpoints.Count; $nn++) {
    $endpointMsSql = $subnet.ServiceEndpoints[$nn]
    if ($endpointMsSql.Service -eq $ServiceEndpointTypeName_SqlDb) {
        $endpointMsSql
        break
    }
    $endpointMsSql = $null
}

if ($null -eq $endpointMsSql) {
    Write-Host "Good: Subnet found, and is already tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'."
    return
} else {
    Write-Host "Caution: Subnet found, but not yet tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'."

    # Ask the user for confirmation.
    $yesno = Read-Host 'Do you want the PS script to apply the endpoint type name to your subnet?  [yes/no]'
    if ('no' -eq $yesno) { return }
}

### 4. Add a Virtual Service endpoint of type name 'Microsoft.Sql', on your subnet.
$setParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    VirtualNetwork  = $vnet
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$vnet = Set-AzVirtualNetworkSubnetConfig @setParams

# Persist the subnet update.
$vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet

for ($nn = 0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $vnet.Subnets[0].ServiceEndpoints # Display.
}
```

<!-- Link references: -->
[sql-db-vnet-service-endpoint-rule-overview-735r]:../vnet-service-endpoint-rule-overview.md
[http-azure-portal-link-ref-477t]: https://portal.azure.com/
