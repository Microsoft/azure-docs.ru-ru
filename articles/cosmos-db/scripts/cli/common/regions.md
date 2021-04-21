---
title: Добавление регионов, изменение приоритета для отработки отказа, запуск отработки отказа для учетной записи Azure Cosmos
description: Добавление регионов, изменение приоритета для отработки отказа, запуск отработки отказа для учетной записи Azure Cosmos
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 4a1a061945fe1c6c6a95eb62d286d40a158281ca
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770829"
---
# <a name="add-regions-change-failover-priority-trigger-failover-for-an-azure-cosmos-account-using-azure-cli"></a>Добавление регионов, изменение приоритета для отработки отказа, запуск отработки отказа для учетной записи Azure Cosmos с помощью Azure CLI
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.9.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

В этом сценарии описаны три операции:

- добавление региона в имеющуюся учетную запись Azure Cosmos;
- изменение приоритета для отработки отказа (применяется к учетным записям, в которых используется автоматический переход на другой ресурс);
- активация перехода на другой ресурс вручную из основных в дополнительные регионы (применяется к учетным записям с переходом на другой ресурс вручную).

> [!NOTE]
> Операции добавления и удаления регионов в учетной записи Cosmos невозможно выполнить при изменении других свойств.

> [!NOTE]
> В этом примере демонстрируется использование учетной записи API SQL (Core), но эти операции идентичны во всех API-интерфейсах базы данных в Cosmos DB.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/common/regions.sh "Regional operations for Cosmos DB.")]

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
| [az cosmosdb update](/cli/azure/cosmosdb#az_cosmosdb_update) | Обновляет учетную запись Azure Cosmos DB (добавление и удаление региона). |
| [az cosmosdb failover-priority-change](/cli/azure/cosmosdb#az_cosmosdb_failover_priority_change) | Обновляет приоритет для отработки отказа или запускает отработку отказа для учетной записи Azure Cosmos DB. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Cosmos DB CLI см. в [этой документации](/cli/azure/cosmosdb).

Все примеры сценариев Azure Cosmos DB CLI можно найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
