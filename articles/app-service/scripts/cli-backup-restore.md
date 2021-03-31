---
title: CLI. Восстановление приложения из резервной копии
description: Сведения об использовании Azure CLI для автоматизации развертывания приложения Службы приложений и управления им. В этом примере показано, как восстановить приложение из резервной копии.
author: msangapu-msft
tags: azure-service-management
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/07/2017
ms.author: msangapu
ms.reviewer: cephalin
ms.custom: mvc, seodec18
ms.openlocfilehash: a8b7d20c3eee57d10a7025b05605603f82437cdb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97006427"
---
# <a name="restore-a-web-app-from-a-backup-using-cli"></a>Восстановление веб-приложения из резервной копии в интерфейсе командной строки

С помощью этого скрипта можно создать веб-приложение в службе приложений со связанными ресурсами, а затем однократно создать для него резервную копию. 

Для запуска этого скрипта требуется существующая резервная копия веб-приложения. Чтобы создать ее, см. инструкции по [резервному копированию веб-приложения](cli-backup-onetime.md) или [созданию резервной копии веб-приложения по расписанию](cli-backup-scheduled.md).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/backup-restore/backup-restore.sh?highlight=3-4,9 "Restore a web app from a backup")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [`az webapp config backup list`](/cli/azure/webapp/config/backup#az-webapp-config-backup-list) | Получение списка резервных копий веб-приложения. |
| [`az webapp config backup restore`](/cli/azure/webapp/config/backup#az-webapp-config-backup-restore) | Восстановление веб-приложения из резервной копии. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).
