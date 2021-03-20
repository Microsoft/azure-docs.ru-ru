---
title: Управление ресурсами API Azure Cosmos DB Core (SQL) с помощью PowerShell
description: Управление ресурсами API Azure Cosmos DB Core (SQL) с помощью PowerShell.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: seodec18
ms.openlocfilehash: 2d99a760e3bf44febcee05bf9827479616bf7bd6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93333449"
---
# <a name="manage-azure-cosmos-db-core-sql-api-resources-using-powershell"></a>Управление ресурсами API Azure Cosmos DB Core (SQL) с помощью PowerShell
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В следующем разделе описано, как использовать PowerShell для создания скриптов и автоматизации управления ресурсами API Azure Cosmos DB Core (SQL), включая учетную запись Cosmos, базу данных, контейнер и пропускную способность. Командлеты PowerShell для других интерфейсов API см. в статьях [Примеры PowerShell для Cassandra](powershell-samples-cassandra.md), [Примеры PowerShell для API MongoDB](powershell-samples-mongodb.md), [Примеры PowerShell для Gremlin](powershell-samples-gremlin.md) и [Примеры PowerShell для таблиц](powershell-samples-table.md).

> [!NOTE]
> В примерах этой статьи используются командлеты управления [Az.CosmosDB](/powershell/module/az.cosmosdb). Сведения о последних изменениях см. на странице справочника по API [Az.CosmosDB](/powershell/module/az.cosmosdb).

