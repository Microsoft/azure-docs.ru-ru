---
title: Примеры Azure CLI для API Cassandra Azure Cosmos DB
description: Примеры Azure CLI для API Cassandra Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: sample
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2dbae196b981b792b47e6c12e500398f772d8451
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93342136"
---
# <a name="azure-cli-samples-for-azure-cosmos-db-cassandra-api"></a>Примеры Azure CLI для API Cassandra в Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В следующей таблице содержатся ссылки на примеры сценариев Azure CLI для Azure Cosmos DB. Щелкните ссылки справа, чтобы перейти к примерам API. Общие примеры одинаковы для всех API. Страницы справки для всех команд интерфейса командной строки Azure Cosmos DB доступны в [справочнике по Azure CLI](/cli/azure/cosmosdb). Примеры сценариев Azure Cosmos DB CLI можно также найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).

Для выполнения этих примеров требуется Azure CLI версии 2.12.1 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, ознакомьтесь со статьей [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="common-samples"></a>Распространенные примеры

Эти примеры применимы ко всем интерфейсам API Azure Cosmos DB.

|Задача | Описание |
|---|---|
| [Добавление регионов или отработка отказа в регионах](scripts/cli/common/regions.md?toc=%2fcli%2fazure%2ftoc.json) | Добавляет регион, изменяет приоритет для отработки отказа и активирует переход на другой ресурс вручную.|
| [Ключи и строки подключения учетной записи](scripts/cli/common/keys.md?toc=%2fcli%2fazure%2ftoc.json) | Выводит ключи учетной записи, ключи только для чтения, повторно создает ключи и выводит строки подключения.|
| [Защита с помощью брандмауэра IP](scripts/cli/common/ipfirewall.md?toc=%2fcli%2fazure%2ftoc.json)| Создает учетную запись Cosmos с настроенным брандмауэром для IP-адресов.|
| [Защита новой учетной записи с помощью конечных точек службы](scripts/cli/common/service-endpoints.md?toc=%2fcli%2fazure%2ftoc.json)| Создает учетную запись Cosmos и обеспечивает безопасность с помощью конечных точек службы.|
| [Защита имеющейся учетной записи с помощью конечных точек службы](scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2fazure%2ftoc.json)| Обновляет учетную запись Cosmos для защиты с помощью конечных точек службы после настройки подсети.|
|||

## <a name="cassandra-api-samples"></a>Примеры API Cassandra

|Задача | Описание |
|---|---|
| [Создание учетной записи Azure Cosmos, пространства ключей и таблицы](scripts/cli/cassandra/create.md?toc=%2fcli%2fazure%2ftoc.json)| Создает учетную запись Azure Cosmos DB, пространство ключей и таблицу для API Cassandra. |
| [Создание учетной записи, пространства ключей и таблицы Azure Cosmos с автомасштабированием](scripts/cli/cassandra/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Создает учетную запись, пространство ключей и таблицу Azure Cosmos DB с автомасштабированием для API Cassandra. |
| [Операции с пропускной способностью](scripts/cli/cassandra/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Чтение и обновление значений пропускной способности, а также переход между автомасштабируемой и стандартной пропускной способностью пространства ключей и таблицы.|
| [Блокировка ресурсов от удаления](scripts/cli/cassandra/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Запрет на удаление ресурсов с помощью блокировок ресурсов.|
|||
