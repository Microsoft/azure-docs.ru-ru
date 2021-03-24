---
title: Интерфейс командной строки. развертывание файлов приложения с помощью FTP
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как создать приложение и развернуть файлы с помощью FTP.
author: msangapu-msft
tags: azure-service-management
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/12/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 50a7d5e4a33762b5ae0725165e6062a83fec9110
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97006290"
---
# <a name="create-an-app-service-app-and-deploy-files-with-ftp-using-azure-cli"></a>Создание приложения Службы приложений и развертывание файлов с помощью протокола FTP в интерфейсе командной строки Azure

Этот пример сценария создает в Службе приложений приложение со связанными ресурсами, а затем развертывает статическую HTML-страницу с помощью протокола FTP. Для передачи по протоколу FTP в качестве примера в скрипте используется программа [cURL](https://en.wikipedia.org/wiki/CURL). Но вы можете использовать любые средства FTP для передачи файлов.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-ftp/deploy-ftp.sh "Create an app and deploy files with FTP")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта 

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az group create`](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az-appservice-plan-create) | Создает план службы приложений. |
| [`az webapp create`](/cli/azure/webapp#az-webapp-create) | Создает приложение Службы приложений. |
| [`az webapp deployment list-publishing-profiles`](/cli/azure/webapp/deployment#az-webapp-deployment-list-publishing-profiles) | Получает сведения о доступных профилях развертывания приложения. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
