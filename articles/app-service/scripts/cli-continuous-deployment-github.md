---
title: CLI. Непрерывное развертывание из GitHub
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как создать приложение с CI/CD из GitHub.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 0205c991-0989-4ca3-bb41-237dcc964460
ms.devlang: azurecli
ms.topic: sample
ms.date: 09/02/2019
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: b67d05951a4fc1a0ed2d625b2406ca5d0bc0fe34
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762305"
---
# <a name="create-an-app-service-app-with-continuous-deployment-from-github-using-cli"></a>Создание приложения Службы приложений с непрерывным развертыванием из GitHub с помощью интерфейса командной строки

Этот пример сценария создает приложение в Службе приложений со связанными ресурсами, а затем настраивает непрерывное развертывание из репозитория GitHub. Дополнительные сведения о развертывании из GitHub, которое не является непрерывным, см. в [этой статье](cli-deploy-github.md). Для этого примера вам потребуются следующие компоненты:

* Репозиторий GitHub с кодом приложения, для которого у вас есть права администратора. Чтобы получить автоматические сборки, структурируйте репозиторий в соответствии с таблицей [Prepare your repository](../deploy-continuous-deployment.md#prepare-your-repository) (Подготовка репозитория).
* [Личный маркер доступа](https://help.github.com/articles/creating-an-access-token-for-command-line-use) для учетной записи GitHub.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-github-continuous/deploy-github-continuous.sh?highlight=3-4 "Create an app with continuous deployment from GitHub")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Создает приложение Службы приложений. |
| [`az webapp deployment source config`](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config) | Связывает приложение Службы приложений с репозиторием Git или Mercurial. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
