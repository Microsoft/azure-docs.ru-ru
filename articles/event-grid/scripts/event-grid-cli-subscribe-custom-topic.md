---
title: Пример скрипта Azure CLI. Подписка на события пользовательского раздела | Документация Майкрософт
description: В этой статье приведен пример скрипта Azure CLI, в котором показано, как подписаться на события Сетки событий для пользовательского раздела.
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: f6004961a43fe13db617f98ae2c65fa5829924e5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766557"
---
# <a name="subscribe-to-events-for-a-custom-topic-with-azure-cli"></a>Создание подписки на события, связанные с пользовательским разделом, с использованием Azure CLI

С помощью этого скрипта в службе "Сетка событий" создается подписка на события, связанные с пользовательским разделом.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

Для примера скрипта предварительной версии требуется расширение службы "Сетка событий". Чтобы установить его, выполните команду `az extension add --name eventgrid`.

## <a name="sample-script---stable"></a>Пример скрипта — стабильная версия

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-custom-topic/subscribe-to-custom-topic.sh "Subscribe to custom topic")]

## <a name="sample-script---preview-extension"></a>Пример скрипта — расширение предварительной версии

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-custom-topic-preview/subscribe-to-custom-topic-preview.sh "Subscribe to custom topic")]


## <a name="script-explanation"></a>Описание скрипта

Чтобы создать подписку на события, в скрипте используются указанные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create) | создание подписки в службе "Сетка событий"; |
| [az eventgrid event-subscription create](/cli/azure/ext/eventgrid/eventgrid/event-subscription#ext-eventgrid-az-eventgrid-event-subscription-create) — версия расширения | создание подписки в службе "Сетка событий"; |

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Запрос к подпискам службы "Сетка событий Azure"](../query-event-subscriptions.md).
* Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
