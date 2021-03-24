---
title: Интерфейс командной строки. развертывание приложения из GitHub
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как развернуть приложение из GitHub.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 0205c991-0989-4ca3-bb41-237dcc964460
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: ffbbeb9eac94cdc76c8086623bf16488d8a2d9e1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97006254"
---
# <a name="create-an-app-service-app-with-deployment-from-github-using-azure-cli"></a>Создание приложения Службы приложений с непрерывным развертыванием из GitHub в интерфейсе командной строки Azure

При помощи этого примера сценария в Службе приложений создается приложение со связанными ресурсами. Затем код приложения развертывается из общедоступного репозитория GitHub (без непрерывного развертывания). Дополнительные сведения о непрерывном развертывании из GitHub см. в статье [Create an App Service app with continuous deployment from GitHub using CLI](cli-continuous-deployment-github.md) (Создание приложения Службы приложений с непрерывным развертыванием из GitHub с использованием CLI).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-github/deploy-github.sh?highlight=3 "Create an app with deployment from GitHub")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта 

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az-appservice-plan-create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az-webapp-create) | Создает приложение Службы приложений. |
| [`az webapp deployment source config`](/cli/azure/webapp/deployment/source#az-webapp-deployment-source-config) | Связывает приложение Службы приложений с репозиторием Git или Mercurial. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
