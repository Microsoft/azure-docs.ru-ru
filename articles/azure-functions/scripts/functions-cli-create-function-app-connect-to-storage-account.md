---
title: Создание приложения-функции с подключенным хранилищем — Azure CLI
description: Пример скрипта Azure CLI для создания функции Azure, которая подключается к службе хранилища Azure.
ms.topic: sample
ms.date: 04/20/2017
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 13120ad4478febf9281ff423a3a7a8f8f3b25845
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97934413"
---
# <a name="create-a-function-app-with-a-named-storage-account-connection"></a>Создание приложения-функции с именованным подключением учетной записи хранения 

При помощи этого примера скрипта в решении "Функции Azure" создается приложение-функция. Затем функция подключается к учетной записи хранения Azure. Параметр созданного приложения с данными подключения можно использовать с [триггером хранилища или привязкой к нему](../functions-bindings-storage-blob.md). 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

Этот пример создает приложение-функцию Azure и добавляет строку подключения хранилища в параметры приложения.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-storage/create-function-app-connect-to-storage-account.sh "Integrate Function App into Azure Storage Account")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создайте группу ресурсов с расположением. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Создайте учетную запись хранения. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Создает приложение-функцию в бессерверном [плане потребления](../consumption-plan.md). |
| [az storage account show-connection-string](/cli/azure/storage/account#az-storage-account-show-connection-string) | Получает строку подключения для учетной записи. |
| [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set) | Задает строку подключения в качестве параметра приложения в приложении-функции. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для Функций Azure см. в [документации по Функциям Azure](../functions-cli-samples.md).