Для управления ресурсами Azure Cosmos DB на нескольких платформах можно использовать командлеты `Az` и `Az.CosmosDB` в [кроссплатформенном средстве PowerShell](/powershell/scripting/install/installing-powershell), а также [Azure CLI](manage-with-cli.md), [REST API][rp-rest-api] или [портал Azure](create-sql-api-dotnet.md#create-account).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="getting-started"></a>Приступая к работе

Следуйте инструкциям из статьи об [установке и настройке Azure PowerShell][powershell-install-configure], чтобы установить средство PowerShell и войти через него в учетную запись Azure.

> [!IMPORTANT]
> Azure Cosmos DB ресурсы не могут быть переименованы, так как это нарушает работу Azure Resource Manager с URI ресурсов.

## <a name="azure-cosmos-accounts"></a>Учетные записи Azure Cosmos

В следующих разделах описано управление учетной записью Azure Cosmos, в том числе такие действия:

* [создание учетной записи Azure Cosmos;](#create-account)
* [обновление учетной записи Azure Cosmos;](#update-account)
* [вывод списка всех учетных записей Azure Cosmos в подписке;](#list-accounts)
* [создание учетной записи Azure Cosmos;](#get-account)
* [Удаление учетной записи Azure Cosmos](#delete-account)
* [обновление тегов учетной записи Azure Cosmos;](#update-tags)
* [вывод списка ключей учетной записи Azure Cosmos;](#list-keys)
* [повторное создание ключей для учетной записи Azure Cosmos;](#regenerate-keys)
* [вывод списка строк подключения для учетной записи Azure Cosmos;](#list-connection-strings)
* [изменение приоритета при отработке отказа в учетной записи Azure Cosmos.](#modify-failover-priority)
* [активация перехода на другой ресурс для учетной записи Azure Cosmos](#trigger-manual-failover)
* [вывод списка блокировок ресурсов в учетной записи Azure Cosmos DB.](#list-account-locks)

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a> Создание учетной записи Azure Cosmos

Эта команда создает учетную запись базы данных Azure Cosmos DB с поддержкой [множества регионов][distribute-data-globally], [автоматического перехода на другой ресурс](how-to-manage-database-account.md#automatic-failover) и [политики согласованности](consistency-levels.md) с ограниченным устареванием.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "BoundedStaleness"
$maxStalenessInterval = 300
$maxStalenessPrefix = 100000
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$true `
    -DefaultConsistencyLevel $consistencyLevel `
    -MaxStalenessIntervalInSeconds $maxStalenessInterval `
    -MaxStalenessPrefix $maxStalenessPrefix
```

* `$resourceGroupName` — группа ресурсов Azure, куда нужно развернуть учетную запись Cosmos. Этот ресурс должен уже существовать.
* `$locations` Регионы для учетной записи базы данных — регион, в котором `FailoverPriority 0` находится регион записи.
* `$accountName`: имя учетной записи Azure Cosmos. Имя должно быть уникальным, состоять из букв нижнего регистра, содержать только буквенно-цифровые символы и символ "-", а также иметь длину от 3 до 31 символа.
* `$apiKind` — тип учетной записи Cosmos, которую нужно создать. Дополнительные сведения см. в описании [API в Cosmos DB](introduction.md#simplified-application-development).
* `$consistencyPolicy`, `$maxStalenessInterval` и `$maxStalenessPrefix` — уровень согласованности и параметры по умолчанию для учетной записи Azure Cosmos. Дополнительные сведения см. в статье о [настраиваемых уровнях согласованности в Azure Cosmos DB](consistency-levels.md).

Для учетных записей Azure Cosmos можно настроить брандмауэр IP-адресов, конечные точки службы для виртуальной сети и (или) частные конечные точки. Сведения о настройке брандмауэра IP-адресов для Azure Cosmos DB см. в [этой статье](how-to-configure-firewall.md). Сведения о включении конечных точек службы для Azure Cosmos DB см. в статье [Настройка доступа из виртуальных сетей](how-to-configure-vnet-service-endpoint.md). Сведения о включении частных конечных точек для Azure Cosmos DB см. в статье [Настройка доступа из частных конечных точек](how-to-configure-private-endpoints.md).

### <a name="list-all-azure-cosmos-accounts-in-a-resource-group"></a><a id="list-accounts"></a> Вывод списка всех учетных записей Azure Cosmos в группе ресурсов

Эта команда выводит список всех учетных записей Azure Cosmos в группе ресурсов.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName
```

### <a name="get-the-properties-of-an-azure-cosmos-account"></a><a id="get-account"></a> Получение свойств учетной записи Azure Cosmos

Эта команда позволяет получить свойства имеющейся учетной записи Azure Cosmos.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName
```

### <a name="update-an-azure-cosmos-account"></a><a id="update-account"></a> Обновление учетной записи Azure Cosmos

Эта команда позволяет обновить свойства учетной записи базы данных Azure Cosmos DB. В контексте свойств можно выполнить такие изменения:

* добавление или удаление регионов;
* изменение политики согласованности по умолчанию;
* изменение фильтра диапазона IP-адресов;
* изменение конфигурации виртуальной сети;
* Включение операций записи в несколько регионов

> [!NOTE]
> Вы не можете одновременно добавлять или удалять регионы (`locations`) и изменять другие свойства для учетной записи Azure Cosmos. Изменение регионов нужно выполнять как отдельную операцию, не объединяя ее с другими изменениями в учетной записи.
> [!NOTE]
> Кроме того, эта команда позволяет добавлять или удалять регионы, но не изменять приоритеты при отработке отказа или активировать отработку отказа вручную. Подробнее см. разделы [об изменении приоритетов при отработке отказа](#modify-failover-priority) и [об активации отработки отказа вручную](#trigger-manual-failover).

```azurepowershell-interactive
# Create account with two regions
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "Session"
$enableAutomaticFailover = $true
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

# Create the Cosmos DB account
New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$enableAutomaticFailover `
    -DefaultConsistencyLevel $consistencyLevel

# Add a region to the account
$locationObject2 = @()
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 2 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject2

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
Write-Host "When region was added, press any key to continue."
$HOST.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown") | OUT-NULL
$HOST.UI.RawUI.Flushinputbuffer()

# Remove West US region from the account
$locationObject3 = @()
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 1 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject3

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
```

### <a name="enable-multiple-write-regions-for-an-azure-cosmos-account"></a><a id="multi-region-writes"></a> Включение нескольких регионов записи для учетной записи Azure Cosmos

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $false
$enableMultiMaster = $true

# First disable automatic failover - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover

# Now enable multi-region writes
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster
```

### <a name="delete-an-azure-cosmos-account"></a><a id="delete-account"></a> Удаление учетной записи Azure Cosmos

Эта команда удаляет имеющуюся учетную запись Azure Cosmos.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Remove-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -PassThru:$true
```

### <a name="update-tags-of-an-azure-cosmos-account"></a><a id="update-tags"></a> Обновление тегов учетной записи Azure Cosmos

Эта команда задает [теги ресурсов Azure][azure-resource-tags] для учетной записи Azure Cosmos. Теги можно задать при создании учетной записи с помощью `New-AzCosmosDBAccount`, а также при обновлении учетной записи с помощью `Update-AzCosmosDBAccount`.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$tags = @{dept = "Finance"; environment = "Production";}

Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Tag $tags
```

### <a name="list-account-keys"></a><a id="list-keys"></a> Вывод ключей учетной записи

При создании учетной записи Azure Cosmos служба создает два основных ключа доступа, которые можно использовать для проверки подлинности при доступе к учетной записи Azure Cosmos. Также создаются ключи только для чтения, используемые для выполнения проверки подлинности операций чтения.
Предоставляя два ключа, Azure Cosmos DB позволяет вам повторно создавать и (или) менять ключи без перерывов в работе учетной записи Azure Cosmos.
У каждой учетной записи Cosmos DB есть два ключа для чтения и записи (первичный и вторичный) и два ключа только для чтения (первичный и вторичный).

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "Keys"
```

### <a name="list-connection-strings"></a><a id="list-connection-strings"></a> Вывод строк подключения

Следующая команда возвращает строки подключения, позволяющие приложениям подключаться к учетной записи Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "ConnectionStrings"
```

### <a name="regenerate-account-keys"></a><a id="regenerate-keys"></a> Повторное создание ключей учетной записи

Для защиты подключений необходимо периодически повторно создавать ключи доступа к учетной записи Azure Cosmos. Для учетной записи назначаются первичный и вторичный ключи доступа. Это позволяет клиентам сохранять доступ, пока один из ключей создается повторно.
Существует четыре типа ключей для учетной записи Azure Cosmos (Primary, Secondary, PrimaryReadonly и SecondaryReadonly).

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "mycosmosaccount" # Must be all lower case
$keyKind = "primary" # Other key kinds: secondary, primaryReadOnly, secondaryReadOnly

New-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -KeyKind $keyKind
```

### <a name="enable-automatic-failover"></a><a id="enable-automatic-failover"></a> Включение автоматического перехода на другой ресурс

Следующая команда включает для учетной записи Cosmos DB автоматическую отработку отказа с переходом в дополнительный регион, если основной регион станет недоступным.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $true
$enableMultiMaster = $false

# First disable multi-region writes - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster

# Now enable automatic failover
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover
```

### <a name="modify-failover-priority"></a><a id="modify-failover-priority"></a> Изменение приоритета при отработке отказа

Для учетных записей, где настроена автоматическая отработка отказа, можно изменить порядок повышения вторичных реплик Cosmos до первичной реплики, когда та становится недоступной.

В приведенном ниже примере предполагается текущий приоритет для отработки отказа: `West US 2 = 0`, `East US 2 = 1`, `South Central US = 2`. Эта команда изменит его на `West US 2 = 0`, `South Central US = 1`, `East US 2 = 2`.

> [!CAUTION]
> Изменение расположения `failoverPriority=0` активирует переход на другой ресурс вручную для учетной записи Azure Cosmos. Любые другие изменения приоритета не приведут к активации отработки отказа.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("West US 2", "South Central US", "East US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="trigger-manual-failover"></a><a id="trigger-manual-failover"></a> Активация отработки отказа вручную

Для учетных записей, где настроена отработка отказа вручную, можно выполнить отработку отказа и повысить уровень вторичной реплики до первичной, изменив `failoverPriority=0`. Эту операцию можно использовать для инициации отработки аварийного восстановления, чтобы протестировать план аварийного восстановления.

В примере ниже предполагается, что учетная запись имеет приоритет `West US 2 = 0` и `East US 2 = 1` при отработке отказа, а затем регионы меняются.

> [!CAUTION]
> Если изменить свойство `locationName` для `failoverPriority=0`, для учетной записи Azure Cosmos активируется переход на другой ресурс вручную. Любые другие изменения приоритета не приведут к активации отработки отказа.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("East US 2", "West US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="list-resource-locks-on-an-azure-cosmos-db-account"></a><a id="list-account-locks"></a> Вывод списка блокировок ресурсов в учетной записи Azure Cosmos DB

Блокировки ресурсов можно применять к ресурсам Azure Cosmos DB, включая базы данных и коллекции. В примере ниже показано, как вывести список всех блокировок ресурсов Azure в учетной записи Azure Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceTypeAccount = "Microsoft.DocumentDB/databaseAccounts"
$accountName = "mycosmosaccount"

Get-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeAccount `
    -ResourceName $accountName
```

## <a name="azure-cosmos-db-database"></a>База данных Azure Cosmos DB

В следующих разделах описано управление базой данных Azure Cosmos DB, включая:

* [создание базы данных Azure Cosmos DB;](#create-db)
* [создание базы данных Azure Cosmos DB с общей пропускной способностью;](#create-db-ru)
* [получение сведений о пропускной способности для базы данных Azure Cosmos DB;](#get-db-ru)
* [Перенос пропускной способности базы данных в Автомасштабирование](#migrate-db-ru)
* [вывод списка всех баз данных Azure Cosmos DB в учетной записи;](#list-db)
* [получение сведений об одной базе данных Azure Cosmos DB;](#get-db)
* [удаление базы данных Azure Cosmos DB.](#delete-db)
* [создание блокировки ресурса в базе данных Azure Cosmos DB для предотвращения удаления;](#create-db-lock)
* [снятие блокировки ресурса в базе данных Azure Cosmos DB.](#remove-db-lock)

### <a name="create-an-azure-cosmos-db-database"></a><a id="create-db"></a>Создание базы данных Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-an-azure-cosmos-db-database-with-shared-throughput"></a><a id="create-db-ru"></a>Создание базы данных Azure Cosmos DB с общей пропускной способностью

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$databaseRUs = 400

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -Throughput $databaseRUs
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-database"></a><a id="get-db-ru"></a>Получение сведений о пропускной способности для базы данных Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabaseThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

## <a name="migrate-database-throughput-to-autoscale"></a><a id="migrate-db-ru"></a>Перенос пропускной способности базы данных в Автомасштабирование

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Invoke-AzCosmosDBSqlDatabaseThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -ThroughputType Autoscale
```

### <a name="get-all-azure-cosmos-db-databases-in-an-account"></a><a id="list-db"></a>Вывод списка всех баз данных Azure Cosmos DB в учетной записи

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName
```

### <a name="get-a-single-azure-cosmos-db-database"></a><a id="get-db"></a>Получение сведений об одной базе данных Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="delete-an-azure-cosmos-db-database"></a><a id="delete-db"></a>Удаление базы данных Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Remove-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-a-resource-lock-on-an-azure-cosmos-db-database-to-prevent-delete"></a><a id="create-db-lock"></a>Создание блокировки ресурса в базе данных Azure Cosmos DB для предотвращения удаления

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-database"></a><a id="remove-db-lock"></a>Снятие блокировки ресурса в базе данных Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="azure-cosmos-db-container"></a>Контейнер Azure Cosmos DB

В следующих разделах описаны процедуры управления контейнером Azure Cosmos DB, включая:

* [создание контейнера Azure Cosmos DB;](#create-container)
* [создание контейнера Azure Cosmos DB с возможностью автомасштабирования;](#create-container-autoscale)
* [создание контейнера Azure Cosmos DB с ключом большого раздела;](#create-container-big-pk)
* [получение сведений о пропускной способности контейнера Azure Cosmos DB;](#get-container-ru)
* [Перенос пропускной способности контейнера в Автомасштабирование](#migrate-container-ru)
* [создание контейнера Azure Cosmos DB с настраиваемой индексацией;](#create-container-custom-index)
* [создание контейнера Azure Cosmos DB с выключенной индексацией;](#create-container-no-index)
* [создание контейнера Azure Cosmos DB с уникальным ключом и сроком жизни;](#create-container-unique-key-ttl)
* [создание контейнера Azure Cosmos DB с разрешением конфликтов;](#create-container-lww)
* [вывод списка всех контейнеров Azure Cosmos DB в базе данных;](#list-containers)
* [получение сведений об одном контейнере Azure Cosmos DB в базе данных;](#get-container)
* [удаление контейнера Azure Cosmos DB.](#delete-container)
* [создание блокировки ресурса в контейнере Azure Cosmos DB для предотвращения удаления;](#create-container-lock)
* [снятие блокировки ресурса в контейнере Azure Cosmos DB.](#remove-container-lock)

### <a name="create-an-azure-cosmos-db-container"></a><a id="create-container"></a>Создание контейнера Azure Cosmos DB

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and throughput at 400 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$throughput = 400 #minimum = 400

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -Throughput $throughput
```

### <a name="create-an-azure-cosmos-db-container-with-autoscale"></a><a id="create-container-autoscale"></a>Создание контейнера Azure Cosmos DB с возможностью автомасштабирования

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and autoscale throughput at 4000 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$autoscaleMaxThroughput = 4000 #minimum = 4000

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -AutoscaleMaxThroughput $autoscaleMaxThroughput
```

### <a name="create-an-azure-cosmos-db-container-with-a-large-partition-key-size"></a><a id="create-container-big-pk"></a>Создание контейнера Azure Cosmos DB с ключом большого раздела

```azurepowershell-interactive
# Create an Azure Cosmos DB container with a large partition key value (version = 2)
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -PartitionKeyVersion 2
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-container"></a><a id="get-container-ru"></a>Получение сведений о пропускной способности контейнера Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainerThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="migrate-container-throughput-to-autoscale"></a><a id="migrate-container-ru"></a>Перенос пропускной способности контейнера в Автомасштабирование

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Invoke-AzCosmosDBSqlContainerThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -ThroughputType Autoscale
```

### <a name="create-an-azure-cosmos-db-container-with-custom-index-policy"></a><a id="create-container-custom-index"></a>Создание контейнера Azure Cosmos DB с настраиваемой политикой индексации

```azurepowershell-interactive
# Create a container with a custom indexing policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$indexPathIncluded = "/*"
$indexPathExcluded = "/myExcludedPath/*"

$includedPathIndex = New-AzCosmosDBSqlIncludedPathIndex -DataType String -Kind Range
$includedPath = New-AzCosmosDBSqlIncludedPath -Path $indexPathIncluded -Index $includedPathIndex

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IncludedPath $includedPath `
    -ExcludedPath $indexPathExcluded `
    -IndexingMode Consistent `
    -Automatic $true

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-indexing-turned-off"></a><a id="create-container-no-index"></a>Создание контейнера Azure Cosmos DB с выключенной индексацией

```azurepowershell-interactive
# Create an Azure Cosmos DB container with no indexing
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IndexingMode None

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-unique-key-policy-and-ttl"></a><a id="create-container-unique-key-ttl"></a>Создание контейнера Azure Cosmos DB с политикой создания уникального ключа и сроком жизни

```azurepowershell-interactive
# Create a container with a unique key policy and TTL of one day
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$uniqueKeyPath = "/myUniqueKeyPath"
$ttlInSeconds = 86400 # Set this to -1 (or don't use it at all) to never expire

$uniqueKey = New-AzCosmosDBSqlUniqueKey `
    -Path $uniqueKeyPath

$uniqueKeyPolicy = New-AzCosmosDBSqlUniqueKeyPolicy `
    -UniqueKey $uniqueKey

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -UniqueKeyPolicy $uniqueKeyPolicy `
    -TtlInSeconds $ttlInSeconds
```

### <a name="create-an-azure-cosmos-db-container-with-conflict-resolution"></a><a id="create-container-lww"></a>Создание контейнера Azure Cosmos DB с разрешением конфликтов

Для записи всех конфликтов в канал ConflictsFeed и их отдельной обработки передайте параметр `-Type "Custom" -Path ""`.

```azurepowershell-interactive
# Create container with last-writer-wins conflict resolution policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionPath = "/myResolutionPath"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type LastWriterWins `
    -Path $conflictResolutionPath

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```

Чтобы создать политику разрешения конфликтов для использования хранимой процедуры, вызовите `New-AzCosmosDBSqlConflictResolutionPolicy` и передайте параметры `-Type` и `-ConflictResolutionProcedure`.

```azurepowershell-interactive
# Create container with custom conflict resolution policy using a stored procedure
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionSprocName = "mysproc"

$conflictResolutionSproc = "/dbs/$databaseName/colls/$containerName/sprocs/$conflictResolutionSprocName"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type Custom `
    -ConflictResolutionProcedure $conflictResolutionSproc

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```


### <a name="list-all-azure-cosmos-db-containers-in-a-database"></a><a id="list-containers"></a>Вывод списка всех контейнеров Azure Cosmos DB в базе данных

```azurepowershell-interactive
# List all Azure Cosmos DB containers in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName
```

### <a name="get-a-single-azure-cosmos-db-container-in-a-database"></a><a id="get-container"></a>Получение сведений об одном контейнере Azure Cosmos DB в базе данных

```azurepowershell-interactive
# Get a single Azure Cosmos DB container in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="delete-an-azure-cosmos-db-container"></a><a id="delete-container"></a>Удаление контейнера Azure Cosmos DB

```azurepowershell-interactive
# Delete an Azure Cosmos DB container
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Remove-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```
### <a name="create-a-resource-lock-on-an-azure-cosmos-db-container-to-prevent-delete"></a><a id="create-container-lock"></a>Создание блокировки ресурса в контейнере Azure Cosmos DB для предотвращения удаления

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-container"></a><a id="remove-container-lock"></a>Снятие блокировки ресурса в контейнере Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="next-steps"></a>Дальнейшие действия

* [Все примеры для PowerShell](powershell-samples.md)
* [Управление учетной записью Azure Cosmos](how-to-manage-database-account.md)
* [Создание контейнера Azure Cosmos DB](how-to-create-container.md)
* [Настройка срока жизни в Azure Cosmos DB](how-to-time-to-live.md)

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[powershell-install-configure]: /powershell/azure/
[scaling-globally]: distribute-data-globally.md#EnableGlobalDistribution
[distribute-data-globally]: distribute-data-globally.md
[azure-resource-groups]: ../azure-resource-manager/management/overview.md#resource-groups
[azure-resource-tags]: ../azure-resource-manager/management/tag-resources.md
[rp-rest-api]: /rest/api/cosmos-db-resource-provider/