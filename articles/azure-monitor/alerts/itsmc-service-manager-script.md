---
title: Создание веб-приложения для соединителя управления службами
description: Создание веб-приложения Service Manager с помощью автоматизированного скрипта для подключения к Azure с помощью соединителя управления ИТ-услугами, а также централизованный мониторинг и управление рабочими элементами ITSM.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 01/23/2018
ms.openlocfilehash: 16b48ac213168eaa58de5b0603025b2d48bb5ed5
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102041677"
---
# <a name="create-service-manager-web-app-using-the-automated-script"></a>Создание веб-приложения Service Manager с помощью автоматизированного скрипта

Чтобы создать веб-приложение для экземпляра Service Manager используйте следующий скрипт. См. дополнительные сведения о подключении [Service Manager](./itsmc-connections-scsm.md).

Запустите скрипт, указав следующие данные:

- сведения о подписке Azure;
- Имя группы ресурсов
- Расположение
- сведения о сервере Service Manager (имя сервера, домен, имя пользователя и пароль);
- префикс имени сайта для веб-приложения;
- пространство имен ServiceBus.

Скрипт создает веб-приложение, используя имя, которое вы указали (с несколькими дополнительными строками, обеспечивающими его уникальность). При этом создается **URL-адрес приложения**, **идентификатор клиента** и **секрет клиента**.

Сохраните эти значения. Вы сможете их использовать при создании подключения с помощью Соединителя управления ИТ-услугами.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

 Windows Management Framework версии 5.0 или выше.
По умолчанию в Windows 10 используется версия 5.1. Скачать нужную версию можно [отсюда](https://www.microsoft.com/download/details.aspx?id=50395):

Используйте следующий скрипт:

```powershell
####################################
# User Configuration Section Begins
####################################

# Subscription name in Azure account. Check in Azure Portal.
$azureSubscriptionName = ""

# Resource group name for resource deployment. Could be an existing resource group or a new one to be created.
$resourceGroupName = ""

# Location for existing resource group or new resource group deployment
################################### List of available regions #################################################
# centralus,eastasia,southeastasia,eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,
# northeurope,westeurope,japaneast,japanwest,brazilsouth,australiasoutheast,australiaeast,westindia,southindia,
# centralindia,canadacentral,canadaeast,uksouth,ukwest.
###############################################################################################################
$location = ""

# Service Manager Authentication Settings
$serverName = ""
$domain = ""
$username = ""
$password = ""


# Azure site Name Prefix. Default is "smoc". It can be configured to any desired value.
$siteNamePrefix = ""

# Service Bus namespace. Please provide an already existing service bus namespace.
# If it doesn't exist, a new one will be created with name $siteName + "sbn" which can also be later reused for any other hybrid connections.
$serviceName = ""

##################################
# User Configuration Section Ends
##################################

################
# Installations
################

# Allowing the execution of the script for current user.  
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

Write-Host "Checking for required modules..."
if(!(Get-PackageProvider -Name NuGet))
{
   Write-Host "Installing NuGet Package Provider..."
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Scope CurrentUser -Force -WarningAction SilentlyContinue
}
$module = Get-Module -ListAvailable -Name Az

if(!$module -or ($module[0].Version.Major -lt 1))
{
    Write-Host "Installing Az Module..."  
    try
    {
        # In case of Win 10 Anniversary update
        Install-Module Az -MinimumVersion 1.0 -Scope CurrentUser -Force -WarningAction SilentlyContinue -AllowClobber
    }
    catch
    {
        Install-Module Az -MinimumVersion 1.0 -Scope CurrentUser -Force -WarningAction SilentlyContinue
    }

}

Write-Host "Requirement check complete!!"

#############
# Parameters
#############

$errorActionPreference = "Stop"

$templateUri = "https://raw.githubusercontent.com/Azure/SMOMSConnector/master/azuredeploy.json"

if(!$siteNamePrefix)
{
    $siteNamePrefix = "smoc"
}

Connect-AzAccount

$context = Set-AzContext -SubscriptionName $azureSubscriptionName -WarningAction SilentlyContinue

$resourceProvider = Get-AzResourceProvider -ProviderNamespace Microsoft.Web

if(!$resourceProvider -or $resourceProvider[0].RegistrationState -ne "Registered")
{
    try
    {
        Write-Host "Registering Microsoft.Web Resource Provider"
        Register-AzResourceProvider -ProviderNamespace Microsoft.Web
    }
    catch
    {
        Write-Host "Failed to Register Microsoft.Web Resource Provider. Please register it in Azure Portal."
        exit
    }
}
do
{
    $rand = Get-Random -Maximum 32000

    $siteName = $siteNamePrefix + $rand

    $resource = Get-AzResource -Name $siteName -ResourceType Microsoft.Web/sites

}while($resource)

$azureSite = "https://"+$siteName+".azurewebsites.net"

##############
# MAIN Begins
##############

# Web App Deployment
####################

$tenant = $context.Tenant.Id
if(!$tenant)
{
    #For backward compatibility with older versions
    $tenant = $context.Tenant.TenantId
}
try
{
    Get-AzResourceGroup -Name $resourceGroupName
}
catch
{
    New-AzResourceGroup -Location $location -Name $resourceGroupName
}

Write-Output "Web App Deployment in progress...."

New-AzResourceGroupDeployment -TemplateUri $templateUri -siteName $siteName -ResourceGroupName $resourceGroupName

Write-Output "Web App Deployed successfully!!"

# AAD Authentication
####################

Add-Type -AssemblyName System.Web

$secret = [System.Web.Security.Membership]::GeneratePassword(30,2).ToString()
$clientSecret = $secret | ConvertTo-SecureString -AsPlainText -Force


try
{

    Write-Host "Creating AzureAD application..."

    $adApp = New-AzADApplication -DisplayName $siteName -HomePage $azureSite -IdentifierUris $azureSite -Password $clientSecret

    Write-Host "AzureAD application created successfully!!"
}
catch
{
    # Delete the deployed web app if Azure AD application fails
    Remove-AzResource -ResourceGroupName $resourceGroupName -ResourceName $siteName -ResourceType Microsoft.Web/sites -Force

    Write-Host "Failure occurred in Azure AD application....Try again!!"

    exit

}

$clientId = $adApp.ApplicationId

$servicePrincipal = New-AzADServicePrincipal -ApplicationId $clientId

# Web App Configuration
#######################
try
{

    Write-Host "Configuring deployed Web-App..."
    $webApp = Get-AzWebAppSlot -ResourceGroupName $resourceGroupName -Name $siteName -Slot production -WarningAction SilentlyContinue

    $appSettingList = $webApp.SiteConfig.AppSettings

    $appSettings = @{}
    ForEach ($item in $appSettingList) {
        $appSettings[$item.Name] = $item.Value
    }

    $appSettings['ida:Tenant'] = $tenant
    $appSettings['ida:Audience'] = $azureSite
    $appSettings['ida:ServerName'] = $serverName
    $appSettings['ida:Domain'] = $domain
    $appSettings['ida:Username'] = $userName
    $appSettings['ida:WhitelistedClientId'] = $clientId

    $connStrings = @{}
    $kvp = @{"Type"="Custom"; "Value"=$password}
    $connStrings['ida:Password'] = $kvp

    Set-AzWebAppSlot -ResourceGroupName $resourceGroupName -Name $siteName -AppSettings $appSettings -ConnectionStrings $connStrings -Slot production -WarningAction SilentlyContinue

}
catch
{
    Write-Host "Web App configuration failed. Please ensure all values are provided in Service Manager Authentication Settings in User Configuration Section"

    # Delete the AzureRm AD Application if configuration fails
    Remove-AzADApplication -ObjectId $adApp.ObjectId -Force

    # Delete the deployed web app if configuration fails
    Remove-AzResource -ResourceGroupName $resourceGroupName -ResourceName $siteName -ResourceType Microsoft.Web/sites -Force

    exit
}


# Relay Namespace
###################

if(!$serviceName)
{
    $serviceName = $siteName + "sbn"
}

$resourceProvider = Get-AzResourceProvider -ProviderNamespace Microsoft.Relay

if(!$resourceProvider -or $resourceProvider[0].RegistrationState -ne "Registered")
{
    try
    {
        Write-Host "Registering Microsoft.Relay Resource Provider"
        Register-AzResourceProvider -ProviderNamespace Microsoft.Relay
    }
    catch
    {
        Write-Host "Failed to Register Microsoft.Relay Resource Provider. Please register it in Azure Portal."
    }
}

$resource = Get-AzResource -Name $serviceName -ResourceType Microsoft.Relay/namespaces

if(!$resource)
{
    $serviceName = $siteName + "sbn"
    $properties = @{
        "sku" = @{
            "name"= "Standard"
            "tier"= "Standard"
            "capacity"= 1
         }
    }
    try
    {
        Write-Host "Creating Service Bus namespace..."
        New-AzResource -ResourceName $serviceName -Location $location -PropertyObject $properties -ResourceGroupName $resourceGroupName -ResourceType Microsoft.Relay/namespaces -ApiVersion 2016-07-01 -Force
    }
    catch
    {
        $err = $TRUE
        Write-Host "Creation of Service Bus Namespace failed...Please create it manually from Azure Portal.`n"
    }

}

