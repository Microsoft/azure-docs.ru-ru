---
title: 'Azure CLI: мониторинг и масштабирование отдельной базы данных в Базе данных SQL Azure'
description: Используйте пример сценария Azure CLI для мониторинга и масштабирования отдельной базы данных в Базе данных SQL Azure.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 06/25/2019
ms.openlocfilehash: 0a72bea08052e5cd17502a21a98b41cb5582a3b9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96493639"
---
# <a name="use-the-azure-cli-to-monitor-and-scale-a-single-database-in-azure-sql-database"></a>Использование Azure CLI для мониторинга и масштабирования отдельной базы данных в Базе данных SQL Azure

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример сценария Azure CLI масштабирует отдельную базу данных в Базе данных SQL Azure до другого объема вычислительных ресурсов после запроса на получение сведений о размере базы данных.

Если вы решили установить и использовать интерфейс командной строки Azure локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Выполнение скрипта

[!code-azurecli-interactive[main](../../../../cli_scripts/sql-database/monitor-and-scale-database/monitor-and-scale-database.sh "Monitor and scale a database in Azure SQL Database")]

> [!TIP]
> Список операций, выполняемых в базе данных, можно получить при помощи команды [az sql db op list](/cli/azure/sql/db/op?#az-sql-db-op-list). Чтобы отменить операцию обновления в базе данных, воспользуйтесь командой [az sql db op cancel](/cli/azure/sql/db/op#az-sql-db-op-cancel).

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте приведенную ниже команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Скрипт | Описание |
|---|---|
| [az sql server](/cli/azure/sql/server) | Команды сервера. |
| [az sql db show-usage](/cli/azure/sql#az-sql-show-usage) | Отображает сведения об используемом размере базы данных. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Другие примеры сценариев CLI см. в статье [Примеры сценариев Azure CLI](../az-cli-script-samples-content-guide.md).
