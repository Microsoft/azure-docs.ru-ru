---
title: Подключение общей папки к приложению-функции Python — Azure CLI
description: Сведения о том, как создать бессерверное приложение-функцию Python и подключить существующую общую папку с помощью Azure CLI.
ms.topic: sample
ms.date: 03/01/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 00bf0ecc9db3cf369fd75b427dcfba686aed0ed5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98035127"
---
# <a name="mount-a-file-share-to-a-python-function-app-using-azure-cli"></a>Подключение общей папки к приложению-функции Python с помощью Azure CLI

При помощи этого примера скрипта в решении "Функции Azure" создается приложение-функция и общая папка в службе "Файлы Azure". Он подключает общую папку, чтобы получить доступ к данным с помощью функций.  

>[!NOTE]
>Созданное приложение-функция выполняется в Python версии 3.7. Функции Azure также [поддерживают Python версий 3.6 и 3.8](../functions-reference-python.md#python-version).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="sample-script"></a>Пример скрипта

Этот скрипт создает приложение-функцию в Функциях Azure с использованием [плана потребления](../consumption-plan.md).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/functions-cli-mount-files-storage-linux/functions-cli-mount-files-storage-linux.sh "Create a function app on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Для каждой команды в таблице приведены ссылки на соответствующую документацию. Этот сценарий использует следующие команды:

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Создает учетную запись хранения Azure. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Создает приложение-функцию. |
| [az storage share create](/cli/azure/storage/share#az-storage-share-create) | Создает общую папку службы "Файлы Azure" в учетной записи хранения. | 
| [az storage directory create](/cli/azure/storage/directory#az-storage-directory-create) | Создает каталог в общей папке. |
| [az webapp config storage-account add](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) | Подключает общую папку к приложению-функции. |
| [az webapp config storage-account list](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-list) | Показывает общие папки, подключенные к приложению-функции. | 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для Функций Azure см. в [документации по Функциям Azure](../functions-cli-samples.md).
