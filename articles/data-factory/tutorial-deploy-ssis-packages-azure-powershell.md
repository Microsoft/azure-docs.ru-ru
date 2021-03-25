---
title: Настройка Azure-SSIS Integration Runtime с помощью PowerShell
description: Узнайте, как с помощью PowerShell настроить Azure-SSIS Integration Runtime в Фабрике данных Azure, чтобы развертывать и запускать пакеты служб SSIS в Azure.
ms.service: data-factory
ms.devlang: powershell
ms.topic: tutorial
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 10/13/2020
author: swinarko
ms.author: sawinark
ms.openlocfilehash: a0a6e115d2b76ab3f060ee1bcb197da58f2c8fa6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100391292"
---
# <a name="set-up-an-azure-ssis-ir-in-azure-data-factory-by-using-powershell"></a>Настройка Azure-SSIS IR в Фабрике данных Azure с помощью PowerShell

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве описано, как с помощью PowerShell подготовить среду выполнения интеграции Azure SQL Server Integration Services (SSIS) в Фабрике данных Azure. Среда выполнения интеграции Azure-SSIS поддерживает следующие возможности:

- выполнение пакетов, развернутых в каталоге SSIS (SSISDB), которые размещаются на сервере Базы данных SQL Azure или в Управляемом экземпляре (модель развертывания для проектов);
- выполнение пакетов, развернутых в файловой системе, службе "Файлы Azure" или базе данных SQL Server (MSDB), которые размещаются в Управляемом экземпляре SQL Azure (модель развертывания для пакетов).

После подготовки Azure-SSIS IR вы можете использовать для развертывания и запуска пакетов в Azure любые привычные средства, уже поддерживающие Azure, например SQL Server Data Tools (SSDT), SQL Server Management Studio (SSMS) или служебные программы командной строки, такие как [dtutil](/sql/integration-services/dtutil-utility) и [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md).

