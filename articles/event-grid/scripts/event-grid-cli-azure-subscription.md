---
title: Пример скрипта Azure CLI. Подписка на события подписки Azure | Документация Майкрософт
description: В этой статье приведен пример скрипта Azure CLI, в котором показано, как подписаться на события Сетки событий с помощью Azure CLI.
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: fe81be64ea4672d233564726cfdae5b60951057d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87494338"
---
# <a name="subscribe-to-events-for-an-azure-subscription-with-azure-cli"></a>Создание подписки на события, связанные с подпиской Azure, с использованием Azure CLI

С помощью этого скрипта в службе "Сетка событий" создается подписка на события, связанные с подпиской Azure.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

Для примера скрипта предварительной версии требуется расширение службы "Сетка событий". Чтобы установить его, выполните команду `az extension add --name eventgrid`.

## <a name="sample-script---stable"></a>Пример скрипта — стабильная версия

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-azure-subscription/subscribe-to-azure-subscription.sh "Subscribe to Azure subscription")]

## <a name="sample-script---preview-extension"></a>Пример скрипта — расширение предварительной версии

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-azure-subscription-preview/subscribe-to-azure-subscription-preview.sh "Subscribe to Azure subscription")]

## <a name="script-explanation"></a>Описание скрипта

Чтобы создать подписку на события, в скрипте используются указанные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create) | создание подписки в службе "Сетка событий"; |
| [az eventgrid event-subscription create](/cli/azure/ext/eventgrid/eventgrid/event-subscription#ext-eventgrid-az-eventgrid-event-subscription-create) — версия расширения | создание подписки в службе "Сетка событий"; |

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Запрос к подпискам службы "Сетка событий Azure"](../query-event-subscriptions.md).
* Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
