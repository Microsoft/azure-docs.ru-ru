---
title: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов MongoDB
description: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов MongoDB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 10/07/2020
ms.openlocfilehash: 54aebb48986b94dbebea400e6b6ee155a54f514b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566220"
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
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create) | Создает учетную запись Azure Cosmos DB. |
| [az cosmosdb mongodb database create](/cli/azure/cosmosdb/mongodb/database#az-cosmosdb-mongodb-database-create) | Создает базу данных API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb collection create](/cli/azure/cosmosdb/mongodb/collection#az-cosmosdb-mongodb-collection-create) | Создает коллекцию API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb database throughput update](/cli/azure/cosmosdb/mongodb/database/throughput#az-cosmosdb-mongodb-database-throughput-update) | Обновляет ЕЗ/с для базы данных API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb collection throughput update](/cli/azure/cosmosdb/mongodb/collection/throughput#az-cosmosdb-mongodb-collection-throughput-update) | Обновляет ЕЗ/с для коллекции API MongoDB в Azure Cosmos DB. |
| [az cosmosdb mongodb database throughput migrate](/cli/azure/cosmosdb/mongodb/database/throughput#az-cosmosdb-mongodb-database-throughput-migrate) | Миграция пропускной способности для базы данных. |
| [az cosmosdb mongodb collection throughput migrate](/cli/azure/cosmosdb/mongodb/collection/throughput#az-cosmosdb-mongodb-collection-throughput-migrate) | Миграция пропускной способности для коллекции. |
| [az group delete](/cli/azure/resource#az-resource-delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Cosmos DB CLI см. в [этой документации](/cli/azure/cosmosdb).

Все примеры сценариев Azure Cosmos DB CLI можно найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
