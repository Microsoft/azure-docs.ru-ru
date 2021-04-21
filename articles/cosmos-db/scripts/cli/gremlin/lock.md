---
title: Создание блокировки ресурсов для базы данных или графа Gremlin в Azure Cosmos DB
description: Создание блокировки ресурсов для базы данных или графа Gremlin в Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 19809d7150c8a461f97282d1583d0d870d6af8bb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770680"
---
# <a name="create-a-resource-lock-for-azure-cosmos-gremlin-api-database-and-graph-using-azure-cli"></a>Создание блокировки ресурсов для базы данных и графа API Gremlin в Azure Cosmos с помощью Azure CLI
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.9.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!IMPORTANT]
> Блокировки ресурсов не работают для изменений, внесенных пользователями, которые подключаются с помощью любого пакета SDK Gremlin или портала Azure, если только учетная запись Cosmos DB сначала не будет заблокирована с включенным свойством `disableKeyBasedMetadataWriteAccess`. Дополнительные сведения о том, как включить это свойство, см. в разделе [Предотвращение изменений в пакетах SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/gremlin/lock.sh "Create a resource lock for an Azure Cosmos DB Gremlin API database and graph.")]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az lock create](/cli/azure/lock#az_lock_create) | Создание блокировки. |
| [az lock list](/cli/azure/lock#az_lock_list) | Вывод сведений о блокировке. |
| [az lock show](/cli/azure/lock#az_lock_show) | Отображение свойств блокировки. |
| [az lock delete](/cli/azure/lock#az_lock_delete) | Удаление блокировки. |

## <a name="next-steps"></a>Дальнейшие действия

- [Блокировка ресурсов для предотвращения непредвиденных изменений](../../../../azure-resource-manager/management/lock-resources.md)

- [Документация по Azure Cosmos DB CLI](/cli/azure/cosmosdb).

- [Репозиторий GitHub для Azure Cosmos DB CLI](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
