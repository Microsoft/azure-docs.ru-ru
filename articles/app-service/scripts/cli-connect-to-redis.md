---
title: CLI. Подключение приложения к Redis
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как подключить приложение к Кэшу Azure для Redis.
author: msangapu-msft
tags: azure-service-management
ms.assetid: bc8345b2-8487-40c6-a91f-77414e8688e6
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 24658525ca1248aa82732a31eb9a40df1403203c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782429"
---
# <a name="connect-an-app-service-app-to-an-azure-cache-for-redis-using-cli"></a>Подключение приложения Службы приложений к кэшу Azure для Redis с помощью CLI

При помощи этого примера сценария создается кэш Azure для Redis и приложение Службы приложений. Затем кэш Azure для Redis связывается с приложением при помощи параметров приложения.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

Если вы решили установить и использовать CLI локально, вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/connect-to-redis/connect-to-redis.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, приложения Службы приложений, кэша Azure для Redis и всех связанных ресурсов в этом сценарии используются приведенные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Создает приложение Службы приложений. |
| [`az redis create`](/cli/azure/redis#az_redis-create) | Создает экземпляр кэша Redis для Azure. |
| [`az redis list-keys`](/cli/azure/redis#az_redis_list_keys) | Создает список ключей доступа для экземпляра кэша Redis для Azure. |
| [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) | Создает или обновляет параметр приложения Службы приложений. Параметры приложения представляются в качестве переменных среды для приложения. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
