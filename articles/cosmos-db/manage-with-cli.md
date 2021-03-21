---
title: Управление ресурсами API Azure Cosmos DB Core (SQL) с помощью Azure CLI
description: Управление ресурсами API Azure Cosmos DB Core (SQL) с помощью Azure CLI.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.openlocfilehash: b13f5bfffced9afd80663d606e30e028e52643ac
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94563844"
---
# <a name="manage-azure-cosmos-core-sql-api-resources-using-azure-cli"></a>Управление ресурсами API Azure Cosmos Core (SQL) с помощью Azure CLI
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этом руководстве приведены команды Azure CLI, позволяющие автоматизировать управление учетными записями, базами данных и контейнерами Azure Cosmos DB. Страницы справки для всех команд интерфейса командной строки Azure Cosmos DB доступны в [справочнике по Azure CLI](/cli/azure/cosmosdb). Вы также можете найти несколько примеров в [Экземплярах Azure CLI для Azure Cosmos DB](cli-samples.md), в том числе и сведения о том, как создавать и управлять учетными записями, базами данных и контейнерами Cosmos DB для MongoDB, Gremlin, Cassandra и Table API.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.12.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

Примеры Azure CLI для других интерфейсов API см. в статьях [Примеры CLI для Cassandra](cli-samples-cassandra.md), [Примеры CLI для API MongoDB](cli-samples-mongodb.md), [Примеры CLI для Gremlin](cli-samples-gremlin.md) и [Примеры CLI для таблиц](cli-samples-table.md).

> [!IMPORTANT]
> Azure Cosmos DB ресурсы не могут быть переименованы, так как это нарушает работу Azure Resource Manager с URI ресурсов.

## <a name="azure-cosmos-accounts"></a>Учетные записи Azure Cosmos

В следующих разделах описано управление учетной записью Azure Cosmos, в том числе такие действия:

