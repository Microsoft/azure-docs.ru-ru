---
title: Получение значений для проверки подлинности приложения
description: Создание субъекта-службы для доступа к базе данных SQL Azure из кода.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1 , devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: d264ee022d2833b092948ac0d1f3fef537bea943
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94841331"
---
# <a name="get-the-required-values-for-authenticating-an-application-to-access-azure-sql-database-from-code"></a>Получение необходимых значений для проверки подлинности приложения при доступе к базе данных SQL Azure из кода
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Чтобы создать базу данных SQL Azure и управлять ей из кода, необходимо зарегистрировать приложение в домене Azure Active Directory (Azure AD) в подписке, в которой были созданы ресурсы Azure.

## <a name="create-a-service-principal-to-access-resources-from-an-application"></a>Создание субъекта-службы для доступа к ресурсам из приложения

В следующих примерах создается приложение Active Directory (AD) и субъект-служба, которые необходимы для проверки подлинности приложения C#. Сценарий выводит значения, необходимые для предыдущего примера на C#. Дополнительные сведения см. в статье [Использование Azure PowerShell для создания субъекта-службы и доступа к ресурсам](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager (RM) по-прежнему поддерживается базой данных SQL, но вся будущая разработка предназначена для модуля AZ. SQL. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

```powershell
# sign in to Azure
Connect-AzAccount

# for multiple subscriptions, uncomment and set to the subscription you want to work with
#$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
#Set-AzContext -SubscriptionId $subscriptionId

$appName = "{app-name}" # display name for your app, must be unique in your directory
$uri = "http://{app-name}" # does not need to be a real uri
$secret = "{app-password}"

# create an AAD app
$azureAdApplication = New-AzADApplication -DisplayName $appName -HomePage $Uri -IdentifierUris $Uri -Password $secret

# create a Service Principal for the app
$svcprincipal = New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

Start-Sleep -s 15 # to avoid a PrincipalNotFound error, pause here for 15 seconds

# if you still get a PrincipalNotFound error, then rerun the following until successful.
$roleassignment = New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $azureAdApplication.ApplicationId.Guid

# output the values we need for our C# application to successfully authenticate
Write-Output "Copy these values into the C# sample app"

Write-Output "_subscriptionId:" (Get-AzContext).Subscription.SubscriptionId
Write-Output "_tenantId:" (Get-AzContext).Tenant.TenantId
Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
Write-Output "_applicationSecret:" $secret
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
# sign in to Azure
az login

# for multiple subscriptions, uncomment and set to the subscription you want to work with
#$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
#az account set --subscription $subscriptionId

$appName = "{app-name}" # display name for your app, must be unique in your directory
$uri = "http://{app-name}" # does not need to be a real uri
$secret = "{app-password}"

# create an AAD app
$azureAdApplication = az ad app create --display-name $appName --homepage $Uri --identifier-uris $Uri --password $secret

# create a Service Principal for the app
$svcprincipal = az ad sp create --id $azureAdApplication.ApplicationId

Start-Sleep -s 15 # to avoid a PrincipalNotFound error, pause for 15 seconds

# if you still get a PrincipalNotFound error, then rerun the following until successful.
$roleassignment = az role assignment create --role "Contributor" --assignee $azureAdApplication.ApplicationId.Guid

# output the values we need for our C# application to successfully authenticate
Write-Output "Copy these values into the C# sample app"

Write-Output "_subscriptionId:" (az account show --query "id")
Write-Output "_tenantId:" (az account show --query "tenantId")
Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
Write-Output "_applicationSecret:" $secret
```

* * *

## <a name="see-also"></a>См. также раздел

[Создание базы данных в базе данных SQL Azure с помощью C #](design-first-database-csharp-tutorial.md)  
[Подключение к базе данных SQL Azure с помощью Azure Active Directory проверки подлинности](authentication-aad-overview.md)