См. дополнительные сведения о [среде выполнения интеграции Azure SSIS](concepts-integration-runtime.md#azure-ssis-integration-runtime).

> [!NOTE]
> В этой статье объясняется, как настроить Azure-SSIS IR с помощью Azure PowerShell. Инструкции по настройке Azure-SSIS IR с помощью портала Azure или Фабрики данных Azure см. в [этом руководстве](./tutorial-deploy-ssis-packages-azure.md). 

Выполняя данное руководство, вы сделаете следующее:
> [!div class="checklist"]
> * Создали фабрику данных.
> * Создавать Azure-SSIS Integration Runtime.
> * Запускать Azure-SSIS Integration Runtime.
> * Выполнять проверку всего скрипта.
> * Развертывать пакеты служб SSIS.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Подписка Azure**. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

- **Сервер Базы данных SQL Azure или Управляемый экземпляр (необязательно).** Если у вас еще нет сервера базы данных, создайте его на портале Azure перед началом работы. Фабрика данных, в свою очередь, создаст экземпляр SSISDB на этом сервере базы данных. 

  Мы рекомендуем создать сервер базы данных в одном регионе Azure со средой интеграции. Эта конфигурация позволяет среде выполнения интеграции записывать журналы выполнения в SSISDB, не пересекая регионы Azure.

  Учитывайте следующие моменты:

  - В зависимости от выбранного сервера базы данных экземпляр SSISDB может быть создан от вашего имени как отдельная база данных, как часть эластичного пула или в виде управляемого экземпляра. Доступ к нему можно получить через общедоступную сеть или после присоединения к виртуальной сети. Инструкции по выбору типа сервера базы данных для размещения SSISDB см. в разделе [Сравнение Базы данных SQL и Управляемого экземпляра SQL](create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance). 
  
    Если сервер Базы данных SQL Azure используется с правилами брандмауэра для IP-адресов или конечными точками служб в виртуальной сети или с управляемым экземпляром и частной конечной точкой для размещения SSISDB, или вам нужен доступ к локальным данным без настройки локальной среды выполнения интеграции, необходимо присоединить среду выполнения интеграции Azure-SSIS к виртуальной сети. См. сведения о [создании среды выполнения интеграции Azure SSIS в виртуальной сети](./create-azure-ssis-integration-runtime.md).

  - Убедитесь, что для сервера базы данных включен параметр **Разрешить доступ к службам Azure**. Этот параметр не применяется, если сервер Базы данных SQL Azure используется с правилами брандмауэра для IP-адресов или конечными точками служб для виртуальной сети или с управляемым экземпляром и частной конечной точкой для размещения базы данных SSISDB. Дополнительные сведения см. в разделе [Создание правила брандмауэра](../azure-sql/database/secure-database-tutorial.md#create-firewall-rules). Сведения о включении этого параметра с помощью PowerShell см. в статье [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

  - Добавьте IP-адрес клиентского компьютера или диапазон IP-адресов, который включает IP-адрес клиентского компьютера, в список IP-адресов клиента в параметрах брандмауэра для сервера базы данных. Дополнительные сведения см. в разделе [Правила брандмауэра уровня сервера Базы данных SQL Azure и уровня базы данных SQL Azure](../azure-sql/database/firewall-configure.md).

  - К серверу базы данных можно подключиться с использованием аутентификации SQL и учетных данных администратора сервера или аутентификации AAD и управляемого удостоверения фабрики данных. Для последнего варианта управляемое удостоверение ADF нужно добавить в группу AAD с разрешениями на доступ к серверу базы данных. См. сведения о [создании среды выполнения интеграции Azure SSIS с использованием аутентификации Azure AD](./create-azure-ssis-integration-runtime.md).

  - Убедитесь, что на сервере базы данных еще нет экземпляра SSISDB. При подготовке Azure-SSIS IR не поддерживается использование существующего экземпляра SSISDB.

- **Azure PowerShell**. Если вы хотите настроить Azure-SSIS IR с помощью скрипта PowerShell, следуйте инструкциям из статьи [Установка модуля Azure PowerShell](/powershell/azure/install-Az-ps).

> [!NOTE]
> Список регионов Azure, в которых сейчас доступны Фабрика данных и Azure-SSIS IR, см. на [странице доступности продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all). 

## <a name="open-the-windows-powershell-ise"></a>Открытие интегрированной среды сценариев Windows PowerShell

Откройте интегрированную среду сценариев (ISE) Windows PowerShell с разрешениями администратора. 

## <a name="create-variables"></a>Создание переменных

Скопируйте приведенный ниже скрипт в ISE. Укажите значения для переменных. 

```powershell
### Azure Data Factory info
# If your input contains a PSH special character (for example, "$"), precede it with the escape character "`" (for example, "`$")
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - Must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS Integration Runtime info; this is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2 
# Azure-SSIS IR edition/license info: Standard or Enterprise 
$AzureSSISEdition = "Standard" # Standard by default, although Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your existing SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to (2 x number of cores) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script

### SSISDB info
$SSISDBServerEndpoint = "[your server name.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you're not using SSISDB]" # WARNING: If you use SSISDB, please ensure that there is no existing SSISDB on your database server, so we can prepare and manage one on your behalf    
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication]"
# For the basic pricing tier, specify "Basic", not "B" - For standard/premium/elastic pool tiers, specify "S0", "S1", "S2", "S3", etc., see https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for SQL Database or leave it empty for SQL Managed Instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 
```

## <a name="sign-in-and-select-your-subscription"></a>Вход и выбор подписки

Чтобы войти и выбрать подписку Azure, добавьте в скрипт следующий код:

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName
```

## <a name="validate-the-connection-to-your-database-server"></a>Проверка подключения к серверу базы данных

Чтобы проверить подключение, добавьте следующий скрипт. 

```powershell
# Validate only if you're using SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword    
    $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
    Try
    {
        $sqlConnection.Open();
    }
    Catch [System.Data.SqlClient.SqlException]
    {
        Write-Warning "Cannot connect, exception: $_";
        Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
        $yn = Read-Host
        if(!($yn -ieq "Y"))
        {
            Return;
        } 
    }
}
```

Создание экземпляра Базы данных Azure SQL в рамках скрипта показано в примере ниже. Задайте значения для переменных, которые еще не определены (например, SSISDBServerName, FirewallIPAddress). 

```powershell
New-AzSqlServer -ResourceGroupName $ResourceGroupName `
    -ServerName $SSISDBServerName `
    -Location $DataFactoryLocation `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $SSISDBServerAdminUserName, $(ConvertTo-SecureString -String $SSISDBServerAdminPassword -AsPlainText -Force))    

New-AzSqlServerFirewallRule -ResourceGroupName $ResourceGroupName `
    -ServerName $SSISDBServerName `
    -FirewallRuleName "ClientIPAddress_$today" -StartIpAddress $FirewallIPAddress -EndIpAddress $FirewallIPAddress

New-AzSqlServerFirewallRule -ResourceGroupName $ResourceGroupName -ServerName $SSISDBServerName -AllowAllAzureIPs
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

Если группа ресурсов уже существует, не копируйте этот код в скрипт. 

```powershell
New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
```

## <a name="create-a-data-factory"></a>Создание фабрики данных

Чтобы создать фабрику данных, выполните следующие команды:

```powershell
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName
```

## <a name="create-an-azure-ssis-integration-runtime"></a>Создание Azure-SSIS Integration Runtime

Чтобы создать среду Azure-SSIS Integration Runtime, запускающую пакеты служб SSIS в Azure, выполните приведенные ниже команды. Если вы не используете SSISDB, вы можете опустить параметры CatalogServerEndpoint, CatalogPricingTier и CatalogAdminCredential.

```powershell
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode

# Add CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you're using SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
    $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier `
        -CatalogAdminCredential $serverCreds
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Standard")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Extended")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data access
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}
```

## <a name="start-the-azure-ssis-integration-runtime"></a>Запуск Azure-SSIS Integration Runtime

Чтобы запустить Azure-SSIS IR, выполните следующие команды:

```powershell
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")                                  
```

> [!NOTE]
> Этот процесс должен завершиться в течение пяти минут, не считая времени на пользовательские процессы настройки.
>
> Если вы используете SSISDB, Фабрика данных Azure подключится к серверу базы данных для подготовки SSISDB. 
> 
> При подготовке Azure-SSIS IR также устанавливается распространяемый компонент Access и пакет функций Azure для служб SSIS. Эти компоненты обеспечивают подключение к файлам Excel или Access и другим источникам данных Azure помимо тех, которые уже поддерживаются встроенными компонентами. Дополнительные сведения о встроенных и предварительно установленных компонентах см. в статье [Встроенные и предварительно установленные компоненты в Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md). Дополнительные сведения о других компонентах, доступных для установки, см. в руководстве по [пользовательской установке для среды выполнения интеграции Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

## <a name="full-script"></a>Полный сценарий

Скрипт PowerShell в этом разделе настраивает экземпляр Azure-SSIS IR, который запускает пакеты SSIS. После успешного запуска скрипта можно развернуть и выполнить пакет SSIS в Azure.

1. Откройте ISE.
2. В командной строке ISE выполните следующую команду:  

    ```powershell
    Set-ExecutionPolicy Unrestricted -Scope CurrentUser
    ```

3. Скопируйте скрипт PowerShell из этого раздела и вставьте его в ISE.
4. Укажите соответствующие значения для всех параметров в начале скрипта.
5. Выполните скрипт. 

```powershell
### Azure Data Factory info
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - Must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS Integration Runtime info - This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2 
# Azure-SSIS IR edition/license info: Standard or Enterprise 
$AzureSSISEdition = "Standard" # Standard by default, while Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your existing SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to (2 x the number of cores) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script

### SSISDB info
$SSISDBServerEndpoint = "[your server name.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you're not using SSISDB]" # WARNING: If you want to use SSISDB, ensure that there is no existing SSISDB on your database server, so we can prepare and manage one on your behalf    
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication]"
# For the basic pricing tier, specify "Basic", not "B" - For standard/premium/elastic pool tiers, specify "S0", "S1", "S2", "S3", etc., see https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for SQL Database or leave it empty for SQL Managed Instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

### Sign in and select subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName

### Validate the connection to database server
# Validate only if you're using SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword    
    $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
    Try
    {
        $sqlConnection.Open();
    }
    Catch [System.Data.SqlClient.SqlException]
    {
        Write-Warning "Cannot connect, exception: $_";
        Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
        $yn = Read-Host
        if(!($yn -ieq "Y"))
        {
            Return;
        } 
    }
}

### Create a data factory
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName

### Create an integration runtime
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode

# Add CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you're using SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
    $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier `
        -CatalogAdminCredential $serverCreds
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Standard")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Extended")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data access
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}

