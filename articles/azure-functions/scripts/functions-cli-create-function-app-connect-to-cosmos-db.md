---
title: Создание приложения-функции, использующего Azure Cosmos DB, с помощью Azure CLI
description: Пример скрипта Azure CLI для создания функции Azure, которая подключается к Azure Cosmos DB.
ms.topic: sample
ms.date: 07/03/2018
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 261ede1493b93ccd3cfed19125bbabc3be871698
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786303"
---
# <a name="create-an-azure-function-that-connects-to-an-azure-cosmos-db"></a>Создание функции Azure, которая подключается к Azure Cosmos DB

При помощи этого примера скрипта, написанного с использованием решения "Функции Azure", создается приложение-функция. Затем функция подключается к базе данных Azure Cosmos DB. Параметр созданного приложения с данными подключения можно использовать с [триггером службы Azure Cosmos DB или привязкой к ней](../functions-bindings-cosmosdb.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="sample-script"></a>Пример скрипта

Этот пример создает приложение-функцию Azure, а также добавляет конечную точку и ключ доступа Cosmos DB в параметры приложения.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-cosmos-db/create-function-app-connect-to-cosmos-db.sh "Create an Azure Function that connects to an Azure Cosmos DB")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создайте группу ресурсов с расположением. |
| [az storage accounts create](/cli/azure/storage/account#az_storage_account_create) | Создание учетной записи хранения |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Создает приложение-функцию в бессерверном [плане потребления](../consumption-plan.md). |
| [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) | Создает базу данных Azure Cosmos DB. |
| [az cosmosdb show](/cli/azure/cosmosdb#az_cosmosdb_show)| Возвращает подключение к учетной записи базы данных. |
| [az cosmosdb list-keys](/cli/azure/cosmosdb#az_cosmosdb_list_keys)| Возвращает ключи для базы данных. |
| [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_set) | Задает строку подключения в качестве параметра приложения в приложении-функции. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев Azure CLI для Функций Azure см. в [документации по Функциям Azure](../functions-cli-samples.md).
