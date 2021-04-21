---
title: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов MongoDB
description: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов MongoDB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 10/07/2020
ms.openlocfilehash: fc477fdd842025722a2bcfcd0172ab9d57fb0d4e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790723"
---
# <a name="throughput-rus-operations-with-azure-cli-for-a-database-or-graph-for-azure-cosmos-db-api-for-mongodb"></a>Выполнение операций с пропускной способностью (ЕЗ/с) через Azure CLI для базы данных или графа с использованием API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.12.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

Этот скрипт создает базу данных MongoDB с общей пропускной способностью и коллекцию с выделенной пропускной способностью, а затем обновляет пропускную способность для обоих элементов. Затем этот скрипт переключает режим пропускной способности со стандартного на автомасштабирование и считывает значение пропускной способности, установленное после переноса.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/mongodb/throughput.sh "Throughput operations for Azure Cosmos DB API for MongoDB.")]

## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) | Создает учетную запись Azure Cosmos DB. |
| [az cosmosdb mongodb database create](/cli/azure/cosmosdb/mongodb/database#az_cosmosdb_mongodb_database_create) | Создает базу данных API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb collection create](/cli/azure/cosmosdb/mongodb/collection#az_cosmosdb_mongodb_collection_create) | Создает коллекцию API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb database throughput update](/cli/azure/cosmosdb/mongodb/database/throughput#az_cosmosdb_mongodb_database_throughput_update) | Обновляет ЕЗ/с для базы данных API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb collection throughput update](/cli/azure/cosmosdb/mongodb/collection/throughput#az_cosmosdb_mongodb_collection_throughput_update) | Обновляет ЕЗ/с для коллекции API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb database throughput migrate](/cli/azure/cosmosdb/mongodb/database/throughput#az_cosmosdb_mongodb_database_throughput_migrate) | Миграция пропускной способности для базы данных. |
| [az cosmosdb mongodb collection throughput migrate](/cli/azure/cosmosdb/mongodb/collection/throughput#az_cosmosdb_mongodb_collection_throughput-migrate) | Миграция пропускной способности для коллекции. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Cosmos DB CLI см. в [этой документации](/cli/azure/cosmosdb).

Все примеры сценариев Azure Cosmos DB CLI можно найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