* [создание учетной записи Azure Cosmos;](#create-an-azure-cosmos-db-account)
* [Добавление и удаление регионов](#add-or-remove-regions)
* [Включить операции записи в несколько регионов](#enable-multiple-write-regions)
* [Задать приоритет для региона отработки отказа](#set-failover-priority)
* [Включить автоматический переход на другой ресурс](#enable-automatic-failover)
* [Запустить отработку отказа вручную](#trigger-manual-failover)
* [Вывод списка ключей учетной записи](#list-account-keys)
* [Список ключей учетной записи только для чтения](#list-read-only-account-keys)
* [Вывод строк подключения](#list-connection-strings)
* [Повторное создание ключей учетных записей](#regenerate-account-key)

### <a name="create-an-azure-cosmos-db-account"></a>Создание учетной записи Azure Cosmos DB

Создание учетной записи Azure Cosmos DB с помощью API SQL, согласованности сеансов в Западная часть США 2 и Восточная часть США 2.

> [!IMPORTANT]
> Имя учетной записи Azure Cosmos должно быть в нижнем регистре и содержать менее 44 символов.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount' #needs to be lower case and less than 44 characters

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --default-consistency-level Session \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False
```

### <a name="add-or-remove-regions"></a>Добавление и удаление регионов

Создайте учетную запись Azure Cosmos с двумя регионами, добавьте регион и удалите регион.

> [!NOTE]
> Вы не можете одновременно добавлять или удалять регионы `locations` и изменять другие свойства для учетной записи Azure Cosmos. Изменение регионов нужно выполнять как отдельную операцию, не объединяя ее с другими изменениями в ресурсе учетной записи.
> [!NOTE]
> Кроме того, эта команда позволяет добавлять или удалять регионы, но не изменять приоритеты при отработке отказа или активировать отработку отказа вручную. См. раздел [Установка приоритета](#set-failover-priority) отработки отказа и [триггера вручную](#trigger-manual-failover).

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Create an account with 2 regions
az cosmosdb create --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False

# Add a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False \
    --locations regionName="South Central US" failoverPriority=2 isZoneRedundant=False

# Remove a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False
```

### <a name="enable-multiple-write-regions"></a>Включить несколько регионов записи

Включение записи в несколько регионов для учетной записи Cosmos

```azurecli-interactive
# Update an Azure Cosmos account from single write region to multiple write regions
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-multiple-write-locations true
```

### <a name="set-failover-priority"></a>Задать приоритет отработки отказа

Задайте приоритет отработки отказа для учетной записи Azure Cosmos, настроенной для автоматического перехода на другой ресурс

```azurecli-interactive
# Assume region order is initially 'West US 2'=0 'East US 2'=1 'South Central US'=2 for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Make South Central US the next region to fail over to instead of East US 2
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'West US 2=0' 'South Central US=1' 'East US 2=2'
```

### <a name="enable-automatic-failover"></a> Включение автоматического перехода на другой ресурс

```azurecli-interactive
# Enable automatic failover on an existing account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-automatic-failover true
```

### <a name="trigger-manual-failover"></a>Запустить отработку отказа вручную

> [!CAUTION]
> Изменение региона с приоритетом 0 приведет к запуску отработки отказа вручную для учетной записи Azure Cosmos. Любые другие изменения приоритета не приведут к активации отработки отказа.

```azurecli-interactive
# Assume region order is initially 'West US 2=0' 'East US 2=1' 'South Central US=2' for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Trigger a manual failover to promote East US 2 as new write region
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'East US 2=0' 'South Central US=1' 'West US 2=2'
```

### <a name="list-all-account-keys"></a><a id="list-account-keys"></a> Вывод всех ключей учетной записи

Получение всех ключей для учетной записи Cosmos.

```azurecli-interactive
# List all account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
   -n $accountName \
   -g $resourceGroupName
```

### <a name="list-read-only-account-keys"></a>Список ключей учетной записи только для чтения

Получите ключи только для чтения для учетной записи Cosmos.

```azurecli-interactive
# List read-only account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type read-only-keys
```

### <a name="list-connection-strings"></a>Вывод строк подключения

Получение строк подключения для учетной записи Cosmos.

```azurecli-interactive
# List connection strings
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type connection-strings
```

### <a name="regenerate-account-key"></a>Повторное создание ключей учетных записей

Повторное создание нового ключа для учетной записи Cosmos.

```azurecli-interactive
# Regenerate secondary account keys
# key-kind values: primary, primaryReadonly, secondary, secondaryReadonly
az cosmosdb keys regenerate \
    -n $accountName \
    -g $resourceGroupName \
    --key-kind secondary
```

## <a name="azure-cosmos-db-database"></a>База данных Azure Cosmos DB

В следующих разделах описано управление базой данных Azure Cosmos DB, включая:

* [Создание базы данных](#create-a-database)
* [Создание базы данных с общей пропускной способностью](#create-a-database-with-shared-throughput)
* [Перенос базы данных на Автомасштабирование пропускной способности](#migrate-a-database-to-autoscale-throughput)
* [Изменить пропускную способность базы данных](#change-database-throughput)
* [Запретить удаление базы данных](#prevent-a-database-from-being-deleted)

### <a name="create-a-database"></a>Создание базы данных

Создайте базу данных Cosmos.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName
```

### <a name="create-a-database-with-shared-throughput"></a>Создание базы данных с общей пропускной способностью

Создайте базу данных Cosmos с общей пропускной способностью.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
throughput=400

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $throughput
```

### <a name="migrate-a-database-to-autoscale-throughput"></a>Перенос базы данных на Автомасштабирование пропускной способности

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

# Migrate to autoscale throughput
az cosmosdb sql database throughput migrate \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    -t 'autoscale'

# Read the new autoscale max throughput
az cosmosdb sql database throughput show \
    -g $resourceGroupName \
    -a $accountName \
    -n $databaseName \
    --query resource.autoscaleSettings.maxThroughput \
    -o tsv
```

### <a name="change-database-throughput"></a>Изменить пропускную способность базы данных

Увеличьте пропускную способность базы данных Cosmos на 1000 единиц запросов в секунду.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql database throughput show \
    -g $resourceGroupName -a $accountName -n $databaseName \
    --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql database throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $newRU
```

### <a name="prevent-a-database-from-being-deleted"></a>Запретить удаление базы данных

Установите блокировку удаления ресурса Azure для базы данных, чтобы предотвратить ее удаление. Эта функция требует, чтобы учетная запись Cosmos была изменена пакетами SDK для плоскости данных. Дополнительные сведения см. в статье [предотвращение изменений в](role-based-access-control.md#prevent-sdk-changes)пакетах SDK. Блокировки ресурсов Azure также могут препятствовать изменению ресурса путем указания `ReadOnly` типа блокировки. Для базы данных Cosmos ее можно использовать для предотвращения изменения пропускной способности.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
databaseLockName="$databaseName-Lock"

# Create a delete lock on database
az lock create --name $databaseLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/sqlDatabases \
    --lock-type $lockType \
    --parent $databaseParent \
    --resource $databaseName

# Delete lock on database
lockid=$(az lock show --name $databaseLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/sqlDatabases \
        --resource $databaseName \
        --parent $databaseParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="azure-cosmos-db-container"></a>Контейнер Azure Cosmos DB

В следующих разделах описаны процедуры управления контейнером Azure Cosmos DB, включая:

* [Создание контейнера](#create-a-container)
* [Создание контейнера с автомасштабированием](#create-a-container-with-autoscale)
* [Создание контейнера с включенным TTL](#create-a-container-with-ttl)
* [Создание контейнера с настраиваемой политикой индексирования](#create-a-container-with-a-custom-index-policy)
* [Изменение пропускной способности контейнера](#change-container-throughput)
* [Перенос контейнера в пропускную способность автомасштабирования](#migrate-a-container-to-autoscale-throughput)
* [Запретить удаление контейнера](#prevent-a-container-from-being-deleted)

### <a name="create-a-container"></a>Создание контейнера

Создайте контейнер Cosmos с политикой индексов по умолчанию, ключом секции и единицами запросов в секунду 400.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput
```

### <a name="create-a-container-with-autoscale"></a>Создание контейнера с автомасштабированием

Создайте контейнер Cosmos с политикой индексов по умолчанию, ключом секции и автомасштабированием единиц запросов в секунду для 4000.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
maxThroughput=4000

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --max-throughput $maxThroughput
```

### <a name="create-a-container-with-ttl"></a>Создание контейнера с TTL

Создайте контейнер Cosmos с включенным TTL.

```azurecli-interactive
# Create an Azure Cosmos container with TTL of one day
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

az cosmosdb sql container update \
    -g $resourceGroupName \
    -a $accountName \
    -d $databaseName \
    -n $containerName \
    --ttl=86400
```

### <a name="create-a-container-with-a-custom-index-policy"></a>Создание контейнера с настраиваемой политикой индексов

Создайте контейнер Cosmos с пользовательской политикой индексов, пространственным индексом, составным индексом, ключом секции и единицами запросов в секунду (400).

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

# Define the index policy for the container, include spatial and composite indexes
idxpolicy=$(cat << EOF
{
    "indexingMode": "consistent",
    "includedPaths": [
        {"path": "/*"}
    ],
    "excludedPaths": [
        { "path": "/headquarters/employees/?"}
    ],
    "spatialIndexes": [
        {"path": "/*", "types": ["Point"]}
    ],
    "compositeIndexes":[
        [
            { "path":"/name", "order":"ascending" },
            { "path":"/age", "order":"descending" }
        ]
    ]
}
EOF
)
# Persist index policy to json file
echo "$idxpolicy" > "idxpolicy-$uniqueId.json"


az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput \
    --idx @idxpolicy-$uniqueId.json

# Clean up temporary index policy file
rm -f "idxpolicy-$uniqueId.json"
```

### <a name="change-container-throughput"></a>Изменение пропускной способности контейнера

Увеличьте пропускную способность контейнера Cosmos на 1000 единиц запросов в секунду.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql container throughput show \
    -g $resourceGroupName -a $accountName -d $databaseName \
    -n $containerName --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql container throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    --throughput $newRU
```

### <a name="migrate-a-container-to-autoscale-throughput"></a>Перенос контейнера в пропускную способность автомасштабирования

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

# Migrate to autoscale throughput
az cosmosdb sql container throughput migrate \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    -t 'autoscale'

# Read the new autoscale max throughput
az cosmosdb sql container throughput show \
    -g $resourceGroupName \
    -a $accountName \
    -d $databaseName \
    -n $containerName \
    --query resource.autoscaleSettings.maxThroughput \
    -o tsv
```

### <a name="prevent-a-container-from-being-deleted"></a>Запретить удаление контейнера

Вставьте блокировку удаления ресурса Azure в контейнере, чтобы предотвратить его удаление. Эта функция требует, чтобы учетная запись Cosmos была изменена пакетами SDK для плоскости данных. Дополнительные сведения см. в статье [предотвращение изменений в](role-based-access-control.md#prevent-sdk-changes)пакетах SDK. Блокировки ресурсов Azure также могут препятствовать изменению ресурса путем указания `ReadOnly` типа блокировки. Для контейнера Cosmos это можно использовать для предотвращения изменения пропускной способности или других свойств.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
containerParent="databaseAccounts/$accountName/sqlDatabases/$databaseName"
containerLockName="$containerName-Lock"

# Create a delete lock on container
az lock create --name $containerLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/containers \
    --lock-type $lockType \
    --parent $containerParent \
    --resource $containerName

# Delete lock on container
lockid=$(az lock show --name $containerLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/containers \
        --resource-name $containerName \
        --parent $containerParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см.:

* [Установка Azure CLI](/cli/azure/install-azure-cli)
* [Справочник по Azure CLI](/cli/azure/cosmosdb)
* [Дополнительные примеры Azure CLI для Azure Cosmos DB](cli-samples.md)