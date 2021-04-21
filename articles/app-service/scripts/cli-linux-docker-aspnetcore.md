---
title: Интерфейс командной строки. создание приложения ASP.NET Core из Docker
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как создать приложение ASP.NET Core из Docker Hub.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 3a2d1983-ff7b-476a-ac44-49ec2aabb31a
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.author: msangapu
ms.custom: devx-track-dotnet, mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: d7f09d8e4b00a079fb012939e88ff585cbc88f4d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787797"
---
# <a name="create-an-aspnet-core-app-in-a-docker-container-from-docker-hub-using-azure-cli"></a>Создание приложения ASP.NET Core в контейнере Docker из Docker Hub с помощью Azure CLI

При помощи этого примера сценария создается группа ресурсов, план службы приложений Linux и приложение. Затем с помощью контейнера Docker развертывается приложение ASP.NET Core.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-linux-docker/deploy-linux-docker.sh?highlight=6 "Linux Docker")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, приложения Службы приложений и всех связанных с ними ресурсов этот сценарий использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Создает приложение Службы приложений. |
| [`az webapp config container set`](/cli/azure/webapp/config/container#az_webapp_config_container_set) | Настраивает контейнер Docker для приложения Службы приложений. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