### Start integration runtime
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")   
```

## <a name="monitor-and-manage-your-azure-ssis-ir"></a>Мониторинг среды Azure-SSIS IR и управление ею

Дополнительные сведения о мониторинге среды Azure-SSIS IR и управлении ею см. в следующих статьях: 

- [Мониторинг Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime)
- [Управление Azure-SSIS IR](manage-azure-ssis-integration-runtime.md)

## <a name="deploy-ssis-packages"></a>Развертывание пакетов служб SSIS.

Если вы используете SSISDB, можете развернуть в нее свои пакеты и запустить их в Azure-SSIS IR с помощью средств SSDT или SSMS с поддержкой Azure. Эти средства подключаются к серверу базы данных через конечную точку сервера. 

- Для сервера Базы данных SQL Azure используется формат конечной точки `<server name>.database.windows.net`.
- Для управляемого экземпляра с частной конечной точкой используется формат `<server name>.<dns prefix>.database.windows.net`.
- Для управляемого экземпляра с общедоступной конечной точкой используется формат `<server name>.public.<dns prefix>.database.windows.net,3342`. 

Если вы не используете SSISDB, вы можете развертывать пакеты в файловой системе, службе "Файлы Azure" или базе данных MSDB, размещенной в Управляемом экземпляре SQL Azure, и запускать их в Azure-SSIS IR с помощью программ командной строки [dtutil](/sql/integration-services/dtutil-utility) и [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md). 

Дополнительные сведения см. в статье [Развертывание проектов и пакетов служб Integration Services (SSIS)](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages).

В обоих случаях вы также можете запустить развернутые пакеты в Azure-SSIS IR, используя действие "Выполнить пакеты служб SSIS" в конвейерах Фабрики данных Azure. См. сведения о [запуске выполнения пакета SSIS с использованием действия Фабрики данных](./how-to-invoke-ssis-package-ssis-activity.md).

Дополнительная документация по SSIS: 

- [Руководство. Развертывание и выполнение пакета служб SQL Server Integration Services (SSI) в Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Connect to on-premises data sources with Windows Authentication](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) (Подключение к локальным источникам данных с помощью аутентификации Windows) 
- [Планирование выполнения пакетов служб SQL Server Integration Services (SSIS), развернутых в Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи: 

> [!div class="checklist"]
> * Создали фабрику данных.
> * Создавать Azure-SSIS Integration Runtime.
> * Запускать Azure-SSIS Integration Runtime.
> * Выполнять проверку всего скрипта.
> * Развертывать пакеты служб SSIS.

Дополнительные сведения о настройке Azure-SSIS Integration Runtime см. в следующей статье:

> [!div class="nextstepaction"]
>[Настройка Azure SSIS IR](./how-to-configure-azure-ssis-ir-custom-setup.md)