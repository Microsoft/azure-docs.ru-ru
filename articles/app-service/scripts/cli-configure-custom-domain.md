---
title: CLI. Сопоставление личного домена с приложением
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как сопоставить личный домен с приложением.
tags: azure-service-management
ms.assetid: 5ac4a680-cc73-4578-bcd6-8668c08802c2
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 3bf9e3ec9dbb042b85e4aa5ff2c65bc38125ce23
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782537"
---
# <a name="map-a-custom-domain-to-an-app-service-app-using-cli"></a>Сопоставление личного домена с приложением Службы приложений с помощью CLI

Этот пример сценария создает приложение в Службе приложений со связанными ресурсами, а затем сопоставляет с ним `www.<yourdomain>`.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/configure-custom-domain/configure-custom-domain.sh?highlight=3 "Map a custom domain to an app")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Создает приложение Службы приложений. |
| [`az webapp config hostname add`](/cli/azure/webapp/config/hostname#az_webapp_config_hostnam_eadd) | Сопоставляет личный домен с приложением Службы приложений. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
