---
title: Пример скрипта Azure CLI. Подписка на учетную запись хранения больших двоичных объектов | Документация Майкрософт
description: В этой статье приведен пример скрипта Azure CLI, в котором показано, как подписаться на события для учетной записи хранилища BLOB-объектов Azure.
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: ab0995a49f944344b6914bd0e51f1d437dd28c69
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766772"
---
# <a name="subscribe-to-events-for-a-blob-storage-account-with-azure-cli"></a>Создание подписки на события, связанные с учетной записью хранения больших двоичных объектов, с использованием Azure CLI

С помощью этого скрипта в службе "Сетка событий" создается подписка на события, связанные с учетной записью хранения больших двоичных объектов. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-blob-storage/subscribe-to-blob-storage.sh "Subscribe to Blob storage")]

## <a name="script-explanation"></a>Описание скрипта

Чтобы создать подписку на события, в скрипте используются указанные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create) | создание подписки в службе "Сетка событий"; |
| [az eventgrid event-subscription create](/cli/azure/ext/eventgrid/eventgrid/event-subscription#ext-eventgrid-az-eventgrid-event-subscription-create) — версия расширения | создание подписки в службе "Сетка событий"; |

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Запрос к подпискам службы "Сетка событий Azure"](../query-event-subscriptions.md).
* Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
