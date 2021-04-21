---
title: Создание приложения-функции с помощью развертывания DevOps — Azure CLI
description: Создание приложения-функции и развертывание кода функции из Azure DevOps.
ms.date: 07/03/2018
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: f31c6a76412939d179cdd282e5e643ab7e8531b5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786231"
---
# <a name="create-a-function-in-azure-that-is-deployed-from-azure-devops"></a>Создание в Azure приложения-функции, которое развертывается из Azure DevOps

В этом разделе показано, как при помощи решения "Функции Azure" создать [бессерверное](https://azure.microsoft.com/solutions/serverless/) приложение-функцию с использованием [плана потребления](../consumption-plan.md). Приложение-функция, которое представляет собой контейнер для функций, непрерывно развертывается из репозитория Azure DevOps. 

Для работы с этой статьей требуется:

* репозиторий Azure DevOps с проектом приложения-функции и разрешением администратора на доступ к такому репозиторию;
* [личный маркер доступа](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) для доступа к репозиторию Azure DevOps.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="sample-script"></a>Пример скрипта

Этот пример создает приложение-функцию Azure и развертывает код функции из Azure DevOps.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/deploy-function-app-with-function-vsts/deploy-function-app-with-function-vsts.sh?highlight=3-4 "Azure Service")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, учетной записи хранения, приложения-функции и всех связанных ресурсов в этом скрипте используются приведенные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Создает учетную запись хранения, необходимую для приложения-функции. |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Создает приложение-функцию в бессерверном [плане потребления](../consumption-plan.md). |
| [az functionapp deployment source config](/cli/azure/functionapp/deployment/source#az_functionapp_deployment_source_config) | Связывает приложение-функцию с репозиторием Git или Mercurial. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для Функций Azure см. в [документации по Функциям Azure](../functions-cli-samples.md).
