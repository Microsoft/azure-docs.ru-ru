---
title: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов таблиц
description: Скрипты Azure CLI для операций с пропускной способностью (ЕЗ/с) с использованием API службы Azure Cosmos DB для ресурсов таблиц
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
ms.date: 10/07/2020
ms.openlocfilehash: 12ee46316a3eadceedf6f1772ae41938d1cc903e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107761944"
---
# <a name="throughput-rus-operations-with-azure-cli-for-a-table-for-azure-cosmos-db-table-api"></a>Операции с пропускной способностью (ЕЗ/с) в Azure CLI для таблиц с использованием API таблиц Azure Cosmos DB
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.12.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

Этот скрипт создает таблицу API таблиц, а затем обновляет пропускную способность для таблицы. Затем этот скрипт переключает режим пропускной способности со стандартного на автомасштабирование и считывает значение пропускной способности, установленное после переноса.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/table/throughput.sh "Throughput operations for Table API.")]

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
| [az cosmosdb table create](/cli/azure/cosmosdb/table#az_cosmosdb_table_create) | Создает таблицу API таблиц в Azure Cosmos DB. |
| [az cosmosdb table throughput update](/cli/azure/cosmosdb/table/throughput#az_cosmosdb_table_throughput_update) | Изменение пропускной способности для таблиц через API таблиц Azure Cosmos. |
| [az cosmosdb table throughput migrate](/cli/azure/cosmosdb/table/throughput#az_cosmosdb_table_throughput_migrate) | Миграция пропускной способности для таблиц через API таблиц Azure Cosmos. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Cosmos DB CLI см. в [этой документации](/cli/azure/cosmosdb).

Все примеры сценариев Azure Cosmos DB CLI можно найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
