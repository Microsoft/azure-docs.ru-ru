---
title: Псевдоним DNS (Azure CLI & PowerShell)
description: Командлеты PowerShell и Azure CLI позволяют перенаправлять новые клиентские подключения к другому серверу SQL Server в Azure без необходимости касаться какой-либо конфигурации клиента.
keywords: dns sql database
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.devlang: PowerShell
ms.topic: how-to
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: genemi, amagarwa, maboja, jrasnick, vanto
ms.date: 05/14/2019
ms.openlocfilehash: 02cfd839ed1b75fd85553f2e5a5150cadc29ff8e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92790463"
---
# <a name="powershell-for-dns-alias-to-azure-sql-database"></a>Управление псевдонимом DNS для Базы данных SQL Azure с помощью PowerShell
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

В этой статье представлен сценарий PowerShell, демонстрирующий способ управления псевдонимом DNS для [SQL Server](logical-servers.md) , на котором размещена база данных SQL Azure.

> [!NOTE]
> В эту статью были внесены изменения для использования модуля Azure PowerShell az или Azure CLI. Вы по-прежнему можете использовать модуль AzureRM, исправления ошибок для которого будут продолжать выпускаться как минимум до декабря 2020 г.
>
> Дополнительные сведения о модуле AZ и совместимости AzureRM см. в статье [Знакомство с модулем Azure PowerShell AZ](/powershell/azure/new-azureps-module-az). Инструкции по установке см. в разделе [Install Azure PowerShell](/powershell/azure/install-az-ps) или [Install Azure CLI](/cli/azure/install-azure-cli).

## <a name="dns-alias-in-connection-string"></a>Использование псевдонима DNS в строке подключения

Для подключения [логического сервера SQL](logical-servers.md)Server клиент, например SQL Server Management Studio (SSMS), может указать имя псевдонима DNS вместо действительного имени сервера. В следующем примере псевдоним *any-unique-alias-name* заменяет имя первого узла с разделителями-точками в строке сервера с четырьмя узлами.

   `<yourServer>.database.windows.net`

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить пример скрипта PowerShell, приведенный в этой статье, потребуются следующие компоненты:

- Подписка и учетная запись Azure для бесплатной пробной версии см. в статье пробная версия [Azure](https://azure.microsoft.com/free/) .
- Два сервера

## <a name="example"></a>Пример

Следующий пример кода начинается с присвоения литеральных значений нескольким переменным.

Чтобы выполнить код, измените значения заполнителей, чтобы они соответствовали реальным значениям в системе.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Используются следующие командлеты:

- [New-азсклсерверднсалиас](/powershell/module/az.Sql/New-azSqlServerDnsAlias): создает псевдоним DNS в системе службы базы данных SQL Azure. Псевдоним относится к серверу 1.
- [Get-азсклсерверднсалиас](/powershell/module/az.Sql/Get-azSqlServerDnsAlias): получение и перечисление всех псевдонимов, назначенных серверу 1.
- [Set-азсклсерверднсалиас](/powershell/module/az.Sql/Set-azSqlServerDnsAlias): изменяет имя сервера, на который ссылается псевдоним, с сервера 1 на сервер 2.
- [Remove-азсклсерверднсалиас](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): удалите псевдоним с сервера 2, используя имя псевдонима.

Чтобы выполнить установку или обновление, см. статью [Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps).

`Get-Module -ListAvailable Az`Чтобы найти версию, используйте в *\_ise.exePowerShell*.

```powershell
$subscriptionName = '<subscriptionName>';
$sqlServerDnsAliasName = '<aliasName>';
$resourceGroupName = '<resourceGroupName>';  
$sqlServerName = '<sqlServerName>';
$resourceGroupName2 = '<resourceGroupNameTwo>'; # can be same or different than $resourceGroupName
$sqlServerName2 = '<sqlServerNameTwo>'; # must be different from $sqlServerName.

# login to Azure
Connect-AzAccount -SubscriptionName $subscriptionName;
$subscriptionId = Get-AzSubscription -SubscriptionName $subscriptionName;

Write-Host 'Assign an alias to server 1...';
New-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName -ServerName $sqlServerName `
    -Name $sqlServerDnsAliasName;

Write-Host 'Get the aliases assigned to server 1...';
Get-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName -ServerName $sqlServerName;

Write-Host 'Move the alias from server 1 to server 2...';
Set-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -TargetServerName $sqlServerName2 `
    -Name $sqlServerDnsAliasName `
    -SourceServerResourceGroup $resourceGroupName -SourceServerName $sqlServerName `
    -SourceServerSubscriptionId $subscriptionId.Id;

Write-Host 'Get the aliases assigned to server 2...';
Get-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -ServerName $sqlServerName2;

Write-Host 'Remove the alias from server 2...';
Remove-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -ServerName $sqlServerName2 `
    -Name $sqlServerDnsAliasName;
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используются следующие команды:

- [AZ SQL Server DNS-Alias Create](/powershell/module/az.Sql/New-azSqlServerDnsAlias): создает псевдоним DNS для сервера. Псевдоним относится к серверу 1.
- [AZ SQL Server DNS-Alias демонстрация](/powershell/module/az.Sql/Get-azSqlServerDnsAlias): получение и перечисление всех псевдонимов, назначенных серверу 1.
- [AZ SQL Server DNS-Alias Set](/powershell/module/az.Sql/Set-azSqlServerDnsAlias): изменяет имя сервера, на который ссылается псевдоним, с сервера 1 на сервер 2.
- [AZ SQL Server DNS-Alias Delete](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): удалите псевдоним с сервера 2, используя имя псевдонима.

Чтобы выполнить установку или обновление, см. сведения в статье [Установка Azure CLI](/cli/azure/install-azure-cli).

```azurecli-interactive
$subscriptionName = '<subscriptionName>';
$sqlServerDnsAliasName = '<aliasName>';
$resourceGroupName = '<resourceGroupName>';  
$sqlServerName = '<sqlServerName>';
$resourceGroupName2 = '<resourceGroupNameTwo>'; # can be same or different than $resourceGroupName
$sqlServerName2 = '<sqlServerNameTwo>'; # must be different from $sqlServerName.

# login to Azure
az login -SubscriptionName $subscriptionName;
$subscriptionId = az account list[0].i -SubscriptionName $subscriptionName;

Write-Host 'Assign an alias to server 1...';
az sql server dns-alias create –-resource-group $resourceGroupName --server $sqlServerName `
    --name $sqlServerDnsAliasName;

Write-Host 'Get the aliases assigned to server 1...';
az sql server dns-alias show –-resource-group $resourceGroupName --server $sqlServerName;

Write-Host 'Move the alias from server 1 to server 2...';
az sql server dns-alias set –-resource-group $resourceGroupName2 --server $sqlServerName2 `
    --name $sqlServerDnsAliasName `
    --original-resource-group $resourceGroupName --original-server $sqlServerName `
    --original-subscription-id $subscriptionId.Id;

Write-Host 'Get the aliases assigned to server 2...';
az sql server dns-alias show –-resource-group $resourceGroupName2 --server $sqlServerName2;

Write-Host 'Remove the alias from server 2...';
az sql server dns-alias delete –-resource-group $resourceGroupName2 --server $sqlServerName2 `
    --name $sqlServerDnsAliasName;
```

* * *

## <a name="next-steps"></a>Дальнейшие действия

Полное описание функции псевдонима DNS для базы данных SQL см. в статье [псевдоним DNS для базы данных SQL Azure](./dns-alias-overview.md).