Write-Host "Note: Please Configure Hybrid connection in the Networking section of the web application in Azure Portal to link to the on-premises system.`n"
Write-Host "App Details"
Write-Host "============"
Write-Host "App Name:"  $siteName
Write-Host "Client Id:"  $clientId
Write-Host "Client Secret:"  $secret
Write-Host "URI:"  $azureSite
if(!$err)
{
    Write-Host "ServiceBus Namespace:"  $serviceName  
}
```

## <a name="troubleshoot-service-manager-web-app-deployment"></a>Устранение неполадок Service Manager развертывании веб-приложений

-   Если у вас возникли проблемы с развертыванием веб-приложения, убедитесь, что у вас есть разрешения на создание и развертывание ресурсов в подписке.
-   При получении ссылки на **объект, не установленной на экземпляр** ошибки при выполнении [скрипта](itsmc-service-manager-script.md), убедитесь, что введены допустимые значения в разделе " **Конфигурация пользователя** ".
-   Если не удалось создать пространство имен ретранслятора служебной шины, убедитесь, что требуемый поставщик ресурсов зарегистрирован в подписке. Если оно не зарегистрировано, вручную создайте пространство имен ретранслятора служебной шины из портал Azure. Его также можно создать при [создании гибридного подключения](./itsmc-connections-scsm.md#configure-the-hybrid-connection) в портал Azure.

## <a name="next-steps"></a>Дальнейшие действия
[Настройте гибридное подключение](./itsmc-connections-scsm.md#configure-the-hybrid-connection).
