---
title: Пример скрипта Azure CLI. Создание пользовательского раздела | Документация Майкрософт
description: В этой статье приведен пример скрипта Azure CLI, в котором показано, как создать пользовательский раздел в службе "Сетка событий Azure".
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: d61dac0e76fc7b7006fa36f1cb96965ccee3d863
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87494134"
---
# <a name="create-event-grid-custom-topic-with-azure-cli"></a>Создание пользовательского раздела службы "Сетка событий" с помощью Azure CLI

С помощью этого скрипта создается пользовательский раздел службы "Сетка событий".

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli[main](../../../cli_scripts/event-grid/create-custom-topic/create-custom-topic.sh "Create custom topic")]

## <a name="script-explanation"></a>Описание скрипта

Чтобы создать пользовательский раздел, в скрипте используются указанные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az eventgrid topic create](/cli/azure/eventgrid/topic#az-eventgrid-topic-create) | Создание пользовательского раздела службы "Сетка событий". |


## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Запрос к подпискам службы "Сетка событий Azure"](../query-event-subscriptions.md).
* Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
