---
title: Создание кэша Azure для Redis уровня "Премиум" с кластеризацией — Azure CLI
description: В этом примере кода Azure CLI показано, как создать кэш Azure для Redis уровня "Премиум" размером 6 ГБ с кластеризацией и двумя сегментами.
author: yegu-ms
ms.author: yegu
tags: azure-service-management
ms.service: cache
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/30/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: bb5a6ac082ebaf978023321f15341ec7f35779a6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96184232"
---
# <a name="create-a-premium-azure-cache-for-redis-with-clustering"></a>Создание кэша Azure для Redis уровня "Премиум" с кластеризацией

Этот сценарий демонстрирует, как создать кэш Azure для Redis уровня "Премиум" размером 6 ГБ с кластеризацией и двумя сегментами.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli[main](../../../cli_scripts/redis-cache/create-premium-cache-cluster/create-premium-cache-cluster.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов и кэша Azure для Redis уровня "Премиум" с кластеризацией в этом скрипте используются приведенные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az redis create](/cli/azure/redis) | Создает экземпляр кэша Azure для Redis. |


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для кэша Azure для Redis см. в [документации по кэшу Azure для Redis](../cli-samples.md).