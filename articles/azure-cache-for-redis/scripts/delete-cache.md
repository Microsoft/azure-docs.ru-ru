---
title: Удаление кэша Azure для Redis — Azure CLI
description: В этом примере кода Azure CLI показано, как удалять Кэш Azure для Redis с помощью команды az redis delete.
author: yegu-ms
ms.author: yegu
tags: azure-service-management
ms.service: cache
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/30/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9ce6e20e5e4866bc4daa3f331fa2a8612ac6c260
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96184167"
---
# <a name="delete-an-azure-cache-for-redis"></a>Удаление кэша Azure для Redis

На примере этого скрипта вы узнаете, как удалить кэш Azure для Redis.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli[main](../../../cli_scripts/redis-cache/delete-cache/delete-cache.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

В этом скрипте используются следующие команды для удаления экземпляра кэша Azure для Redis. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az redis delete](/cli/azure/redis) | Удаляет экземпляр кэша Azure для Redis. |


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для кэша Azure для Redis см. в [документации по кэшу Azure для Redis](../cli-samples.md).